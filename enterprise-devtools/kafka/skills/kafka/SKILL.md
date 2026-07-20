---
schema: "1.0"
name: kafka
version: "1.0.0"
description: Kafka topic/partition design, producer/consumer delivery semantics, and consumer group reliability practices
domain: technology
triggers:
  keywords:
    primary: [kafka, topic, partition, consumer group, broker]
    secondary: [offset, acks, ISR, rebalance, idempotent producer]
  context_boost: [event streaming, log-based messaging, data pipeline]
  context_penalty: [rabbitmq, sqs, redis pub/sub]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Kafka

> Partitions are the unit of parallelism and ordering — get the count and key strategy right early

## Applicable Scenarios

- Designing topic/partition strategy and choosing a partition key
- Configuring producer acknowledgment and consumer commit semantics for the required delivery guarantee
- Diagnosing consumer lag, rebalance storms, or duplicate/missing message processing
- Deciding retention and storage strategy for a topic
- Reviewing exactly-once/idempotent producer configuration

## Core Knowledge

### Topics & Partitions

- A **topic** is split into **partitions**, each an ordered, append-only log — ordering is guaranteed *within* a partition, never across partitions of the same topic
- Partition count is the ceiling on consumer parallelism within a consumer group — a topic with 4 partitions can have at most 4 consumers in one group actively consuming (the rest sit idle)
- **Partition count can be increased later, but not decreased** — and increasing it changes which partition a given key hashes to, breaking the ordering guarantee for existing keys' *future* messages relative to their past ones

### Replication & ISR

- Each partition has a **leader** (handles all reads/writes) and **followers** replicating it; the set of followers caught up enough to be eligible for leader election is the **In-Sync Replica (ISR)** set
- `min.insync.replicas` sets how many replicas (including the leader) must acknowledge a write for it to be considered durably committed when using `acks=all`

### Producer Acknowledgment

| `acks` Setting | Guarantee |
|------|------|
| `0` | Fire-and-forget — no acknowledgment, highest throughput, highest loss risk |
| `1` | Leader acknowledges after its own write — lost if the leader fails before followers replicate |
| `all` (`-1`) | Acknowledges after `min.insync.replicas` replicas have the write — strongest durability |

Combining `acks=all` with `min.insync.replicas >= 2` (and replication factor ≥ 3) is the standard pattern for not losing acknowledged writes on a single broker failure.

### Consumer Groups & Offsets

- Consumers in the same **consumer group** split partitions among themselves — each partition is consumed by exactly one consumer in the group at a time
- **Offset commits** track how far a consumer has processed; auto-commit (`enable.auto.commit=true`) commits on a timer, independent of whether processing actually completed — this can commit an offset for a message that was received but not yet (or never) successfully processed
- **Rebalancing** reassigns partitions among group members when a consumer joins/leaves/times out — a rebalance pauses consumption for the affected partitions until reassignment completes

### Retention

- Kafka retains messages for a configured time/size window (`retention.ms`/`retention.bytes`) regardless of whether they've been consumed — Kafka isn't a queue that deletes on ack, it's a log that ages out
- **Compacted topics** (`cleanup.policy=compact`) retain only the latest value per key indefinitely, useful for changelog/state-restoration use cases rather than event history

## Best Practices

1. **Choose partition count based on target consumer parallelism**, with headroom for future scale — decide early, since decreasing isn't possible
2. **Use `acks=all` with `min.insync.replicas >= 2`** for anything where losing an acknowledged write is unacceptable
3. **Commit offsets only after successful processing** (manual commit), not on an auto-commit timer that's decoupled from actual completion
4. **Choose a partition key that distributes load evenly** — a low-cardinality or skewed key concentrates traffic on a few partitions
5. **Keep consumer processing time within `max.poll.interval.ms`**, or offload slow work asynchronously, to avoid rebalance storms
6. **Use the idempotent producer** (`enable.idempotence=true`) to avoid duplicate writes from producer-side retries

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `acks=1` (or `0`) for data where loss is unacceptable | Use `acks=all` with `min.insync.replicas >= 2` |
| Auto-commit offsets on a timer, decoupled from processing completion | Manually commit offsets only after the message is fully and successfully processed |
| Decreasing partition count to "clean up" | Not possible — plan partition count with future growth in mind from the start |
| Slow, synchronous per-message processing inside the consumer poll loop | Keep the poll loop fast; offload slow work to a separate async pipeline, or tune `max.poll.interval.ms`/`max.poll.records` deliberately |
| Using a low-cardinality or predictable partition key (e.g., a boolean flag) | Choose a key with enough cardinality to spread load evenly across partitions |

## Sharp Edges

### SE-1: Silent Data Loss From Weak Acknowledgment Settings
- **Severity**: critical
- **Situation**: A producer using `acks=1` (or `acks=0`) has a message acknowledged by the partition leader, the leader then fails before any follower replicates the write, and the message is permanently lost even though the producer received a successful acknowledgment
- **Cause**: `acks=1` only requires the leader's own local write, with no guarantee any replica has it yet — if the leader is the only broker that ever had the message, its failure loses that message entirely
- **Symptoms**:
  - Downstream consumers report gaps in expected message sequences
  - Data loss correlates with broker failures/restarts, not application-level errors
