---
schema: "1.0"
name: k6-jmeter
version: "1.0.0"
description: k6 and JMeter load testing — realistic traffic modeling and result interpretation
domain: technology
triggers:
  keywords:
    primary: [k6, jmeter, load testing, performance testing, stress test]
    secondary: [virtual users, ramp-up, throughput, soak test, spike test]
  context_boost: [capacity planning, benchmark, scalability test]
  context_penalty: [unit test, e2e test, playwright]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# k6 / JMeter (Load Testing)

> A load test that doesn't model real traffic just measures how fast your test tool can DDoS you

## Applicable Scenarios

- Designing a load/stress/soak/spike test plan for a service
- Modeling realistic virtual-user behavior (ramp-up, think-time)
- Correlating load test results with backend saturation, not just client-side latency
- Diagnosing whether a load test's findings reflect the target system or the test harness itself
- Coordinating load testing against shared or production-adjacent infrastructure

## Core Knowledge

### Test Types

| Type | Purpose |
|------|------|
| **Load test** | Verify behavior at expected production traffic levels |
| **Stress test** | Find the breaking point by pushing beyond expected load |
| **Soak test** | Sustained moderate load over a long duration — surfaces memory leaks, resource exhaustion, log/disk growth |
| **Spike test** | Sudden, sharp traffic increase — tests auto-scaling and burst handling |

Each answers a different question — a stress test tells you the ceiling, a soak test tells you whether you can stay below it indefinitely without degrading.

### Control Variables: VUs vs. RPS

- **k6** primarily models load in **virtual users (VUs)**, each executing a script in a loop — throughput (requests/sec) is an *emergent* result of VU count, script think-time, and response latency, not directly set
- **JMeter** can be configured either way, but its thread-group model is also fundamentally VU-based by default
- Confusing "set 500 VUs" with "500 requests/sec" is a common miscalibration — actual RPS depends heavily on how long each VU's iteration takes

### Realistic Traffic Modeling

- **Think-time** (a pause between actions, modeling a real user reading/deciding) matters — a script that fires requests back-to-back with no think-time generates far denser traffic per VU than real users ever would, and also defeats caching layers that real usage patterns would actually hit
- **Ramp-up** (gradually increasing VUs rather than an instant jump) reveals different behavior than an instantaneous spike — both are worth testing, but they answer different questions

### Test Environment Parity

- Load test results from an environment that doesn't match production (smaller instance sizes, different database scale, no CDN in front) don't transfer — a system that handles 1000 RPS in a scaled-down staging environment says little about production capacity unless the environment is genuinely representative

## Best Practices

1. **Model think-time and realistic user journeys**, not tight request loops — unrealistic scripts produce unrealistic (usually overly pessimistic or falsely optimistic) results
2. **Watch backend saturation signals** (DB connection pool usage, queue depth, CPU) alongside client-side latency — client-side numbers alone don't show *why* a system degrades
3. **Test against production-parity infrastructure** — a scaled-down environment's results don't extrapolate linearly
4. **Verify the load generator isn't the bottleneck** — monitor the load-generating machine's own CPU/network/file-descriptor usage
5. **Coordinate any test against shared/production infrastructure** with the teams who own it, in advance, every time
6. **Run soak tests, not just peak-load tests** — many real incidents come from slow degradation (leaks, log growth) invisible in a short burst test

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Script with no think-time firing requests as fast as possible | Model realistic pacing between actions to reflect actual user behavior |
| Watching only client-side latency/error rate | Correlate with backend metrics (DB, queue, CPU) to find the actual bottleneck |
| Load testing a staging environment sized nothing like production | Use production-parity infrastructure, or explicitly account for the scaling difference in interpreting results |
| Assuming "500 VUs" means "500 requests/sec" | Measure actual achieved RPS; it depends on per-iteration duration, not VU count alone |
| Running a load test against shared infra without telling anyone | Coordinate timing and scope with the infrastructure/on-call owners beforehand |

## Sharp Edges

### SE-1: The Load Generator Becoming the Bottleneck
- **Severity**: high
- **Situation**: A load test shows a hard ceiling in achievable throughput, and the team concludes the target system can't handle more — but the actual limit was the load-generating machine running out of CPU, network sockets, or file descriptors, not the system under test
- **Cause**: Generating high load is itself resource-intensive (especially with TLS handshakes, many concurrent connections); a single under-resourced test machine can hit its own ceiling well below what the target system could actually handle
- **Symptoms**:
  - The load generator's own CPU/network utilization is pegged near 100% during the test
  - Increasing target system capacity doesn't move the measured throughput ceiling at all
