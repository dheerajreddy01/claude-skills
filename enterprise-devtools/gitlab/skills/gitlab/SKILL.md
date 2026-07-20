---
schema: "1.0"
name: gitlab
version: "1.0.0"
description: GitLab CI/CD pipeline design, runner architecture, and integrated Git platform practices
domain: technology
triggers:
  keywords:
    primary: [gitlab, gitlab ci, .gitlab-ci.yml, merge request]
    secondary: [gitlab runner, container registry, DAG, needs]
  context_boost: [continuous integration, git hosting, devops platform]
  context_penalty: [github actions, jenkins, circleci]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# GitLab

> One platform for Git hosting, CI/CD, and issue tracking — which means one set of rules governs whether your pipeline runs at all

## Applicable Scenarios

- Writing or reviewing `.gitlab-ci.yml` pipeline configuration
- Designing runner topology (shared vs. project-specific)
- Debugging why a pipeline job did or didn't trigger
- Managing CI/CD variables and secrets across environments
- Structuring job dependencies with `needs` for DAG-based pipelines

## Core Knowledge

### Pipeline Configuration

- `.gitlab-ci.yml` defines **stages** (sequential phases) and **jobs** (units of work within a stage) — by default, jobs in the same stage run in parallel, and stages run sequentially
- **`rules`** (the modern mechanism) determines whether a job runs, based on conditions like branch, pipeline source, or variable values — it superseded the older, more limited `only`/`except` keywords, and mixing both in the same job produces confusing, hard-to-predict behavior
- **`needs`** lets a job declare direct dependencies on specific other jobs (regardless of stage order), creating a DAG that can run faster than strict stage-by-stage sequencing — but it changes execution order assumptions that stage-based thinking doesn't prepare you for

### Runners

- A **GitLab Runner** is the agent that actually executes jobs; runners can be **shared** (available to many projects, managed centrally) or **specific** (registered to one project/group)
- Jobs are matched to runners via **tags** — a job requiring a tag with no matching available runner sits in `pending` indefinitely with no error, just silence
- Runner executors (Docker, Shell, Kubernetes, etc.) determine the actual execution environment and isolation level between jobs

### Pipeline Types by Trigger Source

| Trigger | When |
|------|------|
| **Branch pipeline** | A push to a branch |
| **Merge request pipeline** | Opening/updating a merge request — can run different jobs (`rules: if: $CI_PIPELINE_SOURCE == "merge_request_event"`) |
| **Scheduled pipeline** | Cron-like schedule |
| **Merged results / merge trains** | Runs against a merged simulation of the MR + target branch, catching integration issues before actual merge |

Fork-originated merge requests (from a contributor without direct repo access) run with restricted access to protected CI/CD variables by default — a deliberate security boundary that can trip up legitimate contributor workflows if not understood.

### Caching & Artifacts

- **Cache** persists dependencies (e.g., package manager caches) between pipeline runs, keyed explicitly — a poorly scoped cache key can serve stale dependencies across unrelated branches or dependency versions
- **Artifacts** pass build outputs between jobs/stages within the same pipeline run — they're not the same mechanism as cache and have different retention/scoping semantics

## Best Practices

1. **Use `rules` exclusively**, not a mix of `rules` and the legacy `only`/`except`, to keep trigger logic predictable
2. **Tag runners deliberately and verify tag matches exist** before relying on a job — a mismatched tag fails silently as `pending`, not as an error
3. **Scope CI/CD variables appropriately** — mark sensitive ones as "Protected" and "Masked," and understand what's available to merge request pipelines from forks
4. **Key caches precisely** (e.g., on a lockfile hash), not on something broad like the branch name alone
5. **Use `needs` deliberately** for jobs where DAG-based execution genuinely improves pipeline speed, understanding it changes ordering assumptions
6. **Use merge trains** for high-traffic repositories where late-breaking integration conflicts between concurrently-merging MRs are a real risk

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Mixing `rules` with legacy `only`/`except` in the same job | Use `rules` exclusively for predictable trigger behavior |
| Assuming a `pending` job will eventually error out if no runner matches | Verify runner tags exist and are available before relying on a job configuration |
| Broad cache keys (e.g., keyed only on branch name) | Key caches on a hash of the lockfile/dependency manifest |
| Assuming protected variables are available to merge request pipelines from forks | Understand and test the fork-MR variable restriction explicitly, don't assume parity with branch pipelines |
| Treating `needs` purely as a performance knob without considering ordering implications | Review what a `needs`-based DAG actually guarantees about execution order before relying on it |

## Sharp Edges

### SE-1: `rules`/`only`/`except` Interaction Producing Unexpected Trigger Behavior
- **Severity**: high
- **Situation**: A job runs when it shouldn't (or doesn't run when it should), and the root cause is a job definition mixing the legacy `only`/`except` keywords with the modern `rules` keyword, which have different evaluation semantics that don't compose intuitively
- **Cause**: `rules` and `only`/`except` are meant to be mutually exclusive configuration styles; GitLab's precedence behavior when both are present on the same job is a common source of surprises, since most engineers assume simple additive logic
- **Symptoms**:
  - A job's actual trigger behavior doesn't match what the YAML appears to specify
  - Debugging requires explicitly testing multiple trigger scenarios rather than reading the config
