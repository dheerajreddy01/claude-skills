# OpenAPI / GraphQL — Extended Checklists

## API Design Checklist

- [ ] Single source of truth chosen (spec-first or code-first) and enforced in CI — no manually-maintained duplicate
- [ ] Contract tests validate implementation against the OpenAPI spec on every relevant change
- [ ] Every GraphQL list-of-related-object field uses a DataLoader/batching layer, not per-item resolution
- [ ] GraphQL query depth limiting and/or cost analysis enforced before the schema is exposed beyond trusted internal clients
- [ ] Production GraphQL introspection disabled for non-public APIs
- [ ] Field-level authorization enforced in resolvers, not assumed from schema obscurity
- [ ] REST versioning strategy decided before the first breaking change is needed
- [ ] GraphQL field deprecation (`@deprecated(reason: ...)`) used with a migration window before removal
- [ ] Public GraphQL schema deliberately scoped — not a straight mirror of internal data models

## Evolution & Governance Checklist

- [ ] Breaking changes (REST field removal/rename, GraphQL type changes) go through an explicit deprecation announcement and window
- [ ] Schema/spec changes reviewed in PRs the same way code changes are, with API consumers considered as stakeholders
- [ ] Persisted query allowlisting evaluated for production GraphQL endpoints serving untrusted/public clients
- [ ] Client SDK generation (from OpenAPI) re-run and published as part of the release process, not a manual afterthought
- [ ] API changelog maintained and linked from documentation for consumers to track evolution
- [ ] Rate limiting applied per-client/per-query-cost for GraphQL, not just per-request-count like typical REST rate limiting
- [ ] Schema diffing (e.g., GraphQL Inspector) run in CI to flag breaking changes before merge
