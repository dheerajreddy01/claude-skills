---
schema: "1.0"
name: clickhouse
version: "1.0.0"
description: ClickHouse columnar OLAP design, MergeTree engine mechanics, and query performance practices
domain: technology
triggers:
  keywords:
    primary: [clickhouse, OLAP, columnar database, MergeTree]
    secondary: [materialized view, partition key, sparse index, part merge]
  context_boost: [analytics, data warehouse, high-volume aggregation]
  context_penalty: [postgresql, mysql, OLTP]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# ClickHouse

> A primary key that doesn't enforce uniqueness — the single biggest mental model shift coming from a relational database

## Applicable Scenarios

- Designing ClickHouse table schema for analytical (OLAP) workloads
- Choosing MergeTree engine variant, partition key, and ordering key
- Building materialized views for pre-aggregation
- Diagnosing slow queries or excessive small-part accumulation
- Deciding whether ClickHouse is the right fit for a given workload

## Core Knowledge

### Columnar Storage for OLAP

- ClickHouse stores data column-by-column rather than row-by-row — a query that touches 3 of a table's 50 columns only reads those 3 columns from disk, which is what makes wide-table analytical aggregations so much faster than in a row-oriented database
- This tradeoff cuts the other way for OLTP-style access: reading/writing a single full row means touching every column's storage separately, which is comparatively inefficient

### MergeTree Engine Family

- The **MergeTree** engine family is ClickHouse's standard table engine — data is written in small **parts**, which background merge processes periodically combine into larger parts for efficiency
- The **primary key** in MergeTree is a **sparse index** — it stores index entries for every Nth row (not every row), used to locate approximate ranges to scan, not to enforce uniqueness. **This is fundamentally different from a relational primary key**: ClickHouse will happily store multiple rows with the same "primary key" value unless you're using a variant engine (like `ReplacingMergeTree`) specifically designed to handle deduplication, and even then, deduplication happens asynchronously during merges, not immediately on insert
- **Partition key** determines how data is physically split into separate directories (commonly by date) — this is a data-management/lifecycle tool (drop old partitions cheaply), distinct from the primary key's role in query optimization

### Materialized Views

- A materialized view in ClickHouse is a trigger-like mechanism: it runs a query against newly inserted data (not the whole table) and writes the result into a target table — it's an incremental pre-aggregation tool, not a live view recomputed on read like in many relational databases
- Because it only processes newly inserted rows, a materialized view created *after* a table already has data does not retroactively backfill unless explicitly populated

### Why Not OLTP

- ClickHouse has no meaningful support for frequent single-row updates/deletes in the way a relational OLTP database does — updates/deletes are handled via asynchronous mutations that rewrite whole parts, which is expensive and not designed for high-frequency use
- The right mental model: ClickHouse is optimized for high-volume append-mostly ingestion and fast aggregation over huge datasets, not for transactional single-row read-modify-write patterns

## Best Practices

1. **Choose the ordering key (primary key) based on your most common query filter/sort columns**, understanding it's a sparse index for range-scanning, not a uniqueness constraint
2. **Partition by a bounded, lifecycle-relevant dimension** (commonly date) to enable cheap partition drops for data retention, not for query optimization
3. **Use materialized views for incremental pre-aggregation**, and explicitly backfill them if created after historical data already exists
4. **Avoid frequent single-row updates/deletes** — model as append-only/immutable where possible, and use `ReplacingMergeTree`/`CollapsingMergeTree` deliberately (understanding their async deduplication semantics) if upsert-like behavior is needed
5. **Select only the columns you need** in queries — the columnar advantage disappears if every query does `SELECT *`
6. **Monitor part count** and avoid overly granular partition keys that create too many small parts

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming the primary key enforces uniqueness like a relational database | Understand it's a sparse index for scan optimization; use `ReplacingMergeTree` (with its async semantics) if deduplication is genuinely needed |
| Using ClickHouse for frequent single-row updates/deletes (OLTP-style) | Model as append-only/immutable, or route that workload to an OLTP-appropriate database instead |
| `SELECT *` over wide tables | Select only needed columns to realize the columnar storage advantage |
| Creating a materialized view after historical data exists and expecting it to be backfilled automatically | Explicitly backfill (`INSERT INTO target SELECT ... FROM source`) after creating the view |
| Overly granular partition key (e.g., partitioning by minute) | Partition by a coarser, lifecycle-relevant dimension (commonly by day or month) |

## Sharp Edges

### SE-1: Primary Key Doesn't Enforce Uniqueness
- **Severity**: critical
- **Situation**: An application inserts what it believes is an update to an existing row (same "primary key" value) and ends up with duplicate rows in the table, because ClickHouse's MergeTree engine doesn't reject or overwrite on a primary key match the way a relational database would
- **Cause**: MergeTree's primary key is a sparse index for query optimization, not a uniqueness constraint — nothing prevents multiple rows sharing the same key value from coexisting unless using a specific deduplication-oriented engine variant, and even then deduplication happens asynchronously during background merges, not at insert time
- **Symptoms**:
  - Aggregation queries return inflated counts/sums due to duplicate rows
  - A query expecting "one row per key" returns multiple rows for the same key value
