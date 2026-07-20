---
schema: "1.0"
name: vercel-netlify
version: "1.0.0"
description: Vercel/Netlify serverless functions, preview deployments, ISR caching, and JAMstack build practices
domain: technology
triggers:
  keywords:
    primary: [vercel, netlify, jamstack, edge function]
    secondary: [static site, serverless function, preview deployment, ISR]
  context_boost: [frontend deployment, next.js, static site generator]
  context_penalty: [aws, azure, gcp, kubernetes]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Vercel / Netlify

> Preview deployments make every PR a live environment — which means every environment variable scope decision matters per-branch, not just per-project

## Applicable Scenarios

- Designing serverless/edge function architecture for a frontend-hosted app
- Configuring environment variables safely across production/preview/development
- Diagnosing function timeout, cold start, or ISR caching issues
- Reviewing build pipeline and bandwidth/build-minute cost drivers
- Setting up edge middleware for routing/auth logic

## Core Knowledge

### Deployment Model

- Every push to a branch (and every PR) typically gets its own **preview deployment** — a fully live, isolated environment with its own URL, which is the platform's core differentiator versus traditional single-environment hosting
- **Production** deployments are typically triggered by merges to the default branch (or an explicit promote action)

### Functions: Serverless vs. Edge

| Type | Runtime | Characteristics |
|------|------|------|
| **Serverless functions** | Node.js/Python/Go runtime, per-region | Full runtime API access, higher cold-start latency, execution time limits (varies by plan) |
| **Edge functions** | V8-isolate-based, runs at edge locations globally | Lower latency/cold-start, but a restricted runtime API (no full Node.js API surface) |

Edge functions trade full runtime compatibility for latency — code using Node-specific APIs (`fs`, certain native modules) generally can't run there without adaptation.

### ISR (Incremental Static Regeneration)

- ISR lets statically generated pages be regenerated in the background after a configured revalidation interval, without a full rebuild — serving stale content while regenerating, then swapping in the fresh version
- Cache invalidation for ISR is asynchronous and eventually consistent — a content update doesn't guarantee an immediately fresh response for every visitor, especially across multiple edge regions

### Environment Variables

- Variables are scoped per environment (**Production**, **Preview**, **Development**), and per-branch overrides are possible in some platforms — a variable set only for Production won't be available in a preview deployment unless explicitly configured for Preview too
- Preview deployments are, by nature, more widely accessible (shareable URLs, sometimes without auth) than production — anything sensitive scoped into the Preview environment carries a wider exposure surface than the same value in Production

## Best Practices

1. **Scope environment variables deliberately per environment** — don't assume a Production-only variable falls back safely, and don't put production-grade secrets in Preview scope without considering preview URLs' broader accessibility
2. **Design serverless functions to fit within cold-start and timeout budgets** — keep dependencies lean, avoid long-running work that exceeds platform limits
3. **Understand what ISR actually guarantees** — treat it as eventually consistent, not immediate cache invalidation, especially right after a content change
4. **Use edge functions only for logic that fits the restricted runtime** — auth checks, redirects, header manipulation — not for anything needing full Node API access
5. **Monitor build minutes and bandwidth usage** against plan limits, especially for high-traffic sites or frequently-changing repos with many preview builds
6. **Scope edge middleware's route matcher precisely** — an overly broad matcher can intercept routes it wasn't intended to affect

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming a Production environment variable is available in Preview | Explicitly scope variables to every environment that needs them |
| Long-running or heavy-dependency code in a serverless function | Keep functions lean; move genuinely long-running work to a dedicated backend service |
| Assuming ISR-revalidated content is instantly fresh everywhere | Treat ISR cache invalidation as eventually consistent; use on-demand revalidation APIs where immediate freshness matters |
| Overly broad edge middleware matcher (e.g., matching all routes) | Scope the matcher precisely to the routes middleware is actually meant to affect |
| Not monitoring build/bandwidth usage until a bill surprise | Track usage against plan limits proactively, especially before high-traffic launches |

## Sharp Edges

### SE-1: Environment Variables Leaking Across Environment Scopes
- **Severity**: critical
- **Situation**: A secret intended only for Production (e.g., a live payment API key) gets scoped to "All Environments" for convenience, and it's now accessible from preview deployments — which are often more widely shared/less access-controlled than production
- **Cause**: Environment variable scoping is opt-in per environment, and it's easy to default to the broadest scope ("all environments") to avoid configuring each one separately, without considering that preview URLs have a different (often wider) exposure profile than production
- **Symptoms**:
  - A live/production credential is discoverable via a preview deployment's environment, accessible to anyone with the preview URL
