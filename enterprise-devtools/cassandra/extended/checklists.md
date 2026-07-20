# Apache Cassandra — Extended Checklists

## Data Modeling Checklist

- [ ] Application query patterns enumerated before any table is designed
- [ ] One denormalized table created per distinct query pattern — no reliance on joins
- [ ] Partition key chosen for even cardinality/distribution across expected production traffic
- [ ] Partitions bucketed (e.g., by time window) where the clustering key could otherwise grow unbounded
- [ ] Consistency levels chosen deliberately (`QUORUM`/`QUORUM` as the default, `W + R > N` verified) for any read-your-write requirement
- [ ] Compaction strategy matched to workload (Time-Window for TTL'd time-series, Leveled for read-heavy, Size-Tiered as general default)
- [ ] Delete-heavy access patterns reviewed for tombstone impact; TTLs used in place of explicit deletes where feasible

## Operational Health Checklist

- [ ] `nodetool status`/`nodetool tpstats` reviewed for node-level load imbalance (a sign of a hot partition)
- [ ] Tombstone counts monitored per table, with alerting before hitting the tombstone failure threshold
- [ ] Repair scheduled regularly (Cassandra Reaper or equivalent) to keep replicas consistent
- [ ] Large partition warnings in logs/metrics reviewed and root-caused, not ignored
- [ ] Replication factor and multi-datacenter topology reviewed against actual availability requirements
- [ ] Compaction throughput and pending compaction backlog monitored under write-heavy load
- [ ] Backup/snapshot process tested for actual restore capability, not just configured
