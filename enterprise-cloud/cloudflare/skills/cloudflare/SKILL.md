---
schema: "1.0"
name: cloudflare
version: "1.0.0"
description: Cloudflare Workers, R2, DNS/CDN/WAF, and edge platform architecture practices
domain: technology
triggers:
  keywords:
    primary: [cloudflare, cloudflare workers, R2, CDN]
    secondary: [DNS, WAF, KV, durable objects, zero trust]
  context_boost: [edge computing, static site, DDoS protection]
  context_penalty: [aws, azure, gcp]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Cloudflare

> Workers run as isolates at the edge, not containers — the execution model shapes everything else

## Applicable Scenarios

- Designing edge-first architectures with Workers and Durable Objects
- Choosing between R2/KV/D1/Durable Objects for different storage needs
- Configuring DNS, caching, and WAF rules
- Diagnosing Worker CPU limit errors or stale cache/KV reads
- Setting up Zero Trust access for internal tools

## Core Knowledge

### Workers Execution Model

- Workers run in **V8 isolates**, not containers or VMs — they start in milliseconds with no cold-start container overhead, but this also means no persistent filesystem, no long-lived in-memory state between invocations (aside from what's explicitly stored), and a strict **CPU time limit** per request (not wall-clock time — waiting on I/O doesn't count against it, but actual computation does)
- Code runs at whichever of Cloudflare's edge locations is closest to the request, not in one centralized region

### Storage Options

