---
schema: "1.0"
name: python
version: "1.0.0"
description: Python language internals, packaging, typing, async, and production practices
domain: technology
triggers:
  keywords:
    primary: [python, django, flask, fastapi, pandas, pytest]
    secondary: [pip, venv, poetry, uv, asyncio, type hints, GIL, mypy]
  context_boost: [script, backend, data pipeline, API, virtualenv]
  context_penalty: [javascript, java, rust]
  priority: high
dependencies:
  software-skills: [git-workflows, testing-strategies]
author: claude-domain-skills
---

# Python

> Idiomatic, correct, and production-safe Python

## Applicable Scenarios

- Writing or reviewing Python application, script, or library code
- Choosing between packaging/dependency tools (pip, Poetry, uv, pipenv)
- Designing concurrency (asyncio vs threading vs multiprocessing)
- Debugging performance, memory, or dependency issues
- Setting up testing, typing, and linting for a Python codebase

## Core Knowledge

### Environment & Packaging

| Tool | Role | When to Use |
|------|------|------|
| **venv** | Built-in virtual env | Minimal projects, no extra deps |
| **pip + requirements.txt** | Basic dependency install | Legacy/simple projects |
| **Poetry** | Dependency + build + publish | Libraries, reproducible builds |
| **uv** | Fast installer/resolver (Rust-based) | New projects, CI speed matters |
| **pyenv** | Python version management | Multiple Python versions on one machine |

Always pin dependencies via a lockfile (`poetry.lock`, `uv.lock`, or `pip-compile` output) — never ship unpinned `requirements.txt` to production.

### Typing

- Use type hints on public function signatures at minimum; run `mypy` or `pyright` in CI
- `Optional[X]` / `X | None` (3.10+) must be handled explicitly — don't silently assume presence
- `TypedDict`, `Protocol`, and `dataclass` reduce runtime errors caught only at type-check time
- Gradual typing is fine: type the boundaries (API handlers, public functions) before internals

### Concurrency Model

| Approach | Best For | Limitation |
|------|------|------|
| **asyncio** | I/O-bound (network, disk, DB) | Single-threaded; one blocking call stalls the event loop |
| **threading** | I/O-bound, need blocking-lib compat | GIL prevents true CPU parallelism |
| **multiprocessing** | CPU-bound work | Process startup + IPC overhead, no shared memory by default |
| **concurrent.futures** | Simple parallel task pools | Thin wrapper over threading/multiprocessing |

The **GIL** (Global Interpreter Lock, removed only in the experimental free-threaded 3.13 build) means threads never give you CPU parallelism for pure-Python code — only I/O-bound concurrency or C-extension work (NumPy, etc.) benefits.

### Web Frameworks

| Framework | Style | Best For |
|------|------|------|
| **FastAPI** | Async, type-driven | APIs needing OpenAPI docs, high I/O concurrency |
| **Django** | Batteries-included, sync-first (async-capable) | Full apps with admin, ORM, auth built in |
| **Flask** | Minimal, unopinionated | Small services, full control over stack |

## Best Practices

1. **Isolate environments** — never install project dependencies into system Python
2. **Lock dependencies** — commit the lockfile; upgrade deliberately, not via floating versions
3. **Type the boundaries** — function signatures and API schemas, even if internals stay untyped
4. **Fail loudly** — avoid bare `except:`; catch specific exceptions and re-raise unknowns
5. **Test with pytest fixtures**, not shared mutable module state, to keep tests isolated
6. **Profile before optimizing** — use `cProfile`/`py-spy`, don't guess at bottlenecks

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `def f(items=[])` mutable default arg | Use `None` default, create the list inside the function |
| Catching bare `except:` | Catch specific exception types |
| Mixing `asyncio.run()` calls inside already-running loops | Use `await` within the existing loop, one entrypoint only |
| Comparing floats with `==` | Use `math.isclose()` or `Decimal` for money |
| Circular imports from god-module `utils.py` | Split by responsibility, import lazily only if unavoidable |

## Sharp Edges

### SE-1: The Mutable Default Argument Trap
- **Severity**: high
- **Situation**: A function with `def f(cache={})` appears to reset `cache` on every call but actually shares one dict across all calls
- **Cause**: Default argument values are evaluated once, at function definition time, not per-call
- **Symptoms**:
  - Data from one call mysteriously appears in an unrelated call
  - Bug only shows up after the function has been called many times
- **Solution**: Default to `None`, then `cache = cache if cache is not None else {}` inside the function body

### SE-2: GIL-Blind Threading for CPU Work
- **Severity**: critical
- **Situation**: Spinning up `threading.Thread` workers to "parallelize" a CPU-bound loop, but wall-clock time doesn't improve
- **Cause**: The GIL serializes bytecode execution across threads in CPython; threads only help when a call releases the GIL (I/O, or C extensions like NumPy)
- **Symptoms**:
  - More threads added, throughput stays flat or gets worse (context-switch overhead)
  - CPU usage stuck near 100% on a single core despite many threads
- **Solution**: Use `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` for CPU-bound work; reserve threads/asyncio for I/O-bound work
- **Details**: → [extended/checklists.md#concurrency-checklist]

### SE-3: Unpinned Dependencies Breaking Production
- **Severity**: critical
- **Situation**: A `pip install -r requirements.txt` deploy pulls a new transitive dependency version that breaks the app in production
- **Cause**: `requirements.txt` without a lockfile allows floating version ranges to resolve differently over time
- **Symptoms**:
  - "Works on my machine" but fails in CI/prod
  - Deploy that changed no application code suddenly breaks
- **Solution**: Generate and commit a full lockfile (`poetry.lock`, `uv.lock`, or `pip-compile`), install from the lock in CI/prod, upgrade deps via an explicit, reviewed PR

### SE-4: Blocking Calls Inside asyncio Event Loops
- **Severity**: high
- **Situation**: An `async def` handler calls a synchronous blocking function (e.g., `requests.get`, `time.sleep`, sync DB driver) and the entire server stalls for concurrent requests
- **Cause**: asyncio is single-threaded cooperative multitasking — a blocking call never yields control back to the event loop
- **Symptoms**:
  - Latency spikes under concurrent load despite low CPU usage
  - One slow request delays unrelated requests
- **Solution**: Use async-native libraries (`httpx`, async DB drivers) or offload blocking calls via `asyncio.to_thread()` / `run_in_executor()`

### SE-5: Silent Data Corruption From Bare Excepts
- **Severity**: medium
- **Situation**: A bare `except:` (or `except Exception: pass`) swallows a `KeyError` or `TypeError` that indicated a real bug, and the pipeline "succeeds" with wrong output
- **Cause**: Broad exception handling treats programming errors the same as expected failure modes
- **Symptoms**:
  - Downstream data is wrong/missing but no error was ever logged
  - Debugging requires removing the try/except to reproduce
- **Solution**: Catch narrow, specific exception types; log and re-raise unexpected ones; reserve blanket catches for top-level process boundaries with explicit logging

## Recommended Tools

| Category | Tools |
|------|------|
| Packaging | Poetry, uv, pip-tools |
| Type checking | mypy, pyright |
| Linting/formatting | ruff, black |
| Testing | pytest, hypothesis, coverage.py |
| Profiling | py-spy, cProfile, memray |

## Production Readiness

**Full deployment checklist**: → [extended/checklists.md#production-readiness-checklist]

## Related Resources

[Python Docs](https://docs.python.org/3/) | [Real Python](https://realpython.com/) | [PEP Index](https://peps.python.org/)

## Related Domains

[[javascript]] | [[go]] | [[aws]]
