# GitLab — Extended Checklists

## Pipeline Reliability Checklist

- [ ] All jobs use `rules` exclusively — no mixing with legacy `only`/`except`
- [ ] Required runner tags verified to exist on at least one available, healthy runner
- [ ] Pipeline-level or job-level timeouts set so a permanently-pending or hung job doesn't block indefinitely
- [ ] Cache keys derived from a hash of the lockfile/dependency manifest, not a loosely-scoped key like branch name
- [ ] `needs`-based DAG dependencies reviewed to confirm every implicit ordering assumption is explicitly declared
- [ ] Merge request pipelines tested against fork-originated contribution scenarios, not just internal branches
- [ ] Artifact retention/scoping reviewed so large or sensitive artifacts aren't retained longer than needed

## Security & Access Checklist

- [ ] Sensitive CI/CD variables marked "Protected" and "Masked"
- [ ] Protected variable access explicitly verified against fork-originated MR pipelines — no assumption of parity with branch pipelines
- [ ] Secret Detection / Dependency Scanning enabled where available in the GitLab tier in use
- [ ] Container registry access scoped appropriately per project/group
- [ ] Runner registration tokens rotated and not shared broadly; specific runners scoped to the projects that need them
- [ ] Audit events reviewed periodically for unexpected changes to protected branch/variable configuration
