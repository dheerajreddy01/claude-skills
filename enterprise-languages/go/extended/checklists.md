# Go — Extended Checklists

## Concurrency Checklist

Before merging code with goroutines/channels:

- [ ] Every `go f()` has a clear, reachable termination condition (no goroutine that can block forever)
- [ ] `context.Context` is propagated through the call chain for anything doing I/O, with sane timeouts/deadlines set at the entrypoint
- [ ] Shared mutable state is protected by a `Mutex`/`RWMutex`, or avoided by communicating via channels instead
- [ ] `go test -race` passes for all packages touching goroutines/shared state, and runs in CI
- [ ] Channel send/receive pairing is reasoned through explicitly — no unbuffered channel used without a guaranteed active counterpart
- [ ] Worker pools have bounded size (no unbounded goroutine-per-request under load)
- [ ] `pprof`/`runtime.NumGoroutine()` checked under sustained load to confirm no goroutine count growth over time

## Production Readiness Checklist

- [ ] `go vet` and `golangci-lint` run in CI and block merge on failure
- [ ] Every returned `error` is checked; wrapped with `%w` where context should be preserved
- [ ] `go.sum` committed; `go mod verify` run in CI to detect tampering
- [ ] Structured logging (e.g. `slog`) with request/trace IDs, not bare `fmt.Println`
- [ ] Graceful shutdown implemented (drain in-flight requests on `SIGTERM`) for long-running services
- [ ] Resource limits set (max connections, request timeouts, body size limits)
- [ ] Health/readiness endpoints exposed for orchestrators (k8s liveness/readiness probes)
- [ ] Build pinned to a specific Go toolchain version in CI, matching `go.mod`'s `go` directive
- [ ] Table-driven tests cover error paths, not just the happy path
- [ ] Binary built with `CGO_ENABLED=0` where a static binary is desired for minimal container images
