---
schema: "1.0"
name: postman
version: "1.0.0"
description: Postman collection design, environment/secret management, and CI-driven API testing with Newman
domain: technology
triggers:
  keywords:
    primary: [postman, API testing, collection, newman]
    secondary: [environment variable, pre-request script, mock server, API design]
  context_boost: [API endpoint, REST client, integration test]
  context_penalty: [jira, splunk, pagerduty]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Postman

> Collections as living API documentation — worth nothing if secrets are baked in or tests don't assert anything

## Applicable Scenarios

- Designing Postman collections for API exploration, documentation, or testing
- Managing environment variables and secrets across dev/staging/prod
- Writing pre-request and test scripts for request chaining and validation
- Running collections in CI via Newman
- Setting up mock servers for frontend/backend parallel development

## Core Knowledge

### Collections & Requests

- A **collection** is an organized set of requests, often mirroring an API's resource structure — collections can be version-controlled (exported as JSON) or synced via Postman's cloud workspace
- **Folders** within a collection group related requests and can share folder-level pre-request scripts/auth
- Requests reference **variables** (`{{baseUrl}}`, `{{authToken}}`) rather than hardcoded values, so the same collection runs against any environment

### Environments & Variables

| Scope | Use |
|------|------|
| **Global** | Values shared across all collections/environments (rarely appropriate for secrets) |
| **Environment** | Values specific to one target (dev/staging/prod) — base URL, API keys |
| **Collection** | Defaults for a specific collection, overridable by environment |
| **Local/Runtime** | Set dynamically during a run (e.g., a token captured from a login response) |

Variable resolution follows a precedence order (local > environment > collection > global) — a value set at a higher-precedence scope silently shadows a lower one, which can cause confusing "why isn't my environment variable being used" debugging.

### Scripts

- **Pre-request scripts** run before the request fires — commonly used to compute a signature, refresh a token, or set a dynamic variable
- **Test scripts** run after the response arrives — use `pm.test()` assertions to validate status codes, response shape, and values; these are what make a collection a *test suite* rather than just a request library
- Scripts can read/write environment and collection variables, enabling request chaining (e.g., capture an ID from a POST response, use it in the next request)

### Newman (CLI Runner)

- Newman runs a Postman collection headlessly from the command line — the mechanism for running collections in CI/CD pipelines
- Exit code reflects whether all `pm.test()` assertions passed — a non-zero exit code should fail the build
- Environment/secret values for CI runs are typically passed via `-e environment.json` or `--env-var`, sourced from the CI system's secret store, not committed alongside the collection

## Best Practices

1. **Never hardcode secrets in a request or committed environment file** — use environment variables sourced from a secure store, and mark sensitive variables as "secret" type in Postman (masks them in the UI, excludes from some exports)
2. **Write real assertions in test scripts** — a collection with no `pm.test()` calls, or one that never actually checks anything meaningful, gives false confidence
3. **Version-control exported collections** (JSON) alongside the API code, so the collection and the API evolve together and reviewable in PRs
4. **Run collections via Newman in CI** on every relevant change, and fail the build on assertion failure
5. **Use folder-level or collection-level auth/variables** to avoid duplicating the same setup across every request
6. **Use mock servers** for frontend development to unblock work before the real backend endpoint exists

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| API key hardcoded directly in a request URL/header | Reference an environment variable, sourced from a secret store for CI |
| Exporting and sharing an environment file that contains live credentials | Mark sensitive variables as "secret," strip values before sharing/exporting, or use variable scopes that exclude secrets from exports |
| Test scripts with no `pm.test()` assertions (or one trivial always-true check) | Assert on status code, response schema, and key field values meaningfully |
| Collection kept only in Postman's cloud with no version-controlled export | Export and commit the collection JSON alongside the API code for review and history |
| Newman run in CI but its exit code ignored | Fail the CI job explicitly on a non-zero Newman exit code |

## Sharp Edges

### SE-1: Secrets Hardcoded and Shared via Exported Collections
- **Severity**: critical
- **Situation**: A collection or environment file is exported and shared (via Slack, a shared drive, or committed to a public repo) with live API keys or tokens embedded directly in request headers or variable values
- **Cause**: It's easy to paste a working credential directly into a request while testing, and forget it's now baked into anything the collection/environment gets exported to
- **Symptoms**:
  - A secret scanner or manual review finds a live credential inside a `.postman_collection.json` or `.postman_environment.json` file
