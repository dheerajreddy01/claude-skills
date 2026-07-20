---
schema: "1.0"
name: firebase
version: "1.0.0"
description: Firebase Firestore data modeling, security rules, Cloud Functions, and cost management
domain: technology
triggers:
  keywords:
    primary: [firebase, firestore, cloud functions for firebase, firebase auth]
    secondary: [security rules, realtime database, firebase hosting, composite index]
  context_boost: [mobile backend, backend-as-a-service, real-time app]
  context_penalty: [supabase, aws amplify, self-hosted backend]
priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Firebase

> Security rules aren't a filter on your backend — for most Firebase apps, they ARE the backend

## Applicable Scenarios

- Designing Firestore collection/document structure for a mobile or web app
- Writing or auditing Firebase Security Rules for Firestore or Storage
- Deciding when a Cloud Function is needed versus doing everything client-side
- Diagnosing Firestore read-cost spikes or degraded real-time listener performance
- Migrating from the Realtime Database to Firestore, or reasoning about which to use

## Core Knowledge

### Firestore Data Model

- Firestore is a **document/collection** store: documents live in collections, and documents can contain **subcollections**, but there is no native join across collections
- Every query pattern that touches more than one field with an inequality/range needs a matching **composite index** — Firestore does not build these automatically, and an unindexed query fails outright rather than running slowly
- Denormalization is expected and encouraged (mirroring data across documents) because there's no cheap way to join it back together at query time

### Security Rules

- Firebase Security Rules run **server-side, per request**, and are the actual authorization boundary — client SDK code has no enforced permissions of its own, so any check "in the app" that isn't also in the rules is not a real check
- Rules are evaluated per-document for reads and per-write for writes; a rule that looks restrictive on paper can still be too broad if it doesn't account for query-level access (e.g., a `list` query can return any document that individually passes the rule, even ones the developer didn't intend to expose via that query path)
- Rules for Firestore and Cloud Storage are configured and deployed independently — securing one does not secure the other

### Cloud Functions & Cost Model

- Cloud Functions for Firebase bill by invocation count, compute time, and network egress; functions triggered by Firestore writes fire **once per document write**, so a batch operation touching thousands of documents can fan out into thousands of invocations
- Cold starts are a real, measurable latency cost for infrequently-invoked functions — a factor in choosing between Cloud Functions and doing logic client-side (where security rules allow it)

### Real-Time Listeners

- `onSnapshot`/realtime listeners re-deliver the **entire matching result set** whenever any document in the query's result set changes, not just a delta of what changed — a listener on a large, frequently-updated collection can generate far more read cost and bandwidth than a one-time `get()` would suggest
- Every open listener holds a live connection; a client with many simultaneous listeners (common in poorly structured component trees in web/mobile apps) multiplies this cost per user session

## Best Practices

1. **Write security rules first, treat them as your actual API contract** — don't rely on client-side checks for anything that matters
2. **Denormalize deliberately** — duplicate the specific fields a query needs rather than trying to join, and plan for the write-side cost of keeping duplicates in sync
3. **Create composite indexes proactively** during development, not reactively when production queries start failing
4. **Scope real-time listeners narrowly** — listen to a single document or a tightly filtered, paginated query instead of an entire collection
5. **Batch writes and reads** (`writeBatch`, `getAll`) instead of looping individual operations, both for atomicity and for cost
6. **Monitor Firestore usage (reads/writes/deletes) in the console dashboard** as a standard operational check, not just when a bill spikes

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Relying on the app's UI to hide data instead of restricting it in Security Rules | Write rules that enforce the actual access boundary; assume any client can call the API directly |
| Attaching a real-time listener to an entire large collection | Scope listeners to specific documents or narrowly filtered, paginated queries |
| Discovering a missing composite index only when a production query throws | Create indexes proactively from the Firestore emulator's suggested-index errors during development |
| Looping single-document writes for a bulk update | Use `writeBatch` or the Admin SDK's bulk writer for multi-document operations |
| Treating Firestore like a relational database with joins | Denormalize data across documents to match actual query patterns |

## Sharp Edges

### SE-1: Overly Permissive Security Rules Exposing All Data
- **Severity**: critical
- **Situation**: A Firestore security rule written during prototyping (e.g., `allow read, write: if true;` or a rule scoped only by resource path with no ownership check) ships to production, and any authenticated (or even unauthenticated) client can read or write any document
- **Cause**: Firebase's default new-project rules are permissive for a limited trial window, and it's easy to never tighten them before shipping, especially since the app "works fine" during development regardless of rule strictness
- **Symptoms**:
  - A security audit or the Firebase console's rules simulator reveals a collection readable/writable by any client
  - Data corruption or exposure traced to a client that had no legitimate reason to access a document it modified
