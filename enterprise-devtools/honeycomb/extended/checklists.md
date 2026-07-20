# Honeycomb — Extended Checklists

## Instrumentation Checklist

- [ ] Events instrumented as wide events with a generous set of fields, not a minimal subset
- [ ] Business-relevant context (feature flags, config/client versions, user/tenant identifiers) captured proactively, before a specific need arises
- [ ] Raw per-request/per-span events sent, not pre-aggregated summary metrics
- [ ] Trace context explicitly propagated through every async boundary (queues, background jobs), verified end-to-end for critical flows
- [ ] Field cardinality growth reviewed periodically to distinguish deliberate high-cardinality fields from accidentally unbounded ones
- [ ] New services instrumented via OpenTelemetry from the start, avoiding legacy/inconsistent instrumentation paths

## Sampling & Cost Checklist

- [ ] Sampling strategy biases toward keeping errors and slow/anomalous requests over routine successful traffic
- [ ] Sample rate reviewed and adjusted as traffic volume grows, not left at an initial default
- [ ] Tail-based sampling (e.g., via Refinery) considered for services where deciding "is this event interesting" requires seeing the full trace first
- [ ] Usage/cost dashboards reviewed regularly to catch disproportionate volume growth early
- [ ] Rare-but-critical event types verified to actually appear in captured data, not just assumed to be sampled in
