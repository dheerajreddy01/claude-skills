---
schema: "1.0"
name: php
version: "1.0.0"
description: PHP request lifecycle, type juggling, Composer/PSR conventions, and Laravel/Symfony production practices
domain: technology
triggers:
  keywords:
    primary: [php, laravel, symfony, composer]
    secondary: [wordpress, PSR, eloquent, opcache, doctrine]
  context_boost: [web backend, server-side rendering, content management]
  context_penalty: [python, javascript, ruby]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# PHP

> A shared-nothing request model that's forgiving right up until loose comparison bites you in production

## Applicable Scenarios

- Writing or reviewing PHP application code (Laravel, Symfony, or plain PHP)
- Diagnosing type-juggling bugs from loose comparison
- Structuring Composer dependencies and PSR-compliant autoloading
- Reviewing Eloquent/Doctrine ORM usage for N+1 queries
- Understanding the shared-nothing per-request execution model's implications

## Core Knowledge

### Request Lifecycle

- Classic PHP execution is **shared-nothing**: each request starts a fresh process/worker with no in-memory state carried over from the previous request — this is why PHP scales horizontally so easily, but it also means any "global" state (caches, connections) must be explicitly externalized (Redis, a database, opcode cache) rather than kept in a long-lived process
- **OPcache** caches compiled bytecode across requests to avoid re-parsing PHP source on every request — a major performance factor, but it means source file changes may not take effect until the cache is cleared/invalidated depending on configuration

### Type System & Comparison

- PHP is dynamically, weakly typed by default — `==` performs type coercion ("type juggling") before comparing, which has produced well-known surprising results across PHP versions (numeric-looking strings compared to numbers, `0 == "abc"`-style comparisons in older PHP)
- PHP 8+ significantly tightened many of these comparison rules, but the discipline of using `===` (strict comparison, no coercion) remains the safe default regardless of version
- Type declarations (parameter types, return types, `declare(strict_types=1)`) are opt-in per file — without `strict_types`, scalar type declarations still allow coercion at the call boundary

### Composer & Autoloading

- Composer manages dependencies via `composer.json`/`composer.lock`, and PSR-4 autoloading maps namespaces to directory structures so classes load without manual `require` statements
- The lockfile pins exact resolved versions — deploying without it (or with a stale one) risks pulling different dependency versions than what was tested

### ORM Patterns (Eloquent/Doctrine)

- Both are Active Record (Eloquent) or Data Mapper (Doctrine) style ORMs — lazy-loaded relationships accessed in a loop produce the same N+1 query problem seen in any ORM, and it's one of the most common PHP performance issues in production
- Mass assignment (`Model::create($request->all())`) is convenient but dangerous — without an explicit fillable/guarded allowlist, a request can set fields it shouldn't be able to

## Best Practices

