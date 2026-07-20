---
schema: "1.0"
name: circleci
version: "1.0.0"
description: CircleCI workflow design, orb usage, caching strategy, and test parallelism practices
domain: technology
triggers:
  keywords:
    primary: [circleci, circle ci, orb, workflow]
    secondary: [executor, context, parallelism, config.yml]
  context_boost: [continuous integration, build pipeline]
  context_penalty: [github actions, gitlab ci, jenkins]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# CircleCI

> Orbs and caching make CircleCI fast to write and easy to make silently wrong

## Applicable Scenarios

- Writing or reviewing `.circleci/config.yml` jobs and workflows
- Choosing executor types and designing caching strategy
- Managing secrets via contexts across projects
- Configuring test splitting/parallelism
- Diagnosing flaky, slow, or resource-inefficient pipelines

## Core Knowledge

### Config Structure

- **Jobs** define a sequence of steps executed in a chosen environment; **workflows** orchestrate jobs (parallel, sequential, fan-out/fan-in, with approval gates)
- **Orbs** are shareable, reusable packages of config (commands, jobs, executors) — the CircleCI equivalent of a shared library, published to a registry and versioned independently of the consuming project
- Orbs can be pulled by a loose version range (e.g., `circleci/node@5`) — this trades stability for automatically getting fixes, and can also mean automatically getting breaking changes

### Executors

| Executor | Environment |
|------|------|
| **docker** | Runs steps inside a specified Docker image — fast startup, most common choice |
| **machine** | Full VM, needed for Docker-in-Docker or other cases requiring a real kernel |
| **macos** | For iOS/macOS builds |

### Caching

- Cache is stored and restored by an explicit **key** — CircleCI supports partial key matching (restoring the most specific available match), which is convenient but means a slightly-wrong key can silently restore a cache that's close-but-not-exact to what's needed
- Caches don't automatically expire on their own logic tied to content correctness — a cache remains valid (and gets restored) for as long as its key matches, regardless of whether the underlying dependencies it represents are actually still accurate

### Contexts & Parallelism

- **Contexts** hold secrets shared across multiple projects/pipelines (vs. project-level environment variables scoped to one project) — access to a context can be restricted by security group
- **Parallelism** (`parallelism: N`) splits a job's test suite across N containers; CircleCI's test splitting can use timing data to balance load, but only if configured to do so — otherwise it defaults to a naive split (e.g., by filename) that can be very uneven

## Best Practices

1. **Pin orb versions to a specific release**, not a loose major-version range, for anything beyond low-stakes utility orbs
2. **Key caches on a hash of the dependency manifest**, and understand partial-match restore behavior before relying on it
3. **Scope contexts restrictively** — use CircleCI's security groups to control which pipelines can access a given context's secrets
4. **Enable timing-based test splitting** explicitly rather than relying on the default naive split when using parallelism
5. **Use workflow fan-out only where jobs are genuinely independent** — needless fan-out burns concurrent job quota without benefit
6. **Set resource classes deliberately** matched to actual job resource needs, not defaulted to the largest available

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Pulling an orb via a loose version range (`orb@5`) in a production pipeline | Pin to a specific release version, update deliberately via a reviewed PR |
| Assuming cache key partial-matching always restores something safe to use | Understand exactly what a partial match restores, and key precisely for anything correctness-sensitive |
| Storing a secret needed by many projects as a duplicated project-level env var | Use a Context, scoped appropriately, to avoid duplication and inconsistency |
| Enabling `parallelism` without timing-based splitting | Explicitly configure timing-based test splitting for balanced load across containers |
| Using the largest resource class "just in case" | Match resource class to actual measured job resource usage |

## Sharp Edges

### SE-1: Unpinned Orb Version Pulling a Breaking Change
- **Severity**: high
- **Situation**: A pipeline referencing an orb by a loose version constraint (e.g., `circleci/node@5`) starts failing (or behaving differently) after the orb publishes a new release within that range, with no corresponding change to the consuming project's own config
- **Cause**: Orbs published under a version range are resolved at pipeline run time to whatever the latest matching release is — a "minor" version bump from the orb's perspective can still be a breaking change from a consumer's perspective if semver discipline isn't perfectly followed upstream
- **Symptoms**:
  - A pipeline that hasn't been touched starts failing, and the diff turns out to be an orb version resolving differently
