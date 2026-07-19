---
schema: "1.0"
name: rust
version: "1.0.0"
description: Rust ownership/borrowing, error handling, traits, and production systems practices
domain: technology
triggers:
  keywords:
    primary: [rust, cargo, ownership, borrow checker, lifetime]
    secondary: [trait, unsafe, tokio, crate, Result, Option]
  context_boost: [systems programming, CLI tool, WASM, performance-critical]
  context_penalty: [python, javascript, java]
  priority: high
dependencies:
  software-skills: [git-workflows, testing-strategies]
author: claude-domain-skills
---

# Rust

> Memory safety without a garbage collector, enforced at compile time

## Applicable Scenarios

- Writing or reviewing Rust code for systems, CLI tools, or performance-critical services
- Working through ownership/borrowing/lifetime compiler errors
- Designing error handling with `Result`/`Option` and custom error types
- Choosing between sync and async (Tokio) concurrency
- Evaluating when `unsafe` is actually justified

## Core Knowledge

### Ownership Model

| Rule | Meaning |
|------|------|
| **Ownership** | Each value has exactly one owner; when the owner goes out of scope, the value is dropped |
| **Borrowing** | You can have either one mutable reference (`&mut T`) OR any number of immutable references (`&T`), never both at once |
| **Lifetimes** | The compiler tracks how long references are valid to prevent dangling references — usually inferred, occasionally needs explicit annotation (`'a`) |

This model is enforced entirely at compile time — there's no garbage collector and no runtime borrow checking (unlike `Rc`/`RefCell`, which move some checks to runtime deliberately).

### Error Handling

- `Result<T, E>` for recoverable errors, `Option<T>` for optional values — no exceptions, no `null`
- `?` operator propagates errors up the call stack concisely
- `unwrap()`/`expect()` panic on `None`/`Err` — fine in prototypes and tests, a liability in production paths
- Use `thiserror` for library error types, `anyhow` for application-level error aggregation

### Traits & Generics

- Traits define shared behavior (similar to interfaces), implemented via `impl Trait for Type`
- Generic code is monomorphized at compile time — zero runtime cost, but larger binaries
- `dyn Trait` gives runtime polymorphism (trait objects) at the cost of a vtable indirection

### Concurrency

| Approach | Use |
|------|------|
| **`std::thread`** | OS threads, `Send`/`Sync` traits enforced at compile time to prevent data races |
| **`Arc<Mutex<T>>`** | Shared mutable state across threads, safely |
| **Tokio (async)** | Async I/O-bound work; requires an async runtime, not built into the language |
| **Rayon** | Data-parallelism for CPU-bound work (parallel iterators) |

The type system (`Send`/`Sync`) makes most data races a *compile-time* error rather than a runtime bug — this is one of Rust's biggest practical advantages over C/C++/Go for concurrent code.

## Best Practices

1. **Let the compiler teach you** — borrow checker errors usually point at a real design issue, not just syntax to work around
2. **Avoid `unwrap()`/`expect()` on the happy path in production code** — propagate with `?` or handle explicitly
3. **Prefer `&str` over `String` in function signatures** when you don't need ownership — avoids unnecessary allocation
4. **Use `clippy`** — catches idiomatic issues the compiler itself won't flag
5. **Justify every `unsafe` block** with a comment explaining the invariant being upheld
6. **Reach for `Rc<RefCell<T>>` sparingly** — usually a sign the ownership model should be restructured, not routed around

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `.unwrap()` on `Result`/`Option` in library or production code | Propagate with `?`, or handle the `None`/`Err` case explicitly |
| Fighting the borrow checker by cloning everything | Restructure ownership, use references/lifetimes properly, or use `Rc`/`Arc` deliberately |
| Mixing sync blocking calls inside async Tokio tasks | Use `tokio::task::spawn_blocking` for blocking work |
| Reaching for `unsafe` to silence a compiler error without understanding why | Understand the borrow checker's objection first; `unsafe` is rarely the fix for "the compiler won't let me" |
| Ignoring `clippy` warnings | Run `cargo clippy` in CI and fix or explicitly allow with justification |

## Sharp Edges

