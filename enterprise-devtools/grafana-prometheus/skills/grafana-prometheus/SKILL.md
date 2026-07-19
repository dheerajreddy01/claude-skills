---
schema: "1.0"
name: grafana-prometheus
version: "1.0.0"
description: Prometheus scraping/PromQL, Alertmanager routing, and Grafana dashboard practices for the open-source observability stack
domain: technology
triggers:
  keywords:
    primary: [grafana, prometheus, promql, alertmanager, metrics]
    secondary: [scrape, exporter, service discovery, thanos, mimir, cortex, remote write]
  context_boost: [open source, kubernetes, self-hosted observability]
  context_penalty: [datadog, new relic, splunk]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Grafana + Prometheus

> Pull-based metrics, PromQL rate math, and dashboards that don't lie about what they're showing

## Applicable Scenarios

- Designing Prometheus scrape configuration and service discovery
- Writing or debugging PromQL queries and alerting rules
- Configuring Alertmanager routing, grouping, and silences
- Building Grafana dashboards with variables that stay correct across environments
- Diagnosing cardinality-driven Prometheus memory/performance problems

## Core Knowledge

### Prometheus Model

- **Pull-based**: Prometheus scrapes `/metrics` endpoints on a configured interval, rather than receiving pushed data — targets are discovered via static config, file-based SD, or dynamic service discovery (Kubernetes, Consul, etc.)
- **Time series identity**: a series is uniquely identified by its metric name + the full set of label key/value pairs — every distinct label combination is a separate series
- **Local storage**: single-instance Prometheus stores data on local disk with a retention window; for long-term/HA storage, a remote-write target (Thanos, Mimir, Cortex) is standard, not a Prometheus built-in
- **Exporters**: translate a system's native metrics (node stats, database stats, etc.) into Prometheus's text exposition format — `node_exporter`, `postgres_exporter`, etc.

### PromQL Essentials

| Concept | Detail |
|------|------|
| **Counter** | Monotonically increasing value (e.g., total requests) — always wrap with `rate()`/`irate()`, never read raw |
| **Gauge** | Value that can go up or down (e.g., memory usage) — safe to read directly |
| **Histogram** | Bucketed observations (e.g., request duration) — use `histogram_quantile()` over `rate()` of the `_bucket` series |
| **`rate()`** | Per-second average rate of increase over a time window — correctly handles counter resets (restarts) |

`rate()` requires at least two data points within its range window — too short a window relative to the scrape interval returns gaps or `NaN`.

### Alertmanager

- Prometheus alerting rules define *when* something fires; Alertmanager handles *what happens next* — grouping, deduplication, routing, silencing, and notification delivery
- **Routing tree**: alerts match routes top-down based on label matchers; a catch-all route at the bottom prevents alerts from being silently dropped
- **Grouping** batches related alerts (e.g., by `alertname`+`cluster`) into one notification instead of one page per firing series

### Grafana

- **Data sources** connect to Prometheus (and others) per dashboard; a dashboard's variables (e.g., `$environment`, `$cluster`) drive which series a panel queries
- **Template variables** must be wired consistently to every panel's query — a panel with a hardcoded label value ignores the dashboard's variable selector, silently showing the wrong environment's data

## Best Practices

1. **Always `rate()` counters** — reading a raw counter's current value tells you almost nothing useful on its own
2. **Keep label cardinality bounded** — Prometheus's memory usage scales with the number of active series, and unbounded labels (user ID, raw URL path) can crash a scrape target
3. **Set a sane global scrape interval and stick to it** — mismatched intervals across targets make cross-service rate comparisons misleading
4. **Design Alertmanager's routing tree with an explicit catch-all**, so no unmatched alert silently disappears
5. **Wire every dashboard panel to the shared template variables** — no hardcoded label values that silently ignore the environment/cluster selector
6. **Use recording rules for expensive, frequently-queried PromQL** — pre-compute rather than re-evaluating a heavy query on every dashboard load

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Reading a counter's raw value instead of `rate()` | Always wrap counters in `rate()`/`irate()` before using them |
| Unbounded labels on custom metrics (user ID, request ID) | Keep custom metric labels to bounded, low-cardinality dimensions |
| Single Prometheus instance with no HA/remote-write for critical alerting | Run Prometheus HA pairs or federate to Thanos/Mimir/Cortex for durability |
| Alertmanager routing tree with no catch-all route | Add an explicit default route so unmatched alerts aren't silently dropped |
| Grafana panel with a hardcoded label instead of `$variable` | Bind every panel query to the dashboard's template variables |

## Sharp Edges

