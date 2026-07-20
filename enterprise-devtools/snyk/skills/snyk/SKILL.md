---
schema: "1.0"
name: snyk
version: "1.0.0"
description: Snyk dependency (SCA), container, and license vulnerability scanning practices
domain: technology
triggers:
  keywords:
    primary: [snyk, dependency vulnerability, SCA, container scanning]
    secondary: [CVE, license compliance, base image, reachability]
  context_boost: [supply chain security, vulnerability management]
  context_penalty: [sonarqube, static analysis]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Snyk (Dependency & Container Vulnerability Scanning)

> A scan is a snapshot — the CVE disclosed the day after you scanned is already in production

## Applicable Scenarios

- Setting up dependency (SCA) scanning in CI for a project
- Reviewing container base image vulnerabilities
- Triaging vulnerability alerts by actual exploitability, not just severity score
- Configuring license compliance scanning for open-source dependencies
- Deciding between point-in-time CI scans and continuous monitoring of deployed artifacts

## Core Knowledge

### Scan Types

| Type | Covers |
|------|------|
| **SCA (Software Composition Analysis)** | Known vulnerabilities in open-source dependencies and their transitive dependencies |
| **Container scanning** | Vulnerabilities in the base image layers and installed OS/language packages inside a container image |
| **SAST** (separate product/mode) | Vulnerabilities in your own first-party code, not dependencies |
| **License scanning** | Flags dependencies with licenses that conflict with the project's compliance policy (e.g., copyleft licenses in a proprietary codebase) |

### Severity vs. Reachability

- Vulnerability severity (Critical/High/Medium/Low, often CVSS-based) reflects the vulnerability's theoretical impact, not whether *your* code actually exercises the vulnerable code path
- **Reachability analysis** (where available) narrows this down to "is the vulnerable function actually called by your code" — a critical CVE in a function your code never calls is a much lower practical priority than a medium CVE in a function on your hot path

### Testing vs. Monitoring

- **`snyk test`** (or equivalent CI scan) is a **point-in-time** check against currently-known vulnerabilities — it doesn't know about a CVE disclosed the day after the scan ran
- **`snyk monitor`** (continuous monitoring) tracks already-deployed dependency snapshots against newly-disclosed vulnerabilities going forward, alerting on new disclosures without requiring a new CI run
- Relying only on CI-time scanning misses vulnerabilities disclosed *after* deployment — which is a large share of real-world exposure windows, since new CVEs are disclosed continuously against already-shipped software

### Fix Suggestions

- Automated upgrade PRs suggest a dependency version bump to remediate a vulnerability, but a major-version bump can include breaking changes — the suggestion is a starting point, not a guaranteed safe merge

## Best Practices

1. **Run both point-in-time CI scans and continuous monitoring** — CI scans catch known issues before merge, monitoring catches newly-disclosed ones in what's already deployed
2. **Prioritize by reachability, not just severity score** — a theoretically critical CVE in unused code is lower priority than a reachable medium one
3. **Test automated fix/upgrade PRs before merging** — a version bump can introduce breaking changes even while fixing the vulnerability
4. **Scan container base images explicitly**, not just application dependencies — base image vulnerabilities are often the larger, neglected surface
5. **Set a license policy and enforce it in CI** — don't discover a license compliance issue after a dependency is already deep in production
6. **Triage and track vulnerability remediation SLAs by severity/reachability**, rather than letting alerts accumulate unaddressed

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Treating every alert at face value regardless of reachability | Prioritize using reachability analysis where available, focus on actually-exercised code paths |
| Auto-merging dependency upgrade PRs without running tests | Run the full test suite against any automated fix PR before merging |
| Scanning only application dependencies, ignoring the container base image | Scan the full container image, including OS-level packages |
| Only scanning at CI time | Enable continuous monitoring on deployed artifacts to catch newly-disclosed CVEs |
| No license policy configured | Define and enforce a license compliance policy in CI before a problematic dependency ships |

## Sharp Edges

### SE-1: Alert Fatigue From Unreachable Vulnerabilities
- **Severity**: high
- **Situation**: A project accumulates hundreds of vulnerability alerts, most in transitive dependencies or code paths that are never actually exercised, and the team stops meaningfully triaging any of them — including the few that are genuinely exploitable
- **Cause**: Raw vulnerability counts don't distinguish "theoretically present in the dependency tree" from "actually reachable from code your application executes" — without reachability analysis, every CVE looks equally urgent on a dashboard
- **Symptoms**:
  - A large, growing backlog of "open" vulnerabilities that nobody actively works through
  - A genuinely exploitable, high-priority vulnerability sits unaddressed in the same undifferentiated list as hundreds of irrelevant ones
