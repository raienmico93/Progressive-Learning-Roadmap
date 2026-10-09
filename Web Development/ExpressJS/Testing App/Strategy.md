# Test Strategy — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A test strategy is a comprehensive, risk-based plan that defines which types of tests to write, at which layers of the application, in what proportions, and with what tooling — to maximise confidence in software correctness while minimising the cost of writing and maintaining tests.

**Technical Definition:** A test strategy operationalises the **testing pyramid** — a distribution model that prescribes roughly 70% unit tests, 20% integration tests, and 10% end-to-end tests. It encompasses the selection of test doubles (mocks, stubs, fakes, spies), coverage thresholds enforced in CI, environment configuration (`.env.test`), negative and edge-case testing techniques, regression testing discipline, and snapshot testing for stable JSON structures. The strategy is risk-based, not dogmatic: tests are pushed down the pyramid only where mocks can fully validate logic, and higher-tier tests are added wherever mocks can lie — at database boundaries, API contracts, and critical user workflows.

**Beginner-Friendly Explanation:** A test strategy is your plan for how you will test your application. Instead of writing random tests, you decide: "I'll write many fast tests for individual functions, some slower tests for how pieces work together, and a few very thorough tests for the most important user journeys." You also decide when to use fake objects instead of real ones, how much code coverage you require, and how to handle special cases like invalid input and unexpected boundaries.

### Key Characteristics

- **Risk-based distribution:** More unit tests (fast, cheap), fewer integration tests (medium), fewest E2E tests (slow, expensive).
- **Boundary awareness:** Tests are added at every boundary where mocks can lie — database, network, serialization.
- **Deterministic isolation:** Unit tests mock all dependencies; integration tests use real databases (Testcontainers, in-memory servers).
- **Regression discipline:** Every bug fix adds a test that would have caught it.
- **Negative and edge-case coverage:** Invalid inputs, boundary values, and error paths are tested as thoroughly as happy paths.
- **Coverage gates:** CI enforces minimum thresholds for statements, branches, functions, and lines.
- **Environment isolation:** Tests run with `.env.test` configuration, never touching production data.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **A test runner:** Jest, Vitest, or Node's built-in test runner.
- **Assertion libraries:** Jest's built-in `expect`, or Chai.
- **HTTP testing:** Supertest for integration tests.
- **Database tools:** Testcontainers, `mongodb-memory-server`, or SQLite in-memory.
- **CI/CD pipeline:** GitHub Actions, GitLab CI, or similar for coverage enforcement.
- **Environment management:** `dotenv` or built-in `.env` loading.

### Related Programming Areas

- **Unit Testing:** Isolated tests of functions and services.
- **Integration Testing:** Testing routes, middleware, and databases together.
- **End-to-End Testing:** Full user journeys through the real system (Playwright, Cypress).
- **HTTP Testing:** Supertest assertions on API responses.
- **Test Databases:** Isolated, in-memory, or containerised databases for tests.
- **CI/CD:** Automated test execution and coverage gating.

### Core Concepts

1. **The Test Pyramid** — Unit, integration, and E2E tests in the right proportions.
2. **Regression Tests** — Every bug fix gets a test.
3. **Negative Tests** — Invalid inputs must fail gracefully.
4. **Edge-Case Testing** — Boundary value analysis and off-by-one errors.
5. **Code Coverage Thresholds** — Lines, functions, branches, statements.
6. **Test Doubles Strategy** — Choosing mocks vs. stubs vs. fakes vs. spies.
7. **Environment Configuration** — Managing `.env.test` vs. production variables.
8. **Snapshot Testing** — Capturing large JSON API responses.

---

## Core Concept 1: The Test Pyramid — Unit, Integration, and E2E Tests

### Definitions

**Core Definition:** The test pyramid is a distribution model for automated tests that prescribes a large base of fast unit tests, a smaller middle layer of integration tests, and a small top layer of end-to-end tests.

**Technical Definition:** The pyramid is defined by three layers with distinct characteristics:

| Layer | Ratio | Speed | Confidence | Dependencies | Tools |
|-------|-------|-------|------------|-------------|-------|
| **Unit** | 70% | <10ms each | Low-medium | All mocked | Vitest, Jest |
| **Integration** | 20% | <1s each | Medium-high | Partial real | Vitest, Supertest, Testcontainers |
| **E2E** | 10% | <30s each | High | All real | Playwright, Cypress |

**Beginner-Friendly Explanation:** Imagine you are building a house. You first check every brick individually (unit tests). Then you check that the walls are straight and doors fit (integration tests). Finally, you walk through the finished house and check that everything works together (E2E tests). You do not check every brick by walking through the finished house — that would be slow and expensive. You do not skip walking through the house entirely — that would miss problems that only appear when everything is connected.

### Purposes

- To allocate testing effort proportionally to cost and speed.
- To catch logic errors cheaply with unit tests.
- To catch integration failures (serialization, transactions, contracts) with integration tests.
- To verify critical user workflows with E2E tests.
- To provide fast feedback loops for developers.

### Syntax Rules and Structure

**Unit test (AAA pattern):**
```js
test('calculates tax for US orders', () => {
  // Arrange
  const order = { subtotal: 100, region: 'US-CA' };
  // Act
  const tax = calculateTax(order);
  // Assert
  expect(tax).toBe(7.25);
});
```

