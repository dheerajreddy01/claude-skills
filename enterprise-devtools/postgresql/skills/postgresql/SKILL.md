---
schema: "1.0"
name: postgresql
version: "1.0.0"
description: PostgreSQL indexing, query planning, transaction isolation, and operational practices
domain: technology
triggers:
  keywords:
    primary: [postgresql, postgres, psql, SQL, query]
    secondary: [index, EXPLAIN, vacuum, transaction, connection pool]
  context_boost: [database, schema design, query performance]
  context_penalty: [mongodb, redis, nosql]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# PostgreSQL

> Correct indexes, honest query plans, and vacuum that actually keeps up

## Applicable Scenarios

- Designing schema and indexes for query patterns
- Reading `EXPLAIN`/`EXPLAIN ANALYZE` output to diagnose slow queries
- Choosing transaction isolation levels for correctness under concurrency
- Diagnosing connection exhaustion or table bloat
- Setting up replication or connection pooling

## Core Knowledge

### Indexing

| Index Type | Best For |
|------|------|
| **B-tree** (default) | Equality and range queries on most data types |
| **GIN** | Full-text search, JSONB containment, array membership |
| **GiST** | Geometric data, range types, nearest-neighbor searches |
| **Hash** | Equality-only lookups (rarely better than B-tree in practice) |
| **BRIN** | Very large tables with naturally correlated/sorted data (e.g., timestamps) |

An index only helps if the planner chooses to use it — a function applied to the indexed column in the `WHERE` clause (`WHERE lower(email) = ...` against a plain index on `email`) prevents index usage unless the index itself is built on that expression.

### Query Planning

- `EXPLAIN` shows the planned execution without running the query; `EXPLAIN ANALYZE` actually runs it and shows real timing/row counts
- **Seq Scan** on a large table where an **Index Scan** was expected is the most common thing to investigate — check for missing indexes, outdated statistics (`ANALYZE`), or a query pattern that defeats the index
- The planner's row-count estimates come from table statistics (`pg_stats`), refreshed by `ANALYZE` (auto-triggered by autovacuum, but can lag on rapidly changing tables)

### Transactions & Isolation

| Isolation Level | Prevents | Default in Postgres |
|------|------|------|
| **Read Committed** | Dirty reads | ✅ Default |
| **Repeatable Read** | Dirty + non-repeatable reads | |
| **Serializable** | All anomalies, including write skew | Strictest, highest overhead |

Postgres's default (Read Committed) allows non-repeatable reads within a transaction — two `SELECT`s on the same row in the same transaction can return different results if another transaction commits a change in between.

### Vacuum & Bloat

- Postgres uses MVCC (multi-version concurrency control) — updates/deletes don't overwrite rows in place, they mark old versions as dead tuples
- **Autovacuum** reclaims dead tuple space and updates statistics — it's not optional maintenance, it's load-bearing for both performance and (via `XID` wraparound prevention) correctness
- A long-running transaction holds back the oldest row version autovacuum can clean up, causing bloat to accumulate across the entire database, not just the tables that transaction touches

## Best Practices

1. **Index for your actual query patterns**, not speculatively — every index adds write overhead and storage
2. **Run `EXPLAIN ANALYZE` on any query that feels slow** before guessing at a fix
3. **Use a connection pooler (PgBouncer/PgCat)** for applications with many short-lived connections — Postgres connections are relatively expensive (each is a full OS process)
4. **Keep transactions short** — a long-running transaction blocks vacuum progress for the whole database, not just its own tables
5. **Paginate large result sets** (`LIMIT`/`OFFSET` or, better, keyset pagination) — never fetch unbounded rows into application memory
6. **Choose isolation level deliberately** for anything with concurrent read-modify-write logic — Read Committed's default behavior isn't always correct for business logic that assumes a stable snapshot

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `WHERE lower(email) = 'x'` against a plain index on `email` | Create a functional/expression index: `CREATE INDEX ON users (lower(email))` |
| Fetching all rows into the app to filter/paginate in code | Filter and paginate in SQL — let the database do what it's good at |
| Opening a new DB connection per request with no pooling | Use a connection pool (app-level or PgBouncer) sized to actual concurrency needs |
| Long-running transaction left open (e.g., waiting on an external API call mid-transaction) | Keep transactions scoped tightly around the actual DB work; do external calls outside the transaction |
| Assuming `SELECT` results are guaranteed consistent within a Read Committed transaction | Use `REPEATABLE READ` or `SERIALIZABLE` where the logic actually requires a stable snapshot |

## Sharp Edges

