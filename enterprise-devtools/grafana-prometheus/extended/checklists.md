# Grafana + Prometheus — Extended Checklists

## Cardinality and Reliability Checklist

- [ ] No metric labeled with an unbounded identifier (user ID, raw request path, ephemeral pod name at high churn)
- [ ] `prometheus_tsdb_head_series` monitored with an alert on abnormal growth
- [ ] `sample_limit` set on scrape configs to catch a runaway cardinality target before it takes down Prometheus
- [ ] All counters graphed/alerted via `rate()`/`increase()`, never raw
- [ ] Histogram quantiles computed via `histogram_quantile(rate(..._bucket[...]))`, not approximated from raw bucket counts
- [ ] Prometheus runs as an HA pair (or equivalent) for any system where an alerting gap is unacceptable
- [ ] Remote-write configured to Thanos/Mimir/Cortex for durable, long-term storage beyond local retention
- [ ] Recording rules used for expensive PromQL queries that back frequently-loaded dashboards

## Alerting and Dashboard Checklist

- [ ] Alertmanager routing tree has an explicit catch-all/default route
- [ ] Routing tree tested with `amtool` (or equivalent) against realistic label sets, not just the happy path
- [ ] Alert grouping configured to batch related alerts instead of one notification per series
- [ ] Every Grafana panel's query references dashboard template variables — no hardcoded label values inconsistent with the selector
- [ ] Dashboards provisioned as code (JSON/Grafonnet) where consistency across panels matters, rather than manual per-panel editing
- [ ] Scrape intervals consistent across targets used in the same cross-service rate comparison
