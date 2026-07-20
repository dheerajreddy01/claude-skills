---
schema: "1.0"
name: scala
version: "1.0.0"
description: Scala functional/OOP hybrid design, implicit resolution, and JVM concurrency practices with Akka/Spark
domain: technology
triggers:
  keywords:
    primary: [scala, akka, spark, sbt]
    secondary: [case class, implicit, functional programming, pattern matching]
  context_boost: [jvm, data engineering, actor model]
  context_penalty: [java, kotlin, python]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# Scala

> Powerful abstractions (implicits, the type system) cut both ways — they can also hide what's actually running

## Applicable Scenarios

- Writing or reviewing Scala for JVM backends, Spark data pipelines, or Akka actor systems
- Debugging implicit resolution surprises
- Designing immutable data models with case classes and pattern matching
- Composing `Future`-based asynchronous code correctly
- Diagnosing stack overflows or actor mailbox growth

## Core Knowledge

### Functional + OOP Hybrid

- Values are immutable (`val`) by default; mutability (`var`) is opt-in and generally discouraged outside tight local scopes
- **Case classes** provide structural equality, pattern-matching support, and immutable-by-default fields — the standard way to model data
- **Pattern matching** (`match`/`case`) is exhaustiveness-checked by the compiler for sealed trait hierarchies, similar in spirit to Kotlin's sealed classes

### Implicits

- Implicit parameters/conversions let the compiler fill in arguments or convert types automatically based on what's in scope — powerful for things like `ExecutionContext` threading or type-class-style polymorphism, but resolution is based on scope and priority rules that aren't always obvious at the call site
- Ambiguous or unexpected implicit resolution is a well-known source of confusion — the "correct" implicit compiling successfully doesn't mean it's the intended one if multiple candidates are in scope

### Concurrency: Future and Akka

| Mechanism | Model |
|------|------|
| **`Future[T]`** | Represents an eventually-available value; composition via `map`/`flatMap`/`for`-comprehensions requires an implicit `ExecutionContext` |
| **Akka Actors** | Message-passing concurrency — each actor processes one message at a time from its mailbox, no shared mutable state between actors |

`Future` composition executes on whatever `ExecutionContext` is implicitly in scope — using the wrong one (e.g., the global fork-join pool for blocking I/O) causes the same thread-starvation problems as blocking a coroutine dispatcher.

### Recursion & the JVM Stack

- Scala supports tail-call optimization for **directly self-recursive** functions annotated with `@tailrec` (and the compiler verifies eligibility) — but this doesn't extend to mutual recursion or any call that isn't in tail position, which still risks `StackOverflowError` on deep recursion

## Best Practices

1. **Prefer explicit type annotations on public APIs** even though Scala infers types — makes implicit resolution and API contracts legible to callers
2. **Keep implicit scope narrow and well-documented** — avoid implicit conversions that silently change types in surprising ways
3. **Use `@tailrec`** on recursive functions expected to handle large inputs, and let the compiler verify tail-call eligibility
4. **Choose the right `ExecutionContext`** for `Future`-based work — never run blocking I/O on the default global dispatcher
5. **Model domain state with sealed traits + case classes**, and rely on exhaustive pattern matching to catch missing cases at compile time
6. **Keep Akka actor mailboxes bounded** where backpressure matters, rather than assuming unbounded buffering is free

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Blocking I/O executed on the default `ExecutionContext.global` | Use a dedicated `ExecutionContext` backed by a thread pool sized for blocking work |
| Deep recursion without `@tailrec`, assuming the compiler optimizes it automatically | Verify tail-call eligibility explicitly with `@tailrec`, restructure to an iterative or trampolined form if it doesn't qualify |
| Relying on implicit conversions for readability | Prefer explicit conversions/calls; reserve implicits for well-scoped, well-understood patterns (type classes, `ExecutionContext`) |
| Unbounded Akka actor mailboxes for high-throughput message producers | Configure bounded mailboxes with an explicit backpressure/overflow strategy |
| Using `==` on objects without case-class structural equality | Use case classes for value types needing structural equality, or override `equals`/`hashCode` deliberately |

## Sharp Edges

### SE-1: Implicit Resolution Picking the Wrong Value Silently
- **Severity**: high
- **Situation**: Code compiles and runs, but uses an unintended implicit value (a different `ExecutionContext`, a different type-class instance) because it was higher priority in scope resolution than the one the author actually intended
- **Cause**: Implicit resolution follows a defined but non-obvious priority order (local scope, imported scope, companion objects) — a value can satisfy the type-checker while not being the one a reader would expect
- **Symptoms**:
  - Behavior differs subtly from what the code "looks like" it should do (e.g., blocking work runs on the wrong thread pool)
  - Adding an unrelated import changes behavior at a distant call site because it introduced a higher-priority implicit
