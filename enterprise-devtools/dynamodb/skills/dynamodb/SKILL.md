---
schema: "1.0"
name: dynamodb
version: "1.0.0"
description: DynamoDB single-table design, partition key strategy, GSI consistency, and capacity/cost practices
domain: technology
triggers:
  keywords:
    primary: [dynamodb, single table design, partition key, GSI]
    secondary: [on-demand capacity, DAX, sort key, throttling]
  context_boost: [serverless, aws, NoSQL key-value]
  context_penalty: [postgresql, mongodb, cassandra]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Amazon DynamoDB

> Model around access patterns, not entities — and denormalize like you mean it

## Applicable Scenarios

- Designing single-table DynamoDB schema around application access patterns
- Choosing partition/sort keys for even load distribution
- Deciding between Global Secondary Indexes and additional access-pattern modeling
- Choosing on-demand vs. provisioned capacity
- Diagnosing throttling or unexpectedly high cost

## Core Knowledge

### Single-Table Design

- DynamoDB is priced and optimized for a small number of tables serving many access patterns, not one table per entity type the way a relational schema would — the standard advanced pattern is **single-table design**: one table holding multiple entity types, distinguished by key structure (e.g., `PK: USER#123`, `SK: ORDER#456`)
- This is a deliberate inversion of relational normalization: you model the table around your application's actual query patterns first, then fit entities into that structure — not the other way around

### Partition Key & Sort Key

- The **partition key** determines which physical partition stores an item; DynamoDB distributes load based on hashing it — a partition key with low cardinality or skewed access concentrates traffic and triggers throttling on that partition regardless of overall table-level provisioned/on-demand capacity
- The **sort key** (optional) orders items within a partition and enables range queries (`begins_with`, `between`) — this is what allows related entities (e.g., a user and all their orders) to be efficiently queried together via a shared partition key

### Global Secondary Indexes (GSI)

- A GSI lets you query by an alternate key structure — but it's **eventually consistent** (unlike the base table, which supports strongly consistent reads) and has its **own provisioned/on-demand capacity**, separate from the base table
- A write to the base table propagates to GSIs asynchronously — an item just written may not be immediately visible in a GSI query

### Capacity Modes

| Mode | Behavior |
|------|------|
| **On-Demand** | Pay per request, auto-scales instantly — simple, but can be expensive at sustained high, predictable volume |
| **Provisioned** | Pre-allocated read/write capacity units, cheaper at steady predictable load, requires capacity planning and can throttle if traffic exceeds provisioned capacity (unless auto-scaling is configured with enough headroom) |

### Item Size Limit

- Every item (row) has a hard **400KB** size limit — this directly constrains schema choices like storing unbounded lists/maps inside a single item

## Best Practices

1. **Model around access patterns, not entities** — enumerate the actual queries your application needs before designing keys
2. **Choose partition keys for high cardinality and even access distribution** — avoid keys correlated with a small number of "hot" values
3. **Use composite sort keys** to enable range queries and hierarchical access patterns within a partition
4. **Treat GSIs as eventually consistent** — don't assume a just-written item is immediately visible via a GSI query
5. **Start with on-demand capacity for unpredictable/spiky workloads**, move to provisioned (with auto-scaling) once traffic patterns are well understood and steady enough to plan for
6. **Keep items well under the 400KB limit** — use references to a separate table or S3 for large/unbounded content rather than embedding it directly

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Designing a normalized multi-table schema like a relational database | Use single-table design modeled around access patterns |
| Low-cardinality or skewed partition key (e.g., a status flag) | Choose a key with enough cardinality to distribute load evenly |
| Assuming a GSI query reflects a write that just completed on the base table | Account for GSI eventual consistency in application logic, or query the base table directly when strong consistency is required |
| Unbounded lists/maps growing inside a single item | Reference a separate item/table (or S3) for unbounded content instead of embedding it |
| On-demand capacity left in place indefinitely for a high, steady, predictable workload | Evaluate provisioned capacity with auto-scaling once traffic patterns are well understood — often meaningfully cheaper |

## Sharp Edges

