---
schema: "1.0"
name: jenkins
version: "1.0.0"
description: Jenkins pipeline design, plugin management, agent architecture, and credential security practices
domain: technology
triggers:
  keywords:
    primary: [jenkins, jenkinsfile, pipeline, groovy]
    secondary: [blue ocean, shared library, agent, executor]
  context_boost: [continuous integration, build server, self-hosted CI]
  context_penalty: [github actions, gitlab ci, circleci]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# Jenkins

> A build server whose flexibility is exactly its operational liability

## Applicable Scenarios

- Writing or reviewing declarative or scripted Jenkinsfiles
- Managing plugin upgrades and compatibility on a Jenkins instance
- Designing agent/executor topology for build capacity
- Handling credentials securely within pipelines
- Diagnosing flaky, hung, or resource-starved builds

## Core Knowledge

### Pipeline Types

| Type | Style | Best For |
|------|------|------|
| **Declarative** | Structured `pipeline { stages { ... } }` syntax, opinionated | Most teams — easier to read, lint, and restrict |
| **Scripted** | Full Groovy control flow | Complex conditional logic the declarative DSL can't express |

A `Jenkinsfile` is checked into source control alongside the code it builds — pipeline-as-code, versioned and reviewable like any other file.

### Plugin Ecosystem

- Jenkins's core is deliberately minimal; almost all real functionality (Git integration, Docker, credentials, notifications) comes from plugins
- Plugins have their own version compatibility matrix against the Jenkins core version and against each other — this is the single biggest source of Jenkins operational pain, since an upgrade to one plugin can silently break another
- The **Script Security plugin** sandboxes Groovy execution in pipelines and requires explicit admin approval for anything it considers potentially unsafe (accessing certain Java APIs, etc.)

### Agent & Executor Model

- A **controller** (formerly "master") schedules work; **agents** (formerly "slaves") are the machines that actually execute build steps
- Each agent has a configured number of **executors** — concurrent build slots — and pipelines can request a specific agent by label (`agent { label 'docker' }`)
- Running builds directly on the controller (no agent restriction) is a common anti-pattern — it competes with Jenkins's own scheduling/UI responsiveness and, worse, gives every pipeline access to whatever the controller process can reach

### Credentials

- The **Credentials plugin** stores secrets (tokens, SSH keys, usernames/passwords) and pipelines reference them by ID (`credentials('my-token')`) rather than embedding values directly
- Credentials bound into environment variables via `withCredentials` are masked in the console log automatically — but only the exact bound value; any transformation of it (base64-encoding, substring, concatenation with other text) is not masked and can leak into logs

## Best Practices

1. **Use declarative pipelines by default**, reserving scripted syntax for logic that genuinely needs it
2. **Pin plugin versions deliberately** and test upgrades in a non-production Jenkins instance before rolling out
3. **Never echo or print credential values**, even transformed ones — assume anything not passed through `withCredentials`'s exact binding can end up in logs
4. **Restrict every pipeline to a labeled agent**, never letting builds run unrestricted on the controller
5. **Use shared libraries** for logic duplicated across many Jenkinsfiles, but version and test them like any other shared dependency
6. **Set build timeouts and executor limits** so a single hung or runaway job can't starve the rest of the queue

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Running pipeline steps with no `agent` label restriction | Always specify a labeled agent, never let work default to the controller |
| Echoing a credential (even transformed) to the console for debugging | Never print credential values in any form; use `withCredentials` bindings only where needed |
| Upgrading plugins in production without a staging test | Test plugin upgrades on a non-production instance first, given the compatibility matrix risk |
| Blanket-approving Script Security sandbox requests without review | Review each approval request individually — it exists specifically to catch unsafe Groovy |
| No build timeout set | Set `options { timeout(...) }` so a hung build can't hold an executor indefinitely |

## Sharp Edges

### SE-1: Plugin Version Incompatibility Breaking the Instance on Upgrade
- **Severity**: critical
- **Situation**: Upgrading one plugin (or Jenkins core itself) breaks a different, seemingly unrelated plugin, and pipelines across the entire instance start failing
- **Cause**: Plugins depend on specific version ranges of Jenkins core and of each other; the compatibility matrix isn't fully enforced at upgrade time, so an upgrade can succeed and only fail at runtime
- **Symptoms**:
  - Pipelines that worked yesterday fail with plugin-related stack traces after a routine upgrade
  - Jenkins startup logs show plugin loading warnings/errors that were easy to miss during the upgrade
