---
name: testing
description: Ensure code quality through automated testing, covering unit, integration, E2E tests and modern testing best practices.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 1.1.0
---

# Testing

## Purpose

Ensure code quality, reliability, and maintainability through well-structured automated tests, following modern software engineering best practices and patterns.

## Instructions

When asked to work with tests:

1. Analyze the code, feature, or requirement that needs to be tested.
2. Identify the most appropriate type of test (unit, integration, E2E, etc.).
3. Determine the languages, frameworks, and tools available in the project.
4. Make sure the project already has a test setup configured.
5. Identify the critical test scenarios: happy path, error cases, edge cases, and exceptions.
6. Apply best practices: descriptive naming, isolation, appropriate mocks, clear assertions.
7. Run the tests and make sure they pass before finishing.
8. Check test coverage and point out gaps when appropriate.

## Triggers

Keywords and phrases that should trigger this skill:
- "test this"
- "write tests"
- "create tests"
- "unit tests"
- "integration tests"
- "e2e tests"
- "test coverage"
- "testing"
- "tdd"
- "test-driven"

## Best Practices

See the detailed best practices section below for comprehensive guidelines.

## Types of Tests

### Unit Tests

- Test an isolated unit of code (function, class, component).
- Do not access external resources (database, APIs, file system).
- Fast (milliseconds).
- Must be deterministic: same input, same output.
- Use cases: business logic, input validation, data manipulation.

### Integration Tests

- Test the interaction between two or more units.
- May access simulated or real external resources.
- Ensure components work together.
- Use cases: API integration, database, external services.

### E2E (End-to-End) Tests

- Test complete flows from the user's point of view.
- Navigate the UI / perform full requests.
- Slow and fragile, but verify the system as a whole.
- Use cases: checkout flows, authentication, full registration flows.

### Performance Tests

- Verify response time, throughput, resource usage.
- Identify bottlenecks and performance regressions.
- Use cases: slow APIs, heavy queries, batch processing.

### Contract Testing

- Ensures compatibility between services.
- Verifies agreement between provider and consumer.
- Use cases: microservices, integrations between systems.

### Acceptance Testing

- Tests defined acceptance criteria.
- When possible, automated with BDD frameworks.
- Use cases: verification of user requirements.

## Best Practices

### Naming

- Use descriptive names that explain what is being tested and the expected outcome.
- Pattern: "should [behavior] when [condition]"
- Example: "should throw error when password is empty"
- Use domain language (ubiquitous language).
- Avoid generic names like "testSupplier" or "test1".

### Isolation

- Each test must be independent. Execution order must not affect the result.
- Use setup and teardown (beforeEach, afterEach) to prepare the context.
- Prefer creating test data in memory instead of a shared database.
- Use in-memory databases (SQLite, H2) for integration tests when possible.
- Never share state between tests.

### AAA (Arrange-Act-Assert)

- Structure tests in three clear sections:
  - Arrange: prepare the context, data, and mocks
  - Act: execute the action being tested
  - Assert: verify the result
- Separate the sections with comments if the test gets long.

Example in JavaScript:

```javascript
test("should apply discount when customer is premium", () => {
  // Arrange
  const customer = { type: "premium", age: 30 };
  const calculator = new DiscountCalculator();

  // Act
  const discount = calculator.calculate(customer, 100);

  // Assert
  expect(discount).toBe(20);
});
```

### One Assertion Per Test

- Each test should focus on a single behavior.
- Multiple assertions may be acceptable if they test the same concept.
- Tests with many assertions are hard to understand and debug.
- Split complex tests into several smaller tests.

### Test Only Visible Behavior

- Avoid "testing" private implementation (private methods, internal details).
- Focus on public behavior: input and output.
- If something needs to be tested, it should probably be public.
- Testing implementation details produces fragile tests that break on refactoring.

### Avoid Logic in Tests

- Avoid ifs, loops, and complex logic inside tests.
- If you need loops, consider parameterized tests.
- Test logic that fails can hide real bugs.

### Use Realistic Data

- Test data should reflect the real world.
- Avoid always using "foo" or "a" as inputs.
- Use critical valid and invalid cases.
- Consider edge cases: empty strings, null, maximum number, negative, zero.