### SE-1: Hot Partition Throttling From Poor Key Choice
- **Severity**: critical
- **Situation**: A partition key with low cardinality or skewed access (e.g., partitioning by a date that all of today's traffic hits) causes DynamoDB to throttle requests against that specific partition, even though the table's overall provisioned/on-demand capacity looks sufficient in aggregate
- **Cause**: DynamoDB's capacity is distributed across partitions based on the partition key; a hot partition can exceed its per-partition throughput share even when total table capacity is nowhere near exhausted
- **Symptoms**:
  - `ProvisionedThroughputExceededException` (or on-demand throttling) despite low overall table-level consumption
  - Throttling concentrated on requests for a specific, identifiable subset of keys
- **Solution**: Choose partition keys with enough cardinality and even real-world distribution, or add a random/calculated suffix ("write sharding") to a naturally hot key to spread its traffic across multiple physical partitions
- **Details**: → [extended/checklists.md#data-modeling-checklist]

### SE-2: Normalized Multi-Table Schema Fighting DynamoDB's Model
- **Severity**: high
- **Situation**: A team designs DynamoDB tables the way they'd design a relational schema — one table per entity, expecting to join at query time — and discovers DynamoDB has no join operation, forcing multiple round-trip queries and application-side stitching for what would be a single query in a relational database
- **Cause**: DynamoDB's pricing and performance model rewards single-table design with denormalized, access-pattern-driven keys; a normalized schema assumes query flexibility DynamoDB doesn't provide
- **Symptoms**:
  - Common application queries require multiple sequential DynamoDB calls plus in-memory merging
  - Latency and cost scale poorly compared to what single-table design would achieve for the same access patterns
- **Solution**: Do access-pattern-first modeling before creating tables — enumerate every query the application needs, then design a key structure (often within a single table) that serves them directly

### SE-3: GSI Eventual Consistency Causing Invisible Just-Written Items
- **Severity**: high
- **Situation**: An application writes an item, then immediately queries a GSI expecting to find it, and doesn't — because GSI propagation is asynchronous and hadn't caught up yet
- **Cause**: GSIs are updated asynchronously after a base-table write; unlike the base table (which supports strongly consistent reads), GSIs only support eventually consistent reads
- **Symptoms**:
  - A "just created" item is missing from a GSI-backed list view immediately after creation, then appears moments later
  - The issue is intermittent and worse under higher write throughput
- **Solution**: Design around GSI eventual consistency explicitly — query the base table directly (strongly consistent) for read-your-write flows, or build the UI/application logic to tolerate a brief propagation delay for GSI-backed views

### SE-4: Unbounded Item Growth Hitting the 400KB Limit
- **Severity**: medium
- **Situation**: An item design that embeds a growing list or map (e.g., all of a user's activity history inside one item) eventually exceeds DynamoDB's 400KB per-item limit, and further writes to that item fail
- **Cause**: DynamoDB enforces a hard per-item size cap with no exception mechanism — a design that embeds unbounded data directly in an item will eventually hit it as that data grows
- **Symptoms**:
  - Writes to specific "hot"/long-lived items start failing with an item-size-limit error
  - Item size correlates with the age/activity of the entity (older/more active entities hit the limit first)
- **Solution**: Identify unbounded embedded collections at design time and restructure them as separate items (using a shared partition key with a differentiated sort key) or reference external storage (S3) for genuinely large content, rather than growing a single item indefinitely

### SE-5: On-Demand Capacity Cost Surprises at Sustained High Volume
- **Severity**: medium
- **Situation**: A workload that started with unpredictable, spiky traffic (a good fit for on-demand capacity) grows into a high, steady, predictable volume, and the on-demand per-request pricing ends up costing significantly more than provisioned capacity with auto-scaling would have
- **Cause**: On-demand pricing is convenient and simple but priced at a premium per request compared to provisioned capacity — the cost crossover point favors provisioned capacity once traffic is high and predictable enough to plan for
- **Symptoms**:
  - DynamoDB cost grows roughly linearly with request volume with no corresponding architecture change
  - Cost review shows on-demand DynamoDB as a disproportionately large line item relative to actual infrastructure needs
- **Solution**: Periodically review actual traffic patterns against capacity mode choice, and migrate steady, well-understood, high-volume tables to provisioned capacity with auto-scaling once the traffic profile justifies the operational overhead of capacity planning

## Recommended Tools

| Category | Tools |
|------|------|
| Data modeling | NoSQL Workbench for DynamoDB |
| Caching | DynamoDB Accelerator (DAX) |
| IaC | AWS CDK, CloudFormation, Terraform |
| Monitoring | CloudWatch DynamoDB metrics (throttling, consumed capacity) |

## Data Modeling

**Full checklist**: → [extended/checklists.md#data-modeling-checklist]

## Related Resources

[DynamoDB Docs](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) | [DynamoDB Single-Table Design](https://aws.amazon.com/blogs/compute/creating-a-single-table-design-with-amazon-dynamodb/) | [The DynamoDB Book](https://www.dynamodbbook.com/)

## Related Domains

[[aws]] | [[cassandra]] | [[mongodb]] | [[python]]
