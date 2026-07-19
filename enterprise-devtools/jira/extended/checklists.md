# Jira — Extended Checklists

## Admin Hygiene Checklist

- [ ] Workflow schemes consolidated — no unexplained per-project clone of an otherwise-identical workflow
- [ ] Permission schemes explicitly assigned per project; no sensitive project left on a default/global scheme
- [ ] Issue-level security schemes applied where a project needs sub-project-level restriction
- [ ] Automation rules reviewed for self-triggering loops (action satisfies the rule's own trigger condition)
- [ ] Automation audit log spot-checked after adding any new rule that both triggers on and edits the same field
- [ ] Saved filters backing boards/dashboards use structured field matching before any `text ~` search
- [ ] Custom field sprawl reviewed — duplicate or unused custom fields removed periodically
- [ ] Board filters scoped tightly to the intended project(s)/issue types, not overly broad JQL
- [ ] Components/labels standardized with a documented taxonomy, not ad hoc per reporter
- [ ] Sprint scope changes (added/removed mid-sprint) tracked and reviewed in retrospectives, not just the final velocity number