- **Solution**: Use `rules` exclusively across the pipeline, migrating any legacy `only`/`except` jobs deliberately, and test trigger conditions against real branch/MR/schedule scenarios rather than assuming from the YAML alone
- **Details**: → [extended/checklists.md#pipeline-reliability-checklist]

### SE-2: Runner Tag Mismatch Leaving Jobs Silently Pending Forever
- **Severity**: high
- **Situation**: A job requires a runner tag that no currently-available runner has, and instead of failing with a clear error, the job simply sits in `pending` status indefinitely
- **Cause**: GitLab's job-to-runner matching is tag-based, and there's no default timeout or explicit error for "no runner will ever match this tag" — it looks identical to "waiting for a busy runner to free up"
- **Symptoms**:
  - A pipeline appears "stuck," with one or more jobs never starting, and no error message explaining why
  - Investigation reveals the required tag doesn't exist on any registered, available runner
- **Solution**: Verify tag availability before introducing a new required tag in a job, keep a documented mapping of tags to actual registered runners, and set pipeline-level timeouts so a permanently-pending job is at least bounded rather than blocking indefinitely

### SE-3: CI/CD Variable Exposure in Merge Request Pipelines From Forks
- **Severity**: critical
- **Situation**: A pipeline running against a merge request from a forked/external contributor has access to protected CI/CD variables (deploy credentials, API keys) that it shouldn't, because the protection scope wasn't configured correctly
- **Cause**: Protected variables are, by default, restricted to protected branches/tags — but pipeline configuration around merge request pipelines from forks needs to be understood explicitly, since misconfiguration (e.g., overly permissive protected branch rules, or jobs that don't actually check pipeline source) can expose secrets to untrusted code execution
- **Symptoms**:
  - A security review finds that a fork-originated MR pipeline had access to a variable it shouldn't have
  - Untrusted code in a fork branch could exfiltrate a secret via a pipeline job
- **Solution**: Mark sensitive variables as "Protected" (available only on protected branches/tags) and "Masked," explicitly test what a fork-originated MR pipeline can and can't access, and avoid running jobs with access to sensitive variables directly against unreviewed external contributor code

### SE-4: Stale Cache Reuse Across Dependency Changes
- **Severity**: medium
- **Situation**: A dependency version bump doesn't take effect in CI because the pipeline reuses a cache keyed on something that didn't change (e.g., branch name), silently testing against outdated dependencies
- **Cause**: GitLab cache keys are whatever the pipeline configuration specifies — if the key doesn't incorporate something that changes when dependencies change (like a lockfile hash), the cache will be considered valid and reused even when its contents are stale
- **Symptoms**:
  - A dependency bump doesn't appear to take effect in CI test results
  - CI passes with an outdated dependency that would fail if actually resolved fresh
- **Solution**: Key caches on a hash of the lockfile/dependency manifest (`cache: key: files: [package-lock.json]`), so any dependency change invalidates the cache correctly

### SE-5: `needs`-Based DAG Producing Unintended Execution Order/Race Conditions
- **Severity**: medium
- **Situation**: A job configured with `needs` to skip ahead of the normal stage order runs before an assumption it implicitly depended on (e.g., a linting job that was expected to gate it) has completed, because `needs` only encodes the dependencies explicitly listed, not the full intended pipeline semantics
- **Cause**: `needs` creates a DAG based purely on the declared dependencies — it doesn't inherit any implicit ordering from stage position, so a job can start as soon as its `needs` list is satisfied, potentially running in parallel with jobs the team mentally assumed would gate it
- **Symptoms**:
  - A job runs and reports success despite a logically-prerequisite check (not listed in its `needs`) still running or having failed
- **Solution**: Explicitly list every genuine dependency in `needs` rather than relying on stage-order intuition, and review DAG-based pipelines for gaps between "what's declared" and "what the team assumes is guaranteed"

## Recommended Tools

| Category | Tools |
|------|------|
| Runner scaling | GitLab Runner Autoscaler, Kubernetes executor |
| Security | Secret Detection, Dependency Scanning (built into GitLab Ultimate/Premium tiers) |
| Local testing | `gitlab-ci-local` |
| Registry | GitLab Container Registry, Package Registry |

## Pipeline Reliability

**Full checklist**: → [extended/checklists.md#pipeline-reliability-checklist]

## Related Resources

[GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/) | [Pipeline Efficiency Guide](https://docs.gitlab.com/ee/ci/pipelines/pipeline_efficiency.html) | [GitLab Runner Docs](https://docs.gitlab.com/runner/)

## Related Domains

[[git]] | [[jenkins]] | [[github-actions]] | [[docker]]
