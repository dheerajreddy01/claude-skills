---
schema: "1.0"
name: docker
version: "1.0.0"
description: Docker image builds, layer caching, container security, and Compose orchestration practices
domain: technology
triggers:
  keywords:
    primary: [docker, dockerfile, container, docker-compose, image]
    secondary: [multi-stage build, volume, bind mount, entrypoint, registry]
  context_boost: [containerize, build image, local dev environment]
  context_penalty: [kubernetes, helm, cluster]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Docker

> Small, cache-friendly images and containers that don't quietly run as root

## Applicable Scenarios

- Writing or reviewing Dockerfiles for build efficiency and security
- Designing multi-stage builds to minimize final image size
- Setting up docker-compose for local development or small deployments
- Diagnosing slow builds, bloated images, or container networking issues
- Reviewing container security posture (users, secrets, capabilities)

## Core Knowledge

### Image Layers & Caching

- Each Dockerfile instruction creates a layer; Docker caches layers and reuses them if the instruction and its context haven't changed
- **Order matters**: put instructions that change rarely (installing OS packages, dependency manifests) before instructions that change often (copying application source) — this maximizes cache hits on rebuilds
- `COPY package.json .` + `RUN npm install` + `COPY . .` (in that order) means dependency installs are cached even when only source code changes

### Multi-Stage Builds

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o /app

FROM gcr.io/distroless/static-debian12
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

- A **build stage** contains compilers, build tools, and source — never shipped
- A **final stage** copies only the compiled artifact into a minimal base image (distroless, alpine, or `scratch`)
- This is the standard way to keep production images small and reduce attack surface (no shell, no package manager, no build toolchain in the shipped image)

### Networking & Volumes

| Concept | Detail |
|------|------|
| **Bridge network** | Default for standalone containers; containers reach each other by container/service name, not `localhost` |
| **Volumes** | Docker-managed persistent storage, survives container removal |
| **Bind mounts** | Host filesystem path mapped into the container — convenient for local dev, a security/portability liability in production |
| **docker-compose networks** | Services on the same compose network resolve each other by service name automatically |

A very common beginner confusion: two containers can't reach each other via `localhost` — each container has its own network namespace; use the service/container name instead.

### Container Security Basics

- Containers run as `root` inside the container by default unless the image specifies a `USER` — root inside the container plus certain misconfigurations (e.g., `--privileged`, mounted Docker socket) can lead to host-level compromise
- Secrets baked into an image layer (even if a later layer deletes the file) remain recoverable from the image history — layers are immutable and additive
- `.dockerignore` prevents accidentally copying secrets, `.git`, or `node_modules` into the build context

## Best Practices

1. **Order Dockerfile instructions from least-to-most frequently changing** to maximize layer cache reuse
2. **Use multi-stage builds** — never ship compilers/build tools in the runtime image
3. **Set a non-root `USER`** in the final image unless there's a specific, documented reason not to
4. **Never `COPY` or `ARG` secrets into a layer** — use build secrets (`--mount=type=secret`) or runtime environment/secret injection instead
5. **Pin base image versions** (not `latest`) for reproducible builds
6. **Set resource limits** (`--memory`, `--cpus` or Compose `deploy.resources`) so one container can't starve the host

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `COPY . .` before installing dependencies | Copy dependency manifests first, install, then copy source — preserves cache |
| Baking a secret into an early layer, deleting it in a later one | Use BuildKit secret mounts (`--mount=type=secret`) — never write secrets to any layer |
| Running the container process as root with no `USER` directive | Add a non-root `USER` in the final stage |
| Using `latest` tag for base images in production builds | Pin explicit version tags (and ideally digests) for reproducibility |
| Assuming two containers share `localhost` | Use the container/service name for inter-container communication |

## Sharp Edges

### SE-1: Secrets Persisting in Image Layers
- **Severity**: critical
- **Situation**: A build copies a credentials file or `.env` into the image to use during build, then deletes it in a later `RUN rm` — but the secret is still recoverable by anyone who can pull the image and inspect its layer history
- **Cause**: Docker image layers are immutable and additive; deleting a file in a later layer only hides it from the final filesystem view, it doesn't remove it from the layer that added it
- **Symptoms**:
  - `docker history` or extracting individual layers reveals the "deleted" secret
  - A security scan flags credentials embedded in a shipped image