**Integration test (Supertest + real database):**
```js
test('POST /users creates a user in the database', async () => {
  const response = await request(app)
    .post('/users')
    .send({ name: 'Alice', email: 'alice@test.com' })
    .expect(201);

  const userInDb = await User.findById(response.body.id);
  expect(userInDb).not.toBeNull();
});
```

**E2E test (Playwright):**
```js
test('user can sign up and see dashboard', async ({ page }) => {
  await page.goto('/signup');
  await page.fill('[name=email]', 'alice@test.com');
  await page.fill('[name=password]', 'SecurePass123!');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL('/dashboard');
});
```

**Rules:**
- Choose the **lowest tier that can fail the way production fails**.
- Unit tests should not cross process boundaries (no real HTTP, no real DB).
- Integration tests should use real databases (Testcontainers, in-memory servers).
- E2E tests should cover only critical user flows (signup, checkout, payment, auth).
- A unit test of a SQL query string proves nothing about whether the query runs; an integration test against a real PostgreSQL does.

### Annotated Code Example

```js
// test/strategy-example.test.js
const request = require('supertest');
const { calculateDiscount } = require('../utils/discount');
const app = require('../app');

// --- UNIT TEST: pure logic, no dependencies ---
describe('calculateDiscount (unit)', () => {
  it('applies 10% discount for orders over $100', () => {
    expect(calculateDiscount(150)).toBe(15);
  });

  it('applies no discount for orders under $100', () => {
    expect(calculateDiscount(50)).toBe(0);
  });

  it('throws for negative amounts', () => {
    expect(() => calculateDiscount(-10)).toThrow('Amount must be positive');
  });
});

// --- INTEGRATION TEST: route + database ---
describe('POST /orders (integration)', () => {
  it('creates an order and persists it', async () => {
    const response = await request(app)
      .post('/orders')
      .send({ items: [{ id: 1, qty: 2 }], total: 150 })
      .expect(201);

    expect(response.body.discount).toBe(15);
    expect(response.body.finalTotal).toBe(135);

    const orderInDb = await Order.findById(response.body.id);
    expect(orderInDb.discount).toBe(15);
  });
});

// --- E2E TEST (conceptual — would use Playwright) ---
describe('Checkout flow (E2E)', () => {
  it('user can add item to cart and complete checkout', async () => {
    // Browser automation: navigate, add item, checkout, verify confirmation
  });
});
```

**Expected Output:**
```
PASS  test/strategy-example.test.js
  calculateDiscount (unit)
    ✓ applies 10% discount for orders over $100 (2 ms)
    ✓ applies no discount for orders under $100 (1 ms)
    ✓ throws for negative amounts (1 ms)
  POST /orders (integration)
    ✓ creates an order and persists it (45 ms)

Tests: 4 passed, 4 total
```

**Why this output:** The unit tests run in milliseconds and test pure logic. The integration test takes longer because it exercises the full Express stack and a real database. Together, they provide high confidence at low cost.

### Real-World Cases

- **Fintech:** Unit-test interest calculations; integration-test transaction persistence; E2E-test a complete transfer.
- **E-commerce:** Unit-test cart totals; integration-test order creation with inventory; E2E-test checkout.
- **SaaS:** Unit-test permission logic; integration-test API endpoints with RBAC; E2E-test signup and onboarding.

---

## Core Concept 2: Regression Tests

### Definitions

**Core Definition:** Regression tests are tests that verify previously working functionality still works after code changes, and that every bug fix is accompanied by a test that would have caught the bug.

**Technical Definition:** Regression testing is the process of re-running previously executed test cases to ensure that recent code changes have not introduced new defects. The discipline is simple: **a bug you fix without a test is a bug you will fix again**. Every bug fix should begin by reproducing the bug as a failing test, then implementing the fix to make the test pass. Regression tests belong in the permanent test suite and are executed on every commit via CI.

**Beginner-Friendly Explanation:** When you fix a bug, you write a test that would have caught it. That test stays in your test suite forever. If someone accidentally reintroduces the bug months later, the test fails immediately.

### Purposes

- To prevent previously fixed bugs from reappearing.
- To ensure refactoring and cleanup do not break existing behaviour.
- To provide a safety net for large code changes.
- To document the exact conditions of past bugs.

### Syntax Rules and Structure

**Regression test naming convention:**
```js
// Include the issue number in the test name
test('rejects negative refund amounts (issue #1423)', () => {
  expect(() => processRefund(-50)).toThrow('Refund amount must be positive');
});
```

**Rules:**
- Every bug fix must include a regression test.
- Name regression tests with the issue number for traceability.
- Reproduce the bug as a failing test **before** writing the fix.
- Regression tests should be part of the permanent test suite.
- Run regression tests on every commit in CI.

### Annotated Code Example

```js
// test/regression/refund.test.js
const { processRefund } = require('../../services/refundService');

describe('Refund service regression tests', () => {
  // Bug #1423: Negative refund amounts were silently accepted
  test('rejects negative refund amounts (issue #1423)', () => {
    expect(() => processRefund(-50)).toThrow('Refund amount must be positive');
  });

  // Bug #1456: Refund of zero was allowed
  test('rejects zero refund amounts (issue #1456)', () => {
    expect(() => processRefund(0)).toThrow('Refund amount must be greater than zero');
  });

  // Bug #1512: Refund exceeding original charge was allowed
  test('rejects refund exceeding original charge (issue #1512)', () => {
    const originalCharge = 100;
    expect(() => processRefund(150, originalCharge))
      .toThrow('Refund cannot exceed original charge');
  });

  // Happy path (existing behaviour, must not regress)
  test('processes valid refund', () => {
    const result = processRefund(50, 100);
    expect(result.success).toBe(true);
    expect(result.amount).toBe(50);
  });
});
```

