---
schema: "1.0"
name: ruby
version: "1.0.0"
description: Ruby/Rails ActiveRecord patterns, metaprogramming risk, and production practices
domain: technology
triggers:
  keywords:
    primary: [ruby, rails, ruby on rails, gem]
    secondary: [bundler, activerecord, rspec, sidekiq, GIL]
  context_boost: [web backend, convention over configuration]
  context_penalty: [python, php, javascript]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Ruby / Rails

> Convention over configuration is a superpower until the magic hides a query storm or a monkey-patch

## Applicable Scenarios

- Writing or reviewing Ruby/Rails application code
- Diagnosing N+1 queries or unexpected ActiveRecord behavior
- Reviewing metaprogramming/monkey-patching usage for maintainability risk
- Understanding Ruby's concurrency model (GIL/GVL) for CPU-bound work
- Managing Gemfile/Bundler dependencies

## Core Knowledge

### Object Model

- Everything in Ruby is an object, including classes and integers — this enables powerful metaprogramming (`define_method`, `method_missing`, reopening classes) but the same power makes behavior harder to trace statically
- Modules provide mixins (`include`, `extend`, `prepend`) — Ruby's answer to multiple inheritance, forming part of a class's ancestor chain that affects method resolution order

### Rails Conventions

- **Convention over configuration**: Rails infers a huge amount from naming (a `Post` model maps to a `posts` table, `PostsController` to `/posts` routes) — this speeds up development but means deviating from convention requires explicit configuration that's easy to get wrong
- **ActiveRecord** is Rails's Active Record-pattern ORM — models double as both data structure and query interface, and lazy-loaded associations are the norm

### Concurrency: The GVL

- MRI (the standard Ruby implementation) has a Global VM Lock (GVL, historically called GIL) — like Python, this means threads don't achieve true parallelism for CPU-bound Ruby code, only for I/O-bound work (or C-extension code that releases the GVL)
- CPU-bound parallelism requires multiple processes (Puma/Unicorn worker processes, or explicit `fork`) rather than threads within one process

### Bundler & Gems

- `Gemfile`/`Gemfile.lock` pin dependency versions; the lockfile should always be committed and installed from (`bundle install` respecting the lock) rather than regenerated at deploy time
- Gems can monkey-patch core classes (reopen `String`, `Array`, etc.) — convenient, but two gems patching the same method can silently conflict, with whichever loaded last winning

## Best Practices

1. **Eager-load associations** (`includes`) when you know you'll access them across a collection, to avoid N+1 queries
2. **Use CPU-bound parallelism via processes, not threads** — the GVL means threads only help for I/O-bound work
3. **Avoid monkey-patching core classes** in application code; if a gem does it, understand the ancestor chain implications
4. **Guard against mass assignment** with strong parameters (`params.require(...).permit(...)`), never pass raw params to `create`/`update`
5. **Keep ActiveRecord callbacks minimal and predictable** — heavy callback chains create hidden side effects that are hard to trace and test
6. **Pin and commit `Gemfile.lock`**, deploy from the lock, don't let production resolve gem versions independently

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Accessing an association in a loop without eager loading | Use `.includes(:association)` before the loop |
| `Model.create(params)` with raw, unfiltered params | Use strong parameters (`params.require(:model).permit(:field)`) |
| Assuming threads give CPU-bound parallelism | Use multiple processes (Puma workers, Sidekiq processes) for CPU-bound work |
| Heavy business logic stuffed into ActiveRecord callbacks | Extract to service objects/POROs where logic needs to be explicit and testable |
| Monkey-patching a core class from application code | Prefer refinements (scoped monkey-patching) or explicit wrapper methods |

## Sharp Edges

### SE-1: N+1 Queries From Lazy-Loaded Associations
- **Severity**: high
- **Situation**: Fetching a collection of records, then accessing an association on each one in a view or loop, issues one additional query per record instead of a single batched query
- **Cause**: ActiveRecord associations are lazy-loaded by default — the association query fires the first time it's accessed, and a loop over N parent records triggers N additional queries
- **Symptoms**:
  - Page/response render time scales roughly linearly with the number of records displayed
  - The Rails log shows a burst of near-identical queries differing only by a foreign key value