### SE-1: Cardinality Explosion Crashing Prometheus
- **Severity**: critical
- **Situation**: A metric labeled with an unbounded dimension (user ID, raw request path, pod name that churns constantly) causes Prometheus's memory usage to grow until the process OOMs or scrapes start timing out
- **Cause**: Every unique label combination is a separate time series held in memory; an unbounded label multiplies series count directly with the cardinality of that label
- **Symptoms**:
  - Prometheus memory usage grows steadily, uncorrelated with actual infrastructure growth
  - `prometheus_tsdb_head_series` climbs sharply after a specific metric/label was added
- **Solution**: Audit label cardinality before adding a new metric label (use `count by (__name__)(...)` style checks), avoid identifiers as labels, and set `sample_limit`/cardinality alerting on scrape targets to catch regressions early
- **Details**: → [extended/checklists.md#cardinality-and-reliability-checklist]

### SE-2: Counter Reset Misreads From Skipping `rate()`
- **Severity**: high
- **Situation**: A dashboard panel graphs a raw counter value, and every process restart shows as a sharp drop to zero that looks like an incident
- **Cause**: Counters reset to zero on process restart; reading the raw value (instead of `rate()`, which correctly handles resets) conflates "the process restarted" with "the metric decreased"
- **Symptoms**:
  - Graphs show sawtooth patterns that don't correspond to any real operational event
  - Alerts firing on "counter decreased," which should never be a meaningful condition for a counter
- **Solution**: Never alert or graph raw counter values directly — always use `rate()`/`increase()`, which are specifically designed to handle counter resets correctly

### SE-3: Single Point of Failure in Prometheus Without HA/Remote Write
- **Severity**: high
- **Situation**: A single Prometheus instance is the only source of metrics and alerting for a critical system, and it goes down (host failure, disk full, OOM from cardinality) — leaving the team blind and unalerted during exactly the time an incident is most likely to also be happening
- **Cause**: Vanilla Prometheus is a single-node system by design; nothing about the default setup provides redundancy or durable long-term storage
- **Symptoms**:
  - A Prometheus outage coincides with (or causes) a gap in incident visibility
  - No metrics history survives a disk failure because there was no remote-write target
- **Solution**: Run Prometheus in HA pairs (two identically-configured instances scraping the same targets) fronted by Alertmanager's built-in deduplication, and use remote-write to Thanos/Mimir/Cortex for durable, long-term, globally-queryable storage

### SE-4: Alertmanager Routing Gaps Silencing Real Alerts
- **Severity**: critical
- **Situation**: An alert fires in Prometheus but no notification is ever delivered, because it didn't match any route in Alertmanager's routing tree and there was no catch-all
- **Cause**: Alertmanager only delivers notifications for routes an alert actually matches; without an explicit default/catch-all route, an alert with an unanticipated label combination is silently swallowed
- **Symptoms**:
  - A known incident had a firing alert in Prometheus's `/alerts` page, but no page/notification was ever sent
  - Alertmanager's routing tree has specific routes for known teams/services but no fallback
- **Solution**: Always include a catch-all route at the bottom of the routing tree pointing to a monitored channel, and periodically test routing with Alertmanager's route-testing tool against realistic label sets

### SE-5: Grafana Dashboard Variables Showing the Wrong Environment's Data
- **Severity**: medium
- **Situation**: A dashboard has an `$environment` selector, but one panel was built with a hardcoded `env="prod"` label instead of `env="$environment"` — switching the selector to staging still silently shows prod data on that one panel
- **Cause**: Grafana template variables only affect panels whose queries explicitly reference them; nothing forces every panel in a dashboard to use the same variable
- **Symptoms**:
  - One panel's data doesn't change when the dashboard's environment/cluster selector is changed
  - An on-call engineer makes a decision based on the wrong environment's data during an incident
- **Solution**: Audit every panel's query for hardcoded label values that should instead reference `$variable`, and use dashboard JSON model review or provisioning-as-code to catch this systematically rather than per-panel manual review

## Recommended Tools

| Category | Tools |
|------|------|
| Long-term storage / HA | Thanos, Mimir, Cortex |
| Exporters | node_exporter, kube-state-metrics, blackbox_exporter |
| Config validation | promtool, amtool |
| Dashboards as code | Grafonnet, Grafana provisioning YAML |

## Cardinality & Reliability

**Full checklist**: → [extended/checklists.md#cardinality-and-reliability-checklist]

## Related Resources

[Prometheus Docs](https://prometheus.io/docs/) | [PromQL Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/) | [Grafana Docs](https://grafana.com/docs/grafana/latest/)

## Related Domains

[[datadog]] | [[new-relic]] | [[splunk]] | [[pagerduty]]
