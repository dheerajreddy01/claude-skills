---
name: director-pipeline
version: 1.4.0
description: Director → Developer → Tester handoff pipeline — a staged, document-driven workflow that turns a raw request or Jira ticket into HLD/LLD design, an explicit list of which of this library's skills apply, and verified, shipped work instead of one continuous "prompt and pray" pass
triggers: [director pipeline, director dev tester, staged handoff, role pipeline, spec build test, HLD LLD design, high-level design low-level design, jira ticket to plan, skill selection]
keywords: [workflow, methodology, multi-agent, handoff, quality-gate, spec-driven, acceptance-criteria, HLD, LLD, architecture-design, jira, skill-selection, ticket-intake]
author: claude-domain-skills
---

# Director → Dev → Tester Pipeline v1.4.0

> Split any task into three roles with a written handoff between each — Director designs (HLD → LLD) and plans, Dev builds, Tester verifies, and gaps loop back instead of getting silently absorbed

## Quick Start

```bash
/director-pipeline [task description]        # run the full 3-role pipeline on a new task
/director-pipeline ticket [JIRA-ID or pasted ticket content]  # ticket-sourced task: full intake + skill selection
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

**Inputs**: the raw task/request only — no prior code, no prior conversation assumed. This may be a few sentences, or a Jira ticket (see **Ticket Intake** below).

**Ticket Intake** (when the raw input is a Jira ticket or any tracked issue, not free-text): read the *complete* ticket, not just the title —
- **Description** and any **acceptance criteria already written on the ticket** — treat these as raw material to refine, not a finished brief; real tickets are rarely written in checkable Given-When-Then form or with a real HLD/LLD, and producing those is still Director's job
- **Comments**, in chronological order — scope changes and stakeholder clarifications often live here, not in the original description; the most recent decision wins if a comment contradicts the original description, but flag it as an Open Question if it's genuinely unclear which is authoritative
- **Linked issues** (blocks / is blocked by / relates to) — these frequently carry constraints or dependencies the ticket body doesn't restate
- **Labels, components, and fix version** — often the clearest signal for which system/service/tech stack is involved, and directly feed Skill Selection below
- Use the `/jira` skill if the ticket's workflow/field conventions for this specific Jira instance aren't already clear

**Responsibilities**:
- Restate the request in your own words to surface hidden assumptions
- Identify what's explicitly in scope, and just as importantly, what's **out of scope** (non-goals)
- **Skill Selection**: scan the request/ticket for every signal of what it actually touches — named languages/frameworks, cloud providers, databases, infra tools, ticket labels/components, linked service names — and cross-reference against this library's skill catalog (`enterprise-languages/`, `enterprise-cloud/`, `enterprise-devtools/`, and any other relevant category). List every skill that plausibly applies in the brief's **Relevant Skills** section, and name anything the ticket touches that has no matching skill in the library. Err toward including a skill when uncertain — a skill Dev/Tester didn't need costs nothing; a Sharp Edge that goes uncaught because nobody loaded the skill that documents it is expensive.
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

## Relevant Skills
- [skill-name]: [why it applies — e.g. "redis: caching layer for this feature"]
- [skill-name]: [...]
- [Note anything the task touches with no matching skill in the library]

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
- Load and actively apply every skill listed in the brief's **Relevant Skills** section — not as background reading, but as concrete checks against the implementation (e.g., if `redis` is listed, verify an eviction policy and TTLs are actually set, not just that caching works in the happy path)
- Run the **HLD/LLD Completeness Check** (§6) *before* writing any code — catching a design gap before implementation starts is cheaper than discovering it mid-build or waiting for Tester to find it later
- Implement strictly against the brief's scope — do not add unrequested features, do not skip in-scope ones
- Follow the LLD's data model, API contracts, and algorithms as specified rather than re-deriving your own design — if the LLD is genuinely wrong or infeasible (not just a different style preference), that's a deviation to log, not a silent rewrite
- Where the brief is ambiguous *and* has no open question flagged for it, take the most literal reading and document the interpretation (don't silently guess and hide it)
- Where the LLD simply doesn't cover a case you can see is reachable, don't invent behavior for it silently — take your best literal-reading guess to keep moving (same as any other ambiguity), but log it as a **design gap**, not folded quietly into "implementation details"
- Note any deviation from the brief — including the HLD/LLD — and why (e.g., a constraint the brief didn't anticipate)

**Output — `dev-log.md`**:
```markdown
# Dev Log: [task name]

