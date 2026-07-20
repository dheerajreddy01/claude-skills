---
schema: "1.0"
name: mysql
version: "1.0.0"
description: MySQL/InnoDB indexing, isolation levels, replication, and operational practices
domain: technology
triggers:
  keywords:
    primary: [mysql, mariadb, innodb, SQL, EXPLAIN]
    secondary: [replication, binlog, clustered index, gap lock, charset]
  context_boost: [database, relational, query performance]
  context_penalty: [postgresql, mongodb, redis]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# MySQL

> InnoDB's clustered index and default isolation level shape almost everything else

## Applicable Scenarios

- Designing schema, primary keys, and indexes for InnoDB tables
- Diagnosing slow queries via `EXPLAIN`
- Choosing transaction isolation levels and understanding lock behavior
- Setting up and monitoring replication
- Debugging character set/collation issues

## Core Knowledge

### Storage Engine: InnoDB

- InnoDB is the default and standard choice — it's transactional (ACID), row-level locking, and supports foreign keys; MyISAM (older, table-level locking, non-transactional) should be treated as legacy
- **Clustered index**: InnoDB physically stores table rows in primary key order — the primary key *is* the table's data structure, not a separate structure pointing to rows. Every secondary index stores the primary key value and does a second lookup into the clustered index to fetch the full row

### Indexing

- Because the primary key determines physical row order, an unordered/random primary key (e.g., a random UUID) causes new inserts to land in random pages, causing page splits and fragmentation — an auto-increment integer or a time-ordered key avoids this
- `EXPLAIN` shows the query plan; look for `type: ALL` (full table scan) where an index-based access (`ref`, `range`, `const`) was expected
- Composite indexes are used left-to-right — an index on `(a, b, c)` helps queries filtering on `a`, or `a`+`b`, or `a`+`b`+`c`, but not on `b` alone

### Transaction Isolation

- MySQL/InnoDB's **default isolation level is REPEATABLE READ** (unlike PostgreSQL's default of READ COMMITTED) — this has real behavioral consequences, particularly around locking
- InnoDB implements REPEATABLE READ partly via **gap locks** (locking the "gap" between index records, not just existing rows) to prevent phantom reads — this is a common source of unexpected deadlocks that don't have an obvious equivalent in databases with simpler locking models

### Replication