### SE-1: `unwrap()` Panics in Production
- **Severity**: critical
- **Situation**: Code that worked in every test panics in production because `.unwrap()` was called on a `None`/`Err` that only occurs with real-world input
- **Cause**: `unwrap()`/`expect()` convert a recoverable error into an unrecoverable panic — fine for prototypes, a production liability when the failure path is reachable
- **Symptoms**:
  - Process crashes with `thread 'main' panicked at ...` in production logs
  - Crash correlates with a specific class of input (missing config key, empty collection, parse failure)
- **Solution**: Audit `unwrap()`/`expect()` calls on any path that touches external input; propagate with `?` or handle explicitly with `match`/`if let`
- **Details**: → [extended/checklists.md#error-handling-checklist]

### SE-2: Fighting the Borrow Checker by Over-Cloning
- **Severity**: medium
- **Situation**: Every borrow-checker error gets "fixed" by adding `.clone()`, resulting in code that compiles but defeats the performance benefits Rust was chosen for
- **Cause**: Cloning sidesteps the ownership rules instead of addressing the underlying data-flow design that's actually in conflict
- **Symptoms**:
  - Code full of `.clone()` calls, especially on large structs
  - Performance profile shows unexpected allocation/copy overhead in a "zero-cost abstraction" language
- **Solution**: Understand *why* the borrow checker objects (usually a lifetime or aliasing conflict) and restructure ownership — pass references, split structs, or use `Rc`/`Arc` deliberately, not reflexively

### SE-3: Deadlocks and Blocking Calls in Async Code
- **Severity**: high
- **Situation**: A blocking, CPU-heavy, or synchronous I/O call runs directly inside a Tokio async task, stalling the entire worker thread's task queue
- **Cause**: Tokio's async tasks are cooperatively scheduled on a small thread pool; a blocking call doesn't yield, starving other tasks on that worker thread
- **Symptoms**:
  - Latency spikes or apparent "hangs" under concurrent load despite low CPU usage
  - Unrelated async tasks stall when one task does blocking I/O or heavy computation
- **Solution**: Use `tokio::task::spawn_blocking` for blocking/CPU-heavy work, or the async-native equivalent of the blocking call (e.g. `tokio::fs` instead of `std::fs`)

### SE-4: `unsafe` Blocks Hiding Undefined Behavior
- **Severity**: critical
- **Situation**: An `unsafe` block (raw pointer dereference, manual memory management, FFI) violates an invariant the compiler can no longer check, causing intermittent UB — crashes, corruption, or worse, silent wrong results
- **Cause**: `unsafe` opts out of the borrow checker's guarantees for that block; the programmer takes on the burden of upholding memory safety manually
- **Symptoms**:
  - Non-deterministic crashes or corruption that vary by build/optimization level
  - Bugs that Miri (Rust's UB detector) or sanitizers catch but normal testing doesn't
- **Solution**: Minimize `unsafe` surface area, document the exact invariant each block relies on, run Miri in CI for `unsafe`-heavy crates, prefer safe abstractions from vetted crates over hand-rolled `unsafe`

### SE-5: Integer Overflow Behavior Differs by Build Profile
- **Severity**: medium
- **Situation**: Integer overflow panics in debug builds but silently wraps in release builds (by default), so a bug caught in dev "disappears" in production and corrupts data instead
- **Cause**: Rust's default overflow checking is enabled in debug profile and disabled (wraps silently) in release profile for performance
- **Symptoms**:
  - Works correctly (or panics loudly) in `cargo run`, produces wrong numeric results in `cargo run --release`
- **Solution**: Use checked/saturating/wrapping arithmetic explicitly (`checked_add`, `saturating_sub`) wherever overflow is a real possibility, or enable `overflow-checks = true` in the release profile if a panic is preferable to silent wraparound

## Recommended Tools

| Category | Tools |
|------|------|
| Linting | clippy |
| Formatting | rustfmt |
| Testing | `cargo test`, proptest, criterion (benchmarking) |
| Safety auditing | Miri, cargo-audit, cargo-deny |
| Async runtime | Tokio, async-std |

## Production Readiness

**Full deployment checklist**: → [extended/checklists.md#production-readiness-checklist]

## Related Resources

[The Rust Book](https://doc.rust-lang.org/book/) | [Rust by Example](https://doc.rust-lang.org/rust-by-example/) | [Rustonomicon (unsafe)](https://doc.rust-lang.org/nomicon/)

## Related Domains

[[go]] | [[python]] | [[aws]]
