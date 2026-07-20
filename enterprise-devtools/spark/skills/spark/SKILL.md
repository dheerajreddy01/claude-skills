---
schema: "1.0"
name: spark
version: "1.0.0"
description: Apache Spark partitioning, shuffle performance, memory model, and distributed job reliability practices
domain: technology
triggers:
  keywords:
    primary: [spark, apache spark, pyspark, spark job]
    secondary: [shuffle, partition, executor, driver, catalyst optimizer, broadcast join]
  context_boost: [big data, distributed processing, data pipeline]
  context_penalty: [dbt, airflow, single-machine processing]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Apache Spark

> Lazy evaluation and partitioning decide performance long before any action actually runs

## Applicable Scenarios

- Designing or optimizing Spark jobs (batch or structured streaming)
- Diagnosing data skew, slow shuffles, or straggler tasks
- Choosing join strategies (broadcast vs shuffle) for large-scale joins
- Debugging driver out-of-memory errors
- Reviewing output file layout to avoid the small-file problem

## Core Knowledge

### Lazy Evaluation: Transformations vs. Actions

- **Transformations** (`map`, `filter`, `join`, `groupBy`) build up a logical execution plan but don't compute anything
- **Actions** (`collect`, `count`, `write`) trigger actual execution — Spark's Catalyst optimizer plans the whole DAG of transformations at that point, not incrementally as each transformation is called
- `df.explain()` shows the physical execution plan — essential for understanding why a job is slow, similar in spirit to `EXPLAIN` in a SQL database

### Partitioning & Shuffles

- Data is split into **partitions**, processed in parallel across **executors** — the number and size of partitions determines the actual parallelism achieved
- A **shuffle** (triggered by operations like `groupBy`, `join`, `repartition`) redistributes data across the cluster over the network — it's the single most expensive operation type in Spark, and minimizing unnecessary shuffles is most of what "Spark performance tuning" actually means
- `reduceByKey`/`aggregateByKey` combine values locally before shuffling (partial aggregation); `groupByKey` shuffles all raw values first — the same logical result, very different network cost

### Driver vs. Executor Memory

- The **driver** coordinates the job and holds the final result of actions like `collect()` — it is not designed to hold large datasets
- **Executors** do the actual distributed processing and hold partition data in memory/disk during execution
- Calling `.collect()` (or similarly `.toPandas()`) on a large DataFrame pulls all that data to the driver's single-machine memory, regardless of how well-distributed the computation itself was

### Joins

| Join Strategy | Use When |
|------|------|
| **Broadcast join** | One side is small enough to fit in executor memory — sent to every executor, no shuffle needed |
| **Shuffle (sort-merge) join** | Both sides are large — requires a shuffle to co-locate matching keys |

Spark can auto-broadcast small tables below a size threshold, but broadcasting a table that turns out to be larger than expected (e.g., due to unfiltered growth) causes memory pressure instead of the intended speedup.

## Best Practices

1. **Read `explain()` output** before assuming you know why a job is slow
2. **Minimize shuffles** — prefer `reduceByKey` over `groupByKey`, and avoid unnecessary `repartition()` calls
3. **Never `.collect()` a large DataFrame to the driver** — write results distributed (to storage) or aggregate down to a genuinely small summary first
4. **Watch for data skew** — a few very large keys can create straggler tasks that dominate total job time
5. **Control output file count** — too many small files degrades downstream read performance; use `coalesce()`/`repartition()` deliberately on write
6. **Set broadcast join thresholds deliberately**, and verify a table intended for broadcast is actually small before relying on auto-broadcast

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `groupByKey` for a simple aggregation | Use `reduceByKey`/`aggregateByKey` for partial local aggregation before shuffling |
| `.collect()` on a large DataFrame | Write results to storage distributed, or aggregate to a small summary before collecting |
| Ignoring data skew in join/groupBy keys | Salt skewed keys or use adaptive query execution (AQE) to handle skew automatically |
| Writing output with default partition count, producing thousands of tiny files | `coalesce()`/`repartition()` to a sensible file count before writing |
| Assuming a table will auto-broadcast without checking its actual size | Verify size against the broadcast threshold, or force/disable broadcast explicitly |

## Sharp Edges

### SE-1: Data Skew Creating a Straggler Task
- **Severity**: critical
- **Situation**: A `groupBy` or join on a key with a highly uneven distribution (e.g., one customer ID representing 40% of all rows) sends a disproportionate amount of data to a single partition/task, and the entire job waits on that one task long after every other task has finished
- **Cause**: Spark partitions by key hash for shuffle operations — if the key distribution is skewed, partition sizes are skewed too, and total job time is bounded by the slowest (largest) partition, not the average
- **Symptoms**:
  - The Spark UI shows one or a few tasks in a stage taking dramatically longer than the rest
  - Job duration doesn't improve proportionally when adding more executors
