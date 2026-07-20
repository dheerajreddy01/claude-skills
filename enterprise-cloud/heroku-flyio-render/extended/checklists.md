# Heroku / Fly.io / Render — Extended Checklists

## Deployment Readiness Checklist

- [ ] No application code writes durable data (uploads, generated files, local databases) to the local filesystem
- [ ] File uploads/generated assets routed through S3-compatible object storage or a managed database
- [ ] Free/hobby tier sleep-on-idle behavior accounted for, or upgraded away from, for any production-facing app
- [ ] Database connection pool size calculated against (per-instance size × max instance count), not just tested at low scale
- [ ] Config vars/secrets managed through the platform's native secrets UI/CLI, never committed to the deployed repo
- [ ] Buildpack detection verified explicitly in build logs after any dependency or repo structure change
- [ ] Build cache cleared deliberately when a deploy is suspected of using stale layers
- [ ] Deployment region matches the actual concentration of the user base

## Operational Checklist

- [ ] Uptime monitoring thresholds account for expected cold-start latency on free/hobby tiers
- [ ] Health check endpoint configured and used for zero-downtime deploy verification
- [ ] Add-ons/managed services (databases, caches) sized appropriately for the app's actual instance count and load
- [ ] Environment parity checked between staging and production config vars, not just code
- [ ] Rollback procedure verified (previous release/build accessible and deployable) before relying on it during an incident
- [ ] Logs shipped to a persistent log aggregator rather than relying solely on platform-native ephemeral log retention
