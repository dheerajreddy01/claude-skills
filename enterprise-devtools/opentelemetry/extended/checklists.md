# OpenTelemetry — Extended Checklists

## Instrumentation Checklist

- [ ] Trace context explicitly propagated across every non-HTTP boundary (message queues, background jobs, async callbacks)
- [ ] A known end-to-end async flow verified to produce one continuous trace, not disconnected fragments
- [ ] Sampling decisions coordinated across services (propagated head-based decision, or tail-based sampling at the Collector) rather than each service sampling independently
- [ ] Semantic conventions documented and enforced org-wide for span/attribute naming
- [ ] Manual instrumentation layered on top of auto-instrumentation for business-critical context (tenant, feature flag, user segment)
- [ ] Library code instruments against the OTel API only, leaving SDK configuration to the consuming application
- [ ] Cross-service dashboards/queries spot-checked against multiple services to confirm consistent attribute naming actually correlates data

## Collector Pipeline Checklist

- [ ] Collector configuration changes validated in a non-production pipeline before deploying to production
- [ ] Processor ordering reviewed for correctness (e.g., batching, filtering, and sampling processors in the right sequence)
- [ ] Collector's own internal telemetry/health metrics monitored as a standard operational signal
- [ ] Backend ingestion volume monitored for unexpected drops (data loss) or spikes (misconfigured sampling/filtering) after pipeline changes
- [ ] Exporter failure/retry behavior understood and monitored, so a backend outage doesn't silently drop data indefinitely
- [ ] Pipeline capacity (memory, queue limits) sized for peak telemetry volume, not just average load
