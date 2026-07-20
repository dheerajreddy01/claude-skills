---
schema: "1.0"
name: openapi-graphql
version: "1.0.0"
description: OpenAPI/Swagger REST contract design and GraphQL schema/resolver practices
domain: technology
triggers:
  keywords:
    primary: [openapi, swagger, graphql, api design, api schema]
    secondary: [resolver, api spec, contract-first, codegen, DataLoader]
  context_boost: [REST endpoint, API contract, schema definition]
  context_penalty: [postman, database schema]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# OpenAPI / GraphQL (API Design)

> A contract nobody enforces isn't a contract — it's a comment

## Applicable Scenarios

- Designing a REST API contract with OpenAPI/Swagger, contract-first or code-first
- Designing a GraphQL schema, its types, and resolver strategy
- Choosing between REST and GraphQL for a given API's access patterns
- Diagnosing GraphQL performance problems (N+1 queries, unbounded query cost)
- Planning API versioning or schema evolution without breaking existing clients

## Core Knowledge

### REST + OpenAPI

- **OpenAPI/Swagger** describes a REST API's endpoints, request/response shapes, and auth as a machine-readable spec (YAML/JSON) — it can drive documentation, client SDK generation, server stub generation, and contract testing
- **Contract-first**: write the OpenAPI spec first, generate server stubs/client SDKs from it — the spec is the source of truth
- **Code-first**: annotate the implementation (decorators/comments) and generate the OpenAPI spec from the code — the code is the source of truth, spec is derived
- Either approach works, but mixing them inconsistently across a codebase (some endpoints hand-documented, some generated) is how specs silently drift from reality

### GraphQL

| Concept | Role |
|------|------|
| **Schema** | Strongly-typed contract defining every queryable type, field, and mutation — single endpoint, client specifies exactly what it needs |
| **Resolver** | Function that fetches the actual data for a given field — one resolver per field, composed as the query executes |
| **Query/Mutation/Subscription** | The three root operation types — reads, writes, and real-time updates respectively |

GraphQL solves REST's over-fetching/under-fetching problem (client gets exactly the fields it asks for, in one round trip across nested relationships) — but that flexibility means the server has far less control over what a given request actually costs to execute.

### REST Versioning

| Strategy | Tradeoff |
|------|------|
| **URI versioning** (`/v2/users`) | Simple, explicit, but proliferates duplicate routes |
| **Header versioning** (`Accept: application/vnd.api+json;version=2`) | Cleaner URIs, less discoverable/harder to test manually |
| **Additive-only, no versioning** | Works only if changes are strictly backward-compatible (new optional fields, never removing/renaming) |

### GraphQL Schema Evolution

- GraphQL favors **additive, non-breaking evolution**: add new fields/types freely, mark old fields `@deprecated` with a reason, remove only after confirmed client migration
- Because a single schema serves every client, a breaking change (removing/renaming a field, changing a type) breaks every consumer simultaneously — there's no per-client version negotiation the way URI versioning provides in REST

## Best Practices

1. **Pick one source of truth** (spec-first or code-first) and enforce it in CI — don't let the other drift silently
2. **Batch resolver data fetching** (DataLoader pattern) for any GraphQL field that resolves per-item in a list — never let list resolution trigger one backend call per item
3. **Bound GraphQL query cost** — depth limiting, complexity/cost analysis, or persisted queries — before exposing a schema publicly
4. **Version REST APIs deliberately** and communicate deprecation timelines, rather than breaking changes silently
5. **Deprecate, don't delete, GraphQL fields** — give clients a migration window with `@deprecated(reason: "...")`
6. **Validate requests/responses against the OpenAPI spec in CI** (contract testing) so drift is caught automatically, not discovered by a confused API consumer

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Hand-maintaining an OpenAPI spec alongside the implementation with no sync check | Generate the spec from code (or validate code against spec) in CI |
| A list field's resolver issuing one query per item | Use a DataLoader/batching layer to collapse per-item fetches into one batched call |
| Publishing a GraphQL schema with no query depth/complexity limit | Add depth limiting and/or query cost analysis before exposing the schema beyond trusted clients |
| Removing a GraphQL field the moment it's replaced | Mark it `@deprecated` first, remove only after confirming no clients still use it |
| Breaking a REST response shape with no version bump | Version deliberately (URI or header) and communicate a deprecation window |

## Sharp Edges

