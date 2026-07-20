# MongoDB — Extended Checklists

## Schema Design Checklist

- [ ] Every embedded array has a natural bound, or is restructured to a referenced collection before it can grow unbounded
- [ ] Schema design decisions (embed vs. reference) documented against actual access patterns, not just "what felt natural"
- [ ] `$jsonSchema` validation applied to collections where consistent document shape matters
- [ ] Frequently-joined data considered for denormalization if `$lookup` against it is on a hot path
- [ ] Document size monitored for collections with any embedded-array growth pattern

## Query & Write Safety Checklist

- [ ] Every field used in frequent `find`/`$match`/sort operations has a supporting index
- [ ] Compound indexes follow the ESR rule (Equality, Sort, Range)
- [ ] `explain("executionStats")` checked for any query suspected of being slow — confirm `IXSCAN`, not `COLLSCAN`
- [ ] `$lookup` foreign-collection join fields are indexed
- [ ] Aggregation pipelines filter (`$match`) as early as possible, before expensive stages
- [ ] Write concern set to `w: "majority"` for any write where losing data on primary failover is unacceptable
- [ ] Replica set failover tested (not just configured) to confirm actual behavior under primary loss
- [ ] Backup/restore process tested periodically for point-in-time recovery capability