**Expected Output:**
```
PASS  test/regression/refund.test.js
  Refund service regression tests
    ✓ rejects negative refund amounts (issue #1423) (3 ms)
    ✓ rejects zero refund amounts (issue #1456) (2 ms)
    ✓ rejects refund exceeding original charge (issue #1512) (2 ms)
    ✓ processes valid refund (2 ms)

Tests: 4 passed, 4 total
```

**Why this output:** Each test is named with the issue number, documenting exactly which bug it prevents. If any of these bugs are reintroduced, the corresponding test fails immediately, alerting the developer before the code reaches production.

### Real-World Cases

- **Banking:** Regression test for incorrect interest calculation after a refactor.
- **E-commerce:** Regression test for cart total after a pricing engine change.
- **Healthcare:** Regression test for patient data access after an RBAC update.

---

## Core Concept 3: Negative Tests

### Definitions

**Core Definition:** Negative testing (also called failure testing or error path testing) means feeding a system inputs it was never designed to accept and confirming it handles them without crashing, corrupting data, or returning misleading results.

**Technical Definition:** Negative testing covers the invalid input space using five primary techniques: **equivalence partitioning** (grouping similar inputs), **boundary value analysis** (testing at limits), **error guessing** (based on experience), **fuzz testing** (random invalid data), and **fault injection** (simulating component failures). Negative tests belong in the regression suite because a code change can silently break empty-field validation while the happy path still passes. Industrial data shows that negative cases made up only 29% of one test suite but surfaced 71% of defects found.

**Beginner-Friendly Explanation:** Positive tests check that valid data works. Negative tests check that invalid data fails gracefully. If someone tries to log in with an empty password, your app should show a clear error message, not crash or let them in anyway.

### Purposes

- To confirm the system fails gracefully on invalid input.
- To verify error messages are clear and actionable.
- To ensure invalid data does not corrupt the database.
- To catch security gaps (injection, bypass) before users find them.
- To satisfy the requirement that negative cases belong in the regression suite.

### Syntax Rules and Structure

**Negative test categories:**

| Category | Example Input | Expected Behaviour |
|----------|---------------|-------------------|
| Empty input | `""` | 400 with "Field required" |
| Invalid format | `"not-an-email"` | 422 with "Invalid email" |
| Out-of-range | `-1` for age | 422 with "Age must be positive" |
| SQL injection | `"'; DROP TABLE users; --"` | 422 or sanitised; never executed |
| Type mismatch | `123` for name | 422 with "Name must be a string" |
| Missing field | `{}` | 400 with required field message |

**Rules:**
- Test every validation rule with invalid input.
- Verify the error message is specific and actionable.
- Ensure the system does not leak stack traces or internal details.
- Negative test cases belong in the permanent regression suite.

### Annotated Code Example

```js
// test/negative/validation.test.js
const request = require('supertest');
const app = require('../../app');

describe('User API — Negative tests', () => {
  describe('POST /users — invalid inputs', () => {
    it('rejects empty name', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: '', email: 'alice@test.com' })
        .expect(422);

      expect(response.body.errors).toEqual(
        expect.arrayContaining([
          expect.objectContaining({ field: 'name', message: expect.any(String) })
        ])
      );
    });

    it('rejects invalid email format', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: 'Alice', email: 'not-an-email' })
        .expect(422);

      expect(response.body.errors).toEqual(
        expect.arrayContaining([
          expect.objectContaining({ field: 'email' })
        ])
      );
    });

    it('rejects SQL injection in name', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: "'; DROP TABLE users; --", email: 'alice@test.com' })
        .expect(422);

      // Verify the users table still exists
      const users = await User.find({});
      expect(users).toBeDefined();
    });

    it('rejects missing required fields', async () => {
      const response = await request(app)
        .post('/users')
        .send({})
        .expect(400);

      expect(response.body.error).toBeDefined();
    });

    it('rejects type mismatch (number for name)', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: 12345, email: 'alice@test.com' })
        .expect(422);

      expect(response.body.errors).toEqual(
        expect.arrayContaining([
          expect.objectContaining({ field: 'name' })
        ])
      );
    });

    it('does not leak stack traces in error responses', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: '', email: 'bad' })
        .expect(422);

      expect(response.body.stack).toBeUndefined();
    });
  });
});
```

**Expected Output:**
```
PASS  test/negative/validation.test.js
  User API — Negative tests
    POST /users — invalid inputs
      ✓ rejects empty name (22 ms)
      ✓ rejects invalid email format (18 ms)
      ✓ rejects SQL injection in name (25 ms)
      ✓ rejects missing required fields (12 ms)
      ✓ rejects type mismatch (number for name) (16 ms)
      ✓ does not leak stack traces in error responses (14 ms)

Tests: 6 passed, 6 total
```

**Why this output:** Each test feeds invalid input and verifies the system rejects it with the correct status code and a helpful error message. The SQL injection test verifies that the database table still exists after the attempt. The stack trace test ensures sensitive information is not exposed.

### Real-World Cases