- **Solution**: Understand this distinction explicitly when designing schema coming from a relational background, use `ReplacingMergeTree` if last-write-wins semantics are needed (querying with `FINAL` or explicit deduplication logic, since merges are async and not immediate), and design ingestion to avoid relying on ClickHouse to prevent duplicates on write
- **Details**: → [extended/checklists.md#schema-design-checklist]

### SE-2: Using ClickHouse for OLTP-Style Frequent Updates
- **Severity**: high
- **Situation**: An application uses ClickHouse for a workload with frequent single-row updates (e.g., a mutable "current status" field updated often per entity), and update/mutation operations become a major performance bottleneck and resource drain
- **Cause**: ClickHouse mutations (`ALTER TABLE ... UPDATE/DELETE`) are asynchronous, heavyweight operations that rewrite entire affected parts rather than modifying rows in place — they're designed for occasional bulk corrections, not high-frequency transactional updates
- **Symptoms**:
  - Mutation queue backs up, with mutations taking a long time to apply
  - Cluster resource usage (CPU, I/O) spikes disproportionately to actual data volume changed
- **Solution**: Recognize this workload mismatch at design time — route frequently-mutated transactional data to an OLTP-appropriate database (PostgreSQL/MySQL), and use ClickHouse for what it's built for: high-volume analytical aggregation over largely immutable/append-only data

### SE-3: `SELECT *` Defeating the Columnar Advantage
- **Severity**: medium
- **Situation**: Queries habitually use `SELECT *` on wide analytical tables, and query performance is far worse than ClickHouse's reputation would suggest, because every column gets read from disk regardless of what's actually needed
- **Cause**: ClickHouse's core performance advantage comes from reading only the columns a query touches — `SELECT *` forces it to read every column, which negates that advantage entirely and can make it perform no better (or worse) than a row-oriented database for that query
- **Symptoms**:
  - Query latency doesn't improve as expected compared to benchmarks/documentation showing ClickHouse's columnar advantage
  - Query profiling shows I/O dominated by columns the application never actually uses downstream
- **Solution**: Select only the specific columns needed for each query, and review any `SELECT *` usage (especially in application ORMs/query builders that default to it) as a performance red flag on wide tables

### SE-4: Incorrect Materialized View Design Producing Wrong Aggregations
- **Severity**: medium
- **Situation**: A materialized view intended to maintain a running aggregate (e.g., daily counts) produces incorrect results because it was created after historical data already existed, or because its underlying aggregate function/state handling doesn't match ClickHouse's incremental execution model
- **Cause**: A ClickHouse materialized view only processes rows inserted *after* its creation — it doesn't retroactively process existing data — and some aggregate functions require using `AggregatingMergeTree`-style state functions (`-State`/`-Merge` combinators) to combine correctly across multiple insert batches rather than naive re-aggregation
- **Symptoms**:
  - A materialized view's totals don't match a direct `SELECT ... GROUP BY` over the full underlying table
  - Numbers are correct going forward but missing historical data from before the view was created
- **Solution**: Explicitly backfill a materialized view's target table with historical data at creation time, and use `AggregatingMergeTree` with proper `-State`/`-Merge` function combinators for aggregates that need to combine correctly across incremental insert batches

### SE-5: Partition Key Granularity Causing Part Explosion
- **Severity**: medium
- **Situation**: An overly granular partition key (e.g., partitioning by exact timestamp or a high-cardinality dimension) causes ClickHouse to create a very large number of small parts, and background merge processes can't keep up — degrading both insert and query performance
- **Cause**: Each partition maintains its own set of parts that merge independently; a partition key with too many distinct values fragments data into many small partitions/parts instead of a manageable number of larger ones, increasing merge overhead and per-query part-scanning overhead
- **Symptoms**:
  - `system.parts` shows a very high part count relative to total data volume
  - Insert performance degrades over time, and ClickHouse logs warnings about "too many parts"
- **Solution**: Choose a coarser partition key (commonly by day or month, not by finer time granularity or a high-cardinality field), reserving the ordering/primary key — not the partition key — for fine-grained query optimization

## Recommended Tools

| Category | Tools |
|------|------|
| Query analysis | `EXPLAIN`, `system.query_log`, `clickhouse-benchmark` |
| Ingestion | Kafka table engine, `clickhouse-client`, native protocol drivers |
| Visualization | Grafana (ClickHouse data source), Superset |
| Cluster management | ClickHouse Keeper (replacing ZooKeeper), `system.parts`/`system.merges` |

## Schema Design

**Full checklist**: → [extended/checklists.md#schema-design-checklist]

## Related Resources

[ClickHouse Docs](https://clickhouse.com/docs) | [ClickHouse MergeTree Engine](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree) | [ClickHouse Academy](https://learn.clickhouse.com/)

## Related Domains

[[snowflake]] | [[kafka]] | [[postgresql]] | [[grafana-prometheus]]