### Use Mocks and Stubs Correctly

- Mock only slow or unstable external dependencies.
- Avoid mocking the code you are testing.
- Do not assert how many times a method was called unless that is the behavior.
- Prefer fakes or stubs over mocks when appropriate.
- Review the mock setup: if it is complex, the design may be the problem.

### Clear Error Messages

- Use custom messages in assertions when the default is not clear.
- Make the unexpected result explicit.
- Example: "expected the user to be added, but the list remained empty"

### Keep Tests Fast

- Unit tests should run in milliseconds.
- Separate slow tests (integration, E2E) from fast ones.
- Run tests in parallel when possible.
- Consider using factories for efficient test data creation.

### Living Documentation

- Well-written tests are documentation of the expected behavior.
- Anyone reading the tests should understand the system.
- Test business requirements, not just language syntax.

### Snapshot Testing

- Use sparingly: snapshots are useful for stable, serializable output (UI trees, config generation).
- Keep snapshots small and focused; large snapshots get approved without review.
- Never update a failing snapshot without understanding why it changed.
- Prefer explicit assertions when the expected value matters to the business.

### Handling Flaky Tests

- A flaky test is a bug: fix the cause, do not mask it with automatic retries.
- Common causes: shared state, time/date dependence, unawaited async operations, order dependence, real network calls.
- Quarantine flaky tests (skip with a tracking issue) instead of letting them erode trust in the suite.
- Make time and randomness deterministic (fake timers, fixed seeds, injected clocks).

## Test Pyramid

Prefer faster, cheaper tests:

```
        /\
       /E2E\           Few, slow, fragile
      /------\
     /Integration\      Medium
    /--------------\
   /     Unit       \   Many, fast, stable
  /------------------\
```

Ideal: 70% unit, 20% integration, 10% E2E.

## When to Use Parameterized Tests

- Multiple test cases with the same verification logic.
- Testing several valid and invalid inputs.
- Testing edge cases.
- Supporting frameworks: Jest (test.each), pytest (parametrize), JUnit 5 (@ParameterizedTest).

Example in Python (pytest):

```python
@pytest.mark.parametrize("value,expected", [
    ("email@test.com", True),
    ("invalid-email", False),
    ("", False),
    ("@", False),
    ("a@b.com", True),
])
def test_validate_email(value, expected):
    assert validate_email(value) == expected
```

## TDD (Test-Driven Development)

When asked to follow TDD:

1. Red: Write a failing test for the feature you want to implement.
2. Green: Write the minimum code to make the test pass.
3. Refactor: Refactor the code while keeping the tests passing.
4. Repeat until the feature is complete.

Benefits:

- Better design (decoupled, more testable code)
- Living documentation
- Confidence to change
- Fewer bugs

## Test Coverage

- Use coverage as an analysis tool, not as a quality metric.
- High coverage does not guarantee good tests.
- Uncovered lines may indicate dead code or untested cases.
- Do not write tests just to increase the coverage metric.
- Prioritize critical business rules and the highest-risk code.
- Focus on test quality, not just quantity.

## Antipatterns to Avoid

- **Testing frameworks or libraries**: Do not test that React works; test your component.
- **Ordered tests**: Tests that depend on a specific execution order.
- **Flaky tests**: Tests that sometimes pass and sometimes fail without code changes.
- **Magic tests**: Tests with random data and placeholder assertions.
- **Monolithic tests**: Giant tests that test everything at once.
- **Stale/copy-pasted tests**: Tests copied and pasted without understanding whether they still test something real.
- **"It's covered, so it's tested"**: PR review is about the tested behavior, not the number of tests.

## Frameworks and Tools (by language)

### JavaScript/TypeScript

- Jest: complete test runner with mocks, snapshots, and coverage
- Vitest: modern alternative to Jest, integrated with Vite
- Mocha + Chai: flexible, modular
- Cypress: E2E testing
- Playwright: modern, cross-browser E2E testing
- Testing Library: test React components the way users use them

### Python

- pytest: modern and expressive framework
- unittest: standard library framework
- pytest-bdd: BDD support in Python
- unittest.mock: mocking library
- pytest-cov: coverage plugin

