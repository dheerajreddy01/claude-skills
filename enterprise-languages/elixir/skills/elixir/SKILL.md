---
schema: "1.0"
name: elixir
version: "1.0.0"
description: Elixir/BEAM concurrency model, OTP supervision trees, and Phoenix production practices
domain: technology
triggers:
  keywords:
    primary: [elixir, phoenix, erlang, BEAM]
    secondary: [OTP, genserver, mix, liveview]
  context_boost: [actor model, fault tolerance, real-time]
  context_penalty: [python, java, go]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Elixir

> "Let it crash" only works if the supervision tree actually knows how to restart the right thing

## Applicable Scenarios

- Writing or reviewing Elixir/OTP applications (GenServers, supervisors)
- Designing supervision trees and restart strategies
- Building real-time features with Phoenix/LiveView
- Diagnosing process mailbox growth or BEAM scheduler stalls
- Reviewing ETS table usage for shared state

## Core Knowledge

### The BEAM & Actor Model

- The BEAM VM runs extremely lightweight, isolated **processes** (not OS threads/processes) — millions can run concurrently, each with its own heap and no shared mutable memory
- Processes communicate exclusively via **message passing** to a process's **mailbox**; there's no shared-memory concurrency to synchronize, which eliminates a whole class of race conditions but introduces mailbox-based backpressure concerns instead

### OTP & Supervision Trees

- **GenServer** is the standard abstraction for a stateful process with a defined message-handling API (`handle_call`, `handle_cast`, `handle_info`)
- **Supervisors** monitor child processes and restart them according to a strategy (`:one_for_one`, `:one_for_all`, `:rest_for_one`) when they crash
- **"Let it crash"** is the philosophy of not defensively coding around every possible error inside a process — instead, let the process crash and have its supervisor restart it into a known-good state. This only works well when the supervision tree is designed so restarts actually clear the bad state, and doesn't mask a systemic bug that will just crash-loop instead

### Pattern Matching & Immutability

- Function clauses pattern-match on their arguments, which is the primary mechanism for control flow (rather than `if`/`switch` chains) and makes many invalid states unrepresentable at the call site
- All data is immutable — "modifying" a data structure returns a new one, which is central to how safe concurrent access works without locks

### Phoenix & LiveView

- Phoenix is the standard web framework; **LiveView** provides server-rendered, real-time-updating UI over a persistent WebSocket connection without writing client-side JavaScript for most interactivity
- Each connected LiveView is backed by a BEAM process holding the view's state (**socket assigns**) — this state lives on the server for the connection's duration, not in the browser

## Best Practices

1. **Design supervision trees deliberately** — choose a restart strategy that matches actual failure dependencies between children, not just `:one_for_one` by default without thinking it through
2. **Keep GenServer state and message handling minimal** — heavy computation inside `handle_call` blocks the process (and its callers, for `call`) until it finishes
3. **Use bounded queues/backpressure mechanisms** (e.g., GenStage, bounded process pools) for anything that could receive messages faster than it can process them
4. **Avoid NIFs (native code) for long-running work** unless using dirty schedulers — a blocking NIF stalls a BEAM scheduler thread
5. **Treat ETS as a concurrent data structure with real semantics** — understand read/write concurrency guarantees rather than assuming implicit atomicity for multi-step operations
6. **Keep LiveView socket assigns lean** — store only what the view actually needs to render, not entire unrelated domain objects

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `:one_for_one` supervision applied without considering actual child dependencies | Choose the restart strategy (`:one_for_one`, `:one_for_all`, `:rest_for_one`) based on how children actually depend on each other |
| Heavy computation inside a GenServer's `handle_call` | Offload long-running work to a separate `Task`, or make it async via `handle_cast`/`handle_info` with a reply mechanism |
| Assuming an ETS read-then-write is atomic | Use `:ets.update_counter` or GenServer-serialized access for anything requiring atomic read-modify-write |
| Long-running native code in a regular (non-dirty) NIF | Use dirty schedulers for long-running NIFs, or move the work to an external process/port |
| Storing large or unbounded data in a LiveView socket's assigns | Keep assigns scoped to what's needed for rendering; fetch/paginate rather than holding everything in process state |

## Sharp Edges

### SE-1: Supervision Tree Misdesign Masking Real Bugs
- **Severity**: high
- **Situation**: A process crashes repeatedly due to a genuine bug (not a transient failure), and the supervisor keeps restarting it into the same bad state, either crash-looping indefinitely or "succeeding" by restarting so fast the underlying bug never surfaces clearly in monitoring
- **Cause**: "Let it crash" assumes a restart returns the system to a known-good state — if the crash is caused by a deterministic bug (not transient external state), restarting doesn't fix anything, and depending on the restart strategy, a supervisor might restart sibling processes unnecessarily too
- **Symptoms**:
  - Restart counts for a specific process climb continuously, potentially hitting the supervisor's max-restart-intensity limit and taking down the whole supervision subtree
  - The underlying bug is deterministic (same input always crashes) but gets treated as if a restart would help