- **Solution**: Pin orbs to a specific release version for anything beyond trivial/low-risk utility orbs, and review orb changelogs before deliberately bumping a pinned version
- **Details**: → [extended/checklists.md#pipeline-reliability-checklist]

### SE-2: Cache Key Partial-Match Restoring Stale Dependencies
- **Severity**: medium
- **Situation**: A dependency lockfile changes, but the pipeline's cache restore step matches an older, partially-similar cache key and restores outdated dependencies, and the build proceeds without a clear error
- **Cause**: CircleCI's cache restoration supports fallback to a partial key match when an exact match isn't found — this is a convenience feature, but it means "restore succeeded" doesn't guarantee "restored the version I actually wanted"
- **Symptoms**:
  - A dependency bump doesn't take effect in CI even though the lockfile changed
  - Investigation reveals the cache restore step logged a partial-key match, not an exact one
- **Solution**: Key caches on a hash of the lockfile (ensuring exact-match-or-nothing for correctness-sensitive caches), and reserve partial-key fallback matching for genuinely best-effort acceleration caches where slightly-stale data is acceptable

### SE-3: Context Secrets Exposed to Untrusted Fork PR Builds
- **Severity**: critical
- **Situation**: A context holding sensitive credentials is available to a pipeline run triggered by a pull request from an external contributor's fork, and untrusted code in that fork can access the secret during the build
- **Cause**: Context access is controlled by security group configuration, not automatically restricted for fork-originated builds — a context configured without considering the fork-PR threat model can expose secrets to code the project didn't author
- **Symptoms**:
  - A security review finds a context with deploy/API credentials accessible to fork-triggered pipeline runs
- **Solution**: Restrict contexts to security groups appropriate for the trust level of the code that can trigger a pipeline, and avoid exposing deploy/production credentials to any job that could run against unreviewed external contributions

### SE-4: Uneven Test Parallelism From Naive Default Splitting
- **Severity**: medium
- **Situation**: A test suite split across parallel containers via `parallelism: N` shows one container taking dramatically longer than the others, and the job's overall wall-clock time is bottlenecked by that one slow container while others finish early and sit idle
- **Cause**: Without explicit timing-based splitting configured, CircleCI's default split (often by filename or alphabetical grouping) doesn't account for actual test execution time — a few slow test files can all land in the same container
- **Symptoms**:
  - Parallel container timing in the CircleCI UI shows a lopsided distribution
  - Adding more parallelism doesn't proportionally reduce overall job time
- **Solution**: Enable CircleCI's timing-based test splitting (using historical timing data) explicitly rather than relying on the default split strategy, and periodically verify the split remains balanced as the test suite evolves

### SE-5: Workflow Fan-Out Burning Concurrent Job Quota Without Benefit
- **Severity**: medium
- **Situation**: A workflow fans out into many parallel jobs that aren't actually independent (some redundantly repeat setup work, or several could have run sequentially with negligible time cost), consuming the account's concurrent job quota and delaying genuinely time-sensitive jobs from other pipelines
- **Cause**: Fanning out is easy to add without evaluating whether the jobs are truly independent or whether the marginal wall-clock benefit is worth the quota cost, especially on plans with limited concurrency
- **Symptoms**:
  - Concurrent job quota is frequently exhausted, delaying unrelated pipeline runs
  - Some fanned-out jobs individually take only seconds, suggesting the fan-out overhead outweighs the parallelism benefit
- **Solution**: Review fan-out structure for genuine independence and worthwhile parallelism gain, and consolidate jobs that don't meaningfully benefit from running as separate concurrent units

## Recommended Tools

| Category | Tools |
|------|------|
| Local testing | CircleCI CLI (`circleci config validate`, `circleci local execute`) |
| Orb registry | CircleCI Orb Registry |
| Insights | CircleCI Insights (test timing, success rate trends) |
| Security | Context security groups, OIDC token-based cloud auth (avoids long-lived cloud credentials) |

## Pipeline Reliability

**Full checklist**: → [extended/checklists.md#pipeline-reliability-checklist]

## Related Resources

[CircleCI Docs](https://circleci.com/docs/) | [Orbs Registry](https://circleci.com/developer/orbs) | [Optimization Guide](https://circleci.com/docs/optimizations/)

## Related Domains

[[github-actions]] | [[gitlab]] | [[jenkins]] | [[docker]]
