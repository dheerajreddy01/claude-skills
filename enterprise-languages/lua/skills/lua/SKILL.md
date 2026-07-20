---
schema: "1.0"
name: lua
version: "1.0.0"
description: Lua tables, metatables, coroutines, and embedded-scripting sandbox production practices
domain: technology
triggers:
  keywords:
    primary: [lua, luajit, metatable, coroutine]
    secondary: [openresty, love2d, roblox, neovim config]
  context_boost: [embedded scripting, game engine, config scripting]
  context_penalty: [python, javascript, ruby]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Lua

> One data structure (the table) does everything — which makes small mistakes with it propagate everywhere

## Applicable Scenarios

- Writing or reviewing Lua scripts for games (Roblox, LÖVE), config (Neovim), or embedded contexts (nginx/OpenResty)
- Designing OOP-style patterns using metatables
- Diagnosing global-variable leakage or table reference bugs
- Working with coroutines for cooperative concurrency
- Reviewing sandboxing when Lua scripts run inside a host application

## Core Knowledge

### Tables: The One True Data Structure

- A Lua **table** is simultaneously an array, a dictionary/map, and (via metatables) the basis for objects and modules — there's no separate array type or struct type
- Tables are **reference types**: assigning a table to a new variable copies the reference, not the contents; `table1 == table2` compares identity, not structural equality
- Lua arrays are **1-indexed** by convention (`t[1]` is the first element) — this is a language-wide convention, not an enforced rule, but nearly all standard library functions assume it

### Metatables

