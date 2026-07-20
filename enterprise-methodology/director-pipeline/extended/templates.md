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

## High-Level Design (HLD)
[Write "N/A, trivial change" if the task doesn't warrant it — see the scaling
guidance below. Otherwise:]

### Components
- [Component/module name]: [its responsibility]
- [New component vs. extension of an existing one — say which, and why]

### Data Flow
- [How a request/data moves through the components for each key flow]

### Key Architectural Decisions
- [Decision]: [what was chosen and why — e.g. synchronous call vs. queued job,
  where state lives, which service owns this data]

## Low-Level Design (LLD)
[Write "N/A, trivial change" if the task doesn't warrant it. Otherwise, ground
every subsection in the HLD above:]

### Data Model / Schema
- [Entity/table/struct name]: [fields, types, constraints, indexes]

### API / Function Contracts
- [Endpoint or function signature]: [inputs] → [outputs]; [error cases and codes]

### Core Logic / Algorithms
- [Any non-obvious logic, calculation, or business rule worth specifying
  precisely before Dev writes it — don't leave this to be reverse-engineered
  from the acceptance criteria alone]

### Sequence for Key Flows
- [Step-by-step order of operations for the flows that matter, especially
  anything involving multiple components or external calls]

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

## Scaling HLD/LLD to Task Size

Not every brief needs a full architecture-and-detail design pass. Match the depth to the actual complexity:

| Task profile | HLD | LLD |
|------|------|------|
| One-line fix, copy change, config tweak | `N/A, trivial change` | `N/A, trivial change` |
| Small addition to an existing, well-understood component (e.g. a new field on an existing endpoint) | One line: "extends [component], no new components" | Light — just the changed data model/contract, not a full rewrite |
| New endpoint or feature within an existing service | One short paragraph on where it fits | Full — data model, contract, core logic for the new piece |
| New service, cross-cutting change, or anything touching 3+ components | Full — every component involved, data flow between them, the architectural decisions and why | Full — for each component's slice of the work |

The failure mode to avoid in both directions: writing ceremony nobody needed for a trivial task, or skipping design entirely for something architecturally significant and leaving Dev to invent the structure mid-implementation.

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
