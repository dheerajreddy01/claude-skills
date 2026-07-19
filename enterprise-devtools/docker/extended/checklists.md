# Docker — Extended Checklists

## Security Checklist

- [ ] No secrets copied or `ARG`'d into any layer — build-time secrets use `--mount=type=secret`, runtime secrets injected via environment/secret manager
- [ ] Final image runs as a non-root `USER`
- [ ] `--privileged` and Docker socket mounts avoided unless explicitly required and documented
- [ ] Unneeded Linux capabilities dropped (`--cap-drop=ALL`, add back only what's needed)
- [ ] Base images pinned to explicit versions (ideally digests), not `latest`
- [ ] Images scanned for known vulnerabilities (Trivy/Grype/Docker Scout) in CI
- [ ] `.dockerignore` excludes `.git`, `.env`, `node_modules`, and other sensitive/unnecessary build context
- [ ] Multi-stage build used so build tools/compilers never ship in the runtime image

## Build & Runtime Hygiene Checklist

- [ ] Dockerfile instructions ordered least-to-most frequently changing, for cache efficiency
- [ ] Package manager caches cleaned within the same `RUN` layer they're created in
- [ ] Explicit `--memory`/`--cpus` limits set for every production container
- [ ] `--init` (or `tini`) used so PID 1 handles signal forwarding and zombie reaping correctly
- [ ] Health checks (`HEALTHCHECK` instruction or orchestrator-level probes) defined for long-running services
- [ ] Compose networks used deliberately — services reference each other by service name, not `localhost`
- [ ] Image size tracked in CI to catch regressions from accidental bloat
