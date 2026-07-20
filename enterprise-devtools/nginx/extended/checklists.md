# Nginx — Extended Checklists

## Config Safety Checklist

- [ ] `nginx -t` run before every `nginx -s reload` or config deploy
- [ ] `nginx -T` reviewed to see the fully merged/resolved configuration when debugging routing behavior
- [ ] Location block precedence understood explicitly for any config with overlapping prefix and regex locations
- [ ] `proxy_set_header Host` and `X-Forwarded-For`/`X-Real-IP` set on every `proxy_pass` block
- [ ] Explicit timeouts configured (`client_body_timeout`, `client_header_timeout`, `proxy_read_timeout`, `keepalive_timeout`) rather than left at defaults
- [ ] `client_max_body_size` set to an explicit, appropriate cap for each application
- [ ] Buffer sizes (`proxy_buffer_size`, `proxy_buffers`, `large_client_header_buffers`) sized against actual observed payload sizes for large-payload routes

## Capacity & Security Checklist

- [ ] `worker_processes`/`worker_connections` sized against realistic peak concurrency, with OS file descriptor limits raised to match
- [ ] Active connection count monitored as a standard operational metric
- [ ] Rate limiting (`limit_req`) and connection limiting (`limit_conn`) applied to public-facing endpoints
- [ ] Load testing performed against realistic traffic patterns before assuming defaults hold in production
- [ ] TLS certificates managed with automated renewal (Certbot/cert-manager), not manual tracking
- [ ] Internal (post-TLS-termination) traffic segment reviewed for whether it needs its own encryption, given it's plaintext by default after termination
