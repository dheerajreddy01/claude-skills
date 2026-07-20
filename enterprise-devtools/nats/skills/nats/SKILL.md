---
schema: "1.0"
name: nats
version: "1.0.0"
description: NATS core pub/sub and JetStream persistence, subject design, and consumer acknowledgment practices
domain: technology
triggers:
  keywords:
    primary: [nats, jetstream, pub sub, subject]
    secondary: [nats streaming, request-reply, consumer ack, wildcard subject]
  context_boost: [messaging, microservices, event-driven]
  context_penalty: [kafka, rabbitmq, sqs]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# NATS (incl. JetStream)

> Core NATS is fire-and-forget by design — durability is an opt-in layer, not a default

## Applicable Scenarios

- Designing subject hierarchies and wildcard subscriptions
- Choosing between core NATS (at-most-once) and JetStream (persistent, at-least-once)
- Configuring JetStream stream retention and consumer acknowledgment modes
- Diagnosing message loss or unexpected subscriber fan-out
- Implementing request-reply patterns for service-to-service RPC-style calls

## Core Knowledge

### Core NATS vs. JetStream

| | Core NATS | JetStream |
|------|------|------|
| **Delivery** | At-most-once, fire-and-forget | At-least-once (or exactly-once with dedup), persistent |
| **Storage** | None — messages exist only in-flight | Messages stored (memory or file) per stream, replayable |
| **Use case** | Low-latency ephemeral pub/sub, service discovery, request-reply | Durable event streams, work queues, anything needing replay/durability |

Core NATS delivers a message to whichever subscribers are connected *at that instant* — a subscriber that's disconnected simply doesn't receive messages published while it was down, with no built-in way to catch up. This is a deliberate simplicity/performance tradeoff, not a bug, but it's easy to reach for core NATS assuming durability that isn't there.

### Subject-Based Addressing

- Messages are published to a **subject** (a dot-separated string like `orders.created.us-east`), and subscribers match subjects using exact names or **wildcards** (`*` for one token, `>` for one-or-more trailing tokens)
- Subject hierarchy design is the primary architectural decision in NATS — it determines both routing flexibility and the blast radius of a wildcard subscription (`orders.>` catches everything under `orders`, which may be broader than intended)

### Request-Reply

- NATS has a built-in request-reply pattern: a requester publishes with an auto-generated reply subject, and any responder publishes back to it — this makes NATS a natural fit for lightweight synchronous-style service-to-service calls, not just fire-and-forget events
- This is core NATS behavior (not JetStream-specific) and inherits core NATS's at-most-once semantics — a request with no connected responder simply times out

### JetStream Consumers & Acknowledgment

| Ack Mode | Behavior |
|------|------|
| **Auto-ack (`AckNone`)** | Message considered delivered the instant it's sent — no redelivery on consumer crash mid-processing |
| **Explicit ack (`AckExplicit`)** | Consumer must actively acknowledge; unacked messages redeliver after a configurable timeout |

JetStream streams have configurable **retention policies** (limits-based by age/size/count, or work-queue-style removing on ack) — without an explicit retention/storage limit, a stream can grow unbounded and consume disk indefinitely.

## Best Practices

1. **Use JetStream, not core NATS, for anything requiring durability** — core NATS messages published while a subscriber is offline are gone
2. **Design subject hierarchies deliberately**, considering wildcard blast radius before subscribing broadly
3. **Use explicit acknowledgment** in JetStream consumers for anything where processing correctness matters, not auto-ack
4. **Set explicit retention/storage limits on every JetStream stream** to bound disk usage
5. **Use request-reply for lightweight RPC-style calls**, understanding it inherits core NATS's at-most-once, connected-responder-required semantics
6. **Monitor consumer lag and pending/redelivered message counts** on JetStream consumers as a standard operational signal

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using core NATS for messages that must not be lost if a subscriber is briefly offline | Use JetStream, which persists messages for replay |
| Auto-ack consumers for critical processing | Use explicit ack, acknowledging only after successful processing |
| Overly broad wildcard subscriptions (`orders.>`) without considering what that actually catches | Scope wildcard subscriptions deliberately to the intended subset of the subject hierarchy |
| No retention/storage limit on a JetStream stream | Set explicit `max_age`/`max_bytes`/`max_msgs` limits appropriate to the use case |
| Assuming request-reply works if no responder happens to be connected | Handle the request timeout case explicitly; ensure a responder is reliably available for critical request-reply flows |

## Sharp Edges

### SE-1: Message Loss From Using Core NATS Where Durability Was Needed
- **Severity**: critical
- **Situation**: An application uses core NATS pub/sub for events that need to be reliably processed, a subscriber is briefly offline (deploy, crash, network blip), and every message published during that window is permanently lost with no error surfaced anywhere
- **Cause**: Core NATS has no persistence layer — it delivers to currently-connected subscribers only; there's no message log to replay from once a subscriber reconnects
- **Symptoms**:
  - Downstream processing shows gaps correlated with subscriber restarts/deploys, with no corresponding error in publisher logs
  - The issue is invisible in testing (where subscribers are typically always connected) and only surfaces under real operational conditions
