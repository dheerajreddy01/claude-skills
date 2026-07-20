# Kafka — Extended Checklists

## Reliability Checklist

- [ ] `acks=all` with `min.insync.replicas >= 2` (replication factor ≥ 3) used for topics where losing an acknowledged write is unacceptable
- [ ] Idempotent producer (`enable.idempotence=true`) enabled to avoid duplicate writes from producer retries
- [ ] Auto-commit disabled for consumers where processing correctness matters; offsets committed manually only after successful processing
- [ ] Consumer processing logic designed to be idempotent, since at-least-once delivery means reprocessing is possible after any rebalance or restart
- [ ] Partition count planned for expected future scale before topic creation — not something to be decreased later, and increases affect key-ordering guarantees
- [ ] Partition key cardinality/distribution reviewed to avoid hot partitions concentrating load
- [ ] Consumer lag monitored per-partition (not just aggregate), to catch hot-partition or straggler-consumer issues early

## Consumer Group & Operational Checklist

- [ ] Poll loop kept fast; slow per-message work offloaded to an async pipeline rather than blocking `poll()`
- [ ] `max.poll.records`/`max.poll.interval.ms` tuned so a normal batch comfortably fits within the interval
- [ ] Rebalance frequency monitored as a standard operational metric, investigated when it spikes
- [ ] Topic retention (`retention.ms`/`retention.bytes`) set deliberately based on actual replay/recovery needs, not left at defaults
- [ ] Compacted topics (`cleanup.policy=compact`) used only for genuine changelog/latest-state-per-key use cases, not event history
- [ ] Broker disk usage monitored against retention settings to avoid unplanned data loss from disk pressure
- [ ] Schema compatibility (if using Avro/Protobuf + Schema Registry) enforced in CI before producer/consumer deploys
