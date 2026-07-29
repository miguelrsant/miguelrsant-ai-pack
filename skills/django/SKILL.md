---
name: django
description: Build and maintain production-grade Django applications covering architecture, service layers, ORM, migrations, settings, security, observability, and testing, with explicit decision rules for when to apply each pattern.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# Django

## Purpose

Guide the creation, review, and maintenance of Django projects at a Senior/Staff engineering standard: project layout, app boundaries, settings per environment (Twelve-Factor), models/ORM, service layer, migrations, security hardening (OWASP-aligned), structured logging, performance, and testing. Use for the non-API parts of Django; for REST APIs combine with `django-rest-framework`.

## Instructions

### 1. Project bootstrap

1. Standard layout (config module named `config`, domain apps inside `apps/`):

```
project/
├── manage.py
├── config/
│   ├── settings/
│   │   ├── base.py        # everything shared, reads env vars
│   │   ├── dev.py         # DEBUG, debug toolbar, weak email backend
│   │   ├── prod.py        # hardening, real backends
│   │   └── test.py        # fast hasher, in-memory email, migrations optional
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── apps/
│   └── <app_name>/
│       ├── models.py
│       ├── admin.py
│       ├── urls.py
│       ├── views.py
│       ├── forms.py
│       ├── services.py    # writes / business logic (use-case functions)
│       ├── selectors.py   # reads / query logic
│       ├── tasks.py       # background jobs (Celery/RQ), thin wrappers over services
│       ├── migrations/
│       └── tests/
├── pyproject.toml         # or requirements/ with pip-tools; always lock
├── .env.example           # placeholder values ONLY
└── Makefile               # dev commands mirror CI commands
```

2. **Twelve-Factor settings**: all secrets and environment-specific values come from environment variables (`django-environ` or `os.environ`). `base.py` must fail fast if a required variable is missing (`env("SECRET_KEY")` without default). Never hardcode `SECRET_KEY`, DB credentials, or API keys. `.env.example` ships names + placeholders, never values.
3. **Custom user model at project start, always** — retrofitting later requires painful data migrations:

```python
class User(AbstractUser):
    pass

AUTH_USER_MODEL = "users.User"
```

4. Pin Python and Django versions; use only Django LTS or the latest stable release. Lock all dependencies (uv/poetry/pip-tools).

### 2. Architecture and app boundaries

- **One app = one bounded domain concept** (orders, billing, accounts) — not one app per model, not a single `core` app holding everything.
- **Decision rule — when to create a new app**: create one when a concept has its own models, its own lifecycle, and could be explained to a non-developer as a distinct area of the business. Do NOT create a new app for a single helper function, a variation of an existing concept, or "utils".
- Apps depend on each other only through public entry points (`services.py` / `selectors.py`), never by importing another app's internals (forms, private helpers). If two apps import each other, merge them or extract the shared concept.
- **Layering (dependency direction is one-way)**:
  - `views/forms` → shape and validate HTTP input/output; no business rules.
  - `services.py` → use-case functions that mutate state; own `transaction.atomic()`; raise domain exceptions.
  - `selectors.py` → read/query functions returning querysets or DTOs.
  - `models.py` → schema, constraints, trivial properties.
- **Decision rule — when to use a service layer**: use it when a write touches more than one model, has side effects (email, events, payments), enforces business rules, or is invoked from more than one entry point (view + task + management command). Do NOT add a service for a plain single-model CRUD create/update — a ModelForm or serializer `save()` is enough; adding pass-through services is over-engineering.
- **Decision rule — when NOT to reach for DDD/Clean Architecture**: for typical CRUD-heavy Django apps, the layering above IS the pragmatic clean architecture. Introduce richer domain objects (entities/value objects independent of the ORM) only when business rules become complex enough that testing them requires no database. Never add repositories/interfaces "for the future".

### 3. Models and ORM

