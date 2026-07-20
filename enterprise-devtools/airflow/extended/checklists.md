# Apache Airflow — Extended Checklists

## DAG Design Checklist

- [ ] Every task designed to be idempotent — safe to retry or rerun for the same execution date without corrupting data
- [ ] Writes use upsert, delete-then-insert-for-partition, or an explicit dedup key rather than blind inserts
- [ ] No expensive I/O, API calls, or heavy computation at DAG-file top level (outside task functions)
- [ ] XCom used only for small values (IDs, paths, short status strings); large data passed via object storage/database references
- [ ] Sensors with non-trivial wait times use `mode="reschedule"` or deferrable operators, not default `poke` mode
- [ ] `catchup` explicitly set (`False` unless backfill is genuinely intended) on every DAG
- [ ] Retries, retry delays, and SLAs set explicitly per task based on actual expected failure characteristics

## Scheduler & Operations Checklist

- [ ] DAG parse duration monitored as a standard operational metric
- [ ] Metadata database size monitored for unexpected growth tied to XCom misuse
- [ ] Worker pool capacity sized against actual concurrent task + sensor demand
- [ ] Backfills run explicitly via `airflow dags backfill` with a reviewed date range, not triggered accidentally by enabling a DAG
- [ ] DAG integrity tests (import errors, cycle detection) run in CI before deployment
- [ ] Connections/Variables managed through Airflow's secrets backend integration, not hardcoded in DAG files
- [ ] Task-level alerting (on failure/SLA miss) configured and routed to the team that owns the pipeline
