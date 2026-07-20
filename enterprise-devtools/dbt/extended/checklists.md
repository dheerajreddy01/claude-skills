# dbt — Extended Checklists

## Model Correctness Checklist

- [ ] Every reference to another model/source uses `ref()`/`source()`, never a hardcoded schema-qualified name
- [ ] Incremental predicates include a lookback window to catch late-arriving or updated rows, not just "since last run"
- [ ] `dbt run --full-refresh` tested in CI (on a sample/test dataset) and diffed against incremental output for divergence
- [ ] Primary keys on every model feeding a report have `unique` + `not_null` schema tests
- [ ] Custom data tests cover known business-logic invariants (e.g., "revenue should never be negative")
- [ ] `dbt compile` reviewed for any model using non-trivial Jinja macros, to confirm generated SQL matches intent
- [ ] Source freshness checks configured for time-sensitive raw sources
- [ ] Model documentation (descriptions, column docs) kept current, not written once and abandoned

## CI and Project Hygiene Checklist

- [ ] `dbt build` (run + test together) gates merges in CI, not just `dbt run`
- [ ] Ephemeral materializations used for reusable logic that doesn't need its own table, to avoid unnecessary storage/compute
- [ ] Materialization choice (view/table/incremental) matches actual query frequency and rebuild cost tradeoffs, not just an arbitrary team default
- [ ] Slim CI (`--defer`/state comparison) used to avoid rebuilding the entire DAG on every PR in large projects
- [ ] Deprecated/unused models pruned periodically to keep the DAG navigable
- [ ] Exposures documented so downstream consumers (dashboards, other teams) of a model are visible in the DAG
