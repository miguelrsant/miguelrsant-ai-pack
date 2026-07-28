---
name: github-actions
description: Create and maintain secure, fast GitHub Actions workflows with least-privilege permissions, caching, service containers, environments with approvals, OIDC, reusable workflows, and safe secrets handling.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# GitHub Actions

## Purpose

Author GitHub Actions workflows (`.github/workflows/*.yml`): triggers, jobs, permissions, caching, service containers, matrices, artifacts, secrets, environments/approvals, OIDC cloud auth, and Docker build/push. Use the `ci-cd` skill for pipeline strategy; this skill covers GitHub Actions mechanics and security.

## Instructions

### 1. Reference CI workflow (Django + PostgreSQL)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements-dev.txt
      - run: ruff check . && ruff format --check .
      - run: mypy .

  test:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U test"
          --health-interval 5s --health-timeout 3s --health-retries 5
    env:
      DATABASE_URL: postgres://test:test@localhost:5432/test
      DJANGO_SETTINGS_MODULE: config.settings.test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements-dev.txt
      - run: python manage.py makemigrations --check --dry-run
      - run: pytest --cov --cov-report=xml --cov-fail-under=80
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: coverage.xml
```

### 2. Structure and hardening rules

- One workflow per concern: `ci.yml` (lint+test), `deploy.yml`, `release.yml`. Split jobs (`lint`, `test`, `build`) so they parallelize and fail independently; chain with `needs:` when order matters.
- Always set `concurrency` with `cancel-in-progress: true` for PR workflows, and `timeout-minutes` on every job (default 6h is a bill, not a safeguard).
- **Least privilege**: `permissions: contents: read` at workflow level; raise per job only when needed (`packages: write`, `id-token: write`). Never leave the default write-all token.
- Pin actions: major tag (`@v4`) minimum; **pin third-party/non-official actions to a full commit SHA** — tag hijacking is a real supply-chain vector.
- **`pull_request_target` is dangerous**: it runs with secrets against fork code. Avoid it; if unavoidable, never check out the PR head with it. Fork PRs must not receive deploy secrets.
- Never build shell commands by interpolating untrusted input (`${{ github.event.pull_request.title }}` in `run:` = script injection). Pass untrusted values through `env:` and quote them.

### 3. Docker build & push (GHCR)

```yaml
  build:
    needs: [lint, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      - name: Scan image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/${{ github.repository }}:${{ github.sha }}
          severity: CRITICAL
          exit-code: "1"
```

### 4. Deploy workflow with environments

```yaml
  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    environment: production        # enforces reviewers + scoped secrets
    concurrency: production-deploy # never two prod deploys at once
    steps:
      - name: Deploy image ${{ github.sha }}
        run: ./scripts/deploy.sh "${{ github.sha }}"
```

- GitHub **Environments** give manual approval (required reviewers), wait timers, deployment branch rules, and environment-scoped secrets — use them for staging/production instead of ad-hoc `if:` guards alone.
- Add `workflow_dispatch` with an `inputs.sha` for manual redeploy/rollback of a previous image.
- **OIDC over long-lived keys**: for AWS/GCP/Azure, use `id-token: write` + the provider's official auth action (e.g. `aws-actions/configure-aws-credentials` with a role ARN). Never store static cloud keys as secrets when OIDC is available.

### 5. Secrets and variables

- Secrets: `${{ secrets.NAME }}` from repo/org/environment settings. Never commit values, never `echo` them; GitHub masks known secrets in logs but derived values leak — don't transform then print.
- Non-sensitive config: `${{ vars.NAME }}`.
- Scope deploy secrets to environments so PR/CI jobs can't read them.

### 6. Reuse and matrices

- Repeated job sequences → reusable workflow (`on: workflow_call`), called with `uses: ./.github/workflows/tests.yml` and explicit `secrets:`/`inputs:`.
- Repeated steps → composite action in `.github/actions/<name>/action.yml`.
- Matrices for multi-version testing:

```yaml
strategy:
  fail-fast: false
  matrix:
    python-version: ["3.11", "3.12"]
```

### 7. Debugging

- Validate locally: `actionlint` (static check; add it to pre-commit or CI) and `act` for limited local runs.
- Re-run with debug logging: repo secret/variable `ACTIONS_STEP_DEBUG=true`.
- Inspect context safely: `echo '${{ toJSON(github.event_name) }}'` — never dump full `secrets`/`github` contexts.

## Checklists

### Before implementing
- [ ] Workflow concern defined (CI vs deploy vs release) — one file per concern
- [ ] Required permissions listed per job (least privilege)
- [ ] Secrets identified and scoped (repo vs environment); fork-PR exposure considered

### During implementation
- [ ] `concurrency` + `timeout-minutes` set; jobs parallelized with `needs:` where ordered
- [ ] Actions pinned (SHA for third-party); no untrusted input interpolated into `run:`
- [ ] Dependency + Docker layer caching enabled
- [ ] Artifacts uploaded with `if: always()`

### Before delivering
- [ ] `actionlint` clean; workflow green on a real PR
- [ ] Deploy job protected by an environment with required reviewers
- [ ] No secrets printed in any step's logs
- [ ] `workflow_dispatch` rollback path tested
- [ ] Total PR feedback time acceptable (< 10 min target)

## Best Practices

- Keep workflows under ~150 lines; extract reusable workflows beyond that.
- Guard deploy jobs with environments + `if: github.ref == 'refs/heads/main'`.
- Fail fast: lint before tests, cheap before expensive.
- Treat workflow files as production code: reviewed, linted, and minimal.

## Triggers

- "github actions", "workflow", "pipeline", ".github/workflows", "actions yml", "OIDC", "GHCR"

## Related Skills

- `ci-cd`: pipeline strategy, gates, and promotion model
- `docker`: images built and scanned in workflows
- `testing`: what the CI jobs run