## HLD/LLD Completeness Check
- [ ] Pass — design was complete enough to implement without inventing structure
- [ ] Gaps found before/during implementation — see Design Gaps Found below

## Implementation Summary
[what was built, where]

## Interpretations Made
- AC-2 was ambiguous about ___ → interpreted as ___

## Design Gaps Found
- [Anything the HLD/LLD didn't cover that you had to make a call on anyway —
  e.g. a component the HLD never assigned an owner to, an error path the LLD's
  contract didn't define, a step in the Sequence for Key Flows that didn't hold
  given the real codebase. Log these even if you took your best guess and kept
  moving — this is what lets Tester/Director route it correctly instead of it
  quietly becoming "how the code just happens to work."]

## Deviations From Brief
- [if any, with rationale]

## Known Gaps
- [anything from the brief not yet implemented, and why]
```

### Tester

**Inputs**: `director-brief.md`, `dev-log.md`, and the actual code/output — not the developer's summary of what they think they built.

**Responsibilities**:
- Load every skill listed in the brief's **Relevant Skills** section and verify the implementation against its Sharp Edges specifically — a passing acceptance criterion doesn't mean a known pitfall (e.g., a missing cache eviction policy, an unindexed hot query, a secret baked into a container layer) was actually avoided; the AC often never asked about it
- Run the **HLD/LLD Completeness Check** (§6) first, before grading anything — it determines whether a given finding is a Dev bug or a Director-owned design gap, which changes how it's routed
- Check off each acceptance criterion independently against the real implementation, not against the dev log's claims
- Actively test the edge cases the Director enumerated, not just the happy path
- Assign severity to every finding (blocking / should-fix / nice-to-have)
- Route each finding to the right place (see §4) — use the HLD/LLD Completeness Check's results to tell a genuine implementation bug apart from a design that was incomplete all along

**Output — `test-report.md`**:
```markdown
# Test Report: [task name]

## HLD/LLD Completeness Check
- [ ] Pass — design was complete enough that any finding below is a genuine
      implementation issue, not a design gap
- [ ] Gaps found — see notes; some findings below may route to Director instead of Dev

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

Not every finding goes to the same place — routing it correctly is what keeps the loop from thrashing. Run the **HLD/LLD Completeness Check** (§6) before making these calls: it's the difference between "Dev deviated from a complete LLD" (routes to Dev) and "the LLD never covered this case" (routes to Director), which otherwise looks identical from the symptom alone.

This check isn't only Tester's job. Dev runs the same check *before* implementation starts (§6), which means a design gap can surface — and route to Director — before a single line of code is written, instead of only being discovered after a full implementation and test pass. Tester's check is the second, independent line of defense for whatever Dev's own check didn't catch — the worked example in `extended/examples.md` shows a gap that got past Dev's pre-implementation check but was caught by Tester's, which is the normal case, not a failure of the process.

| Finding type | Symptom | Routes to | Why |
|------|------|------|------|
| **Implementation bug** | Code doesn't do what the brief says | **Dev** | The brief was right, the build was wrong |
| **Requirements gap** | The brief didn't cover a case Dev or Tester found | **Director** | The brief needs amending before Dev touches it again |
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

When spawning those subagents, explicitly invoke every skill named in the brief's **Relevant Skills** section as part of their briefing (not just hand them the brief and hope they infer which domain expertise applies) — e.g., if the brief lists `redis` and `postgresql`, load both skills into the Dev subagent's context alongside `director-brief.md`. This is what turns "the brief mentions caching" into the subagent actually knowing Redis's specific Sharp Edges, rather than reasoning about caching from generic knowledge.

If running everything in one continuous session instead, at minimum: write each stage's document to a real file before moving to the next stage, and explicitly re-read the file (not your memory of writing it) when starting the next role.

---

## 6. Quality Gates

**Before Dev starts:**
- [ ] Relevant Skills section lists every skill this task plausibly touches (languages, cloud platforms, devtools) — cross-checked against ticket labels/components/linked issues if the source was a Jira ticket, not just the description text
- [ ] Task complexity assessed and design depth matched to it (trivial → N/A; anything multi-component or architecturally significant → full HLD + LLD)
- [ ] HLD names every component involved and its responsibility, with no "TBD" left for Dev to resolve
- [ ] LLD specifies data models/API contracts/algorithms concretely enough that Dev isn't inventing structure while also writing code
- [ ] Every acceptance criterion is independently checkable (not "make it good")
- [ ] Out-of-scope items are explicit, not just omitted
- [ ] Every open question is either answered or explicitly deferred to a human

