---
schema: "1.0"
name: datadog
version: "1.0.0"
description: Datadog tagging strategy, monitors, unified service tagging, and cost management across metrics/logs/APM
domain: technology
triggers:
  keywords:
    primary: [datadog, dd-agent, monitor, unified service tagging]
    secondary: [custom metric, log pipeline, APM, dashboard, tag cardinality]
  context_boost: [observability, monitoring, infrastructure metrics]
  context_penalty: [new relic, splunk, grafana]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Datadog

> Tags are the join key across every product — get them consistent or nothing correlates

## Applicable Scenarios

- Designing tagging strategy across infrastructure, APM, and logs
- Writing or tuning monitors (alerts) to avoid flapping/noise
- Diagnosing unexpected billing (custom metrics, log indexing, APM host count)
- Setting up unified service tagging for cross-product correlation
- Building dashboards that stay fast and accurate as infrastructure scales

## Core Knowledge

### The Three Pillars, One Join Key

| Pillar | What It Covers |
|------|------|
| **Metrics** | Time-series numeric data (infra, custom app metrics) |
| **Logs** | Structured/unstructured log ingestion and indexing |
| **APM/Traces** | Distributed tracing, service maps, trace-level analytics |

All three correlate through **tags** — the same `env`, `service`, and `version` tags applied consistently across metrics, logs, and traces are what let you pivot from a trace to its logs to its infra metrics in one click. Inconsistent tagging silently breaks that correlation without erroring.

### Unified Service Tagging

Datadog's convention: every entity should carry `env`, `service`, and `version` tags with matching values across all three pillars. Applied at the agent/library level, this is what powers the Service Catalog and cross-product navigation — skipping it doesn't break ingestion, it just breaks the workflows that depend on it.

### Tag Cardinality

- Tags are indexed for filtering/grouping — a tag with unbounded unique values (request ID, user ID, raw timestamp) creates a new time series combination for every distinct value
- Custom metrics are billed **per unique tag combination (time series)**, not per metric name — a metric with 3 low-cardinality tags might be a handful of time series; add one high-cardinality tag and it becomes thousands

### Monitors

| Monitor Type | Use |
|------|------|
| **Metric threshold** | Fixed value breach over an evaluation window |
| **Anomaly** | Deviation from a learned seasonal baseline |
| **Composite** | Combines multiple existing monitors with boolean logic |
| **Log/APM/Process/etc.** | Same threshold model, scoped to a different data source |

Evaluation window and "require N of M data points" settings control flapping — a monitor with no minimum duration alerts on a single noisy sample.

## Best Practices

1. **Apply unified service tagging (`env`, `service`, `version`) consistently** across the agent, APM libraries, and log pipelines
2. **Treat tags as a cost lever** — every additional tag dimension on a custom metric multiplies billed time series
3. **Use monitor "require N of M evaluations"** to avoid single-sample flapping
4. **Scope log indexing deliberately** — use exclusion filters/log pipelines to index only what's actually queried, archive the rest
5. **Group related alerts with multi-alert monitors** (grouped by tag) instead of one monitor per host/instance
6. **Review the Usage tab regularly** — custom metrics, indexed logs, and APM host count are the primary cost drivers

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Tagging custom metrics with request ID / user ID | Keep custom metric tags to bounded dimensions (service, region, status) |
| Inconsistent `service` tag spelling across metrics/logs/traces | Standardize via unified service tagging at the agent/library config level |
| Monitor alerting on a single data point spike | Require multiple consecutive evaluations before triggering |
| Indexing 100% of logs by default | Use exclusion filters to index selectively, keep the rest in log archives |
| One monitor per host instead of a grouped monitor | Use a single monitor scoped/grouped by tag, with per-group alerting |

## Sharp Edges

### SE-1: Tag Cardinality Explosion Driving Custom Metric Cost
- **Severity**: critical
- **Situation**: A custom metric tagged with a high-cardinality dimension (user ID, container ID that churns constantly, raw request path) silently multiplies the billed time series count, and the custom metrics bill spikes
- **Cause**: Datadog bills custom metrics per unique tag-value combination (time series), and nothing blocks a high-cardinality tag from being added
- **Symptoms**:
  - Usage tab shows custom metric time series count far exceeding the number of distinct metric names in use
  - Cost grows in proportion to a churny dimension (e.g., ephemeral container/pod count) rather than actual metric volume
