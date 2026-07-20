---
schema: "1.0"
name: linear
version: "1.0.0"
description: Linear cycles, triage workflow, and opinionated issue-tracking practices
domain: technology
triggers:
  keywords:
    primary: [linear, issue tracker, cycle, triage]
    secondary: [roadmap, sub-issue, linear api, workflow state]
  context_boost: [engineering workflow, sprint planning, backlog]
  context_penalty: [jira, notion, confluence]
  priority: high
dependencies:
  software-skills: [project-management]
author: claude-domain-skills
---

# Linear

> Fast because it's opinionated — the moment you fight the defaults, you lose the thing you chose it for

## Applicable Scenarios

- Setting up cycles, workflow states, and triage process for an engineering team
- Structuring projects/roadmap and sub-issue hierarchies
- Diagnosing triage backlog buildup or cycle scope creep
- Wiring GitHub/GitLab integration for automatic issue linking
- Deciding how much to customize Linear's defaults vs. keeping them

## Core Knowledge

### Cycles

- **Cycles** are Linear's time-boxed, sprint-like unit — fixed duration, auto-rolling, with built-in scope/velocity tracking
- Unlike Jira's fully configurable sprints, cycles are intentionally opinionated (start/end automatically, incomplete issues roll to the next cycle by default) — the tool nudges toward small, consistently-shipped increments rather than heavy up-front planning ceremony
- Cycle scope should be set near the start and largely held — the value of time-boxing comes from the commitment, and it erodes if scope is added freely mid-cycle without deliberate tracking

### Triage

- **Triage** is a dedicated inbox-like state for newly created issues (especially ones filed by non-engineers, via integrations, or bug reports) before they're accepted into the backlog/a cycle
- The point of triage is to force a deliberate accept/decline/route decision on every incoming issue — if it's left unprocessed, new work silently piles up outside the team's actual planning view

### Workflow States & Sub-Issues

- Default workflow states (Backlog → Todo → In Progress → In Review → Done, roughly) are intentionally simple — Linear's philosophy leans toward keeping this minimal rather than modeling every possible team-specific nuance as a distinct state
- **Sub-issues** break a larger issue into trackable pieces, and parent issue progress rolls up from sub-issue completion — this only stays accurate if sub-issues are actually kept in sync with real scope, not created once and left stale

### Integrations

- GitHub/GitLab integration links issues to PRs/branches and can auto-transition issue state based on PR status (e.g., merging a linked PR can auto-close the issue) — powerful, but only as accurate as the linking discipline (branch naming, PR description references) that feeds it

## Best Practices

1. **Keep workflow states close to Linear's defaults** — the tool's speed and clarity come partly from not over-modeling process
2. **Process the triage queue regularly** — treat an empty triage inbox as a routine hygiene target, not an occasional cleanup task
3. **Set cycle scope early and hold it** — track and discuss additions rather than silently expanding scope mid-cycle
4. **Keep sub-issue hierarchies genuinely in sync with real scope**, not created once and forgotten
5. **Verify PR/branch linking conventions are actually followed** before relying on auto-transition behavior
6. **Use projects/roadmap for cross-cycle initiatives**, keeping cycles focused on the current time-boxed slice of work

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Heavily customizing workflow states to mirror a previous tool's process | Lean into Linear's simpler default states; resist recreating a more complex system's ceremony |
| Triage queue left unprocessed for days/weeks | Assign clear ownership for triage and process it as a routine, frequent task |
| Adding scope to an active cycle without tracking it | Track additions explicitly and discuss the tradeoff, preserving the point of time-boxing |
| Sub-issues created once and never updated as real scope changes | Keep sub-issue hierarchies in sync with actual current scope |
| Relying on auto-close-on-merge without verifying branch/PR linking conventions are followed | Confirm linking discipline (branch naming, PR references) is consistently followed before trusting automation |

## Sharp Edges

### SE-1: Over-Customizing Workflow States Loses the Tool's Designed-In Simplicity
- **Severity**: medium
- **Situation**: A team migrating from a heavily configured tool (like Jira) recreates a dozen granular workflow states in Linear, and the tool's speed/clarity advantage — a big part of why it was adopted — erodes as issues bounce through many fine-grained states nobody consistently updates
- **Cause**: Linear supports custom workflow states, but its design philosophy and UI speed assume a small number of them; heavy customization fights the tool's actual strength rather than leveraging it
- **Symptoms**:
  - Issues frequently sit in an intermediate custom state with no one updating them promptly
  - Team members report the tool "feels slower" than expected, tracing back to complex state transitions