1. **Use `===`/`!==` by default**, reserve `==`/`!=` only for cases where coercion is genuinely intended and understood
2. **Enable `declare(strict_types=1)`** in new files to make type declarations actually enforce, not just coerce
3. **Guard against mass assignment** explicitly — define `$fillable`/`$guarded` in Eloquent models, never blindly pass request input to `create()`/`fill()`
4. **Eager-load relationships** (`with()`) when you know you'll access them in a loop, to avoid N+1 queries
5. **Never suppress errors with `@`** — it hides real failures instead of handling them; use proper exception handling or explicit checks
6. **Escape/parameterize all database and shell input** — never interpolate user input directly into a SQL query or shell command

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Using `==` for comparisons involving mixed types | Use `===` to avoid type-juggling surprises |
| `Model::create($request->all())` with no fillable guard | Define `$fillable`/`$guarded` explicitly, or use a validated DTO |
| Accessing a lazy relationship inside a loop | Eager-load with `with()` before the loop |
| Suppressing errors with `@$value` | Handle the specific error condition explicitly instead of silencing it |
| String-concatenating user input into a SQL query | Use parameterized queries/prepared statements (or the ORM's query builder) |

## Sharp Edges

### SE-1: Loose Comparison Type Juggling
- **Severity**: high
- **Situation**: A comparison using `==` between a string and a number (or between two differently-typed values) produces an unexpected true/false result, sometimes with real security consequences (e.g., a hash comparison that coerces in an attacker-exploitable way)
- **Cause**: `==` triggers PHP's type coercion rules before comparing, and while PHP 8 tightened several of the most infamous cases, coercion still happens and can still surprise — `===` is unaffected by any of this
- **Symptoms**:
  - A conditional behaves unexpectedly for specific input values that "shouldn't" match
  - A security-sensitive comparison (token/hash check) is bypassable with a crafted input
- **Solution**: Default to `===`/`!==` everywhere; if loose comparison is genuinely needed, understand and document exactly why, and never use `==` for anything security-sensitive (hash/token comparisons should use `hash_equals()` specifically, which is also timing-attack-safe)
- **Details**: → [extended/checklists.md#security-checklist]

### SE-2: Mass Assignment Vulnerabilities
- **Severity**: critical
- **Situation**: A controller passes raw request input directly into an Eloquent model's `create()`/`fill()` without a fillable allowlist, and a user submits an unexpected field (e.g., `is_admin=1`) that gets silently persisted
- **Cause**: Without `$fillable`/`$guarded` defined, mass-assignment methods accept whatever keys are present in the input array, with no distinction between fields the request should and shouldn't be able to set
- **Symptoms**:
  - A privilege-escalation or data-tampering bug traces back to a field the request wasn't supposed to be able to control
- **Solution**: Always define `$fillable` (allowlist) or `$guarded` on Eloquent models, and prefer validating input into an explicit DTO/array of only the expected fields before passing to `create()`/`update()`

### SE-3: N+1 Queries From Lazy-Loaded Relationships
- **Severity**: high
- **Situation**: Fetching a list of models, then accessing a relationship on each one inside a loop (e.g., `foreach ($posts as $post) { echo $post->author->name; }`), issues one additional query per item instead of one query total
- **Cause**: Eloquent/Doctrine relationships are lazy-loaded by default — accessing them for the first time triggers a query, and a loop over N parent models triggers N additional queries for the child relationship
- **Symptoms**:
  - Response time scales linearly with result set size
  - Query logs show a burst of near-identical queries differing only by an ID
- **Solution**: Eager-load relationships with `->with('author')` (Eloquent) or an explicit `JOIN`/fetch strategy (Doctrine) before the loop, and use query logging in development to catch N+1 patterns before they reach production

### SE-4: Silent Failures From Error Suppression
- **Severity**: medium
- **Situation**: The `@` error-suppression operator hides a real failure (a file that failed to open, a function called with wrong arguments), and the code continues executing on bad/missing data as if nothing went wrong
- **Cause**: `@` suppresses PHP's error reporting for the expression it's attached to, converting what should be a visible warning/error into silence — the underlying problem still happened, it's just no longer reported
- **Symptoms**:
  - Data corruption or incorrect output with no corresponding error in logs
  - Debugging requires temporarily removing `@` operators to see what's actually failing
- **Solution**: Never use `@` to suppress errors; handle the specific failure condition explicitly (check return values, catch exceptions, validate preconditions) so failures are visible and actionable

### SE-5: Stale OPcache After Deployment
- **Severity**: medium
- **Situation**: A deployment updates PHP source files, but the running application continues serving old behavior because OPcache is still serving compiled bytecode from before the deployment
- **Cause**: OPcache caches compiled bytecode keyed by file path/timestamp (depending on configuration) to avoid recompiling PHP source on every request — if timestamp validation is disabled for performance (common in production configs) or the cache isn't explicitly cleared, updated source won't take effect
- **Symptoms**:
  - A deployed bug fix doesn't appear to take effect until a server restart or manual cache clear
  - Behavior is inconsistent across a load-balanced fleet where OPcache was cleared on some workers but not others
- **Solution**: Include an explicit OPcache reset (`opcache_reset()` or a full worker restart) as a deployment step when `opcache.validate_timestamps` is disabled, and ensure the deployment process clears the cache consistently across every worker/server in the fleet

## Recommended Tools

| Category | Tools |
|------|------|
| Dependency management | Composer |
| Static analysis | PHPStan, Psalm |
| Testing | PHPUnit, Pest |
| Code style | PHP-CS-Fixer, PSR-12 |

## Security Hardening

**Full checklist**: → [extended/checklists.md#security-checklist]

## Related Resources

[PHP Manual](https://www.php.net/manual/en/) | [PSR Standards](https://www.php-fig.org/psr/) | [Laravel Docs](https://laravel.com/docs)

## Related Domains

[[mysql]] | [[javascript]] | [[docker]]
