# Chaos Engineering — Extended Checklists

## Experiment Safety Checklist

- [ ] A specific, measurable steady-state metric and abort threshold defined before the experiment starts
- [ ] Steady-state metric actively monitored/visible throughout the experiment, not checked only after the fact
- [ ] Abort/rollback mechanism tested (ideally validated in non-production) before relying on it in production
- [ ] Blast radius mapped against known shared dependencies/cascade paths, not just the directly targeted component
- [ ] Experiment starts at the smallest viable magnitude and scope, expanding only after success
- [ ] On-call and relevant stakeholders informed in advance of the experiment window
- [ ] Broader monitoring (beyond just the targeted component) active during the experiment to catch unexpected blast-radius escape
- [ ] Clear start/end communication so responders know when to stand down

## Program Maturity Checklist

- [ ] Experiments repeated on a regular cadence, not treated as one-time certifications
- [ ] Experiments re-run after significant changes to the components/paths they validate
- [ ] Findings from each experiment tracked to a concrete follow-up action, not just documented in a report
- [ ] Progression from non-production to production experiments follows a deliberate, staged plan
- [ ] Game days scheduled periodically to validate human response/process, not just system behavior
- [ ] Postmortem-style review conducted after every experiment, successful or not, to capture learnings