- Classic MySQL replication is **asynchronous** by default (binlog-based, statement or row-based) — a replica can lag behind the primary by a measurable amount, and reads against it are not guaranteed to reflect the latest committed writes
- **Semi-synchronous** replication (waiting for at least one replica to acknowledge before committing) reduces (but doesn't eliminate) this risk; **Group Replication**/InnoDB Cluster provides stronger consistency guarantees at additional operational complexity

## Best Practices

1. **Use an auto-increment (or otherwise monotonic) primary key** for InnoDB tables — avoid random UUIDs as the clustered index key
2. **Always use `utf8mb4`, never `utf8`** for new schemas — MySQL's `utf8` is a legacy 3-byte encoding that can't store full Unicode (including emoji) and silently truncates or errors
3. **Run `EXPLAIN` on any query that feels slow** before guessing at an index fix
4. **Understand what isolation level you're actually running under** — REPEATABLE READ's gap-locking behavior needs to be accounted for in transaction design, not assumed away
5. **Treat replica reads as eventually consistent** — don't read-your-own-write from a replica immediately after writing to the primary without accounting for lag
6. **Set `max_connections` deliberately and use connection pooling** — don't rely on defaults for production load

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using `utf8` charset for new tables | Use `utf8mb4` — `utf8` cannot store full Unicode and silently truncates |
| Random UUID as InnoDB primary key | Use an auto-increment integer, or a time-sortable ID (e.g., ULID) if a non-sequential ID is required |
| Assuming a query is using an index without checking | Run `EXPLAIN` and verify `type` isn't `ALL` (full scan) |
| Reading from a replica immediately after writing to the primary and expecting consistency | Read from the primary for read-your-own-write flows, or use semi-sync/Group Replication if strong consistency is required |
| Long transactions under REPEATABLE READ holding gap locks | Keep transactions short and scoped; understand the locking implications of range queries inside a transaction |

## Sharp Edges

### SE-1: `utf8` vs `utf8mb4` Silently Truncating Data
- **Severity**: critical
- **Situation**: A table created with the legacy `utf8` charset silently truncates or rejects data containing emoji or certain non-BMP Unicode characters, and the failure mode is either silent truncation or a cryptic insert error
- **Cause**: MySQL's `utf8` charset is actually a 3-byte-max encoding, not full UTF-8 — it cannot represent 4-byte Unicode code points (which includes emoji and some CJK extension characters); `utf8mb4` is MySQL's actual full UTF-8 implementation
- **Symptoms**:
  - Emoji or certain international text gets truncated or causes an `Incorrect string value` error on insert
  - Data that displays correctly in one system silently loses characters after round-tripping through MySQL
- **Solution**: Use `utf8mb4` (with `utf8mb4_unicode_ci` or `utf8mb4_0900_ai_ci` collation) for all new schemas and migrate legacy `utf8` tables deliberately, checking application-layer connection charset settings match
- **Details**: → [extended/checklists.md#schema-and-query-checklist]

### SE-2: Unexpected Deadlocks From Gap Locking Under REPEATABLE READ
- **Severity**: high
- **Situation**: Two concurrent transactions performing range-based inserts/updates deadlock in ways that seem to have no direct row overlap, under MySQL's default REPEATABLE READ isolation
- **Cause**: InnoDB's REPEATABLE READ uses gap locks and next-key locks (locking ranges between index entries, not just matched rows) to prevent phantom reads — two transactions locking overlapping *gaps* (not necessarily the same rows) can deadlock in ways unfamiliar to engineers used to simpler locking models
- **Symptoms**:
  - `ERROR 1213: Deadlock found when trying to get lock` on queries that don't appear to touch the same rows
  - Deadlocks concentrated on tables with frequent range inserts/updates under concurrent load
- **Solution**: Keep transactions short and touch rows in a consistent order across the application, consider `READ COMMITTED` isolation (which disables gap locking for non-locking reads) if the application's consistency needs allow it, and use `SHOW ENGINE INNODB STATUS` to inspect actual deadlock details when they occur

### SE-3: Missing Index Causing Full Table Scans at Scale
- **Severity**: high
- **Situation**: A query that performs well in development becomes a major bottleneck in production as the table grows, because it lacks a supporting index and falls back to a full table scan
- **Cause**: Without an appropriate index, MySQL has no faster path than scanning every row to find matches — this scales linearly and only becomes visible once the table is large
- **Symptoms**:
  - `EXPLAIN` shows `type: ALL` with a `rows` estimate close to the table's total row count
  - Query latency grows roughly linearly with table size
- **Solution**: Add indexes matching actual `WHERE`/`JOIN`/`ORDER BY` columns, respect the left-to-right rule for composite indexes, and verify with `EXPLAIN` that the planner is actually using the intended index

### SE-4: Replication Lag Causing Stale Reads
- **Severity**: high
- **Situation**: An application writes to the primary and immediately reads from a replica (for read scaling), and the read doesn't reflect the just-written data because the replica hasn't caught up yet
- **Cause**: Standard asynchronous MySQL replication applies binlog events to replicas after the fact, with no guaranteed bound on lag — under load or with slow replica hardware, lag can grow from milliseconds to seconds or more
- **Symptoms**:
  - A user's own just-submitted change appears to be "missing" immediately after submission, then shows up moments later
  - `SHOW REPLICA STATUS` shows a non-trivial `Seconds_Behind_Source` value correlated with the issue
- **Solution**: Route read-your-own-write queries to the primary (or a replica known to be caught up), monitor replication lag as a standard operational metric, and consider semi-synchronous replication or Group Replication if the application genuinely requires stronger consistency guarantees from replicas

### SE-5: Connection Exhaustion Under Load
- **Severity**: medium
- **Situation**: An application under load opens more connections than `max_connections` allows, and new connection attempts start failing with `Too many connections`, taking down the whole application rather than degrading gracefully
- **Cause**: Each MySQL connection consumes server resources, and `max_connections` is a hard cap; without pooling, an application's connection count can scale directly with concurrent request count rather than staying bounded
- **Symptoms**:
  - `ERROR 1040: Too many connections` under traffic spikes
  - Connection count in `SHOW PROCESSLIST` scales with request volume rather than staying roughly constant
- **Solution**: Use connection pooling (application-level pool or a proxy like ProxySQL) sized to a sane fraction of `max_connections`, especially in architectures with many app instances or serverless functions that could otherwise each open their own pool

## Recommended Tools

| Category | Tools |
|------|------|
| Query analysis | `EXPLAIN`, `EXPLAIN ANALYZE` (8.0+), `SHOW ENGINE INNODB STATUS`, Performance Schema |
| Connection pooling/proxy | ProxySQL, application-level connection pools |
| Replication monitoring | `SHOW REPLICA STATUS`, Orchestrator |
| Migrations | Flyway, golang-migrate, Percona Toolkit (`pt-online-schema-change`) |

## Schema & Query Health

**Full checklist**: → [extended/checklists.md#schema-and-query-checklist]

## Related Resources

[MySQL Docs](https://dev.mysql.com/doc/) | [Use The Index, Luke](https://use-the-index-luke.com/) | [High Performance MySQL (O'Reilly)](https://www.oreilly.com/library/view/high-performance-mysql/9781492080503/)

## Related Domains

[[postgresql]] | [[redis]] | [[java]] | [[python]]
