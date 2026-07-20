---
schema: "1.0"
name: supabase
version: "1.0.0"
description: Supabase Postgres data modeling, Row Level Security, realtime subscriptions, and auto-generated API practices
domain: technology
triggers:
  keywords:
    primary: [supabase, row level security, RLS, supabase auth]
    secondary: [supabase realtime, service_role key, supabase functions, postgres policies]
  context_boost: [backend-as-a-service, postgres backend, auto-generated api]
  context_penalty: [firebase, mongodb, dynamodb]
priority: high
dependencies:
  software-skills: [postgresql]
author: claude-domain-skills
---

# Supabase

> It's Postgres underneath — every PostgreSQL sharp edge still applies, plus a new authorization model on top

## Applicable Scenarios

- Designing schema and Row Level Security policies for a Supabase-backed app
- Deciding what to expose through the auto-generated REST/GraphQL API versus a server-side function
- Diagnosing why the auto-generated API returns more or less data than expected
- Managing schema migrations across local, staging, and production Supabase projects
- Reviewing realtime subscription usage and connection limits

## Core Knowledge

### Postgres Underneath

- Supabase is a managed Postgres instance plus a set of generated services (REST API via PostgREST, realtime, auth, storage) layered on top — every core PostgreSQL concept (indexing, `EXPLAIN`, isolation levels, connection limits) applies directly, it isn't abstracted away
- Because it's real Postgres, arbitrary SQL, extensions (e.g., `pgvector`, `pg_cron`), and direct database connections are all available — a meaningfully different capability set than a proprietary document-store BaaS

### Row Level Security (RLS)

- RLS policies, not application code, are the actual authorization boundary for anything accessed through the auto-generated API — a table with RLS **disabled** is fully open to any request using the anon/public API key
- Policies are written as SQL predicates (`USING`/`WITH CHECK` clauses) evaluated per-row, per-operation (`SELECT`/`INSERT`/`UPDATE`/`DELETE` each need their own policy if access differs)
- The **`service_role` key bypasses RLS entirely** — it's meant for trusted server-side contexts only; if it ends up in client-side code, RLS provides no protection at all against a client using that key directly

### Auto-Generated API

- PostgREST auto-generates a REST API directly from the database schema — every table and column is potentially exposed unless explicitly restricted via RLS, views, or schema exposure settings
- Database **views** are the standard way to expose a controlled subset of columns/joins without changing the underlying table structure or over-exposing raw data

### Realtime

- Realtime subscriptions are built on Postgres's logical replication (WAL) — enabling realtime on a table has a real resource cost and a connection-count ceiling that scales differently than simple polling
- Realtime, by default, still respects RLS for `SELECT`-equivalent visibility, but this must be explicitly configured/verified per table, not assumed automatically consistent with REST API access

## Best Practices

1. **Enable RLS on every table by default**, then add specific policies — never leave a table with RLS disabled "temporarily"
2. **Never ship the `service_role` key to any client-side code** — it belongs only in trusted server environments (Edge Functions, your own backend)
3. **Use views to scope the auto-generated API** to exactly the columns/joins a client should see, rather than exposing raw tables broadly
4. **Manage schema changes through version-controlled migration files** (Supabase CLI), not by editing the schema directly in the dashboard
5. **Write a policy per operation type** (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) rather than assuming one policy covers all access patterns
6. **Load-test realtime subscription counts** before assuming they'll scale the same way a stateless REST API would

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Leaving RLS disabled on a table "just for now" during development | Enable RLS from the start and write policies alongside the schema, before any client integration |
| Shipping the `service_role` key in a mobile app or frontend bundle | Keep `service_role` server-side only; use the `anon` key + RLS for client access |
| Exposing every table/column directly through the auto-generated API | Use views to scope exactly what should be publicly queryable |
| Editing schema directly in the Supabase dashboard in production | Use versioned migration files (`supabase migration new`) applied consistently across environments |
| Assuming one `USING` policy covers inserts and updates too | Write explicit policies per operation — a `SELECT` policy doesn't govern `INSERT`/`UPDATE`/`DELETE` |

## Sharp Edges

### SE-1: RLS Disabled or Misconfigured Leaving Tables Fully Open
- **Severity**: critical
- **Situation**: A table has RLS left disabled (the Postgres default), or a policy written too permissively (e.g., `USING (true)` on every operation), and any client holding the public `anon` API key can read or write the entire table
- **Cause**: RLS is opt-in per table in Postgres — a newly created table has no row-level restriction until RLS is explicitly enabled and a policy defined, and it's easy to skip this step while focused on getting a feature working
- **Symptoms**:
  - A security review finds a table accessible via the REST API with no ownership/role restriction
  - Data appears modifiable by users who have no legitimate relationship to it
