# k6 / JMeter — Extended Checklists

## Test Design Checklist

- [ ] Load generator's own CPU/network/file-descriptor usage monitored during every test run
- [ ] Load distributed across multiple generator machines once a single generator's limits are reached
- [ ] Scripts model realistic think-time between actions, not back-to-back requests
- [ ] Request payloads/parameters varied to reflect real traffic diversity, not identical repeated requests
- [ ] Target environment is production-parity (instance sizes, data volume, CDN/network topology) or differences are explicitly accounted for
- [ ] Backend saturation signals (DB connections, queue depth, CPU) instrumented and correlated with the same test time window
- [ ] Test scenario matches the actual question being asked (load vs. stress vs. soak vs. spike)

## Coordination and Safety Checklist

- [ ] Any test against shared or production-adjacent infrastructure coordinated with its owners beforehand
- [ ] Blast radius understood and contained (isolated environment preferred over shared staging)
- [ ] Rate limits/circuit breakers on the target system understood so the test doesn't trip unrelated safety mechanisms unexpectedly
- [ ] Soak tests run for a duration long enough to surface slow degradation (memory leaks, log/disk growth), not just peak-load snapshots
- [ ] Results compared against previous baselines to catch performance regressions over time, not just pass/fail against a single threshold
- [ ] Findings documented with both client-side and backend metrics, not latency numbers alone
