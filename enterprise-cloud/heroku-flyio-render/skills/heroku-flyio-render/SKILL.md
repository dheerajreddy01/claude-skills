---
schema: "1.0"
name: heroku-flyio-render
version: "1.0.0"
description: Heroku, Fly.io, and Render simple-PaaS deployment practices — ephemeral filesystems, sleep-on-idle, and buildpacks
domain: technology
triggers:
  keywords:
    primary: [heroku, fly.io, flyio, render, buildpack]
    secondary: [dyno, procfile, fly toml, render.yaml, ephemeral filesystem]
  context_boost: [simple deployment, small app hosting, side project deploy]
  context_penalty: [kubernetes, aws, terraform]
priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Heroku / Fly.io / Render (Simple PaaS)

> The filesystem is not disk — nothing you write to it survives the next deploy or restart

## Applicable Scenarios

- Deploying a small-to-medium app without managing raw infrastructure
- Choosing between Heroku, Fly.io, and Render for a given project's needs
- Diagnosing "my uploaded files disappeared" or slow first-request issues
- Managing environment/config variables and add-ons across environments
- Planning database connection limits across scaled-out app instances

## Core Knowledge

### The Ephemeral Filesystem Model

- Every one of these platforms runs your app in a container/VM whose local filesystem is **wiped on every restart, redeploy, or dyno cycle** — nothing written to local disk (uploaded files, SQLite databases, generated caches) is durable
- This is a deliberate platform design choice enabling stateless horizontal scaling, not a bug — durable storage requires an explicit external service (S3-compatible object storage, a managed database, Fly.io Volumes for the specific case where a single-instance stateful workload is intentional)

### Sleep-on-Idle / Cold Starts

- Free/hobby tiers on these platforms commonly **sleep an app after a period of inactivity**, and the next incoming request pays the cost of waking it back up — this can take several seconds and looks like an outage to a user or an uptime monitor
- Paid tiers typically remove this behavior, but the free-tier default is easy to forget about until a demo or low-traffic production app hits it

### Build Model: Buildpacks vs. Docker

| Approach | How it Works | Tradeoff |
|------|------|------|
| **Buildpacks** | Platform auto-detects language/framework and builds an image without a Dockerfile | Fast to get started, less control, occasional wrong-detection or stale-cache issues |
| **Docker-based deploy** | You provide the Dockerfile/image directly | Full control, more setup, consistent with how the app might be deployed elsewhere |

Buildpack auto-detection can pick the wrong build path (e.g., detecting a `package.json` in a monorepo subfolder incorrectly) or reuse a cached layer that misses a genuinely new dependency — worth understanding as a real (if infrequent) failure mode, not assuming it "just works" forever.

### Platform-Specific Notes

- **Heroku**: Config vars per app, a mature add-on marketplace (databases, monitoring, etc.), Procfile defines process types
- **Fly.io**: Deploys as Firecracker microVMs, closer to raw infrastructure with global anycast networking and explicit region placement; supports persistent Volumes for genuinely stateful single-instance workloads
- **Render**: Similar developer experience to Heroku with a modern dashboard, native cron jobs and background workers, `render.yaml` for infrastructure-as-code style config

## Best Practices

