---
schema: "1.0"
name: javascript
version: "1.0.0"
description: JavaScript/TypeScript, Node.js, async patterns, and frontend/backend production practices
domain: technology
triggers:
  keywords:
    primary: [javascript, typescript, node, nodejs, react, npm]
    secondary: [async, promise, event loop, webpack, vite, jest, tsconfig]
  context_boost: [frontend, backend, package.json, browser, server]
  context_penalty: [python, java, rust]
  priority: high
dependencies:
  software-skills: [git-workflows, testing-strategies]
author: claude-domain-skills
---

# JavaScript / TypeScript

> The event loop, the type system, and the dependency graph — used correctly

## Applicable Scenarios

- Writing or reviewing JavaScript/TypeScript for frontend, Node.js backend, or full-stack apps
- Diagnosing async bugs (race conditions, unhandled rejections, event loop blocking)
- Choosing package managers, bundlers, or module systems
- Setting up TypeScript strictness, testing, and build tooling
- Reviewing dependency footprint and supply-chain risk

## Core Knowledge

### Runtime & Module Systems

| Concept | Detail |
|------|------|
| **Event loop** | Single-threaded; macrotasks (setTimeout, I/O) vs microtasks (Promises) — microtasks always drain before the next macrotask |
| **CommonJS (`require`)** | Synchronous, Node's original module system |
| **ESM (`import`/`export`)** | Standard, async-capable, tree-shakeable — default for new projects |
| **Node.js** | Server-side runtime; non-blocking I/O via libuv thread pool |

### Package Managers

| Tool | Notes |
|------|------|
| **npm** | Default, ships with Node, `package-lock.json` |
| **pnpm** | Content-addressable store, fastest installs, strict by default (no phantom deps) |
| **yarn** | Workspaces support, `yarn.lock` |

Always commit the lockfile. Use `npm ci` (not `npm install`) in CI for reproducible, lockfile-exact installs.

### TypeScript

- Enable `strict: true` in `tsconfig.json` for new projects — catches null/undefined bugs at compile time
- `any` defeats the type system; prefer `unknown` and narrow explicitly
- Structural typing means two unrelated types are compatible if their shapes match — don't assume nominal typing semantics
- Types are erased at runtime — never rely on `typeof`/`instanceof` behaving like compile-time type checks for structural types

### Async Patterns

| Pattern | Use |
|------|------|
| `Promise.all()` | Run independent async ops concurrently, fail fast on first rejection |
| `Promise.allSettled()` | Run independent async ops concurrently, collect all results/errors |
| `async`/`await` | Sequential-looking async code; still needs `try/catch` for errors |
| Event emitters / streams | Backpressure-aware, high-throughput I/O |

## Best Practices

1. **Always handle promise rejections** — an unhandled rejection can crash a Node process (since Node 15+)
2. **Use `===`, never `==`** — avoid type coercion surprises
3. **Lock dependencies and audit them** — `npm audit`, Dependabot/Renovate for updates
4. **Prefer `const`, then `let`** — never `var` (function-scoped, hoisting footguns)
5. **Type your API boundaries first** — request/response shapes, then work inward
6. **Keep the bundle lean** — check bundle size impact before adding a dependency for a trivial utility

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `==` for comparisons | `===` / `!==` |
| Un-awaited promises in a loop (`forEach` with async callback) | Use `for...of` with `await`, or `Promise.all(items.map(...))` |
| Mutating state directly in React | Use immutable updates / state setters |
| `any` everywhere in TypeScript | `unknown` + narrowing, or precise types |
| Blocking the event loop with sync heavy computation | Offload to a worker thread or a queue/worker service |

## Sharp Edges

### SE-1: Unhandled Promise Rejections Crash the Process
- **Severity**: critical
- **Situation**: A background `async` call that isn't awaited throws, and the entire Node process crashes or silently fails
- **Cause**: Node terminates the process on an unhandled promise rejection by default (as of Node 15+); browsers only log a warning
- **Symptoms**:
  - Process exits with no clear stack trace pointing at the actual caller
  - Works fine until a rare error path is hit in production
- **Solution**: Await every promise, attach `.catch()` to fire-and-forget calls, and add a top-level `process.on('unhandledRejection', ...)` as a safety net (log + alert, don't just swallow)

### SE-2: `this` Binding Loses Context
- **Severity**: high
- **Situation**: A method passed as a callback (`element.addEventListener('click', obj.method)`) loses its `this` binding and throws or reads `undefined`
- **Cause**: Regular functions get `this` from how they're *called*, not where they're defined; only arrow functions lexically bind `this`
- **Symptoms**:
  - `Cannot read property 'x' of undefined` inside a callback that worked fine called directly
- **Solution**: Use arrow functions for callbacks that need the enclosing `this`, or `.bind(this)` explicitly, or class fields (`method = () => {}`)

### SE-3: Blocking the Event Loop
- **Severity**: critical
- **Situation**: A synchronous CPU-heavy operation (large JSON.parse, regex backtracking, tight loop) freezes the whole server — every concurrent request stalls
- **Cause**: Node is single-threaded for JS execution; nothing else runs until the call stack clears
- **Symptoms**:
  - Latency spikes correlated with specific request types
  - Health checks time out under load despite low overall CPU
- **Solution**: Move CPU-heavy work to a `worker_threads` pool or a separate service/queue; avoid catastrophic-backtracking regexes; paginate/stream large payloads
- **Details**: → [extended/checklists.md#event-loop-checklist]

### SE-4: Dependency Supply-Chain Risk
- **Severity**: high
- **Situation**: A transitive dependency is compromised (malicious postinstall script, typosquat package) and runs arbitrary code during `npm install` or at runtime
- **Cause**: Deep, unaudited dependency trees with auto-run install scripts and loosely pinned versions
- **Symptoms**:
  - Unexpected network calls or file writes during install
  - Package published under a near-identical name to a popular one
- **Solution**: Pin exact versions, run `npm audit`/Socket/Snyk in CI, disable install scripts where possible (`--ignore-scripts`), review new dependencies before adding

### SE-5: TypeScript `any` Hides Real Bugs
- **Severity**: medium
- **Situation**: A function typed with `any` (or an untyped external API response) propagates through the codebase, and a runtime `TypeError` surfaces far from the actual bad input
- **Cause**: `any` disables type checking for everything it touches, so the compiler can't catch a mismatched shape
- **Symptoms**:
  - Errors happen deep in unrelated code, not at the boundary where bad data entered
  - Refactors don't get flagged by the compiler even though they break call sites
- **Solution**: Type external boundaries with `unknown` + a validation library (Zod, io-ts) that narrows at runtime; forbid `any` via lint rule (`@typescript-eslint/no-explicit-any`)

## Recommended Tools

| Category | Tools |
|------|------|
| Bundlers | Vite, esbuild, webpack |
| Testing | Vitest, Jest, Playwright |
| Linting/formatting | ESLint, Prettier, Biome |
| Runtime validation | Zod, io-ts |
| Monorepo/workspaces | pnpm workspaces, Turborepo, Nx |

## Production Readiness

**Full deployment checklist**: → [extended/checklists.md#production-readiness-checklist]

## Related Resources

[MDN Web Docs](https://developer.mozilla.org/) | [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | [Node.js Docs](https://nodejs.org/docs/latest/api/)

## Related Domains

[[python]] | [[go]] | [[aws]]