- **Solution**: Use BuildKit's `--mount=type=secret` (secrets available only during the build step, never written to a layer), or fetch secrets at container runtime instead of build time
- **Details**: → [extended/checklists.md#security-checklist]

### SE-2: Running as Root With Excess Privileges
- **Severity**: high
- **Situation**: A container runs as root by default, and combined with a mounted Docker socket, `--privileged` mode, or a kernel vulnerability, a container compromise escalates to host-level access
- **Cause**: Docker images default to root unless a `USER` is explicitly set; root inside a container has more attack surface than a non-root user, especially when other container isolation is loosened
- **Symptoms**:
  - Security scan flags containers running as UID 0
  - An unrelated container compromise leads to unexpected host access
- **Solution**: Set a non-root `USER` in the final image stage, avoid `--privileged` and Docker socket mounts unless absolutely required, and drop unneeded Linux capabilities (`--cap-drop=ALL` plus only what's needed)

### SE-3: Unbounded Image Bloat From Poor Layer Ordering
- **Severity**: medium
- **Situation**: Every code change triggers a full dependency reinstall during CI builds, and the shipped image is far larger than necessary because build tools and intermediate artifacts never got separated into a discarded build stage
- **Cause**: Copying source before installing dependencies invalidates the dependency-install layer's cache on every change; skipping multi-stage builds means every build tool and cache directory ships in the final image
- **Symptoms**:
  - CI build times climb even for one-line code changes
  - Image size is dominated by build tooling that's never used at runtime
- **Solution**: Reorder Dockerfile instructions (manifests + install before source copy), adopt multi-stage builds, and clean package manager caches within the same layer they were created in (`RUN apt-get install ... && rm -rf /var/lib/apt/lists/*`)

### SE-4: Container Without Resource Limits Starving the Host
- **Severity**: high
- **Situation**: A container with a memory leak or a runaway process consumes all available host memory/CPU, degrading or crashing every other container (and the host itself) sharing that machine
- **Cause**: Containers share the host kernel's resources; without explicit `--memory`/`--cpus` limits (or Kubernetes resource limits in orchestrated environments), one container can consume unbounded resources
- **Symptoms**:
  - Host becomes unresponsive or triggers OOM-killer behavior affecting unrelated containers
  - One noisy container correlates with broad, unrelated service degradation
- **Solution**: Set explicit memory and CPU limits on every container in production (`docker run --memory=512m --cpus=1`, or Compose's `deploy.resources.limits`), and monitor actual usage to set realistic limits rather than guessing

### SE-5: Zombie Processes and Signal Handling as PID 1
- **Severity**: medium
- **Situation**: A container doesn't respond to `docker stop` promptly (hangs until the timeout, then gets SIGKILLed), or accumulates zombie processes over time
- **Cause**: The process running as PID 1 inside a container has special responsibilities in Linux — it must forward signals to child processes and reap zombies; many application runtimes (Node, shell scripts) don't do this by default when run directly as PID 1
- **Symptoms**:
  - `docker stop` always takes the full grace period before force-killing
  - `ps` inside a long-running container shows accumulating `<defunct>` processes
- **Solution**: Use an init process (`docker run --init`, or `tini` in the image) as PID 1 so signals are forwarded correctly and zombies are reaped, rather than running the application directly as PID 1

## Recommended Tools

| Category | Tools |
|------|------|
| Build | BuildKit, Docker Buildx (multi-platform builds) |
| Security scanning | Trivy, Docker Scout, Grype |
| Local orchestration | Docker Compose |
| Image minimization | distroless base images, Alpine |

## Security & Build Hygiene

**Full checklist**: → [extended/checklists.md#security-checklist]

## Related Resources

[Docker Docs](https://docs.docker.com/) | [Dockerfile Best Practices](https://docs.docker.com/build/building/best-practices/) | [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

## Related Domains

[[kubernetes]] | [[aws]] | [[go]] | [[python]]
