---
schema: "1.0"
name: sonarqube
version: "1.0.0"
description: SonarQube static analysis, quality gates, and code health practices
domain: technology
triggers:
  keywords:
    primary: [sonarqube, static analysis, code quality, quality gate]
    secondary: [code smell, technical debt, coverage, security hotspot]
  context_boost: [code review, ci pipeline, code health]
  context_penalty: [snyk, dependency vulnerability]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# SonarQube (Static Analysis / Code Quality)

> A quality gate that blocks on legacy debt trains everyone to ignore it

## Applicable Scenarios

- Setting up quality gates to block merges on defined thresholds
- Distinguishing and triaging bugs, vulnerabilities, code smells, and security hotspots
- Deciding between new-code and overall-code analysis scope
- Reviewing false positives and rule suppressions
- Tracking technical debt trends over time

## Core Knowledge

### Issue Categories

| Category | Meaning |
|------|------|
| **Bug** | A coding error likely to cause incorrect behavior at runtime |
| **Vulnerability** | A coding pattern exploitable as a security weakness |
| **Code Smell** | Maintainability issue — not necessarily wrong, but harder to work with over time |
| **Security Hotspot** | Code that needs human review to determine if it's actually a vulnerability in context — doesn't fail the build by default |

Security Hotspots are deliberately **not** auto-failing — they require a human "reviewed: safe" or "reviewed: fixed" decision, which means an unreviewed backlog of hotspots provides zero actual security assurance despite showing up as "detected."

### Quality Gates

- A **quality gate** is a set of pass/fail conditions (e.g., "no new bugs," "coverage on new code ≥ 80%," "no new critical vulnerabilities") evaluated on each analysis
- The default recommended gate (**Sonar way**) is scoped to **new code** — conditions apply to code changed since a defined baseline, not the entire codebase's historical state
- Applying gate conditions to overall/absolute code metrics (rather than new-code-only) means any pre-existing legacy debt permanently blocks every future PR touching that area, regardless of whether the PR made things better or worse

### New Code Definition

- The "new code period" (a number of days, a specific version, or since a baseline analysis) determines what counts as "new" for gate evaluation and reporting
- Getting this wrong — e.g., leaving it at a stale baseline — either makes everything "new" (effectively reverting to overall-code gating) or nothing "new" (gate stops being meaningful)

### Technical Debt Ratio

- SonarQube estimates a "cost to fix" for maintainability issues and expresses it as a ratio against estimated development cost — useful as a trend indicator, not a precise, actionable number in isolation

## Best Practices

1. **Scope quality gates to new/changed code**, not the whole codebase — this makes the gate actionable without being blocked by inherited legacy debt
2. **Triage false positives explicitly** (mark as "won't fix" with a reason) rather than blanket-disabling rules across the whole project
3. **Actively review Security Hotspots** on a schedule — they don't fail builds automatically, so someone has to look
4. **Treat coverage percentage as a signal, not a target to game** — trivial tests written purely to hit a number don't improve real quality
5. **Re-run analysis after any quality profile/rule configuration change** — stale analysis results don't reflect the new rules
6. **Track technical debt trend over time**, not just a single snapshot — a stable or shrinking trend matters more than an absolute number

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Quality gate scoped to overall code, blocking every PR on legacy debt | Scope the gate to new/changed code only |
| False positives blanket-suppressed by disabling the rule project-wide | Triage individually — mark specific instances as won't-fix with a documented reason |
| Coverage targets hit with trivial, assertion-free tests | Treat coverage as a signal alongside actual test quality review |
| Security Hotspots left permanently unreviewed | Schedule regular hotspot review; track review completion as its own metric |
| Quality profile changed without re-running analysis | Re-trigger analysis after any rule/profile configuration change |

## Sharp Edges

### SE-1: Overall-Code Quality Gates Blocking on Legacy Debt Forever
- **Severity**: high
- **Situation**: A quality gate condition checks overall/absolute code metrics (e.g., "overall code coverage ≥ 80%"), and a codebase with pre-existing legacy debt below that threshold has every single PR blocked, regardless of whether the PR itself is an improvement
- **Cause**: Overall-code conditions don't distinguish "code this PR touched" from "code that's always been below standard" — a team can't realistically fix all legacy debt before merging an unrelated feature
- **Symptoms**:
  - Every PR requires an override/bypass to merge, defeating the purpose of having a gate
  - Developers start ignoring or routinely overriding the quality gate because it's permanently red