- **Solution**: Use reachability analysis (where the tooling supports it) to prioritize triage, and establish a remediation SLA tiered by severity *and* reachability/exploitability, not severity score alone
- **Details**: → [extended/checklists.md#vulnerability-management-checklist]

### SE-2: Automated Fix PRs Merged Without Testing
- **Severity**: high
- **Situation**: An automated dependency upgrade PR (bumping a major version to remediate a vulnerability) is merged on the assumption that "it's just a security fix," and it breaks the build or introduces a runtime regression because the major version bump included breaking API changes
- **Cause**: A version bump sufficient to include a vulnerability fix can also cross a semver major boundary with unrelated breaking changes — the security fix and the breaking change are bundled in the same release
- **Symptoms**:
  - A "routine" security-fix merge causes a production incident or CI failure unrelated to the vulnerability itself
- **Solution**: Run the full test suite (and ideally a staging deploy) against every automated fix PR before merging, just as with any other dependency upgrade, rather than fast-tracking security PRs past normal review

### SE-3: Base Image Vulnerabilities Treated as "Not Our Problem"
- **Severity**: high
- **Situation**: Application dependency scanning is thorough, but the container base image (OS packages, language runtime) carries known vulnerabilities that never get addressed because "we didn't write that code"
- **Cause**: A container image's attack surface includes everything in every layer, not just the application code added on top — an outdated or bloated base image is a real, often larger, vulnerability surface that's easy to overlook if scanning is scoped only to `package.json`/`requirements.txt`-style application dependencies
- **Symptoms**:
  - Container scan results show a large number of OS-level vulnerabilities that predate the application layer entirely
  - Base image hasn't been updated in a long time despite the application code being actively maintained
- **Solution**: Scan the full container image (not just application-level dependency manifests), prefer minimal base images (distroless, slim variants) to reduce surface area, and keep base images updated on a regular cadence independent of application release cycles

### SE-4: License Compliance Violations Discovered Too Late
- **Severity**: medium
- **Situation**: A dependency with a copyleft or otherwise incompatible license (for a proprietary codebase) is discovered already deeply integrated in production, and removing/replacing it is a significant undertaking rather than a quick pre-merge catch
- **Cause**: Without license scanning configured as a CI gate, a dependency's license terms simply aren't checked at the point where blocking it would be cheap (before it's added and built upon)
- **Symptoms**:
  - A legal/compliance review flags a dependency's license as a problem well after the dependency is load-bearing in production
- **Solution**: Configure and enforce a license policy in CI (blocking disallowed licenses at PR time, the same way a vulnerability gate would), so violations are caught before the dependency becomes entrenched

### SE-5: CI-Only Scanning Missing Post-Deployment Disclosures
- **Severity**: high
- **Situation**: A project passes its CI vulnerability scan cleanly, ships to production, and a new CVE is disclosed against one of its dependencies a week later — with only CI-time scanning configured, nothing alerts the team because no new CI run has happened since
- **Cause**: A point-in-time scan only checks against vulnerabilities known *at scan time* — new disclosures are a continuous stream, independent of the application's own commit/deploy cadence
- **Symptoms**:
  - A vulnerability that's been publicly known for weeks or months is discovered in an already-deployed service only during an unrelated audit, not through automated alerting
- **Solution**: Enable continuous monitoring on deployed dependency snapshots (not just CI-time testing), so newly-disclosed vulnerabilities generate alerts against what's actually running in production, independent of the next code change

## Recommended Tools

| Category | Tools |
|------|------|
| SCA/container scanning | Snyk Open Source, Snyk Container |
| Complementary code-level analysis | SonarQube, Semgrep, CodeQL (first-party code, distinct from dependency scanning) |
| Base image minimization | distroless images, Alpine, `docker scout` |
| SBOM generation | Snyk SBOM, Syft |

## Vulnerability Management

**Full checklist**: → [extended/checklists.md#vulnerability-management-checklist]

## Related Resources

[Snyk Docs](https://docs.snyk.io/) | [Snyk Vulnerability Database](https://security.snyk.io/) | [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)

## Related Domains

[[sonarqube]] | [[docker]] | [[github-actions]] | [[kubernetes]]
