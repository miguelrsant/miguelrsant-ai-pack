# AGENTS.md

This repository is a personal AI Pack for professional software development, focused on:

- Python
- Django
- Django REST Framework
- PostgreSQL
- Docker
- GitHub Actions
- React
- Vite
- TypeScript
- TailwindCSS
- TDD
- Security
- Technical Documentation

## Standard Workflow

Always follow:

spec → plan → test → implement → review → verify → document

## Communication Language

**User-facing output MUST be in Portuguese** — responses, clarifying questions, progress updates, status messages, summaries, and deliverables (documents, reports, app copy) unless the user explicitly requests another language. English tool outputs, search results, error messages, or documentation are NEVER a reason to reply in English: translate the relevant substance into Portuguese.

Skill documentation, code, identifiers, terminal commands, and tool syntax remain in English.

## Skills Organization

Official skills are located in:

`.abacusai/skills/`

Materials imported or adapted from third parties are located in:

`vendor/ecc-selected/`

### Skill Inventory

#### Professional Skills (v2.0.0)

| Skill | Author | Purpose |
|-------|--------|---------|
| `ci-cd` | Miguel Angelo | Design production-grade CI/CD pipelines with staged quality gates, security scanning, artifact promotion, environment strategy, safe deployments, rollback plans, and release traceability |
| `django` | Miguel Angelo | Build and maintain production-grade Django applications covering architecture, service layers, ORM, migrations, settings, security, observability, and testing, with explicit decision rules for when to apply each pattern |
| `django-rest-framework` | Miguel Angelo | Design and implement production-grade REST APIs with Django REST Framework covering serializers, viewsets, authentication, authorization, versioning, pagination, error contracts, idempotency, throttling, OWASP API security, and OpenAPI documentation |
| `docker` | Miguel Angelo | Containerize applications with Docker and Docker Compose using secure multi-stage builds, non-root runtime, healthchecks, secrets management, image scanning, and clear dev/prod separation |
| `github-actions` | Miguel Angelo | Create and maintain secure, fast GitHub Actions workflows with least-privilege permissions, caching, service containers, environments with approvals, OIDC, reusable workflows, and safe secrets handling |
| `postgresql` | Miguel Angelo | Design, query, index, and operate PostgreSQL in production covering schema modeling, transactions and locking, concurrency, query optimization, zero-downtime migrations, backup/recovery, and operational monitoring |
| `readme-architecture-docs` | Miguel Angelo | Generate and maintain professional README files, C4-style architecture documentation with Mermaid diagrams, ADRs, runbooks, and onboarding docs grounded in the project's actual codebase |
| `testing` | Miguel Angelo | Ensure code quality through automated testing, covering unit, integration, E2E tests and modern testing best practices |

#### Utility Skills (v1.x)

| Skill | Author | Purpose |
|-------|--------|---------|
| `git-commit` | Miguel Angelo | Generate professional Git commit messages following the Conventional Commits specification |
| `skill-creator` | Miguel Angelo | Create new skills for the Abacus AI Agent with proper structure, metadata, and best practices |

#### Specialized Skills

| Skill | Author | Version | Purpose |
|-------|--------|---------|---------|
| `onp-spec-driven` | Vitor Manoel (O Novo Programador) | 3.1.0 | Spec-anchored development with mechanical audit — from spec to audit with full traceability. Includes embedded JavaScript motor for spec-driven workflows |

#### Quick Reference Skills

| Skill | Purpose |
|-------|---------|
| `api-contracts-openapi` | REST contract guidelines and OpenAPI validation checklist |
| `django-drf-production` | Django REST Framework production readiness checklist |
| `production-readiness` | Backend/frontend/devops production readiness checklist |
| `react-vite-tailwind-integration` | React/Vite/Tailwind integration setup checklist |

## Working Standards

- Prefer simple solutions — complexity is a cost.
- Document technical decisions and trade-offs explicitly.
- Use automated tests as a quality gate.
- Avoid undocumented code — documentation is part of deliverables.
- Review security before marking complete.
- Use semantic commits with clear intent.
- Update README when project behavior changes.
- Avoid duplicating skills with the same purpose.
- Use materials in `vendor/ecc-selected/` as reference, not as primary workflow.

## Main Tech Stack

**Backend:**
- Python
- Django
- Django REST Framework
- PostgreSQL
- pytest
- Docker

**Frontend:**
- React
- Vite
- TypeScript
- TailwindCSS

**DevOps:**
- GitHub Actions
- Docker Compose
- Basic CI/CD

## Quality Criteria

A quality deliverable must include:

- Requirement understanding confirmed
- Clear implementation plan
- Simple, maintainable implementation
- Automated tests
- Verification steps
- Documentation
- Semantic commit
- PR checklist

## Development Principles

1. **Test-Driven Development** — Write tests before production code when possible.
2. **Security by Design** — Review security early in the development cycle.
3. **Documentation as Code** — Keep documentation in sync with implementation.
4. **Explicit Decision Rules** — Document when to use each pattern and when not to.
5. **Production Readiness** — Design with observability, monitoring, and operational needs in mind.
6. **Clean Code** — Self-documenting code with clear naming and structure.
