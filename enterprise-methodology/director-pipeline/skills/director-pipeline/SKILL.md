---
name: director-pipeline
version: 1.0.0
description: Director → Developer → Tester handoff pipeline — a staged, document-driven workflow for turning a raw request into verified, shipped work instead of one continuous "prompt and pray" pass
triggers: [director pipeline, director dev tester, staged handoff, role pipeline, spec build test]
keywords: [workflow, methodology, multi-agent, handoff, quality-gate, spec-driven, acceptance-criteria]
author: claude-domain-skills
---

# Director → Dev → Tester Pipeline v1.0.0

> Split any task into three roles with a written handoff between each — Director plans, Dev builds, Tester verifies, and gaps loop back instead of getting silently absorbed

## Quick Start

```bash
/director-pipeline [task description]        # run the full 3-role pipeline on a new task
/director-pipeline brief [task description]  # stop after the Director stage, produce only the brief
/director-pipeline resume [stage]            # resume from an existing brief/dev-log/test-report
/director-pipeline loop                      # re-enter the loop after a test-report flags issues
```

---

## 1. Core Philosophy

### Why Split Into Roles At All

| Single-pass "prompt and pray" | Staged Director → Dev → Tester pipeline |
|------|------|
| Requirements, implementation, and QA blur into one continuous stream of assumptions | Each stage has one job and one written artifact to produce |
| A misunderstood requirement gets built *and* "verified" by the same reasoning that misunderstood it | The Tester reads the brief fresh, not the Dev's justification for what they built |
| Nothing forces a review of edge cases before code is written | Director must define acceptance criteria and edge cases before Dev starts |
| Feedback ("this is wrong") has nowhere specific to attach | Every finding attaches to a named acceptance criterion in the brief |

This generalizes the "requirements handshake" pattern used in staged design pipelines (brief → build → review) from UI-specific work to any software task: **a written document is the only thing that crosses a role boundary** — not conversational context, not assumptions, not "I already explained this earlier."

### The Three Roles

| Role | Job | Reads | Produces |
|------|------|------|------|
| **Director** | Analyze the request, resolve ambiguity, define scope and acceptance criteria | The raw task/prompt only | `director-brief.md` |
| **Dev** | Implement exactly what the brief specifies | `director-brief.md` only | Working code + `dev-log.md` |
| **Tester** | Verify the implementation against the brief's acceptance criteria | `director-brief.md` + `dev-log.md` + the actual code | `test-report.md` |

**Critical rule**: Dev never talks to Director directly, and Tester never grades against what Dev *meant* to build — only against what the brief actually says. If the brief itself is wrong or incomplete, that's a finding that routes back to Director, not something Dev or Tester quietly patches over.

---

## 2. The Pipeline

```
Raw request
    │
    ▼
┌─────────────┐
│  DIRECTOR   │  Analyze → clarify ambiguity → define scope, non-goals, acceptance criteria
└─────────────┘
    │  director-brief.md
    ▼
┌─────────────┐
│    DEV      │  Implement strictly against the brief → document decisions and deviations
└─────────────┘
    │  dev-log.md + code
    ▼
┌─────────────┐
│   TESTER    │  Verify each acceptance criterion → run tests → flag gaps and defects
└─────────────┘
    │  test-report.md
    ▼
 All criteria pass?
    │
   ┌┴────────────────┐
   │ NO               │ YES
   ▼                  ▼
Route the finding    Ship / merge
(see §4 — The Loop)
```

Each arrow is a **checkpoint**: the receiving role should not start until the producing role's document exists and is complete. Skipping a checkpoint — e.g., Dev starting from a verbal description instead of a finished brief — is the single most common way this pipeline degrades back into "prompt and pray."

---

## 3. Role Definitions

### Director

**Inputs**: the raw task/request only — no prior code, no prior conversation assumed.

**Responsibilities**:
- Restate the request in your own words to surface hidden assumptions
- Identify what's explicitly in scope, and just as importantly, what's **out of scope** (non-goals)
- Enumerate edge cases and failure modes the request doesn't mention but implies
- Convert vague goals into specific, checkable acceptance criteria (Given-When-Then form where possible)
- Flag any decision that genuinely needs a human call instead of silently picking one

**Output — `director-brief.md`**:
```markdown
# Director Brief: [task name]

## Restated Request
[one paragraph, in your own words]

## In Scope
- ...

## Out of Scope (Non-Goals)
- ...

## Acceptance Criteria
- [ ] AC-1: Given ___, when ___, then ___
- [ ] AC-2: ...

## Edge Cases & Failure Modes
- ...

## Open Questions for a Human
- ...
```

### Dev

**Inputs**: `director-brief.md` only.