1. **Never write anything you need to keep to the local filesystem** — use object storage (S3-compatible) for uploads, a managed database for persistent data
2. **Account for cold-start latency** on free/hobby tiers in any user-facing or monitored production path — upgrade the tier or add a keep-alive ping if it matters
3. **Pin dependency versions and clear build caches deliberately** when a buildpack build seems to be using stale layers
4. **Size database connection pools per-instance with the total scaled-out instance count in mind** — 10 connections × 10 dynos can exceed a managed database's connection limit fast
5. **Deploy region close to your actual user base**, especially on platforms like Fly.io where region choice is explicit and consequential
6. **Use the platform's native config var/secrets management**, never commit `.env` files with real credentials to the deployed repo

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Storing user file uploads on local disk | Use S3-compatible object storage (or the platform's native equivalent) |
| Assuming a free-tier app is always warm and fast to respond | Account for sleep-on-idle cold starts, or upgrade to a tier without that behavior |
| Each of N scaled-out instances opening its own full-size DB connection pool | Size total connections against the database's actual limit, or use a connection pooler |
| Committing real secrets in a `.env` file to the deployed repo | Use the platform's config var/secrets UI or CLI |
| Assuming buildpack detection will always pick the right language/framework | Verify the detected buildpack explicitly, especially in monorepos or multi-language repos |

## Sharp Edges

### SE-1: Ephemeral Filesystem Silently Losing Uploaded Data
- **Severity**: critical
- **Situation**: An app writes user-uploaded files or generated reports to the local filesystem, it works fine in testing (no restart happened), then a routine redeploy or dyno cycle wipes everything that was written, with no error or warning at write time
- **Cause**: The local filesystem on these platforms is explicitly ephemeral — nothing in the write path itself fails or warns, the data simply doesn't exist anymore after the next container cycle
- **Symptoms**:
  - User-reported "my file disappeared" complaints correlated with deploy timestamps, not with any user action
  - Files present immediately after upload but gone after the next scheduled restart or deploy
- **Solution**: Route all file storage through an external durable service (S3-compatible object storage) from the start, and treat any local-disk write as scratch space only, cleared on every cycle by design
- **Details**: → [extended/checklists.md#deployment-readiness-checklist]

### SE-2: Sleep-on-Idle Cold Start Mistaken for an Outage
- **Severity**: medium
- **Situation**: A free/hobby-tier app that's had no traffic for a while receives a request, and the response takes several seconds while the platform spins the dyno/instance back up — an uptime monitor or an impatient user reports it as down
- **Cause**: Free/hobby tiers on these platforms intentionally sleep idle apps to conserve shared resources, and the wake-up latency is a real, if usually brief, cold-start cost
- **Symptoms**:
  - Uptime monitoring alerts fire for a "down" service that resolves itself on the very next check
  - The first request after a period of inactivity is measurably slower than subsequent ones
- **Solution**: Upgrade to a paid tier without sleep-on-idle for anything genuinely production-facing, or if staying on the free tier is a deliberate choice, configure uptime monitoring thresholds/expectations accordingly rather than treating cold starts as incidents

### SE-3: Per-Instance Connection Pooling Exceeding Database Limits
- **Severity**: high
- **Situation**: An app is scaled out to multiple dynos/instances for load handling, each opening its own configured-size database connection pool, and the total connection count across all instances exceeds the managed database's connection limit — causing new connections (including from legitimate instances) to be refused
- **Cause**: Connection pool size is typically configured per-instance in application code, and nothing automatically coordinates or caps the aggregate total across however many instances happen to be running
- **Symptoms**:
  - `too many connections` (or platform-equivalent) database errors that appear only after scaling out, not at low instance counts
  - Intermittent connection failures that correlate with autoscaling events
- **Solution**: Calculate total connections as (per-instance pool size × max instance count) against the database's actual connection limit, size pools conservatively, and consider a connection pooler (e.g., PgBouncer, or the platform's managed pooling add-on) for high instance-count scenarios

### SE-4: Buildpack Misdetection or Stale Build Cache
- **Severity**: medium
- **Situation**: A deploy silently uses a cached build layer that predates a genuinely new dependency, or buildpack auto-detection picks the wrong language/framework in a monorepo, and the deployed app runs old code or fails to start despite the source repo being correct
- **Cause**: Buildpacks cache layers for speed and auto-detect the app type from repo structure — both are convenience features that can occasionally produce a stale or incorrect result without a clear error pointing at the actual cause
- **Symptoms**:
  - A dependency added in the latest commit isn't present in the running app after deploy
  - Build logs show the wrong language/framework being detected for a monorepo subproject
- **Solution**: Clear the build cache explicitly when a build seems to be using stale layers, pin the buildpack explicitly rather than relying purely on auto-detection for non-trivial repo structures, and verify build logs show the expected detected stack on every deploy that changes dependencies

### SE-5: Region Mismatch Causing Unnecessary Latency
- **Severity**: low
- **Situation**: An app is deployed to a default or arbitrarily chosen region, and the actual user base is concentrated somewhere geographically distant, adding consistent, avoidable latency to every request
- **Cause**: These platforms don't always default to the "right" region for a given user base, and region selection is easy to skip past during initial setup when everything "works" in local testing regardless of region
- **Symptoms**:
  - Latency metrics show a consistent baseline delay disproportionate to the app's actual processing time
  - User complaints about sluggishness concentrated in a specific geography
- **Solution**: Deploy to the region closest to the actual (or primary) user base, and for platforms like Fly.io that support multi-region deployment, consider deploying closer to multiple concentrated user populations rather than a single region for everyone

## Recommended Tools

| Category | Tools |
|------|------|
| CLI/deployment | Heroku CLI, `flyctl`, Render CLI/`render.yaml` |
| Object storage | AWS S3, Cloudflare R2, Backblaze B2 |
| Connection pooling | PgBouncer, platform-managed pooling add-ons |
| Monitoring | Platform-native metrics, external uptime monitors (with cold-start-aware thresholds) |

## Deployment Readiness

**Full checklist**: → [extended/checklists.md#deployment-readiness-checklist]

## Related Resources

[Heroku Dev Center](https://devcenter.heroku.com/) | [Fly.io Docs](https://fly.io/docs/) | [Render Docs](https://render.com/docs)

## Related Domains

[[docker]] | [[postgresql]] | [[aws]]