- **Solution**: Distinguish between transient failures (network blip, expected to clear on restart) and deterministic bugs (fix the code, don't rely on restart to paper over it); set sensible max-restart-intensity limits so a genuinely broken process escalates to a human rather than crash-looping silently forever
- **Details**: → [extended/checklists.md#otp-reliability-checklist]

### SE-2: Process Mailbox Unbounded Growth
- **Severity**: critical
- **Situation**: A process receives messages faster than it can process them (no backpressure mechanism), and its mailbox grows without bound, eventually consuming all available memory
- **Cause**: BEAM process mailboxes have no default size limit — a slow consumer with a fast producer accumulates messages indefinitely unless something explicitly bounds the queue or applies backpressure
- **Symptoms**:
  - Memory usage climbs correlated with a specific process's message volume
  - `Process.info(pid, :message_queue_len)` shows a mailbox growing continuously for a specific process
- **Solution**: Use a backpressure-aware pattern (GenStage/Broadway for producer-consumer pipelines, bounded task pools, or explicit flow control) for anything where message production rate could exceed consumption rate, and monitor mailbox length as a standard operational metric

### SE-3: Blocking NIFs Stalling the BEAM Scheduler
- **Severity**: high
- **Situation**: A native (NIF) function that runs for more than a few milliseconds blocks the BEAM scheduler thread it's running on, preventing that scheduler from running any other process — including unrelated ones — until the NIF returns
- **Cause**: Regular NIFs execute synchronously on a scheduler thread as if they were part of the BEAM's own execution; the whole "millions of lightweight processes" concurrency model assumes each unit of work yields quickly, which a long-running NIF violates
- **Symptoms**:
  - Latency spikes across seemingly unrelated processes correlated with a specific NIF-calling code path
  - The BEAM's normally very low and predictable scheduling latency degrades sharply under specific workloads
- **Solution**: Use dirty schedulers (`:dirty_cpu`/`:dirty_io` NIF flags) for any NIF that might run longer than roughly a millisecond, or move genuinely long-running native work to an external process communicated with via a port instead of an in-VM NIF

### SE-4: ETS Race Conditions on Non-Atomic Operations
- **Severity**: medium
- **Situation**: Code reads a value from an ETS table, computes a new value based on it, and writes it back — under concurrent access from multiple processes, two processes can interleave this read-modify-write sequence and lose an update
- **Cause**: Individual ETS operations (a single `lookup` or a single `insert`) are atomic, but a read-then-compute-then-write sequence composed of multiple separate ETS calls is not atomic as a whole — nothing prevents another process from writing in between
- **Symptoms**:
  - A counter or aggregate value stored in ETS is occasionally lower than expected under concurrent load, consistent with lost updates
- **Solution**: Use ETS's atomic primitives (`:ets.update_counter/3`, `:ets.select_replace/2`) for read-modify-write patterns instead of separate lookup-then-insert calls, or route the whole operation through a single GenServer that serializes access if the logic is too complex for ETS's built-in atomic operations

### SE-5: LiveView Socket Assign Bloat
- **Severity**: medium
- **Situation**: A LiveView holds large or unbounded data structures in its socket assigns (e.g., an entire unpaginated dataset, deeply nested unrelated domain objects), and per-connection memory usage scales with that data, multiplied by every concurrently connected user
- **Cause**: Each connected LiveView is backed by a dedicated BEAM process holding its assigns in memory for the life of the connection — unlike a stateless HTTP request/response, this state persists and accumulates per active user
- **Symptoms**:
  - Server memory usage scales roughly linearly with concurrent connected LiveView users, disproportionate to what each view actually renders
  - A specific LiveView's memory footprint is much larger than the data actually visible on screen
- **Solution**: Keep assigns scoped to what's needed for the current render (paginate, fetch on demand), use `Phoenix.LiveView.Async`/streams for large or growing collections instead of holding the full set in assigns, and profile per-connection memory usage for LiveViews expected to handle high concurrent connection counts

## Recommended Tools

| Category | Tools |
|------|------|
| Build/deps | Mix, Hex |
| Testing | ExUnit, StreamData (property-based) |
| Static analysis | Credo, Dialyzer |
| Observability | `:observer`, Telemetry, LiveDashboard |

## OTP Reliability

**Full checklist**: → [extended/checklists.md#otp-reliability-checklist]

## Related Resources

[Elixir Docs](https://elixir-lang.org/docs.html) | [Elixir School](https://elixirschool.com/) | [The Erlang/OTP Design Principles](https://www.erlang.org/doc/design_principles/des_princ.html)

## Related Domains

[[rabbitmq]] | [[kafka]] | [[postgresql]]
