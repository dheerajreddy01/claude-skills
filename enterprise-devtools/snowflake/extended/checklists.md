# Snowflake — Extended Checklists

## Cost Management Checklist

- [ ] Auto-suspend configured aggressively (commonly 60-300 seconds) on every warehouse unless a documented latency need justifies keeping it warm
- [ ] Warehouse size right-sized per workload, starting small and scaling based on observed query latency/queueing
- [ ] Separate warehouses used per workload type (ETL, BI, ad hoc) instead of one warehouse serving everything
- [ ] Resource Monitors configured with credit quota alerts/suspension actions at the account and warehouse level
- [ ] Time Travel retention set per table based on actual recovery needs, not a blanket maximum
- [ ] Storage cost reviewed periodically by table to catch high-churn tables with unnecessarily long retention
- [ ] Zero-copy clones tracked with an expected lifecycle; stale/unused clones cleaned up
- [ ] Recurring expensive queries evaluated for materialized views or scheduled aggregation instead of repeated full scans
- [ ] `ACCOUNT_USAGE`/`ORGANIZATION_USAGE` views reviewed regularly to attribute credit consumption to actual teams/workloads

## Query & Data Design Checklist

- [ ] Query filters target clustered/pruning-friendly columns where possible to maximize micro-partition pruning
- [ ] Clustering keys evaluated for very large tables where natural ingestion order doesn't match query patterns
- [ ] `EXPLAIN`/Query Profile used to check scan size before running exploratory queries on large tables
- [ ] Query result cache benefit understood — recurring queries kept as identical as possible to actually hit cache
- [ ] Semi-structured data (VARIANT columns) usage reviewed for query performance impact vs. flattened schema alternatives
