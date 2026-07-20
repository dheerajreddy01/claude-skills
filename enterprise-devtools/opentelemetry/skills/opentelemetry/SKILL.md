---
schema: "1.0"
name: opentelemetry
version: "1.0.0"
description: OpenTelemetry instrumentation, context propagation, Collector pipeline design, and sampling practices
domain: technology
triggers:
  keywords:
    primary: [opentelemetry, otel, otel collector, instrumentation]
    secondary: [trace context propagation, span, otlp, semantic conventions]
  context_boost: [vendor-neutral observability, distributed tracing standard]
  context_penalty: [datadog, new-relic, honeycomb]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# OpenTelemetry

> One instrumentation standard, many backends — but only if context propagation and semantic conventions actually hold together

## Applicable Scenarios

- Instrumenting services with vendor-neutral traces, metrics, and logs
- Designing an OpenTelemetry Collector pipeline (receivers/processors/exporters)
- Propagating trace context across service, queue, and async boundaries
- Deciding between auto-instrumentation and manual instrumentation
- Migrating between observability backends without re-instrumenting application code

## Core Knowledge

### The Three Signals

| Signal | Captures |
|------|------|
| **Traces** | Request flow across services, as a tree of spans |
| **Metrics** | Numeric measurements over time (counters, histograms, gauges) |
| **Logs** | Discrete timestamped events, increasingly correlated with trace context |

OpenTelemetry's value proposition is a single, vendor-neutral way to produce all three, so switching observability backends (Datadog, Honeycomb, Grafana, New Relic) doesn't require re-instrumenting application code — only reconfiguring the exporter.

### API vs. SDK

- Application code instruments against the **API** (stable interface for creating spans/metrics)
- The **SDK** (configured separately, usually at process startup) decides how that data is actually processed and exported
- This split is what makes libraries safe to instrument — a library can depend on the OTel API without forcing a specific SDK/backend choice on whoever uses it

### Context Propagation

- Trace context (trace ID, span ID, sampling decision) propagates via headers using the **W3C Trace Context** standard (`traceparent`/`tracestate`) for HTTP, and analogous mechanisms for other transports
- Context propagation across **non-HTTP boundaries** — message queues, background job systems, async callbacks — has to be wired manually by attaching context to the message/job payload and extracting it on the other side; nothing does this automatically outside the HTTP path

### The Collector

- The **OpenTelemetry Collector** is a standalone pipeline: **receivers** (ingest data, e.g. OTLP, Jaeger, Prometheus), **processors** (batch, filter, transform, sample), **exporters** (send to one or more backends)
- Running data through a Collector (rather than exporting directly from every service) centralizes sampling, filtering, and backend routing — but a misconfigured pipeline (wrong processor order, missing batching) affects every service sending data through it at once

### Auto-Instrumentation vs. Manual

| Approach | Captures | Misses |
|------|------|------|
| **Auto-instrumentation** | Framework-level spans (HTTP handlers, DB calls, common libraries) automatically | Business-logic-specific context (a span attribute for "which tenant," "which feature flag") |
| **Manual instrumentation** | Whatever the developer explicitly adds | Requires ongoing developer effort to keep current with code changes |

Most production setups use both — auto-instrumentation for baseline coverage, manual instrumentation layered in for the spans and attributes that actually matter during an investigation.

## Best Practices

1. **Propagate context across every boundary**, not just HTTP — queues, background jobs, and async callbacks need explicit context attachment/extraction
2. **Adopt semantic conventions consistently** across services/teams (attribute naming, span naming) so cross-service correlation actually works
3. **Layer manual instrumentation on top of auto-instrumentation** for business-logic context that auto-instrumentation can't infer
4. **Centralize sampling decisions** (ideally at the Collector, using consistent/coordinated sampling) rather than each service deciding independently
5. **Test the Collector pipeline configuration** before relying on it — a wrong processor order can silently drop or malform data
6. **Treat the API/SDK split seriously in libraries** — depend on the API only, let the consuming application own SDK configuration

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Trace context not propagated through a message queue | Explicitly attach/extract context on message payloads at both publish and consume |
| Each service picking its own sampling rate independently | Coordinate sampling (head-based or tail-based at the Collector) so traces aren't inconsistently sampled across services |
| Relying solely on auto-instrumentation | Layer in manual spans/attributes for the business context auto-instrumentation can't know about |
| Inconsistent attribute naming across teams (`user_id` vs `userId` vs `uid`) | Adopt and enforce OpenTelemetry semantic conventions org-wide |
| Collector pipeline changes deployed untested | Validate Collector config changes in a staging pipeline before touching the production data path |

## Sharp Edges

