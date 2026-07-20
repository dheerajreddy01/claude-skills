# Haskell — Extended Checklists

## Laziness and Performance Checklist

- [ ] Accumulation over large collections uses `foldl'` (strict), not `foldl`
- [ ] Strict fields (`!` / `BangPatterns`) used deliberately in records/data structures known to accumulate large values
- [ ] Heap profiling (`-hc`/`-hy`) run on data-heavy code paths before assuming performance is acceptable
- [ ] No partial functions (`head`, `tail`, `fromJust`) used on data that isn't provably non-empty/non-Nothing
- [ ] `-Wall -Wincomplete-patterns` enabled to catch non-exhaustive pattern matches at compile time
- [ ] `+RTS -s` (runtime stats) checked for unexpectedly high memory residency during development
- [ ] Infinite/lazy data structures consumed with a clear termination strategy, not accidentally forced in full

## Build & Tooling Checklist

- [ ] Team standardized on one build tool (Stack or Cabal) for a given project
- [ ] Lock/freeze file (`stack.yaml.lock` or `cabal.project.freeze`) committed for reproducible builds
- [ ] CI uses the exact same build tool and lock file as local development
- [ ] HLint run in CI to catch idiomatic issues
- [ ] Property-based tests (QuickCheck) used for pure functions with well-defined invariants, not just example-based tests
- [ ] Monad transformer stacks kept as shallow as the problem requires; `mtl`-style constraints considered over concrete stacks where flexibility matters
