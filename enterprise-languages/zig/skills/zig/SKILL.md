---
schema: "1.0"
name: zig
version: "1.0.0"
description: Zig manual memory management, comptime, allocators, and error-union production practices
domain: technology
triggers:
  keywords:
    primary: [zig, comptime, allocator, error union]
    secondary: [zig build, defer, arena allocator, undefined behavior]
  context_boost: [systems programming, no garbage collection, C alternative]
  context_penalty: [rust, go, garbage collected]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Zig

> No hidden control flow, no hidden allocations — which means every allocation and error path is your responsibility, explicitly

## Applicable Scenarios

- Writing or reviewing Zig code for systems programming or as a C replacement
- Choosing and passing the right allocator for a given workload
- Working through `comptime` code for generics or compile-time computation
- Designing error handling with error unions and `try`/`catch`
- Diagnosing memory leaks or undefined behavior differences between build modes

## Core Knowledge

### No Hidden Control Flow, No Hidden Allocations

- Zig deliberately has no operator overloading, no exceptions, no hidden function calls (like C++ destructors or Rust's `Drop`), and no implicit heap allocation anywhere in the language — if a function allocates, it takes an `Allocator` parameter explicitly
- This makes resource usage highly legible at the call site (you can see exactly what allocates), but it means there's no automatic cleanup — `defer` is the idiomatic manual-cleanup mechanism, used explicitly at every allocation site

### Allocators as an Explicit Dependency

| Allocator | Best For |
|------|------|
| **`GeneralPurposeAllocator`** | Default general-purpose use, has built-in leak/safety checking in debug builds |
| **`ArenaAllocator`** | A batch of allocations with the same lifetime, freed all at once — much cheaper than individually freeing many small allocations |
| **`FixedBufferAllocator`** | Allocating from a pre-sized stack/static buffer, no heap involved |
| **`page_allocator`** | Direct OS page allocation, low-level |

Because allocators are passed explicitly (not a global), choosing the wrong one for a workload (e.g., general-purpose for a batch of same-lifetime allocations that an arena would handle far more cheaply) is purely a design decision with a real performance cost, not a language limitation.

### comptime

- `comptime` lets code execute at compile time — Zig's mechanism for generics (a comptime parameter can be a type), constant computation, and compile-time reflection over types
- This replaces the need for a separate macro system or template language, but comptime type errors can be genuinely confusing for newcomers since the "generic function" is really just a regular function whose parameters happen to be evaluated at compile time

### Error Handling: Error Unions

- Zig's error handling model is **error unions** (`!T`, meaning "either an error or a `T`") — explicit, checked at compile time, not exceptions
- `try expr` propagates an error up to the caller automatically (similar in spirit to Rust's `?`); `catch` handles it inline
- There is no "forgot to handle this" silent failure the way exceptions can be silently swallowed — but `try`-propagating an error all the way up without ever handling it meaningfully at any level is still a common design smell

## Best Practices

1. **Pass the allocator that matches the allocation pattern** — arena for batch/same-lifetime allocations, general-purpose for everything else by default
2. **Pair every allocation with a `defer` for its cleanup**, immediately after the allocation, not somewhere else in the function
3. **Use the `GeneralPurposeAllocator`'s safety checks during development** — it catches leaks and use-after-free that release-mode allocators won't
4. **Keep `comptime` code simple and well-commented** — it's powerful but harder to debug than runtime code
5. **Handle errors meaningfully at some level of the call stack** — don't `try`-propagate all the way to `main` without any actual handling if the error is genuinely recoverable
6. **Test in both Debug and ReleaseFast/ReleaseSafe modes** — undefined behavior can differ between them

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Allocating without a corresponding `defer` for cleanup | Pair every allocation with `defer allocator.free(...)` (or equivalent) immediately |
| Using `GeneralPurposeAllocator` for many small, same-lifetime allocations | Use `ArenaAllocator` and free the whole arena at once |
| Assuming release builds behave identically to debug builds | Test in the actual release mode used in production — undefined behavior may not manifest in Debug |
| `try`-propagating every error to the top with no handling anywhere | Handle errors meaningfully at the appropriate layer, not just pass them up indefinitely |
| Writing complex `comptime` logic without testing it in isolation | Build and test comptime-heavy generic code incrementally, verifying compiler output at each step |

## Sharp Edges

### SE-1: Manual Memory Management Without RAII
- **Severity**: critical
- **Situation**: A function allocates memory via an explicit allocator call but a code path (an early return, an error branch) skips the corresponding free, leaking memory that accumulates over the program's lifetime
- **Cause**: Zig has no garbage collector and no automatic RAII-style destructor mechanism (unlike C++/Rust) — `defer` is the manual tool for ensuring cleanup runs, but it must be used correctly and consistently at every allocation site, including every exit path
- **Symptoms**:
  - Memory usage grows over the program's runtime with no corresponding decrease
  - `GeneralPurposeAllocator`'s leak-detection (in debug builds) reports leaked allocations at program exit
- **Solution**: Pair every allocation with an immediate `defer` for its cleanup so it's inseparable from the allocation in the source, and rely on `GeneralPurposeAllocator`'s built-in leak checking during development/testing to catch leaks before release
- **Details**: → [extended/checklists.md#memory-safety-checklist]

### SE-2: Wrong Allocator Choice Hurting Performance
- **Severity**: medium
- **Situation**: Code performs many small, same-lifetime allocations (e.g., parsing a document into many small nodes, all freed together at the end) using a general-purpose allocator, and performance is far worse than it should be due to per-allocation bookkeeping overhead
- **Cause**: A general-purpose allocator has real per-allocation overhead (safety checks, free-list management); an `ArenaAllocator` batches allocations and frees them all at once with dramatically lower overhead for exactly this same-lifetime pattern
- **Symptoms**:
  - Profiling shows allocation/deallocation as a disproportionate share of runtime for a workload with an obvious "allocate a batch, free it all together" shape
- **Solution**: Recognize same-lifetime allocation batches and use an `ArenaAllocator` for them, reserving the general-purpose allocator for allocations with genuinely independent, unpredictable lifetimes

### SE-3: Confusing Compiler Errors From comptime Code
- **Severity**: medium
- **Situation**: A generic function using `comptime` parameters produces a compiler error that's hard to map back to the actual mistake, especially for developers new to Zig's comptime model
- **Cause**: comptime code is regular Zig code evaluated at compile time — errors surface through the same type-checking machinery as runtime code, but because the "generic" behavior is really just compile-time function execution, the error messages can reference intermediate compile-time states that aren't obviously connected to the source line that triggered them
- **Symptoms**:
  - A type error in a generic function points to a line deep inside the function body rather than the call site that produced the invalid instantiation
- **Solution**: Build and test comptime-heavy code incrementally (verify each piece works before composing), keep comptime logic as simple as the problem allows, and read Zig compiler errors from the top of the trace (the ultimate call site) rather than just the innermost reported line

### SE-4: Unhandled Error Unions Propagated Without Real Handling
- **Severity**: medium
- **Situation**: A chain of functions all use `try` to propagate an error union upward, and by the time it reaches `main` (or another top-level boundary), there's no meaningful handling — the error just crashes the program with a generic message, even though it might have been genuinely recoverable at an earlier point
- **Cause**: `try` makes error propagation extremely easy and low-friction, which can lead to a design where errors are never actually *handled*, just passed along until they reach the top by default
- **Symptoms**:
  - Production failures show a generic top-level error message with no context about what specifically went wrong or how to recover
  - An error that a mid-level caller could have retried or worked around instead just propagates and crashes
- **Solution**: Identify the appropriate layer for each error to be meaningfully handled (retried, logged with context, converted to a user-facing message) rather than defaulting to `try`-propagate-everything, and use `catch` with explicit handling at those boundaries

### SE-5: Undefined Behavior Differing Between Build Modes
- **Severity**: high
- **Situation**: A bug involving undefined behavior (e.g., an out-of-bounds access, integer overflow in unchecked arithmetic) doesn't manifest in Debug builds (which include safety checks) but causes silent corruption or a crash in ReleaseFast builds (which strip many of those checks for performance)
- **Cause**: Zig's build modes trade off safety checking for performance — Debug and ReleaseSafe include bounds checking, overflow checking, and other runtime safety assertions that ReleaseFast and ReleaseSmall omit for speed/size, so code relying on those checks to "catch" a bug during testing won't be protected the same way in a stripped release build
- **Symptoms**:
  - "Works in dev, breaks in production" bugs that trace back to a real UB issue masked by Debug-mode safety checks
- **Solution**: Test in the actual build mode used for production (or at minimum ReleaseSafe, which keeps safety checks with better performance than Debug) rather than assuming Debug-mode correctness transfers, and treat any UB caught only by Debug-mode assertions as a real bug to fix, not just an assertion to silence

## Recommended Tools

| Category | Tools |
|------|------|
| Build | `zig build` (native build system, no separate Makefile needed) |
| Testing | Zig's built-in `zig test` |
| Memory safety | `GeneralPurposeAllocator` leak/safety checking, Valgrind |
| Package management | Zig package manager (`build.zig.zon`) |

## Memory Safety

**Full checklist**: → [extended/checklists.md#memory-safety-checklist]

## Related Resources

[Zig Documentation](https://ziglang.org/documentation/master/) | [Zig Learn](https://ziglearn.org/) | [Ziglings (exercises)](https://github.com/ratfactor/ziglings)

## Related Domains

[[rust]] | [[cpp]] | [[go]]
