---
name: django-rest-framework
description: Design and implement production-grade REST APIs with Django REST Framework covering serializers, viewsets, authentication, authorization, versioning, pagination, error contracts, idempotency, throttling, OWASP API security, and OpenAPI documentation.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# Django REST Framework

## Purpose

Build and review REST APIs with Django REST Framework at a production standard: API design and versioning, serializers, viewsets/routers, authentication and object-level authorization, consistent error contracts, pagination/filtering, throttling, idempotency, OWASP API Top 10 mitigations, and OpenAPI docs. Use together with the `django` skill for project-level concerns.

## Instructions

### 1. Setup and defaults

```bash
pip install djangorestframework drf-spectacular django-filter djangorestframework-simplejwt django-cors-headers
```

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
        "rest_framework.authentication.SessionAuthentication",  # same-origin browser clients
    ],
    "DEFAULT_PERMISSION_CLASSES": ["rest_framework.permissions.IsAuthenticated"],
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 20,
    "DEFAULT_FILTER_BACKENDS": [
        "django_filters.rest_framework.DjangoFilterBackend",
        "rest_framework.filters.OrderingFilter",
        "rest_framework.filters.SearchFilter",
    ],
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.AnonRateThrottle",
        "rest_framework.throttling.UserRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {"anon": "60/min", "user": "1000/min"},
    "DEFAULT_SCHEMA_CLASS": "drf_spectacular.openapi.AutoSchema",
    "EXCEPTION_HANDLER": "config.api.exceptions.api_exception_handler",
}
```

Secure by default: authentication required, pagination on, throttling on. Open individual endpoints with `AllowAny` explicitly — never the reverse.

### 2. API design rules

- **Resources, not verbs**: `GET /api/v1/orders/`, `POST /api/v1/orders/{id}/cancel/` (domain action via `@action`), never `GET /getOrders`.
- **Versioning**: URL-path versioning (`/api/v1/`) from day one — adding it later breaks every client. Bump the version only for breaking changes (removing/renaming fields, changing semantics). Additive changes (new optional fields/endpoints) do not require a new version. Keep at most two live versions and publish a deprecation window.
- **Contract-first mindset**: the OpenAPI schema is the contract. Breaking the schema = breaking change; CI should diff the generated schema (`drf-spectacular --fail-on-warn`, schema snapshot test).
- **Consistent error contract** (single shape for ALL errors — write a custom exception handler):

```json
{
  "type": "validation_error",
  "errors": [{"code": "required", "detail": "This field is required.", "attr": "email"}]
}
```

  (or adopt RFC 9457 `application/problem+json`; either way, one format everywhere, no raw stack traces, no Django HTML error pages on `/api/`).
- Correct status codes: 200/201/204 success, 400 validation, 401 unauthenticated, 403 forbidden, 404 not found (also for objects the user must not know exist), 409 conflict, 429 throttled.
- **Idempotency**: GET/PUT/DELETE must be idempotent by design. For POST operations with side effects that clients may retry (payments, order creation), accept an `Idempotency-Key` header: store the key + response, return the stored response on replay. Do NOT bother for internal, non-retried endpoints.

### 3. Serializers

- Explicit `fields` list — never `fields = "__all__"` (mass assignment / accidental data exposure, OWASP API3).
- Separate read and write concerns when they diverge: `OrderReadSerializer` vs `OrderWriteSerializer`, or `read_only_fields` for server-controlled fields (`id`, `owner`, `created_at`, `status`).
- Field validation in `validate_<field>`, cross-field in `validate()`. Business rules that touch other tables or services belong in the service layer, not the serializer.
- Never accept the acting user from the payload — inject it in `perform_create`:

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

- Nested writes: keep to one level; for complex creation flows, take flat input and let a service function compose the objects.
- Avoid `SerializerMethodField` doing queries per row (N+1) — prefetch in the queryset and read from prefetched data.

### 4. ViewSets, querysets, authorization

```python
class OrderViewSet(viewsets.ModelViewSet):
    serializer_class = OrderSerializer
    filterset_fields = ["status"]
    ordering_fields = ["created_at", "total"]

    def get_queryset(self):
        return (
            Order.objects.filter(owner=self.request.user)
            .select_related("customer")
            .prefetch_related("items")
        )

    @action(detail=True, methods=["post"])
    def cancel(self, request, pk=None):
        order = self.get_object()
        cancel_order(order=order, actor=request.user)   # service layer
        return Response(OrderSerializer(order).data)
