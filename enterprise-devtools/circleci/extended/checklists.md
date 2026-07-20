# CircleCI — Extended Checklists

## Pipeline Reliability Checklist

- [ ] Orbs pinned to a specific release version, not a loose major-version range, for anything beyond trivial utility orbs
- [ ] Cache keys derived from a hash of the lockfile/dependency manifest for correctness-sensitive caches
- [ ] Partial-match cache restore behavior understood and reserved for genuinely best-effort acceleration, not correctness-critical caching
- [ ] Timing-based test splitting explicitly enabled for any job using `parallelism`
- [ ] Test split balance reviewed periodically as the test suite evolves
- [ ] Resource classes matched to actual measured job resource usage, not defaulted to the largest available
- [ ] Workflow fan-out reviewed for genuine job independence and worthwhile parallelism gain
- [ ] Concurrent job quota usage monitored to catch unnecessary fan-out consuming shared capacity

## Security Checklist

- [ ] Contexts scoped to security groups appropriate for the trust level of code that can trigger a pipeline
- [ ] Deploy/production credentials never exposed to jobs that could run against unreviewed fork PRs
- [ ] OIDC-based cloud authentication used where supported, instead of long-lived static cloud credentials
- [ ] Secrets never echoed or logged, even in transformed form, within any step
- [ ] SSH keys added to jobs scoped to only the repositories/hosts actually needed