- **Authentication:** Testing empty passwords, SQL injection in usernames, and account lockout.
- **Payment processing:** Testing negative amounts, expired cards, and invalid CVV.
- **File uploads:** Testing oversized files, wrong MIME types, and malicious filenames.

---

## Core Concept 4: Edge-Case Testing

### Definitions

**Core Definition:** Edge-case testing verifies system behaviour at the boundaries of input domains — where off-by-one errors, overflow, and empty-input defects concentrate.

**Technical Definition:** **Boundary Value Analysis (BVA)** is the core technique: design test cases for the minimum, maximum, minimum − 1, and maximum + 1 of each input range. For example, if a field accepts 1–100 characters, test 0, 1, 2, 99, 100, and 101 characters. **Equivalence partitioning** groups similar inputs that should produce the same output, reducing redundant tests. Edge cases are prioritised because critical errors commonly manifest at input boundaries due to precise logic conditions.

**Beginner-Friendly Explanation:** If your form accepts ages 0 to 120, you should test age 0, age 120, age -1, and age 121. The boundaries are where bugs hide — programmers often write "less than" when they mean "less than or equal to."

### Purposes

- To catch off-by-one errors in loops and conditionals.
- To verify handling of empty, null, and undefined inputs.
- To test maximum and minimum values before overflow.
- To ensure date boundaries (leap years, month ends, timezones).
- To reduce redundant tests through equivalence partitioning.

### Syntax Rules and Structure

**Boundary value analysis template:**

| Input Range | Test Values |
|-------------|-------------|
| `1` to `100` characters | `0`, `1`, `2`, `99`, `100`, `101` |
| `0` to `120` years | `-1`, `0`, `1`, `119`, `120`, `121` |
| Array of `1–10` items | `0`, `1`, `2`, `9`, `10`, `11` |
| Positive integer | `0`, `1`, `MAX_SAFE_INTEGER`, `MAX_SAFE_INTEGER + 1` |

**Rules:**
- Test at the boundary, just below, and just above.
- Test empty and null inputs for every string or array field.
- Test maximum values for numeric fields (overflow, precision).
- Use equivalence partitioning to avoid testing every value in a range.

### Annotated Code Example

```js
// test/edge-cases/password.test.js
const { validatePassword } = require('../../utils/password');

describe('Password validation — Edge cases', () => {
  describe('Length boundaries (8–64 characters)', () => {
    it('rejects 7 characters (min − 1)', () => {
      expect(validatePassword('a'.repeat(7))).toBe(false);
    });

    it('accepts exactly 8 characters (min)', () => {
      expect(validatePassword('a'.repeat(8))).toBe(true);
    });

    it('accepts 9 characters (min + 1)', () => {
      expect(validatePassword('a'.repeat(9))).toBe(true);
    });

    it('accepts exactly 64 characters (max)', () => {
      expect(validatePassword('a'.repeat(64))).toBe(true);
    });

    it('rejects 65 characters (max + 1)', () => {
      expect(validatePassword('a'.repeat(65))).toBe(false);
    });
  });

  describe('Character class boundaries', () => {
    it('rejects empty string', () => {
      expect(validatePassword('')).toBe(false);
    });

    it('rejects null', () => {
      expect(validatePassword(null)).toBe(false);
    });

    it('rejects undefined', () => {
      expect(validatePassword(undefined)).toBe(false);
    });

    it('accepts password with all required character types', () => {
      expect(validatePassword('Abcdef1!')).toBe(true);
    });
  });

  describe('Numeric boundaries for age field', () => {
    it('rejects age -1 (min − 1)', () => {
      expect(validateAge(-1)).toBe(false);
    });

    it('accepts age 0 (min)', () => {
      expect(validateAge(0)).toBe(true);
    });

    it('accepts age 120 (max)', () => {
      expect(validateAge(120)).toBe(true);
    });

    it('rejects age 121 (max + 1)', () => {
      expect(validateAge(121)).toBe(false);
    });

    it('rejects age with decimal', () => {
      expect(validateAge(25.5)).toBe(false);
    });
  });
});
```

**Expected Output:**
```
PASS  test/edge-cases/password.test.js
  Password validation — Edge cases
    Length boundaries (8–64 characters)
      ✓ rejects 7 characters (min − 1) (2 ms)
      ✓ accepts exactly 8 characters (min) (1 ms)
      ✓ accepts 9 characters (min + 1) (1 ms)
      ✓ accepts exactly 64 characters (max) (1 ms)
      ✓ rejects 65 characters (max + 1) (1 ms)
    Character class boundaries
      ✓ rejects empty string (1 ms)
      ✓ rejects null (1 ms)
      ✓ rejects undefined (1 ms)
      ✓ accepts password with all required character types (1 ms)
    Numeric boundaries for age field
      ✓ rejects age -1 (min − 1) (1 ms)
      ✓ accepts age 0 (min) (1 ms)
      ✓ accepts age 120 (max) (1 ms)
      ✓ rejects age 121 (max + 1) (1 ms)
      ✓ rejects age with decimal (1 ms)

Tests: 14 passed, 14 total
```

**Why this output:** Each test targets a specific boundary. The password length tests verify the exact minimum (8) and maximum (64), plus one character outside each boundary. The null/undefined/empty tests ensure graceful handling of missing data. The age tests cover negative, zero, maximum, and decimal values.

### Real-World Cases

