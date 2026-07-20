---
schema: "1.0"
name: rabbitmq
version: "1.0.0"
description: RabbitMQ exchange/queue topology, delivery guarantees, and broker reliability practices
domain: technology
triggers:
  keywords:
    primary: [rabbitmq, message queue, AMQP, exchange, queue]
    secondary: [publisher confirm, dead-letter, prefetch, binding, durable]
  context_boost: [async processing, event-driven, worker queue]
  context_penalty: [kafka, sqs, redis pub/sub]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# RabbitMQ

> Durability and acknowledgment are opt-in — a queue that "just works" in dev can quietly drop messages in production

## Applicable Scenarios

- Designing exchange/queue/binding topology for a messaging system
- Choosing delivery guarantees (at-most-once vs. at-least-once) and configuring for them
- Setting up dead-letter exchanges for poison message handling
- Diagnosing message loss, unbounded queue growth, or unfair consumer dispatch
- Configuring publisher confirms and consumer acknowledgment modes

## Core Knowledge

### Exchanges & Routing

| Exchange Type | Routing Behavior |
|------|------|
| **Direct** | Routes to queues bound with an exact matching routing key |
| **Topic** | Routes based on pattern matching on the routing key (`order.*.created`) |
| **Fanout** | Broadcasts to every bound queue, ignoring routing key |
| **Headers** | Routes based on message header attributes rather than routing key |

Producers publish to an **exchange**, never directly to a queue — the exchange, via its **bindings**, determines which queue(s) actually receive the message. A message published to an exchange with no matching binding is silently dropped unless the exchange is configured with an alternate-exchange fallback.

### Durability

- **Durable queues/exchanges** survive a broker restart (the queue/exchange definition persists) — non-durable ones are recreated empty
- **Persistent messages** (delivery mode 2) are written to disk, surviving a broker restart — transient messages are not, even if published to a durable queue
- Both need to be configured together for genuine durability: a durable queue with non-persistent messages still loses those messages on restart

### Delivery Guarantees

| Mechanism | Guarantee |
|------|------|
| **Publisher confirms** | Broker acknowledges receipt of a published message — without this, publishing is fire-and-forget over TCP, and a broker-side failure can silently drop a message the application believes was sent |
| **Consumer acknowledgment (manual ack)** | Consumer explicitly acks after successfully processing — the broker redelivers if the consumer disconnects before acking |
| **Auto-ack** | Message is considered delivered the instant it's sent to the consumer, before processing — a crash mid-processing loses the message with no redelivery |

### Prefetch (QoS)

