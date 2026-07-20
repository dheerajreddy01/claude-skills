# Postman — Extended Checklists

## Collection Hygiene Checklist

- [ ] No live secrets hardcoded in requests, collection files, or committed environment files
- [ ] Sensitive variables marked as "secret" type; real credentials sourced from a secure store at runtime for CI
- [ ] Every meaningful request has `pm.test()` assertions covering status code, response schema, and key field values — not just "response received"
- [ ] Collection exported and version-controlled alongside the API code, reviewed in the same PRs that change endpoints
- [ ] Environment names unambiguous (e.g., `staging`, `prod`) with no risk of confusing which is selected
- [ ] Destructive requests (DELETE, bulk mutations) guarded against accidental execution against production
- [ ] Mock servers used for frontend/backend parallel development where the real endpoint isn't ready yet

## CI Integration Checklist

- [ ] Newman runs in CI on every relevant API/collection change
- [ ] CI job explicitly checks Newman's exit code and fails the build on non-zero
- [ ] CI job configuration audited periodically to confirm the test step is actually gating, not just informational
- [ ] Correct environment file explicitly specified per CI job — no reliance on an implicit/default environment
- [ ] Newman reporters (HTML/JUnit) configured so failures are visible without digging through raw logs
