---
schema: "1.0"
name: haskell
version: "1.0.0"
description: Haskell laziness, type system, monads, and Cabal/Stack production practices
domain: technology
triggers:
  keywords:
    primary: [haskell, cabal, stack, ghc]
    secondary: [monad, typeclass, lazy evaluation, algebraic data type]
  context_boost: [functional programming, pure function, type safety]
  context_penalty: [python, javascript, imperative]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Haskell

> Purity and a powerful type system don't prevent space leaks or partial-function crashes — laziness needs its own discipline

## Applicable Scenarios

- Writing or reviewing Haskell code for correctness and performance
- Diagnosing space leaks or unexpected memory growth from laziness
- Designing types (ADTs, typeclasses) to make invalid states unrepresentable
- Choosing between Cabal and Stack for project tooling
- Working through monad transformer stacks for effectful code

## Core Knowledge

### Purity & Referential Transparency

- A pure function's result depends only on its arguments, with no side effects — this makes equational reasoning and refactoring safe, but means *all* effects (I/O, mutation, exceptions) must be explicitly threaded through types like `IO`
- The type system distinguishes pure code from effectful code at compile time (`IO a` vs `a`) — this is Haskell's core safety guarantee, not a stylistic preference

### Laziness

- Expressions are evaluated only when their value is actually needed (call-by-need), and intermediate results are memoized as **thunks** (unevaluated computation graphs)
- This enables elegant patterns (infinite lists, decoupling generation from consumption) but means an accumulation of unevaluated thunks can silently consume unbounded memory — a "space leak" that has no equivalent in strict languages
- `foldl` builds a thunk chain instead of computing eagerly; `foldl'` (strict fold) forces evaluation as it goes and is almost always what's actually wanted for numeric accumulation over large lists

### Type System

- **Algebraic Data Types (ADTs)**: sum types (`data Shape = Circle Double | Square Double`) and product types let you model a domain so that illegal states are literally unrepresentable
- **Typeclasses**: ad-hoc polymorphism (`Eq`, `Ord`, `Functor`, `Monad`) — similar in spirit to interfaces, but resolved via type inference rather than explicit implementation declarations at the call site

### Monads in Practice

- `Maybe`, `Either`, `IO`, and `[]` are all monads — the practical pattern is "a way to sequence computations that each produce a value wrapped in some context," not the category theory behind it
- Monad transformers (`StateT`, `ReaderT`, `ExceptT` stacked together) let you combine effects, but stacking several deep produces type errors and inferred types that are notoriously hard to read for newcomers

### Build Tooling

| Tool | Approach |
|------|------|
| **Cabal** | The original package/build tool; `cabal.project` + per-package `.cabal` files |
| **Stack** | Curated, reproducible package snapshots (Stackage) on top of Cabal, historically simpler to get reproducible builds with |

Both resolve the same package ecosystem (Hackage) but with different dependency resolution strategies — mixing them in one project inconsistently is a common source of confusing build failures.

## Best Practices

1. **Use strict folds (`foldl'`) for accumulation over large data**, not the lazy `foldl`
2. **Make illegal states unrepresentable in the type** rather than validating them at runtime — this is Haskell's biggest practical advantage when used well
3. **Avoid partial functions** (`head`, `tail`, `fromJust`) on data that isn't provably non-empty/non-Nothing — use pattern matching or `Maybe`-returning alternatives
4. **Keep monad transformer stacks shallow** where possible — prefer simpler effect handling (e.g., plain `IO` with explicit error values) unless the complexity genuinely earns its keep
5. **Pick one build tool (Stack or Cabal) per project** and don't mix workflows inconsistently
6. **Use `{-# LANGUAGE #-}` pragmas and `BangPatterns`/strictness annotations deliberately** where laziness is causing measurable overhead, not everywhere reflexively

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using `foldl` for large numeric accumulations | Use `foldl'` (strict) from `Data.List` |
| Using `head`/`fromJust` on data that might be empty/Nothing | Pattern match exhaustively, or use `Maybe`-safe alternatives (`listToMaybe`, `maybe`) |
| Building deep, poorly-understood monad transformer stacks early | Start with simple `IO` + explicit error types, add transformers only when the complexity is justified |
| Ignoring space usage until a program mysteriously runs out of memory | Profile with `-hc`/`-hy` heap profiling proactively on data-heavy code paths |
| Mixing Cabal and Stack commands inconsistently in the same project | Standardize on one tool for the whole team/project |

## Sharp Edges

### SE-1: Space Leaks From Unevaluated Thunks
- **Severity**: critical
- **Situation**: A program that looks like simple, correct accumulation (e.g., summing a large list) consumes far more memory than expected, or crashes with an out-of-memory error, despite the logic being straightforward
- **Cause**: Lazy evaluation defers computation and builds up a chain of unevaluated thunks instead of computing a running result — the thunk chain itself, not the "real" data, is what consumes the memory
- **Symptoms**:
  - Heap profile shows memory dominated by thunks rather than actual data structures
  - A function that's "obviously O(n) memory" in a strict language behaves as if it's O(n) *thunks*, which can be far more expensive than the data itself