- **Date handling:** Testing February 28, 29 (leap year), March 1, and December 31.
- **Pagination:** Testing page 0, page 1, last page, and page beyond last.
- **Array operations:** Testing empty arrays, single-element arrays, and maximum-size arrays.
- **String parsing:** Testing empty strings, whitespace-only strings, and maximum-length strings.

---

## Core Concept 5: Code Coverage Thresholds

### Definitions

**Core Definition:** Code coverage thresholds are minimum coverage percentages — for statements, branches, functions, and lines — that a test suite must meet, enforced by the test runner or CI pipeline.

**Technical Definition:** Jest provides a `coverageThreshold` configuration option that sets minimum coverage percentages for the entire project or per-path glob patterns. The four metrics are: **statements** (percentage of executable statements executed), **branches** (percentage of conditional branches taken), **functions** (percentage of functions called), and **lines** (percentage of lines executed). If coverage falls below any threshold, Jest exits with a non-zero code, failing the CI build. Thresholds can also be expressed as negative numbers, meaning "maximum uncovered count" rather than a percentage.

**Beginner-Friendly Explanation:** Coverage thresholds are like a minimum grade you must achieve. You might require 80% of your lines to be tested. If your tests only cover 75%, the build fails and you cannot merge. This ensures that testing keeps pace with development.

### Purposes

- To prevent test coverage from regressing over time.
- To enforce a minimum quality bar for new code.
- To identify untested code paths (especially branches).
- To provide visibility into testing gaps during code review.
- To enforce stricter thresholds for critical modules (e.g., 100% for utilities).

### Syntax Rules and Structure

**Jest configuration (`jest.config.js`):**
```js
module.exports = {
  coverageThreshold: {
    global: {
      statements: 80,
      branches: 70,
      functions: 80,
      lines: 80
    },
    './src/utils/': {
      statements: 100,
      branches: 100,
      functions: 100,
      lines: 100
    },
    './src/services/': {
      statements: 90,
      branches: 80,
      functions: 90,
      lines: 90
    }
  }
};
```

**Negative thresholds (maximum uncovered count):**
```js
coverageThreshold: {
  global: {
    branches: -10  // No more than 10 uncovered branches allowed
  }
}
```

| Metric | Definition | Why It Matters |
|--------|-----------|----------------|
| **Statements** | % of executable statements run | General execution coverage. |
| **Branches** | % of `if`/`else`/`switch` branches taken | Catches untested edge cases. |
| **Functions** | % of functions called | Ensures all functions are exercised. |
| **Lines** | % of lines executed | Similar to statements but line-based. |

**Rules:**
- Start with realistic thresholds (e.g., 50–70%) and raise them gradually.
- Set higher thresholds for critical modules (utilities, services).
- Use branches coverage as the most meaningful metric — it reveals untested edge cases.
- Run coverage in a dedicated CI job, not in every test run.
- Never lower thresholds to make a build pass; fix the coverage gap instead.

### Annotated Code Example

```js
// jest.config.js
module.exports = {
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/**/*.test.js',
    '!src/**/index.js'
  ],
  coverageThreshold: {
    global: {
      statements: 70,
      branches: 60,
      functions: 70,
      lines: 70
    },
    './src/utils/': {
      statements: 100,
      branches: 100,
      functions: 100,
      lines: 100
    }
  }
};
```

```bash
# Run tests with coverage
npx jest --coverage

# Expected output when thresholds are met:
# ------------------------|---------|----------|---------|---------|
# File                    | % Stmts | % Branch | % Funcs | % Lines |
# ------------------------|---------|----------|---------|---------|
# All files               |   85.71 |    75.00 |   83.33 |   85.71 |
#  utils/                 |  100.00 |   100.00 |  100.00 |  100.00 |
#   discount.js           |  100.00 |   100.00 |  100.00 |  100.00 |
# ------------------------|---------|----------|---------|---------|
# Coverage thresholds: PASSED

# Expected output when thresholds are NOT met:
# Coverage threshold for branches (55%) not met: 60%
# Jest exited with code 1
```

**Expected Output (CI log):**
```
PASS  test/utils/discount.test.js
...
Coverage threshold for branches (55%) not met: 60%
npm ERR! Test failed.
```

**Why this output:** The global threshold requires 60% branch coverage. The actual coverage is 55%, so Jest fails. The `utils/` directory requires 100% coverage, ensuring all utility functions — which are pure and easy to test — are fully exercised. When the thresholds are met, Jest exits successfully.

### Real-World Cases

- **Open-source projects:** Enforce coverage thresholds in CI to maintain quality.
- **Enterprise:** Set higher thresholds for security-critical modules (auth, encryption).
- **Team onboarding:** Coverage thresholds prevent new developers from skipping tests.

---

## Core Concept 6: Test Doubles Strategy — Mocks, Stubs, Fakes, and Spies

### Definitions

**Core Definition:** A test double is a replaceable object used in place of a real dependency in tests. There are five types: **dummy**, **stub**, **spy**, **fake**, and **mock**, each with different behaviours and appropriate use cases.

**Technical Definition:**

| Type | Definition | Expectations | Records Calls | Use When |
|------|-----------|-------------|---------------|----------|
| **Dummy** | Placeholder passed but never used | No | No | You need to satisfy a parameter. |
| **Stub** | Returns canned answers | No | No | You need predictable return values. |
| **Spy** | Wraps real object, records calls | No | Yes | You need to verify a method was called. |
| **Fake** | Working implementation (simplified) | No | No | You need a database or service replacement. |
| **Mock** | Pre-programmed with expectations | Yes | Yes | You need to verify exact interactions. |