- **Solution**: Start from Linear's default states and add custom ones sparingly, only for genuinely distinct process needs, and periodically prune states that aren't earning their complexity
- **Details**: → [extended/checklists.md#workflow-hygiene-checklist]

### SE-2: Triage Queue Neglect Letting Incoming Work Go Invisible
- **Severity**: high
- **Situation**: Issues filed via bug reports, integrations, or non-engineering stakeholders accumulate in the Triage inbox unprocessed for weeks, and legitimate incoming work is effectively invisible to planning even though it technically "exists" in the system
- **Cause**: Triage is a deliberate holding area requiring an active accept/decline/route decision — without a routine process to work through it, it just grows rather than surfacing into the team's actual backlog/cycle view
- **Symptoms**:
  - Triage count grows steadily with no corresponding decline in backlog additions
  - Stakeholders report filing issues that "went nowhere" despite the issue technically existing in the system
- **Solution**: Assign clear ownership for processing triage (rotating or dedicated), treat an empty or near-empty triage queue as a standard operational target, and set an SLA for how quickly new issues get triaged

### SE-3: Cycle Scope Creep Undermining Time-Boxing
- **Severity**: high
- **Situation**: Issues get added to an active cycle freely as they come up, and by the cycle's end, completed work looks nothing like what was planned at the start — velocity and planning metrics become meaningless, and the team loses the predictability time-boxing was meant to provide
- **Cause**: Linear doesn't prevent adding issues to an active cycle, and without a team norm against it, the path of least resistance is always "just add it now" rather than deferring to the next cycle
- **Symptoms**:
  - Cycle completion reports show a large gap between initially scoped and finally completed issues
  - Velocity trends are too noisy to inform planning
- **Solution**: Set cycle scope deliberately near the start and treat it as a real commitment, track and discuss additions explicitly (a specific "why is this urgent enough to break scope" conversation) rather than silently expanding, and review scope-change patterns in retrospectives

### SE-4: Sub-Issue Hierarchy Drift Making Progress Tracking Inaccurate
- **Severity**: medium
- **Situation**: A parent issue's sub-issues were broken down accurately at creation time, but as the actual work evolved, new necessary work wasn't added as sub-issues and completed sub-issues don't reflect what actually shipped — the parent's rollup progress percentage misleads anyone using it to gauge status
- **Cause**: Sub-issue completion rollup is a snapshot of whatever sub-issues currently exist — it has no awareness of whether the sub-issue breakdown still matches real scope
- **Symptoms**:
  - A parent issue shows "80% complete" but the actual remaining work is substantial and unaccounted-for in any sub-issue
  - Status updates based on rollup percentage turn out to be misleading in retrospect
- **Solution**: Treat sub-issue breakdown as something to actively maintain, not a one-time decomposition — add/adjust sub-issues as real scope becomes clearer, and cross-check rollup percentages against an actual conversation about remaining work for anything status-report-critical

### SE-5: Integration Webhook Misconfiguration Causing Incorrect Auto-Transitions
- **Severity**: medium
- **Situation**: The GitHub/GitLab integration auto-closes or auto-transitions an issue based on PR merge status, but a mislinked PR (wrong issue reference, or a PR merged for unrelated reasons) causes an issue to close before the actual work is verified complete
- **Cause**: Auto-transition automation trusts the PR-to-issue link (via branch naming or PR description reference) at face value — it has no independent verification that the linked PR actually represents completed work for that specific issue
- **Symptoms**:
  - An issue is marked Done while the actual feature/fix isn't deployed or verified
  - A PR merged for an unrelated cleanup accidentally closes an issue it was loosely linked to
- **Solution**: Establish and follow a consistent branch-naming/PR-linking convention, treat auto-transitions as a convenience rather than a substitute for actual verification on anything customer-facing or high-stakes, and periodically spot-check that auto-closed issues actually correspond to genuinely completed work

## Recommended Tools

| Category | Tools |
|------|------|
| Integration | Linear GitHub/GitLab sync, Linear API/GraphQL, Slack notifications |
| Automation | Linear Workflow automations, Zapier/native webhooks |
| Reporting | Linear Insights, cycle/velocity reports |
| Import | Linear's Jira/Asana/Trello importers |

## Workflow Hygiene

**Full checklist**: → [extended/checklists.md#workflow-hygiene-checklist]

## Related Resources

[Linear Documentation](https://linear.app/docs) | [Linear Method](https://linear.app/method) | [Linear API Docs](https://developers.linear.app/docs)

## Related Domains

[[jira]] | [[project-management]] | [[github-actions]] | [[notion]]
