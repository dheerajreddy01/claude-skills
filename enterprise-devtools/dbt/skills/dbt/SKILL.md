---
schema: "1.0"
name: dbt
version: "1.0.0"
description: dbt (data build tool) model design, DAG dependencies, materializations, and incremental transformation practices
domain: technology
triggers:
  keywords:
    primary: [dbt, data build tool, dbt model, dbt run]
    secondary: [ref, source, materialization, incremental model, jinja, dbt test]
  context_boost: [analytics engineering, ELT, data warehouse transformation]
  context_penalty: [airflow, spark, etl orchestration]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# dbt (data build tool)

> Transformation lives in the warehouse, the DAG lives in `ref()`, and tests are not optional

## Applicable Scenarios

- Designing dbt models and deciding materialization strategy (view/table/incremental/ephemeral)
- Structuring a project's DAG via `ref()`/`source()` dependencies
- Writing incremental models that must handle late-arriving or updated data correctly
- Adding schema tests and custom data tests to catch broken data before it reaches dashboards
- Debugging Jinja-templated SQL that doesn't compile to what the author intended

## Core Knowledge

### ELT, Not ETL

- dbt only handles the **T** — transformation. Extraction and loading (getting raw data into the warehouse) is someone else's job (Fivetran, Airbyte, custom scripts, or Airflow-orchestrated loaders)
- Models are SQL `SELECT` statements (optionally templated with Jinja); dbt compiles and runs them against the warehouse, materializing results as views/tables

### The DAG: `ref()` and `source()`

- `ref('other_model')` tells dbt "this model depends on that model" — dbt uses this to build the DAG and run models in the correct order, and to rewrite the reference to the correct compiled table name per environment (dev/staging/prod schemas)
- `source('raw_db', 'raw_table')` does the same for raw, unmodeled source tables, and enables source freshness checks
- Hardcoding a schema-qualified table name instead of using `ref()`/`source()` invisibly breaks the DAG — dbt can't see that dependency, so it can't guarantee run order or environment-correct table resolution

### Materializations

| Materialization | Behavior | Best For |
|------|------|------|
| **view** | Compiled as a SQL view, recomputed on every query | Cheap to build, fine for lightweight/rarely-queried models |
| **table** | Fully rebuilt every run | Simpler correctness guarantees, more compute per run |
| **incremental** | Only processes new/changed rows on subsequent runs | Large fact tables where a full rebuild is too expensive |
| **ephemeral** | Inlined as a CTE into downstream models, never materialized itself | Reusable SQL logic that doesn't need its own table |

Incremental models trade simplicity for performance — the incremental predicate (usually a `WHERE` clause on a timestamp) has to correctly capture exactly the rows that changed, including late-arriving data, or the model silently drifts from what a full rebuild would produce.

### Tests

- **Schema tests** (`unique`, `not_null`, `relationships`, `accepted_values`) are declarative, defined in YAML, and run automatically
- **Custom data tests** are SQL queries that should return zero rows if the data is correct — anything else fails the test
- Tests are what make a dbt project trustworthy — a model with no tests is a model nobody can be confident about after the first refactor

## Best Practices

1. **Always use `ref()`/`source()`**, never a hardcoded table name, so the DAG and environment resolution stay correct
2. **Test every model that feeds a dashboard or downstream decision** — at minimum `not_null`/`unique` on primary keys
3. **Design incremental predicates defensively** — account for late-arriving data with a lookback window, not just "since last run"
4. **Run both incremental and full-refresh in CI** for any incremental model, so divergence is caught before production
5. **Keep Jinja macros simple and inspectable** — use `dbt compile` to check the generated SQL matches intent before trusting it
6. **Document models** (descriptions, column-level docs) so the DAG is understandable to someone who didn't write it

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Hardcoding a table name instead of `ref()`/`source()` | Always reference other models/sources through `ref()`/`source()` |
| Incremental predicate using only `> max(loaded_at)` with no lookback | Add a lookback window to catch late-arriving/updated rows |
| Treating tests as optional or added only when convenient | Add schema tests to every model touching a primary key or feeding a report |
| Never running a full-refresh in CI for incremental models | Test both incremental and `--full-refresh` runs before merging |
| Complex Jinja macros with no visibility into compiled output | Run `dbt compile` and review generated SQL when debugging |

## Sharp Edges

