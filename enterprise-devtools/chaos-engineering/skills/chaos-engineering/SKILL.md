---
schema: "1.0"
name: chaos-engineering
version: "1.0.0"
description: Chaos engineering experiment design, blast radius control, steady-state metrics, and resilience validation practices
domain: technology
triggers:
  keywords:
    primary: [chaos engineering, gremlin, chaos mesh, fault injection]
    secondary: [blast radius, steady state, resilience testing, game day]
  context_boost: [reliability testing, incident response readiness, disaster recovery drill]
  context_penalty: [unit testing, load testing]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Chaos Engineering

> A controlled experiment with a hypothesis and an abort plan — not "randomly break things and see what happens"

## Applicable Scenarios

- Designing a chaos experiment to validate a specific resilience assumption
- Planning the blast-radius progression from non-production to production experiments
- Running a game day to test on-call readiness and incident response process
- Diagnosing why a system failed in ways prior "resilience" work didn't predict
- Reviewing whether existing redundancy/failover mechanisms actually work under real failure

## Core Knowledge

### The Scientific Method Applied to Systems

A real chaos experiment has, before anything is broken:

1. **Hypothesis** — a specific, falsifiable claim ("if the primary database fails, traffic fails over within 30 seconds with no user-visible errors")
2. **Steady-state metric** — a measurable signal that defines "normal," monitored throughout the experiment
3. **Blast radius** — the deliberately bounded scope of what's allowed to be affected
4. **Abort/rollback plan** — a pre-defined, tested way to stop the experiment and restore normal state if the steady-state metric breaches a threshold

Without all four, it's not an experiment — it's just breaking something and hoping to learn something useful from the resulting incident.

### Blast Radius Progression

- Start in **non-production** (staging, a dedicated chaos environment) to validate the experiment mechanics themselves before touching anything real
- In production, start with the **smallest meaningful blast radius** — one instance, one AZ, a small percentage of traffic — and only expand once smaller experiments succeed
- Magnitude and blast radius are two separate dials: a "small" experiment can still be high-magnitude (kill one instance completely) or low-magnitude (add 50ms latency) — both dimensions should be deliberately chosen, not just "big vs. small"

### Common Experiment Types

| Experiment | Validates |
|------|------|
| **Latency injection** | Timeout handling, circuit breakers, degraded-mode behavior |
| **Resource exhaustion** (CPU/memory/disk) | Autoscaling, backpressure, graceful degradation under load |
| **Dependency failure** | Fallback logic, retry behavior, circuit breaker tripping correctly |
| **Instance/zone/region failure** | Failover, redundancy, load balancer health checking |
| **Network partition** | Split-brain handling, consensus protocol behavior |

### Game Days

- A **game day** is a scheduled, announced exercise where a team runs a chaos experiment (or a simulated incident) together, testing not just the system but the *people and process* — on-call response, runbook accuracy, communication
- Distinct from ad-hoc automated chaos experiments in that the explicit goal includes validating human response, not just system behavior

## Best Practices

1. **Always define a steady-state metric and abort threshold before starting** — no experiment without a way to know when to stop
2. **Progress blast radius deliberately** — non-production first, then smallest viable production scope, expanding only after success
3. **Ensure on-call/stakeholders know an experiment is running** — a surprise chaos experiment is indistinguishable from a real incident to the people responding
4. **Automate rollback**, don't rely on a human remembering to manually revert under pressure
5. **Treat a successful experiment as one data point**, not proof of permanent resilience — systems change, and yesterday's validated resilience can regress silently
6. **Feed findings back into the system**, not just into a report — a chaos experiment that reveals a gap and doesn't lead to a fix has only documented risk, not reduced it

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Running an experiment with no defined steady-state metric | Define and monitor a specific, measurable steady-state signal before starting |
| No tested rollback/abort mechanism | Build and test the abort path before running the experiment for real |
| Jumping straight to production without non-production validation | Validate experiment mechanics in non-production first |
| Running experiments without on-call/stakeholder awareness | Announce and coordinate experiments, especially early on, so they don't get mistaken for real incidents |
| Treating one successful experiment as permanent proof of resilience | Repeat experiments periodically, especially after significant system changes |

## Sharp Edges

### SE-1: No Steady-State Metric, No Way to Know When to Abort
- **Severity**: critical
- **Situation**: A chaos experiment is run with a vague goal ("see what happens if we kill this service") but no specific metric being watched, and by the time someone notices real user impact, it's already been happening for a while
- **Cause**: Without a pre-defined, actively monitored steady-state metric and threshold, there's no objective signal for "this experiment has gone beyond acceptable impact" — the team is relying on someone happening to notice
- **Symptoms**:
  - An experiment's actual user-facing impact is discovered after the fact via customer reports or unrelated alerting, not via the experiment's own monitoring
  - Post-experiment review can't answer "how bad did it actually get and for how long"