The **mock hierarchy** recommends using the lightest option: Spies → Stubs → Fakes → Full mocks. **Fakes are preferred** over stubs and mocks for simplicity, because they provide working implementations without requiring a mocking framework.

**Beginner-Friendly Explanation:** When you test a function that depends on a database, you do not want to use the real database. Instead, you use a fake object that behaves like the database but is faster and simpler. The five types of test doubles give you different levels of control — from a simple placeholder (dummy) to a fully scripted actor with expectations (mock).

### Purposes

- To isolate the unit under test from external dependencies.
- To make tests faster and more deterministic.
- To verify that dependencies are called correctly (mocks and spies).
- To provide realistic behaviour without real infrastructure (fakes).
- To return specific values for edge-case testing (stubs).

### Syntax Rules and Structure

**Decision matrix:**

| Need | Test Double | Example |
|------|-------------|---------|
| Predictable return value | Stub | `mockRepo.findById.mockResolvedValue(user)` |
| Verify a method was called | Spy | `jest.spyOn(service, 'validate')` |
| Full behaviour without real DB | Fake | In-memory repository |
| Verify exact interaction order | Mock | `expect(mock.charge).toHaveBeenCalledWith(5000)` |
| Satisfy a parameter | Dummy | `jest.fn()` passed but not called |

**Rules:**
- Use **dependency injection** to make dependencies replaceable.
- Mock at system boundaries (network, database) — not internal implementation details.
- Prefer **fakes** over mocks when possible — they are simpler and more maintainable.
- Use **spies** when you need real behaviour plus call tracking.
- Avoid mocking what you do not own — mock adapters instead.

### Annotated Code Example

```js
// test/test-doubles.test.js
const { OrderService } = require('../services/orderService');

describe('Test doubles strategy', () => {
  let mockPaymentGateway;
  let fakeUserRepository;
  let service;

  beforeEach(() => {
    // --- STUB: predictable return value ---
    mockPaymentGateway = {
      charge: jest.fn().mockResolvedValue({ success: true, id: 'pay_123' })
    };

    // --- FAKE: working in-memory implementation ---
    fakeUserRepository = {
      users: new Map(),
      async findById(id) { return this.users.get(id) || null; },
      async save(user) { this.users.set(user.id, user); return user; }
    };

    service = new OrderService(mockPaymentGateway, fakeUserRepository);
  });

  it('uses a STUB for predictable payment response', async () => {
    const result = await service.checkout({ userId: 1, total: 5000 });
    expect(result.paymentId).toBe('pay_123');
    expect(mockPaymentGateway.charge).toHaveBeenCalledWith(1, 5000);
  });

  it('uses a FAKE for a working user repository', async () => {
    await fakeUserRepository.save({ id: 1, name: 'Alice' });
    const user = await fakeUserRepository.findById(1);
    expect(user.name).toBe('Alice');
  });

  it('uses a SPY to verify internal validation was called', async () => {
    const validateSpy = jest.spyOn(service, 'validateOrder');
    await service.checkout({ userId: 1, total: 5000 });
    expect(validateSpy).toHaveBeenCalledWith({ userId: 1, total: 5000 });
    validateSpy.mockRestore();
  });

  it('uses a MOCK to verify exact interaction', async () => {
    const mockLogger = { info: jest.fn() };
    const loggingService = new OrderService(mockPaymentGateway, fakeUserRepository, mockLogger);
    await loggingService.checkout({ userId: 1, total: 5000 });
    expect(mockLogger.info).toHaveBeenCalledTimes(1);
    expect(mockLogger.info).toHaveBeenCalledWith(
      expect.stringContaining('Order placed')
    );
  });
});
```

**Expected Output:**
```
PASS  test/test-doubles.test.js
  Test doubles strategy
    ✓ uses a STUB for predictable payment response (8 ms)
    ✓ uses a FAKE for a working user repository (3 ms)
    ✓ uses a SPY to verify internal validation was called (5 ms)
    ✓ uses a MOCK to verify exact interaction (4 ms)

Tests: 4 passed, 4 total
```

**Why this output:** Each test demonstrates a different test double. The stub provides a fixed payment response. The fake provides a working in-memory user repository. The spy records calls to the internal validation method. The mock verifies the exact number and arguments of logger calls.

### Real-World Cases

- **Payment processing:** Stub the payment gateway to return success/failure without real charges.
- **Database testing:** Fake the repository with an in-memory Map for fast unit tests.
- **Audit logging:** Spy on the logger to verify compliance without writing real logs.
- **Third-party APIs:** Mock HTTP clients (Axios, Stripe SDK) to simulate responses.

---

## Core Concept 7: Environment Configuration — Managing `.env.test`

### Definitions

**Core Definition:** Environment configuration for testing means providing a dedicated set of environment variables (`.env.test`) that override production values, ensuring tests run against test databases, test API keys, and test configuration.

**Technical Definition:** A `.env.test` file is loaded by the test runner before tests execute, typically via `dotenv` or a Jest `globalSetup` script. It specifies `NODE_ENV=test`, a test database URL, test API keys, and any other environment-specific values. The production `.env` file is never loaded during tests, preventing accidental data mutation. In Jest, `DOTENV_CONFIG_PATH=.env.test` can be set in the test script, or `loadEnvConfig` from `@next/env` can be used for Next.js projects.

