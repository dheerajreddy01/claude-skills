# Elixir — Extended Checklists

## OTP Reliability Checklist

- [ ] Supervision restart strategy (`:one_for_one`/`:one_for_all`/`:rest_for_one`) chosen based on actual dependencies between child processes
- [ ] Max-restart-intensity limits set sensibly so a genuinely broken (deterministically crashing) process escalates rather than crash-looping forever
- [ ] Transient failures (expected to clear on restart) distinguished from deterministic bugs (need an actual code fix)
- [ ] Process mailbox length monitored (`Process.info(pid, :message_queue_len)` or Telemetry) as a standard operational metric
- [ ] Backpressure-aware patterns (GenStage/Broadway, bounded pools) used wherever producer rate could exceed consumer rate
- [ ] Long-running NIFs use dirty scheduler flags (`:dirty_cpu`/`:dirty_io`), or work is moved to a port instead
- [ ] ETS read-modify-write sequences use atomic primitives (`update_counter`, `select_replace`) instead of separate lookup-then-insert calls

## Application & LiveView Checklist

- [ ] LiveView socket assigns scoped to what's needed for rendering; large/unbounded data paginated or streamed instead of held in full
- [ ] Per-connection memory usage profiled for LiveViews expected to handle high concurrent connections
- [ ] Credo and Dialyzer run in CI for static analysis and type-spec checking
- [ ] GenServer `handle_call` kept fast; heavy work offloaded to `Task`/async patterns instead of blocking callers
- [ ] Telemetry/LiveDashboard configured for production visibility into process counts, mailbox sizes, and scheduler utilization
