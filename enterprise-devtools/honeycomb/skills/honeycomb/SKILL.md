---
schema: "1.0"
name: honeycomb
version: "1.0.0"
description: Honeycomb high-cardinality event instrumentation, BubbleUp investigation, and sampling practices
domain: technology
triggers:
  keywords:
    primary: [honeycomb, observability, trace, high cardinality]
    secondary: [bubbleup, event, span, sampling strategy]
  context_boost: [distributed tracing, production debugging, incident investigation]
  context_penalty: [datadog, grafana-prometheus, splunk]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Honeycomb

> The value is in the raw, high-cardinality events — pre-aggregating before you send them throws away the thing that makes it useful

## Applicable Scenarios

- Instrumenting services with rich, high-cardinality event data for debugging
- Using BubbleUp to investigate an anomaly and find what's different about it
- Designing sampling strategy for high-volume services
- Propagating trace context across service boundaries for connected traces
- Deciding what fields are actually worth attaching to an event

## Core Knowledge

### High-Cardinality, High-Dimensionality Events

- Honeycomb's core model is the **wide event** — a single structured event per unit of work (a request, a span) carrying as many fields as are useful (user ID, build version, shopping cart size, feature flags active, query parameters), queried at read time rather than pre-aggregated at write time
- This is a deliberate contrast to traditional metrics systems, which require deciding what to aggregate *before* an incident happens — Honeycomb's value proposition is being able to ask a question you didn't anticipate, *after* the fact, using whatever fields you captured
- **High cardinality** (many unique values, like user ID or request ID) and **high dimensionality** (many different fields per event) are both explicitly supported and central to the value — this is the opposite of the "avoid high-cardinality tags" advice common in traditional metrics/monitoring tools

### BubbleUp

- **BubbleUp** is Honeycomb's mechanism for automated anomaly investigation: select an area of interest in a visualization (e.g., a latency spike), and it automatically surfaces which field values are disproportionately common in that selection versus the baseline
- BubbleUp's usefulness is entirely bounded by what fields were actually captured on the events — it can't surface a dimension that was never instrumented, no matter how relevant that dimension would have been to the investigation

### Tracing

