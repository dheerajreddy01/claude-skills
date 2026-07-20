# NATS — Extended Checklists

## Reliability Checklist

- [ ] JetStream (not core NATS) used for any message flow where subscriber-downtime message loss is unacceptable
- [ ] Explicit acknowledgment mode used for JetStream consumers where processing correctness matters, not auto-ack
- [ ] Consumer processing logic designed to be idempotent, since at-least-once delivery means redelivery after a crash-before-ack is expected
- [ ] Every JetStream stream has explicit retention limits (`max_age`/`max_bytes`/`max_msgs`) set deliberately
- [ ] Subject hierarchy documented, including which prefixes are "owned" by which wildcard subscribers
- [ ] New subject additions under an existing prefix reviewed against existing wildcard subscriptions before publishing
- [ ] Critical request-reply flows have a reliably available, health-checked responder
- [ ] Request-reply timeout handling distinguishes (where possible) "no responder" from "slow responder" for debuggability

## Operational Health Checklist

- [ ] Stream storage usage monitored per stream, with alerting before disk pressure affects the server
- [ ] Consumer pending/redelivered message counts monitored as a standard operational signal
- [ ] NATS server clustering configured for HA where a single-node outage is unacceptable
- [ ] `nats` CLI or NATS Surveyor used to periodically audit subject usage against documented hierarchy design
- [ ] Client reconnect/retry behavior tested explicitly, not just assumed to work
