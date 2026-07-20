# RabbitMQ — Extended Checklists

## Reliability Checklist

- [ ] Publisher confirms enabled for anything where publish-time message loss is unacceptable
- [ ] Negative-ack/timeout on publisher confirms handled explicitly (retry, alert, or dead-letter), not ignored
- [ ] Consumer acknowledgment mode is manual (not auto-ack) for anything where mid-processing loss matters
- [ ] Durable queues/exchanges AND persistent message delivery mode configured together — verified by testing an actual broker restart
- [ ] Dead-letter exchange configured on every queue where a poison message is plausible
- [ ] Retry/requeue logic includes a bounded retry count before dead-lettering, not infinite requeue
- [ ] Queue length limits (`x-max-length`/`x-max-length-bytes`) or lazy queue mode set on queues that could plausibly back up
- [ ] `prefetch_count` set explicitly, matched to realistic per-consumer processing throughput
- [ ] Exchange/routing key/binding combinations verified to actually route as intended — no silent message drops from unmatched bindings
- [ ] Queue depth and broker memory usage monitored with alerting configured before hitting the memory high watermark