### Other Languages

- Java: JUnit 5, TestNG, Mockito
- Go: built-in testing package, testify
- Rust: cargo test, rstest
- .NET: xUnit, NUnit, Moq

## Verification Before Finishing (CI-ready)

When completing any testing task, and before approving a change in CI:

1. Run unit tests and confirm they all pass.
2. Run integration tests.
3. Run linting if available.
4. Check code coverage against the project's minimum (when applicable).
5. Run relevant regression tests.
6. Confirm the tests follow the best practices listed above.
7. Ensure the tests are runnable in CI/CD environments and block the merge if they fail.

Example (GitHub Actions):

- pytest
- npm test
- coverage report

## Language

- Test names must be written in English, keeping consistency with the project.

## Output

Analyze the code or requirement, create the appropriate tests following the best practices, run them, and guide the user on the result. Report problems and suggest improvements when appropriate.

## Notes

- This skill focuses on test quality and best practices, not framework-specific syntax
- Always verify that tests can run in CI/CD environments
- Consider test execution time when designing test suites
- Balance between test coverage and maintainability
- Tests should be treated as first-class production code

## Examples

Example 1: Writing a unit test for a discount calculation function

```javascript
describe("DiscountCalculator", () => {
  describe("calculate", () => {
    test("should apply 20% discount for premium customers", () => {
      // Arrange
      const customer = { type: "premium", age: 30 };
      const calculator = new DiscountCalculator();

      // Act
      const discount = calculator.calculate(customer, 100);

      // Assert
      expect(discount).toBe(20);
    });

    test("should apply 10% discount for regular customers over 60", () => {
      // Arrange
      const customer = { type: "regular", age: 65 };
      const calculator = new DiscountCalculator();

      // Act
      const discount = calculator.calculate(customer, 100);

      // Assert
      expect(discount).toBe(10);
    });

    test("should reject invalid customer types", () => {
      // Arrange
      const customer = { type: "invalid", age: 30 };
      const calculator = new DiscountCalculator();

      // Act & Assert
      expect(() => calculator.calculate(customer, 100)).toThrow(
        "Invalid customer type"
      );
    });
  });
});
```

Example 2: Integration test for user API endpoint

```python
def test_user_registration_flow(client):
    """Test complete user registration with email verification"""
    # Arrange
    user_data = {
        "email": "test@example.com",
        "password": "SecurePass123!",
        "name": "Test User"
    }

    # Act - Register user
    response = client.post("/api/users/register", json=user_data)
    assert response.status_code == 201
    user_id = response.json()["id"]

    # Assert - User exists in database
    db_user = get_user_by_id(user_id)
    assert db_user.email == user_data["email"]
    assert db_user.is_active is False  # Requires verification

    # Act - Verify email
    verification_token = get_verification_token(user_id)
    response = client.post(
        "/api/users/verify", 
        json={"token": verification_token}
    )
    assert response.status_code == 200

    # Assert - User is now active
    db_user = get_user_by_id(user_id)
    assert db_user.is_active is True
```

Example 3: E2E test for checkout flow

```typescript
describe("Checkout Flow", () => {
  test("should complete purchase for guest user", async ({ page }) => {
    // Navigate to product page
    await page.goto("/products/123");
    
    // Add to cart
    await page.click("button[data-testid='add-to-cart']");
    await page.click("button[data-testid='view-cart']");
    
    // Verify cart contents
    await expect(page.locator(".cart-item")).toHaveCount(1);
    await expect(page.locator(".cart-total")).toContainText("$99.99");
    
    // Proceed to checkout
    await page.click("button[data-testid='checkout']");
    await page.fill("input[name='email']", "guest@example.com");
    await page.fill("input[name='address']", "123 Test St");
    await page.click("button[data-testid='place-order']");
    
    // Verify order confirmation
    await expect(page.locator(".order-confirmation")).toBeVisible();
    await expect(page.locator(".order-id")).toMatch(/ORD-\d+/);
  });
});
```

## Related Skills

- `git-commit`: Generate professional Git commit messages for test-related changes
- `skill-creator`: Create new skills following proper structure and best practices
- `onp-spec-driven`: Spec-anchored development workflow with integrated testing
