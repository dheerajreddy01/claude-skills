# SonarQube — Extended Checklists

## Quality Gate Checklist

- [ ] Quality gate conditions scoped to new/changed code, not overall codebase metrics
- [ ] "New code period" definition set deliberately (baseline version, date, or previous version) and reviewed for staleness
- [ ] Gate conditions cover the categories that actually matter for the project (new bugs, new vulnerabilities, coverage on new code, duplication on new code)
- [ ] Gate failures block merge in CI — not just displayed as a dashboard warning
- [ ] Analysis re-triggered immediately after any quality profile or gate configuration change to confirm it took effect
- [ ] Quality gate bypass/override process requires explicit justification, tracked and reviewed periodically

## Code Health Checklist

- [ ] False positives triaged individually (marked won't-fix with a reason), not resolved by disabling rules project-wide
- [ ] Security Hotspots reviewed on a recurring cadence; review completion tracked as its own metric
- [ ] Coverage numbers reviewed alongside actual test quality (meaningful assertions), not treated as a standalone target
- [ ] Technical debt ratio trend tracked over time, not just a single snapshot
- [ ] Disabled rules documented with rationale, reviewed periodically for re-enablement
- [ ] Quality profile reviewed periodically against current language/framework best practices, not left at defaults indefinitely