### SE-1: Context Propagation Gaps Across Async Boundaries
- **Severity**: high
- **Situation**: A trace looks complete and correct for the synchronous HTTP portion of a request, then appears to terminate the moment a message is published to a queue — the consumer's processing shows up as an entirely disconnected trace
- **Cause**: W3C Trace Context propagation is automatic for instrumented HTTP clients/servers, but message queues, background job frameworks, and other non-HTTP transports require the application to manually attach the context to the outgoing message and extract it when consuming — nothing does this by default
- **Symptoms**:
  - Traces for async workflows show as multiple disconnected traces instead of one continuous trace
  - Root-cause investigation across an async boundary requires manually correlating timestamps/IDs instead of following a single trace
- **Solution**: Explicitly inject trace context into message headers/attributes at publish time and extract it at consume time for every async transport in use, and verify end-to-end trace continuity for critical async flows as part of instrumentation review
- **Details**: → [extended/checklists.md#instrumentation-checklist]

### SE-2: Inconsistent Sampling Producing Incomplete Traces
- **Severity**: high
- **Situation**: Different services in the same call chain apply different, uncoordinated sampling rates, and a trace ends up with some services' spans present and others missing — producing a trace that looks broken even though nothing actually failed
- **Cause**: Head-based sampling decisions made independently per-service (rather than propagated as part of the trace context, or coordinated via tail-based sampling at a Collector) can disagree about whether a given trace should be kept
- **Symptoms**:
  - Traces show gaps between services that should logically be connected, with no error to explain the gap
  - Sample rate audits reveal each team configured sampling independently with no shared strategy
- **Solution**: Propagate the sampling decision as part of the trace context (so downstream services respect the upstream decision) or move to tail-based sampling at the Collector, which can see the whole trace before deciding whether to keep it

### SE-3: Collector Pipeline Misconfiguration Dropping or Delaying Data
- **Severity**: high
- **Situation**: A change to the Collector's processor chain (wrong ordering, a misconfigured filter, missing batch processor) causes silent data loss or a significant increase in backend load, affecting every service sending data through that Collector at once
- **Cause**: The Collector's receivers → processors → exporters pipeline executes in the configured order, and processors can filter, drop, or transform data — a misconfiguration doesn't error loudly, it just changes what data makes it through
- **Symptoms**:
  - A sudden drop in trace/metric volume at the backend with no corresponding drop in actual service traffic
  - Backend ingestion cost or load spikes unexpectedly after a Collector config change
- **Solution**: Validate Collector configuration changes in a non-production pipeline first, monitor the Collector's own internal telemetry (it can emit metrics about its own pipeline health) as a standard operational signal, and review processor ordering carefully since some processors depend on running before/after others

### SE-4: Auto-Instrumentation Missing Business-Critical Context
- **Severity**: medium
- **Situation**: A production incident investigation needs to know "which tenant" or "which feature flag variant" was involved in a slow/failing request, but auto-instrumentation only captured generic framework-level spans (HTTP method, route, DB query) with none of that business context attached
- **Cause**: Auto-instrumentation operates at the framework/library level and has no visibility into application-specific concepts — it can't know what a "tenant" or "feature flag" means to a given codebase
- **Symptoms**:
  - Investigations repeatedly hit a wall where the trace shows *that* something was slow/failed but not the business context needed to explain *why* for that specific case
- **Solution**: Layer manual instrumentation (custom span attributes) on top of auto-instrumentation specifically for the business dimensions that recur in investigations — tenant ID, feature flag state, user segment — treating this as an ongoing part of feature development, not a one-time setup task

### SE-5: Semantic Convention Drift Undermining Cross-Service Correlation
- **Severity**: medium
- **Situation**: Different teams instrument their services with inconsistent attribute names for conceptually identical data (`http.status_code` in one service, `status` in another, `httpStatusCode` in a third), and queries/dashboards that try to correlate across services silently miss data from services using a different naming convention
- **Cause**: OpenTelemetry defines semantic conventions precisely to prevent this, but nothing enforces their use — a team unaware of or ignoring the conventions produces technically-valid-but-inconsistent telemetry
- **Symptoms**:
  - Cross-service dashboards/queries return incomplete results with no error, because the query only matches one naming variant
  - Onboarding a new service reveals it doesn't "just work" with existing dashboards despite using OpenTelemetry correctly in isolation
- **Solution**: Adopt and document the OpenTelemetry semantic conventions as an org-wide standard, enforce via code review or linting where feasible, and audit existing services periodically for drift

## Recommended Tools

| Category | Tools |
|------|------|
| Collector | OpenTelemetry Collector (contrib and core distributions) |
| SDKs | Language-specific OTel SDKs, auto-instrumentation agents |
| Backends | Any OTLP-compatible backend (Honeycomb, Grafana Tempo, Datadog, New Relic, Jaeger) |
| Validation | `otel-cli`, Collector's own internal telemetry/health metrics |

## Instrumentation Quality

**Full checklist**: → [extended/checklists.md#instrumentation-checklist]

## Related Resources

[OpenTelemetry Docs](https://opentelemetry.io/docs/) | [Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/) | [Collector Docs](https://opentelemetry.io/docs/collector/)

## Related Domains

[[honeycomb]] | [[datadog]] | [[new-relic]] | [[grafana-prometheus]]