**Dev's HLD/LLD Completeness Check** (run this *before* writing code — catching a gap here is cheaper than discovering it mid-build or leaving it for Tester to find):
- [ ] Every component the LLD requires you to touch is actually named in the HLD's Components list — if you need to touch something the HLD never mentioned, stop and flag it rather than silently expanding scope
- [ ] The LLD's Data Model/Schema section defines every field/entity you'll actually need to persist or transmit — a field you need that the LLD never mentioned is a gap to flag, not a silent addition
- [ ] The LLD's API/Function Contracts cover every input, output, and error case you can actually construct in the real implementation — an error condition the contract doesn't address gets flagged before you invent behavior for it
- [ ] The LLD's Sequence for Key Flows is actually followable against the real codebase/dependencies — a step that doesn't hold is a design gap, not something to quietly route around
- [ ] The HLD's Key Architectural Decisions don't contradict what implementing the LLD as written would actually require — if they do, stop and flag it rather than silently picking one
- [ ] If HLD/LLD was marked "N/A, trivial change," confirm that's still true once you can see the real scope of the change — if it's grown non-trivial, that's worth flagging before continuing to build ad hoc

**Before Tester starts:**
- [ ] Dev log documents every interpretation made on an ambiguous point
- [ ] Dev log's Design Gaps Found section captures anything the HLD/LLD didn't cover, even if Dev's completeness check passed initially and a gap only surfaced during implementation
- [ ] Dev log lists known gaps rather than presenting partial work as complete

**Tester's HLD/LLD Completeness Check** (run this *before* grading findings as bug vs. gap — it's what makes §4's routing decisions defensible instead of a coin flip):
- [ ] Every component the implementation actually touches is named in the HLD's Components list — an undocumented component introduced without a logged deviation is itself a finding
- [ ] The HLD's Data Flow matches how data actually moves through the implementation — trace at least one real request/operation end-to-end, don't just read the diagram and assume
- [ ] Every Key Architectural Decision in the HLD (sync vs. async, fail-open vs. fail-closed, where state lives) is verifiable directly in the code, not just asserted in the dev log
- [ ] Every entity/field the implementation actually persists or transmits is covered by the LLD's Data Model/Schema section — flag any field or table the LLD never mentioned
- [ ] Every API/function contract the implementation exposes matches the LLD's specified inputs, outputs, *and* error cases — check the error paths, not just the happy path
- [ ] Every step in the LLD's Sequence for Key Flows is actually followed, in the specified order, for each key flow that was implemented
- [ ] For every edge case actually reachable in the implementation, the LLD specified expected behavior for it — if the implementation had to invent behavior for something the LLD never covered, that's a **design gap** (routes to Director), not a **bug** (routes to Dev)
- [ ] If HLD/LLD was marked "N/A, trivial change," confirm the change really was trivial — an implementation that grew non-trivial without ever getting a design pass is a finding in its own right

**Before shipping:**
- [ ] Every acceptance criterion is PASS or has an explicit, accepted follow-up
- [ ] No finding is still "in flight" between roles
- [ ] Zero blocking-severity findings remain open

---

## 7. Common Pitfalls

