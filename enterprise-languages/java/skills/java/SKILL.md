---
schema: "1.0"
name: java
version: "1.0.0"
description: Java, JVM memory/GC model, Spring ecosystem, concurrency, and production service practices
domain: technology
triggers:
  keywords:
    primary: [java, jvm, spring, maven, gradle, hibernate]
    secondary: [junit, thread pool, garbage collection, classpath, bean]
  context_boost: [enterprise, microservice, backend, spring boot]
  context_penalty: [python, javascript, rust]
  priority: high
dependencies:
  software-skills: [git-workflows, testing-strategies]
author: claude-domain-skills
---

# Java

> The JVM, the Spring ecosystem, and disciplined concurrency at enterprise scale

## Applicable Scenarios

- Writing or reviewing Java application/service code
- Diagnosing memory leaks, GC pauses, or classpath conflicts
- Designing concurrency with threads, executors, or virtual threads (Java 21+)
- Working with Spring Boot, dependency injection, and JPA/Hibernate
- Setting up build tooling (Maven/Gradle) and testing

## Core Knowledge

### Build Tools

| Tool | Notes |
|------|------|
| **Maven** | XML-based, convention over configuration, mature dependency resolution |
| **Gradle** | Groovy/Kotlin DSL, faster incremental builds, more flexible |

Both resolve transitive dependencies from Maven Central by default — dependency version conflicts ("dependency hell") are resolved by nearest-wins (Maven) or explicit resolution strategies (Gradle); don't assume the "right" version wins by default.

### JVM Memory & GC

| Concept | Detail |
|------|------|
| **Heap** | Object storage; divided into young/old generations for most collectors |
| **Garbage Collectors** | G1 (default, balanced), ZGC/Shenandoah (low-pause, large heaps), Parallel (throughput-focused) |
| **`-Xmx`/`-Xms`** | Max/initial heap size — set explicitly in production, don't rely on JVM defaults which are host-memory-percentage-based |
| **Memory leaks** | Usually not literal leaks (JVM has GC) but unintended object retention — static collections, unclosed listeners, ThreadLocal not cleared |

### Concurrency

| Approach | Use |
|------|------|
| **`Thread`/`Runnable`** | Low-level, rarely used directly today |
| **`ExecutorService`** | Managed thread pools — standard for bounded concurrent work |
| **Virtual Threads (Java 21+)** | Lightweight, JVM-managed threads for massive I/O-bound concurrency without the executor-tuning overhead of platform threads |
| **`CompletableFuture`** | Composable async pipelines |
| **`java.util.concurrent`** | `ConcurrentHashMap`, `AtomicInteger`, `CountDownLatch`, etc. — prefer these over hand-rolled synchronization |

### Spring Ecosystem

- **Dependency Injection**: beans are managed by the `ApplicationContext`; constructor injection is preferred over field injection (`@Autowired` on fields hides required dependencies and complicates testing)
- **Spring Boot**: auto-configuration + embedded server; `application.yml`/`application.properties` for environment-specific config, profiles (`@Profile`) for environment switching
- **Spring Data JPA / Hibernate**: ORM that can silently generate inefficient SQL (see Sharp Edges — N+1) if entity relationships and fetch strategies aren't reviewed

## Best Practices

1. **Prefer constructor injection** over field injection — makes dependencies explicit and testable without a Spring context
2. **Set explicit heap and GC flags in production** — don't rely on container-inferred defaults for critical services
3. **Use `try-with-resources`** for anything `Closeable` — connections, streams, file handles
4. **Favor immutability** — `final` fields, immutable value objects, reduces a whole class of concurrency bugs
5. **Log with structured context** (SLF4J + MDC), not string concatenation
6. **Write tests against the thinnest slice that proves behavior** — avoid spinning up the full Spring context for unit tests that don't need it

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Catching `Exception` broadly and swallowing it | Catch specific exception types, log and rethrow or handle deliberately |
| `@Autowired` field injection everywhere | Constructor injection — explicit, testable, immutable-friendly |
| Lazy-loading JPA associations accessed in a loop (N+1 queries) | Use `JOIN FETCH`, `@EntityGraph`, or DTO projections |
| Comparing boxed `Integer`/`Long` with `==` | Use `.equals()` — boxed types outside the `-128..127` cache range aren't reference-equal |
| Unbounded thread pool creation (`new Thread()` per request) | Use a bounded `ExecutorService` or virtual threads for I/O-bound fan-out |

