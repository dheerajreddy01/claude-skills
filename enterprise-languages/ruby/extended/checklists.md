# Ruby / Rails — Extended Checklists

## ActiveRecord Checklist

- [ ] Associations accessed in loops/views are eager-loaded (`.includes`), not lazy-loaded per iteration
- [ ] `bullet` gem (or equivalent) enabled in development/test to catch N+1 queries automatically
- [ ] Strong parameters used on every mass-assignment call — no raw `params` passed to `create`/`update`
- [ ] Model callbacks kept minimal; business logic with broad side effects lives in service objects, not callback chains
- [ ] Database indexes reviewed for foreign keys and frequently-queried columns
- [ ] `explain`/query logging reviewed in development for any suspiciously slow request

## Production & Dependency Checklist

- [ ] `Gemfile.lock` committed; production installs from the lock (`bundle install --deployment` or equivalent), not a fresh resolution
- [ ] CPU-bound work parallelized via processes (Puma workers, Sidekiq workers), not threads
- [ ] No monkey-patching of core classes in application code; refinements used if scoped patching is genuinely needed
- [ ] Background job idempotency reviewed (Sidekiq/Resque can redeliver on failure/retry)
- [ ] Structured logging in place, with request/trace IDs
- [ ] Secrets loaded from environment/credentials store (Rails encrypted credentials or equivalent), not hardcoded
- [ ] Test suite covers callback-triggered side effects explicitly, not just the primary save path