- **Solution**: Audit custom metric tags for cardinality before shipping, avoid tagging with identifiers that are unique per request/user, and use the Metrics Summary page to find and fix high-cardinality offenders
- **Details**: → [extended/checklists.md#cost-and-tagging-checklist]

### SE-2: Broken Cross-Product Correlation From Inconsistent Tagging
- **Severity**: high
- **Situation**: A trace shows a service name that doesn't match the tag used on that service's logs or infrastructure metrics, so the "jump from trace to related logs" workflow returns nothing
- **Cause**: Unified service tagging (`env`/`service`/`version`) has to be applied consistently at the agent, APM library, and log pipeline level — if one path uses a different tag value or is missing the tag entirely, correlation silently fails
- **Symptoms**:
  - "Related Logs" or "Related Traces" tabs show no results despite logs/traces clearly existing for that service
  - Service Catalog shows duplicate or fragmented entries for what should be one service
- **Solution**: Standardize tag values (including case) across every ingestion path for a service, verify with a known trace that related logs/metrics actually resolve, and fix at the source (agent config/library init) rather than patching downstream

### SE-3: Log Indexing Costs From Unfiltered Ingestion
- **Severity**: high
- **Situation**: All logs are indexed (fully searchable) by default, and verbose or low-value log sources (health checks, debug-level app logs) dominate the indexed-log bill
- **Cause**: Indexed logs are billed separately and more expensively than log ingestion/archiving — indexing everything by default doesn't distinguish "needs to be searchable" from "just needs to be retained for compliance"
- **Symptoms**:
  - Usage tab shows indexed log volume dominated by a small number of noisy, rarely-queried sources
- **Solution**: Use exclusion filters or log pipelines to route low-value logs to cheaper archive storage (S3/GCS-backed) instead of the indexed tier, and reserve indexing for logs actually used in search/monitors/dashboards

### SE-4: Monitor Flapping From Missing Evaluation Windows
- **Severity**: medium
- **Situation**: A metric monitor fires and resolves repeatedly within minutes because it alerts on any single data point crossing the threshold, generating alert fatigue
- **Cause**: Without an explicit "require N of M evaluations" or a wider evaluation window, a monitor reacts to normal single-sample noise rather than a sustained condition
- **Symptoms**:
  - Alert channel shows the same monitor triggering and auto-resolving repeatedly in a short window
  - On-call reports the monitor as "always flapping" and starts ignoring it
- **Solution**: Set an evaluation window and a "require N of M data points" threshold appropriate to how quickly the underlying metric genuinely changes, and use anomaly detection for metrics with natural seasonal variance instead of a fixed threshold

### SE-5: APM Host/Container Count Driving Unexpected Billing
- **Severity**: medium
- **Situation**: APM is billed per host (or per container in some plans), and autoscaling or a container churn pattern (many short-lived containers) inflates the billed host count well beyond the steady-state count
- **Cause**: Some Datadog billing dimensions count based on distinct hosts/containers observed over a billing period, not concurrent count at any instant — high churn environments (spot instances, aggressive autoscaling, CI runners) can rack up count quickly
- **Symptoms**:
  - APM host count in the Usage tab is much higher than the infrastructure team's understanding of steady-state fleet size
- **Solution**: Review the Usage tab's host/container count methodology for the current billing model, and consider excluding short-lived CI/ephemeral instances from full APM instrumentation if they're not the ones being investigated in practice

## Recommended Tools

| Category | Tools |
|------|------|
| Tagging/config | Datadog Agent config, Unified Service Tagging setup |
| Cost visibility | Usage & Cost pages, Metrics Summary (cardinality) |
| Dashboards/alerts | Datadog Dashboards, Monitors, Notebooks |
| Log management | Log Pipelines, Exclusion Filters, Log Archives |

## Cost & Tagging Hygiene

**Full checklist**: → [extended/checklists.md#cost-and-tagging-checklist]

## Related Resources

[Datadog Docs](https://docs.datadoghq.com/) | [Unified Service Tagging](https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/) | [Datadog Learning Center](https://learn.datadoghq.com/)

## Related Domains

[[new-relic]] | [[splunk]] | [[grafana-prometheus]] | [[pagerduty]]