- **Solution**: Use JetStream for any messages where loss during subscriber downtime is unacceptable — it persists messages to a stream and lets a reconnecting/new consumer replay from where it left off
- **Details**: → [extended/checklists.md#reliability-checklist]

### SE-2: Auto-Ack Losing Messages on Consumer Crash
- **Severity**: high
- **Situation**: A JetStream consumer configured with auto-ack (or effectively equivalent behavior) crashes mid-processing of a message, and on restart, that message is never redelivered — it was already considered "delivered" the instant it was sent, regardless of whether processing actually completed
- **Cause**: Auto-ack semantics tie acknowledgment to delivery, not to successful processing — this is the same class of pitfall as Kafka's auto-commit and RabbitMQ's auto-ack: convenient, but silently trades away at-least-once guarantees
- **Symptoms**:
  - A message provably never completes its expected processing (missing side effects), despite the consumer having received it
  - Loss correlates with consumer crashes/restarts, not steady-state operation
- **Solution**: Use explicit acknowledgment mode, acknowledging only after the message has been fully and successfully processed, and design consumer processing logic to be idempotent since redelivery after a crash-before-ack is expected behavior, not an edge case

### SE-3: Subject Wildcard Design Mistakes Causing Unintended Fan-Out
- **Severity**: high
- **Situation**: A subscriber uses a wildcard subject (e.g., `orders.>`) intending to catch a specific subset of events, but the actual subject hierarchy in use is broader than expected, and the subscriber receives (and has to filter out, or worse, mishandles) a much larger volume and variety of messages than intended
- **Cause**: Wildcard matching is purely structural (`*` matches exactly one token, `>` matches one-or-more trailing tokens) — it has no awareness of the *semantic* scope the subject hierarchy was meant to represent, so a hierarchy that grows over time (new subject types added under an existing prefix) silently expands what an existing wildcard subscription catches
- **Symptoms**:
  - A subscriber's processing load or error rate increases after an unrelated team starts publishing new message types under a shared subject prefix
  - Message types the subscriber was never designed to handle show up in its processing path
- **Solution**: Design subject hierarchies deliberately with wildcard blast radius in mind from the start, document which prefixes are "owned" by which subscribers' wildcard patterns, and treat adding a new subject under an existing prefix as a change that needs review against existing wildcard subscribers

### SE-4: Unbounded JetStream Stream Growth
- **Severity**: medium
- **Situation**: A JetStream stream created without explicit retention limits grows indefinitely as messages accumulate, eventually consuming all available disk space on the NATS server
- **Cause**: Without `max_age`, `max_bytes`, or `max_msgs` configured, a stream retains messages according to its retention policy defaults, which may not impose a meaningful practical bound for a high-volume stream
- **Symptoms**:
  - Disk usage on NATS JetStream servers climbs steadily and doesn't correlate with expected retention needs
  - Server eventually hits disk pressure, potentially affecting all streams on that server, not just the unbounded one
- **Solution**: Set explicit retention limits (age, byte size, or message count) on every JetStream stream based on actual replay/durability needs, and monitor stream storage usage as a standard operational metric

### SE-5: Request-Reply Timeouts With No Connected Responder
- **Severity**: medium
- **Situation**: A service makes a NATS request-reply call expecting a response, but no responder happens to be connected/subscribed to that subject at that moment, and the request silently times out with no immediate indication of *why* — was the responder down, slow, or does no responder for that subject exist at all?
- **Cause**: Request-reply inherits core NATS's connectionless-delivery semantics — there's no built-in distinction between "no responder is currently listening" and "a responder is listening but slow to reply," both just manifest as a timeout
- **Symptoms**:
  - Intermittent request timeouts that are hard to distinguish from genuine responder slowness vs. a responder simply not being deployed/connected
- **Solution**: Ensure critical request-reply flows have a reliably available responder (health-checked, appropriately scaled), handle the timeout case explicitly in calling code with clear error semantics, and consider whether a genuinely critical flow should use JetStream-backed request/response instead for better observability into what happened

## Recommended Tools

| Category | Tools |
|------|------|
| CLI/monitoring | `nats` CLI, NATS Surveyor, `nats-top` |
| Client libraries | Official NATS clients per language |
| Clustering | NATS clustering, leaf nodes for edge/hybrid topologies |
| Observability | NATS Prometheus exporter |

## Reliability

**Full checklist**: → [extended/checklists.md#reliability-checklist]

## Related Resources

[NATS Docs](https://docs.nats.io/) | [JetStream Concepts](https://docs.nats.io/nats-concepts/jetstream) | [NATS by Example](https://natsbyexample.com/)

## Related Domains

[[kafka]] | [[rabbitmq]] | [[go]] | [[kubernetes]]