- `basic.qos(prefetch_count=N)` limits how many unacknowledged messages a consumer can have outstanding at once
- Without a prefetch limit, RabbitMQ can dispatch messages to a consumer far faster than it can process them (round-robin dispatch doesn't account for actual consumer speed), or one slow consumer can hold an unbounded number of unacked messages

### Dead-Letter Exchanges (DLX)

- A queue can be configured with a **dead-letter exchange** — messages that are rejected, expire (TTL), or exceed a max-length policy get republished there instead of being lost or endlessly retried
- Without a DLX, a message a consumer can never successfully process (a "poison message") gets requeued and redelivered indefinitely if the consumer keeps nacking/requeuing it, consuming resources in an infinite loop

## Best Practices

1. **Enable publisher confirms** for anything where message loss on publish is unacceptable
2. **Use manual consumer acknowledgment**, not auto-ack, for anything where losing a message mid-processing matters
3. **Configure both durable queues/exchanges AND persistent messages** together — one without the other doesn't give real durability
4. **Set a prefetch count** appropriate to consumer processing capacity — never leave it unbounded
5. **Configure a dead-letter exchange** on every queue where a poison message is plausible, so failures are visible and handled rather than looping silently
6. **Set queue length limits or TTLs** to bound memory usage — an unbounded queue can exhaust broker memory if consumers fall behind

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Publishing without confirms and assuming the message arrived | Enable publisher confirms and handle the negative-ack/timeout case explicitly |
| Auto-ack consumers for critical processing | Use manual ack, acknowledging only after successful processing |
| Durable queue with transient (non-persistent) messages | Set delivery mode to persistent for messages that must survive a restart |
| No dead-letter exchange, letting failed messages requeue forever | Configure a DLX so poison messages are captured and visible, not endlessly retried |
| Unbounded prefetch, one consumer hoarding all outstanding messages | Set an explicit, capacity-appropriate `prefetch_count` |

## Sharp Edges

### SE-1: Message Loss From Fire-and-Forget Publishing
- **Severity**: critical
- **Situation**: An application publishes a message and considers it "sent" as soon as the TCP write succeeds, but the broker never actually received or routed it (network blip, broker restart, no matching binding) — the message is gone with no error surfaced to the application
- **Cause**: Without publisher confirms, publishing over AMQP is asynchronous and unacknowledged by default — a successful client-side call doesn't guarantee broker-side receipt
- **Symptoms**:
  - Downstream consumers report missing events/messages with no corresponding error in the publisher's logs
  - Message loss correlates with broker restarts or network instability, not application errors
- **Solution**: Enable publisher confirms and handle the negative-acknowledgment/timeout case explicitly (retry, alert, or dead-letter on the publish side), and additionally verify the exchange/routing key combination actually has a matching binding (or configure an alternate-exchange fallback)
- **Details**: → [extended/checklists.md#reliability-checklist]

### SE-2: Non-Durable Configuration Losing Everything on Restart
- **Severity**: critical
- **Situation**: A broker restart (planned maintenance or unplanned crash) wipes out queued messages that were assumed to be safely persisted
- **Cause**: Queue durability and message persistence are two separate settings that both need to be configured — a durable queue definition survives restart, but if messages within it were published as transient (delivery mode 1), the messages themselves are lost even though the empty queue reappears
- **Symptoms**:
  - Queues exist after a restart but are empty, despite having contained unprocessed messages before
- **Solution**: Configure both durable queues/exchanges and persistent message delivery mode together for anything where message loss on restart is unacceptable, and test this explicitly (restart a non-production broker and verify messages survive) rather than assuming it works

### SE-3: Poison Messages Looping Without a Dead-Letter Exchange
- **Severity**: high
- **Situation**: A message that a consumer can never successfully process (malformed payload, a bug triggered only by that message's content) gets nacked and requeued repeatedly, consuming consumer and broker resources in an infinite retry loop while never actually resolving
- **Cause**: Without a dead-letter exchange, a rejected-and-requeued message just goes back to the same queue for the same (or another) consumer to fail on again — nothing routes it out of the retry loop
- **Symptoms**:
  - Consumer logs show the same message ID/content being processed (and failing) repeatedly
  - Queue throughput degrades because consumers spend cycles repeatedly failing on the same stuck message
- **Solution**: Configure a dead-letter exchange on every queue, so messages exceeding a retry count, TTL, or explicitly rejected without requeue get routed to a DLX for visibility and manual/automated handling, rather than looping indefinitely in the primary queue

### SE-4: Unbounded Queue Growth Exhausting Broker Memory
- **Severity**: high
- **Situation**: Consumers fall behind (or stop entirely) while producers keep publishing, and the queue grows without bound until the broker hits its memory high watermark and starts blocking publishers (or, in worse misconfigurations, crashes)
- **Cause**: Without a queue length limit, TTL, or lazy queue mode, RabbitMQ will keep accepting and holding messages in memory as long as resources allow, with no automatic backpressure until the broker-wide memory alarm triggers
- **Symptoms**:
  - Broker memory usage climbs correlated with a growing queue backlog
  - Publishers experience blocked/slow publishes once the memory high watermark is hit
- **Solution**: Set `x-max-length`/`x-max-length-bytes` policies (with an overflow behavior like drop-head or reject-publish, chosen deliberately) on queues that could plausibly back up, use lazy queues for high-volume queues to keep messages on disk rather than in memory, and monitor queue depth as a standard operational metric with alerting before it becomes a memory crisis

### SE-5: Unbounded Prefetch Causing Unfair Dispatch or Consumer Overload
- **Severity**: medium
- **Situation**: With prefetch unset (unlimited), RabbitMQ dispatches messages to whichever consumer's TCP connection is ready fastest, and a single fast-connecting-but-slow-processing consumer can end up holding a large batch of unacknowledged messages while other idle consumers starve
- **Cause**: Without a `prefetch_count` limit, round-robin dispatch happens based on delivery readiness, not actual processing capacity — RabbitMQ has no visibility into how fast a consumer can actually process what it's been given
- **Symptoms**:
  - Uneven load across consumer instances despite them being configured identically
  - One consumer instance shows a large number of unacknowledged messages while others are idle
- **Solution**: Set an explicit `prefetch_count` matched to each consumer's realistic processing throughput, so the broker distributes work more evenly and no single consumer becomes a bottleneck holding a large batch of undelivered-to-others messages

## Recommended Tools

| Category | Tools |
|------|------|
| Management | RabbitMQ Management UI/HTTP API |
| Monitoring | Prometheus RabbitMQ exporter, `rabbitmqctl` |
| Client libraries | Language-specific AMQP clients (pika, amqplib, amqp091-go) |
| Alternatives/complements | Kafka (high-throughput log-style streaming), SQS (managed, simpler semantics) |

## Reliability

**Full checklist**: → [extended/checklists.md#reliability-checklist]

## Related Resources

[RabbitMQ Docs](https://www.rabbitmq.com/docs) | [RabbitMQ Reliability Guide](https://www.rabbitmq.com/docs/reliability) | [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts)

## Related Domains

[[python]] | [[go]] | [[java]] | [[kubernetes]]
