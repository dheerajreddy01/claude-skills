---
schema: "1.0"
name: sqlite
version: "1.0.0"
description: SQLite embedded database concurrency model, WAL mode, and appropriate-use practices
domain: technology
triggers:
  keywords:
    primary: [sqlite, embedded database, single-file database]
    secondary: [WAL mode, database locked, type affinity, vacuum]
  context_boost: [mobile app, edge, local-first, desktop app]
  context_penalty: [postgresql, mysql, high-concurrency server]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# SQLite

> The best database for one writer at a time — know that boundary before you hit it in production

## Applicable Scenarios

- Choosing whether SQLite is appropriate for a given workload
- Configuring WAL mode and understanding its concurrency implications
- Diagnosing `SQLITE_BUSY`/"database is locked" errors
- Reviewing schema design under SQLite's dynamic typing (type affinity) model
- Deploying SQLite on networked or shared storage

## Core Knowledge

### Single-File, Embedded Architecture

- The entire database — schema, data, indexes — lives in one file (plus WAL/journal side-files when active) with no separate server process; the application links directly against the SQLite library
- This makes SQLite ideal for embedded, mobile, desktop, and edge use cases where "install and run a database server" is the wrong amount of operational weight

### Concurrency Model

| Mode | Behavior |
|------|------|
| **Rollback journal** (default historically) | Writers take an exclusive lock on the whole database file during a write; readers are blocked during that window |
| **WAL (Write-Ahead Logging)** | Writes append to a separate WAL file; readers can proceed concurrently with a single writer, and don't block each other |

Even in WAL mode, SQLite allows only **one writer at a time** — there is no multi-writer concurrency model. This is a fundamental design constraint, not a tunable performance knob: SQLite is built for "one process/thread writing, many reading," not for the many-concurrent-writers pattern a client-server database like PostgreSQL or MySQL handles natively.

### Type Affinity

- SQLite is dynamically typed at the storage level — a column has a **type affinity** (a preference), not a strict type constraint; you can insert a string into a column with `INTEGER` affinity and SQLite will store it as-is if it doesn't cleanly convert
- This is a deliberate design choice for flexibility, but it means schema mistakes that a strictly-typed database would reject outright are silently accepted by default

### When SQLite Is (and Isn't) the Right Choice

| Good Fit | Poor Fit |
|------|------|
| Mobile/desktop app local storage | Multi-writer web server backend under real concurrent write load |
| Edge/IoT devices | High-concurrency OLTP with many simultaneous writers |
| Local-first apps, embedded config/cache | Networked/shared-storage deployments (NFS, network drives) |
| Testing/prototyping | Applications requiring strict schema type enforcement |

## Best Practices

1. **Enable WAL mode** (`PRAGMA journal_mode=WAL;`) for any application with concurrent readers and a writer — it's a substantial concurrency improvement over the default rollback journal
2. **Treat SQLite as single-writer** in application design — serialize writes at the application layer if multiple threads/processes might write concurrently, rather than fighting `SQLITE_BUSY` reactively
3. **Never put the database file on a network filesystem** (NFS, SMB, cloud-synced folders) — SQLite's locking assumes local filesystem semantics that networked storage doesn't reliably provide
4. **Use `STRICT` tables (SQLite 3.37+)** where real type enforcement matters, since default type affinity is permissive
5. **Set a `busy_timeout`** so a writer waiting on a lock retries briefly instead of failing immediately on the first contention
6. **Run `VACUUM`/`PRAGMA optimize` periodically** for databases with significant delete/update churn, to reclaim space and keep the query planner's statistics current

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Leaving the default rollback journal mode for an app with concurrent readers/writer | Enable WAL mode explicitly |
| Choosing SQLite for a multi-writer server backend under real concurrent load | Use a client-server database (PostgreSQL/MySQL) designed for that concurrency pattern |
| Placing the database file on a network drive or NFS mount | Keep the database file on local storage only |
| Assuming a column's declared type is enforced like a relational database's | Use `STRICT` tables where real type enforcement is required |
| No `busy_timeout` set, causing immediate failures under any write contention | Set a reasonable `busy_timeout` so transient contention retries instead of erroring immediately |

## Sharp Edges

### SE-1: `SQLITE_BUSY` Under Concurrent Writers
- **Severity**: high
- **Situation**: An application with multiple threads or processes writing to the same SQLite database intermittently fails with `SQLITE_BUSY` ("database is locked"), especially under load
- **Cause**: SQLite allows only one writer at a time regardless of journal mode; a second writer attempting to write while another transaction holds the write lock is rejected (or blocks, depending on `busy_timeout`) rather than queued transparently
- **Symptoms**:
  - Intermittent `database is locked` errors that correlate with concurrent write load, not any single request in isolation
  - Errors that don't reproduce in single-threaded testing but appear consistently in production concurrency
