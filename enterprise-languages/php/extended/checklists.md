# PHP — Extended Checklists

## Security Checklist

- [ ] `===`/`!==` used by default; `==` reserved for deliberate, documented coercion cases
- [ ] `hash_equals()` used for any security-sensitive comparison (tokens, hashes), never `==`/`===`
- [ ] Every Eloquent model defines `$fillable` or `$guarded` — no unguarded mass assignment
- [ ] All SQL queries parameterized/use the query builder — no raw string interpolation of user input
- [ ] No `@` error suppression operator used anywhere in the codebase (lint-enforced where possible)
- [ ] `declare(strict_types=1)` set in new files
- [ ] User input validated via an explicit validation layer/DTO before reaching models or shell commands
- [ ] Shell command execution (`exec`, `shell_exec`) avoided or strictly escaped/allowlisted

## Performance & Deployment Checklist

- [ ] Relationships accessed in loops are eager-loaded (`with()`), not lazy-loaded per iteration
- [ ] Query logging enabled in development to catch N+1 patterns before production
- [ ] OPcache reset/worker restart included as an explicit deployment step
- [ ] `composer.lock` committed and installed from (`composer install --no-dev` in production), not regenerated at deploy time
- [ ] PHPStan/Psalm run in CI at a meaningful strictness level
- [ ] Session storage externalized (Redis/database) for multi-server deployments, not left on local disk
