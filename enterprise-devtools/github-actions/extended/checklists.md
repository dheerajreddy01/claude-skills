# GitHub Actions — Extended Checklists

## Security Checklist

- [ ] No workflow combines `pull_request_target` with checking out and executing the PR's own untrusted code
- [ ] All third-party Actions pinned to a full commit SHA, not a mutable tag or branch
- [ ] Dependabot (or equivalent) configured for the `github-actions` ecosystem to get reviewable pin-update PRs
- [ ] `permissions:` block set explicitly at workflow or job level, scoped to least privilege
- [ ] Secrets never echoed to logs, even for debugging; masked-value assumptions not relied upon as the only protection
- [ ] Production-deploy workflows gated by environment protection rules (required reviewers)
- [ ] Self-hosted runners on public repos are ephemeral/autoscaling, not persistent, to limit blast radius of untrusted job execution
- [ ] `zizmor` or an equivalent workflow security linter run against `.github/workflows/`

## Reliability & Cost Checklist

- [ ] `concurrency:` groups configured on deploy workflows to cancel superseded runs
- [ ] Cache keys derived from a hash of the lockfile/dependency manifest, not a loosely-scoped key
- [ ] Matrix build dimensions reviewed for actual necessity — unnecessary combinations multiply runner-minute cost
- [ ] Workflow run history reviewed periodically for consistently flaky jobs and root-caused, not just re-run
- [ ] Long-running or expensive jobs have appropriate `timeout-minutes` set to fail fast instead of hanging
