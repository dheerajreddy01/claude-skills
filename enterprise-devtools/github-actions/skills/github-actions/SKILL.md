---
schema: "1.0"
name: github-actions
version: "1.0.0"
description: GitHub Actions CI/CD workflow design, secrets handling, and supply-chain security practices
domain: technology
triggers:
  keywords:
    primary: [github actions, CI/CD, workflow, pipeline, runner]
    secondary: [matrix build, self-hosted runner, composite action, workflow_dispatch]
  context_boost: [continuous integration, continuous deployment, build pipeline]
  context_penalty: [jenkins, gitlab ci, circleci]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# GitHub Actions

> Pipelines that don't leak secrets and don't run untrusted code with write access

## Applicable Scenarios

- Designing or reviewing CI/CD workflows (build, test, deploy)
- Handling secrets and credentials securely in pipelines
- Setting up matrix builds, caching, or reusable workflows
- Auditing third-party Action usage for supply-chain risk
- Diagnosing flaky, slow, or duplicate pipeline runs

## Core Knowledge

### Workflow Structure

```yaml
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make test
```

| Concept | Detail |
|------|------|
| **Triggers (`on:`)** | Events that start a workflow — `push`, `pull_request`, `schedule`, `workflow_dispatch` (manual), `workflow_call` (reusable) |
| **Jobs** | Run in parallel by default, on separate runners; use `needs:` to sequence |
| **Steps** | Sequential within a job, share the runner's filesystem |
| **Runners** | GitHub-hosted (ephemeral, clean per run) or self-hosted (persistent, your responsibility to secure) |

### Secrets

- Repository/organization/environment **secrets** are injected as environment variables at runtime, never written to logs by default (GitHub masks known secret values in log output automatically, but only for the literal value)
- **Environment protection rules** (required reviewers, wait timers) gate deploy-target secrets — use environments for anything touching production
- Secrets are **not** available to workflows triggered by `pull_request` from a fork by default — this is a deliberate security boundary (a fork's PR could otherwise exfiltrate secrets from the base repo's CI)

### The `pull_request_target` Trap

- `pull_request` runs with the PR's own code but no access to base-repo secrets
- `pull_request_target` runs with base-repo secrets/permissions but **checks out the PR's untrusted code by default if configured to do so** — combining the two (base-repo secrets + untrusted PR code) is the single most common GitHub Actions security mistake

### Caching & Matrix Builds

- `actions/cache` speeds up dependency installs by keying on a hash of the lockfile — a stale or incorrectly-keyed cache can silently serve outdated dependencies
- Matrix builds (`strategy.matrix`) run the same job across multiple parameter combinations (OS, language version) in parallel — useful for compatibility testing, but multiplies runner-minute cost

## Best Practices

