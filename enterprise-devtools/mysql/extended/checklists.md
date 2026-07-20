# MySQL — Extended Checklists

## Schema and Query Checklist

- [ ] All new tables use `utf8mb4` (not legacy `utf8`), with application connection charset matching
- [ ] Primary keys are auto-increment or otherwise monotonic — no random UUID as the InnoDB clustered index key
- [ ] `EXPLAIN` run on any query suspected of being slow; `type: ALL` (full scan) investigated before accepting it
- [ ] Composite indexes designed with left-to-right column order matching actual query filters
- [ ] Transaction isolation level understood explicitly (REPEATABLE READ is the default) and its gap-locking implications accounted for in transaction design
- [ ] Long transactions avoided, especially ones performing range scans that could hold gap locks
- [ ] Foreign key constraints used where InnoDB referential integrity is actually desired

## Operational Health Checklist

- [ ] Connection pooling (application-level or ProxySQL) in place, sized against `max_connections`
- [ ] Replication lag (`Seconds_Behind_Source`) monitored and alerted on
- [ ] Read-your-own-write flows routed to the primary, not an async replica
- [ ] Semi-synchronous replication or Group Replication evaluated if stronger consistency is required
- [ ] Slow query log enabled and reviewed periodically, not just investigated reactively
- [ ] Backup/restore process (mysqldump, Percona XtraBackup, or managed snapshot) tested periodically
- [ ] Schema migrations for large tables use online/non-locking tooling (`pt-online-schema-change`, `gh-ost`) to avoid long table locks