- **Solution**: Mark sensitive variables as "secret" type (Postman masks and excludes them from some export paths), source real credentials from a secure store at runtime rather than storing them in the collection/environment file itself, and treat any discovered exposure as a credential requiring immediate rotation
- **Details**: → [extended/checklists.md#collection-hygiene-checklist]

### SE-2: Test Scripts That Don't Actually Assert Anything
- **Severity**: high
- **Situation**: A collection has test scripts attached to every request and shows green in Newman/CI runs, but the scripts only check that a response was received — not that it's actually correct — so a broken API change sails through undetected
- **Cause**: `pm.test()` blocks that don't contain a real assertion (or assert something trivially always true, like checking the response object exists) pass regardless of the actual response content
- **Symptoms**:
  - Newman/CI shows all green, but a manually-verified bug in the API response wasn't caught
  - Test scripts consist mostly of `console.log` or status-only checks with no schema/value validation
- **Solution**: Write meaningful assertions — status code, response schema (via `pm.response.to.have.jsonSchema` or explicit field checks), and specific value checks for anything business-critical — and periodically review test scripts for coverage gaps, not just pass/fail status

### SE-3: Environment Variable Leakage Across Environments
- **Severity**: high
- **Situation**: A test run intended for staging accidentally executes against production (or vice versa) because the wrong environment was selected, and destructive test requests (e.g., a `DELETE` in a test suite) run against real data
- **Cause**: Postman doesn't prevent running a collection against any environment currently selected in the UI/CLI invocation — nothing inherently stops a staging-only test suite from being pointed at a prod `baseUrl`
- **Symptoms**:
  - Unexpected data changes/deletions in production correlate with a test run
  - A CI job's environment file argument was copy-pasted incorrectly from another job
- **Solution**: Use clearly distinct, unambiguous environment names, add a safety check in pre-request scripts that refuses to run destructive requests against a production-flagged environment, and double-check the environment argument explicitly in CI job configuration rather than relying on defaults

### SE-4: Collection Drifting From Actual API Behavior
- **Severity**: medium
- **Situation**: A Postman collection meant to serve as living API documentation gradually diverges from the real API as endpoints change, and new team members (or external API consumers) get misled by stale examples
- **Cause**: Nothing automatically keeps a manually maintained collection in sync with the actual API implementation — updates to the collection are a separate, easy-to-forget step from updating the API code
- **Symptoms**:
  - A request in the collection returns an error or unexpected shape when run against the current API
  - Onboarding engineers report the "documented" API doesn't match what they observe
- **Solution**: Run the collection in CI regularly (not just on-demand) so drift surfaces as test failures quickly, and treat collection updates as part of the same PR that changes the corresponding API endpoint

### SE-5: Newman CI Failures Ignored or Not Actually Blocking Merges
- **Severity**: medium
- **Situation**: Newman runs in CI and reports failures, but the pipeline is configured in a way that doesn't actually block the merge/deploy (e.g., a non-blocking job, or exit code not checked)
- **Cause**: A CI step can "run" without its result being wired into the actual pass/fail gate for the pipeline — this is a pipeline configuration issue, not a Newman one, but it silently defeats the entire point of running the tests
- **Symptoms**:
  - Newman shows failed assertions in CI logs, but the PR/deploy proceeds anyway
  - Nobody notices API test failures until a manual investigation, despite them being visible in CI history all along
- **Solution**: Verify the CI job explicitly checks Newman's exit code and fails the build/pipeline on non-zero, and periodically audit CI configuration to confirm test jobs are actually gating, not just informational

## Recommended Tools

| Category | Tools |
|------|------|
| CI runner | Newman, Newman HTML/JUnit reporters |
| Collection management | Postman CLI, Postman API (programmatic collection sync) |
| Mocking | Postman Mock Servers |
| Alternatives | Insomnia, Bruno, HTTPie (for lighter-weight needs) |

## Collection Hygiene

**Full checklist**: → [extended/checklists.md#collection-hygiene-checklist]

## Related Resources

[Postman Learning Center](https://learning.postman.com/) | [Newman Docs](https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/) | [Postman Scripting Reference](https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/)

## Related Domains

[[github-actions]] | [[javascript]] | [[python]]