- **Solution**: Write rules that check authentication and ownership/role explicitly for every collection (`request.auth.uid == resource.data.ownerId`), test with the Rules Playground/emulator before every deploy, and treat "if true" rules as a hard blocker for any production release
- **Details**: → [extended/checklists.md#security-rules-checklist]

### SE-2: Real-Time Listener Read Amplification
- **Severity**: high
- **Situation**: A real-time listener attached to a moderately large, frequently-updated collection generates far more Firestore reads — and cost — than expected, because every update to any matching document re-delivers the whole result set to every subscribed client
- **Cause**: `onSnapshot` listeners don't send deltas by default in terms of billed reads — a listener matching N documents re-counts reads for the affected documents (and in some SDK/version behaviors, potentially the full set) on every relevant write, and this multiplies across every client with that listener open
- **Symptoms**:
  - Firestore usage dashboard shows read counts far exceeding what a naive "N users × N documents" estimate would predict
  - Cost scales with write frequency on a shared collection, not just with user count
- **Solution**: Narrow listeners to the smallest reasonable query (specific document, or a tightly filtered/paginated query), detach listeners when a component/screen unmounts, and use one-time `get()` reads for data that doesn't need live updates

### SE-3: Missing Composite Indexes Failing Only in Production
- **Severity**: high
- **Situation**: A query with multiple filters/an inequality plus a sort works fine locally against the emulator or a small dataset, then fails outright in production once real query complexity or data volume triggers Firestore's composite index requirement
- **Cause**: Firestore requires an explicit composite index for queries combining certain filter/sort patterns, and it doesn't build these automatically — a missing index causes an immediate error, not a slow query
- **Symptoms**:
  - `FAILED_PRECONDITION: The query requires an index` errors in production logs
  - A feature that worked in development throws immediately for real users hitting a previously-untested query shape
- **Solution**: Watch for the index-creation-link errors during development and create the suggested indexes proactively, review `firestore.indexes.json` as part of code review, and deploy indexes as part of the same release as any new query shape

### SE-4: Fan-Out Cost From Per-Document Cloud Function Triggers
- **Severity**: medium
- **Situation**: A Cloud Function triggered `onWrite`/`onCreate` for a Firestore collection is designed assuming occasional single-document writes, but a bulk import or batch operation writes thousands of documents at once, firing thousands of function invocations nearly simultaneously
- **Cause**: Firestore-triggered Cloud Functions fire once per matching document write with no built-in batching — the function has no visibility into "this write was part of a bulk operation"
- **Symptoms**:
  - A bulk data import correlates with a spike in Cloud Functions invocation count and cost
  - Downstream systems the function calls (e.g., a third-party API with rate limits) get overwhelmed by the sudden invocation burst
- **Solution**: For bulk operations, write through a path that bypasses the per-document trigger (e.g., a dedicated batch-processing Cloud Function or Admin SDK script that does the downstream work directly), or add throttling/queuing logic inside the trigger for genuinely per-document use cases that can spike in volume

### SE-5: Unbounded Document/Array Growth Hitting the 1MB Document Limit
- **Severity**: medium
- **Situation**: A document with an array field that grows by appending (e.g., a running list of comments or activity events) eventually approaches Firestore's 1MB per-document size limit, and further writes to that document fail
- **Cause**: Similar to other document databases, embedding an unbounded array inside a single document works until the natural growth of that array outpaces the size limit — nothing in Firestore's API warns about this until the limit is actually hit
- **Symptoms**:
  - Writes to specific long-lived "hot" documents start failing with a size-limit error
  - The failure correlates with the age/activity level of the parent entity, not with total collection size
- **Solution**: Identify unbounded array fields at design time and restructure them into a subcollection (one document per item) instead of a single growing array field, well before the size limit becomes a production incident

## Recommended Tools

| Category | Tools |
|------|------|
| Local development | Firebase Local Emulator Suite |
| Security testing | Rules Playground, `@firebase/rules-unit-testing` |
| Monitoring | Firebase console usage dashboard, Cloud Monitoring |
| CI/CD | Firebase CLI, GitHub Actions Firebase deploy action |

## Security Rules Hardening

**Full checklist**: → [extended/checklists.md#security-rules-checklist]

## Related Resources

[Firebase Docs](https://firebase.google.com/docs) | [Firestore Data Modeling Guide](https://firebase.google.com/docs/firestore/manage-data/structure-data) | [Security Rules Reference](https://firebase.google.com/docs/rules)

## Related Domains

[[supabase]] | [[gcp]] | [[mongodb]] | [[javascript]]
