---
schema: "1.0"
name: airflow
version: "1.0.0"
description: Apache Airflow DAG design, task idempotency, XCom usage, and scheduler operational practices
domain: technology
triggers:
  keywords:
    primary: [airflow, DAG, apache airflow, task idempotency]
    secondary: [XCom, sensor task, backfill, scheduler, DAG run]
  context_boost: [data pipeline, workflow orchestration, ETL scheduling]
  context_penalty: [dbt, kubernetes, ci/cd pipeline]
priority: high
dependencies:
  software-skills: [python]
author: claude-domain-skills
---

# Apache Airflow

> A task will be retried and rerun eventually — design for that from the first line, not as an afterthought

## Applicable Scenarios

- Designing DAGs for data pipeline orchestration and scheduling
- Deciding how to pass data between tasks (XComs vs. external storage)
- Diagnosing scheduler slowness or DAG parsing performance issues
- Reviewing sensor task usage and its resource consumption
- Reasoning about backfill/catchup behavior when enabling or modifying a DAG

## Core Knowledge

### DAGs as the Scheduling Abstraction

- A **DAG** (directed acyclic graph) defines tasks and their dependencies; Airflow's scheduler continuously parses DAG files and creates **DAG runs** based on the schedule
- DAG files are Python — this is powerful (full programming language for defining pipeline structure) but also means DAG-file-level code executes repeatedly outside of any task, every time the scheduler re-parses the file

### Scheduler & Executor Architecture

- The **scheduler** parses DAG files, determines which tasks are ready to run, and hands them to an **executor** (LocalExecutor, CeleryExecutor, KubernetesExecutor, etc.) which actually runs the task on a worker
- Scheduler performance is a shared resource across every DAG in the deployment — a single badly-behaved DAG (e.g., doing expensive work at parse time) can degrade scheduling latency for all other DAGs

### Idempotency as a Design Requirement

- Airflow's execution model assumes tasks **can and will be retried** (on failure, on manual rerun, during a backfill) — a task is not "done once," it's "done for this particular execution date, potentially more than once"
- A task that isn't idempotent (e.g., `INSERT` without a check for existing data, or an API call with a side effect that isn't safe to repeat) will produce duplicate or corrupted results the first time a retry or backfill actually happens

### XComs

- **XCom** (cross-communication) is Airflow's built-in mechanism for small pieces of data passed between tasks, stored in the metadata database
- XComs are meant for small values (IDs, file paths, short status strings), not for passing large datasets — doing so bloats the metadata database and can degrade overall Airflow performance well beyond just the DAG that misused it

### Sensors

- A **sensor** task polls for a condition (a file existing, an external job completing) and holds a worker slot while it waits, by default — this is fine at small scale but becomes a resource problem when many sensors are waiting simultaneously, since each consumes a worker slot doing essentially nothing but polling

## Best Practices

1. **Design every task to be idempotent** — assume it will run more than once for the same execution date at some point
2. **Keep DAG-file top-level code cheap** — no expensive I/O, API calls, or heavy computation outside of task functions; that code runs on every scheduler parse cycle
3. **Use XComs only for small values**; pass large data via external storage (object storage, a database) and pass a reference (path/ID) through XCom instead
4. **Use deferrable/"smart" sensors (or `reschedule` mode) instead of default `poke` mode** for long-wait sensors, to free the worker slot between checks
5. **Understand `catchup`/backfill behavior explicitly** before enabling a new DAG or changing its `start_date` — know what will actually run
6. **Set explicit retries, retry delays, and SLAs per task** rather than relying on defaults that may not fit the task's actual failure characteristics

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| A task that does a plain `INSERT` with no duplicate-check on retry | Design writes to be idempotent (upsert, delete-then-insert for the partition, or a dedup key) |
| Expensive computation or API calls at DAG-file top level | Move all real work inside task functions/operators; keep parse-time code minimal |
| Passing a large DataFrame or file content through XCom | Write the data to object storage/a database, pass only the reference through XCom |
| Default `poke` mode sensors holding worker slots for long waits | Use `mode="reschedule"` or deferrable operators for sensors with long wait times |
| Enabling a new DAG with `catchup=True` and an old `start_date` without expecting the consequences | Understand exactly what backfill will trigger before enabling; set `catchup=False` unless historical backfill is genuinely intended |

## Sharp Edges

### SE-1: Non-Idempotent Tasks Corrupting Data on Retry or Backfill
- **Severity**: critical
- **Situation**: A task that inserts records without a duplicate-check runs successfully, then gets retried (due to a transient failure after the insert but before Airflow marked it successful) or manually rerun/backfilled, and the same data gets inserted again — silently duplicating records
- **Cause**: Airflow's execution model treats retries and reruns as normal, expected operations, not exceptional ones — any task that isn't designed to produce the same end state regardless of how many times it runs for a given execution date will eventually be run more than once
- **Symptoms**:
  - Duplicate records appear in a downstream table, traceable to a specific DAG run that was retried or manually rerun
  - Data inconsistencies correlate with task retry counts in the Airflow UI
