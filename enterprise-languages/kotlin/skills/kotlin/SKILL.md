---
schema: "1.0"
name: kotlin
version: "1.0.0"
description: Kotlin null safety, structured concurrency with coroutines, and Android/JVM production practices
domain: technology
triggers:
  keywords:
    primary: [kotlin, android, jetpack compose, coroutine]
    secondary: [gradle, null safety, ktor, sealed class]
  context_boost: [android app, jvm backend, mobile development]
  context_penalty: [swift, java, javascript]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# Kotlin

> Null safety is enforced at compile time — until Java interop or `!!` quietly hands that guarantee back

## Applicable Scenarios

- Writing or reviewing Kotlin for Android apps or JVM backends
- Designing coroutine-based concurrency and structured scope lifetimes
- Interoperating with existing Java code/libraries safely
- Modeling domain state with sealed classes and data classes
- Diagnosing crashes from null pointer exceptions or coroutine leaks

## Core Knowledge

### Null Safety

- Types are non-nullable by default (`String`); nullable types are explicit (`String?`) and the compiler forces a null check before use
- The `!!` operator forcibly asserts non-null, throwing `NullPointerException` if wrong — it's an explicit opt-out of the entire safety guarantee the type system provides
- **Platform types** (from Java interop, denoted `String!`) carry no null information — Kotlin can't verify nullability for values coming from Java, so the compiler doesn't enforce a check there

### Coroutines & Structured Concurrency

| Concept | Detail |
|------|------|
| **`suspend fun`** | A function that can pause/resume without blocking the underlying thread |
| **CoroutineScope** | Defines the lifetime a coroutine is tied to — cancelling the scope cancels all coroutines launched in it |
| **Structured concurrency** | Coroutines launched in a scope are children of it; a parent scope won't complete until its children do, and cancellation propagates down |
| **Dispatchers** | `Main` (UI thread), `IO` (blocking I/O), `Default` (CPU-bound) — choosing the wrong one blocks work that should be concurrent |

Launching into `GlobalScope` (or any scope that outlives its intended caller) breaks structured concurrency — the coroutine keeps running even after the screen/component that started it is gone.

### Sealed Classes & Data Classes

- **Sealed classes** restrict a type hierarchy to a known, closed set of subtypes — `when` expressions over them can be exhaustive (compiler-checked) without an `else` branch, which is the main tool for modeling state machines/results safely
- **Data classes** auto-generate `equals`/`hashCode`/`copy` based on constructor properties — using a `var` (mutable) property in a data class used as a `HashMap`/`HashSet` key is dangerous, since mutating it after insertion breaks the hash-based lookup

### Java Interop

- Kotlin compiles to JVM bytecode and interoperates directly with Java, but Java has no concept of Kotlin's null safety — every value crossing that boundary is a platform type until annotated (`@Nullable`/`@NotNull`) or explicitly checked

## Best Practices

1. **Avoid `!!` outside of tests/prototypes** — handle the null case explicitly with `?.`, `?:`, or a proper branch
2. **Never launch coroutines in `GlobalScope`** — tie every coroutine to a scope matching its intended lifecycle (`viewModelScope`, `lifecycleScope`, or a structured parent scope)
3. **Treat Java interop return values as untrusted** — add explicit null checks or annotations at the boundary rather than assuming platform types are safe
4. **Use sealed classes for closed state/result modeling** so `when` exhaustiveness catches missing cases at compile time
5. **Avoid mutable properties in data classes used as map/set keys**
6. **Use the right dispatcher for the work** — `Dispatchers.IO` for blocking calls, `Dispatchers.Default` for CPU-bound work, never blocking calls on `Dispatchers.Main`

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using `!!` to silence a nullable-type compiler warning | Handle the null case explicitly with safe calls or a conditional |
| Launching a coroutine in `GlobalScope` | Launch in a scope tied to the component's actual lifecycle |
| Assuming a Java library's return value is non-null | Treat platform types as nullable until verified, add explicit checks |
| Mutable `var` property in a data class used as a `HashMap` key | Use `val` (immutable) properties for anything used as a key |
| Blocking I/O call inside `Dispatchers.Main` or `Dispatchers.Default` | Use `Dispatchers.IO` (or `withContext(Dispatchers.IO)`) for blocking work |

## Sharp Edges

### SE-1: Coroutine Scope Leaks From `GlobalScope`
- **Severity**: critical
- **Situation**: A coroutine launched in `GlobalScope` inside an Android Activity/Fragment/ViewModel keeps running after the component is destroyed, holding a reference to it and continuing to do work (or crashing when it tries to update destroyed UI)
- **Cause**: `GlobalScope` is tied to the application's entire process lifetime, not any specific component — structured concurrency's automatic cancellation-on-scope-end never applies to it
- **Symptoms**:
  - `IllegalStateException`/crashes referencing a destroyed Fragment/Activity from a background coroutine
  - Memory leaks traced to a coroutine holding a reference to a long-gone UI component
