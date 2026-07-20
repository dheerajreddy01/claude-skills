# C# / .NET — Extended Checklists

## Async Checklist

- [ ] No `async void` methods outside of framework-mandated event handlers
- [ ] No `.Result`/`.Wait()` blocking calls on a `Task` anywhere in the async call chain
- [ ] Library code uses `ConfigureAwait(false)` where context capture isn't needed
- [ ] Every `IDisposable` is wrapped in `using`/`await using`, not left to the finalizer
- [ ] Deferred LINQ queries materialized (`.ToList()`/`.ToArray()`) when a stable snapshot is required
- [ ] `CancellationToken` propagated through async call chains for anything cancellable
- [ ] Unhandled exception/task fault logging configured (`TaskScheduler.UnobservedTaskException` or equivalent) as a safety net

## DI & Production Checklist

- [ ] No scoped or transient service injected directly into a singleton (captive dependency)
- [ ] `IServiceScopeFactory` used where a singleton genuinely needs scoped-service access
- [ ] Nullable reference types enabled for new projects
- [ ] Roslyn analyzers / `dotnet format` run in CI
- [ ] Connection pooling and `HttpClient` reuse (via `IHttpClientFactory`, not `new HttpClient()` per call) configured correctly
- [ ] Structured logging in place, not bare `Console.WriteLine`
- [ ] Health check endpoints exposed for orchestrated deployments
