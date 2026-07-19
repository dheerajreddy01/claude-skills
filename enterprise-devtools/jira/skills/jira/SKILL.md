---
schema: "1.0"
name: jira
version: "1.0.0"
description: Jira issue tracking, workflows, JQL queries, and agile board administration
domain: technology
triggers:
  keywords:
    primary: [jira, sprint, backlog, JQL, epic]
    secondary: [board, workflow, story points, kanban, scrum, issue type]
  context_boost: [ticket, issue tracking, project management, agile]
  context_penalty: [code review, deployment, infrastructure]
  priority: high
dependencies:
  software-skills: [project-management]
author: claude-domain-skills
---

# Jira

> Issue tracking that stays fast and trustworthy as the instance grows

## Applicable Scenarios

- Designing or auditing Jira workflows, issue types, and screen schemes
- Writing JQL queries for boards, filters, dashboards, or automation
- Setting up Scrum/Kanban boards and sprint processes
- Diagnosing slow searches, misconfigured permissions, or noisy automation
- Structuring epics/stories/subtasks for a project or migration

## Core Knowledge

### Issue Hierarchy

| Level | Purpose |
|------|------|
| **Initiative/Theme** | Cross-project strategic grouping (higher-tier plans, if enabled) |
| **Epic** | Large body of work spanning multiple sprints |
| **Story/Task** | A unit of deliverable work, estimable and sprint-sized |
| **Subtask** | A breakdown of a Story/Task for individual tracking |
| **Bug** | A defect, tracked like a Story but flagged for triage priority |

### JQL (Jira Query Language)

- Basic form: `project = "ABC" AND status = "In Progress" ORDER BY priority DESC`
- Common clauses: `assignee = currentUser()`, `sprint in openSprints()`, `updated >= -7d`, `status changed to "Done" after -30d`
- Saved filters back boards, dashboards gadgets, and automation triggers — a slow filter slows everything built on it
- `text ~ "keyword"` full-text search is far more expensive than field-based matching (`status =`, `assignee =`) — avoid combining broad text search with a huge date range

### Workflows

- A **workflow** defines statuses and the transitions allowed between them, optionally with validators, conditions, and post-functions
- **Workflow schemes** map issue types to workflows per project — too many bespoke workflows across projects becomes unmaintainable
- **Transition post-functions/automation** (e.g., auto-assign on transition, notify on status change) run synchronously during the transition — a broken automation rule can block users from transitioning issues at all

### Boards

| Type | Best For |
|------|------|
| **Scrum** | Fixed-length sprints, sprint planning/velocity tracking |
| **Kanban** | Continuous flow, WIP limits, no fixed iterations |

Board scope is defined by a saved filter — if the filter is too broad (e.g., matches issues from unrelated projects), the board shows noise; too narrow, and legitimate work goes untracked.

## Best Practices

1. **Keep workflows simple** — fewer statuses and transitions means less confusion and fewer automation edge cases
2. **Standardize issue types and workflows across similar projects** — bespoke-per-project setups don't scale past a handful of teams
3. **Index-friendly JQL** — filter by structured fields (project, status, assignee) before adding text search
4. **Use components/labels consistently** — inconsistent taxonomy makes filtering and reporting unreliable
5. **Review automation rules for loops** — a rule that triggers on the same condition it creates can recurse
6. **Set explicit permission schemes per project** — don't rely on default/global permissions for anything sensitive

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| One giant workflow shared across unrelated projects "for consistency" | Scope workflows to how each team actually works; consolidate only genuinely similar processes |
| Broad `text ~` search combined with `updated >= -365d` | Narrow with structured fields first, keep text search on a tight date range |
| Story points estimated inconsistently across teams | Agree on a reference-story calibration per team; don't compare velocity across teams |
| Automation rule that edits a field it also triggers on | Add a condition to prevent the rule from re-triggering itself |
| Board filter scoped too broadly, showing unrelated project issues | Scope the underlying saved filter tightly to the team's actual project(s)/issue types |

## Sharp Edges

