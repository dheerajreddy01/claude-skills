---
schema: "1.0"
name: go
version: "1.0.0"
description: Go language, goroutines/channels, error handling, and production service practices
domain: technology
triggers:
  keywords:
    primary: [go, golang, goroutine, channel, go.mod]
    secondary: [interface, defer, gofmt, go vet, race detector]
  context_boost: [microservice, concurrency, CLI tool, backend]
  context_penalty: [python, javascript, java]
  priority: high
dependencies:
  software-skills: [git-workflows, testing-strategies]
author: claude-domain-skills
---

# Go

> Explicit errors, simple concurrency primitives, and a small, boring standard toolchain

## Applicable Scenarios

- Writing or reviewing Go services, CLIs, or libraries
- Designing concurrency with goroutines and channels
- Debugging goroutine leaks, deadlocks, or data races
- Structuring error handling and wrapping
- Setting up modules, tooling, and testing for a Go project

## Core Knowledge

### Modules & Tooling

| Tool | Purpose |
|------|------|
| `go.mod` / `go.sum` | Module definition and dependency checksums (built-in, no external package manager needed) |
| `go fmt` | Canonical formatting — non-negotiable, run on save |
| `go vet` | Static analysis for suspicious constructs |
| `golangci-lint` | Aggregates multiple linters (staticcheck, errcheck, etc.) |
| `go test -race` | Runs the data-race detector — use in CI for anything concurrent |

### Concurrency Primitives

| Primitive | Use |
|------|------|
| **Goroutine** | Lightweight concurrent function (`go f()`) — cheap, but must have a clear exit condition |
| **Channel** | Typed pipe for communication between goroutines (`ch := make(chan T)`) |
| **`select`** | Wait on multiple channel operations at once |
| **`sync.WaitGroup`** | Wait for a group of goroutines to finish |
| **`sync.Mutex`** | Protect shared mutable state not passed via channels |
| **`context.Context`** | Cancellation and deadline propagation through a call chain |

Rule of thumb: "Don't communicate by sharing memory; share memory by communicating" — prefer channels for handoff, but a `Mutex` is fine (and often simpler) for protecting a small piece of shared state.

### Error Handling

- Errors are values, returned explicitly as the last return value — no exceptions for control flow
- Wrap errors with context using `fmt.Errorf("doing X: %w", err)` to preserve the chain
- Use `errors.Is`/`errors.As` to check wrapped errors, not string matching
- Only `panic` for truly unrecoverable programmer errors (e.g., invariant violations at init); recover only at well-defined boundaries (e.g., HTTP middleware)

### Interfaces

- Interfaces are satisfied implicitly — no `implements` keyword
- Keep interfaces small (often one method) and defined at the *consumer*, not the implementer
- `nil` interface vs `nil` concrete value inside an interface are NOT the same — a classic gotcha (see Sharp Edges)

## Best Practices

1. **Always check errors** — an ignored `err` is a silent failure waiting to happen
2. **Run `-race` in CI** for any code using goroutines/channels — races are Heisenbugs in production
3. **Propagate `context.Context`** through call chains that do I/O, so cancellation/timeouts work end-to-end
4. **Keep goroutines' lifetimes obvious** — every `go f()` should have a clear way it terminates
5. **Prefer composition over inheritance-like patterns** — embed structs/interfaces rather than building deep hierarchies
6. **Table-driven tests** — Go's idiomatic pattern for covering multiple cases concisely

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Ignoring returned `error` | Check and handle/wrap every `error` return |
| Looping and starting goroutines that never get joined | Use `sync.WaitGroup` or a worker pool with clear shutdown |
| Comparing errors with `==` after wrapping | Use `errors.Is`/`errors.As` |
| Capturing loop variables incorrectly in closures (pre-1.22) | Shadow the variable inside the loop body, or upgrade to Go 1.22+ (per-iteration scoping) |
| Returning a typed `nil` pointer as an `error` interface | Return a literal `nil`, not a `nil`-valued typed pointer, when there's no error |

