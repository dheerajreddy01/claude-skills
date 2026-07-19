---
schema: "1.0"
name: new-relic
version: "1.0.0"
description: New Relic APM, NRQL queries, distributed tracing, and alerting practices
domain: technology
triggers:
  keywords:
    primary: [new relic, newrelic, NRQL, APM, distributed tracing]
    secondary: [entity, synthetics, alert policy, ingest, custom event]
  context_boost: [observability, performance monitoring, application monitoring]
  context_penalty: [datadog, splunk, grafana]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# New Relic

> Application performance monitoring with ingest costs that scale with what you send it

## Applicable Scenarios

- Instrumenting applications with New Relic APM agents
- Writing or optimizing NRQL queries and dashboards
- Designing alert policies (NRQL alerts, baseline/anomaly alerts)
- Diagnosing distributed tracing gaps across service boundaries
- Reviewing ingest volume and cost drivers

## Core Knowledge

### Entity Model

| Entity Type | Covers |
|------|------|
| **APM** | Application-level performance (transactions, error rate, throughput, Apdex) |
| **Infrastructure** | Host/container-level metrics |
| **Browser** | Real-user monitoring (RUM) for frontend performance |
| **Synthetics** | Scripted uptime/functional checks from external locations |
| **Logs** | Correlated log ingestion, linked to APM entities |

New Relic's "entity" concept ties these together — a well-instrumented service shows APM, infrastructure, and logs correlated under one entity, which is what makes cross-signal debugging fast.

### NRQL (New Relic Query Language)

- SQL-like syntax: `SELECT average(duration) FROM Transaction WHERE appName = 'checkout-service' SINCE 1 hour ago`
- `FACET` groups results (similar to `GROUP BY`) — heavy use of `FACET` on high-cardinality attributes (user ID, request ID) is a common cost/performance trap
- Custom events and custom attributes are billed as ingest — every attribute added to a transaction event multiplies across every transaction

### Distributed Tracing

