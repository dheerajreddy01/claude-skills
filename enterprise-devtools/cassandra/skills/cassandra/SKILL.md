---
schema: "1.0"
name: cassandra
version: "1.0.0"
description: Apache Cassandra query-first data modeling, tunable consistency, and ring architecture practices
domain: technology
triggers:
  keywords:
    primary: [cassandra, wide column, ring, gossip protocol]
    secondary: [consistency level, compaction, CQL, partition key]
  context_boost: [distributed database, high write throughput, multi-region]
  context_penalty: [postgresql, mysql, relational]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Apache Cassandra

> Design the table around the query, not the query around the table

## Applicable Scenarios

- Designing Cassandra table schema and partition/clustering keys
- Choosing consistency levels for reads and writes
- Diagnosing hot partitions, tombstone buildup, or read latency
- Planning for multi-datacenter/multi-region replication
- Reviewing compaction strategy for a given workload

## Core Knowledge

### Query-First Data Modeling

- Cassandra has no joins and discourages ad-hoc queries — the standard workflow is: enumerate your application's actual query patterns *first*, then design one denormalized table per query pattern
- This inverts the relational instinct to normalize first and query later; a Cassandra schema with multiple tables holding duplicated data (one per access pattern) is normal and expected, not a smell

### Partitioning & the Ring

- Data is distributed across nodes by hashing the **partition key** — each node (via **gossip protocol**, a peer-to-peer membership/state-sharing mechanism) owns a range of the hash ring
- The **clustering key** determines sort order *within* a partition — a table's primary key is `(partition key, clustering key...)`, and this combination is what determines both distribution (partition key) and in-partition ordering (clustering key)
- All rows sharing a partition key live together on the same replica set — this is what makes single-partition reads fast, and what makes partition key choice the single most consequential schema decision

### Tunable Consistency

| Consistency Level | Meaning |
|------|------|
| **ONE** | Acknowledged/read from a single replica — fastest, weakest consistency |
| **QUORUM** | A majority of replicas — balances consistency and availability |
| **ALL** | Every replica — strongest consistency, least available (any replica down blocks the operation) |

Cassandra's consistency is tunable *per query*, not fixed cluster-wide — writing at `ONE` and reading at `ONE` gives no read-your-write guarantee unless you happen to read from the replica you wrote to; `W + R > N` (write consistency + read consistency > replication factor) is the standard rule of thumb for strong consistency.

### Compaction

- Writes (including deletes, which are actually special **tombstone** markers) accumulate in SSTables (immutable on-disk files); **compaction** merges SSTables over time, removing overwritten/deleted data
- Different compaction strategies (Size-Tiered, Leveled, Time-Window) suit different workloads — Time-Window Compaction is standard for time-series data with TTL-based expiry, for example

## Best Practices

1. **Design tables around query patterns**, one table per distinct access pattern, denormalizing freely
2. **Choose partition keys for even distribution** across the expected data/traffic profile — avoid low-cardinality or skewed keys
3. **Keep partitions bounded** — an ever-growing clustering key (e.g., appending forever) eventually creates an oversized partition that degrades performance
4. **Use `QUORUM` consistency for both reads and writes** by default unless a specific use case justifies weaker or stronger levels
5. **Choose a compaction strategy matched to the workload** — Time-Window for TTL'd time-series data, Leveled for read-heavy workloads, Size-Tiered as the general-purpose default
6. **Minimize deletes and frequent overwrites on hot rows** where tombstone accumulation would be a concern — model with TTLs or bucketing instead where possible

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Designing a normalized schema and relying on joins | Denormalize into one table per query pattern — Cassandra has no joins |
| Choosing a low-cardinality partition key (e.g., a status flag) | Choose a key with enough cardinality to distribute evenly across the cluster |
| Writing at `ONE` and reading at `ONE` from different replicas, expecting consistency | Use `QUORUM` for both, or ensure `W + R > N` for the desired guarantee |
| An ever-growing clustering key with no bound (e.g., appending forever without bucketing) | Bucket partitions (e.g., by time window) to keep them bounded |
| Frequent deletes/updates on the same rows without accounting for tombstones | Model around TTLs and immutable/append-style writes where possible; tune `gc_grace_seconds` and compaction deliberately if deletes are unavoidable |

## Sharp Edges

### SE-1: Normalized Schema Design Requiring Joins Cassandra Can't Do
- **Severity**: critical
- **Situation**: A team designs Cassandra tables the way they'd design a relational schema — normalized, with foreign-key-like relationships — and discovers post-launch that common queries require joining across tables, which Cassandra doesn't support
- **Cause**: Cassandra's data model is built around single-partition reads being fast and multi-partition/cross-table joins not existing at all — a normalized schema assumes exactly the query flexibility Cassandra doesn't provide
- **Symptoms**:
  - Application code ends up doing "joins" manually via multiple sequential queries and in-memory merging, which is slow and doesn't scale
  - A schema redesign is needed after the fact once real query patterns are understood
