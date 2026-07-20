# Cloudflare — Extended Checklists

## Edge Architecture Checklist

- [ ] Worker logic profiled for actual CPU time consumption; heavy computation offloaded to a backend service or async pattern
- [ ] `waitUntil()` used for background work that shouldn't block the response
- [ ] Storage primitive matched to consistency need: KV for eventually-consistent reads, Durable Objects for strong consistency, R2 for large objects
- [ ] Durable Object ID scheme reviewed for hot-instance risk; sharding applied where a single coordination point could bottleneck
- [ ] DNS records reviewed for proxied ("orange-cloud") vs. DNS-only status, matched to whether CDN/WAF protection is actually intended
- [ ] Cache-control headers set explicitly at origin, not left to dashboard defaults alone
- [ ] Per-user/session-specific responses never cached without an appropriately scoped cache key

## Security & Access Checklist

- [ ] WAF/firewall rules ordered so specific allow rules take precedence over broader blocking rules where intended
- [ ] Rule changes tested against known legitimate traffic patterns before relying on them in production
- [ ] Cloudflare Access policies reviewed for internal tools, scoped to actual required identity/group membership
- [ ] R2 bucket access scoped appropriately; no unintended public access
- [ ] API tokens scoped to least privilege, not using the global API key for automation
