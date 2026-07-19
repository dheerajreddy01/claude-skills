# JavaScript / TypeScript — Extended Checklists

## Event Loop Checklist

Before shipping a hot-path handler:

- [ ] No synchronous CPU-heavy work (large parsing, tight loops, regex over untrusted input) on the main thread
- [ ] Every `await`-able call is actually awaited or explicitly fire-and-forget with `.catch()`
- [ ] Loops over async work use `for...of` + `await` or `Promise.all`/`allSettled`, never `forEach` with an async callback
- [ ] Regexes on user input are checked for catastrophic backtracking (or use a linear-time engine)
- [ ] Large payloads are streamed/paginated rather than buffered fully in memory

## Production Readiness Checklist

- [ ] Lockfile committed; CI uses `npm ci`/`pnpm install --frozen-lockfile`, not a mutable install
- [ ] `tsconfig.json` has `strict: true`; no unreviewed `any` in new code (lint-enforced)
- [ ] Top-level `unhandledRejection`/`uncaughtException` handlers log and alert, don't silently continue
- [ ] Environment-specific config comes from env vars/secret manager, not hardcoded
- [ ] Dependency audit (`npm audit`, Snyk, or Socket) runs in CI with a failure threshold
- [ ] Structured logging with request/trace IDs, not `console.log` in production paths
- [ ] Health check endpoint and graceful shutdown (drain in-flight requests) for servers
- [ ] Bundle size tracked in CI for frontend builds; regressions flagged
- [ ] Source maps generated but not publicly exposed in production
- [ ] Test suite covers async error paths, not just resolved-promise happy paths
