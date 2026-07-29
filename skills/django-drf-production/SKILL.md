---
name: django-drf-production
description: Quick生产和ready checklist for Django REST Framework builds complementing comprehensive django-rest-framework skill with rapid validation checks
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# Django DRF Production Checklist

## Purpose

Use this skill for rapid validation that Django REST Framework APIs are production-ready. This is a **complement** to the comprehensive `django-rest-framework` skill — use `django-rest-framework` for design and architecture, use this for final validation and checklist.

## IMPORTANT: Relationship to django-rest-framework

- **`django-rest-framework`** (v2.0.0): Comprehensive skill for designing and implementing REST APIs with full architectural guidance, decision rules, and examples
- **`django-drf-production`** (v2.0.0): Quick checklist for validating production readiness after implementation is complete

Use this skill AFTER you have completed implementation using `django-rest-framework`.

## Instructions

### Pre-Implementation Check

1. **Requirement Understanding**
   - [ ] Business requirements are clearly documented
   - [ ] User stories have acceptance criteria
   - [ ] Success metrics are defined
   - [ ] Edge cases and error scenarios are identified

### Architecture & Design

2. **Data Modeling**
   - [ ] Database models are defined with proper relationships
   - [ ] Model indexes are planned for query optimization
   - [ ] Migrations strategy is considered (backward compatibility)
   - [ ] Model methods and properties contain business logic, not views

3. **Serializer Design**
   - [ ] Serializers separate validation/serialization from business logic
   - [ ] Nested serializers are used appropriately (avoid deep nesting)
   - [ ] Read-only fields are properly marked
   - [ ] Custom validators are used for complex validation rules
   - [ ] Serializer contexts are leveraged (request.user, etc.)

4. **ViewSet Design**
   - [ ] Viewsets are thin; business logic is in services
   - [ ] Actions are explicitly defined (list, create, retrieve, update, partial_update, destroy)
   - [ ] Permission classes are defined explicitly
   - [ ] Filter backends are configured appropriately
   - [ ] Throttling classes are defined for public endpoints

### Implementation Checklist

5. **Implementation Order**
   - [ ] Write tests BEFORE implementation (TDD)
   - [ ] Implement models first
   - [ ] Create serializers
   - [ ] Implement views/viewsets
   - [ ] Add URLs and routing
   - [ ] Configure permissions
   - [ ] Add authentication if needed
   - [ ] Implement pagination
   - [ ] Add filtering/sorting
   - [ ] Generate/update OpenAPI schema

### Quality Checks

6. **Query Optimization**
   - [ ] N+1 queries are avoided (use `select_related`, `prefetch_related`)
   - [ ] Query performance verified with django-debug-toolbar
   - [ ] Database indexes are created for frequently filtered fields
   - [ ] Pagination is implemented for list endpoints
   - [ ] Large datasets are handled efficiently (page size limits)

7. **Security**
   - [ ] Authentication is configured (JWT, Session, OAuth)
   - [ ] Authorization is explicit (permission classes)
   - [ ] Input validation is comprehensive (via serializers)
   - [ ] SQL injection prevention verified (use ORM, not raw SQL)
   - [ ] Rate limiting is configured for public endpoints
   - [ ] CORS is configured appropriately
   - [ ] Proper error handling without exposing sensitive data

8. **Testing Coverage**
   - [ ] Model tests defined (CRUD operations)
   - [ ] Serializer tests defined (validation)
   - [ ] Viewset tests defined (HTTP methods, permissions, filters)
   - [ ] Integration tests defined (request-response flow)
   - [ ] Edge cases and error scenarios tested
   - [ ] Coverage meets minimum threshold (80%+ for critical paths)

9. **Documentation**
   - [ ] OpenAPI schema is generated (drf-spectacular)
   - [ ] Request/response examples are provided
   - [ ] Error responses are documented
   - [ ] Authentication/authorization requirements are explicit
   - [ ] README or feature documentation is updated
   - [ ] Migration notes are documented if applicable

### Validation & Verification

10. **Before Final Review**
    - [ ] All tests passing
    - [ ] linting passes (flake8, black)
    - [ ] mypy type-checking passes
    - [ ] No TODO or FIXME in production code
    - [ ] Dead code removed
    - [ ] Code self-review completed
    - [ ] Pull request created for review

