# SQLite — Extended Checklists

## Concurrency Checklist

- [ ] WAL mode explicitly enabled (`PRAGMA journal_mode=WAL;`) and verified active
- [ ] `busy_timeout` set to a reasonable value so transient write contention retries instead of failing immediately
- [ ] Application-level write serialization in place if multiple threads/processes could write concurrently
- [ ] Workload confirmed to fit SQLite's single-writer model before committing to it as the backend for a server application
- [ ] Migration path to a client-server database identified in advance if concurrent write growth is plausible
- [ ] `PRAGMA synchronous` setting reviewed against the durability-vs-performance tradeoff needed
- [ ] Long-running read transactions checked for blocking WAL checkpoint progress

## Deployment & Schema Checklist

- [ ] Database file confirmed to live on local storage, never a network filesystem or cloud-sync folder shared across processes/machines
- [ ] `STRICT` tables used where real type enforcement is required (SQLite 3.37+)
- [ ] Application-layer validation in place for type/shape assumptions where `STRICT` tables aren't used
- [ ] `VACUUM`/`PRAGMA optimize` scheduled periodically for databases with meaningful delete/update churn
- [ ] Backup strategy in place (file copy during a checkpoint, or a tool like Litestream) — no assumption that a running database file is safely copyable at any moment
- [ ] Database file permissions reviewed if it contains sensitive data, since there's no built-in server-side access control layer