- **Solution**: Keep implicit scope narrow and explicit (prefer local, clearly-named implicit vals over broad wildcard imports), and when debugging unexpected behavior, use `-Xlog-implicits` (or IDE "show implicit" tooling) to see exactly which implicit was actually selected
- **Details**: → [extended/checklists.md#concurrency-and-correctness-checklist]

### SE-2: Stack Overflow From Non-Tail-Recursive Functions
- **Severity**: high
- **Situation**: A recursive function that processes a large input (deep tree, long list) throws `StackOverflowError` in production despite working fine on small test inputs
- **Cause**: The JVM stack has a fixed depth, and Scala only optimizes directly self-recursive tail calls (verified by `@tailrec`) — any recursion that isn't in tail position, or is mutually recursive, still consumes stack frames proportional to depth
- **Symptoms**:
  - `StackOverflowError` correlated with unusually large/deep input, absent in typical development-sized test data
- **Solution**: Annotate recursive functions with `@tailrec` so the compiler verifies eligibility at compile time (a compile error if it isn't actually tail-recursive), and restructure non-tail-recursive or mutually recursive functions to an iterative form or a trampoline if deep recursion on large inputs is expected

### SE-3: Blocking Inside `Future` Composition Starving the Thread Pool
- **Severity**: high
- **Situation**: A `Future`-based pipeline calls a blocking operation (JDBC call, synchronous HTTP client) inside a `map`/`flatMap` without a dedicated execution context, and under load the shared thread pool backing `Future`s becomes exhausted, stalling unrelated concurrent work
- **Cause**: `Future` composition runs on whatever `ExecutionContext` is implicitly in scope — the commonly-used `ExecutionContext.global` is a fork-join pool sized for CPU-bound work, not built to have threads parked on blocking calls
- **Symptoms**:
  - Application-wide latency degradation under load that doesn't correlate with CPU usage
  - Unrelated `Future`-based code slows down when a specific blocking-heavy code path is exercised concurrently
- **Solution**: Run blocking calls on a dedicated `ExecutionContext` backed by a bounded or cached thread pool sized for blocking work, keeping the default/global execution context reserved for genuinely non-blocking, CPU-bound composition

### SE-4: Unbounded Akka Actor Mailbox Growth
- **Severity**: high
- **Situation**: An actor receiving messages faster than it can process them accumulates an ever-growing mailbox, and memory usage climbs until the JVM runs out of heap
- **Cause**: Akka's default mailbox is unbounded — nothing backpressures a fast producer sending to a slow-consuming actor unless a bounded mailbox with an explicit overflow strategy is configured
- **Symptoms**:
  - Memory usage grows correlated with message throughput imbalance between producer and a specific actor
  - `OutOfMemoryError` traced to a large retained mailbox on a specific actor
- **Solution**: Configure bounded mailboxes with an explicit overflow strategy (drop, fail, or a custom backpressure mechanism) for actors that could plausibly receive messages faster than they process them, and monitor mailbox size as a standard operational metric

### SE-5: Type Erasure Breaking Generic Pattern Matching
- **Severity**: medium
- **Situation**: A pattern match on a generic type parameter (e.g., matching `case list: List[String] =>` distinctly from `case list: List[Int] =>`) compiles with an "unchecked" warning and behaves incorrectly at runtime — both patterns can match the same list regardless of its actual element type
- **Cause**: The JVM erases generic type parameters at runtime (type erasure), so `List[String]` and `List[Int]` are indistinguishable to a runtime pattern match — the compiler can only check the outer type (`List`), not the type parameter
- **Symptoms**:
  - A pattern match that "looks" type-specific matches regardless of the actual generic parameter, and a `ClassCastException` occurs later when the wrongly-matched branch tries to use an element as the assumed type
  - The compiler emits an "unchecked" warning that gets ignored
- **Solution**: Avoid pattern matching on generic type parameters directly; use a `TypeTag`/`ClassTag` (or restructure with a sealed trait wrapper carrying explicit type information) when runtime type discrimination on generics is genuinely required, and don't suppress "unchecked" warnings without addressing the underlying erasure issue

## Recommended Tools

| Category | Tools |
|------|------|
| Build | sbt, Mill |
| Testing | ScalaTest, munit, ScalaCheck (property-based) |
| Static analysis | Scalafix, Scalafmt, WartRemover |
| Big data/actors | Apache Spark, Akka |

## Concurrency & Correctness

**Full checklist**: → [extended/checklists.md#concurrency-and-correctness-checklist]

## Related Resources

[Scala Docs](https://docs.scala-lang.org/) | [Akka Documentation](https://doc.akka.io/docs/akka/current/) | [Scala Style Guide](https://docs.scala-lang.org/style/)

## Related Domains

[[java]] | [[kafka]] | [[snowflake]] | [[clickhouse]]
