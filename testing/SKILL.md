---
name: testing
description: "Write unit tests, integration tests, and end-to-end tests. Use when implementing test suites, improving coverage, or ensuring code quality through testing."
---

# Testing Skill

Create comprehensive, maintainable test suites to ensure code reliability and quality.

## When to Use

Use this skill when the user wants to:
- Write unit tests for functions and modules
- Create integration tests for API endpoints
- Implement end-to-end tests with E2E frameworks
- Improve test coverage
- Mock external dependencies
- Set up test environments
- Test database operations

## Testing Strategies

### Unit Testing
- Test individual functions and methods in isolation
- Use mocks and stubs for dependencies
- Test edge cases and error conditions
- Aim for high coverage (>80%)

### Integration Testing
- Test component interactions
- Test API endpoint combinations
- Test database operations and migrations
- Verify data flow across modules

### E2E Testing
- Test complete user workflows
- Use frameworks like Playwright, Cypress, or Puppeteer
- Test from user perspective
- Verify visual regression

## Best Practices

- **AAA pattern**: Arrange, Act, Assert
- **Tests should be**: Independent, repeatable, fast, and self-contained
- **Naming**: Describe what is being tested
- **One assertion per test** (when possible)
- **Arrange-Act-Assert** structure

## Frameworks

- **JavaScript**: Jest, Mocha, Vitest
- **Python**: pytest, unittest, nose2
- **Java**: JUnit, TestNG
- **Ruby**: RSpec, Minitest

## Deliverables

- Test suite implementation
- Test configuration
- Coverage report
- Test data setup scripts
- Test utilities

## Quality Checklist

- Tests are meaningful and descriptive
- Edge cases are covered
- Tests are fast and don't block builds
- Setup/teardown is handled
- Mocks are properly isolated
- Tests fail on bugs
