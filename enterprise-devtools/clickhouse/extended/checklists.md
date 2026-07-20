# ClickHouse — Extended Checklists

## Schema Design Checklist

- [ ] Primary/ordering key chosen based on the most common query filter/sort columns, understood as a sparse index, not a uniqueness constraint
- [ ] `ReplacingMergeTree`/`CollapsingMergeTree` used deliberately (with async deduplication semantics understood) if upsert-like behavior is needed
- [ ] Partition key is coarse and lifecycle-relevant (commonly day/month), not fine-grained or high-cardinality
- [ ] Materialized views backfilled explicitly for historical data if created after the source table already has rows
- [ ] Aggregate materialized views use `AggregatingMergeTree` with proper `-State`/`-Merge` combinators where incremental correctness matters
- [ ] Frequently-updated/mutable transactional data identified and routed to an OLTP-appropriate database instead of ClickHouse
- [ ] Queries reviewed for `SELECT *` usage on wide tables and narrowed to actually-needed columns

## Operational Health Checklist

- [ ] `system.parts` part count monitored per table/partition, with alerting on abnormal growth ("too many parts")
- [ ] Mutation queue (`system.mutations`) monitored for backlog if any mutations are in regular use
- [ ] Query performance reviewed via `system.query_log` for queries reading disproportionate column volume
- [ ] Replication (if using `ReplicatedMergeTree`) and ClickHouse Keeper health monitored
- [ ] TTL-based data expiration configured where retention policy allows, to bound storage growth automatically
- [ ] Backup strategy (`clickhouse-backup` or cloud-native snapshotting) tested for actual restore capability
