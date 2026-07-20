# Supabase — Extended Checklists

## RLS Security Checklist

- [ ] RLS enabled on every table, with no table left in the default "disabled" state before client integration
- [ ] Explicit policies written per operation (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) rather than assuming one policy covers all
- [ ] `service_role` key confirmed absent from any client-side bundle, mobile app, or public repository
- [ ] `anon` key used for all client-side access, paired with correctly scoped RLS policies
- [ ] Views used to scope the auto-generated API to intended columns/joins, not raw tables exposed directly
- [ ] Secret scanning (gitleaks/trufflehog or platform-native) run in CI to catch accidental `service_role` key exposure
- [ ] RLS policy status (`pg_tables`/dashboard) audited on a recurring basis, not just at initial setup

## Schema & Scaling Checklist

- [ ] All schema changes made through versioned migration files (`supabase migration new`), not direct dashboard edits
- [ ] `supabase db diff` checked periodically to catch drift between local migrations and live environments
- [ ] Realtime enabled only on tables that genuinely need live updates, not broadly by default
- [ ] Realtime connection count monitored against plan limits as user base grows
- [ ] Database connection pooling (Supavisor/PgBouncer) configured appropriately for the app's concurrency profile
- [ ] Extensions (`pgvector`, `pg_cron`, etc.) reviewed for version compatibility before major Supabase platform upgrades