- **Solution**: Enable WAL mode to reduce reader/writer contention, set an explicit `busy_timeout` so writers retry briefly instead of failing immediately, and serialize writes at the application layer (a single writer queue/thread) if the workload genuinely has multiple concurrent write sources
- **Details**: → [extended/checklists.md#concurrency-checklist]

### SE-2: Choosing SQLite for a High-Concurrency Server Workload
- **Severity**: critical
- **Situation**: SQLite is chosen as the backend for a multi-user web application expecting many concurrent writers, and the application hits a hard concurrency ceiling that no amount of tuning fixes, because the constraint is architectural, not a configuration problem
- **Cause**: SQLite's single-writer model is a fundamental design decision, not a default that can be reconfigured — it was built for embedded/single-process use cases, not as a drop-in replacement for a client-server RDBMS under real multi-writer concurrency
- **Symptoms**:
  - Write throughput plateaus and further tuning (indexes, WAL mode, timeouts) doesn't meaningfully raise the ceiling
  - The application's growth trajectory outpaces what any single-writer database can sustain
- **Solution**: Recognize this at design time — SQLite is the wrong tool for a server backend expecting concurrent multi-writer load; migrate to PostgreSQL/MySQL before the constraint becomes a production emergency, not after

### SE-3: WAL Mode Not Enabled, Leaving Concurrency on the Table
- **Severity**: medium
- **Situation**: An application experiences more read/write contention than necessary because it never explicitly enabled WAL mode and is still running under the default rollback journal
- **Cause**: WAL mode isn't the default for backward-compatibility reasons in many SQLite deployments/bindings — it must be explicitly enabled via `PRAGMA journal_mode=WAL;`, and it's easy to never discover this if the application "just works" under light load
- **Symptoms**:
  - Reads block during writes more than expected for the actual concurrency level
  - `PRAGMA journal_mode;` reveals `delete` (rollback journal) rather than `wal`
- **Solution**: Explicitly set WAL mode at database initialization/connection setup, and verify it's actually active with `PRAGMA journal_mode;` rather than assuming a library default

### SE-4: Network Filesystem Corruption Risk
- **Severity**: critical
- **Situation**: A SQLite database file is placed on a network filesystem (NFS, SMB share, or a cloud-sync folder like Dropbox), and file locking behaves inconsistently across clients — leading to database corruption, especially under any concurrent access
- **Cause**: SQLite's locking implementation assumes POSIX/local filesystem locking semantics; many network filesystems implement locking unreliably, incompletely, or with significant latency that breaks SQLite's assumptions about lock atomicity
- **Symptoms**:
  - Sporadic database corruption with no clear application-level cause
  - Issues that only manifest when the database file is accessed from more than one machine/mount point
- **Solution**: Never place a SQLite database file on networked or cloud-synced storage accessed by more than one process/machine; if shared access across machines is genuinely required, use a client-server database instead

### SE-5: Type Affinity Silently Accepting Mismatched Data
- **Severity**: medium
- **Situation**: A column declared with `INTEGER` affinity silently accepts and stores a string value, and application code written assuming strict typing (as in most other databases) breaks unexpectedly when it later reads that value back and gets a type it didn't expect
- **Cause**: SQLite's type system is affinity-based (a storage preference) rather than a strict constraint by default — unlike most relational databases, it doesn't reject an insert just because the value's type doesn't match the column's declared type
- **Symptoms**:
  - A value inserted without validation ends up stored as an unexpected type
  - Application code that assumes type safety (common when porting from a strictly-typed database) encounters a type it didn't account for
- **Solution**: Use `STRICT` tables (available since SQLite 3.37) wherever real type enforcement matters, and validate data types at the application layer for older SQLite versions or non-strict tables

## Recommended Tools

| Category | Tools |
|------|------|
| Inspection | `sqlite3` CLI, DB Browser for SQLite |
| Bindings | Language-native SQLite drivers (most languages ship one) |
| Migrations | golang-migrate, Alembic, Flyway |
| Replication/sync | Litestream (WAL-based streaming backup/replication) |

## Concurrency & Safety

**Full checklist**: → [extended/checklists.md#concurrency-checklist]

## Related Resources

[SQLite Docs](https://www.sqlite.org/docs.html) | [SQLite WAL Mode](https://www.sqlite.org/wal.html) | [Appropriate Uses for SQLite](https://www.sqlite.org/whentouse.html)

## Related Domains

[[postgresql]] | [[mysql]] | [[python]] | [[go]]