1. **Pin third-party Actions to a full commit SHA**, not a mutable tag (`@v4` can be repointed by the Action's maintainer, or an attacker if the account is compromised)
2. **Never combine `pull_request_target` with checking out and running untrusted PR code** unless you fully understand and mitigate the risk
3. **Use environment protection rules** (required reviewers) for any workflow that deploys to production
4. **Scope `GITHUB_TOKEN` permissions explicitly** (`permissions:` block) — default is often broader than a given job needs
5. **Set `concurrency:` groups** to cancel superseded runs and prevent duplicate/racing deploys
6. **Cache dependencies, not secrets or build output that should always be fresh**

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Referencing a third-party Action by mutable tag (`@v1`, `@main`) | Pin to a full commit SHA, update deliberately via Dependabot |
| `pull_request_target` checking out PR code with base-repo secrets available | Avoid the combination; if unavoidable, strip secrets from that job entirely |
| No `permissions:` block, relying on the (often broad) default `GITHUB_TOKEN` scope | Set least-privilege `permissions:` explicitly per workflow/job |
| No `concurrency:` group on deploy workflows | Add one, so a superseded run cancels instead of racing a newer deploy |
| Secrets echoed into build logs via `run: echo $SECRET` for debugging | Never echo secrets, even temporarily — logs may be viewable by more people than the secret should be |

## Sharp Edges

### SE-1: `pull_request_target` Combined With Untrusted Code Checkout
- **Severity**: critical
- **Situation**: A workflow triggered by `pull_request_target` checks out the PR's head commit and runs its code (e.g., to build/test it), while still having access to the base repository's secrets — a malicious PR can exfiltrate those secrets or attack the CI environment
- **Cause**: `pull_request_target` was designed to let maintainers safely run label/triage automation on fork PRs with base-repo permissions, but checking out and executing the PR's own code defeats that safety boundary entirely
- **Symptoms**:
  - A security audit or `pull_request_target` usage scan flags a workflow that both checks out `github.event.pull_request.head.sha` and has secrets in scope
- **Solution**: Use `pull_request` (not `pull_request_target`) for anything that builds/runs a fork's code; if `pull_request_target` is genuinely needed (e.g., posting a comment), never check out the PR's code in that same job, or explicitly strip all secrets from any step that does
- **Details**: → [extended/checklists.md#security-checklist]

### SE-2: Unpinned Third-Party Actions as a Supply-Chain Risk
- **Severity**: critical
- **Situation**: A workflow references `some-org/some-action@v2`; the action's maintainer (or an attacker who compromises their account/repo) pushes malicious code to the `v2` tag, and every workflow using it now runs that code with whatever permissions/secrets the job has
- **Cause**: Git tags are mutable — pinning to a tag pins to whatever commit that tag currently points to, not a specific, immutable version
- **Symptoms**:
  - Supply-chain security scanners flag Actions pinned by tag rather than SHA
  - A previously-trusted Action's behavior changes without any corresponding change in your own repo
- **Solution**: Pin every third-party Action to a full commit SHA (`uses: some-org/some-action@a1b2c3d...`), and use Dependabot's Actions ecosystem support to get automated, reviewable update PRs

### SE-3: Overly Broad Default `GITHUB_TOKEN` Permissions
- **Severity**: high
- **Situation**: A workflow with no explicit `permissions:` block gets the repository's default token scope, which may include write access to contents, packages, or pull requests — far more than a simple build/test job needs
- **Cause**: Without an explicit `permissions:` block, `GITHUB_TOKEN` inherits the repository or organization's default permission setting, which is often broader than any individual workflow requires
- **Symptoms**:
  - A compromised dependency or malicious step in a low-trust job (e.g., running third-party code) can push commits, modify releases, or comment as the repo
- **Solution**: Set `permissions:` explicitly at the workflow or job level, scoped to read-only or the minimum write scope each job actually needs (e.g., `contents: read` for a test job, `contents: write` only for a release job)

### SE-4: Missing Concurrency Control Causing Racing Deploys
- **Severity**: high
- **Situation**: Two pushes to the same branch in quick succession trigger two overlapping deploy workflow runs, and the older one finishes *after* the newer one, overwriting the newer deploy with stale code
- **Cause**: Without a `concurrency:` group, GitHub Actions runs every triggered workflow instance independently and in parallel — nothing cancels a superseded run by default
- **Symptoms**:
  - Production occasionally reverts to an older version shortly after a newer deploy, with no corresponding rollback action taken
  - Two deploy runs show overlapping timestamps in the Actions history
- **Solution**: Add a `concurrency:` group keyed on the branch/environment (e.g., `concurrency: { group: deploy-prod, cancel-in-progress: true }`) so a new run cancels any in-flight superseded one

### SE-5: Stale or Poisoned Dependency Cache
- **Severity**: medium
- **Situation**: A cache key doesn't fully capture what should invalidate it (e.g., keyed only on branch name, not the lockfile hash), so a workflow silently uses outdated cached dependencies instead of what the lockfile actually specifies
- **Cause**: `actions/cache` restores whatever matches the given key (with fallback to partial/prefix matches) — an imprecise key can restore a cache that no longer matches the current dependency state
- **Symptoms**:
  - A dependency bump doesn't take effect in CI even though the lockfile changed
  - CI passes with an old dependency version that would fail if actually resolved fresh
- **Solution**: Key caches on a hash of the lockfile (`hashFiles('**/package-lock.json')` or equivalent) so any dependency change invalidates the cache correctly, and periodically verify a clean/no-cache run still passes

## Recommended Tools

| Category | Tools |
|------|------|
| Security scanning | Dependabot, `zizmor` (workflow security linter), CodeQL |
| Reusable workflows | `workflow_call`, composite actions |
| Self-hosted runner security | Ephemeral/autoscaling runners over persistent ones for public repos |
| Local testing | `act` (run workflows locally) |

## Security Hardening

**Full checklist**: → [extended/checklists.md#security-checklist]

## Related Resources

[GitHub Actions Docs](https://docs.github.com/en/actions) | [Security Hardening Guide](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions) | [Awesome Actions](https://github.com/sdras/awesome-actions)

## Related Domains

[[git]] | [[docker]] | [[terraform]] | [[kubernetes]]