- **Solution**: Scope quality gate conditions to new/changed code (the default "Sonar way" gate does this), so the gate reflects whether a given change made things better or worse, not the codebase's entire history
- **Details**: → [extended/checklists.md#quality-gate-checklist]

### SE-2: Blanket Rule Suppression Instead of Triage
- **Severity**: medium
- **Situation**: A noisy rule with some false positives gets disabled project-wide to stop the noise, and in the process, every legitimate instance of that rule's real issue type also stops being detected going forward
- **Cause**: Disabling a rule is faster than triaging individual false positives, but it discards the rule's true-positive detection along with the false positives
- **Symptoms**:
  - A class of real bug/smell that the disabled rule would have caught starts appearing in the codebase again, undetected
  - Quality profile shows a substantial number of disabled rules with no documented rationale
- **Solution**: Triage false positives individually (mark specific issue instances as "won't fix" or "false positive" with a reason), keeping the rule active for genuinely new violations, and only disable a rule project-wide when it's genuinely not applicable to the codebase's context

### SE-3: Gaming Coverage Metrics Without Improving Quality
- **Severity**: medium
- **Situation**: A team under pressure to hit a coverage threshold writes tests that execute code paths without asserting anything meaningful, satisfying the metric while providing no actual regression protection
- **Cause**: Code coverage measures *execution*, not *verification* — a test that runs a function without checking its output counts fully toward coverage
- **Symptoms**:
  - Coverage percentage looks healthy, but bugs still slip through that "covered" code paths should have caught
  - Test suite grows in size without a corresponding drop in production defects
- **Solution**: Review test quality alongside coverage numbers (are there meaningful assertions, not just execution), and treat coverage as one signal among several — combined with code review — not a standalone quality target

### SE-4: Stale Analysis Results After Configuration Changes
- **Severity**: medium
- **Situation**: A quality profile's rule set is updated (new rules enabled, thresholds changed), but the dashboard and gate results still reflect the previous analysis run, leading to confusion about whether the new rules are actually being enforced
- **Cause**: SonarQube analysis is triggered by a scan (typically part of the CI pipeline); configuration changes don't retroactively re-analyze code until the next scan runs
- **Symptoms**:
  - A newly-enabled rule doesn't show any violations even in code that should clearly trigger it, until the next CI run
  - Team confusion about whether a policy change actually took effect
- **Solution**: Trigger a fresh analysis run immediately after any quality profile or gate configuration change to confirm it's reflected, rather than assuming the dashboard is live-updating

### SE-5: Security Hotspots Left Permanently Unreviewed
- **Severity**: high
- **Situation**: SonarQube flags a substantial number of Security Hotspots over time, none of which fail the build (by design), and because nothing forces a review, they accumulate indefinitely with zero actual security assessment ever performed
- **Cause**: Hotspots require human judgment to classify as safe-in-context or genuinely exploitable — SonarQube surfaces them but doesn't (and can't) auto-resolve them, and without a review process this becomes a permanent backlog of unknowns
- **Symptoms**:
  - A large "to review" Security Hotspot count that only grows, never shrinks
  - A security audit finds hotspots that were flagged months or years ago with no review status
- **Solution**: Establish a recurring review cadence (e.g., as part of sprint work or a dedicated security review rotation) for new hotspots, track hotspot review completion as an explicit metric distinct from the quality gate, and treat an unreviewed hotspot backlog as a real, growing risk rather than a passive dashboard number

## Recommended Tools

| Category | Tools |
|------|------|
| CI integration | SonarScanner (per-language), SonarQube GitHub/GitLab integration |
| Complementary security scanning | Snyk (dependency/container vulnerabilities, distinct from SonarQube's code-level analysis) |
| Alternatives | CodeQL, Semgrep |

## Quality Gate Design

**Full checklist**: → [extended/checklists.md#quality-gate-checklist]

## Related Resources

[SonarQube Docs](https://docs.sonarsource.com/sonarqube/latest/) | [Clean as You Code](https://docs.sonarsource.com/sonarqube/latest/user-guide/clean-as-you-code/) | [SonarSource Rules Explorer](https://rules.sonarsource.com/)

## Related Domains

[[snyk]] | [[github-actions]] | [[gitlab]] | [[java]]