## Sharp Edges

### SE-1: The N+1 Query Problem
- **Severity**: critical
- **Situation**: Fetching a list of entities, then accessing a lazy-loaded association on each one in a loop, triggers one query per entity instead of one query total
- **Cause**: JPA/Hibernate lazy loading defers association fetches until accessed; a loop over N parent entities accessing a child association issues N additional queries
- **Symptoms**:
  - API response time scales linearly with result set size
  - Database query logs show a burst of near-identical `SELECT` statements
- **Solution**: Use `JOIN FETCH` in JPQL, `@EntityGraph`, or map to a DTO projection that fetches everything in one query
- **Details**: → [extended/checklists.md#jpa-performance-checklist]

### SE-2: NullPointerException From Boxed Type Auto-Unboxing
- **Severity**: high
- **Situation**: A `Long`/`Integer` field that's `null` (e.g. from a database column or DTO) gets auto-unboxed in an arithmetic or comparison expression, throwing an NPE with a vague stack trace
- **Cause**: Java silently auto-unboxes wrapper types in numeric contexts; if the wrapper is `null`, unboxing throws NPE
- **Symptoms**:
  - `NullPointerException` at a line with no obvious null dereference (e.g. `if (someLong > 0)`)
- **Solution**: Guard with an explicit null check before unboxing, prefer primitive types where `null` isn't a meaningful state, or use `Optional` for genuinely optional values

### SE-3: Memory Leaks via Unbounded Static Collections
- **Severity**: high
- **Situation**: Heap usage grows steadily over the app's lifetime despite the GC running, eventually leading to `OutOfMemoryError`
- **Cause**: Objects held by a `static` collection, a registered listener never unregistered, or a `ThreadLocal` never cleared are reachable forever, so the GC can never collect them
- **Symptoms**:
  - Heap dumps show a single collection/map holding a growing number of objects
  - GC pauses lengthen over time as old-gen fills up
- **Solution**: Avoid unbounded static caches (use a bounded cache like Caffeine with eviction), always unregister listeners/callbacks, clear `ThreadLocal` values in a `finally` block (critical in thread-pooled environments where threads are reused)

### SE-4: Checked Exception Overuse Forces Bad Handling
- **Severity**: medium
- **Situation**: A method signature riddled with checked exceptions forces every caller up the stack to either declare, wrap, or swallow them — teams end up swallowing with an empty catch block just to compile
- **Cause**: Checked exceptions are part of the method signature, and Java doesn't allow ignoring them silently at the call site
- **Symptoms**:
  - Empty or log-only `catch` blocks scattered through the codebase
  - Exception types that don't map to anything the caller can meaningfully act on
- **Solution**: Reserve checked exceptions for conditions the caller can realistically recover from; use unchecked (`RuntimeException` subclasses) for programmer errors and unrecoverable conditions, and handle/log deliberately at a real boundary (e.g., a global `@ControllerAdvice` handler)

### SE-5: Thread Pool Starvation and Deadlock
- **Severity**: critical
- **Situation**: Tasks submitted to a fixed-size `ExecutorService` block waiting on other tasks submitted to the *same* pool, exhausting all threads and deadlocking the whole pool
- **Cause**: A bounded thread pool has no notion of task priority or dependency — if every thread is blocked waiting on a task still sitting in the queue, nothing can make progress
- **Symptoms**:
  - Application hangs under load with CPU near-idle
  - Thread dump shows all pool threads `WAITING` on a `Future.get()` for tasks still `QUEUED`
- **Solution**: Never submit a task to a pool and block-wait on another task submitted to the *same* pool; use separate pools for dependent stages, or switch to virtual threads (Java 21+) which don't have this fixed-capacity starvation mode for I/O-bound waits

## Recommended Tools

| Category | Tools |
|------|------|
| Build | Maven, Gradle |
| Testing | JUnit 5, Mockito, Testcontainers |
| Profiling | JFR (Java Flight Recorder), VisualVM, async-profiler |
| Static analysis | SpotBugs, Error Prone, SonarQube |

## Production Readiness

**Full deployment checklist**: → [extended/checklists.md#production-readiness-checklist]

## Related Resources

[Java Docs](https://docs.oracle.com/en/java/) | [Spring Docs](https://docs.spring.io/spring-framework/reference/) | [Baeldung](https://www.baeldung.com/)

## Related Domains

[[go]] | [[python]] | [[aws]]
