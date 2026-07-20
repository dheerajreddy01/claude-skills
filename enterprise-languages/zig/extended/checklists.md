# Zig — Extended Checklists

## Memory Safety Checklist

- [ ] Every allocation is paired with an immediate `defer` for its cleanup, covering all exit paths (including error returns)
- [ ] `GeneralPurposeAllocator`'s leak/safety checking used during development and testing, not just release builds
- [ ] Same-lifetime allocation batches use `ArenaAllocator` instead of individually-freed general-purpose allocations
- [ ] No allocator assumed globally available — every function that allocates takes an explicit `Allocator` parameter
- [ ] Error unions handled meaningfully at an appropriate layer, not `try`-propagated all the way to `main` by default
- [ ] Code tested in the actual production build mode (ReleaseSafe/ReleaseFast), not assumed safe based on Debug-mode behavior alone
- [ ] Any UB caught only by Debug-mode safety checks is treated and fixed as a real bug, not silenced

## Production Readiness Checklist

- [ ] `comptime`-heavy generic code tested incrementally, with each piece verified before composing into larger generics
- [ ] Build modes reviewed deliberately per target (Debug for development, ReleaseSafe/ReleaseFast for production) rather than defaulting to one everywhere
- [ ] Dependencies managed via `build.zig.zon` with pinned versions
- [ ] Cross-compilation targets verified explicitly if shipping to multiple platforms — Zig's cross-compilation is a strength worth testing, not assuming
- [ ] Valgrind or equivalent run periodically to catch memory issues `GeneralPurposeAllocator` might miss