- **Solution**: Use `acks=all` combined with `min.insync.replicas >= 2` (and a replication factor of at least 3) for any topic where losing an acknowledged write is unacceptable, accepting the latency tradeoff as the cost of that durability
- **Details**: → [extended/checklists.md#reliability-checklist]

### SE-2: Auto-Commit Causing Silent Message Loss on Crash
- **Severity**: high
- **Situation**: A consumer with `enable.auto.commit=true` receives a batch of messages, auto-commit fires on its timer before processing finishes, and the consumer crashes mid-processing — on restart, it resumes from the committed offset, skipping messages it never actually finished processing
- **Cause**: Auto-commit is time-based, not tied to actual processing completion — it commits "I've received up to here," not "I've successfully processed up to here"
- **Symptoms**:
  - Specific messages are provably never fully processed (missing side effects) despite the consumer group's offset having moved past them
  - Data loss correlates with consumer restarts/crashes, not steady-state operation
- **Solution**: Disable auto-commit and manually commit offsets only after a message (or batch) has been fully and successfully processed, accepting at-least-once semantics with idempotent processing downstream rather than accidentally getting weaker guarantees from auto-commit timing

### SE-3: Increasing Partition Count Breaking Key-Based Ordering
- **Severity**: high
- **Situation**: A topic's partition count is increased to scale consumer parallelism, and messages for the same key that previously landed on one partition (preserving order) now sometimes hash to a different partition — a consumer processing the "new" partition can see a message for a key out of order relative to that key's older messages still queued on the original partition
- **Cause**: Kafka's default partitioner hashes the key modulo partition count — changing the partition count changes the hash-to-partition mapping for existing keys, and Kafka doesn't rebalance/migrate existing data to preserve per-key ordering across the change
- **Symptoms**:
  - A consumer sees an out-of-order sequence for a specific key shortly after a partition count increase
  - Bugs that depend on strict per-key ordering surface only around the time of a partition scaling change
- **Solution**: Provision partition count for expected future scale from the start rather than increasing it later on a topic with strict per-key ordering requirements; if a partition increase is unavoidable, understand and communicate that ordering guarantees only hold prospectively from the change, not retroactively

### SE-4: Consumer Group Rebalance Storms From Slow Processing
- **Severity**: high
- **Situation**: A consumer's per-poll-loop processing occasionally takes longer than `max.poll.interval.ms`, the group coordinator considers it dead, triggers a rebalance, and the newly-reassigned consumer for that partition reprocesses messages the "dead" consumer was still working on (or the situation repeats immediately) — this can cascade into frequent, disruptive rebalances
- **Cause**: Kafka's consumer liveness is determined by how promptly `poll()` is called, not by a separate heartbeat thread being alive — synchronous, slow processing inside the poll loop directly risks triggering "consumer is dead" rebalances even though the process is healthy and simply busy
- **Symptoms**:
  - Frequent rebalance events visible in consumer group logs/metrics, correlated with processing time spikes
  - Duplicate processing of messages around rebalance events
- **Solution**: Keep the poll loop itself fast — offload slow per-message work to a separate thread/async pipeline and only call `poll()` again once ready, tune `max.poll.records` down so each batch fits comfortably within `max.poll.interval.ms`, and monitor rebalance frequency as a standard operational signal

### SE-5: Hot Partitions From Poor Key Distribution
- **Severity**: medium
- **Situation**: A partition key with low cardinality or a skewed value distribution (e.g., a tenant ID where one tenant dominates traffic) concentrates most messages onto a small number of partitions, and the consumers assigned those partitions become bottlenecks while others sit comparatively idle
- **Cause**: Kafka distributes messages to partitions based on the key's hash — if the key values themselves are unevenly distributed in the real traffic, the resulting partition load is unevenly distributed too, regardless of partition count
- **Symptoms**:
  - Consumer lag metrics show a small number of partitions consistently behind while others are caught up
  - Overall group throughput is bottlenecked by a subset of partitions/consumers
- **Solution**: Choose partition keys with high enough cardinality and even enough real-world distribution to spread load, or use a composite/salted key for known-skewed dimensions, accepting the tradeoff against strict per-original-key ordering if salting is applied

## Recommended Tools

| Category | Tools |
|------|------|
| Cluster management | Kafka Manager/CMAK, Cruise Control, Confluent Control Center |
| Monitoring | Burrow/consumer lag exporters, Prometheus JMX exporter |
| Schema management | Confluent Schema Registry (Avro/Protobuf) |
| Stream processing | Kafka Streams, ksqlDB, Flink |

## Reliability

**Full checklist**: → [extended/checklists.md#reliability-checklist]

## Related Resources

[Kafka Docs](https://kafka.apache.org/documentation/) | [Confluent Kafka Guides](https://developer.confluent.io/) | [Designing Event-Driven Systems (O'Reilly)](https://www.confluent.io/designing-event-driven-systems/)

## Related Domains

[[rabbitmq]] | [[elasticsearch]] | [[java]] | [[go]]
