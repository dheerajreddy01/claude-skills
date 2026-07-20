# Lua — Extended Checklists

## Scoping and Safety Checklist

- [ ] `luacheck` (or equivalent) run in CI to catch missing `local` declarations
- [ ] No unintended global variable creation across modules/scripts
- [ ] Table copy semantics reviewed anywhere a "copy" is assumed — explicit copy function used where independence is required
- [ ] Array-style table access consistently uses 1-based indexing; `ipairs`/`#t` preferred over manual index arithmetic
- [ ] Metatable/`__index` inheritance chains kept shallow and documented
- [ ] Coroutines yield at appropriate points; none block the cooperative scheduler indefinitely
- [ ] Embedding sandbox (restricted `_ENV`/global table) explicitly designed, not left at the interpreter's full default environment
- [ ] Host-exposed APIs to embedded scripts reviewed and minimized to what's actually needed

## Production Readiness Checklist

- [ ] Dependencies managed via LuaRocks with pinned versions where reproducibility matters
- [ ] Unit tests (Busted or equivalent) cover core logic, not just manual/interactive testing
- [ ] Performance-critical paths profiled and, if needed, moved to LuaJIT or optimized table access patterns
- [ ] Embedding-specific API differences (Roblox vs. stock Lua vs. OpenResty) documented if code is shared across embeddings
- [ ] Error handling uses `pcall`/`xpcall` at appropriate boundaries rather than letting uncaught errors propagate into the host application