### SE-1: Incremental Logic Silently Duplicating or Dropping Rows
- **Severity**: critical
- **Situation**: An incremental model's predicate only looks at rows newer than the last run's watermark, but a source record arrives late (after its "true" timestamp) or gets updated after already being loaded — it's silently missed or duplicated depending on the exact logic
- **Cause**: Incremental materializations only reprocess what the predicate says changed; if the predicate assumes strictly monotonic, on-time arrival, any late-arriving or updated row falls outside that window
- **Symptoms**:
  - Row counts between an incremental model and a full-refresh of the same model don't match
  - Downstream aggregates are quietly wrong for a specific date range, discovered only when someone cross-checks against a fresh full-refresh
- **Solution**: Build a lookback window into the incremental predicate (e.g., reprocess the last N days, not just "since last run"), and periodically diff incremental output against a full-refresh to catch silent drift
- **Details**: → [extended/checklists.md#model-correctness-checklist]

### SE-2: `ref()`/`source()` Bypass Breaking the DAG
- **Severity**: high
- **Situation**: A model references another table by its literal schema-qualified name instead of `ref()`, and it either runs against the wrong environment's table or runs before/after it should relative to its actual dependency
- **Cause**: dbt's DAG (and therefore its run ordering and environment-aware name resolution) is built entirely from `ref()`/`source()` calls — anything outside that is invisible to dbt
- **Symptoms**:
  - A model queries stale or wrong-environment data despite the "correct" model having already run
  - `dbt docs generate`'s dependency graph is missing an edge that clearly should exist
- **Solution**: Enforce `ref()`/`source()` usage via code review and linting (e.g., `dbt-checkpoint` or custom pre-commit checks), and grep for schema-qualified literals in model SQL as a periodic audit

### SE-3: Full-Refresh and Incremental Behavior Diverging
- **Severity**: high
- **Situation**: An incremental model works correctly in day-to-day runs, but running `dbt run --full-refresh` produces different (usually more correct, sometimes less) results — the two code paths were never actually tested against each other
- **Cause**: The incremental materialization's `is_incremental()` Jinja branch is genuinely different SQL from the full-refresh path; nothing forces them to produce equivalent output for the same underlying data
- **Symptoms**:
  - A full-refresh (run after a schema change or data quality investigation) produces numbers that don't match what stakeholders have been seeing
  - The discrepancy is only noticed when someone runs a full-refresh for an unrelated reason
- **Solution**: Include a full-refresh run in CI (on a test/sample dataset) alongside the standard incremental run, and diff the outputs to catch behavioral divergence before it reaches production

### SE-4: Tests Treated as Optional, Letting Broken Data Ship
- **Severity**: high
- **Situation**: A model has no tests (or only a token test on an unrelated column), a source data quality issue introduces duplicate keys or nulls, and it flows straight through to a production dashboard without dbt ever failing
- **Cause**: dbt doesn't require tests — a model with zero tests runs "successfully" even if its output is nonsensical, because success is defined as "the SQL executed," not "the data is correct"
- **Symptoms**:
  - A stakeholder reports a dashboard number that's obviously wrong, and root cause traces back to a data quality issue that a `unique`/`not_null` test would have caught immediately
- **Solution**: Require at minimum `unique`+`not_null` tests on primary keys for any model that feeds a report, and block deploys on test failures in CI rather than treating dbt test as a manual, optional step

### SE-5: Jinja Macro Complexity Obscuring Actual Generated SQL
- **Severity**: medium
- **Situation**: A heavily macro-ized model produces subtly wrong SQL under certain input combinations, and debugging it by reading the Jinja source is far harder than debugging plain SQL would be
- **Cause**: Jinja macros are a templating layer over SQL — the author's mental model of what the macro does can drift from what it actually compiles to, especially with conditional logic or macro composition
- **Symptoms**:
  - A model fails or produces wrong output only for specific input data, and the failure isn't obvious from reading the macro-templated source
- **Solution**: Use `dbt compile` routinely to inspect the actual generated SQL, keep macros focused on genuine reuse (not cleverness for its own sake), and prefer straightforward SQL over a macro when the macro doesn't save meaningful duplication

## Recommended Tools

| Category | Tools |
|------|------|
| Development | dbt Core, dbt Cloud, dbt Power User (VS Code extension) |
| Testing/quality | dbt test, dbt-expectations, Elementary |
| Documentation | `dbt docs generate` (auto-generated DAG + column docs) |
| CI | dbt Cloud CI, GitHub Actions with `dbt build` |

## Model Correctness

**Full checklist**: → [extended/checklists.md#model-correctness-checklist]

## Related Resources

[dbt Docs](https://docs.getdbt.com/) | [dbt Best Practices Guide](https://docs.getdbt.com/best-practices) | [dbt Discourse](https://discourse.getdbt.com/)

## Related Domains

[[airflow]] | [[snowflake]] | [[postgresql]] | [[spark]]