- **Solution**: Use `.includes(:association)` (or `.eager_load`/`.preload` for specific loading strategies) to batch-load associations before iterating, and use the `bullet` gem in development to catch N+1 patterns automatically
- **Details**: → [extended/checklists.md#activerecord-checklist]

### SE-2: Mass Assignment Without Strong Parameters
- **Severity**: critical
- **Situation**: A controller action passes raw `params` directly to `Model.create`/`update` without filtering, and a request includes an unexpected field (e.g., `role=admin`) that gets persisted
- **Cause**: Without strong parameters explicitly permitting only expected fields, ActiveRecord will attempt to assign whatever attributes are present in the passed hash
- **Symptoms**:
  - A privilege escalation or data tampering bug traces back to a field the request shouldn't have been able to set
- **Solution**: Always use `params.require(:model).permit(:field1, :field2, ...)` (strong parameters) rather than passing raw `params` to a mass-assignment method

### SE-3: GVL Limiting CPU-Bound Thread Parallelism
- **Severity**: high
- **Situation**: A team spins up multiple threads to "parallelize" a CPU-bound Ruby computation (e.g., data processing, image manipulation in pure Ruby), and wall-clock time doesn't improve — sometimes it gets worse
- **Cause**: MRI's Global VM Lock serializes Ruby bytecode execution across threads in one process; threads only provide concurrency benefit for I/O-bound work (network, disk) or C-extension code that explicitly releases the GVL during its work
- **Symptoms**:
  - Adding more threads doesn't improve throughput for CPU-heavy work; CPU usage stays pinned near one core's worth
- **Solution**: Use multiple OS processes (additional Puma/Unicorn workers, Sidekiq worker processes, or explicit `Process.fork`) for genuine CPU-bound parallelism, reserving threads for I/O-bound concurrency within a process

### SE-4: Hidden Side Effects From Heavy ActiveRecord Callbacks
- **Severity**: medium
- **Situation**: Saving a record triggers a long, implicit chain of `before_save`/`after_save`/`after_commit` callbacks (sending emails, updating related records, enqueueing jobs), and a seemingly simple `record.save` call has far-reaching, hard-to-predict side effects that make testing and reasoning about the code difficult
- **Cause**: ActiveRecord callbacks are defined on the model and fire automatically on any save from anywhere in the codebase — there's no way to see, at the call site, what will actually happen without reading the model's full callback chain
- **Symptoms**:
  - A test or script that just needs to persist a record unexpectedly sends emails or triggers background jobs
  - Debugging "why did X happen" requires tracing through a chain of callbacks scattered across the model definition
- **Solution**: Keep callbacks minimal and focused on data integrity concerns intrinsic to the model itself; move business logic with broader side effects (notifications, cross-model updates) into explicit service objects called deliberately from controllers/jobs, not implicitly from model callbacks

### SE-5: Monkey-Patch Conflicts Between Gems
- **Severity**: medium
- **Situation**: Two gems both reopen the same core class and override the same method, and depending on load order, one silently overwrites the other's behavior — producing a bug that only appears with a specific combination of gems and load order
- **Cause**: Ruby allows any code to reopen and modify any class, including core classes like `String` or `Array` — there's no namespacing protection, so the last-loaded definition of a monkey-patched method wins
- **Symptoms**:
  - A gem's documented behavior for a monkey-patched method doesn't match what actually happens in the app
  - The bug appears/disappears when gem load order changes (e.g., after a `Gemfile` reorder or a Bundler resolution change)
- **Solution**: Avoid monkey-patching core classes in application code; if a gem does it and causes a conflict, prefer refinements (Ruby's scoped alternative to monkey-patching) or report/patch the conflicting gem rather than working around it with another patch

## Recommended Tools

| Category | Tools |
|------|------|
| Dependency management | Bundler |
| Testing | RSpec, Minitest |
| N+1 detection | bullet gem |
| Background jobs | Sidekiq, Resque |

## ActiveRecord & Production Health

**Full checklist**: → [extended/checklists.md#activerecord-checklist]

## Related Resources

[Ruby Docs](https://www.ruby-lang.org/en/documentation/) | [Rails Guides](https://guides.rubyonrails.org/) | [Ruby Style Guide](https://rubystyle.guide/)

## Related Domains

[[postgresql]] | [[redis]] | [[python]] | [[docker]]
