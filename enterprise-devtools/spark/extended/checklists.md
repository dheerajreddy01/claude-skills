# Apache Spark — Extended Checklists

## Performance Checklist

- [ ] `explain()` reviewed for any job suspected of being slow, before guessing at a fix
- [ ] Data skew checked via Spark UI task duration distribution for `groupBy`/join keys
- [ ] Adaptive Query Execution (AQE) enabled where the Spark version/platform supports it
- [ ] `reduceByKey`/`aggregateByKey` used instead of `groupByKey` for aggregations
- [ ] No `.collect()`/`.toPandas()` call on a DataFrame that hasn't been reduced to a genuinely small size first
- [ ] Broadcast join candidates verified to actually be within the broadcast size threshold, not assumed
- [ ] Output partition count controlled via `coalesce()`/`repartition()` before writing, avoiding the small-file problem
- [ ] Shuffle read/write volume in the Spark UI checked against expected data volume for the operation

## Job Reliability Checklist

- [ ] Executor and driver memory sized based on actual workload profiling, not copied defaults
- [ ] Retries/checkpointing configured for long-running jobs where partial failure shouldn't mean starting over
- [ ] Skewed keys salted (or AQE skew handling enabled) for known-skewed join/groupBy operations
- [ ] Storage format (Parquet/Delta/Iceberg) chosen deliberately for the read/write pattern, not defaulted without consideration
- [ ] Job orchestration (Airflow or equivalent) has alerting on job duration regressions, not just failures
- [ ] Cluster autoscaling limits reviewed so a runaway job can't consume unbounded cluster resources
