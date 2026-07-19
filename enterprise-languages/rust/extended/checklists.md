# Rust — Extended Checklists

## Error Handling Checklist

Before merging code with fallible operations:

- [ ] No `.unwrap()`/`.expect()` on a `Result`/`Option` derived from external input (network, filesystem, user input, config)
- [ ] Library crates expose typed errors (`thiserror`), not stringly-typed errors
- [ ] Application-level code aggregates errors sensibly (`anyhow` is fine at the top level)
- [ ] Every `?`-propagated error retains enough context to debug (wrap with `.context(...)` where the original error alone is ambiguous)
- [ ] Panics are reserved for genuine programmer-error invariant violations, not expected failure modes
- [ ] `catch_unwind` boundaries exist where a panic in one task/request shouldn't take down the whole process (e.g. per-request handler in a server)

## Production Readiness Checklist

- [ ] `cargo clippy --all-targets -- -D warnings` runs in CI and blocks merge
- [ ] `cargo audit` (or cargo-deny) runs in CI to catch known-vulnerable dependencies
- [ ] `Cargo.lock` committed for binaries (reproducible builds); reviewed on dependency bumps
- [ ] Release profile reviewed: `overflow-checks`, `panic = "abort"` vs `"unwind"`, LTO/opt-level chosen deliberately
- [ ] Any `unsafe` block has a comment documenting the invariant it relies on; Miri run in CI if feasible
- [ ] Structured logging (`tracing`) with request/trace IDs, not bare `println!`
- [ ] Graceful shutdown handled for long-running async services (drain in-flight tasks on `SIGTERM`)
- [ ] Binary size and startup time checked if targeting constrained environments (containers, edge)
- [ ] Tests cover error paths (`Err`/`None` branches), not only the success path
- [ ] Cross-compilation target confirmed if shipping to a different OS/arch than CI runs on
