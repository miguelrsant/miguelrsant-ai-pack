---
name: production-readiness
description: Production readiness checklist for backend, frontend, and DevOps components before deployment
metadata:
  author: Miguel Angelo
  license: MIT
  version: 2.0.0
---

# Production Readiness

## Purpose

Use this skill before considering any feature, component, or project ready for production deployment. It provides comprehensive checklists for backend, frontend, and DevOps aspects, ensuring quality, security, and operational readiness.

## Instructions

When preparing for production deployment, complete all relevant checklists below. DO NOT deploy unless all applicable items are verified and documented.

---

## Backend Checklist

### Configuration

- [ ] `.env.example` is up-to-date with all required environment variables
- [ ] Environment-specific settings are properly configured (development/staging/production)
- [ ] Secrets are documented but NEVER committed to repository
- [ ] Database connection strings use environment variables
- [ ] Allowed hosts for production are configured
- [ ] CORS settings are restrictive and production-appropriate
- [ ] Debug mode is disabled in production configuration

### Database & Migrations

- [ ] All migrations are generated and reviewed
- [ ] Migrations are tested on staging database copy
- [ ] Zero-downtime migration strategy is considered if needed
- [ ] Backup/restore procedures are documented and tested
- [ ] Database indexes are evaluated for critical queries
- [ ] Transaction boundaries are clearly defined

### Testing

- [ ] All tests passing (`pytest` /manage.py test)
- [ ] Test coverage meets minimum threshold (>80% for critical paths)
- [ ] Integration tests cover API contracts
- [ ] End-to-end tests cover critical user flows
- [ ] Performance tests verify acceptable response times under load

### Security

- [ ] Authentication and authorization are properly configured
- [ ] Input validation is implemented on all endpoints
- [ ] SQL injection prevention is verified (use ORMs, parameterized queries)
- [ ] XSS protection is enabled
- [ ] CSRF tokens are properly configured
- [ ] Rate limiting is implemented for public endpoints
- [ ] Secret scanning is enabled in CI/CD pipeline
- [ ] Dependency vulnerabilities are scanned and addressed

### Monitoring & Observability

- [ ] Structured logging is implemented with appropriate log levels
- [ ] Logs include correlation IDs for request tracing
- [ ] Health check endpoint is available (`/healthz`)
- [ ] Readiness probe is available (`/readyz`)
- [ ] Metrics are exposed (Prometheus format) for monitoring
- [ ] Error tracking is configured (Sentry, Rollbar, or similar)
- [ ] Performance monitoring is active (APM)

### API Documentation

- [ ] OpenAPI schema is up-to-date with all endpoints
- [ ] Request/response examples are provided
- [ ] Error responses are documented with status codes
- [ ] API versioning is implemented if applicable
- [ ] Rate limits are documented

### Code Quality

- [ ] Code passes linting (flake8, black, mypy)
- [ ] No TODO or FIXME comments in production code
- [ ] Code review has been completed and approved
- [ ] Dead code has been removed

---

## Frontend Checklist

### Build & Performance

- [ ] Build process completes without errors
- [ ] Bundle size is optimized (tree-shaking, code-splitting)
- [ ] Lazy loading is used for routes and large components
- [ ] Images are optimized (WebP, compression, responsive loading)
- [ ] Critical CSS is inlined, rest is async
- [ ] Static assets are served with proper cache headers

### Error & Loading States

- [ ] All error states are handled and user-friendly
- [ ] Loading states are shown for async operations
- [ ] Empty states are shown when no data exists
- [ ] Network errors are handled with retry logic
- [ ] Form validation errors are displayed clearly

### Testing

- [ ] All tests passing (Vitest, Jest, or similar)
- [ ] Component tests cover critical UI flows
- [ ] Integration tests cover API interactions
- [ ] E2E tests cover critical user journeys
- [ ] Visual regression tests are configured if applicable

### Type Safety

- [ ] TypeScript is enabled with strict mode
- [ ] No `any` types are used (use `unknown` instead for truly dynamic data)
- [ ] API types match backend OpenAPI schema
- [ ] Props are typed correctly
- [ ] Strict mode violations are resolved

### Accessibility

- [ ] Keyboard navigation works
- [ ] Screen reader compatibility verified
- [ ] Color contrast meets WCAG AA standards
- [ ] Focus indicators are visible
- [ ] ARIA labels are used for interactive elements
- [ ] Form inputs have associated labels

### Environment Configuration

- [ ] Environment variables are documented in `.env.example`
- [ ] API base URLs are environment-specific
- [ ] Feature flags are configurable
- [ ] Debug/development tools are disabled in production build

