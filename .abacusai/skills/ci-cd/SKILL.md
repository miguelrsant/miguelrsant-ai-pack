---
name: ci-cd
description: Design production-grade CI/CD pipelines with staged quality gates, security scanning, artifact promotion, environment strategy, safe deployments, rollback plans, and release traceability.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# CI/CD

## Purpose

Define the strategy and structure of CI/CD pipelines: stages, quality gates, security scanning, environments, artifact promotion, deployment styles, rollback, and release management. Tool-agnostic — use the `github-actions` skill for GitHub-specific implementation.

## Instructions

### 1. Standard pipeline stages (in order)

1. **Lint & static analysis** — fastest feedback first: formatter check, linter (ruff), type check (mypy), security linter (bandit).
2. **Unit tests** — with coverage report; fail below the agreed threshold.
3. **Integration tests** — against real services (Postgres/Redis as service containers), including `makemigrations --check` for Django.
4. **Security scans** — dependency audit (`pip-audit`/`npm audit`), secret scan (gitleaks), SAST; run in parallel with tests.
5. **Build artifact** — Docker image tagged with the git SHA. **Build ONCE; promote the same digest** through every environment.
6. **Image scan** — Trivy/Scout on the built image; fail on critical CVEs.
7. **Deploy to staging** — automatic on merge to `main`; run migrations as an explicit release step before switching traffic.
8. **Deploy to production** — the SAME artifact, gated by manual approval (environment protection) and/or automated checks.
9. **Post-deploy verification** — smoke tests / health endpoint checks; automatic or one-command rollback on failure.

Principle: cheap and probable failures first; expensive and rare failures last.

### 2. Quality gates (block merge/deploy when failing)

Merge gates (PR → `main`):
- Format + lint + type check clean.
- All tests green; no missing migrations.
- Coverage does not decrease (ratchet) or ≥ the agreed % (see `testing` skill).
- No secrets detected in the diff; no critical vulnerable dependencies introduced.
- Branch protection: green CI + at least one human review required; no direct pushes to `main`.

Deploy gates (staging → production):
- Same artifact that passed CI (digest match — never rebuild per environment).
- Staging smoke tests green.
- Manual approval for production (protected environment), by someone other than the author where policy requires it.

**Decision rule — when a gate may be advisory instead of blocking**: new scanners start advisory (report-only) for a bounded period to burn down findings, then become blocking. A permanently advisory gate is theater — either enforce it or delete it.

### 3. Environment strategy

- `dev` (local) → `staging` (mirrors production: same image, same Postgres major, same topology) → `production`.
- Configuration differs ONLY via environment variables/secrets (Twelve-Factor). If staging needs different code, it is not staging.
- Secrets scoped per environment in the CI provider's secret store; production secrets are never exposed to PR builds, especially from forks.
- Database migrations: explicit release step, backward-compatible with the still-running previous version (`postgresql` skill zero-downtime rules).
- **Decision rule — do you need staging?** Any system with real users: yes. A staging that drifts from production is worse than none — automate its deploy so it never rots.

### 4. Deployment styles and rollback

- **Rolling** (default): replace instances gradually behind a load balancer; requires health checks and graceful shutdown.
- **Blue-green**: two identical environments, switch traffic atomically; simplest and fastest rollback; costs double capacity during the switch.
- **Canary**: shift a % of traffic to the new version; requires real metrics (error rate, latency) to evaluate — do NOT do canary without observability.
- **Rollback is designed BEFORE deploying**:
  - Application: redeploy the previous image tag (kept warm in the registry; retain last N releases).
  - Database: forward-fix migrations, never restoring dumps over live data; this is why migrations must be backward-compatible.
  - Document the exact rollback command in the runbook; rehearse it in staging.
- Feature flags decouple deploy from release: ship dark, enable gradually, kill-switch without redeploying. Prefer a flag over a long-lived feature branch.

### 5. Speed and reliability

- Cache dependencies and Docker layers; parallelize independent jobs (lint ∥ tests ∥ security scans); shard slow test suites.
- Cancel superseded runs on the same branch.
- Flaky tests: quarantine with an issue and a deadline; never normalize "retry until green" — it hides real races.
- Targets: PR feedback < 10 minutes; merge-to-production capability < 1 hour.
- Pipeline is code: versioned, reviewed, and reproducible locally (same commands via Makefile/justfile as in CI).

### 6. Versioning, releases, traceability

- Conventional Commits (see `git-commit` skill) → automated changelog + SemVer tags (`v1.4.2`).
- Production deploys run from tags or from `main` with approval — pick one model per project and document it.
- Every deploy traceable end-to-end: image tag = git SHA; release notes reference commits; deploy events recorded (who, what, when) for audit/compliance.
- Keep a CHANGELOG generated from commits; humans curate only release highlights.

### 7. Observability of the pipeline and deploys

- Notify deploy start/success/failure to the team channel; annotate monitoring dashboards with deploy markers.
- Track DORA-style signals pragmatically: deploy frequency, lead time, change failure rate, time to restore — trends matter more than absolute numbers.
- Post-deploy: watch error rate and latency for the bake period before calling a deploy done.

## Checklists

### Before implementing (new pipeline)
- [ ] Stages mapped to the standard order; merge and deploy gates defined
- [ ] Environments and secret scopes defined; fork-PR secret exposure reviewed
- [ ] Deployment style chosen (rolling/blue-green/canary) with health checks available
- [ ] Rollback procedure written BEFORE the first deploy

### During implementation
- [ ] Build once, promote by digest; image tagged with git SHA
- [ ] Migrations as explicit release step, backward-compatible
- [ ] Security scans wired (deps, secrets, image) with defined severity policy
- [ ] Caching + parallelism + concurrency cancellation configured

### Before delivering
- [ ] Full pipeline green on a real PR and a real merge
- [ ] Staging auto-deploy verified; production gate requires approval
- [ ] Smoke test actually fails when the app is broken (test the test)
- [ ] Rollback rehearsed at least once in staging
- [ ] Branch protection rules enabled and documented

## Best Practices

- Build once, deploy many — never rebuild per environment.
- Small, frequent deploys over large risky ones; keep `main` always deployable.
- Secrets only via the CI provider's secret store; never in YAML, logs, or artifacts.
- Every manual step in the release process is a future outage: automate or document it as an explicit approval.
- Don't add stages you won't enforce; a lean, trusted pipeline beats an elaborate ignored one.

## Triggers

- "CI/CD", "pipeline", "deploy", "esteira", "integração contínua", "entrega contínua", "rollback", "staging", "release", "canário", "blue-green"

## Related Skills

- `github-actions`: implementation on GitHub
- `docker`: artifact format and image scanning
- `testing`: what the pipeline executes and coverage gates
- `git-commit`: commit conventions feeding releases
- `postgresql`: zero-downtime migration rules for release steps
