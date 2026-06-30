# Testing

> Human-readable mirror of [`../rules/05-testing.mdc`](../rules/05-testing.mdc). Cursor reads the `.mdc` file; edit both when updating this rule.


## Purpose

Define how this project tests software — what to test, how to test it, and how AI assistants should approach testing alongside implementation.

Testing is **part of development**, not a phase after feature work. Framework choices may change; these principles stay constant.

## Core Principles

| Principle | Meaning |
| --------- | ------- |
| Testing is part of development | Write or update tests with the feature — not in a follow-up PR |
| Test behavior, not implementation | Assert outcomes and contracts, not internal method calls |
| Confidence over coverage percentage | High-value tests beat arbitrary coverage targets |
| Readable and maintainable | Tests are documentation — clear names, minimal setup |
| Deterministic and independent | No order dependency, no flaky timing, no shared mutable state |

### FIRST Principles

| Principle | Meaning |
| --------- | ------- |
| **F**ast | Tests should run quickly — slow suites don't get run |
| **I**ndependent | Tests must not depend on each other or on execution order |
| **R**epeatable | Same result every run — no environment luck |
| **S**elf-validating | Pass or fail automatically — no manual inspection |
| **T**imely | Written alongside production code — not deferred |

### Scope

Applies to all test code in `apps/` and `packages/`. Coding style for tests follows `03-coding-standards.mdc`.

Keep this rule **framework-agnostic** — tools (test runners, assertion libraries, E2E drivers) may change without rewriting these principles.

## Rules

### Testing Pyramid

Prefer this balance:

```
        ┌─────────────┐
        │     E2E     │  few — critical user journeys only
        ├─────────────┤
        │ Integration │  moderate — module & API boundaries
        ├─────────────┤
        │    Unit     │  majority — business logic, pure functions
        └─────────────┘
```

| Level | Focus | When to use |
| ----- | ----- | ----------- |
| **Unit** | Isolated logic, validators, utilities, domain rules | Default for business logic |
| **Integration** | Components working together — API + DB, service + repository | Boundaries and contracts |
| **End-to-end** | Full user flows through the real stack | Critical paths only (auth, checkout, core learning flow) |

### Test Naming Pattern

Every test name should follow:

```
should <expected behavior> when <condition>
```

When the condition is obvious, `should <expected behavior>` alone is acceptable.

Examples:

```
should return 401 when refresh token is expired
should create a new course when the payload is valid
should reject duplicate email addresses
```

Avoid:

```
creates user
login works
test auth
```

Test reports should read like English.

### Test File Naming

Use descriptive suffixes by test level:

| Pattern | Example |
| ------- | ------- |
| `*.test.ts` | `auth.service.test.ts` |
| `*.integration.test.ts` | `course.controller.integration.test.ts` |
| `*.e2e.test.ts` | `login.e2e.test.ts` |

Colocate tests with the code they cover unless the project establishes a dedicated `__tests__/` convention in that package.

### Unit Testing

- Test **one behavior** per test — one clear assertion theme.
- Follow the **test naming pattern** above.
- Follow **Arrange → Act → Assert** — separate setup, execution, and expectations.
- Avoid testing **private implementation** — test public API and observable behavior.
- **Mock only external dependencies** — network, filesystem, clock, third-party SDKs — not your own domain logic.
- Cover **happy path, edge cases, and error paths** for business rules.

### Integration Testing

- Verify **interactions between components or modules** — not every line of every file.
- Use a **real database where practical** — isolated test database or container; reset between tests.
- **Avoid excessive mocking** — if everything is mocked, it's a unit test with extra steps.
- Test **API request/response contracts**, auth middleware chains, and repository + service flows.
- Keep integration tests **focused** — one scenario per test; shared setup via helpers, not copy-paste.

### End-to-End Testing

- Cover **critical user flows only** — login, registration, core learning path, payment lifecycle (when introduced: success, cancellation, webhooks, refunds).
- Keep scenarios **focused and stable** — avoid brittle selectors and timing-dependent assertions.
- **Avoid duplicate coverage** — don't E2E-test what unit and integration tests already prove.
- Prefer **realistic but minimal** journeys — setup only what the flow requires.

### Regression Tests

Whenever a bug is fixed:

- Add a **regression test** whenever practical.
- Verify the test **fails before the fix** and **passes after the fix**.
- Name the test to describe the bug scenario — future readers should understand what broke.

This prevents the same defect from returning months later.

### Test Data

- Prefer **factories or builders** over hardcoded objects scattered across files.
- Keep test data **minimal** — only fields relevant to the scenario under test.
- Make each test case **self-explanatory** — a reader should understand inputs without hunting fixtures.
- Reset shared state between tests — no leaked data from prior test runs.

### Code Coverage

- Do **not** chase 100% coverage — it encourages low-value tests.
- Focus on **critical business logic**, auth, validation, and data transformations.
- Treat coverage as a **signal**, not a goal — investigate gaps in high-risk areas, ignore boilerplate.

### What to Test

- Business rules and validation logic
- Auth and authorization boundaries
- API contracts and error responses
- Edge cases and failure modes
- Regressions for previously fixed bugs

### What to Skip

- Trivial getters, one-line wrappers, and framework boilerplate
- **Generated or framework-managed code** unless custom behavior has been added
- **Third-party libraries** — don't test that React Router works; test your routing configuration, not the library itself
- Private methods in isolation — test through public behavior
- Snapshot tests unless UI regression is the explicit goal
- Duplicate assertions already covered at a lower pyramid level

## Examples

### ✅ Good test names

```
should return 401 when refresh token is expired
should create a new course when the payload is valid
should reject duplicate email addresses
should calculate course progress as percentage of completed modules
```

### ❌ Avoid test names

```
creates user
login works
test auth
test1
handles error
```

### ✅ Good structure (Arrange → Act → Assert)

```
// Arrange — build input and dependencies
// Act — invoke the behavior under test
// Assert — verify outcome only
```

### ❌ Avoid

```
Testing that a private method was called with specific arguments
Asserting internal state not exposed by the public API
Writing tests that prove a third-party library behaves correctly
```

### ✅ Good integration test focus

```
POST /auth/login returns 200 with valid credentials and sets session cookie
UserRepository persists user and enforces unique email constraint
```

### ❌ Avoid integration test focus

```
Every service method called in sequence with all dependencies mocked
```

### ✅ Good test file names

```
auth.service.test.ts
course.controller.integration.test.ts
login.e2e.test.ts
```

## AI Behavior

When generating or modifying code:

- **Suggest tests** for new business logic, validation, and auth flows.
- **Update existing tests** when behavior changes — don't leave broken or stale tests.
- **Prefer extending an existing test suite** over creating a new file unless separation improves clarity.
- **Avoid brittle tests** tied to implementation details, DOM structure, or exact call order.
- **Explain why a test is valuable** when proposing non-obvious coverage.
- Prefer **one focused test** over a large ambiguous test block.
- Match existing test patterns, helpers, file layout, and naming in the target package.

**Never:**

- Generate tests that only assert mocks were called without verifying behavior.
- Skip tests for critical logic because "we'll add them later".
- Add E2E tests for logic better suited to unit or integration tests.
- Create many tiny test files when an existing suite is the natural home.

## Anti-patterns

- **Testing implementation details** — asserting private methods, internal state, or call order.
- **Massive test files** — hundreds of lines with duplicated setup; split or extract helpers.
- **Flaky tests** — random data, `setTimeout` races, dependence on test execution order.
- **Copy-pasted test data** — same 50-line object in every test; use factories.
- **Unclear test names** — `works`, `test case 3`, `should handle it`.
- **Over-mocking** — mock so much that the test verifies nothing real.
- **Coverage chasing** — tests for getters, setters, and re-exports with no behavioral value.
- **Shared mutable fixtures** — one test mutates data the next test assumes is clean.
- **E2E for everything** — slow, flaky suite that duplicates lower-level coverage.
- **Testing third-party behavior** — verifying library internals instead of your integration with them.

## Exceptions

- **Framework selection** (test runner, E2E tool) — deferred until monorepo tooling is initialized; follow project config once established.
- **Snapshot testing** — acceptable for intentional UI regression suites when documented in the feature or ADR.
- **Advanced testing practices** (mutation testing, property-based testing, contract testing, performance/load testing, accessibility testing) — out of scope for now; revisit when building real features or via a future dedicated rule.