| Pitfall | Why it breaks the pipeline | Fix |
|------|------|------|
| Director writes goals, not checkable criteria ("make the UI nice") | Tester has nothing concrete to verify against | Rewrite every criterion in Given-When-Then form |
| Director restates a Jira ticket's title/description but skips the comments and linked issues | Misses a scope change or constraint that only lives in a comment thread, and the brief is wrong from the start | Read the full ticket per Ticket Intake — comments, links, labels — not just the top-level fields |
| Skill Selection skipped or guessed loosely ("this is probably just backend stuff") | Dev/Tester reason from generic knowledge instead of this library's documented Sharp Edges, and known pitfalls (missing index, cache eviction, secret in a container layer) go uncaught | Explicitly cross-reference ticket signals (labels, components, named tech) against the skill catalog and list every plausible match, erring toward inclusion |
| Director skips HLD/LLD on a multi-component task and jumps straight to acceptance criteria | Dev ends up inventing the architecture mid-implementation, causing rework when Director's mental model turns out different | Design HLD (components, data flow) then LLD (schemas, contracts, algorithms) before writing criteria, for anything beyond a trivial change |
| Director writes a full HLD/LLD for a one-line fix | Wastes a round-trip on ceremony the task doesn't need | Scale design depth to task complexity — "N/A, trivial change" is a valid HLD/LLD section |
| Dev reads the brief once, then works from memory | Implementation drifts from the brief (and its HLD/LLD) over a long task | Re-read the brief at the start of each major sub-task |
| Dev disagrees with the LLD and silently implements something different | Tester verifies against a brief that no longer matches what was built, and the mismatch surfaces late | Log it as a deviation with rationale — let Tester/Director see the divergence, don't hide it |
| Dev hits a case the LLD doesn't cover and quietly invents behavior for it without logging it | The gap becomes "how the code happens to work" instead of a tracked design decision, and Tester has no reason to suspect it's not in the LLD | Run the HLD/LLD Completeness Check before coding; log any gap found in the Design Gaps Found section even if you took your best guess and kept moving |
| Tester grades against the dev log's description instead of the actual code | Bugs the developer didn't notice never get caught | Tester must independently exercise the acceptance criteria, not read a summary |
| Tester routes every finding to Dev by default | Design gaps get "fixed" as one-off patches instead of the LLD being corrected, and the same gap reopens next time a similar case comes up | Run the HLD/LLD Completeness Check first — a finding the design never covered routes to Director, not Dev |
| Findings get fixed ad hoc without updating the brief | The same ambiguity resurfaces in the next round | Route requirements gaps to Director first, always |
| The loop has no defined exit condition | Endless back-and-forth with no ship decision | Define "done" via §6's quality gates, not a feeling |

---

## Extended Resources

- Full document templates → [extended/templates.md](./extended/templates.md)
- Worked example (a small feature end-to-end, plus a ticket-intake/skill-selection example) → [extended/examples.md](./extended/examples.md)
- Pairs well with `/jira` for reading ticket structure/workflow conventions during Ticket Intake, `/tech-spec-gen` for a larger Director stage, and `/consistency-checker` before the Tester stage signs off

---

## Why It Matters

1. **Catches misunderstood requirements before they're built**, not after
2. **Turns a raw ticket into more than a to-do list** — Ticket Intake pulls in comments and linked issues a quick skim would miss, and Skill Selection means Dev/Tester apply this library's documented Sharp Edges instead of reasoning from generic knowledge
3. **Separates architecture from implementation detail** — HLD decides the shape of the solution, LLD decides its concrete mechanics, so Dev isn't inventing structure while also writing code
4. **Catches design gaps at the earliest point they can be caught** — Dev's own completeness check surfaces most of them before a line of code is written; Tester's independent pass catches whatever Dev's own check missed
5. **Forces edge cases into the open** instead of leaving them to whichever role happens to think of them
6. **Gives feedback a specific place to attach** — a named acceptance criterion — instead of a vague "this feels off"
7. **Makes "done" a checklist, not a feeling** — every criterion PASS or an explicit, accepted gap

---

## Version History

| Version | Date | Changes |
|------|------|------|
| 1.4.0 | 2026-07-20 | Added Ticket Intake (reading a Jira ticket's full description, comments, linked issues, and labels — not just the title) and Skill Selection (cross-referencing what the task touches against this library's skill catalog and listing every applicable skill in the brief's new Relevant Skills section). Dev and Tester now load and actively apply those skills' Sharp Edges instead of reasoning from generic knowledge. |
| 1.3.0 | 2026-07-20 | Added the same HLD/LLD Completeness Check to Dev's role, run before implementation starts — shifts design-gap detection earlier (before code is written) instead of relying solely on Tester to catch it after the fact; adds a Design Gaps Found section to dev-log.md |
| 1.2.0 | 2026-07-20 | Added a Tester quality gate — the HLD/LLD Completeness Check — that verifies the design docs are actually complete before findings are graded, so a genuine implementation bug (routes to Dev) can be told apart from a design gap the LLD never covered (routes to Director) |
| 1.1.0 | 2026-07-20 | Director stage now explicitly designs HLD (components, data flow, architectural decisions) then LLD (data models, API contracts, algorithms, sequences) before writing acceptance criteria, scaled to task complexity |
| 1.0.0 | 2026-07-20 | Initial version |
