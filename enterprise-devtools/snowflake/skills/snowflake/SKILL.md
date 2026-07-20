---
schema: "1.0"
name: snowflake
version: "1.0.0"
description: Snowflake virtual warehouse cost management, micro-partitioning, time travel, and zero-copy cloning practices
domain: technology
triggers:
  keywords:
    primary: [snowflake, data warehouse, virtual warehouse, snowpipe]
    secondary: [time travel, zero-copy clone, credits, micro-partition]
  context_boost: [analytics, BI, ELT pipeline]
  context_penalty: [postgresql, mysql, OLTP]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Snowflake

> Compute and storage are billed separately — most cost surprises come from forgetting that compute keeps running

## Applicable Scenarios

- Sizing and managing virtual warehouses for cost efficiency
- Understanding micro-partitioning and automatic clustering behavior
- Using time travel and zero-copy cloning for recovery/dev workflows
- Diagnosing unexpected credit consumption
- Designing an ELT pipeline around Snowflake's architecture

## Core Knowledge

### Separation of Storage and Compute

- Storage (the data itself) and compute (**virtual warehouses**, which execute queries) are billed and scaled independently — this is Snowflake's core architectural difference from traditional data warehouses
- A virtual warehouse can be resized, suspended, or resumed without affecting the underlying data, and multiple warehouses can query the same data concurrently without contention

### Virtual Warehouses & Credits

- Warehouses are billed in **credits per second while running** (rounded to the nearest whole second in most cases, with per-warehouse-size credit rates) — a warehouse that's "on" but idle still burns credits
- **Auto-suspend** stops a warehouse after a period of inactivity; **auto-resume** starts it again on the next query — both should be configured deliberately, since a long or disabled auto-suspend window means paying for idle compute
- Warehouse **size** (X-Small through 6X-Large) determines compute power and credit rate — oversizing a warehouse for the actual query workload wastes credits without a proportional performance benefit for queries that aren't actually compute-bound

### Micro-Partitioning & Clustering

- Snowflake automatically divides table data into **micro-partitions** (contiguous units of storage, typically 50-500MB uncompressed) and maintains metadata (min/max values per column) for each — this is what enables **pruning**: skipping micro-partitions that can't contain matching rows for a given query's filters
- Unlike traditional databases, there's no manual index to create — clustering happens automatically based on natural data ingestion order, or can be explicitly influenced via a **clustering key** for very large tables where query patterns don't align with natural ingestion order

### Time Travel & Fail-Safe

| Feature | Purpose | Retention |
|------|------|------|
| **Time Travel** | Query/restore data as it existed at a past point, or recover dropped objects | Configurable, up to 90 days (Enterprise+) |
| **Fail-Safe** | Snowflake-managed recovery of last resort, not user-accessible | Fixed 7 days after Time Travel window ends |

Longer Time Travel retention on high-churn tables means Snowflake retains more historical micro-partition versions, which directly increases storage cost — this is a cost lever, not a free safety net.

### Zero-Copy Cloning

- `CREATE TABLE ... CLONE` creates an instant, metadata-only copy of a table/schema/database — no data is physically duplicated at clone time, making it extremely fast and (initially) storage-free
- As the clone and the original diverge (either gets modified), storage cost accrues for the *changed* micro-partitions only — a clone isn't "free forever," it's free until data starts changing

## Best Practices

1. **Configure auto-suspend aggressively** (often 60-300 seconds) unless a specific warehouse genuinely needs to stay warm for latency reasons
2. **Right-size warehouses to the actual query workload** — start smaller and scale up based on observed query performance, not a guess
3. **Use separate warehouses for separate workload types** (e.g., ETL vs. BI dashboards vs. ad hoc analyst queries) so one workload's spikes don't force oversizing for everything
4. **Set Time Travel retention based on actual recovery needs per table**, not a blanket maximum — high-churn tables with long retention are a real storage cost driver
5. **Use zero-copy clones for dev/test environments and pre-migration snapshots**, understanding that ongoing divergence from the source does accrue storage cost over time
6. **Monitor credit consumption by warehouse and by query** regularly, not just at the account level, to attribute cost to actual workloads

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Auto-suspend disabled or set very high "to avoid cold-start latency" | Set a short auto-suspend interval; accept the brief resume latency, or keep only genuinely latency-sensitive warehouses warm |
| Oversized warehouse used for all workloads regardless of actual compute need | Size per workload, using the smallest warehouse that meets latency requirements |
| Running unfiltered/exploratory queries on large tables without checking estimated cost | Use `EXPLAIN` and query history to understand scan size before running exploratory queries on very large tables |
| Setting maximum Time Travel retention on every table by default | Set retention per table based on actual recovery/audit needs — high-churn tables at max retention are expensive |
| Assuming a zero-copy clone stays free indefinitely | Understand that clone storage cost grows with divergence from the source; monitor clone storage over time |

## Sharp Edges