```

- **Broken Object Level Authorization (OWASP API1) is the #1 API bug**: ALWAYS scope `get_queryset` to what the requesting user may access. Object permissions alone don't prevent listing; queryset scoping handles list/detail/update/delete in one place.
- Function-level authorization (OWASP API5): per-action permissions via `get_permissions()` when create/destroy need stronger roles than read.
- Views stay thin: view → serializer (shape/validation) → service (business logic) → model. A view method longer than ~15 lines is a smell.
- **Decision rule — ViewSet vs APIView**: ViewSet+router for resource CRUD; `APIView`/`GenericAPIView` for one-off endpoints (login, webhooks, reports) that don't map to a resource. Don't force everything into routers.

### 5. Pagination and filtering

- Paginate EVERY list endpoint; cap `max_page_size`. Use `CursorPagination` for large or frequently-inserted tables (stable ordering, no deep `OFFSET` cost); `PageNumberPagination` is fine for admin-ish lists.
- Filtering via `django-filter` `FilterSet` classes for anything beyond trivial equality; whitelist `ordering_fields` and `search_fields` explicitly — never let clients order/filter by arbitrary fields (data exposure + unindexed-sort DoS, OWASP API8).

### 6. Authentication and transport security

- SPA/mobile: JWT (`simplejwt`) with short-lived access tokens (≤15 min), refresh rotation and blacklist enabled. Same-origin server-rendered clients: session + CSRF.
- Store tokens client-side responsibly (never in query strings; prefer httpOnly cookies for browser refresh tokens).
- CORS via `django-cors-headers` with an explicit origin allowlist — never `CORS_ALLOW_ALL_ORIGINS = True` with credentials.
- Throttle harder on auth endpoints (`ScopedRateThrottle`, e.g. `"login": "10/min"`) to slow credential stuffing (OWASP API2).
- Webhook receivers: `AllowAny` + mandatory signature validation (HMAC with a shared secret), reject on timestamp skew.
- Never log tokens, passwords, or full request bodies of auth endpoints.

### 7. OpenAPI documentation (drf-spectacular)

```python
urlpatterns += [
    path("api/schema/", SpectacularAPIView.as_view(), name="schema"),
    path("api/docs/", SpectacularSwaggerView.as_view(url_name="schema")),
]
```

- Annotate non-obvious endpoints and custom actions with `@extend_schema` (request/response types, error responses, examples).
- Docs UI: open in dev/staging; in production either require auth or keep only the machine-readable schema, per project policy.
- The schema must generate warning-free; treat schema warnings as CI failures.

### 8. Performance

- Kill N+1 with queryset prefetching; assert with `django-assert-num-queries`/`assertNumQueries` in tests for list endpoints.
- Heavy/slow operations (report generation, bulk imports) → return `202 Accepted` + status endpoint, work in background jobs (see `django` skill decision rules).
- Cache only measured hot read endpoints; prefer short-TTL response caching for anonymous data; send `ETag`/`Last-Modified` where clients benefit.

### 9. Testing

- `APIClient` + pytest + factories. Test matrix for every endpoint:
  1. Unauthenticated → 401
  2. Authenticated but not allowed → 403 (or 404 for other users' objects)
  3. Invalid payload → 400 with the standard error shape
  4. Happy path → correct status + response body
  5. **Cross-tenant check**: user A can never read/modify user B's objects (list and detail)
- Contract tests: snapshot the OpenAPI schema; a diff means a reviewed, intentional contract change.
- Test throttles and permissions where they carry business meaning.

## Checklists

### Before implementing
- [ ] Endpoint modeled as a resource/action; fits the existing URL + version scheme
- [ ] Breaking change? If yes: new version or additive redesign
- [ ] AuthZ model defined: who can list/read/write/delete this resource?

### During implementation
- [ ] `get_queryset` scoped to the requesting user/tenant
- [ ] Explicit serializer `fields`; server-controlled fields read-only
- [ ] Pagination + whitelisted filters/ordering
- [ ] Errors follow the project-wide error contract
- [ ] Business logic in services, not views/serializers

### Before delivering
- [ ] Full endpoint test matrix (401/403/400/2xx/cross-tenant) green
- [ ] No N+1 on list endpoints (query-count assertion)
- [ ] OpenAPI schema regenerates without warnings; examples added for non-obvious payloads
- [ ] Throttling adequate for the endpoint's exposure
- [ ] No secrets/tokens in code, fixtures, or logs

## Best Practices

- Secure defaults globally, relax explicitly per endpoint.
- One error format, one pagination style, one auth story — consistency is the API's UX.
- Document deprecations in the schema (`deprecated=True`) and give clients a migration window.
- Prefer additive evolution over new versions; version bumps are a last resort.

## Triggers

- "DRF", "django rest framework", "API REST", "serializer", "viewset", "endpoint", "JWT", "swagger", "openapi", "versionamento de API", "rate limit"

## Related Skills

- `django`: project structure, service layer, security baseline
- `postgresql`: indexing for API query patterns
- `testing`: test strategy
- `ci-cd`: schema checks and API tests in pipelines