- **Solution**: Scope secrets to the narrowest environment that needs them, use separate (test-mode) credentials for Preview/Development environments wherever the underlying service supports it, and treat preview deployment URLs as having a meaningfully different trust boundary than production
- **Details**: → [extended/checklists.md#deployment-safety-checklist]

### SE-2: Serverless Function Timeouts Breaking Long-Running Requests
- **Severity**: high
- **Situation**: A serverless function performing a longer operation (large file processing, a slow third-party API call, report generation) works in development but gets killed mid-execution in production once real-world latency or payload size exceeds the platform's execution time limit
- **Cause**: Serverless functions on these platforms have a hard execution time limit (which varies by plan/tier) — there's no way to extend a single invocation beyond it, unlike a traditional long-running server process
- **Symptoms**:
  - Requests fail with a timeout/function-execution-limit error, correlated with slower upstream dependencies or larger payloads
- **Solution**: Move genuinely long-running work to a background job/queue pattern (return immediately, process asynchronously, and let the client poll or receive a webhook/websocket update), or to a dedicated backend service not bound by the same execution limits

### SE-3: ISR Cache Invalidation Not Propagating as Expected
- **Severity**: high
- **Situation**: Content is updated in the source (CMS, database), but visitors continue to see stale content past the expected revalidation window, or different visitors see different versions depending on which edge region served them
- **Cause**: ISR's background regeneration is triggered per-page, per-region, and is eventually consistent — a revalidation interval is a target, not a guarantee, and on-demand revalidation (if not explicitly triggered) doesn't happen automatically on content change
- **Symptoms**:
  - Content updates take longer to appear than the configured revalidation interval would suggest
  - Different users see different content versions simultaneously, correlated with different edge regions
- **Solution**: Use on-demand revalidation APIs (triggered directly by the CMS/content update event) for content where near-immediate freshness matters, rather than relying solely on a time-based revalidation interval, and set expectations that ISR is an eventually-consistent caching strategy, not a live database read

### SE-4: Build Minutes and Bandwidth Cost Surprises at Scale
- **Severity**: medium
- **Situation**: A project with frequent commits/PRs (each triggering a preview build) or a site with high traffic/large assets accumulates build-minute or bandwidth usage well beyond what was budgeted, resulting in an unexpected bill or hitting a plan limit that blocks further deploys
- **Cause**: Preview deployments multiply build frequency (every PR/push, not just production merges), and bandwidth scales directly with traffic and asset size — neither is capped by default on usage-based plans
- **Symptoms**:
  - Billing dashboard shows build minutes or bandwidth usage climbing disproportionately to actual production deploy frequency
  - Deploys start failing or getting throttled once a plan limit is hit
- **Solution**: Monitor build-minute and bandwidth usage proactively (not just after a bill surprise), optimize asset sizes and caching headers to reduce bandwidth, and consider limiting preview deployment triggers (e.g., only on draft-PR-ready, not every push) if build volume is a genuine cost driver

### SE-5: Edge Middleware Matcher Scope Creep
- **Severity**: medium
- **Situation**: Edge middleware written for a specific purpose (e.g., an auth check on `/admin/*`) is configured with an overly broad route matcher (or no matcher, defaulting to all routes), and it ends up running on — and potentially affecting — routes it was never intended to touch
- **Cause**: Middleware route matchers are explicit configuration; without precise scoping, the default behavior can apply middleware globally, and its side effects (redirects, header changes, auth checks) then apply everywhere
- **Symptoms**:
  - Unrelated routes start behaving unexpectedly (unexpected redirects, added headers, or auth prompts) after middleware is added for a specific route's needs
- **Solution**: Scope middleware matchers precisely to the intended routes using explicit path patterns, and test that unrelated routes are genuinely unaffected before relying on a broad or default matcher configuration

## Recommended Tools

| Category | Tools |
|------|------|
| Local development | Vercel CLI, Netlify CLI |
| Frameworks | Next.js, Astro, Remix, Gatsby |
| Monitoring | Vercel Analytics/Speed Insights, Netlify Analytics |
| Forms/functions testing | Netlify Dev, Vercel `vercel dev` |

## Deployment Safety

**Full checklist**: → [extended/checklists.md#deployment-safety-checklist]

## Related Resources

[Vercel Docs](https://vercel.com/docs) | [Netlify Docs](https://docs.netlify.com/) | [Next.js Docs](https://nextjs.org/docs)

## Related Domains

[[cloudflare]] | [[javascript]] | [[github-actions]]