### SE-1: Idle Virtual Warehouses Burning Credits
- **Severity**: critical
- **Situation**: A virtual warehouse is left running (auto-suspend disabled, or set to a very long interval) well past the last actual query, and credits accumulate for compute that's doing nothing
- **Cause**: Snowflake bills for warehouse uptime, not query activity — a warehouse that's "resumed" and sitting idle costs the same per second as one actively executing a query
- **Symptoms**:
  - Credit consumption reports show warehouses with high uptime relative to actual query volume
  - Cost review reveals a warehouse running 24/7 for a workload that only needs a few hours of compute per day
- **Solution**: Set an aggressive auto-suspend interval (commonly 60-300 seconds) on every warehouse unless there's a specific, documented latency reason to keep one warm, and review warehouse uptime-vs-query-count regularly as a cost health check
- **Details**: → [extended/checklists.md#cost-management-checklist]

### SE-2: Oversized Warehouses Wasting Credits
- **Severity**: high
- **Situation**: A warehouse is sized much larger than the actual query workload needs "to be safe" or "for speed," and credit consumption is disproportionately high relative to the performance gained
- **Cause**: Warehouse credit cost roughly doubles with each size increase, but query performance doesn't necessarily improve proportionally for workloads that aren't actually bottlenecked on compute (e.g., queries limited by I/O, small result sets, or simple lookups)
- **Symptoms**:
  - Query history shows queries completing in well under a second on a large warehouse, suggesting significant headroom
  - Cost per query is high relative to the complexity of what's actually being computed
- **Solution**: Start with a smaller warehouse size and scale up based on observed query latency/queueing, use separate appropriately-sized warehouses per workload type rather than one large warehouse serving everything, and periodically review whether warehouse size still matches actual workload characteristics

### SE-3: Unfiltered Queries on Large Tables Burning Credits With No Caching Benefit
- **Severity**: high
- **Situation**: An analyst or dashboard runs a broad, unfiltered query against a very large table repeatedly, and because the query (or its parameters) changes slightly each time, Snowflake's result cache never applies — full compute cost is paid on every run
- **Cause**: Snowflake's query result cache only serves an identical previous query with unchanged underlying data; a query with the exact same shape but a different date filter each day, for example, doesn't benefit from caching and pays full compute cost every time
- **Symptoms**:
  - Recurring, expensive queries show up repeatedly in query history with similar-but-not-identical text
  - Credit consumption is dominated by a small number of large, repeated (but not cached) queries
- **Solution**: Use `EXPLAIN`/query profiling to understand scan size before running broad queries on large tables, filter as narrowly as possible on clustered/pruning-friendly columns, and consider materialized views or scheduled aggregation tables for genuinely recurring expensive queries instead of re-running the full scan each time

### SE-4: Long Time Travel Retention Driving Storage Cost on High-Churn Tables
- **Severity**: medium
- **Situation**: Time Travel retention is set to the maximum (or a high default) across all tables uniformly, and a small number of high-churn tables (frequently updated/deleted) end up responsible for a disproportionate share of storage cost, because Snowflake retains historical micro-partition versions for the full retention window
- **Cause**: Storage cost for Time Travel scales with how much data changes during the retention window — a low-churn table costs little extra for a long retention window, but a high-churn table retained for the same window accumulates much more historical data
- **Symptoms**:
  - Storage cost breakdown shows a small number of tables dominating Time Travel-related storage
  - Storage cost grows faster than raw current-state data volume would suggest
- **Solution**: Set Time Travel retention per table based on actual recovery/audit needs rather than a blanket account-wide maximum, and review storage cost breakdown by table periodically to catch high-churn tables with unnecessarily long retention

### SE-5: Zero-Copy Clones Accruing Unexpected Storage Cost Over Time
- **Severity**: medium
- **Situation**: A zero-copy clone created for a dev/test environment or a pre-migration snapshot is treated as a permanent, cost-free artifact, but months later it's still around, has diverged significantly from the source (or the source has been modified/deleted), and is now responsible for real, ongoing storage cost
- **Cause**: A clone is only free at the moment of creation — as either the clone or the original table changes, the diverging micro-partitions are no longer shared, and each side accrues its own storage cost for the changed portions
- **Symptoms**:
  - Storage cost review reveals old clones still consuming meaningful storage long after their original purpose (a one-time test, a migration snapshot) was served
- **Solution**: Treat clones as having a lifecycle — set a deliberate expectation for how long a dev/test clone or snapshot clone should live, and clean up clones that are no longer needed rather than assuming they stay free indefinitely

## Recommended Tools

| Category | Tools |
|------|------|
| Cost visibility | Snowflake Resource Monitors, `ACCOUNT_USAGE`/`ORGANIZATION_USAGE` views |
| Query analysis | Query Profile, `EXPLAIN` |
| Ingestion | Snowpipe, external stages |
| Orchestration | dbt, Airflow, Snowflake Tasks |

## Cost Management

**Full checklist**: → [extended/checklists.md#cost-management-checklist]

## Related Resources

[Snowflake Docs](https://docs.snowflake.com/) | [Snowflake Cost Management Guide](https://docs.snowflake.com/en/user-guide/cost-understanding-compute) | [Snowflake University](https://learn.snowflake.com/)

## Related Domains

[[clickhouse]] | [[aws]] | [[gcp]] | [[azure]]