- **Solution**: Test upgrades on a non-production Jenkins instance with a representative plugin set before applying to production, upgrade plugins incrementally rather than in one large batch, and keep a rollback plan (config/plugin backup) ready before any upgrade
- **Details**: → [extended/checklists.md#operational-safety-checklist]

### SE-2: Credentials Leaking Into Build Logs via Unmasked Transformation
- **Severity**: critical
- **Situation**: A credential bound via `withCredentials` is masked correctly when printed directly, but a script that base64-encodes it, concatenates it into a URL, or substrings it for debugging leaks the value in plaintext in the console log
- **Cause**: Jenkins's log masking matches the exact literal string bound to the credential variable — any transformation produces a different string that the masking doesn't recognize
- **Symptoms**:
  - A security review of build logs finds a recognizable (if transformed) credential value
  - The leak wasn't caught because the raw credential variable itself never appeared unmasked
- **Solution**: Never transform, echo, or log a credential value in any form, even for debugging; if a transformed form must be used (e.g., a base64-encoded auth header), construct it in a way that never touches the console (write directly to a file/env var consumed by the next step without an intermediate `echo`)

### SE-3: Shared Library Changes Breaking Every Pipeline That Uses Them
- **Severity**: high
- **Situation**: A change to a Jenkins shared library (used across many Jenkinsfiles for common logic) breaks pipelines organization-wide simultaneously, because every consumer references the same unpinned branch
- **Cause**: Shared libraries are commonly referenced by branch name (e.g., `master`) rather than a pinned version/tag, so any change to the library takes effect immediately for every pipeline that uses it, with no staged rollout
- **Symptoms**:
  - Many unrelated pipelines start failing at the same time, all pointing to the same shared library step
- **Solution**: Version shared libraries with tags and have consumers pin to a specific version, test shared library changes against representative consumer pipelines before merging, and treat shared library changes with the same review rigor as changes to production code

### SE-4: Agent Resource Exhaustion From Unbounded Concurrent Builds
- **Severity**: high
- **Situation**: A burst of triggered builds (e.g., many PRs pushed at once) exhausts available executors, and builds queue for a long time or agents run out of memory/disk trying to handle more concurrent work than they can support
- **Cause**: Without per-agent or per-pipeline concurrency limits, Jenkins will schedule as many builds as there are available executors, and nothing inherently prevents oversubscribing an agent's actual resource capacity relative to its configured executor count
- **Symptoms**:
  - Agents show high memory/disk pressure correlated with build bursts
  - Build queue times spike unpredictably during high-activity periods
- **Solution**: Size executor count per agent to genuine resource capacity (not just "more is better"), use `options { disableConcurrentBuilds() }` or throttle plugins for pipelines that shouldn't run many instances simultaneously, and monitor agent resource usage as a standard operational signal

### SE-5: Script Security Approval Fatigue Leading to Blanket Trust
- **Severity**: medium
- **Situation**: Admins get repeated Script Security sandbox approval requests for pipeline scripts, and over time start approving them reflexively without review, defeating the sandbox's purpose
- **Cause**: The sandbox flags any Groovy construct it can't statically prove safe, which in practice includes a lot of legitimate code — the resulting approval queue volume creates pressure to rubber-stamp requests
- **Symptoms**:
  - Approval history shows a pattern of rapid, low-scrutiny approvals
  - A genuinely unsafe script (e.g., one that could access credentials store internals or the file system broadly) gets approved without being caught
- **Solution**: Review each approval request for what it actually does before approving, prefer well-tested shared library functions over ad hoc inline Groovy that keeps triggering new approval requests, and periodically audit the list of approved signatures for anything that shouldn't have been granted

## Recommended Tools

| Category | Tools |
|------|------|
| Pipeline authoring | Blue Ocean, Jenkinsfile Runner (local testing) |
| Security | Script Security plugin, Credentials Binding plugin |
| Scaling | Kubernetes plugin (dynamic agents), Jenkins Configuration as Code (JCasC) |
| Monitoring | Jenkins Prometheus plugin, Monitoring plugin |

## Operational Safety

**Full checklist**: → [extended/checklists.md#operational-safety-checklist]

## Related Resources

[Jenkins Docs](https://www.jenkins.io/doc/) | [Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/) | [Jenkins Security Advisories](https://www.jenkins.io/security/advisories/)

## Related Domains

[[github-actions]] | [[gitlab]] | [[circleci]] | [[docker]]