- Keep models thin: fields, `Meta`, `__str__`, cheap properties. Business logic lives in `services.py`.
- Always define `__str__`; define `Meta.indexes`, `Meta.constraints` (`UniqueConstraint`, `CheckConstraint`) so invariants live in the database, not only in Python. `full_clean()` does not run on `save()` — DB constraints are the real safety net.
- Add `created_at = DateTimeField(auto_now_add=True)` / `updated_at = DateTimeField(auto_now=True)` to every business table (use an abstract `TimeStampedModel`).
- **Decision rule — UUID vs BigAutoField PK**: keep `BigAutoField` (default) as the internal PK. Add a separate indexed `uuid` field (or use UUID PK) only when IDs are exposed to external clients and must be non-guessable/non-enumerable, or when rows are generated across systems and merged. UUIDv4 PKs hurt index locality on large write-heavy tables — do not use them "by default".
- **Decision rule — soft delete**: use a `deleted_at` field + custom manager only when the business requires restore/audit of deleted rows; otherwise real deletes with `on_delete=PROTECT` on critical FKs.
- Query discipline:
  - `select_related` (FK/OneToOne) and `prefetch_related` (M2M/reverse FK) to kill N+1; verify with `django-debug-toolbar` or `assertNumQueries` in tests.
  - `only()`/`defer()` and `values()` for wide tables in hot paths.
  - `F()` expressions for atomic increments (`stock = F("stock") - 1`) — never read-modify-write counters.
  - `bulk_create`/`bulk_update` with `batch_size` for bulk work.
  - `select_for_update()` inside `transaction.atomic()` when two concurrent writers could race on the same row (e.g. balance updates); see `postgresql` skill for locking rules.
- Wrap multi-write operations in `transaction.atomic()` inside the service; side effects that must happen only after commit go in `transaction.on_commit(lambda: task.delay(...))` — never enqueue a task referencing a row that may roll back.

### 4. Migrations

- One logical change per migration; review autogenerated migrations before committing.
- Never edit an applied migration — create a new one. `makemigrations --check` runs in CI.
- Data migrations: separate from schema migrations, use `apps.get_model()` (never import models directly), and write them reversible when feasible.
- Follow the zero-downtime rules from the `postgresql` skill (nullable-first columns, `AddIndexConcurrently`, expand → migrate → contract for renames/drops).
- Name migrations meaningfully: `0007_add_order_status_index`, not `0007_auto_...`.

### 5. Views, URLs, forms

- Class-based views for standard CRUD (`ListView`, `CreateView`...); function-based views for simple or highly custom flows. Do not fight generic CBVs with heavy method overrides — if a CBV needs 4+ overridden methods, write an FBV.
- Namespace URLs per app (`app_name = "orders"`); reverse by name, never hardcode paths.
- Validate ALL input through Forms/ModelForms (or DRF serializers); never trust `request.GET/POST` directly. Validation errors return to the user; unexpected exceptions are logged and become a generic 500 page — never leak stack traces or internal messages in prod.
- Authorization on every view: `LoginRequiredMixin`/`login_required` plus object-level checks (does this user own/can access this object?). Filtering the queryset by user is the primary defense against IDOR — a 404 for someone else's object is correct behavior.
- Paginate every list view; never render unbounded querysets.

### 6. Security (OWASP-aligned, prod)

- `DEBUG = False`; explicit `ALLOWED_HOSTS`; `CSRF_TRUSTED_ORIGINS` when behind a proxy.
- Set `SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`, `SECURE_HSTS_SECONDS` (start low, e.g. 3600, raise after verifying), `SECURE_PROXY_SSL_HEADER` when behind a load balancer, `X_FRAME_OPTIONS = "DENY"`, `SECURE_REFERRER_POLICY`.
- Run `python manage.py check --deploy` in CI for the prod settings module and fix every warning.
- Never disable CSRF middleware or use `csrf_exempt` without a documented reason (webhook endpoints must instead validate signatures).
- Passwords: Django's default hashers (argon2 preferred — `pip install argon2-cffi`); enforce `AUTH_PASSWORD_VALIDATORS`.
- File uploads: validate content type and size, store outside the web root (object storage in prod), never trust user-supplied filenames.
- Rate-limit authentication endpoints (`django-axes` or reverse-proxy limits).
- Static files via WhiteNoise or CDN; media via object storage (`django-storages`) in prod.

### 7. Configuration, logging, observability

