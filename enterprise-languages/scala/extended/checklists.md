# Scala — Extended Checklists

## Concurrency and Correctness Checklist

- [ ] Blocking calls run on a dedicated `ExecutionContext`, never the default global fork-join pool
- [ ] Implicit scope kept narrow; ambiguous or surprising implicit resolution checked with `-Xlog-implicits` or IDE tooling when debugging
- [ ] Recursive functions expected to handle large/deep input are annotated `@tailrec` and verified by the compiler
- [ ] Akka actor mailboxes are bounded (with an explicit overflow strategy) for actors that could receive messages faster than they process them
- [ ] Mailbox size and actor processing latency monitored as standard operational metrics
- [ ] No pattern matching directly discriminates on erased generic type parameters
- [ ] "Unchecked" compiler warnings reviewed and addressed, not suppressed blindly

## Code Quality Checklist

- [ ] Domain state modeled with sealed traits + case classes, with exhaustive pattern matching (no unnecessary catch-all masking missing cases)
- [ ] `val` preferred over `var`; mutability scoped tightly when genuinely needed
- [ ] Scalafix/Scalafmt run in CI for consistent style and to catch common anti-patterns
- [ ] Public API signatures have explicit type annotations, not relying solely on inference
- [ ] Property-based tests (ScalaCheck) used for functions with clear invariants, not just example-based tests
