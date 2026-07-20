# Snyk — Extended Checklists

## Vulnerability Management Checklist

- [ ] Both point-in-time CI scanning (`snyk test`) and continuous monitoring (`snyk monitor`) configured
- [ ] Vulnerability triage prioritized by reachability/exploitability, not severity score alone
- [ ] Remediation SLA tiered by severity and reachability, tracked and reviewed against actual completion
- [ ] Automated fix/upgrade PRs run through the full test suite before merging, never fast-tracked past normal review
- [ ] Container base images scanned in addition to application-level dependencies
- [ ] Base images kept current on a regular cadence, independent of application release cycles
- [ ] License policy configured and enforced as a CI gate, not just informational

## Scanning Coverage Checklist

- [ ] All manifests/lockfiles across the repo (including monorepo sub-projects) are included in scan scope
- [ ] Transitive dependency vulnerabilities reviewed, not just direct dependencies
- [ ] SBOM (Software Bill of Materials) generated and retained for audit/compliance purposes
- [ ] New disclosures against already-deployed dependency snapshots trigger alerts, not just new CI runs
- [ ] Scan results integrated into the same PR review flow as other CI checks, not siloed in a separate dashboard nobody checks
- [ ] Vulnerability dashboard reviewed on a recurring cadence, not only reactively after an incident