| Service | Model | Best For |
|------|------|------|
| **R2** | S3-compatible object storage | Large blobs, no egress fees (Cloudflare's key differentiator vs. S3) |
| **KV** | Global, eventually-consistent key-value | Read-heavy, rarely-changing data (config, feature flags) |
| **Durable Objects** | Single-instance, strongly consistent, stateful actor | Coordination/state requiring strong consistency (e.g., a chat room, a rate limiter) |
| **D1** | SQLite-based distributed relational database | Relational data at the edge |

KV trades strong consistency for global read speed — writes propagate to edge locations asynchronously, so a read shortly after a write in a different region can return stale data.

### DNS, CDN & WAF

- Cloudflare sits in front of origin infrastructure as a reverse proxy — DNS records "proxied" (orange-clouded) through Cloudflare get CDN caching, DDoS protection, and WAF rule evaluation; "DNS only" (grey-clouded) records bypass all of that
- **WAF/firewall rules** evaluate in a defined order — a broad blocking rule earlier in the chain can shadow a more specific allow rule intended to override it for legitimate traffic
- Cache behavior is controlled by cache rules/page rules and origin cache-control headers together — misconfiguration between the two is a common source of serving stale content

### Zero Trust / Access

- Cloudflare Access puts an identity-aware authentication layer in front of internal applications/tools without a traditional VPN, based on policies (identity provider group membership, device posture, etc.)

## Best Practices

1. **Design Workers for the CPU-time model** — offload genuinely long-running work rather than assuming wall-clock-style execution budgets
2. **Choose the storage primitive that matches the consistency need** — KV for eventually-consistent reads, Durable Objects for strongly consistent coordination
3. **Understand proxied vs. DNS-only records** before assuming CDN/WAF protection applies to a given hostname
4. **Order WAF rules deliberately** — put specific allow rules before broader blocking rules where legitimate traffic could otherwise be caught
5. **Set explicit cache-control headers at origin** rather than relying solely on dashboard cache rules, so behavior is consistent and version-controlled
6. **Use R2 for large-object storage** where S3 egress fees would otherwise be a significant cost driver

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming Worker execution has a generous wall-clock budget | Design around CPU time limits; offload long computation to a backend service |
| Using KV for data requiring strong read-after-write consistency | Use Durable Objects (or a backend database) for anything requiring immediate consistency |
| Leaving a DNS record unproxied ("DNS only") assuming it's protected by Cloudflare's WAF/DDoS mitigation | Proxy ("orange-cloud") any record that should get CDN/WAF/DDoS protection |
| Broad WAF block rule placed before a needed allow rule | Order rules so specific allows take precedence over broader blocks where intended |
| No cache-control headers set at origin, relying entirely on dashboard defaults | Set explicit, version-controlled cache-control headers at origin |

## Sharp Edges

### SE-1: Workers CPU Time Limits Killing Requests Silently
- **Severity**: high
- **Situation**: A Worker performing heavier computation (e.g., image processing, complex data transformation) works fine in testing with small inputs but gets terminated mid-execution in production once inputs are larger, with the request failing
- **Cause**: Workers enforce a CPU time limit per invocation (separate from wall-clock time) — I/O waiting doesn't count against it, but actual JavaScript execution does, and exceeding the limit terminates the isolate
- **Symptoms**:
  - Requests fail intermittently, correlated with payload size or computational complexity, not traffic volume
  - Error logs show a CPU-limit-exceeded termination rather than an application-level exception
- **Solution**: Profile actual CPU time consumed by Worker logic, offload heavy computation to a backend service or a queue-based async pattern, and use Durable Objects or `waitUntil()` for background work that shouldn't block the response
- **Details**: → [extended/checklists.md#edge-architecture-checklist]

### SE-2: KV Eventual Consistency Causing Stale Reads
- **Severity**: high
- **Situation**: An application writes a value to KV and immediately reads it back (potentially from a different edge location), and the read returns the old value because the write hasn't propagated globally yet
- **Cause**: KV is designed for high-read-throughput, globally-distributed access with asynchronous propagation — it deliberately trades strong consistency for that read performance profile
- **Symptoms**:
  - A just-updated config value or feature flag appears to "not have updated" for some users/requests shortly after the change
  - The issue resolves itself after a short delay, consistent with eventual propagation
- **Solution**: Use KV only for data where eventual consistency and occasional staleness are acceptable (config, feature flags, cached data); use Durable Objects or an external database for anything requiring immediate read-after-write consistency

### SE-3: Cache Rule Misconfiguration Serving Stale or Wrong Content
- **Severity**: high
- **Situation**: A content update doesn't appear for users, or worse, one user's personalized/private response gets cached and served to a different user, because cache rules and origin cache-control headers disagree about what should be cached and for how long
- **Cause**: Caching behavior is the product of both dashboard-configured cache rules and origin-sent cache-control headers — when these two configurations conflict or one is left at a permissive default, the actual runtime behavior can differ from what either configuration alone suggests
- **Symptoms**:
  - Stale content persists past an expected TTL, or (more severely) one user's response is served to another
  - A cache purge "fixes" the symptom temporarily, masking the underlying rule misconfiguration
- **Solution**: Set explicit, deliberate cache-control headers at origin (don't rely solely on dashboard defaults), never cache responses containing per-user/session-specific data without an appropriately scoped cache key, and test cache behavior explicitly rather than assuming dashboard settings and origin headers agree

### SE-4: WAF/Firewall Rule Ordering Blocking Legitimate Traffic
- **Severity**: medium
- **Situation**: A broad security rule (e.g., blocking a suspicious user-agent pattern or geographic region) unintentionally blocks legitimate traffic because it's evaluated before a more specific allow rule meant to carve out an exception
- **Cause**: WAF/firewall rules evaluate in a defined priority order — a rule higher in that order can take effect (and stop further evaluation, depending on the action) before a later, more specific rule is ever reached
- **Symptoms**:
  - Legitimate users/partners/integrations report being blocked despite an allow rule that appears to cover their case
  - The allow rule "looks correct" in isolation but never actually executes due to rule ordering
- **Solution**: Explicitly order rules so specific allow exceptions are evaluated before broader blocking rules, and test rule changes against known legitimate traffic patterns before relying on them in production

### SE-5: Durable Object Hot-Instance Bottlenecking
- **Severity**: medium
- **Situation**: A Durable Object handling a high-traffic coordination point (e.g., a globally shared counter or a single popular chat room) becomes a throughput bottleneck, because all requests for that object are serialized through a single instance
- **Cause**: Each Durable Object ID maps to exactly one instance globally, and that instance processes requests for it one at a time — this strong consistency guarantee is exactly what makes it useful for coordination, but it means a single "hot" object can't be horizontally scaled the way stateless Workers can
- **Symptoms**:
  - Latency for a specific Durable Object (or class of them, if IDs aren't well distributed) grows under load while other, less-hot objects remain fast
- **Solution**: Shard hot coordination points across multiple Durable Object instances (e.g., partition a global counter into N sub-counters and aggregate), and design ID schemes to distribute load rather than concentrating it on a small number of IDs

## Recommended Tools

| Category | Tools |
|------|------|
| Development | Wrangler CLI, Miniflare (local Workers emulation) |
| Infrastructure as Code | Terraform Cloudflare provider |
| Monitoring | Workers Analytics Engine, Cloudflare dashboard analytics |
| Security | Cloudflare Access, WAF managed rulesets |

## Edge Architecture

**Full checklist**: → [extended/checklists.md#edge-architecture-checklist]

## Related Resources

[Cloudflare Docs](https://developers.cloudflare.com/) | [Workers Runtime APIs](https://developers.cloudflare.com/workers/runtime-apis/) | [Cloudflare Learning Center](https://www.cloudflare.com/learning/)

## Related Domains

[[vercel-netlify]] | [[aws]] | [[javascript]] | [[terraform]]
