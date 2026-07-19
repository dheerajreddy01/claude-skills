---
schema: "1.0"
name: pagerduty
version: "1.0.0"
description: PagerDuty on-call scheduling, escalation policies, event orchestration, and incident response practices
domain: technology
triggers:
  keywords:
    primary: [pagerduty, on-call, escalation policy, incident response]
    secondary: [schedule, urgency, event orchestration, postmortem, MTTR]
  context_boost: [alerting, incident management, SRE, ops]
  context_penalty: [jira, splunk, datadog]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# PagerDuty

> Escalation policies and schedules that actually get the right human paged

## Applicable Scenarios

- Designing on-call schedules and escalation policies
- Configuring event routing/orchestration from monitoring tools into PagerDuty
- Diagnosing alert fatigue or missed pages
- Setting incident urgency and notification rules
- Structuring postmortem/incident review processes

## Core Knowledge

### Core Objects

| Object | Role |
|------|------|
| **Service** | Represents a monitored component; receives events via integration (API, email, or a monitoring tool's native integration) |
| **Escalation Policy** | Ordered list of who gets notified, and after how long, if the previous level doesn't acknowledge |
| **Schedule** | Defines who is on-call at any given time, layered (e.g., primary + secondary + overrides) |
| **Incident** | A triggered alert routed through a service's escalation policy; has states: triggered → acknowledged → resolved |
| **Event Orchestration** | Rules that route, suppress, or enrich incoming events before they become (or don't become) an incident |

### Escalation Policies

- Each escalation **level** has a notification target (a user, or a schedule) and a timeout before escalating to the next level
- If nobody acknowledges within the timeout, PagerDuty escalates automatically — a policy with only one level (no fallback) means an unavailable on-call person results in a silently dropped incident
- Multiple services can share one escalation policy, or each service can have its own — sharing makes sense when the same team owns everything routed to those services

### Schedules

- **Layers** stack (e.g., a primary rotation layer plus a secondary/backup layer) — the schedule's *effective* on-call is the union/override resolution across layers, not just the primary layer
- **Overrides** temporarily replace who's on-call without altering the underlying rotation — used for one-off swaps (PTO, sick day)
- Time zone handling matters for distributed teams — a schedule built in one time zone can produce unexpected on-call windows when viewed by someone in another

### Urgency & Notification Rules

- **Urgency** (high/low) on an incident determines whether it triggers push/SMS/phone notifications (high) or a quieter channel (low) — services can set static urgency or support urgency changes based on time-of-day (e.g., low urgency outside business hours)
- Each user configures their own **notification rules** (how many minutes after a page to escalate to the next contact method) — a user with only email notification rules effectively won't be paged in any urgent sense

### Event Orchestration

- Global and service-level orchestration rules can suppress, route, deduplicate, or enrich events *before* they create/update an incident
- Deduplication (`dedup_key`) prevents the same underlying condition from creating a new incident every time the monitoring tool re-sends it — critical for noisy upstream integrations

## Best Practices

1. **Always configure at least two escalation levels** — a single-level policy has no fallback if the primary on-call is unreachable
2. **Layer schedules with a secondary/backup rotation**, not just a lone primary
3. **Use event orchestration deduplication** for any noisy upstream integration, so retriggering doesn't spam a new incident per alert
4. **Set incident urgency deliberately** — not everything routed to PagerDuty deserves a 2am phone call
5. **Verify every on-call user has a real urgent notification rule** (push/SMS/phone), not just email
6. **Run postmortems on every high-severity incident**, focused on process/system gaps, not individual blame

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Single-level escalation policy with no fallback | Add at least one additional level (secondary on-call, or team lead) with a reasonable timeout |
| Every alert routed at high urgency regardless of actual impact | Set/derive urgency from real severity; route low-impact alerts at low urgency or to a non-paging channel |
| No deduplication on a flapping upstream check | Configure `dedup_key`/orchestration rules to collapse repeated triggers into one incident |
| Schedule built without accounting for on-call members' time zones | Build and review schedules in each affected member's local time, not just the admin's |
| Postmortems that end at "the on-call engineer should have known" | Focus on system/process fixes — better alerting, runbooks, automation — not individual blame |

## Sharp Edges

### SE-1: Escalation Policy Gaps Leaving Nobody Paged
- **Severity**: critical
- **Situation**: An incident triggers, the primary on-call doesn't acknowledge in time, and the escalation policy has nowhere else to go — the incident sits unacknowledged indefinitely
- **Cause**: A single-level escalation policy (or a policy whose only fallback is another unavailable person) has no genuine redundancy
- **Symptoms**:
  - Postmortem timeline shows a long gap between "incident triggered" and "someone actually responded"
  - Escalation policy audit shows levels pointing to the same person/schedule with no independent backup
- **Solution**: Require at least two independent escalation levels (e.g., primary on-call → secondary on-call → team lead), and periodically audit that each level actually resolves to a different, reachable target
- **Details**: → [extended/checklists.md#on-call-reliability-checklist]

### SE-2: Alert Fatigue From Unfiltered, Duplicate Alerts
- **Severity**: high
- **Situation**: A flapping health check or a noisy monitoring integration creates dozens of incidents for the same underlying condition, and the on-call engineer starts acknowledging without reading, eventually missing a genuinely different incident buried in the noise
- **Cause**: Without deduplication/suppression rules, every re-triggered event from an upstream source becomes a fresh incident (or a fresh notification) rather than being collapsed into the existing one
- **Symptoms**:
  - Incident history shows many near-identical, short-lived incidents from the same service in a short window
  - On-call feedback describes a specific integration as "the one that always spams"
- **Solution**: Configure event orchestration deduplication (`dedup_key`) for noisy sources, tune the upstream monitor's alerting threshold/flap detection, and route genuinely low-value pages to a non-urgent channel instead of paging

### SE-3: Schedule Gaps or Overlaps From Time Zone Misconfiguration
- **Severity**: high
- **Situation**: A rotation built by an admin in one time zone produces an unexpected on-call window (or a coverage gap) when a distributed team member's local time is factored in
- **Cause**: Schedules have an underlying time zone setting that affects rotation boundaries; building or reviewing a schedule only in the admin's local view can hide gaps visible only in another member's time zone
- **Symptoms**:
  - A team member reports being on-call at an unexpected time, or nobody was on-call during a specific window
  - The schedule "looks right" in the admin's view but wrong in another member's
- **Solution**: Review schedules in each affected member's time zone (PagerDuty supports per-user schedule view), and use the schedule overview/gaps report to check for coverage holes before relying on a new rotation

### SE-4: Urgency Misconfiguration Paging Humans for Low-Priority Noise
- **Severity**: medium
- **Situation**: A service routes all incoming events at high urgency regardless of actual severity, so on-call engineers get woken up at 2am for conditions that could have waited until business hours
- **Cause**: Urgency defaults to a static per-service setting unless explicitly configured to vary — every alert from that service inherits the same urgency whether it's a critical outage or an informational warning
- **Symptoms**:
  - On-call feedback that most night-time pages "weren't actually urgent"
  - Postmortem/incident review shows a pattern of low-severity incidents at high urgency
- **Solution**: Use support-hours-based or event-driven urgency rules so low-impact conditions route to low urgency (or a non-paging channel) outside business hours, and periodically review incident severity vs. urgency correlation

### SE-5: Missing Runbook Links Slowing MTTR
- **Severity**: medium
- **Situation**: An on-call engineer acknowledges an incident but has no documented context on what the alert means or how to start investigating, and precious minutes are lost searching for the right runbook or asking around
- **Cause**: PagerDuty incidents/services support attaching runbook links and custom details, but this is opt-in — nothing forces a service to have one configured
- **Symptoms**:
  - Incident timelines show a long gap between "acknowledged" and "first meaningful investigation action"
  - Postmortems repeatedly note "had to figure out from scratch what this alert even meant"
- **Solution**: Attach a runbook link (or at minimum a clear description with next steps) to every service/alert type, and review during postmortems whether the linked runbook was actually sufficient

## Recommended Tools

| Category | Tools |
|------|------|
| Event routing | Event Orchestration, native integrations (Datadog, New Relic, Splunk, Prometheus/Alertmanager) |
| Scheduling | PagerDuty Schedules, Slack/calendar sync |
| Postmortems | PagerDuty Postmortems, Status Updates |
| Analytics | PagerDuty Analytics (MTTA/MTTR trends) |

## On-Call Reliability

**Full checklist**: → [extended/checklists.md#on-call-reliability-checklist]

## Related Resources

[PagerDuty Docs](https://support.pagerduty.com/docs) | [Incident Response Guide](https://response.pagerduty.com/) | [PagerDuty University](https://learn.pagerduty.com/)

## Related Domains

[[datadog]] | [[new-relic]] | [[grafana-prometheus]] | [[jira]]