- Structured logging: configure `LOGGING` in settings; JSON formatter in prod (`python-json-logger` or structlog), human-readable in dev. Never use `print`.
- Log with context: request id (add a request-id middleware), user id, and the action — but NEVER passwords, tokens, or full card/PII data.
- Error tracking (Sentry or equivalent) wired in prod settings with environment + release tags.
- Health endpoint (`/health/`) that checks the process and, on a deeper variant (`/health/ready/`), DB connectivity — used by Docker healthchecks and deploy verification.
- Expose metrics when the platform supports it (`django-prometheus`), at minimum request latency and error rate.

### 8. Performance and scalability

- **Decision rule — when to use cache**: cache only after measuring — a slow, frequently-read, rarely-changing computation or query. Start with the lowest-risk options: per-view `cache_page` for anonymous pages, template fragment caching, `cache.get_or_set` for expensive lookups. Always define invalidation strategy (TTL first; explicit invalidation only when TTL is unacceptable). Do NOT cache per-user data in shared keys, and do not add Redis to a project whose DB queries are simply missing indexes.
- **Decision rule — when to use background jobs**: move work to Celery/RQ when it is slower than ~500ms, unreliable (external APIs), or not needed for the response (emails, reports, image processing). Tasks must be **idempotent** and take IDs as arguments, never model instances. Enqueue via `transaction.on_commit`. Do NOT add a task queue for work that a cron/management command covers.
- **Decision rule — when to use async views**: only for I/O-bound endpoints that aggregate multiple external calls, under ASGI. The ORM's async support is still limited; a mixed sync/async codebase adds cost. Default to sync + Gunicorn workers; scale horizontally first.
- Serve with `gunicorn` (WSGI) or `uvicorn` workers (ASGI) behind a reverse proxy; never `runserver` in prod. Set worker count from CPU (`2*cores+1` as a start) and a request timeout.
- Set `CONN_MAX_AGE` and use PgBouncer when connection counts grow (see `postgresql`).

### 9. Testing

- `pytest` + `pytest-django` + `factory_boy`. Structure per app: `tests/test_models.py`, `test_services.py`, `test_views.py`.
- Cover: model constraints (attempt the violation, expect `IntegrityError`), service business rules (the bulk of tests — fast, focused), view/endpoint auth + status codes, and permissions (user A cannot see user B's objects).
- Use `test.py` settings: `PASSWORD_HASHERS = ["django.contrib.auth.hashers.MD5PasswordHasher"]`, in-memory email backend.
- Mock external services at the boundary (`responses`/`respx`); never call real third parties in tests.
- See the `testing` skill for strategy, coverage thresholds, and CI wiring.

## Checklists

### Before implementing
- [ ] Does this belong to an existing app or justify a new one (decision rule §2)?
- [ ] Does the change need a service function (multi-model write / side effects) or is plain CRUD enough?
- [ ] Are new invariants expressible as DB constraints?
- [ ] Is the migration zero-downtime safe (`postgresql` skill §5)?

### During implementation
- [ ] Input validated via Form/serializer; authorization checked per object
- [ ] Multi-write operations in `transaction.atomic()`; side effects via `on_commit`
- [ ] N+1 avoided (`select_related`/`prefetch_related`)
- [ ] No secrets or environment-specific values hardcoded
- [ ] Logging with context added on error paths

### Before delivering
- [ ] `pytest` green, including new tests for the change
- [ ] `makemigrations --check` clean; migrations reviewed and named
- [ ] `manage.py check --deploy` clean for prod settings
- [ ] Lint/format (`ruff check`, `ruff format --check`) clean
- [ ] No `print`, no commented-out code, no TODOs without an issue reference

## Best Practices

- Keep apps small and cohesive; expose behavior through `services.py`/`selectors.py`.
- Prefer boring, idiomatic Django over clever abstractions; add patterns only when the pain is real and measured.
- Same commands locally and in CI (Makefile/justfile) — dev/prod parity.
- Every bug fix ships with a regression test.

## Triggers

- "django", "criar app django", "model", "migration", "ORM", "admin", "settings", "service layer", "manage.py"

## Related Skills

- `django-rest-framework`: REST APIs on top of Django
- `postgresql`: schema, locking, and zero-downtime migration rules
- `docker`: containerizing the Django app
- `testing`: test strategy and coverage
