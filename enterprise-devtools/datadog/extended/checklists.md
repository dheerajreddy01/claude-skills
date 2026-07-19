# Datadog — Extended Checklists

## Cost and Tagging Checklist

- [ ] Custom metric tags audited for cardinality; no unbounded identifiers (user ID, request ID) used as metric tags
- [ ] Unified service tagging (`env`, `service`, `version`) applied consistently across agent, APM libraries, and log pipelines
- [ ] A known trace verified to correctly surface related logs and infra metrics (correlation smoke test)
- [ ] Log exclusion filters/pipelines route low-value logs to archive tier instead of full indexing
- [ ] Indexed log volume reviewed in the Usage tab; dominant noisy sources identified and filtered
- [ ] Custom metrics time series count (Metrics Summary) reviewed for cardinality outliers
- [ ] APM host/container billing count reviewed against actual steady-state fleet size, especially in high-churn environments
- [ ] Usage & Cost pages reviewed on a recurring cadence, not just when a bill surprise happens

## Monitor Hygiene Checklist

- [ ] Every metric monitor has an explicit evaluation window / "require N of M" setting, not single-sample alerting
- [ ] Anomaly detection used for metrics with natural seasonal variance instead of static thresholds
- [ ] Alerts grouped by tag (one monitor, multiple groups) instead of duplicated per host/instance
- [ ] Alert-to-incident correlation reviewed periodically; consistently flapping monitors retuned or removed
- [ ] Notification routing (who gets paged for what severity) reviewed against current on-call structure
