# Director Pipeline — Full Templates

## `director-brief.md` Template

```markdown
# Director Brief: [task name]

## Restated Request
[Rephrase the raw request in your own words. If you can't restate it precisely,
that's a sign it needs an Open Question, not a guess.]

## In Scope
- [Concrete, bulleted list of what will be built]

## Out of Scope (Non-Goals)
- [Just as important as in-scope — say explicitly what this task will NOT do]

## Acceptance Criteria
- [ ] AC-1: Given [context], when [action], then [expected result]
- [ ] AC-2: Given [context], when [action], then [expected result]
- [ ] AC-3: ...

## Edge Cases & Failure Modes
- [What happens on empty input / concurrent access / network failure / etc.,
  even if the raw request never mentioned it]

## Constraints
- [Technical, time, or compatibility constraints that shape the solution space]

## Open Questions for a Human
- [Anything genuinely ambiguous that a human should resolve before Dev starts.
  If this section is empty, double-check you didn't just silently guess instead.]
```

## `dev-log.md` Template

```markdown
# Dev Log: [task name]

## Implementation Summary
[What was built, where (files/modules touched), and the overall approach]

## Mapping to Acceptance Criteria
- AC-1: [implemented in file/function X]
- AC-2: [implemented in file/function Y]

## Interpretations Made
- [AC-N] was ambiguous about [what] → interpreted as [decision], because [reasoning]

## Deviations From Brief
- [Anything built differently than specified, and why — a constraint the brief
  didn't anticipate, a technical impossibility, etc. Never silent.]

## Known Gaps
- [Anything from the brief not yet implemented, and why — don't present partial
  work as complete]

## Notes for Tester
- [Anything that would help testing go faster — how to run it, what data to use,
  known-tricky areas to look at closely]
```

## `test-report.md` Template

```markdown
# Test Report: [task name]

## Acceptance Criteria Results
- [x] AC-1: PASS
- [ ] AC-2: FAIL — [what happened instead, severity, repro steps]
- [x] AC-3: PASS

## Edge Cases Tested
- [Edge case from the brief] → [result]

## Findings
| # | Finding | Severity | Routes To | Notes |
|---|---------|----------|-----------|-------|
| 1 | ... | blocking | Dev | Repro: ... |
| 2 | ... | should-fix | Director | Brief didn't cover this case |
| 3 | ... | nice-to-have | Dev | Not blocking, optional polish |

## Overall Verdict
- [ ] Ready to ship (all criteria PASS, no blocking findings)
- [ ] Blocked — see findings above
- [ ] Ready to ship with accepted follow-ups: [list, with Director sign-off]
```

## Severity Definitions

| Severity | Meaning | Blocks shipping? |
|------|------|------|
| **blocking** | An acceptance criterion fails, or a defect breaks core functionality | Yes |
| **should-fix** | A real gap or bug, but doesn't break the core ask | No, but must be tracked as an accepted follow-up if shipping anyway |
| **nice-to-have** | Polish or an enhancement beyond what was asked | Never — route to Director only if it's worth expanding scope |