- A metatable attached to a table defines how it behaves for operations like indexing (`__index`), arithmetic (`__add`), and calling (`__call`) — this is how Lua implements OOP-style inheritance and operator overloading without built-in class syntax
- `__index` as a function or table enables "inheritance chains" (falling back to a parent table's fields) — useful but can become hard to trace through several levels of delegation

### Scoping & Globals

- Variables are **global by default** unless declared with `local` — omitting `local` silently creates or modifies a global, which is easy to do accidentally and can corrupt state used elsewhere in a large codebase or embedding host
- `local` variables are lexically scoped and the idiomatic default for anything not intentionally shared

### Coroutines

- Lua coroutines are cooperative (not preemptive) — a coroutine runs until it explicitly `yield`s, giving the caller full control over when execution resumes, unlike OS threads or Go-style goroutines
- Used for generators, cooperative task scheduling, and async-style code in embedding contexts (e.g., game engine frame loops)

### Embedding Contexts

- Lua is frequently embedded inside a host application (game engines, nginx via OpenResty, Neovim, Redis scripting) — the host controls what globals/APIs are exposed to the script, and a poorly configured sandbox can let a script access more than intended
- Different embeddings have different standard library availability and conventions (e.g., Roblox's modified Lua, LuaJIT's FFI) — code isn't automatically portable across embeddings

## Best Practices

1. **Always use `local`** unless a global is genuinely intended — make this a lint-enforced habit, not a manual discipline
2. **Remember Lua is 1-indexed** when porting logic/intuition from 0-indexed languages
3. **Keep metatable/inheritance chains shallow** — deep `__index` delegation chains become hard to debug
4. **Understand table reference semantics** — copy a table explicitly (shallow or deep, as needed) rather than assuming assignment copies it
5. **Design coroutine yield points deliberately** — a coroutine that never yields blocks the whole cooperative scheduler just like a blocking call would
6. **Review what a host embedding exposes to scripts** — don't assume a script is sandboxed just because it's "just Lua"

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Forgetting `local`, creating an accidental global | Always declare with `local`; use a linter to catch missing `local` |
| Assuming 0-indexed arrays | Remember Lua tables used as arrays are 1-indexed by convention |
| Comparing tables with `==` expecting structural equality | Implement explicit deep-equality comparison if structural equality is needed |
| Assuming `t2 = t1` copies the table | Understand this only copies the reference; explicitly copy fields if an independent table is needed |
| Writing a coroutine that never yields, expecting cooperative scheduling to still work | Ensure every coroutine yields at appropriate points so the scheduler can resume other work |

## Sharp Edges

### SE-1: Accidental Global Variable Pollution
- **Severity**: high
- **Situation**: A typo or an omitted `local` keyword creates a global variable that silently shadows or corrupts state used elsewhere in the codebase or, worse, in the host application embedding the Lua script
- **Cause**: Lua's default scoping rule makes any un-declared assignment a global — there's no compiler error, no warning by default, just silent global creation
- **Symptoms**:
  - A variable used in one module unexpectedly has a value set by unrelated code elsewhere
  - Bugs that appear only when two modules or scripts happen to use the same variable name
- **Solution**: Always declare variables with `local`, use a linter (`luacheck`) configured to flag global assignments, and in embedding contexts, consider restricting or auditing the global table (`_G`) available to scripts
- **Details**: → [extended/checklists.md#scoping-and-safety-checklist]

### SE-2: 1-Indexed Off-by-One Bugs
- **Severity**: medium
- **Situation**: Code ported from (or written by someone experienced in) a 0-indexed language accesses `t[0]` expecting the first element, or does index arithmetic assuming 0-based offsets, producing subtly wrong results or `nil` errors
- **Cause**: Lua array-style tables conventionally start at index 1, not 0 — this is baked into the standard library (`#t`, `table.insert`, `ipairs`) and any manual indexing must match
- **Symptoms**:
  - `t[0]` silently returns `nil` instead of erroring, so the bug doesn't immediately surface
  - Loop bounds or offset calculations are consistently off by one compared to equivalent code in a 0-indexed language
- **Solution**: Internalize Lua's 1-based convention, use `ipairs`/`#t` rather than manual index arithmetic where possible, and be extra careful when porting algorithms directly from 0-indexed language examples

### SE-3: Table Reference vs. Value Confusion
- **Severity**: medium
- **Situation**: Code assumes assigning a table to a new variable (or passing it to a function) creates an independent copy, and mutations made through one reference unexpectedly show up when accessed through the other
- **Cause**: Tables are reference types in Lua — `local t2 = t1` makes `t2` point to the same underlying table as `t1`, not a copy
- **Symptoms**:
  - Modifying what looks like a "local copy" of a table also changes the original, causing hard-to-trace state corruption
- **Solution**: Explicitly write (or use a library providing) a shallow/deep copy function when an independent table is actually needed, and be deliberate about when a function is meant to mutate its table argument versus operate on a copy

### SE-4: Deep Metatable Inheritance Chains Becoming Unmaintainable
- **Severity**: medium
- **Situation**: An OOP-style codebase built with `__index`-based inheritance chains grows several levels deep, and tracing where a given method or field actually comes from requires manually walking the chain — refactors become risky because it's unclear what depends on what
- **Cause**: `__index` delegation is a flexible but implicit mechanism — Lua doesn't provide any built-in tooling to visualize or validate an inheritance hierarchy the way a class-based language's tooling might
- **Symptoms**:
  - Debugging "where does this method come from" requires manually inspecting multiple metatables
  - Adding a method at the wrong level of the hierarchy silently shadows or fails to override a parent's implementation
- **Solution**: Keep inheritance chains shallow (prefer composition over deep multi-level `__index` chains), document the hierarchy explicitly, and consider a lightweight class library convention consistently across the codebase rather than ad hoc metatable patterns per module

### SE-5: Embedding Sandbox Escapes or Unintended Global Access
- **Severity**: high
- **Situation**: A Lua script running inside a host application (game engine, nginx/OpenResty, a plugin system) accesses host-environment globals or APIs that weren't meant to be exposed to script authors, potentially allowing unintended file access, network calls, or interference with the host process
- **Cause**: Embedding a Lua interpreter doesn't automatically sandbox it — by default, a Lua environment has access to its full standard library (including `io`, `os` in some builds) unless the host application explicitly restricts the global environment made available to scripts
- **Symptoms**:
  - A script is able to call functions (file I/O, process control) that the embedding was never designed to allow
  - Security review of a plugin/scripting system finds the sandbox boundary is weaker than assumed
- **Solution**: Explicitly construct a restricted environment (a limited `_ENV`/global table) for embedded scripts rather than exposing the full standard library, and treat the embedding's sandbox boundary as something to be actively designed and audited, not assumed by default

## Recommended Tools

| Category | Tools |
|------|------|
| Linting | luacheck |
| Testing | Busted |
| Package management | LuaRocks |
| Runtime variants | LuaJIT (performance), embedding-specific runtimes (Roblox, OpenResty) |

## Scoping & Safety

**Full checklist**: → [extended/checklists.md#scoping-and-safety-checklist]

## Related Resources

[Programming in Lua](https://www.lua.org/pil/) | [Lua 5.4 Reference Manual](https://www.lua.org/manual/5.4/) | [Lua Users Wiki](http://lua-users.org/wiki/)

## Related Domains

[[nginx]] | [[docker]] | [[go]]