**Beginner-Friendly Explanation:** You do not want your tests to accidentally send real emails or charge real credit cards. A `.env.test` file holds fake credentials and points to a test database. Your test runner loads it instead of the production `.env` file, so tests are always safe.

### Purposes

- To isolate test configuration from production configuration.
- To prevent accidental data mutation in production databases.
- To provide deterministic test API keys and secrets.
- To enable CI pipelines to inject test configuration via environment variables.
- To document which environment variables the application requires.

### Syntax Rules and Structure

**`.env.test` example:**
```bash
NODE_ENV=test
DATABASE_URL=postgresql://test:test@localhost:5433/test_db
JWT_SECRET=test-secret-not-used-in-production
STRIPE_SECRET_KEY=sk_test_...
EMAIL_SERVICE_API_KEY=test-email-key
LOG_LEVEL=error
```

**Jest configuration (`jest.config.js`):**
```js
module.exports = {
  setupFiles: ['<rootDir>/test/setup-env.js']
};
```

**`test/setup-env.js`:**
```js
const path = require('path');
require('dotenv').config({
  path: path.resolve(__dirname, '../.env.test')
});
```

**Rules:**
- Never commit `.env.test` with real secrets — use placeholder test values.
- Set `NODE_ENV=test` to trigger test-specific behaviour.
- Point `DATABASE_URL` to a test database, never production.
- Use `DOTENV_CONFIG_PATH=.env.test` in the test script or load it in `globalSetup`.
- In CI, inject environment variables via the pipeline's secret management.

### Annotated Code Example

```js
// jest.config.js
module.exports = {
  setupFiles: ['<rootDir>/test/setup-env.js'],
  globalSetup: '<rootDir>/test/global-setup.js'
};
```

```js
// test/setup-env.js
const path = require('path');
const dotenv = require('dotenv');

// Load .env.test — this runs before every test file
const result = dotenv.config({
  path: path.resolve(__dirname, '..', '.env.test')
});

if (result.error) {
  throw new Error('Could not load .env.test — is the file present?');
}

// Verify we are using the test database
if (!process.env.DATABASE_URL.includes('test')) {
  throw new Error('DATABASE_URL does not appear to be a test database!');
}
```

```js
// test/database.test.js
const request = require('supertest');
const app = require('../app');

describe('Database environment', () => {
  it('uses the test database', () => {
    expect(process.env.NODE_ENV).toBe('test');
    expect(process.env.DATABASE_URL).toContain('test');
  });

  it('does not use production secrets', () => {
    expect(process.env.STRIPE_SECRET_KEY).toMatch(/^sk_test_/);
  });
});
```

**Expected Output:**
```
PASS  test/database.test.js
  Database environment
    ✓ uses the test database (2 ms)
    ✓ does not use production secrets (1 ms)

Tests: 2 passed, 2 total
```

**Why this output:** The `setup-env.js` file loads `.env.test` before any test runs. The guard clause verifies that `DATABASE_URL` contains "test" — if someone accidentally points it at production, the test suite fails immediately. The test verifies that the correct environment variables are loaded.

### Real-World Cases

- **Stripe integration:** Using `sk_test_...` keys in tests to avoid real charges.
- **Email services:** Using a test API key that sends emails to a test inbox.
- **Database isolation:** Pointing `DATABASE_URL` at a Docker container or in-memory database.
- **CI pipelines:** Injecting secrets via GitHub Actions secrets or GitLab CI variables.

---

## Core Concept 8: Snapshot Testing

### Definitions

**Core Definition:** Snapshot testing captures the output of a function or API response and compares it against a saved baseline in future test runs, failing if the output changes unexpectedly.

**Technical Definition:** Jest's `toMatchSnapshot()` records the serialised output of a value — a React component tree, an object, or a JSON API response — into a `__snapshots__` directory. On subsequent runs, Jest compares the current output against the saved snapshot and reports a diff if they differ. Snapshots are ideal for validating complex JSON API responses that have a stable structure. Best practices include: keeping snapshots small (under 50 lines), using `toMatchInlineSnapshot()` for small outputs, avoiding dynamic data (IDs, timestamps), and reviewing snapshot diffs carefully before updating.

**Beginner-Friendly Explanation:** A snapshot test takes a "photo" of what your API response looks like. The next time you run the test, it compares the new response to the photo. If they match, the test passes. If they differ, the test fails and shows you what changed — so you can decide whether the change was intentional.

### Purposes

- To detect unintended changes in large, complex JSON API responses.
- To document the expected shape of an API response without writing dozens of assertions.
- To catch accidental breaking changes in serialised output.
- To reduce the maintenance burden of manually asserting on every field.

### Syntax Rules and Structure

**Basic snapshot test:**
```js
test('GET /users/:id matches snapshot', async () => {
  const response = await request(app).get('/users/usr_123');
  expect(response.status).toBe(200);
  expect(response.body).toMatchSnapshot();
});
```

**Inline snapshot (for small outputs):**
```js
test('formats currency', () => {
  expect(formatCurrency(10.5)).toMatchInlineSnapshot(`"$10.50"`);
});
```

