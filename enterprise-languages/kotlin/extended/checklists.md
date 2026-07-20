# Kotlin — Extended Checklists

## Coroutine Safety Checklist

- [ ] No coroutine launched in `GlobalScope`; every coroutine tied to a scope matching its intended lifecycle
- [ ] `viewModelScope`/`lifecycleScope` (or an explicitly cancelled custom scope) used in Android components
- [ ] Blocking I/O wrapped in `withContext(Dispatchers.IO)`, never run directly on `Main` or `Default`
- [ ] Cancellation checked/respected in long-running coroutine loops (`ensureActive()`/`isActive`)
- [ ] Structured concurrency preserved — no coroutine builder used in a way that detaches it from its parent's lifecycle
- [ ] Exception handling in coroutines uses `CoroutineExceptionHandler` or structured try/catch, not silently swallowed failures
- [ ] Flow collection scoped correctly, cancelled when the collecting component is destroyed

## Null Safety & Interop Checklist

- [ ] No `!!` outside of test code or already-verified invariants
- [ ] Java interop boundaries have explicit null checks or `@Nullable`/`@NotNull` annotations on the Java side
- [ ] Data classes used as `HashMap`/`HashSet` keys use only `val` (immutable) properties
- [ ] Sealed classes used for closed state/result modeling, with exhaustive `when` (no unnecessary `else` masking missing cases)
- [ ] detekt/ktlint run in CI to catch style and common-pitfall patterns automatically