- **Solution**: Enable RLS on every table as it's created, write explicit per-operation policies before any client integration touches that table, and audit `pg_tables`/the Supabase dashboard's RLS status column regularly as a standing check
- **Details**: → [extended/checklists.md#rls-security-checklist]

### SE-2: `service_role` Key Leaking Into Client-Side Code
- **Severity**: critical
- **Situation**: The `service_role` key — meant only for trusted server contexts — ends up bundled into a mobile app, a frontend JS bundle, or a public repo, and anyone who extracts it gets full, RLS-bypassing access to the entire database
- **Cause**: The `service_role` and `anon` keys look superficially similar (both are JWTs configured in the same project settings), and it's an easy mistake to use the more powerful one during development because "it just works" without fighting RLS policies
- **Symptoms**:
  - Static analysis of a shipped app or a public repository reveals a `service_role`-looking key
  - Data modification occurs that no RLS policy should have permitted, tracing back to direct use of the bypass key
- **Solution**: Use the `anon` key plus correctly scoped RLS policies for all client-side access, keep `service_role` exclusively in server environments (Supabase Edge Functions, your own backend) and secret managers, and add a secret-scanning check for accidental key exposure in CI

### SE-3: Auto-Generated API Over-Exposing Schema
- **Severity**: high
- **Situation**: A table with columns never meant for public consumption (internal flags, cost data, other users' identifiers) is fully queryable through the auto-generated REST API because nothing scoped the exposed columns
- **Cause**: PostgREST generates API endpoints directly from the schema — every column is technically queryable (subject to RLS row-level filtering) unless deliberately restricted at the column level via a view or explicit API exposure configuration
- **Symptoms**:
  - A client can request columns via the REST API that the frontend never displays but that contain sensitive internal data
  - A security review finds internal-only fields returned in API responses
- **Solution**: Create views exposing only the intended columns/joins for public API consumption, and grant API access to the view rather than the underlying table directly

### SE-4: Dashboard Schema Edits Diverging From Migration History
- **Severity**: medium
- **Situation**: A schema change made directly in the Supabase dashboard's table editor works fine in that environment, but the change was never captured in a migration file — the next `supabase db push` to another environment doesn't include it, and staging/production silently diverge from what's assumed in the codebase
- **Cause**: The dashboard's visual schema editor makes direct changes convenient, but it's a separate path from the CLI's migration-file-based workflow, and nothing forces the two to stay in sync
- **Symptoms**:
  - A feature works in the environment where the dashboard edit was made but breaks in any environment provisioned from migration files alone
  - `supabase db diff` shows unexpected drift between the local migration history and a live environment
- **Solution**: Treat all schema changes as migration files from the start (`supabase migration new`), even for quick fixes, and use dashboard edits only for one-off inspection, immediately codifying any change into a migration afterward

### SE-5: Realtime Subscription Connection Limits Hit Unexpectedly
- **Severity**: medium
- **Situation**: An app scales its user base, and realtime subscriptions (each holding an open connection) hit the project's connection ceiling far sooner than the REST API would have, causing new subscription attempts to fail or existing ones to drop
- **Cause**: Realtime is built on Postgres logical replication with a bounded number of concurrent replication slots/connections — it doesn't scale the same way a stateless, poolable REST API does, and this ceiling is easy to overlook until real concurrent usage hits it
- **Symptoms**:
  - Realtime subscriptions start failing to establish or randomly disconnect as concurrent user count grows
  - The failure correlates with total connected clients, not with database query load
- **Solution**: Monitor realtime connection count against the project's plan limits, scope realtime usage to genuinely live-updating UI needs (not everything that could theoretically be live), and consider a dedicated pooling/connection strategy or plan upgrade before hitting the ceiling in production

## Recommended Tools

| Category | Tools |
|------|------|
| Local development | Supabase CLI, local Supabase stack (Docker-based) |
| Schema management | `supabase migration`, `supabase db diff` |
| Security testing | RLS policy testing via `psql`/pgTAP, Supabase dashboard policy editor |
| Extensions | `pgvector` (embeddings), `pg_cron` (scheduled jobs) |

## RLS Security Hardening

**Full checklist**: → [extended/checklists.md#rls-security-checklist]

## Related Resources

[Supabase Docs](https://supabase.com/docs) | [Row Level Security Guide](https://supabase.com/docs/guides/auth/row-level-security) | [PostgREST Docs](https://postgrest.org/)

## Related Domains

[[postgresql]] | [[firebase]] | [[vector-databases]]
