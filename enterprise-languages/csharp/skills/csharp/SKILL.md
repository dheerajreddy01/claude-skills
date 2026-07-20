---
schema: "1.0"
name: csharp
version: "1.0.0"
description: C#/.NET async patterns, LINQ semantics, dependency injection lifetimes, and production practices
domain: technology
triggers:
  keywords:
    primary: [csharp, c#, dotnet, .net, asp.net core]
    secondary: [async await, LINQ, nuget, entity framework, nullable reference types]
  context_boost: [backend, web api, enterprise application]
  context_penalty: [java, python, javascript]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# C# / .NET

> A managed runtime that still lets you leak resources, deadlock, and swallow exceptions if you're not careful

## Applicable Scenarios

- Writing or reviewing C#/.NET application and API code
- Diagnosing async/await deadlocks or unobserved exceptions
- Designing dependency injection lifetimes in ASP.NET Core
- Reviewing LINQ queries for deferred-execution correctness
- Managing unmanaged/disposable resources correctly

## Core Knowledge

### CLR & Garbage Collection

- The CLR manages memory via a generational, tracing garbage collector — most objects never need manual cleanup, but anything wrapping an unmanaged resource (file handles, sockets, DB connections) implements `IDisposable` and needs explicit disposal
- The GC does not know about unmanaged resources; a class holding one must implement `IDisposable` (and a finalizer as a backstop) or that resource leaks even though the managed wrapper object eventually gets collected

### Async/Await

- `async`/`await` compiles to a state machine; `Task`/`Task<T>` represent the eventual result — awaiting doesn't block a thread, it schedules a continuation
- `async void` should be reserved for event handlers only — exceptions thrown inside an `async void` method can't be awaited/caught by the caller and instead crash the process or get lost, unlike `async Task` where the exception is captured on the returned `Task`
- Mixing blocking calls (`.Result`, `.Wait()`) with `async` code in a context that has a `SynchronizationContext` (classic ASP.NET, WinForms/WPF) is the classic cause of deadlocks

### LINQ

- Most LINQ query operators are **deferred** — a query variable doesn't execute until enumerated (`foreach`, `.ToList()`, etc.); the same query variable re-executed twice re-runs the whole pipeline, including any side effects or database round-trips (via EF Core) each time
- `IQueryable` (translated to SQL by EF Core) vs `IEnumerable` (executed in-memory) look identical in code but behave very differently — a LINQ method not translatable to SQL can silently force the rest of the query to execute client-side, pulling far more data than intended

### Dependency Injection Lifetimes

| Lifetime | Behavior |
|------|------|
| **Transient** | New instance every time it's requested |
| **Scoped** | One instance per request (in ASP.NET Core) |
| **Singleton** | One instance for the application's lifetime |

Injecting a scoped or transient service into a singleton captures it for the singleton's entire lifetime — a "captive dependency" that can hold stale data or an already-disposed scoped resource (like a `DbContext`) indefinitely.

## Best Practices

1. **Never use `async void`** except for top-level event handlers — use `async Task` everywhere else so exceptions propagate correctly
2. **Always dispose `IDisposable` resources** via `using`/`await using`, don't rely on the finalizer as the primary cleanup path
3. **Avoid blocking on async code** (`.Result`, `.Wait()`) — await all the way up the call stack
4. **Materialize LINQ queries deliberately** (`.ToList()`) when you need a stable snapshot, rather than re-enumerating a deferred query
5. **Match DI lifetimes to actual usage** — never inject a scoped/transient service into a singleton without understanding the captive dependency consequence
6. **Enable nullable reference types** (`<Nullable>enable</Nullable>`) for new projects to catch a large class of null-reference bugs at compile time

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `async void` on a non-event-handler method | Use `async Task`, so exceptions propagate to the caller |
| `.Result`/`.Wait()` on a `Task` in a context with a `SynchronizationContext` | Await asynchronously all the way up, or use `ConfigureAwait(false)` in library code |
| Re-enumerating a deferred LINQ query expecting cached results | Call `.ToList()`/`.ToArray()` once to materialize when a stable snapshot is needed |
| Injecting a scoped `DbContext` into a singleton service | Inject `IServiceScopeFactory` and create a scope per operation instead |
| Forgetting `using` on an `IDisposable` (file, connection, etc.) | Wrap in `using`/`await using` so disposal happens deterministically |

## Sharp Edges

### SE-1: Async Void Swallowing Exceptions
- **Severity**: critical
- **Situation**: An `async void` method throws an exception, and instead of being observable by any caller, it either crashes the process (unhandled) or is silently lost, depending on context
- **Cause**: `async void` methods have no `Task` for the caller to await or attach exception handling to — the compiler-generated state machine has nowhere to route the exception except straight to the synchronization context or `AppDomain`-level unhandled exception handling
- **Symptoms**:
  - An error clearly logged as happening inside an async method never appears in any catch block up the call stack
  - The process crashes unexpectedly from what looks like a handled scenario
- **Solution**: Use `async Task` for every async method except top-level UI/event handlers that are required by the framework to return `void`; wrap any unavoidable `async void` handler's body in a top-level try/catch
- **Details**: → [extended/checklists.md#async-checklist]

### SE-2: Deadlocks From Blocking on Async Code
- **Severity**: critical
- **Situation**: Calling `.Result` or `.Wait()` on a `Task` from a thread that has a captured `SynchronizationContext` (classic ASP.NET MVC, WPF/WinForms UI thread) hangs the application entirely
- **Cause**: The awaited task's continuation is scheduled to resume on the captured context, but that context's thread is blocked waiting on `.Result` — the continuation can never run because the very thread it needs is stuck waiting for it
- **Symptoms**:
  - The application hangs indefinitely on a specific code path, with no exception or timeout
  - The hang disappears if the blocking call is replaced with `await`
- **Solution**: Await asynchronously through the entire call chain instead of blocking; if a synchronous call site is unavoidable, use `ConfigureAwait(false)` in library code to avoid capturing the context in the first place (ASP.NET Core doesn't have this specific issue since it has no `SynchronizationContext`, but the discipline still matters for portability)

### SE-3: LINQ Deferred Execution Re-Querying Unexpectedly
- **Severity**: high
- **Situation**: A LINQ query against an EF Core `DbSet` is enumerated twice (e.g., once to check `.Any()` and again in a `foreach`), and it silently issues two separate database round-trips — or worse, returns different results if the underlying data changed between the two enumerations
- **Cause**: LINQ query operators build a deferred expression tree; each enumeration re-executes the full query against its source, including a live database in EF Core's case
- **Symptoms**:
  - Database query logs show the same query executing multiple times per logical operation
  - A "consistent" view of data isn't actually consistent because it was fetched at two different points in time
- **Solution**: Materialize the query once with `.ToList()`/`.ToArray()` when a stable, single-fetch snapshot is needed, and reuse that materialized collection for subsequent operations

### SE-4: Captive Dependency From DI Lifetime Mismatch
- **Severity**: high
- **Situation**: A singleton service has a scoped service (commonly a `DbContext`) injected via constructor injection, and that scoped instance is now held for the singleton's entire application lifetime instead of being recreated per request
- **Cause**: The DI container resolves the singleton once at first use, capturing whatever scoped instance was active at that moment — the singleton never gets a fresh scoped instance again
- **Symptoms**:
  - A `DbContext`-backed singleton throws `ObjectDisposedException` once the original scope ends
  - Data appears stale because the same scoped instance (and its change tracker/cache) is reused indefinitely
- **Solution**: Never inject a scoped service directly into a singleton; inject `IServiceScopeFactory` and create a new scope (and resolve the scoped service from it) for each operation that needs one

### SE-5: Resource Leaks From Missing `using`
- **Severity**: medium
- **Situation**: A class holding an unmanaged resource (file stream, DB connection, HTTP client misuse) isn't wrapped in `using`, and under load the application exhausts file handles or connections well before any GC-driven finalization would have cleaned them up
- **Cause**: The garbage collector doesn't proactively finalize objects promptly — finalizers run on a background thread on the GC's schedule, which can lag far behind the rate at which a busy application is allocating disposable resources
- **Symptoms**:
  - `IOException: too many open files` or connection pool exhaustion under sustained load, despite objects eventually being collected
- **Solution**: Wrap every `IDisposable` in `using`/`await using` for deterministic, immediate disposal — don't rely on the finalizer as anything other than a last-resort backstop

## Recommended Tools

| Category | Tools |
|------|------|
| Build/package | dotnet CLI, NuGet |
| Testing | xUnit, NUnit, Moq |
| Analysis | Roslyn analyzers, `dotnet-trace`, BenchmarkDotNet |
| ORM | Entity Framework Core |

## Production Readiness

**Full checklist**: → [extended/checklists.md#async-checklist]

## Related Resources

[.NET Docs](https://learn.microsoft.com/dotnet/) | [C# Language Reference](https://learn.microsoft.com/dotnet/csharp/) | [ASP.NET Core Docs](https://learn.microsoft.com/aspnet/core/)

## Related Domains

[[java]] | [[azure]] | [[postgresql]] | [[docker]]