## Delivery Standard

When completing a Django/DRF feature, provide:

1. **Summary**: Brief description of changes (1-2 sentences)
2. **Files Modified**: List of files changed with line counts
3. **Migrations Created**: List of new migrations and what they do
4. **Endpoints Changed**: List of added/modified/removed endpoints
5. **Tests Added**: List of new tests and coverage changes
6. **Commands Executed**: Commands used for testing, linting, migration
7. **Risks Identified**: Any known risks, limitations, or points requiring attention
8. **Database Performance**: Any query optimizations or index additions

## Examples of Good Delivery

### Summary

```
Created order management API with CRUD operations, filtering, and pagination.
Optimized queries to avoid N+1 issues. Added comprehensive tests (95% coverage).
```

### Files Modified

```
- models/order.py (45 lines)
- serializers/order.py (68 lines)
- views/order.py (34 lines)
- tests/test_orders.py (156 lines)
- migrations/0003_add_order_model.py (22 lines)
```

### Endpoints Created

```
GET    /api/v1/orders/              - List orders with filtering
POST   /api/v1/orders/              - Create order
GET    /api/v1/orders/{id}/         - Get order by ID
PUT    /api/v1/orders/{id}/         - Update order (full)
PATCH  /api/v1/orders/{id}/         - Update order (partial)
DELETE /api/v1/orders/{id}/         - Delete order
```

### Tests Added

```
- Order model tests (CRUD, relations)
- Order serializer tests (validation, nested fields)
- Order viewset tests (permissions, filters, pagination)
- Integration tests (full request-response cycles)
```

### Commands Executed

```bash
# Testing
pytest --cov=orders tests/ -v

# Linting
flake8 orders/
black orders/

# Type checking
mypy orders/

# Migration
python manage.py makemigrations orders
python manage.py migrate orders
```

### Risks Identified

```
- Order creation may be slow with large products (consider async processing)
- Rate limiting configured but not tested under load
- Database index needed for status filtering (migration includes)
```

## Common Mistakes to Avoid

1. **Business Logic in Views**: ViewSets should orchestrate, not contain complex logic. Use services.
2. **Fat Serializers**: Serializers should validate/serialize, not contain business rules.
3. **N+1 Queries**: Always optimize queries with select_related/prefetch_related.
4. **Skipping Pagination**: List endpoints must paginate to prevent performance issues.
5. **Hardcoded Permissions**: Use permission classes, not if-statements in views.
6. **Forgot API Versioning**: Always design for versioning from the start.
7. **No Error Handling**: Generic error messages frustrate users. Be specific.
8. **Skipping Query Optimization**: "Works locally" doesn't mean works with production data volumes.
9. **Incomplete Tests**: Testing happy paths only is not testing.
10. **No Authorization Review**: Every endpoint must have explicit permission checks.

## Best Practices

1. **Thin Views, Fat Services**: Viewsets orchestrate, services contain logic
2. **Separation of Concerns**: Models → Serializers → Views → Routers (one direction)
3. **Performance First**: Design for production data volumes, not sample data
4. **Security by Default**: Whitelist permissions, not blacklist
5. **Test-Driven Development**: Write tests before implementation
6. **Explicit is Better Than Implicit**: Configure explicitly, don't rely on defaults
7. **Version Early**: API versioning in URLs, not headers
8. **Document Everything**: If it's not documented, it doesn't exist

## Triggers

This skill is activated when:

- User mentions "Django DRF production checklist", "DRF ready for production"
- Completing Django REST Framework feature and need validation
- Reviewing Django REST Framework implementation for production readiness
- After implementing with django-rest-framework skill and need final checks

## Related Skills

- `django-rest-framework` — Comprehensive DRF design and implementation
- `django` — Backend architecture and service layer patterns
- `testing` — Testing strategies and best practices
- `api-contracts-openapi` — API contract design and OpenAPI documentation
- `production-readiness` — Comprehensive production readiness checklist

## Notes

- This skill is for **validation only**, not for design/implementation
- For architectural decisions, use `django-rest-framework` skill
- Always combine with `production-readiness` for complete production validation
- This checklist complements — does not replace — comprehensive testing and review