- **Solution**: Use strict folds (`foldl'`) and strict data structures/fields (`BangPatterns`, strict fields in records) for accumulation-heavy code, and use GHC's heap profiling (`-hc`, `-hy`) to confirm where thunk buildup is actually happening rather than guessing
- **Details**: → [extended/checklists.md#laziness-and-performance-checklist]

### SE-2: `foldl` vs `foldl'` Performance Trap
- **Severity**: high
- **Situation**: A numeric accumulation over a large list using the default `foldl` is dramatically slower and more memory-hungry than the equivalent loop would be in almost any other language
- **Cause**: `foldl` builds up a lazy chain of unevaluated additions (`((((0+1)+2)+3)+...)`) rather than computing incrementally, and forcing that entire chain at the end can blow the stack or consume excessive memory
- **Symptoms**:
  - A simple `sum`/`foldl (+) 0` over a large list is unexpectedly slow or crashes with a stack overflow
- **Solution**: Default to `foldl'` (from `Data.List`, strict) for any accumulation over a sizable collection, and understand that `foldr` is fine (and sometimes preferable) for lazy/infinite-friendly consumption but `foldl` almost never is for eager numeric accumulation

### SE-3: Partial Functions Crashing Despite "Type Safety"
- **Severity**: high
- **Situation**: Code using `head`, `tail`, or `fromJust` crashes at runtime with a generic pattern-match failure when given an empty list or `Nothing`, despite Haskell's reputation for catching errors at compile time
- **Cause**: These functions are *partial* — they don't have a defined result for all possible inputs of their type (an empty list has no `head`) — and the type system doesn't prevent calling a partial function on an input where it's undefined; that's a runtime failure, not a type error
- **Symptoms**:
  - `Prelude.head: empty list` or `Maybe.fromJust: Nothing` runtime exceptions in production
  - The crash happens on a specific, previously-untested input shape (empty collection, missing optional value)
- **Solution**: Avoid partial functions on data that isn't provably safe; use exhaustive pattern matching, `Data.Maybe.listToMaybe`, `maybe`/`fromMaybe` with an explicit default, or enable `-Wall`/`-Wincomplete-patterns` to catch non-exhaustive pattern matches at compile time

### SE-4: Monad Transformer Stack Complexity Obscuring Errors
- **Severity**: medium
- **Situation**: A function using a deep monad transformer stack (e.g., `ReaderT Env (StateT AppState (ExceptT AppError IO))`) produces a type error that spans dozens of lines and gives little practical guidance on what's actually wrong
- **Cause**: Each layer of a transformer stack adds complexity to the inferred types, and a small mistake (wrong `lift`, mismatched effect ordering) can produce a type error far removed in presentation from its actual cause
- **Symptoms**:
  - Newcomers (and even experienced developers) spend disproportionate time deciphering a type error compared to the actual bug
  - Small refactors require touching `lift`/`liftIO` calls throughout the stack
- **Solution**: Keep transformer stacks as shallow as the problem genuinely requires, consider `mtl`-style typeclass constraints (`MonadState`, `MonadError`) instead of concrete stacks to decouple code from the exact stack shape, and introduce stack complexity incrementally with tests at each step rather than designing the full stack upfront

### SE-5: Cabal/Stack Dependency Resolution Conflicts
- **Severity**: medium
- **Situation**: A project's build breaks or behaves inconsistently across team members' machines because some use `cabal build` and others use `stack build`, each resolving package versions differently
- **Cause**: Cabal (especially with a `cabal.project.freeze`) and Stack (with a curated Stackage snapshot) use different dependency resolution strategies that can select different package versions for the same declared dependencies, and mixing workflows means "it builds for me" isn't a reliable signal
- **Symptoms**:
  - A build that works via `stack build` fails via `cabal build` (or vice versa) with version-mismatch errors
  - CI uses a different tool than what most developers use locally, causing "works locally, fails in CI" reports
- **Solution**: Standardize on one build tool for a given project and document it clearly, commit the relevant lock/freeze file (`stack.yaml.lock` or `cabal.project.freeze`), and ensure CI uses the exact same tool and lock file developers use locally

## Recommended Tools

| Category | Tools |
|------|------|
| Build | Stack, Cabal |
| Linting | HLint |
| Profiling | GHC heap profiling (`-hc`, `-hy`), `+RTS -s` |
| Testing | Hspec, QuickCheck (property-based testing) |

## Laziness & Performance

**Full checklist**: → [extended/checklists.md#laziness-and-performance-checklist]

## Related Resources

[Learn You a Haskell](http://learnyouahaskell.com/) | [Haskell Documentation](https://www.haskell.org/documentation/) | [Real World Haskell](http://book.realworldhaskell.org/)

## Related Domains

[[scala]] | [[rust]] | [[go]]
