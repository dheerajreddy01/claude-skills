# Vercel / Netlify — Extended Checklists

## Deployment Safety Checklist

- [ ] Environment variables scoped to the narrowest environment (Production/Preview/Development) that needs them — no "all environments" default for sensitive values
- [ ] Test-mode credentials used for Preview/Development environments wherever the underlying service supports it
- [ ] Preview deployment URLs treated as having a wider exposure surface than production; sensitive data/features gated accordingly
- [ ] Serverless function execution time budgeted against the platform's hard limit; long-running work moved to a background job/queue pattern
- [ ] Edge function code reviewed for runtime API compatibility (no assumptions of full Node.js API access)
- [ ] On-demand revalidation used for content where near-immediate ISR freshness matters, not relying solely on time-based intervals
- [ ] Edge middleware matchers scoped precisely to intended routes, tested to confirm unrelated routes are unaffected

## Cost & Performance Checklist

- [ ] Build-minute and bandwidth usage monitored proactively against plan limits
- [ ] Preview deployment trigger frequency reviewed if build volume is a genuine cost driver
- [ ] Asset sizes and cache-control headers optimized to reduce bandwidth
- [ ] Cold-start latency measured for serverless functions on the critical path; edge functions considered where latency matters and the runtime restriction is acceptable
- [ ] Analytics/Speed Insights reviewed regularly to catch performance regressions, not just at launch
