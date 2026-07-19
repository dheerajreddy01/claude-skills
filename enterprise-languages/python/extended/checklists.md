# Python — Extended Checklists

## Concurrency Checklist

Before choosing a concurrency model:

- [ ] Is the bottleneck I/O-bound (network/disk) or CPU-bound (computation)?
  - I/O-bound → asyncio (preferred) or threading
  - CPU-bound → multiprocessing or a C-extension (NumPy, etc.)
- [ ] If using asyncio, are ALL calls in the hot path async-native (no blocking `requests`, `time.sleep`, sync DB drivers)?
- [ ] If using threading for I/O, are shared mutable objects protected by a `Lock` or avoided entirely?
- [ ] If using multiprocessing, is the per-task payload large enough to amortize process/IPC overhead?
- [ ] Have you load-tested under realistic concurrency, not just verified correctness at concurrency=1?

## Production Readiness Checklist

- [ ] Dependencies pinned via a committed lockfile; CI installs from the lock, not floating ranges
- [ ] `mypy`/`pyright` runs in CI on at least the public API surface
- [ ] No bare `except:` in the codebase (lint rule enforced, e.g. ruff `E722`)
- [ ] Structured logging in place (not bare `print`), with request/trace IDs where applicable
- [ ] Health check endpoint and graceful shutdown handling for long-running services
- [ ] Secrets loaded from environment/secret manager, never committed or hardcoded
- [ ] `python -O` / debug flags disabled in production; `DEBUG=False` equivalent verified
- [ ] Resource limits set (worker count, timeouts, max request size) — no unbounded queues
- [ ] Test suite covers error paths, not just the happy path
- [ ] Dependency licenses reviewed for compliance if shipping externally