### SE-1: Workflow Scheme Sprawl
- **Severity**: high
- **Situation**: Dozens of near-identical, project-specific workflows accumulate over time, and nobody can safely change one without checking if it's actually shared
- **Cause**: Each new project request ("just like X but with one extra status") gets a workflow clone instead of a parameterized shared workflow
- **Symptoms**:
  - Admins are afraid to touch any workflow because they don't know what else depends on it
  - Simple process changes (e.g., adding a "Blocked" status) require editing 20+ workflows
- **Solution**: Consolidate to a small set of shared workflow schemes across similar project types; use conditions/validators for team-specific variation instead of full workflow clones
- **Details**: → [extended/checklists.md#admin-hygiene-checklist]

### SE-2: Expensive JQL Degrading Instance Performance
- **Severity**: high
- **Situation**: A dashboard gadget or saved filter using broad `text ~` search across a huge date range runs slowly, and it's baked into a board that reloads on every user action
- **Cause**: Full-text search is far more expensive than structured field matching, and unbounded date ranges force scanning far more issues than necessary
- **Symptoms**:
  - Board/dashboard load times climb as the instance grows
  - Search timeouts on filters that "used to be fast"
- **Solution**: Filter by structured fields first (project, status, issue type), narrow date ranges, and avoid `ORDER BY` on infrequently-indexed custom fields

### SE-3: Permission Scheme Gaps Exposing Sensitive Projects
- **Severity**: critical
- **Situation**: A project meant to be restricted (e.g., HR, security incidents, executive planning) is discoverable or editable by users outside the intended group because it inherited a default/global permission scheme
- **Cause**: New projects default to a shared permission scheme unless explicitly assigned a restricted one, and it's easy to skip that step under time pressure
- **Symptoms**:
  - Users outside the intended team can view or comment on sensitive issues
  - A permission audit turns up projects using the default scheme that shouldn't
- **Solution**: Explicitly assign a restricted permission scheme (and issue-level security schemes where needed) to any sensitive project at creation time; audit permission schemes periodically, not just at setup

### SE-4: Automation Rule Loops and Rate Limiting
- **Severity**: medium
- **Situation**: An automation rule that updates a field also has a trigger condition matching that same update, causing it to re-fire repeatedly until Jira's built-in loop protection kills it (or, without protection, spikes API/execution usage)
- **Cause**: Automation rules are event-driven; a rule whose action satisfies its own trigger condition creates an implicit loop
- **Symptoms**:
  - Automation audit log shows the same rule firing many times in rapid succession on one issue
  - Automation execution quota consumed unexpectedly fast
- **Solution**: Add explicit conditions to prevent a rule from matching the state its own action produces, and review the automation audit log after creating any rule that both triggers on and modifies the same field

### SE-5: Sprint Scope Creep Invalidating Velocity Metrics
- **Severity**: medium
- **Situation**: Issues are added to or removed from an active sprint mid-cycle without tracking, and velocity/burndown reports no longer reflect what the team actually committed to
- **Cause**: Jira allows adding/removing issues from an active sprint freely, and by default doesn't distinguish "committed at sprint start" from "added mid-sprint" in the headline velocity number
- **Symptoms**:
  - Velocity swings wildly sprint to sprint with no process explanation
  - Retrospectives argue about whether the team "actually" hit its commitment
- **Solution**: Track scope changes explicitly (Jira's sprint report shows added/removed issues), agree on a team norm for mid-sprint additions, and calculate velocity trends over several sprints rather than reacting to one outlier

## Recommended Tools

| Category | Tools |
|------|------|
| Query/reporting | JQL, Jira dashboards, eazyBI |
| Automation | Jira Automation (native), scripted webhooks |
| Migration | Jira Cloud Migration Assistant, CSV importer |
| API access | Jira REST API, Atlassian Python API |

## Admin Hygiene

**Full checklist**: → [extended/checklists.md#admin-hygiene-checklist]

## Related Resources

[Jira Documentation](https://support.atlassian.com/jira-software-cloud/) | [JQL Reference](https://support.atlassian.com/jira-software-cloud/docs/jql-fields/) | [Atlassian Community](https://community.atlassian.com/)

## Related Domains

[[project-management]] | [[pagerduty]] | [[splunk]]