- **Solution**: Design every task's write path to be idempotent — use upserts, delete-and-reinsert for the specific partition/date being processed, or an explicit dedup key — and treat "what happens if this task runs twice for the same execution date" as a required design question for every task
- **Details**: → [extended/checklists.md#dag-design-checklist]

### SE-2: Expensive DAG-Parsing-Time Code Slowing the Entire Scheduler
- **Severity**: high
- **Situation**: A DAG file contains top-level code that makes a network call, queries a database, or does non-trivial computation to dynamically generate tasks — this code runs every time the scheduler re-parses the file (frequently, by default), and it slows down scheduling for every DAG in the deployment, not just the offending one
- **Cause**: Airflow's scheduler must parse every DAG file's Python code repeatedly to detect changes and determine the DAG structure — anything expensive at the top level (outside a task/operator) pays that cost on every parse cycle, and scheduler capacity is shared across all DAGs
- **Symptoms**:
  - Overall DAG scheduling latency increases across the whole Airflow deployment, not just for one DAG
  - Scheduler logs show high parse times attributable to a specific DAG file
- **Solution**: Move all expensive operations inside task functions (which run once per actual execution, not once per parse cycle), use Airflow's Variable/Connection caching appropriately instead of re-fetching at parse time, and monitor DAG parse duration as a standard operational metric

### SE-3: XCom Misuse Bloating the Metadata Database
- **Severity**: medium
- **Situation**: A task passes a large dataset (a DataFrame, a full file's contents, a large JSON blob) to a downstream task via XCom, and this pattern repeated across many DAG runs bloats the metadata database, degrading Airflow's overall performance (not just for that DAG)
- **Cause**: XCom values are stored in Airflow's metadata database, which is sized and tuned for small operational metadata, not for bulk data transfer — nothing prevents pushing a large value through XCom, but doing so treats the metadata DB as a data pipeline datastore it wasn't designed to be
- **Symptoms**:
  - Metadata database size grows disproportionately to the number of actual DAG runs
  - General Airflow UI/scheduler responsiveness degrades over time, traced back to metadata database bloat
- **Solution**: Pass only small references (file paths, record IDs, short status values) through XCom, and write/read actual data through object storage or a proper database, with the task doing the write passing only the resulting location through XCom

### SE-4: Sensor Tasks Starving the Worker Pool
- **Severity**: high
- **Situation**: Many sensor tasks are simultaneously waiting (in default `poke` mode) for external conditions, each holding a worker slot for the entire wait duration, and eventually there are no worker slots left for tasks that actually have real work to do
- **Cause**: Default `poke` mode sensors occupy a worker slot continuously while polling, rather than releasing it between checks — as sensor count grows (common in pipelines with many external dependencies), this scales poorly against a fixed worker pool size
- **Symptoms**:
  - Tasks queue and wait even though the Airflow UI shows "capacity" nominally available, because most of it is consumed by long-waiting sensors
  - Pipeline throughput degrades as the number of concurrently-waiting sensors grows, independent of actual workload volume
- **Solution**: Use `mode="reschedule"` (which releases the worker slot between poll attempts) or deferrable operators (which use a separate, more efficient triggerer process) for any sensor with a non-trivial expected wait time, reserving default `poke` mode only for sensors expected to resolve almost immediately

### SE-5: Unexpected Backfill Flood From Catchup Behavior
- **Severity**: medium
- **Situation**: A new DAG is enabled (or an existing DAG's `start_date` is changed) with `catchup=True` (Airflow's default in many versions/configurations), and Airflow immediately schedules and runs DAG runs for every interval between the `start_date` and now — potentially hundreds of historical runs firing at once, overwhelming downstream systems or racking up unexpected cost
- **Cause**: `catchup` behavior is designed for genuine backfill use cases, but it's easy to enable a DAG without realizing how far back its `start_date` is set, or without realizing `catchup` is even active
- **Symptoms**:
  - A newly enabled DAG immediately shows dozens or hundreds of DAG runs queued or running, far more than the "one run per new interval" a user might have expected
  - Downstream systems (APIs, databases) see a sudden burst of load correlated with a DAG being enabled
- **Solution**: Set `catchup=False` explicitly unless historical backfill is genuinely intended, and when backfill IS intended, run it deliberately via the `airflow dags backfill` command with an explicit, reviewed date range rather than relying on enabling-a-DAG side effects

## Recommended Tools

| Category | Tools |
|------|------|
| Local development | Astro CLI, Airflow's Breeze development environment |
| Managed Airflow | Astronomer, AWS MWAA, Google Cloud Composer |
| Monitoring | Airflow's own UI/metrics, StatsD/Prometheus exporters |
| Testing | `pytest`-based DAG validation, DAG integrity tests in CI |

## DAG Design Safety

**Full checklist**: → [extended/checklists.md#dag-design-checklist]

## Related Resources

[Airflow Docs](https://airflow.apache.org/docs/) | [Airflow Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html) | [Astronomer Airflow Guides](https://www.astronomer.io/docs/learn/)

## Related Domains

[[dbt]] | [[spark]] | [[python]] | [[postgresql]]
