---
schema: "1.0"
name: nginx
version: "1.0.0"
description: Nginx reverse proxy configuration, load balancing, location block matching, and TLS termination practices
domain: technology
triggers:
  keywords:
    primary: [nginx, reverse proxy, load balancer, location block]
    secondary: [proxy_pass, upstream, ssl termination, apache, haproxy]
  context_boost: [web server config, ingress, traffic routing]
  context_penalty: [istio, kubernetes ingress controller detail, cloudflare]
priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Nginx (Web Server / Reverse Proxy)

> Location block specificity rules decide where a request goes — get the matching order wrong and traffic silently goes to the wrong place

## Applicable Scenarios

- Configuring Nginx as a reverse proxy in front of one or more backend services
- Setting up load balancing across multiple upstream servers
- Terminating TLS/SSL and forwarding plain HTTP internally
- Diagnosing requests being routed to the wrong upstream or losing expected headers
- Tuning for high-concurrency workloads (connection/worker limits, buffering)

## Core Knowledge

### Event-Driven Worker Model

- Nginx uses a small number of **worker processes**, each handling many connections asynchronously via an event loop — fundamentally different from a process/thread-per-connection model (like Apache's traditional `prefork`/`worker` MPMs), which is why Nginx scales to very high concurrent connection counts with modest memory use
- `worker_processes` and `worker_connections` together bound the theoretical max concurrent connections; hitting that ceiling under real load causes connection refusals that can look like an application-level outage

### Reverse Proxy & Load Balancing

```nginx
upstream backend {
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}
server {
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

- `proxy_pass` forwards the request to an upstream; by default, Nginx does **not** forward the original `Host` header or client IP — these must be set explicitly via `proxy_set_header`, and backend applications that rely on them (routing, logging, rate limiting by IP) silently misbehave without them
- Load balancing methods: round-robin (default), `least_conn`, `ip_hash` (session affinity by client IP), weighted (`server ... weight=3`)

### Location Block Matching

- Location blocks are matched by a specific precedence order, **not** simply top-to-bottom: exact match (`=`) first, then longest matching prefix among regular string locations, then regex locations (`~`/`~*`) in the order they appear, then the longest non-regex prefix as fallback
- This ordering is a frequent source of surprise — a broad prefix location and a more specific regex location can interact in ways that don't match the order they're written in the config file

### TLS Termination

- Nginx commonly terminates TLS at the edge and proxies plain HTTP to backend services internally — this simplifies certificate management (one place to renew/configure) but means the internal network segment must itself be trusted, since traffic between Nginx and the backend is unencrypted unless separately configured

## Best Practices

1. **Always set `proxy_set_header Host` and `X-Forwarded-For`/`X-Real-IP`** when proxying — don't assume backends see the original request context
2. **Understand location block precedence** before relying on write-order — test with `nginx -T` to see the fully resolved config, not just what's in one file
3. **Set explicit timeouts** (`proxy_connect_timeout`, `proxy_read_timeout`, client body/header timeouts) — the defaults aren't tuned for every workload and can leave slow-client connections open too long
4. **Tune `worker_connections`/`worker_processes` and OS file descriptor limits together** — one without the other just moves the bottleneck
5. **Rate-limit and cap request body size** (`limit_req`, `client_max_body_size`) at the edge to protect backends from abuse or accidental overload
6. **Test config changes with `nginx -t` before reloading** — a syntax error in a hot-reloaded config can take down routing for everything behind it

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Assuming location blocks match top-to-bottom like a switch statement | Understand the actual precedence order (exact > longest prefix > first-matching regex > prefix fallback); verify with `nginx -T` |
| Omitting `proxy_set_header Host`/`X-Forwarded-For` | Always set these explicitly for any backend that does host-based routing, logging, or IP-based logic |
| Leaving default timeouts in place for a workload with slow clients or long-running requests | Set explicit timeouts matched to actual expected request duration |
| No `client_max_body_size` limit, allowing arbitrarily large uploads to reach the backend | Set an explicit body size cap appropriate to the application |
| Reloading a config change without testing it first | Run `nginx -t` before every `nginx -s reload` |

## Sharp Edges

### SE-1: Location Block Precedence Routing to the Wrong Upstream
- **Severity**: high
- **Situation**: A request that should be handled by a specific, narrowly-scoped location block instead gets matched by a broader or differently-ordered block, silently proxying it to the wrong upstream or applying the wrong headers/rate limits
- **Cause**: Nginx's location matching precedence (exact match, then longest-prefix, then first-matching regex in file order, then prefix fallback) doesn't follow simple top-to-bottom file order, and regex vs. prefix interactions are easy to get wrong without knowing the actual algorithm
- **Symptoms**:
  - A newly added location block "doesn't seem to take effect" even though it's syntactically correct
  - Requests intermittently or consistently reach the wrong backend despite an apparently correct-looking config
- **Solution**: Learn and apply the actual precedence rules explicitly when adding overlapping location blocks, use `nginx -T` to review the fully merged/resolved configuration, and test routing behavior with real request paths after any location block change
- **Details**: → [extended/checklists.md#config-safety-checklist]

### SE-2: Missing `proxy_set_header` Silently Breaking Backend Logic
- **Severity**: high
- **Situation**: A backend application that relies on the `Host` header for routing/multi-tenancy, or on `X-Forwarded-For` for client IP-based logic (rate limiting, geo-blocking, audit logging), gets the wrong value (or none) because Nginx's default `proxy_pass` behavior doesn't forward these headers automatically
- **Cause**: `proxy_pass` forwards the request body and most headers, but `Host` defaults to the upstream's own address unless explicitly overridden, and there's no `X-Forwarded-For` at all unless explicitly set
- **Symptoms**:
  - Multi-tenant routing logic behind the proxy misbehaves or applies the wrong tenant context
  - Rate limiting or audit logs show the proxy's IP instead of the real client IP
- **Solution**: Always explicitly set `proxy_set_header Host $host;` and `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;` (plus `X-Real-IP` if the backend expects it) in any reverse proxy configuration

### SE-3: Buffer Size Misconfiguration Truncating Large Requests/Responses
- **Severity**: medium
- **Situation**: A request with large headers, or a response with a large body, gets truncated or rejected (`502`/`400`-class errors) because Nginx's default proxy buffer sizes are smaller than what the specific workload actually needs
- **Cause**: Nginx buffers proxied responses (and, depending on config, requests) up to configured size limits before either passing them through or spilling to temp files; defaults are conservative and not tuned for every application's payload sizes
- **Symptoms**:
  - Specific endpoints returning large payloads fail intermittently or consistently with proxy-related errors
  - Errors correlate with response/header size rather than with backend logic
- **Solution**: Size `proxy_buffer_size`, `proxy_buffers`, and `large_client_header_buffers` deliberately based on actual observed payload sizes for the specific routes that need it, rather than leaving defaults for a workload they weren't tuned for

### SE-4: `worker_connections`/File Descriptor Limits Causing Silent Connection Refusals Under Load
- **Severity**: high
- **Situation**: Under a real traffic spike, Nginx starts refusing new connections even though the backend services and network have capacity to spare, and the failure looks like an application-level outage rather than a proxy-tier limit
- **Cause**: The maximum concurrent connections Nginx can hold is bounded by `worker_processes × worker_connections`, which is itself bounded by the OS file descriptor limit (`ulimit -n`) available to the Nginx process — hitting either ceiling refuses new connections with no obvious upstream symptom
- **Symptoms**:
  - Connection refused/timeout errors that spike with traffic volume rather than with any backend error rate
  - `worker_connections are not enough` warnings in the Nginx error log
- **Solution**: Size `worker_connections` and the OS file descriptor limit together based on realistic peak concurrency, monitor active connection count as a standard operational metric, and load-test before assuming defaults will hold under production traffic

### SE-5: Slow-Client Resource Exhaustion From Missing Timeout/Rate Limits
- **Severity**: medium
- **Situation**: A client (malicious or simply on a bad connection) that sends request data very slowly, or holds a connection open with minimal activity, ties up a worker connection for an extended period, and enough concurrent slow clients degrade service for everyone else — a slowloris-style resource exhaustion pattern
- **Cause**: Without explicit, reasonably tight timeouts (`client_body_timeout`, `client_header_timeout`, `keepalive_timeout`) and rate limiting (`limit_req`, `limit_conn`), Nginx will hold a connection open as long as the client is technically still communicating within the (permissive) defaults
- **Symptoms**:
  - Service degradation under load with no corresponding backend error rate increase
  - A pattern of many connections open a long time with very low data transfer per connection
- **Solution**: Set explicit, tight timeouts appropriate to normal client behavior, apply `limit_req`/`limit_conn` at the edge for public-facing endpoints, and consider a dedicated DDoS/bot mitigation layer (e.g., Cloudflare) in front of Nginx for internet-facing services at real risk

## Recommended Tools

| Category | Tools |
|------|------|
| Config validation | `nginx -t`, `nginx -T` |
| Load testing | k6, `wrk`, Apache Bench |
| TLS management | Certbot/Let's Encrypt, cert-manager (in Kubernetes) |
| Alternatives/complements | Apache HTTP Server, HAProxy, Envoy |

## Configuration Safety

**Full checklist**: → [extended/checklists.md#config-safety-checklist]

## Related Resources

[Nginx Docs](https://nginx.org/en/docs/) | [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) | [Nginx Pitfalls and Common Mistakes](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/)

## Related Domains

[[istio-linkerd]] | [[docker]] | [[kubernetes]] | [[cloudflare]]