- **Solution**: Enable Adaptive Query Execution (AQE) skew join optimization if available, or manually salt the skewed key (add a random suffix to split it across more partitions, then aggregate the salted results), and identify skew early via the Spark UI's task duration distribution
- **Details**: → [extended/checklists.md#performance-checklist]

### SE-2: Driver OOM From Collecting Too Much Data
- **Severity**: critical
- **Situation**: A job that processes terabytes of data across the cluster fine crashes with a driver out-of-memory error the moment `.collect()` or `.toPandas()` is called on a DataFrame that turned out to be much larger than expected
- **Cause**: `.collect()`/`.toPandas()` pull the entire result set into the driver's single-JVM (or single-process) memory — the distributed processing that worked fine across the cluster doesn't apply to this step at all
- **Symptoms**:
  - `java.lang.OutOfMemoryError` on the driver, not the executors
  - The failure happens specifically at a `.collect()`/`.toPandas()`/`.show()`-with-large-n call
- **Solution**: Avoid collecting large result sets to the driver — write distributed to storage instead, or aggregate/sample down to a genuinely small result before collecting; if a Pandas DataFrame is truly needed, ensure the data has actually been reduced to a size that fits in driver memory first

### SE-3: Unnecessary Shuffles From Avoidable Operations
- **Severity**: high
- **Situation**: A job spends most of its wall-clock time in shuffle stages that could have been avoided or reduced, because the code used a shuffle-heavy operation where a cheaper equivalent existed
- **Cause**: Operations like `groupByKey` (shuffles all raw values) or an unnecessary `repartition()` before a join trigger full-data network shuffles that a partial-aggregation-first approach (`reduceByKey`) or better partition planning could avoid
- **Symptoms**:
  - The Spark UI shows shuffle read/write volume disproportionate to the actual data volume needed for the result
  - Job runtime is dominated by shuffle stages rather than compute stages
- **Solution**: Prefer `reduceByKey`/`aggregateByKey` over `groupByKey`, avoid redundant `repartition()` calls, and use `explain()` to check whether a shuffle is actually structurally necessary for the operation being performed

### SE-4: The Small-File Problem From Uncontrolled Output Partitioning
- **Severity**: high
- **Situation**: A job writes output with a very large number of small files (one per partition, and the job had far more partitions than the data volume warranted), and every downstream job/query reading that output pays a large per-file overhead cost, plus metastore/filesystem strain
- **Cause**: Spark writes one output file per partition by default; if upstream processing left the DataFrame with many small partitions (common after heavy filtering or a wide shuffle), the write step inherits that same excessive partition count
- **Symptoms**:
  - Downstream jobs reading this output are slower than the data volume alone would suggest
  - Storage/metastore shows an unexpectedly large file count for a modest total data size
- **Solution**: Use `coalesce()` (cheaper, no shuffle) or `repartition()` (shuffles, but can also fix skew) to control output partition count deliberately before writing, targeting a reasonable per-file size for the storage format in use

### SE-5: Broadcast Join Misfire on an Underestimated Table Size
- **Severity**: medium
- **Situation**: A table assumed to be "small" (and either auto-broadcast or explicitly hinted for broadcast) turns out to have grown past the point where broadcasting it is actually cheap, and every executor now holds a full copy of a much larger-than-expected dataset in memory
- **Cause**: Broadcast joins trade shuffle cost for the cost of replicating the small side to every executor — if that "small side" isn't actually small, this replication becomes a bigger memory burden than the shuffle join it was meant to avoid
- **Symptoms**:
  - Executor OOM errors or severe GC pressure correlated with a join that previously ran fine
  - The issue appears only after the "small" table's underlying source grew past a threshold
- **Solution**: Periodically verify that tables hinted or relied upon for auto-broadcast are still actually within the broadcast size threshold, and prefer explicit `broadcast()` hints with size monitoring over blind reliance on auto-broadcast heuristics for tables that could grow over time

## Recommended Tools

| Category | Tools |
|------|------|
| Debugging/tuning | Spark UI, `explain()`, Adaptive Query Execution (AQE) |
| Orchestration | Airflow, Databricks Jobs |
| Storage formats | Parquet, Delta Lake, Iceberg |
| Development | Databricks notebooks, PySpark, Spark SQL |

## Performance

**Full checklist**: → [extended/checklists.md#performance-checklist]

## Related Resources

[Spark Docs](https://spark.apache.org/docs/latest/) | [Spark: The Definitive Guide](https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/) | [Databricks Performance Tuning Guide](https://docs.databricks.com/en/optimizations/index.html)

## Related Domains

[[dbt]] | [[airflow]] | [[snowflake]] | [[clickhouse]]