### Integration

- [ ] API integration points are tested with backend
- [ ] Error handling matches backend error contracts
- [ ] Authentication tokens are stored securely (httpOnly cookies or secure storage)
- [ ] CSRF tokens are included in requests if required
- [ ] Timeout handling is configured for API calls

---

## DevOps Checklist

### Docker

- [ ] Dockerfile follows best practices (multi-stage, non-root user)
- [ ] Docker image builds successfully
- [ ] Image size is optimized (alpine base when possible)
- [ ] Health check is configured in Dockerfile
- [ ] Images are tagged with version numbers, not `latest`
- [ ] Container security scan is configured in CI/CD
- [ ] Docker Compose is configured for local development

### CI/CD Pipeline

- [ ] Pipeline runs successfully on every commit
- [ ] Automated tests are run in pipeline
- [ ] Code quality checks are enforced (lint, format, type-check)
- [ ] Security scans are configured (dependency, secret, container)
- [ ] Automated deployment is configured for staging
- [ ] Canary or blue-green deployment strategy is defined for production
- [ ] Rollback procedure is tested and documented
- [ ] Pipeline has approval gates for production deployments

### GitHub Actions

- [ ] Workflow files are valid and tested
- [ ] Secrets are properly configured (never hardcoded)
- [ ] OIDC is used for cloud authentication (no long-lived credentials)
- [ ] Caching is configured for dependencies
- [ ] Self-hosted runners are used when appropriate
- [ ] Artifact retention policy is configured
- [ ] Rate limits are monitored and configured

### Monitoring & Alerting

- [ ] Application metrics are collected and visualized (Grafana, Datadog)
- [ ] Alerts are configured for critical failures (5xx errors, high error rate)
- [ ] Uptime monitoring is in place
- [ ] Log aggregation is configured (ELK, CloudWatch, etc.)
- [ ] Incident response runbook is documented
- [ ] On-call rotation is established if applicable

### Documentation

- [ ] README is updated with setup and deployment instructions
- [ ] Environment variables are documented
- [ ] Architecture diagrams are up-to-date
- [ ] API documentation is accessible
- [ ] Runbooks for common operations are documented
- [ ] Changelog is maintained for releases

---

## Done Criteria

A feature or project is considered **production-ready** when:

1. **Evidence Required**:
   - Test execution logs showing all tests passing
   - Lint/format/type-check execution logs
   - Build logs showing successful production build
   - Security scan results (no critical/high vulnerabilities)
   - Deployment logs showing successful staging deployment

2. **Documentation Completed**:
   - README updated with new features
   - All environment variables documented
   - API contracts documented in OpenAPI
   - Migration/deployment guide updated
   - Known risks and limitations listed

3. **Verification Steps**:
   - Manual smoke test on staging environment
   - Performance benchmarks meet requirements
   - End-to-end user flows verified
   - Rollback procedure tested
   - Monitoring and alerting verified

4. **Approvals Obtained**:
   - Code review approved
   - Security review completed (if applicable)
   - Product owner verified feature requirements
   - DevOps team approved deployment plan

## Best Practices

1. **Shift Left**: Start production readiness considerations EARLY in development, not right before deployment
2. **Automate**: Automate as many checks as possible in CI/CD pipeline to reduce manual errors
3. **Fail Fast**: Enable strict checks in CI/CD so issues are caught immediately
4. **Document Assumptions**: Make implicit assumptions explicit in documentation
5. **Practice Rollback**: Regularly practice rollbacks, not just deployments
6. **Measure Everything**: You can't improve what you don't measure — collect metrics from day one
7. **Security Default**: Design with security in mind, not as an afterthought

## Common Pitfalls to Avoid

- Deploying to production without staging environment testing
- Assuming "it works on my machine" means it works in production
- Skipping database migration testing
- Hardcoding credentials or environment-specific values
- Neglecting error handling (happy path only)
- Deploying without monitoring and alerting in place
- Forgetting to update documentation alongside code
- Skipping review of generated dependencies (npm/pip freeze)

## Triggers

This skill is activated when:

- User mentions "production-ready", "production readiness", "deploy to production"
- Completing a feature and preparing for deployment
- Reviewing pull request for production suitability
- After completing implementation phase
- Before creating production deployment branch

## Related Skills

- `ci-cd` — Design production-grade CI/CD pipelines
- `django` — Backend architecture and patterns
- `docker` — Containerization and multi-stage builds
- `github-actions` — Secure workflows and automation
- `testing` — Comprehensive testing strategies