- **Solution**: Launch coroutines in a scope tied to the actual component lifecycle (`viewModelScope`, `lifecycleScope`, or a custom `CoroutineScope` explicitly cancelled in `onCleared`/`onDestroy`), never `GlobalScope`, for anything with a bounded lifetime
- **Details**: → [extended/checklists.md#coroutine-safety-checklist]

### SE-2: Platform Type NPEs From Java Interop
- **Severity**: high
- **Situation**: A value returned from a Java library (a platform type, shown as `String!` in Kotlin) is used without a null check because Kotlin's type system implies safety, and it turns out to be `null` at runtime, throwing an NPE the compiler never warned about
- **Cause**: Kotlin cannot verify nullability across the Java interop boundary unless the Java code is annotated with `@Nullable`/`@NotNull` — platform types silently skip the compiler's null-safety enforcement
- **Symptoms**:
  - An NPE at a line that "should be null-safe Kotlin," tracing back to an unchecked Java library return value
- **Solution**: Treat every Java interop boundary as untrusted — add explicit null checks, or wrap Java APIs in a thin Kotlin layer that asserts/handles nullability once, rather than trusting platform types throughout the codebase

### SE-3: `!!` Crashing in Production
- **Severity**: high
- **Situation**: Code using `!!` to satisfy the compiler during development works fine until a real-world input path produces `null`, and the app crashes with a `NullPointerException` in production
- **Cause**: `!!` is a deliberate escape hatch from null safety — it converts a compile-time-preventable bug into a runtime crash, and it's easy to reach for it just to make code compile without addressing the actual null case
- **Symptoms**:
  - Crash reports show `NullPointerException` at a `!!` call site, correlated with a specific input/state combination not exercised in testing
- **Solution**: Replace `!!` with explicit handling (`?:` for a default, `?.let {}` for conditional execution, or an early return/exception with a meaningful message) everywhere except test code and cases where nullness is a genuine, already-checked invariant

### SE-4: Blocking Calls Inside Coroutines Stalling the Dispatcher
- **Severity**: high
- **Situation**: A `suspend fun` calls a blocking API (synchronous network/file I/O) while running on `Dispatchers.Default` or `Dispatchers.Main`, and it blocks the limited thread pool backing that dispatcher, starving unrelated coroutines
- **Cause**: Coroutine dispatchers run on a bounded thread pool; a blocking call doesn't yield the thread the way a genuine `suspend` function does, so it monopolizes one of a small number of shared threads
- **Symptoms**:
  - UI freezes (if blocking `Dispatchers.Main`) or unrelated background coroutines stall (if blocking `Dispatchers.Default`)
  - Symptom appears only under concurrent load, not in isolated testing
- **Solution**: Wrap blocking calls in `withContext(Dispatchers.IO)`, which uses a larger, elastic thread pool designed for blocking work, and never perform blocking I/O directly on `Main` or `Default`

### SE-5: Data Class Equality Breaking With Mutable Properties
- **Severity**: medium
- **Situation**: A data class with a `var` property is inserted into a `HashSet`/used as a `HashMap` key, the property is later mutated, and the object can no longer be found by lookup (or a `contains()`/`remove()` silently fails) because its hash code changed after insertion
- **Cause**: Data class `hashCode()`/`equals()` are generated from all constructor properties — hash-based collections index by hash code at insertion time and don't re-index if that hash changes later
- **Symptoms**:
  - `set.contains(x)` returns `false` for an object that's provably still in the set
  - Duplicate-seeming entries appear in a `HashSet` after mutation
- **Solution**: Use only `val` (immutable) properties in data classes used as hash-based collection keys; if mutable state is genuinely needed, use a regular class with explicit identity-based equality or a stable, immutable key

## Recommended Tools

| Category | Tools |
|------|------|
| Build | Gradle (Kotlin DSL) |
| Testing | Kotest, MockK, JUnit 5 |
| Static analysis | detekt, ktlint |
| Android | Jetpack Compose, Android Studio |

## Coroutine Safety

**Full checklist**: → [extended/checklists.md#coroutine-safety-checklist]

## Related Resources

[Kotlin Docs](https://kotlinlang.org/docs/home.html) | [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html) | [Android Kotlin Guides](https://developer.android.com/kotlin)

## Related Domains

[[java]] | [[swift]] | [[gitlab]] | [[sqlite]]
