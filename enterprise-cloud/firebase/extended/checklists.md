# Firebase — Extended Checklists

## Security Rules Checklist

- [ ] No collection uses `allow read, write: if true;` (or equivalent unconditional access) in production
- [ ] Every rule checks authentication (`request.auth != null`) before checking anything else
- [ ] Ownership/role checks compare `request.auth.uid` against the actual document's owner/role field, not just resource existence
- [ ] Rules tested against realistic `list`/query access patterns, not just single-document `get`
- [ ] Firestore and Cloud Storage rules reviewed independently — securing one does not secure the other
- [ ] Rules Playground or `@firebase/rules-unit-testing` used in CI to catch rule regressions before deploy
- [ ] Rules deployed as part of the same release/PR as any schema or query pattern change they need to support

## Data & Cost Management Checklist

- [ ] Composite indexes created proactively for every multi-filter/sort query shape, tracked in `firestore.indexes.json`
- [ ] Real-time listeners scoped to specific documents or narrowly filtered, paginated queries — not entire collections
- [ ] Listeners explicitly detached when a component/screen/session ends
- [ ] Unbounded array fields identified and restructured into subcollections before hitting the 1MB document limit
- [ ] Bulk write operations use `writeBatch`/Admin SDK bulk writer, not per-document loops
- [ ] Cloud Functions triggered by Firestore writes reviewed for fan-out risk under bulk/import operations
- [ ] Firestore usage dashboard (reads/writes/deletes) monitored regularly, not just reviewed after a cost spike