**Responsibilities**:
- Implement strictly against the brief's scope — do not add unrequested features, do not skip in-scope ones
- Where the brief is ambiguous *and* has no open question flagged for it, take the most literal reading and document the interpretation (don't silently guess and hide it)
- Note any deviation from the brief and why (e.g., a constraint the brief didn't anticipate)

**Output — `dev-log.md`**:
```markdown
# Dev Log: [task name]

## Implementation Summary
[what was built, where]

## Interpretations Made
- AC-2 was ambiguous about ___ → interpreted as ___

## Deviations From Brief
- [if any, with rationale]

## Known Gaps
- [anything from the brief not yet implemented, and why]
```

### Tester

**Inputs**: `director-brief.md`, `dev-log.md`, and the actual code/output — not the developer's summary of what they think they built.

**Responsibilities**:
- Check off each acceptance criterion independently against the real implementation, not against the dev log's claims
- Actively test the edge cases the Director enumerated, not just the happy path
- Assign severity to every finding (blocking / should-fix / nice-to-have)
- Route each finding to the right place (see §4)

**Output — `test-report.md`**:
```markdown
# Test Report: [task name]

## Acceptance Criteria Results
- [x] AC-1: PASS
- [ ] AC-2: FAIL — [what happened, severity, repro steps]

## Edge Cases Tested
- ...

## Findings
| Finding | Severity | Routes To |
|------|------|------|
| ... | blocking / should-fix / nice-to-have | Dev / Director |
```

---

## 4. The Loop

Not every finding goes to the same place — routing it correctly is what keeps the loop from thrashing:

| Finding type | Symptom | Routes to | Why |
|------|------|------|------|
| **Implementation bug** | Code doesn't do what the brief says | **Dev** | The brief was right, the build was wrong |
| **Requirements gap** | The brief didn't cover a case the Tester found | **Director** | The brief needs amending before Dev touches it again |
| **Misinterpretation** | Dev's documented interpretation of an ambiguous AC was reasonable but wrong | **Director** first (clarify the AC), then **Dev** | Don't let Dev guess again on the same ambiguity |
| **Scope creep** | Dev built something not in the brief | **Director** (decide: accept into scope, or cut) | Never silently keep unrequested work |

```
test-report.md has findings
        │
        ▼
  Bug in existing scope? ──yes──▶ back to Dev, fix, re-test
        │ no
        ▼
  Brief gap or ambiguity? ──yes──▶ back to Director, amend brief, cascades to Dev, re-test
        │ no
        ▼
  Out-of-scope addition? ──yes──▶ Director decides: keep (amend brief) or cut
```

Stop the loop when: every acceptance criterion is PASS, or Director explicitly accepts remaining gaps as a documented follow-up — never silently dropped.

---

## 5. Running the Roles as Separate Contexts

The pipeline's value comes from each role seeing *only* its own inputs — not from the roles being run by different people. In a single continuous session this is easy to violate accidentally: the same context "remembers" what it decided as Director when it's supposed to be testing as Tester with fresh eyes.

**Recommended**: run Dev and Tester as separate subagent calls, briefed only with the relevant document(s), rather than as continued turns in the conversation that wrote the brief. A subagent whose prompt contains *only* `director-brief.md` genuinely cannot fall back on unstated conversational context — the same way a real developer working from a spec can't read their PM's mind.

If running everything in one continuous session instead, at minimum: write each stage's document to a real file before moving to the next stage, and explicitly re-read the file (not your memory of writing it) when starting the next role.

---

## 6. Quality Gates

**Before Dev starts:**
- [ ] Every acceptance criterion is independently checkable (not "make it good")
- [ ] Out-of-scope items are explicit, not just omitted
- [ ] Every open question is either answered or explicitly deferred to a human

**Before Tester starts:**
- [ ] Dev log documents every interpretation made on an ambiguous point
- [ ] Dev log lists known gaps rather than presenting partial work as complete

**Before shipping:**
- [ ] Every acceptance criterion is PASS or has an explicit, accepted follow-up
- [ ] No finding is still "in flight" between roles
- [ ] Zero blocking-severity findings remain open

---

## 7. Common Pitfalls

| Pitfall | Why it breaks the pipeline | Fix |
|------|------|------|
| Director writes goals, not checkable criteria ("make the UI nice") | Tester has nothing concrete to verify against | Rewrite every criterion in Given-When-Then form |
| Dev reads the brief once, then works from memory | Implementation drifts from the brief over a long task | Re-read the brief at the start of each major sub-task |
| Tester grades against the dev log's description instead of the actual code | Bugs the developer didn't notice never get caught | Tester must independently exercise the acceptance criteria, not read a summary |
| Findings get fixed ad hoc without updating the brief | The same ambiguity resurfaces in the next round | Route requirements gaps to Director first, always |
| The loop has no defined exit condition | Endless back-and-forth with no ship decision | Define "done" via §6's quality gates, not a feeling |

---

## Extended Resources

- Full document templates → [extended/templates.md](./extended/templates.md)
- Worked example (a small feature end-to-end) → [extended/examples.md](./extended/examples.md)
- Pairs well with `/tech-spec-gen` for a larger Director stage, and `/consistency-checker` before the Tester stage signs off

---

## Why It Matters

1. **Catches misunderstood requirements before they're built**, not after
2. **Forces edge cases into the open** instead of leaving them to whichever role happens to think of them
3. **Gives feedback a specific place to attach** — a named acceptance criterion — instead of a vague "this feels off"
4. **Makes "done" a checklist, not a feeling** — every criterion PASS or an explicit, accepted gap

---

## Version History

| Version | Date | Changes |
|------|------|------|
| 1.0.0 | 2026-07-20 | Initial version |