### SE-1: The GraphQL N+1 Resolver Problem
- **Severity**: critical
- **Situation**: A query for a list of items, each resolving a related field (e.g., `posts { author { name } }`), triggers one database/backend call per post to resolve each author — a list of 100 posts becomes 101 backend calls for what should be 2
- **Cause**: GraphQL resolvers execute independently per field per object by default — nothing automatically batches or dedupes the repeated `author` resolution across every post in the list
- **Symptoms**:
  - Backend/database query logs show a burst of near-identical queries proportional to list length
  - Response time scales linearly (or worse) with the number of items in a list field
- **Solution**: Use a batching/dedup layer (the DataLoader pattern) so all `author` lookups for a single query execution are collected and issued as one batched backend call
- **Details**: → [extended/checklists.md#api-design-checklist]

### SE-2: OpenAPI Spec Drifting From Actual Implementation
- **Severity**: high
- **Situation**: An OpenAPI spec that's supposed to document the API no longer matches what the API actually returns — fields were added/removed/renamed in the implementation without a corresponding spec update
- **Cause**: Nothing enforces the spec and implementation stay in sync unless it's actively wired into the build/test pipeline — a hand-maintained spec is just documentation that happens to be structured, and structured documentation rots exactly like prose documentation does
- **Symptoms**:
  - Generated client SDKs fail at runtime on fields the spec didn't mention
  - API consumers report the "documented" contract doesn't match observed behavior
- **Solution**: Generate the spec from code annotations (code-first) or validate the implementation against the spec in CI (contract testing, e.g., via Dredd or Schemathesis) so any drift fails the build instead of surfacing in production

### SE-3: Unbounded GraphQL Query Cost Enabling DoS-Like Load
- **Severity**: critical
- **Situation**: A single deeply nested or broadly-fanned-out GraphQL query (accidental from a client, or deliberate from an attacker) triggers an enormous amount of backend work, degrading the service for everyone
- **Cause**: GraphQL's flexibility means the client effectively composes the query the server has to execute — without a depth limit or cost analysis, nothing caps how expensive a syntactically valid query can be
- **Symptoms**:
  - A latency/load spike traces back to a single unusually large or deeply nested query
  - Server resource usage is disproportionate to normal request volume during the spike
- **Solution**: Implement query depth limiting, field-level cost/complexity analysis with a maximum threshold, and/or restrict production traffic to an allowlist of persisted queries rather than accepting arbitrary ad hoc queries from untrusted clients

### SE-4: REST Breaking Changes Shipped Without Versioning
- **Severity**: high
- **Situation**: A REST endpoint's response shape changes (a field renamed, a type changed, a field removed) without a version bump, and every existing client integration breaks simultaneously on deploy
- **Cause**: Without an explicit versioning strategy, "the API" is really just "whatever the latest deploy happens to return" — there's no mechanism for a client to keep working against the old shape while migrating
- **Symptoms**:
  - Client-reported errors spike immediately after a specific deploy
  - The breaking field change is found in a diff of the OpenAPI spec (or would have been, if contract testing existed)
- **Solution**: Adopt an explicit versioning strategy (URI or header-based) before the first breaking change is needed, treat additive-only changes as the default for unversioned endpoints, and require an explicit deprecation window for anything that must break

### SE-5: Over-Permissive GraphQL Schema Exposing Internal Fields
- **Severity**: medium
- **Situation**: A GraphQL schema exposes fields or mutations that were only meant for internal/admin tooling, because the resolver layer was built against the full internal data model rather than a deliberately scoped public schema
- **Cause**: It's easy to expose a field simply because the underlying data model has it, rather than because an external consumer should be able to query it — GraphQL's introspection makes the full available schema (including anything under-considered) discoverable to any client
- **Symptoms**:
  - A security review or introspection query reveals fields/mutations with no legitimate external use case
  - An internal-only mutation is called by an unauthorized external client because nothing but obscurity was protecting it
- **Solution**: Design the public schema deliberately as its own artifact (not a straight mirror of internal models), disable introspection in production for non-public APIs, and enforce field-level authorization in resolvers rather than relying on the schema simply not being "discovered"

## Recommended Tools

| Category | Tools |
|------|------|
| OpenAPI tooling | Swagger UI/Editor, Redocly, openapi-generator |
| Contract testing | Dredd, Schemathesis, Pact |
| GraphQL servers | Apollo Server, GraphQL Yoga, graphql-js |
| GraphQL tooling | DataLoader, GraphQL Inspector (schema diffing), Apollo Studio |

## API Design Discipline

**Full checklist**: → [extended/checklists.md#api-design-checklist]

## Related Resources

[OpenAPI Specification](https://spec.openapis.org/oas/latest.html) | [GraphQL Docs](https://graphql.org/learn/) | [DataLoader](https://github.com/graphql/dataloader)

## Related Domains

[[postman]] | [[javascript]] | [[python]] | [[github-actions]]
