# New Relic — Extended Checklists

## Cost and Ingest Checklist

- [ ] Data Explorer reviewed regularly to identify which event/data types dominate ingest volume
- [ ] Custom events/attributes audited for actual query/debugging value; unused ones removed
- [ ] High-cardinality attributes (user ID, session ID, request ID) not indexed on every transaction by default
- [ ] Sampling configured for high-volume debug-only custom events
- [ ] Ingest growth tracked against traffic growth — disproportionate growth investigated promptly
- [ ] Log ingestion filtered at the source (verbose DEBUG logs not shipped by default)

## Alerting and Tracing Checklist

- [ ] Alert thresholds derived from historical baselines, not a single observed incident
- [ ] Baseline/anomaly alert types used for metrics with natural variance instead of static thresholds
- [ ] Alert evaluation windows matched to how quickly the underlying metric actually changes
- [ ] Alert-to-incident correlation reviewed periodically; consistently noisy conditions pruned or retuned
- [ ] Trace context propagation verified across every async boundary (queues, background jobs) in critical flows
- [ ] Sampling rules configured to always capture error traces, not just a uniform default sample rate
- [ ] Dashboard `FACET` queries checked for cardinality — bounded fields only on frequently-refreshed panels