- **Solution**: Monitor the load generator's own resource usage during every test run, and distribute load generation across multiple machines (most tools support this natively) once a single generator's own limits are reached
- **Details**: → [extended/checklists.md#test-design-checklist]

### SE-2: Unrealistic Scripts Producing Misleading Results
- **Severity**: high
- **Situation**: A script with no think-time and identical repeated requests either dramatically understates real-world load (each VU generates far more request density than a real user) or overstates cache effectiveness (identical requests hit warm caches every time, unlike varied real traffic) — either way, the results don't predict production behavior
- **Cause**: A load test script is only as representative as the traffic pattern it encodes; naive "loop as fast as possible" scripts model a fundamentally different workload than real usage
- **Symptoms**:
  - Load test results look dramatically better (or worse) than what production actually experiences under comparable nominal traffic
  - Production incidents occur at traffic levels the load test "proved" were safe
- **Solution**: Model realistic think-time and request variety (varied query parameters, varied user data) that reflects actual production traffic patterns, ideally derived from real traffic logs/analytics rather than guessed

### SE-3: False Confidence From Non-Production-Parity Environments
- **Severity**: high
- **Situation**: A load test against a staging environment with smaller instances, a smaller database, and no CDN shows the system comfortably handling target load — production, under the same nominal load, degrades badly
- **Cause**: Performance characteristics (especially database query plans, cache hit rates, and network topology effects) often don't scale linearly with instance size — a system that looks fine at small scale can hit entirely different bottlenecks at production scale
- **Symptoms**:
  - A "passed" load test is followed by a real production incident at comparable or lower traffic
  - Bottleneck in production (e.g., a specific slow query) never appeared in staging because the staging dataset was too small to trigger it
- **Solution**: Test against infrastructure and data volume that's genuinely representative of production, or explicitly document and account for known differences when interpreting results — don't treat a staging pass as a production guarantee

### SE-4: Watching Only Client-Side Latency, Missing Backend Saturation
- **Severity**: medium
- **Situation**: A load test shows client-side latency climbing and error rates rising, but the team can't explain why — because nobody was watching database connection pool exhaustion, queue depth, or CPU saturation on the backend during the run
- **Cause**: Client-side metrics show *that* something degraded, not *why* — the actual bottleneck (a connection pool, a lock, a downstream dependency) is invisible without correlating backend telemetry to the same time window
- **Symptoms**:
  - Post-test analysis can describe symptoms (latency, errors) but not root cause
  - The same load test re-run after an unrelated change shows different results with no clear explanation
- **Solution**: Instrument and monitor backend saturation signals (connection pool usage, queue depth, CPU/memory, downstream dependency latency) during every load test run, with dashboards spanning the same time window as the test, so results are actionable rather than just descriptive

### SE-5: Load Testing Shared Infrastructure Without Coordination
- **Severity**: critical
- **Situation**: A load test run against a shared staging environment (or worse, production-adjacent shared infrastructure) causes a real incident for other teams relying on that environment at the same time
- **Cause**: Load tests intentionally push a system toward its limits — without coordination, this looks and behaves exactly like an unplanned traffic spike or a self-inflicted denial of service to everyone else using the shared resource
- **Symptoms**:
  - An unrelated team's incident correlates exactly with a load test's start time
  - Postmortem reveals a load test as the actual root cause of a "production" incident
- **Solution**: Always coordinate load test timing, scope, and expected impact with the owners of any shared infrastructure beforehand, and prefer isolated, production-parity environments dedicated to load testing where the blast radius is contained

## Recommended Tools

| Category | Tools |
|------|------|
| Modern scriptable load testing | k6, Gatling |
| GUI-based / legacy | Apache JMeter |
| Distributed load generation | k6 Cloud, JMeter distributed testing, Locust |
| Correlation/observability | Grafana + Prometheus (backend metrics alongside load test results) |

## Test Design

**Full checklist**: → [extended/checklists.md#test-design-checklist]

## Related Resources

[k6 Docs](https://grafana.com/docs/k6/latest/) | [Apache JMeter Docs](https://jmeter.apache.org/usermanual/index.html) | [Grafana k6 Load Testing Guide](https://grafana.com/load-testing/)

## Related Domains

[[grafana-prometheus]] | [[postgresql]] | [[kubernetes]] | [[playwright-cypress]]
