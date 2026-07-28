---
name: api-contracts-openapi
description: Design and maintain REST API contracts with OpenAPI specification, standardized error responses, and predictable endpoints for backend-frontend integration
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# API Contracts and OpenAPI

## Purpose

Ensure REST APIs between Django/DRF backend and React frontend are predictable, documented, and easy to consume. Use this skill when designing new API endpoints, reviewing existing contracts, or validating OpenAPI schemas.

## Instructions

### API Contract Design

1. **Endpoint Naming**: Use consistent, RESTful naming conventions
   - Use kebab-case for URL paths: `/api/users/123`, `/api/orders/123/items`
   - Use plural nouns for collections: `/users`, `/products`
   - Use HTTP verbs correctly: GET (read), POST (create), PUT/PATCH (update), DELETE (remove)

2. **Status Codes**: Return correct HTTP status codes
   - `200 OK` — Successful GET, PUT, PATCH
   - `201 Created` — Successful POST
   - `204 No Content` — Successful DELETE
   - `400 Bad Request` — Validation errors
   - `401 Unauthorized` — Authentication missing/failed
   - `403 Forbidden` — Authorization failed (authenticated but not permitted)
   - `404 Not Found` — Resource not found
   - `409 Conflict` — Business rule violation
   - `422 Unprocessable Entity` — Semantic errors
   - `429 Too Many Requests` — Rate limiting
   - `500 Internal Server Error` — Unexpected errors

3. **Error Response Structure**: Use predictable error format
   ```json
   {
     "detail": "Error message for simple errors",
     "errors": [
       {
         "field": "email",
         "message": "This field is required.",
         "code": "required"
       }
     ],
     "code": "validation_error"
   }
   ```

4. **Pagination**: Define pagination strategy
   - Use `page` and `page_size` query parameters
   - Return `count`, `next`, `previous` in response
   ```json
   {
     "count": 100,
     "next": "http://api.example.com/users/?page=2",
     "previous": null,
     "results": [...]
   }
   ```

5. **Filtering and Sorting**: Standardize query parameters
   - Filters: `?status=active&category=books`
   - Sorting: `?ordering=-created_at,name`
   - Document all available filters in OpenAPI schema

### OpenAPI Documentation

1. **Generate Schema via drf-spectacular**:
   ```python
   # settings.py
   INSTALLED_APPS = [
       'drf_spectacular',
   ]
   REST_FRAMEWORK = {
       'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
   }
   ```

2. **Auto-generate OpenAPI docs**:
   ```bash
   python manage.py spectacular --color --file schema.yml
   ```

3. **Request/Response Types**: Document all payloads in OpenAPI
   - Use `@extend_schema` decorator for custom documentation
   - Define response serializers explicitly
   - Add examples for complex payloads

### Versioning

1. **Avoid Breaking Changes**: Never break existing contracts
   - Add new fields to responses (backward compatible)
   - Never remove fields (deprecate first, document removal)
   - Use versioning in URL: `/api/v1/users`, `/api/v2/users`

2. **Deprecation Strategy**:
   - Add `X-Deprecated: true` header to deprecated endpoints
   - Document migration path in OpenAPI description
   - Maintain deprecated endpoints for at least one major release

### Frontend Integration

1. **Type Generation**: Enable frontend type safety
   - Generate TypeScript types from OpenAPI schema
   - Use `openapi-typescript-codegen` or similar tool
   ```bash
   npx openapi-typescript-codegen -i http://api.example.com/schema.yml -o src/api
   ```

2. **TypeScript Interfaces**: Match frontend types to API responses
   - All requests and responses should be typed
   - DO NOT use `any` — use generated types
   - Maintain sync between API changes and types

## Pre-Flight Checklist

Before considering an API endpoint complete:

- [ ] Endpoint name is clear and RESTful
- [ ] HTTP method is correct (GET/POST/PUT/PATCH/DELETE)
- [ ] Status code is correct for all scenarios
- [ ] Request payload is documented in OpenAPI
- [ ] Response payload is documented in OpenAPI
- [ ] Errors follow standard structure
- [ ] Authentication/permission requirements are explicit
- [ ] Frontend can consume without guessing fields
- [ ] Pagination strategy defined if listing
- [ ] Filter/sort parameters documented
- [ ] No breaking changes to existing contracts
- [ ] OpenAPI schema updated and validated

## Best Practices

1. **Consistency First**: Use the same patterns across all endpoints
2. **Document Everything**: Every field should have a description
3. **Test Contracts**: Write tests that verify OpenAPI schema matches implementation
4. **Version Explicitly**: Assume APIs will evolve, plan for it
5. **Never Assume Types**: Frontend should never `guess` field types
6. **Error Clarity**: Error messages should explain what went wrong and how to fix
7. **Performance**: Design for N+1 queries being the norm, use select_related/prefetch_related

## Examples

### Django Serializer Example

```python
from rest_framework import serializers
from drf_spectacular.utils import extend_schema, OpenApiExample
from .models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'email', 'first_name', 'last_name', 'created_at']
        read_only_fields = ['id', 'created_at']

    def to_representation(self, instance):
        data = super().to_representation(instance)
        data['full_name'] = f"{instance.first_name} {instance.last_name}"
        return data
```

### ViewSet with OpenAPI Documentation

```python
from rest_framework import viewsets
from drf_spectacular.utils import extend_schema, extend_schema_view
from .serializers import UserSerializer

@extend_schema_view(
    list=extend_schema(
        summary="List all users",
        description="Returns paginated list of users with filtering support",
    ),
    retrieve=extend_schema(
        summary="Get user by ID",
        description="Returns detailed information about a specific user",
    ),
)
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filterset_fields = ['is_active']
    ordering_fields = ['created_at', 'email']
```

### Error Response Example

```python
from rest_framework.exceptions import ValidationError
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    if isinstance(exc, ValidationError):
        return Response({
            'code': 'validation_error',
            'errors': exc.detail,
        }, status=400)
    return exception_handler(exc, context)
```

## Triggers

This skill is activated when:

- User mentions "API contract", "OpenAPI", "REST API design"
- Creating new API endpoints
- Reviewing API documentation
- Frontend integration with backend API
- Generating TypeScript types from OpenAPI

## Notes

- Always validate OpenAPI schema with `spectacular lint --file schema.yml`
- Keep frontend types in sync with API changes — automate this as part of CI/CD
- Use API versioning from day one, not as an afterthought
- Document all breaking changes in changelog and migration guide
- When in doubt, prefer explicit documentation over implicit assumptions

## Related Skills

- `django-rest-framework` — Design complete REST APIs with DRF
- `django` — Backend architecture and service layers
- `react-vite-tailwind-integration` — Frontend integration patterns