- **Solution**: Define a specific, quantifiable steady-state metric (error rate, latency percentile, successful-transaction rate) and an explicit abort threshold before designing the rest of the experiment, and have it actively displayed/monitored by the person running the experiment throughout
- **Details**: → [extended/checklists.md#experiment-safety-checklist]

### SE-2: Untested Abort Mechanism Turning an Experiment Into a Real Incident
- **Severity**: critical
- **Situation**: The steady-state metric breaches threshold, the team tries to abort/rollback the experiment, and the rollback itself doesn't work cleanly (or takes far longer than expected) — a controlled experiment becomes an actual, uncontrolled incident
- **Cause**: An abort plan that's only ever been theorized, not actually tested, can have the same kind of failure modes as any untested procedure — assumptions about how quickly a fault can be reversed often don't hold under real conditions
- **Symptoms**:
  - Time-to-recovery during the experiment significantly exceeds the planned abort time
  - The "chaos experiment" postmortem reads like a real incident postmortem
- **Solution**: Test the abort/rollback mechanism itself, ideally as part of validating the experiment in non-production first, and have it be a fast, well-rehearsed action (automated where possible) rather than a manual multi-step process improvised under pressure

### SE-3: Blast Radius Escaping Its Intended Boundary
- **Severity**: high
- **Situation**: An experiment designed to affect "one instance" or "one AZ" ends up affecting a much larger portion of the system than intended, because of an unanticipated coupling (a shared dependency, a connection pool exhausted cluster-wide, a cascading retry storm)
- **Cause**: Systems have hidden coupling that isn't always visible from the architecture diagram — a "contained" failure in one component can cascade through shared resources (databases, caches, thread pools) that weren't accounted for in the blast-radius design
- **Symptoms**:
  - Monitoring during the experiment shows impact in systems/services that weren't the intended target
  - The actual blast radius, measured after the fact, is significantly larger than planned
- **Solution**: Map out shared dependencies and potential cascade paths before designing the experiment's blast radius, start with the smallest possible magnitude/scope even for a "well-understood" component, and have broader monitoring in place during the experiment than just the directly targeted component

### SE-4: Surprise Experiments Mistaken for Real Incidents
- **Severity**: medium
- **Situation**: A chaos experiment runs without on-call being informed, on-call pages, investigates, and burns significant time and stress treating it as a genuine incident before realizing it was a planned experiment
- **Cause**: A well-executed chaos experiment is, by design, hard to distinguish from a real failure from the responder's point of view — without advance coordination, there's no way for on-call to know the difference
- **Symptoms**:
  - On-call reports being paged for something that turned out to be a scheduled experiment
  - Trust in the chaos engineering program erodes because it's associated with unnecessary stress/false alarms
- **Solution**: Announce experiments in advance to on-call and relevant stakeholders (with enough detail to plan around, but without necessarily revealing every detail if testing detection is also a goal), and clearly communicate the experiment's start/end so responders know when to stand down

### SE-5: One Successful Experiment Treated as Permanent Proof of Resilience
- **Severity**: medium
- **Situation**: A system passes a chaos experiment validating failover behavior, everyone considers that risk "handled," and months later a real failure of the same type causes an outage — because the system had changed in the interim and the validated behavior had silently regressed
- **Cause**: Resilience isn't a static property — deploys, configuration changes, dependency upgrades, and scaling changes can all quietly break a failure-handling path that was validated once and never re-tested
- **Symptoms**:
  - A real incident occurs in exactly the failure mode a past chaos experiment supposedly validated
  - Investigation reveals a configuration or code change, unrelated to the original experiment, broke the failover/fallback path at some point after it was last tested
- **Solution**: Repeat chaos experiments on a regular cadence (not just once), and especially after significant changes to the components/paths a given experiment validates, treating chaos engineering as continuous validation rather than a one-time certification

## Recommended Tools

| Category | Tools |
|------|------|
| Fault injection platforms | Gremlin, Chaos Mesh, LitmusChaos |
| Kubernetes-native | Chaos Mesh, PowerfulSeal |
| AWS-specific | AWS Fault Injection Simulator |
| Game day facilitation | Runbooks, incident response tooling (PagerDuty) |

## Experiment Safety

**Full checklist**: → [extended/checklists.md#experiment-safety-checklist]

## Related Resources

[Principles of Chaos Engineering](https://principlesofchaos.org/) | [Chaos Engineering (O'Reilly)](https://www.oreilly.com/library/view/chaos-engineering/9781491988459/) | [Gremlin Docs](https://www.gremlin.com/docs/)

## Related Domains

[[pagerduty]] | [[kubernetes]] | [[grafana-prometheus]] | [[launchdarkly]]