- A **trace** connects spans across service boundaries into one coherent view of a request's full path — this requires trace context (trace ID, span ID) to be explicitly propagated through every hop, including asynchronous ones (queues, background jobs)
- A broken trace (a hop that doesn't propagate context) looks like the trace simply ends at that point, with the continuation showing up as an unconnected, separate trace — easy to miss unless someone is specifically looking for connectivity gaps

### Sampling

- At high event volume, sending every single event becomes cost-prohibitive — **sampling** reduces volume while trying to preserve statistical usefulness
- Naive uniform sampling (keep 1 in N events) is simple but risks losing rare, high-value events (a single error in a sea of successful requests) at the same rate as routine ones — deliberate strategies like keeping all errors/slow requests while sampling routine fast ones preserve more investigative value per byte sent

## Best Practices

1. **Instrument wide events with many useful fields**, not a minimal set — the fields you don't capture are the questions you won't be able to ask later
2. **Propagate trace context through every service boundary**, including async/queue-based hops, not just synchronous HTTP calls
3. **Use deliberate, non-uniform sampling** at scale — bias toward keeping errors, slow requests, and otherwise unusual events over routine successful ones
4. **Use BubbleUp as an investigation starting point**, not a replacement for understanding what fields are actually relevant to a given system
5. **Avoid pre-aggregating data before sending it** — send the raw, detailed event and let query-time aggregation do the summarizing
6. **Review field/event volume growth periodically** to catch unbounded cardinality growth before it becomes a cost surprise

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Sending minimal, sparse events with few fields | Instrument wide events with every field plausibly useful for future debugging |
| Pre-aggregating metrics before sending (defeating the point) | Send raw, per-request/per-span events and let Honeycomb's query layer aggregate at read time |
| Uniform sampling that treats errors the same as routine successes | Bias sampling toward keeping errors/slow requests, sample routine traffic more aggressively |
| Missing trace context propagation through queues/async work | Explicitly propagate trace/span IDs through every hop, synchronous or not |
| Treating BubbleUp as magic that surfaces anything relevant | Recognize BubbleUp can only surface fields that were actually captured — instrumentation quality bounds its usefulness |

## Sharp Edges

### SE-1: Under-Instrumentation Limiting What BubbleUp Can Surface
- **Severity**: high
- **Situation**: During an incident, BubbleUp is used to find what's different about the affected requests, but the events don't carry a field that would have immediately explained the anomaly (e.g., a specific feature flag, a client SDK version, a config value) — the investigation has to fall back to manual, slower methods
- **Cause**: BubbleUp can only compare fields that were actually captured on the events; an under-instrumented event schema bounds what any analysis — automated or manual — can discover after the fact
- **Symptoms**:
  - An incident retrospective identifies a root-cause dimension that "we could have found faster if we'd been capturing that field"
  - BubbleUp results consistently point at generic fields (status code, endpoint) without surfacing anything more specific
- **Solution**: Proactively instrument wide events with a generous set of fields (feature flags, config versions, client metadata, business-relevant context) even before a specific need is known, since the cost of an unused field is much lower than the cost of not having it during an incident
- **Details**: → [extended/checklists.md#instrumentation-checklist]

### SE-2: Over-Aggressive Sampling Losing Rare, Critical Events
- **Severity**: high
- **Situation**: A high-traffic service samples events uniformly at a low rate to control cost, and a rare but critical error — one that occurred only a handful of times — falls entirely outside the sampled fraction, leaving no record it happened at all
- **Cause**: Uniform sampling treats every event as equally likely to be dropped, regardless of whether it represents a routine success or a rare, high-value anomaly
- **Symptoms**:
  - A user-reported issue has no corresponding captured event in Honeycomb despite confirmed occurrence
  - Error rate calculated from sampled data doesn't match error counts reported by other systems
- **Solution**: Use deliberate, non-uniform sampling — keep all (or a much higher fraction of) errors and slow/anomalous requests, sample only routine successful traffic aggressively, so rare high-value events aren't lost at the same rate as routine ones

### SE-3: Treating Honeycomb Like a Pre-Aggregated Metrics Tool
- **Severity**: medium
- **Situation**: A team instruments Honeycomb by computing and sending pre-aggregated summary metrics (e.g., average latency per minute) instead of raw per-request events, and loses the ability to ask any question that wasn't anticipated at aggregation time — defeating the reason Honeycomb was chosen over a traditional metrics tool
- **Cause**: Pre-aggregating at write time (the traditional metrics model) discards the per-event detail that makes Honeycomb's read-time, arbitrary-dimension querying possible
- **Symptoms**:
  - Queries can only answer questions that match how data was pre-aggregated, not arbitrary new questions
  - BubbleUp and high-cardinality breakdowns aren't meaningfully usable because the underlying events don't carry per-request detail
- **Solution**: Send raw, per-unit-of-work events (one event per request/span) with full field detail, and let Honeycomb's query layer perform aggregation at query time — resist the instinct to pre-summarize before sending

### SE-4: Broken Trace Context Across Async Boundaries
- **Severity**: high
- **Situation**: A trace for a request that continues into an async job (queue-triggered processing, background task) appears to end at the point the message was published, with the actual async processing showing up as a disconnected, separate trace — making it hard to follow the full request lifecycle
- **Cause**: Trace context propagation requires explicit code to carry trace/span IDs across boundaries that aren't automatically instrumented (message queues, custom RPC, scheduled jobs) — nothing propagates it automatically through an arbitrary message payload
- **Symptoms**:
  - A trace that should span multiple services/async stages visibly terminates at a queue publish step
  - Root-causing an issue that spans sync and async work requires manually correlating timestamps/IDs across separate traces
- **Solution**: Manually propagate trace context through every async boundary (attach trace/span IDs to message payloads, extract and continue the trace on the consumer side), and verify end-to-end trace continuity for critical async flows specifically, not just the synchronous HTTP path

### SE-5: Cost Surprises From Unbounded High-Cardinality Field Growth
- **Severity**: medium
- **Situation**: A field intended to carry a bounded set of values (e.g., a status enum) actually receives unbounded, ever-growing unique values in practice (e.g., a raw user-generated string used as a field value), and event volume/cost grows disproportionately as a result
- **Cause**: While high cardinality is a supported and encouraged part of Honeycomb's model, cost still scales with actual event volume and field richness — an unintentionally unbounded field (as opposed to a deliberately high-cardinality one like user ID, which is genuinely useful) adds cost without adding proportional investigative value
- **Symptoms**:
  - Usage/cost dashboards show volume growth disproportionate to actual traffic growth
  - A specific field's unique-value count is far higher than expected for what it's supposed to represent
- **Solution**: Periodically review field cardinality growth, distinguishing deliberately high-cardinality fields (worth the cost) from accidentally unbounded ones (a bug or a design flaw in what's being captured), and fix the latter at the instrumentation source

## Recommended Tools

| Category | Tools |
|------|------|
| Instrumentation | OpenTelemetry (Honeycomb's recommended SDK path), Beelines (legacy) |
| Investigation | BubbleUp, Query Builder, Triggers (alerting) |
| Sampling | Refinery (Honeycomb's tail-based sampling proxy) |
| Correlation | Honeycomb Service Map |

## Instrumentation Quality

**Full checklist**: → [extended/checklists.md#instrumentation-checklist]

## Related Resources

[Honeycomb Documentation](https://docs.honeycomb.io/) | [Observability Engineering (O'Reilly, by Honeycomb authors)](https://www.honeycomb.io/observability-engineering-oreilly-book-download) | [OpenTelemetry Docs](https://opentelemetry.io/docs/)

## Related Domains

[[datadog]] | [[sentry]] | [[grafana-prometheus]] | [[kafka]]
