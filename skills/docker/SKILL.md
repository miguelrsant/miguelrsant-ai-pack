---
name: docker
description: Containerize applications with Docker and Docker Compose using secure multi-stage builds, non-root runtime, healthchecks, secrets management, image scanning, and clear dev/prod separation.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# Docker

## Purpose

Create and maintain Dockerfiles and Docker Compose setups at a production standard: multi-stage builds, layer optimization, container security (non-root, read-only, minimal images), healthchecks, secrets handling, dev/prod parity, and image lifecycle (tagging, scanning, size control). Optimized for Python/Django but applicable to any stack.

## Instructions

### 1. Production Dockerfile (Python/Django reference)

```dockerfile
# ---- builder ----
FROM python:3.12-slim AS builder
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1 PIP_DISABLE_PIP_VERSION_CHECK=1
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends build-essential libpq-dev \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --prefix=/install -r requirements.txt

# ---- runtime ----
FROM python:3.12-slim AS runtime
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y --no-install-recommends libpq5 curl \
    && rm -rf /var/lib/apt/lists/* \
    && useradd --create-home --uid 1000 appuser
WORKDIR /app
COPY --from=builder /install /usr/local
COPY --chown=appuser:appuser . .
USER appuser
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=15s --retries=3 \
    CMD curl -fsS http://localhost:8000/health/ || exit 1
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3", "--timeout", "60"]
```

Rules:
- **Multi-stage always**: compilers and build deps never reach the runtime image.
- **Pin the base image** at least to minor (`python:3.12-slim`); pin by digest (`@sha256:...`) for security-critical services. Prefer `-slim`; use `alpine` only when every dependency ships musl wheels; consider distroless for maximum surface reduction when no shell debugging is needed.
- **Layer order = change frequency**: system packages → dependency manifest → dependency install → source code. Copying source before installing deps destroys the cache on every commit.
- **Non-root, fixed UID** — required by hardened orchestrators; `--chown` on COPY so the user can read files without a extra layer.
- One process per container. Migrations run as an explicit release step in the pipeline, NOT in `CMD`/entrypoint (multiple replicas would race).
- Exec-form `CMD` (JSON array) so signals reach the process; ensure the app handles `SIGTERM` for graceful shutdown.
- No `ENV`/`ARG` with real secrets — they persist in image history. Build-time secrets (private index tokens): `RUN --mount=type=secret,...`.

### 2. .dockerignore (always create it)

```
.git
.venv
__pycache__
*.pyc
.env*
!.env.example
node_modules
media/
docs/
tests/
*.md
```

- Never let `.env`, credentials, `.git`, or local artifacts into the build context. A leaked `.env` inside an image is a real incident.
- A lean context also speeds up builds and improves cache hits.

### 3. docker-compose.yml (development)

```yaml
services:
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 3s
      retries: 5
volumes:
  pgdata:
```

- `depends_on.condition: service_healthy` + healthchecks — never sleep hacks.
- Bind DB ports to `127.0.0.1` so dev databases are not exposed on the LAN.
- **Dev vs prod separation**:
  - Dev: bind-mount source for hot reload, `runserver`, weak credentials acceptable locally only.
  - Prod: code baked into the image, no bind mounts, gunicorn/uvicorn, secrets injected by the platform.
  - Use override files: `docker compose -f docker-compose.yml -f docker-compose.prod.yml up`.
- **Decision rule — Compose vs orchestrator**: Compose is fine for local dev and single-host deployments of small apps. Move to a managed platform (ECS, Cloud Run, Kubernetes) when you need multi-host scaling, zero-downtime rollouts, or autoscaling — do NOT introduce Kubernetes for a single small service.

### 4. Container security

- Run as non-root (Dockerfile `USER`) and, where the platform allows, `read_only: true` with explicit `tmpfs` for writable paths, plus `security_opt: ["no-new-privileges:true"]` and dropped capabilities (`cap_drop: [ALL]`).
- **Secrets at runtime**: environment variables injected by the platform/CI secret store, or mounted secret files — never in the image, never in compose files committed to git (`env_file: .env` with `.env` gitignored).
- Scan images in CI (Trivy/Docker Scout/Grype) and fail on critical CVEs; rebuild images regularly to pick up base-image patches — an unrebuilt image rots even without code changes.
- Set resource limits (memory/CPU) in production so one container cannot starve the host.
- Logs to stdout/stderr only (Twelve-Factor); the platform collects them. No log files inside containers.

### 5. Image lifecycle

- Tag with the git SHA (immutable) plus optional semver; **never deploy `latest`** — it is unauditable and unrollbackable.
- Build ONCE in CI, push to a registry, promote the same digest through staging → production (see `ci-cd`).
- Keep images small: check with `docker image ls`; inspect layers with `docker history` / `dive`. Target < ~300MB for a Python web app.
- Retention: keep the last N release images available for instant rollback; garbage-collect the rest.

### 6. Common commands

```bash
docker compose up -d --build
docker compose logs -f web
docker compose exec web python manage.py migrate
docker build -t app:$(git rev-parse --short HEAD) .
docker history app:latest            # inspect layers
trivy image app:latest               # or docker scout cves
```

## Checklists

### Before implementing
- [ ] Multi-stage build planned; base image chosen and pinned
- [ ] What must be writable at runtime identified (tmp, media) — everything else read-only
- [ ] Health endpoint exists in the app for HEALTHCHECK
- [ ] Secrets source defined (platform env/secret store) — nothing baked in

### During implementation
- [ ] Layers ordered by change frequency; dependency install cached
- [ ] `.dockerignore` present; `.env` and `.git` excluded
- [ ] Non-root user with fixed UID; exec-form CMD; SIGTERM handled
- [ ] Healthcheck with sane interval/start-period

### Before delivering
- [ ] Image builds and container runs healthy (`docker compose up` + healthcheck green)
- [ ] Image scanned; no critical CVEs
- [ ] Image size inspected; no build deps in the final stage
- [ ] `docker history` shows no secrets in any layer
- [ ] Tagged with git SHA; `latest` not used for deploys

## Best Practices

- Dev/prod parity: same image family, same Postgres major version, config differs only by environment variables.
- Rebuild base layers on a schedule, not only on code change.
- Prefer BuildKit cache mounts and registry cache (`cache-from`) for fast CI builds.
- Keep the Dockerfile boring and readable; document any non-obvious instruction inline.

## Triggers

- "docker", "dockerfile", "container", "docker compose", "containerizar", "imagem docker", "multi-stage"

## Related Skills

- `django`: the app being containerized
- `postgresql`: DB service in compose
- `github-actions`, `ci-cd`: building, scanning, and promoting images in pipelines
