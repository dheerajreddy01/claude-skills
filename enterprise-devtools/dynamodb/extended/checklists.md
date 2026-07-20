# DynamoDB — Extended Checklists

## Data Modeling Checklist

- [ ] Application access patterns enumerated before table/key design
- [ ] Single-table design evaluated before defaulting to one table per entity type
- [ ] Partition key chosen for high cardinality and even real-world access distribution
- [ ] Write sharding (calculated suffix) considered for any naturally hot partition key
- [ ] Sort key structure supports the range/hierarchical queries the application actually needs
- [ ] GSIs treated as eventually consistent in application logic — no read-your-write assumption against a GSI
- [ ] Item size reviewed against the 400KB limit for any design with embedded lists/maps
- [ ] Large or unbounded content referenced externally (separate item or S3) rather than embedded

## Capacity & Cost Checklist

- [ ] Capacity mode (on-demand vs. provisioned) chosen based on actual traffic predictability, reviewed periodically as traffic patterns mature
- [ ] Auto-scaling configured with sufficient headroom if using provisioned capacity
- [ ] CloudWatch throttling metrics monitored and alerted on, per table and per GSI
- [ ] DAX evaluated for read-heavy workloads where caching would meaningfully reduce cost/latency
- [ ] Cost reviewed regularly against request volume trends to catch on-demand-vs-provisioned crossover points
- [ ] Backup strategy (point-in-time recovery or on-demand backups) configured and tested