- Requires consistent trace context propagation (W3C Trace Context or New Relic's proprietary header) across every service hop, including through message queues and async boundaries
- A trace breaks (shows a gap) wherever a hop doesn't propagate the incoming trace header to its outgoing calls — often missed in newly added services or non-HTTP integrations (queues, background jobs)

### Alerting

| Alert Type | Use |
|------|------|
| **Static NRQL alert** | Fixed threshold breach (e.g., error rate > 5%) |
| **Baseline alert** | Deviates from a dynamically learned normal range — good for metrics without a fixed "correct" value |
| **Anomaly detection** | ML-based deviation detection for less predictable signals |

Alert conditions evaluate over a defined window; too short a window causes flapping on normal noise, too long delays real incident detection.

## Best Practices

1. **Instrument consistently** — every service in a call chain needs the agent (or manual trace context propagation) or traces break at that hop
2. **Keep custom attribute cardinality bounded** — don't attach unbounded values (user IDs, request IDs) as indexed attributes on every event
3. **Use dashboards backed by NRQL you've checked for cost**, not ad hoc high-cardinality FACETs on frequently-refreshed panels
4. **Tag services consistently** (environment, team, service name) so alerts and dashboards can be filtered reliably
5. **Set alert evaluation windows deliberately** — match the window to how quickly the underlying metric actually changes
6. **Review ingest by data type regularly** — APM, logs, and custom events often have very different cost profiles

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Adding high-cardinality custom attributes (user ID, session ID) to every transaction | Use them sparingly, or send as non-indexed attributes where full querying isn't needed |
| Alert threshold set from a single "bad day" observation | Base thresholds on historical baselines, or use baseline/anomaly alert types |
| Assuming a trace is complete just because APM shows green | Verify actual trace continuity across service hops, especially through queues/async work |
| Ignoring ingest cost until the bill arrives | Review the Data Explorer / ingest usage dashboard proactively, before cost becomes a surprise |
| `FACET` on an unbounded high-cardinality field in a frequently-run dashboard query | Use bounded facets (service name, status code) or pre-aggregate |

## Sharp Edges

### SE-1: Ingest Cost Overrun From Verbose Custom Events
- **Severity**: critical
- **Situation**: A team adds detailed custom events or high-cardinality custom attributes to every transaction "for debugging," and the ingest bill spikes well beyond expectations
- **Cause**: New Relic's consumption pricing bills by data ingested (GB); custom events/attributes multiply by transaction volume, and there's no default cap stopping a verbose instrumentation choice from scaling cost with traffic
- **Symptoms**:
  - Data Explorer shows a custom event type dominating ingest volume
  - Cost climbs in proportion to traffic growth, disproportionate to infra cost growth
- **Solution**: Audit custom event/attribute usage against actual query/debugging value, drop attributes that are never queried, and use sampling for high-volume debug-only events
- **Details**: → [extended/checklists.md#cost-and-ingest-checklist]

### SE-2: Alert Fatigue From Poorly Tuned Thresholds
- **Severity**: high
- **Situation**: A static threshold alert (e.g., "error rate > 1%") fires constantly on normal traffic variance, and the team starts ignoring or muting alerts entirely — including real incidents
- **Cause**: Static thresholds set without reference to actual historical variance don't account for normal fluctuation (traffic patterns, deploys, time-of-day)
- **Symptoms**:
  - High volume of alert notifications with low correlation to actual user-impacting incidents
  - On-call engineers report alert channels being muted or ignored
- **Solution**: Use baseline or anomaly-detection alert types for metrics with natural variance, tune evaluation windows to avoid flapping, and periodically review alert-to-incident correlation to prune noisy conditions

### SE-3: Broken Distributed Traces Across Async Boundaries
- **Severity**: high
- **Situation**: A trace shows a gap or appears to terminate at a message queue publish, with no visible continuation in the consumer service
- **Cause**: Trace context propagation must be manually wired through non-HTTP integrations (queues, background job frameworks, custom RPC) — the agent doesn't automatically propagate context through arbitrary message payloads
- **Symptoms**:
  - A trace for a request that clearly continues asynchronously (e.g., queued job) shows as "complete" at the publish step
  - Root-cause investigation across async services requires manually correlating timestamps/IDs instead of following one trace
- **Solution**: Manually propagate trace context (attach New Relic's or W3C trace headers to the message payload) at every async boundary, and verify trace continuity end-to-end for critical async flows, not just synchronous HTTP paths

### SE-4: High-Cardinality FACET Queries Slowing Dashboards
- **Severity**: medium
- **Situation**: A dashboard panel using `FACET userId` (or similarly unbounded field) becomes slow to render and expensive to evaluate on every refresh
- **Cause**: `FACET` on a high-cardinality attribute forces the query engine to compute and return a huge number of distinct groups, which doesn't scale the way faceting on a bounded field (e.g., `status code`, `region`) does
- **Symptoms**:
  - Specific dashboard panels consistently slower to load than others
  - NRQL query timing out or truncating results at New Relic's series/group limits
- **Solution**: Facet on bounded, low-cardinality fields; for high-cardinality analysis, use `WHERE` filters to scope to a specific value rather than faceting across all of them

### SE-5: Default Agent Sampling Missing Critical Transactions
- **Severity**: medium
- **Situation**: A rare but important transaction type (e.g., a specific error path or slow outlier) doesn't appear in APM traces because it fell outside the agent's default trace sampling
- **Cause**: APM agents sample traces (not every transaction is traced in detail) to control overhead and ingest cost by default
- **Symptoms**:
  - A known-slow or known-erroring transaction type is hard to find a representative trace for
  - Trace count for a given transaction type is much lower than its actual call volume
- **Solution**: Configure targeted sampling rules (e.g., always trace errors, or a higher sample rate for a specific critical transaction) rather than relying on the default uniform sampling rate

## Recommended Tools

| Category | Tools |
|------|------|
| Query/dashboards | NRQL, New Relic Query Builder, Dashboards |
| Alerting | NRQL Alerts, Workflows (notification routing) |
| Instrumentation | APM agents (per-language), OpenTelemetry integration |
| Cost visibility | Data Explorer, ingest usage dashboards |

## Cost & Ingest Management

**Full checklist**: → [extended/checklists.md#cost-and-ingest-checklist]

## Related Resources

[New Relic Docs](https://docs.newrelic.com/) | [NRQL Reference](https://docs.newrelic.com/docs/nrql/nrql-syntax-clauses-functions/) | [New Relic University](https://learn.newrelic.com/)

## Related Domains

[[datadog]] | [[splunk]] | [[grafana-prometheus]] | [[pagerduty]]