**Property matching (for dynamic fields):**
```js
expect(response.body).toMatchSnapshot({
  id: expect.any(String),
  createdAt: expect.any(String),
  name: 'Alice'
});
```

| Command | Purpose |
|---------|---------|
| `toMatchSnapshot()` | Create/compare an external snapshot file. |
| `toMatchInlineSnapshot()` | Store the snapshot inline in the test file. |
| `jest --updateSnapshot` (`-u`) | Update snapshots after intentional changes. |

**Rules:**
- Commit snapshot files to version control.
- Keep snapshots under 50 lines; break larger ones into focused assertions.
- Do not snapshot dynamic data (random IDs, dates, timestamps) — use `expect.any()`.
- Review the diff carefully before running `-u` to update.
- Use snapshots for stable JSON API responses, not for frequently changing output.

### Annotated Code Example

```js
// test/snapshot/api-response.test.js
const request = require('supertest');
const app = require('../../app');

describe('API response snapshots', () => {
  it('GET /api/users/:id matches snapshot with dynamic fields handled', async () => {
    const response = await request(app)
      .get('/api/users/1')
      .expect(200);

    // Handle dynamic fields with property matchers
    expect(response.body).toMatchSnapshot({
      id: expect.any(String),
      createdAt: expect.any(String),
      updatedAt: expect.any(String)
    });
  });

  it('GET /api/products matches snapshot for list shape', async () => {
    const response = await request(app)
      .get('/api/products')
      .expect(200);

    // Snapshot only the first product to keep the snapshot small
    expect(response.body.data[0]).toMatchSnapshot({
      id: expect.any(String),
      createdAt: expect.any(String)
    });
  });

  it('error response matches RFC 7807 snapshot', async () => {
    const response = await request(app)
      .get('/api/users/999999')
      .expect(404);

    expect(response.body).toMatchSnapshot({
      instance: expect.any(String)
    });
  });
});
```

**Expected Output (snapshot file `__snapshots__/api-response.test.js.snap`):**
```
exports[`API response snapshots GET /api/users/:id matches snapshot with dynamic fields handled 1`] = `
Object {
  "createdAt": Any<String>,
  "email": "alice@example.com",
  "id": Any<String>,
  "name": "Alice",
  "roles": Array [
    "admin",
    "user",
  ],
  "updatedAt": Any<String>,
}
`;

exports[`API response snapshots GET /api/products matches snapshot for list shape 1`] = `
Object {
  "createdAt": Any<String>,
  "id": Any<String>,
  "name": "Laptop",
  "price": 1200,
}
`;

exports[`API response snapshots error response matches RFC 7807 snapshot 1`] = `
Object {
  "detail": "No user exists with the specified ID",
  "instance": Any<String>,
  "status": 404,
  "title": "User Not Found",
  "type": "https://api.example.com/problems/user-not-found",
}
`;
```

**Why this output:** The snapshot captures the exact shape of the response, including nested objects and arrays. Dynamic fields (`id`, `createdAt`, `updatedAt`, `instance`) are matched with `expect.any(String)` so they do not cause false failures. The snapshot serves as documentation of the API contract and catches any unintended structural changes.

### Real-World Cases

- **Public APIs:** Snapshot the response shape of every endpoint to catch accidental breaking changes.
- **Configuration generation:** Snapshot generated configuration files to detect unintended changes.
- **Error responses:** Snapshot RFC 7807 error payloads to ensure consistent error formatting.
- **GraphQL APIs:** Snapshot the shape of query responses to detect schema changes.

---

## References

- Testing Strategy Skill — Testing Pyramid, Framework Selection, Mocking Patterns — https://cdn.jsdelivr.net/npm/skills-ws@1.10.0/skills/testing-strategy/SKILL.md
- Unit Testing Core Knowledge (bmad-labs) — https://github.com/bmad-labs/skills/blob/main/skills/typescript-unit-testing/references/common/knowledge.md
- Negative Testing Techniques Guide (Minitap, September 2026) — https://www.minitap.ai/magazine/negative-testing-techniques-guide
- Jest Coverage Threshold Enforcement (mondominator/sappho PR #191) — https://github.com/mondominator/sappho/pull/191
- Use Test Doubles in Android (Android Developers) — https://developer.android.google.cn/training/testing/fundamentals/test-doubles
- TRAE-Skills: Snapshot Testing Jest — https://github.com/HighMark-31/TRAE-Skills/blob/main/testing/Snapshot_Testing_Jest.md
- How to Snapshot Test in Node.js (CoreUI, January 2026) — https://coreui.io/blog/how-to-snapshot-test-in-node-js/
- Boundary Value Analysis Skill (sethdford/claude-skills) — https://github.com/sethdford/claude-skills/blob/main/qa-engineer/functional-testing/skills/boundary-value-analysis/SKILL.md
- Software Regression Testing (NASA SWE-191) — https://swehb.nasa.gov
- Regression Testing Best Practices (PractiTest, July 2025) — https://www.practitest.com
- Test Doubles Strategy Skill (sethdford/claude-skills) — https://github.com/sethdford/claude-skills/blob/main/engineer/testing/skills/test-doubles-strategy/SKILL.md
- Snapshot Testing Best Practices (ReallyArtificial/mcp-jest) — https://github.com/ReallyArtificial/mcp-jest/blob/main/docs/guides/snapshot-testing.md
- Environment Configuration for Jest (Stack Overflow) — https://stackoverflow.com/questions/78441017