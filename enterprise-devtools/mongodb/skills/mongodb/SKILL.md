---
schema: "1.0"
name: mongodb
version: "1.0.0"
description: MongoDB schema design, indexing, aggregation pipelines, and replica set/write concern practices
domain: technology
triggers:
  keywords:
    primary: [mongodb, mongo, document database, aggregation pipeline]
    secondary: [replica set, sharding, write concern, BSON, mongoose]
  context_boost: [nosql, flexible schema, JSON documents]
  context_penalty: [postgresql, mysql, relational]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# MongoDB

> Schema design is still a design decision — "schemaless" just moves it to query time

## Applicable Scenarios

- Designing document schema (embedding vs. referencing)
- Writing or optimizing aggregation pipelines
- Choosing indexes for actual query patterns
- Configuring replica sets and write/read concern for durability requirements
- Diagnosing unbounded document growth or slow queries

## Core Knowledge

### Schema Design: Embed vs. Reference

| Approach | Use When | Risk |
|------|------|------|
| **Embed** | Data is always accessed together, bounded in size (e.g., an order's line items) | Unbounded embedded arrays can hit the 16MB document size limit |
| **Reference** | Data is large, grows unboundedly, or is accessed independently (e.g., a user's posts) | Requires a `$lookup` (join-like) or multiple round trips |

MongoDB's flexibility doesn't remove the need for schema design — it moves the decision from "what does the table look like" to "what does the document look like and where do relationships live," which is just as consequential for performance and correctness.

### Indexing

- Every query pattern needs an index unless the collection is genuinely tiny — MongoDB doesn't magically avoid the same collection-scan-at-scale problem relational databases have
- Compound indexes should follow the **ESR rule**: Equality fields first, then Sort fields, then Range fields, for the most effective index usage
- `explain("executionStats")` shows whether a query used an index (`IXSCAN`) or fell back to a full collection scan (`COLLSCAN`)

### Aggregation Pipeline

- A sequence of stages (`$match`, `$group`, `$sort`, `$lookup`, `$project`, etc.) that transform documents, conceptually similar to SQL's `WHERE`/`GROUP BY`/`JOIN`/`SELECT`
- **Filter early**: put `$match` as early as possible in the pipeline so later stages process fewer documents — the same principle as filtering-first in any query language
- `$lookup` (the closest thing to a join) doesn't scale as gracefully as a well-indexed relational join, especially against large foreign collections — it's often better to denormalize (embed) frequently-joined data instead

### Replication & Write Concern

| Write Concern | Guarantee |
|------|------|
| `w: 1` (default) | Acknowledged once the primary has written it |
| `w: "majority"` | Acknowledged once a majority of replica set members have written it — survives a primary failure without data loss |
| `w: 0` | Fire-and-forget, no acknowledgment |

A replica set provides high availability (automatic failover if the primary goes down), but write durability depends entirely on the write concern used — `w: 1` can lose the most recent writes if the primary fails before replicating them.

## Best Practices

1. **Design the schema around actual access patterns**, not around normalizing data "because that's correct" — MongoDB rewards denormalization for frequently-together data
2. **Index every field used in `$match`/`sort`/`find` filters** that runs against a non-trivial collection size
3. **Filter (`$match`) as early as possible** in aggregation pipelines
4. **Use `w: "majority"` write concern** for anything where losing the most recent write on failover is unacceptable
5. **Cap unbounded arrays** (or restructure to a referenced collection) before they risk the 16MB document limit
6. **Enable schema validation** (`$jsonSchema` validator) for collections where consistent document shape actually matters, rather than relying on application-layer discipline alone

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Embedding an unbounded array (e.g., all of a user's activity log) in one document | Reference a separate collection once the array can grow without bound |
| No index on fields used in frequent queries | Add indexes matching actual query patterns; verify with `explain()` |
| `$lookup` against a large, unindexed foreign collection in a hot path | Index the foreign collection's join field, or denormalize the frequently-needed data instead |
| Relying on default `w: 1` write concern for critical financial/inventory writes | Use `w: "majority"` where losing a just-acknowledged write on failover isn't acceptable |
| No schema validation, leading to inconsistent document shapes across records over time | Use `$jsonSchema` validation for collections where consistency genuinely matters |

## Sharp Edges

### SE-1: Unbounded Document Growth Hitting the 16MB Limit
- **Severity**: high
- **Situation**: A document with an embedded array that grows indefinitely (e.g., appending every comment or event to a parent document) eventually hits MongoDB's hard 16MB document size limit, and further writes to that document fail
- **Cause**: Embedding is a good choice for bounded, always-together data, but an array with no natural cap keeps growing as long as the application keeps appending to it
- **Symptoms**:
  - Writes to specific "hot" documents start failing with a document-size-limit error
  - Document size grows correlated with the age/activity of the parent entity (older users/orders hit the limit first)
- **Solution**: Identify unbounded embedded arrays at design time and either cap them (e.g., keep only the most recent N, archive the rest) or restructure to a referenced child collection before the limit becomes a production incident
- **Details**: → [extended/checklists.md#schema-design-checklist]

### SE-2: Missing Indexes Causing Collection Scans at Scale
- **Severity**: high
- **Situation**: A query that filters or sorts on a field with no supporting index works fine on a small development dataset but becomes a major bottleneck as the collection grows in production
- **Cause**: Without a matching index, MongoDB falls back to scanning every document in the collection (`COLLSCAN`) to satisfy the query
- **Symptoms**:
  - `explain("executionStats")` shows `COLLSCAN` and a `totalDocsExamined` far larger than `nReturned`
  - Query latency scales roughly linearly with collection size
- **Solution**: Add indexes matching actual query filter/sort fields, follow the ESR rule for compound indexes, and verify index usage with `explain()` rather than assuming an index is being used

### SE-3: Write Concern `w: 1` Losing Data on Failover
- **Severity**: critical
- **Situation**: A write is acknowledged to the application under the default `w: 1` write concern, the primary fails before that write replicates to any secondary, and a new primary is elected without that write — the acknowledged write is silently gone
- **Cause**: `w: 1` only requires the primary to persist the write locally before acknowledging; it says nothing about replication to other members, so a primary failure in that narrow window loses the write
- **Symptoms**:
  - A write the application believes succeeded (got an acknowledgment) is missing after a failover event
  - Data loss incidents correlate with replica set primary elections
- **Solution**: Use `w: "majority"` write concern for any write where this kind of loss is unacceptable (financial transactions, inventory, anything with real business consequence), accepting the added latency as the cost of that durability guarantee

### SE-4: Inefficient `$lookup` at Scale
- **Severity**: medium
- **Situation**: An aggregation pipeline using `$lookup` to join against a large, unindexed foreign collection becomes a major performance bottleneck as both collections grow
- **Cause**: `$lookup` performs, in effect, a lookup per document against the foreign collection — without an index on the foreign collection's join field, each of those lookups can be a collection scan
- **Symptoms**:
  - Aggregation pipeline queries with `$lookup` show disproportionately high `executionStats` time compared to pipelines without it
- **Solution**: Ensure the foreign collection has an index on the field used in `$lookup`'s `foreignField`, filter (`$match`) both collections as early as possible before the join, or denormalize frequently-joined data into the primary document if the join happens on every hot-path query

### SE-5: Schema Drift From No Validation
- **Severity**: medium
- **Situation**: Over time, documents in the same collection accumulate inconsistent shapes — some have a field as a string, others as a number, some are missing fields entirely — because nothing enforced a consistent schema, and application code written at different times made different assumptions
- **Cause**: MongoDB doesn't enforce a schema by default; without explicit `$jsonSchema` validation, any document shape can be inserted, and inconsistency accumulates silently as application code evolves
- **Symptoms**:
  - Application code has defensive type-checking/coercion scattered everywhere to handle "documents that don't look like they should"
  - A query or aggregation stage fails or produces wrong results only for a subset of documents with an unexpected shape
- **Solution**: Add `$jsonSchema` validation to collections where consistent shape matters, and backfill/migrate existing inconsistent documents deliberately rather than letting application code silently paper over the drift

## Recommended Tools

| Category | Tools |
|------|------|
| Query analysis | `explain("executionStats")`, MongoDB Compass |
| Schema validation | `$jsonSchema` collection validators |
| Monitoring | MongoDB Atlas Performance Advisor, `mongostat`/`mongotop` |
| Migrations | Mongock, custom migration scripts |

## Schema & Write Safety

**Full checklist**: → [extended/checklists.md#schema-design-checklist]

## Related Resources

[MongoDB Docs](https://www.mongodb.com/docs/) | [MongoDB Schema Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary) | [MongoDB University](https://learn.mongodb.com/)

## Related Domains

[[postgresql]] | [[redis]] | [[javascript]] | [[python]]