## Sharp Edges

### SE-1: The Typed-Nil Interface Trap
- **Severity**: critical
- **Situation**: A function returns `error` from a variable that's a `nil` pointer of a concrete error type, and `if err != nil` evaluates `true` even though "there's no error"
- **Cause**: An interface value is `nil` only if both its type AND value are nil; a `nil *MyError` boxed into the `error` interface has a non-nil type, so the interface itself is non-nil
- **Symptoms**:
  - `if err != nil { ... }` triggers even when the underlying pointer is `nil`
  - Bug only appears when the error path returns a typed pointer instead of a bare `nil`
- **Solution**: Return a bare `nil` (untyped) when there's no error; never return a `nil`-valued concrete pointer through an `error`-typed return

### SE-2: Goroutine Leaks
- **Severity**: high
- **Situation**: A goroutine blocks forever on a channel send/receive that never happens (e.g., the receiver already returned), and it never gets cleaned up
- **Cause**: Goroutines aren't garbage collected while blocked — they leak memory and scheduler overhead indefinitely
- **Symptoms**:
  - Goroutine count (visible via `pprof`/`runtime.NumGoroutine()`) grows unbounded over time
  - Gradual memory growth with no obvious allocation source
- **Solution**: Always give goroutines a cancellation path (`context.Context`, a `done` channel), and ensure every send/receive has a corresponding, reachable counterpart
- **Details**: → [extended/checklists.md#concurrency-checklist]

### SE-3: Data Races Not Caught Without `-race`
- **Severity**: critical
- **Situation**: Two goroutines read/write the same variable without synchronization; the program "works" in testing but corrupts data or crashes intermittently in production
- **Cause**: Go doesn't prevent data races at compile time, and normal test runs don't detect them
- **Symptoms**:
  - Intermittent, non-reproducible corruption or panics under load
  - Works fine on a single core, fails under real concurrency
- **Solution**: Run `go test -race` in CI for every package touching shared state; protect shared state with a `Mutex` or restructure to communicate via channels

### SE-4: Deadlocks From Unbuffered Channel Misuse
- **Severity**: high
- **Situation**: A goroutine sends on an unbuffered channel with no active receiver (or vice versa), and the program hangs forever
- **Cause**: Unbuffered channel sends/receives block until both sides are ready — if the ordering doesn't line up, it's a permanent deadlock
- **Symptoms**:
  - Program hangs with no error, `panic: all goroutines are asleep - deadlock!` from the runtime in the simplest cases
- **Solution**: Reason explicitly about send/receive pairing, use buffered channels when producer/consumer rates can diverge, or use `select` with a `default`/timeout to avoid permanent blocks

### SE-5: Slice Aliasing Surprises
- **Severity**: medium
- **Situation**: Appending to a slice derived from another slice (`sub := full[2:4]`) unexpectedly mutates the original backing array
- **Cause**: Slices share an underlying array; `append` only allocates a new array if capacity is exceeded, otherwise it writes into the shared backing array
- **Symptoms**:
  - Modifying one slice changes data seen through a supposedly independent slice
  - Bug appears only for certain input sizes (whenever capacity headroom exists)
- **Solution**: Use `copy()` to create an independent slice when you need one, or slice with a full three-index expression (`full[2:4:4]`) to cap capacity and force reallocation on append

## Recommended Tools

| Category | Tools |
|------|------|
| Linting | golangci-lint, staticcheck |
| Testing | `testing` (stdlib), testify, gomock |
| Profiling | pprof, `go test -race`, `go test -bench` |
| Build/release | goreleaser |

## Production Readiness

**Full deployment checklist**: → [extended/checklists.md#production-readiness-checklist]

## Related Resources

[Effective Go](https://go.dev/doc/effective_go) | [Go by Example](https://gobyexample.com/) | [Go Blog](https://go.dev/blog/)

## Related Domains

[[python]] | [[rust]] | [[aws]]