### SE-1: Missing Index Causing Sequential Scans at Scale
- **Severity**: high
- **Situation**: A query that was fast in development (small table) becomes a major bottleneck in production once the table grows, because it's doing a full sequential scan instead of an index lookup
- **Cause**: Without an appropriate index, the planner has no faster path than scanning every row — this scales linearly with table size and only becomes visible once the table is large enough to matter
- **Symptoms**:
  - `EXPLAIN ANALYZE` shows `Seq Scan` on a large table for a query filtering on a specific column
  - Query latency grows roughly linearly with table row count
- **Solution**: Add an index matching the actual `WHERE`/`JOIN`/`ORDER BY` columns, verify with `EXPLAIN ANALYZE` that the planner switches to an Index Scan, and run `ANALYZE` if statistics are stale
- **Details**: → [extended/checklists.md#query-performance-checklist]

### SE-2: Long-Running Transactions Blocking Autovacuum
- **Severity**: critical
- **Situation**: A long-running transaction (an idle-in-transaction connection, a slow batch job) prevents autovacuum from cleaning up dead tuples across the *entire* database, not just tables the transaction touches — bloat accumulates until performance degrades broadly
- **Cause**: Postgres's MVCC model requires keeping old row versions visible to any transaction that might still need them; a transaction open since before a row was updated blocks cleanup of that row's old version until it commits or aborts
- **Symptoms**:
  - Table/index bloat grows steadily with no corresponding legitimate write volume increase
  - `pg_stat_activity` shows a connection idle in a transaction for an unusually long time
- **Solution**: Set `idle_in_transaction_session_timeout` to auto-terminate stalled transactions, keep transactions scoped tightly around actual DB work, and monitor `pg_stat_activity` for long-running transactions as a standard operational check

### SE-3: Connection Exhaustion Without Pooling
- **Severity**: high
- **Situation**: An application opens a new database connection per request (or per worker without pooling), and under load it hits Postgres's `max_connections` limit, causing new connections — and the whole application — to fail
- **Cause**: Each Postgres connection is a full backend OS process with real memory overhead; `max_connections` is set conservatively for a reason, and application-level connection sprawl (especially across many app instances/serverless functions) can exceed it quickly
- **Symptoms**:
  - `FATAL: too many connections` errors under load
  - Database memory usage scales with concurrent request count, not query volume
- **Solution**: Use a connection pooler (PgBouncer in transaction mode, or an application-level pool) sized to the database's actual `max_connections` budget, especially for serverless/many-instance architectures where naive per-instance pooling multiplies connection count

### SE-4: Implicit Type Casting Preventing Index Usage
- **Severity**: medium
- **Situation**: A query filters an indexed integer column against a string literal (or vice versa), and the planner silently applies a type cast that defeats the index, falling back to a sequential scan
- **Cause**: Comparing mismatched types requires an implicit cast, and depending on the cast direction, Postgres may not be able to use a standard index on the original column for that comparison
- **Symptoms**:
  - A query on an indexed column still shows `Seq Scan` in `EXPLAIN`, with no obvious missing-index explanation
  - The issue disappears when the literal's type in the query is corrected to match the column
- **Solution**: Match parameter/literal types to the column's actual type in application code and queries, and check `EXPLAIN` for unexpected `::type` casts in the plan when an indexed column still triggers a sequential scan

### SE-5: Unbounded Result Sets Exhausting Application Memory
- **Severity**: medium
- **Situation**: A query with no `LIMIT` returns far more rows than expected (a bug, or simply data growth over time), and the application tries to load the entire result set into memory, causing an OOM or severe latency spike
- **Cause**: Nothing in Postgres or most ORMs enforces a result size cap by default — a query that returns 100 rows in development can return millions in production as the dataset grows
- **Symptoms**:
  - Application memory usage spikes correlated with a specific endpoint/query
  - A query that "always worked" starts failing only after the underlying table grew significantly
- **Solution**: Always paginate (prefer keyset/cursor pagination over `OFFSET` for large offsets, which itself gets slow), and set application-level safety limits on result set size as a backstop against unexpectedly large queries

## Recommended Tools

| Category | Tools |
|------|------|
| Query analysis | `EXPLAIN ANALYZE`, `pg_stat_statements`, `auto_explain` |
| Connection pooling | PgBouncer, PgCat |
| Monitoring | `pg_stat_activity`, `pgBadger`, pganalyze |
| Migrations | Flyway, golang-migrate, Alembic |

## Query Performance

**Full checklist**: → [extended/checklists.md#query-performance-checklist]

## Related Resources

[PostgreSQL Docs](https://www.postgresql.org/docs/current/) | [Use The Index, Luke](https://use-the-index-luke.com/) | [PostgreSQL Wiki: Performance](https://wiki.postgresql.org/wiki/Performance_Optimization)

## Related Domains

[[redis]] | [[mongodb]] | [[python]] | [[go]]
