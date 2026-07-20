# PostgreSQL — Extended Checklists

## Query Performance Checklist

- [ ] `EXPLAIN ANALYZE` run on any query suspected of being slow, before guessing at a fix
- [ ] Indexes match actual `WHERE`/`JOIN`/`ORDER BY` column usage, including functional/expression indexes where a function wraps the column
- [ ] No implicit type mismatches between query literals/parameters and indexed column types
- [ ] Table statistics current (`ANALYZE` run recently, or autovacuum's analyze threshold appropriate for the table's write rate)
- [ ] Every unbounded-looking query has an explicit `LIMIT`/pagination strategy
- [ ] Large-offset pagination replaced with keyset/cursor pagination where offsets get deep
- [ ] `pg_stat_statements` reviewed periodically to find the actual highest-impact slow queries, not just the ones that "feel" slow

## Operational Health Checklist

- [ ] `idle_in_transaction_session_timeout` set to auto-terminate stalled transactions
- [ ] `pg_stat_activity` monitored for long-running transactions/queries as a standard check
- [ ] Connection pooling (PgBouncer/PgCat or app-level pool) in place, sized against `max_connections`
- [ ] Autovacuum settings reviewed for high-write tables — default thresholds don't always keep up with very high churn
- [ ] Table/index bloat monitored, not just discovered reactively after performance degrades
- [ ] Replication lag (if using read replicas) monitored and alerted on
- [ ] Backup/restore process tested periodically, not just configured and assumed working
