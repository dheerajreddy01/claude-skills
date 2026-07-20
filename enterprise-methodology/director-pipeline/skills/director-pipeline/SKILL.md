---
name: director-pipeline
version: 1.1.0
description: Director → Developer → Tester handoff pipeline — a staged, document-driven workflow that carries a request through HLD/LLD design before build, and through verified, shipped work instead of one continuous "prompt and pray" pass
triggers: [director pipeline, director dev tester, staged handoff, role pipeline, spec build test, HLD LLD design, high-level design low-level design]
keywords: [workflow, methodology, multi-agent, handoff, quality-gate, spec-driven, acceptance-criteria, HLD, LLD, architecture-design]
author: claude-domain-skills
---

# Director → Dev → Tester Pipeline v1.1.0

> Split any task into three roles with a written handoff between each — Director designs (HLD → LLD) and plans, Dev builds, Tester verifies, and gaps loop back instead of getting silently absorbed

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
| **Director** | Analyze the request, resolve ambiguity, design the solution (HLD → LLD), define scope and acceptance criteria | The raw task/prompt only | `director-brief.md` (including HLD + LLD) |
| **Dev** | Implement exactly what the brief's HLD/LLD and acceptance criteria specify | `director-brief.md` only | Working code + `dev-log.md` |
| **Tester** | Verify the implementation against the brief's acceptance criteria | `director-brief.md` + `dev-log.md` + the actual code | `test-report.md` |

**Critical rule**: Dev never talks to Director directly, and Tester never grades against what Dev *meant* to build — only against what the brief actually says. If the brief itself is wrong or incomplete, that's a finding that routes back to Director, not something Dev or Tester quietly patches over.

---

## 2. The Pipeline

```
Raw request
    │
    ▼
┌─────────────┐
│  DIRECTOR   │  Analyze → clarify ambiguity → HLD (components) → LLD (details) → scope, non-goals, acceptance criteria
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
- Design the solution in two passes, coarse to fine, **before** writing acceptance criteria:
  - **HLD (High-Level Design)** — the architecture pass: what components/modules exist, what each is responsible for, how data flows between them, and the key architectural decisions (sync vs. async, where state lives, what's a new component vs. an extension of an existing one)
  - **LLD (Low-Level Design)** — the implementation-detail pass, grounded in the HLD: data models/schemas, API/function contracts (inputs, outputs, error cases), core algorithms or business logic, and the sequence of operations for each key flow
- Enumerate edge cases and failure modes the request doesn't mention but implies — the LLD pass is what usually surfaces these, since they show up as unhandled branches in a data model or flow
- Convert the design into specific, checkable acceptance criteria (Given-When-Then form where possible) — the HLD/LLD should make criteria easy to write precisely, not vaguely
- Flag any decision that genuinely needs a human call instead of silently picking one

**Scale the design depth to the task.** A one-line bug fix or copy change needs neither HLD nor LLD — write "N/A, trivial change" and move on. A new endpoint on an existing service usually needs a light LLD only (the HLD is just "extends the existing service," not worth its own section). A new service, a cross-cutting change, or anything touching multiple components warrants both passes in full — that's where skipping them causes Dev to invent architecture mid-implementation instead of Director having decided it upfront.

**Output — `director-brief.md`**:
```markdown
# Director Brief: [task name]

## Restated Request
[one paragraph, in your own words]

## In Scope
- ...

## Out of Scope (Non-Goals)
- ...

## High-Level Design (HLD)
[N/A for trivial changes — otherwise:]
### Components
- [Component/module name]: [its responsibility]

### Data Flow
- [How data/requests move between components for the key flows]

### Key Architectural Decisions
- [Decision]: [what was chosen and why, e.g. sync call vs. queued job]

## Low-Level Design (LLD)
[N/A for trivial changes — otherwise:]
### Data Model / Schema
- [Entity/table/struct]: [fields, types, constraints]

### API / Function Contracts
- [Endpoint or function signature]: [inputs] → [outputs], [error cases]

### Core Logic / Algorithms
- [Any non-obvious logic worth specifying before Dev writes it]

### Sequence for Key Flows
- [Step-by-step order of operations for the flows that matter]

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
- Follow the LLD's data model, API contracts, and algorithms as specified rather than re-deriving your own design — if the LLD is genuinely wrong or infeasible (not just a different style preference), that's a deviation to log, not a silent rewrite
- Where the brief is ambiguous *and* has no open question flagged for it, take the most literal reading and document the interpretation (don't silently guess and hide it)
- Note any deviation from the brief — including the HLD/LLD — and why (e.g., a constraint the brief didn't anticipate)

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
- [ ] Task complexity assessed and design depth matched to it (trivial → N/A; anything multi-component or architecturally significant → full HLD + LLD)
- [ ] HLD names every component involved and its responsibility, with no "TBD" left for Dev to resolve
- [ ] LLD specifies data models/API contracts/algorithms concretely enough that Dev isn't inventing structure while also writing code
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
| Director skips HLD/LLD on a multi-component task and jumps straight to acceptance criteria | Dev ends up inventing the architecture mid-implementation, causing rework when Director's mental model turns out different | Design HLD (components, data flow) then LLD (schemas, contracts, algorithms) before writing criteria, for anything beyond a trivial change |
| Director writes a full HLD/LLD for a one-line fix | Wastes a round-trip on ceremony the task doesn't need | Scale design depth to task complexity — "N/A, trivial change" is a valid HLD/LLD section |
| Dev reads the brief once, then works from memory | Implementation drifts from the brief (and its HLD/LLD) over a long task | Re-read the brief at the start of each major sub-task |
| Dev disagrees with the LLD and silently implements something different | Tester verifies against a brief that no longer matches what was built, and the mismatch surfaces late | Log it as a deviation with rationale — let Tester/Director see the divergence, don't hide it |
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
2. **Separates architecture from implementation detail** — HLD decides the shape of the solution, LLD decides its concrete mechanics, so Dev isn't inventing structure while also writing code
3. **Forces edge cases into the open** instead of leaving them to whichever role happens to think of them
4. **Gives feedback a specific place to attach** — a named acceptance criterion — instead of a vague "this feels off"
5. **Makes "done" a checklist, not a feeling** — every criterion PASS or an explicit, accepted gap

---

## Version History

| Version | Date | Changes |
|------|------|------|
| 1.1.0 | 2026-07-20 | Director stage now explicitly designs HLD (components, data flow, architectural decisions) then LLD (data models, API contracts, algorithms, sequences) before writing acceptance criteria, scaled to task complexity |
| 1.0.0 | 2026-07-20 | Initial version |