- **Solution**: Do query-first modeling from day one — enumerate actual application queries before writing any `CREATE TABLE`, and design one denormalized table per query pattern, accepting data duplication as the normal cost of this model
- **Details**: → [extended/checklists.md#data-modeling-checklist]

### SE-2: Hot Partitions From Poor Partition Key Choice
- **Severity**: high
- **Situation**: A partition key with low cardinality or a skewed real-world distribution (e.g., partitioning by `tenant_id` where one tenant dominates traffic) concentrates reads/writes onto a small number of nodes, and those nodes become bottlenecks while the rest of the cluster is comparatively idle
- **Cause**: Cassandra distributes load by hashing the partition key across the ring — if the actual key value distribution in production traffic is skewed, the resulting node load is skewed too, regardless of cluster size
- **Symptoms**:
  - A small number of nodes show consistently higher CPU/latency than the rest of the cluster
  - Overall cluster throughput is bottlenecked by specific nodes rather than scaling with node count
- **Solution**: Choose partition keys with enough cardinality and even enough real-world distribution to spread load, or use a composite/salted key for known-skewed dimensions (accepting the tradeoff of needing to query across salted buckets)

### SE-3: Consistency Level Misunderstanding Causing Read-Your-Write Failures
- **Severity**: high
- **Situation**: An application writes at consistency level `ONE`, immediately reads at consistency level `ONE`, and doesn't see the value it just wrote — because the read happened to hit a different replica than the write
- **Cause**: `ONE` consistency for both operations gives no guarantee that the read and write touch the same replica; without `W + R > N`, there's no mathematical guarantee of overlap between the replica set acknowledging the write and the replica set serving the read
- **Symptoms**:
  - Data a user just submitted appears to be missing, then shows up on a subsequent request
  - The issue is intermittent and correlates with which replica happens to serve each request
- **Solution**: Use `QUORUM` for both reads and writes (satisfying `W + R > N` for any standard replication factor), or explicitly design for eventual consistency in the application if `ONE`/`ONE` is a deliberate tradeoff for latency

### SE-4: Tombstone Accumulation Degrading Read Performance
- **Severity**: high
- **Situation**: A table with frequent deletes or overwrites (each of which creates a tombstone marker) accumulates so many tombstones that reads over that partition become slow, sometimes triggering a `TombstoneOverwhelmingException` and failing outright
- **Cause**: Cassandra deletes are not immediate removals — they're tombstone markers that persist until compaction and `gc_grace_seconds` allow their actual removal; a read has to scan past all live tombstones to assemble the current state of a partition
- **Symptoms**:
  - Read latency on specific partitions grows over time correlated with delete/update frequency, not partition size alone
  - Cassandra logs warn about a high tombstone-to-live-cell ratio, or reads fail with a tombstone threshold exception
- **Solution**: Model around TTLs and append-only/immutable writes where the use case allows it instead of frequent deletes, tune compaction strategy and `gc_grace_seconds` deliberately for tables that do need deletes, and monitor tombstone counts as a standard operational metric for delete-heavy tables

### SE-5: Unbounded Partition Growth From an Ever-Growing Clustering Key
- **Severity**: medium
- **Situation**: A table's clustering key grows without bound (e.g., appending a new row per event indefinitely under the same partition key), and the partition eventually becomes so large it degrades read/write performance and repair operations for that partition
- **Cause**: All rows under one partition key live together on the same replica set; there's a practical (not always hard-enforced) size ceiling beyond which large partitions cause real operational problems — compaction, repair, and read latency all degrade
- **Symptoms**:
  - A specific partition (e.g., a very active user or long-running entity) shows disproportionately worse latency than others
  - Cluster warnings about large partition sizes in logs/metrics
- **Solution**: Bucket partitions by a bounded dimension (commonly time-based, e.g., `(entity_id, date_bucket)` as a compound partition key) so no single partition grows without bound, deciding the bucket granularity based on expected write volume per entity

## Recommended Tools

| Category | Tools |
|------|------|
| Query language | CQL (Cassandra Query Language) |
| Monitoring | `nodetool`, Cassandra Reaper (repair scheduling) |
| Data modeling | Query-first modeling worksheets, DataStax data modeling guides |
| Drivers | DataStax drivers per language |

## Data Modeling

**Full checklist**: → [extended/checklists.md#data-modeling-checklist]

## Related Resources

[Cassandra Docs](https://cassandra.apache.org/doc/latest/) | [DataStax Data Modeling Guide](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dataModeling/intro.html) | [Cassandra: The Definitive Guide (O'Reilly)](https://www.oreilly.com/library/view/cassandra-the-definitive/9781098115160/)

## Related Domains

[[dynamodb]] | [[kafka]] | [[mongodb]] | [[java]]
