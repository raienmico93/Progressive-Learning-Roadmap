# Testing Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Software testing is the systematic process of evaluating a software application to verify that it behaves as expected, meets requirements, and is free of defects. Testing fundamentals encompass the types of tests, methodologies, isolation techniques, and quality metrics that guide effective verification.

**Technical Definition:** Testing is the dynamic verification of program behavior against a specification or oracle. It involves executing the software under controlled conditions (test cases), observing the outcomes, and comparing them to expected results. Testing fundamentals include the classification of tests by scope (unit, integration, end-to-end), the strategic allocation of testing effort (test pyramid), development methodologies (TDD, BDD), structural coverage metrics (test coverage), the isolation of the system under test using test doubles, and the principles that govern maintainable, reliable test suites.

**Beginner-Friendly Explanation:** Testing is how you check that your code actually works. Instead of hoping for the best, you write small programs (tests) that run your code with known inputs and compare the results to what you expect. Testing fundamentals are the vocabulary and rules of thumb — like "write lots of small tests, fewer big ones" — that help you test effectively.

### Key Characteristics

- **Verification and validation:** Testing confirms both that the software is built correctly (verification) and that the right software is built (validation).
- **Leveled scope:** Tests range from tiny (single function) to massive (full user flow across systems).
- **Automation-first:** Modern testing emphasizes automated tests that run in CI/CD pipelines.
- **Isolation is foundational:** Good tests isolate the system under test from external dependencies and from other tests.
- **Feedback speed matters:** Faster tests run more often; slower tests run less often but catch different classes of bugs.

### Prerequisites

- A codebase written in a language with a testing framework (e.g., Jest/Vitest for JavaScript, pytest for Python, JUnit for Java).
- Basic understanding of the software's requirements and expected behavior.
- Familiarity with assertions and test runners.
- A CI/CD pipeline (recommended) for running tests automatically.

### Related Programming Areas

- **Test-Driven Development (TDD)** — writing tests before code to drive design.
- **Behavior-Driven Development (BDD)** — writing tests in business-readable language.
- **Continuous Integration (CI)** — automating test execution on every change.
- **Code quality metrics** — coverage, mutation testing, static analysis.
- **Refactoring** — tests act as a safety net for restructuring code.

### Core Concepts / Features

---

## 1. Testing

### Definitions

**Core Definition:** Testing is the process of executing a program or application with the intent of finding defects and verifying that it meets its requirements.

**Technical Definition:** Testing is a dynamic verification technique that involves designing test cases (inputs, preconditions, and expected outcomes), executing the software with those inputs, and comparing the actual outcomes to the expected outcomes. It provides evidence of correctness but does not prove correctness — testing can reveal the presence of defects, not their absence (Dijkstra's principle). Testing encompasses multiple levels (unit, integration, E2E), types (functional, non-functional), and methodologies (TDD, BDD).

**Beginner-Friendly Explanation:** Testing is how you check that your code does what it's supposed to do. You give your code some input, see what comes out, and check if it matches what you expected. If it doesn't match, you found a bug.

### Purposes

- To verify that software behaves according to its specification.
- To detect defects early, before they reach production.
- To provide a safety net that enables confident refactoring.
- To document expected behavior through executable examples.
- To reduce the risk and cost of software failures.

### Syntax Rules and Structure

**Anatomy of a test (Arrange-Act-Assert pattern):**

```js
// Arrange: set up the test conditions
const input = 5;
const expected = 25;

// Act: execute the code under test
const actual = square(input);

// Assert: verify the outcome
expect(actual).toBe(expected);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Arrange | Prepare inputs, dependencies, and expected outcomes. |
| Act | Execute the function or trigger the behavior under test. |
| Assert | Compare actual results to expected results using an assertion library. |

**Syntax Rules:**

1. Every test must have at least one assertion; a test without an assertion is a "dead test" that provides false confidence.
2. Tests should be self-validating — pass/fail without manual inspection.
3. Follow the FIRST principles (see Section 11).
4. One logical concept per test to make failures easy to diagnose.

**Constraints and Limitations:**

- Testing cannot prove the absence of bugs — only their presence.
- Tests are code too; they must be maintained and can contain bugs.
- Over-testing implementation details makes refactoring harder.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: A Basic Unit Test (JavaScript, Vitest)

```js
// math.js — the code under test
export function square(n) {
  return n * n;
}

// math.test.js — the test
import { describe, test, expect } from 'vitest';
import { square } from './math.js';

describe('square', () => {
  test('returns the square of a positive number', () => {
    // Arrange
    const input = 5;
    const expected = 25;

    // Act
    const actual = square(input);

    // Assert
    expect(actual).toBe(expected);
  });

  test('returns 0 for 0', () => {
    expect(square(0)).toBe(0);
  });
});
```

**Expected Output:** Both tests pass (green). If `square` were implemented incorrectly (e.g., returning `n + n`), the first test would fail with "expected 25 but got 10."

**Why this output:** The test arranges the input, acts by calling `square`, and asserts that the result equals the expected value. The second test covers the edge case of zero.

### Real-World Cases

- **Regression prevention:** A test suite catches bugs when code changes break existing behavior.
- **Documentation:** Tests serve as executable specifications of how the code should behave.
- **Refactoring confidence:** A comprehensive test suite allows developers to restructure code without fear of breaking functionality.

### References

- Martin Fowler — Test Pyramid — https://martinfowler.com/articles/practical-test-pyramid.html
- ISTQB — Certified Tester Foundation Level Syllabus — https://www.istqb.org/certifications/certified-tester-foundation-level

---

## 2. Test Types

### 2.1 Unit Tests

#### Definitions

**Core Definition:** A unit test verifies the behavior of a single, isolated unit of code — typically a function, method, or small class — in isolation from its dependencies.

**Technical Definition:** A unit test exercises the smallest testable piece of software, replacing all external dependencies (databases, APIs, file systems) with test doubles. Unit tests are fast (milliseconds), deterministic, and run in isolation from other tests. They form the base of the test pyramid and are the most numerous type of test in a healthy suite.

**Beginner-Friendly Explanation:** A unit test checks one tiny piece of your code — like a single function — all by itself. It doesn't touch the database or the network; it just checks that the function returns the right output for a given input.

#### Purposes

- To verify the correctness of individual functions and logic units.
- To catch logic errors and edge cases early in development.
- To provide fast feedback during development and in CI.
- To document the intended behavior of a unit's API.

#### Syntax Rules and Structure

```js
test('unit under test behaves as expected', () => {
  // Arrange: inputs and expected outcome
  // Act: call the unit
  // Assert: verify the outcome
});
```

**Constraints and Limitations:**

- Unit tests do not verify integration between components — they intentionally isolate.
- Mocking too aggressively can produce tests that pass even when integration is broken.
- Unit tests cannot catch configuration or wiring errors.

#### Annotated Code Example

```js
// priceCalculator.js
export function calculateTotal(price, quantity) {
  if (price < 0 || quantity < 0) throw new Error('Invalid input');
  return price * quantity;
}

// priceCalculator.test.js
import { test, expect } from 'vitest';
import { calculateTotal } from './priceCalculator.js';

test('calculates total for valid inputs', () => {
  expect(calculateTotal(10, 3)).toBe(30);
});

test('throws on negative price', () => {
  expect(() => calculateTotal(-1, 3)).toThrow('Invalid input');
});
```

**Expected Output:** Both tests pass. The first verifies the happy path; the second verifies input validation.

#### Real-World Cases

- **Utility functions:** Date formatting, string manipulation, math calculations.
- **Business logic:** Tax calculation, discount rules, validation logic.
- **State reducers:** Redux reducers or pure state transformation functions.

#### References

- Martin Fowler — Unit Test — https://martinfowler.com/bliki/UnitTest.html
- Android Developers — Test Doubles — https://developer.android.com/training/testing/fundamentals/test-doubles

---

### 2.2 Integration Tests

#### Definitions

**Core Definition:** An integration test verifies that multiple units or modules work together correctly — for example, a component interacting with a database, an API, or another component.

**Technical Definition:** Integration tests sit above unit tests in the test pyramid. They exercise the boundaries between components, verifying that data flows correctly across interfaces, that wiring is correct, and that external dependencies (databases, message queues, HTTP clients) behave as expected. Integration tests are slower than unit tests but catch a different class of bugs — integration and configuration errors.

**Beginner-Friendly Explanation:** An integration test checks that different parts of your app work together. For example, it might check that your code correctly saves data to a real database and reads it back, or that two services can talk to each other.

#### Purposes

- To verify that modules interact correctly across their boundaries.
- To catch wiring, configuration, and contract errors.
- To test database queries, API clients, and service integrations.
- To provide confidence that the system works as a whole, not just in pieces.

#### Syntax Rules and Structure

```js
test('component integrates with database', async () => {
  // Set up a real or test-container database
  // Exercise the integration point
  // Assert on the result
});
```

**Constraints and Limitations:**

- Slower than unit tests — require careful management in CI.
- Often require external infrastructure (databases, containers, mock servers).
- Can be flaky if dependencies are not properly controlled.

#### Annotated Code Example

```js
// userRepository.js
export class UserRepository {
  constructor(db) { this.db = db; }
  async findById(id) {
    const result = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] || null;
  }
}

// userRepository.integration.test.js
import { test, expect, beforeAll, afterAll } from 'vitest';
import { createTestDb } from './testing/createTestDb.js';
import { UserRepository } from './userRepository.js';

let db, repo;

beforeAll(async () => {
  db = await createTestDb(); // e.g., a test container or in-memory DB
  repo = new UserRepository(db);
  await db.query("INSERT INTO users (id, name) VALUES (1, 'Alice')");
});

afterAll(async () => { await db.close(); });

test('finds an existing user by id', async () => {
  const user = await repo.findById(1);
  expect(user.name).toBe('Alice');
});

test('returns null for a missing user', async () => {
  const user = await repo.findById(999);
  expect(user).toBeNull();
});
```

**Expected Output:** Both tests pass against a real (test) database. The first verifies the query and mapping; the second verifies the null case.

#### Real-World Cases

- **API endpoint testing:** Verifying that HTTP handlers correctly parse requests and return responses.
- **Database layer testing:** Ensuring SQL queries return expected results.
- **Service integration:** Testing that a payment service correctly calls a third-party gateway (using a mock server or sandbox).

#### References

- Martin Fowler — Integration Test — https://martinfowler.com/bliki/IntegrationTest.html
- Google Testing Blog — Just Say No to More End-to-End Tests — https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html

---

### 2.3 End-to-End (E2E) Tests

#### Definitions

**Core Definition:** An end-to-end test verifies a complete user workflow through the entire application stack — from the UI through the backend to the database — in an environment that closely resembles production.

**Technical Definition:** E2E tests drive the application through its external interfaces (usually a browser for web apps) and assert that the system behaves correctly from the user's perspective. They are the slowest and most expensive tests in the pyramid, and they are reserved for critical user journeys (login, checkout, payment). E2E tests catch issues that unit and integration tests miss — such as UI rendering problems, navigation bugs, and cross-system failures.

**Beginner-Friendly Explanation:** An end-to-end test acts like a real user. It opens the app, clicks buttons, fills out forms, and checks that everything works from start to finish — including the parts behind the scenes.

#### Purposes

- To verify critical user flows work through the full stack.
- To catch integration issues that unit and integration tests miss.
- To validate the system in an environment close to production.
- To provide confidence for releases and deployments.

#### Syntax Rules and Structure

```js
// Playwright example
test('user can log in', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'user@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```

**Constraints and Limitations:**

- Slow (seconds to minutes per test) and expensive to maintain.
- Flaky by nature — depend on timing, network, and environment.
- Should be a small portion of the test suite (typically ~10%).

#### Annotated Code Example

```js
// checkout.spec.js — Playwright E2E test
import { test, expect } from '@playwright/test';

test('complete checkout flow', async ({ page }) => {
  await page.goto('/products');
  await page.click('text=Add to Cart');
  await page.click('text=Checkout');
  await page.fill('[name="address"]', '123 Main St');
  await page.click('text=Place Order');
  await expect(page.locator('.confirmation')).toContainText('Order confirmed');
});
```

**Expected Output:** The test passes if the full checkout flow works. If any step fails (e.g., the cart button is broken, the address form rejects valid input, or the order confirmation is missing), the test fails at that step.

#### Real-World Cases

- **E-commerce:** Complete purchase flow — search → add to cart → checkout → payment → confirmation.
- **Authentication:** Sign-up, login, logout, and password reset flows.
- **SaaS onboarding:** Account creation → onboarding wizard → first dashboard view.

#### References

- Playwright — End-to-End Testing — https://playwright.dev/docs/test-intro
- Cypress — End-to-End Testing — https://docs.cypress.io/guides/end-to-end-testing/writing-your-first-end-to-end-test

---

### 2.4 Snapshot Tests

#### Definitions

**Core Definition:** A snapshot test captures the rendered output of a component (or serialized data) and compares it to a previously saved snapshot; if the output changes, the test fails until the snapshot is updated.

**Technical Definition:** Snapshot testing serializes a component's rendered output (or any serializable value) to a file on the first run. On subsequent runs, the new output is compared to the stored snapshot. If they differ, the test fails, and the developer must either approve the change (update the snapshot) or fix the regression. Snapshot tests are common in UI component libraries for catching unintended rendering changes.

**Beginner-Friendly Explanation:** A snapshot test takes a "photo" of what your component renders. The next time it runs, it compares the new photo to the old one. If they look different, it tells you something changed.

#### Purposes

- To detect unintended changes in rendered output (UI regressions).
- To quickly add broad coverage for component rendering without writing explicit assertions.
- To document the visual structure of components.

#### Syntax Rules and Structure

```js
test('renders correctly', () => {
  const tree = renderer.create(<Button label="Click" />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

**Constraints and Limitations:**

- Snapshots are easy to accidentally approve without reviewing the diff — they can hide regressions if not carefully managed.
- Large snapshots are hard to review and understand.
- Snapshot tests are brittle — they fail on trivial changes (whitespace, class name ordering).

#### Annotated Code Example

```jsx
// Button.jsx
export function Button({ label }) {
  return <button className="btn">{label}</button>;
}

// Button.test.jsx
import { test, expect } from 'vitest';
import { render } from '@testing-library/react';
import { Button } from './Button.jsx';

test('renders the button with the label', () => {
  const { container } = render(<Button label="Click me" />);
  expect(container.firstChild).toMatchSnapshot();
});
```

**Expected Output:** The first run creates a `__snapshots__` file containing the serialized HTML. Subsequent runs pass if the output matches; fail if it differs.

#### Real-World Cases

- **UI component libraries:** Ensuring that changes to a design system do not unintentionally alter rendered markup.
- **Configuration objects:** Snapshotting large configuration outputs to detect changes.
- **Error messages:** Snapshotting error message formatting.

#### References

- Jest — Snapshot Testing — https://jestjs.io/docs/snapshot-testing
- Vitest — Snapshot — https://vitest.dev/guide/snapshot

---

### 2.5 Visual Regression Tests

#### Definitions

**Core Definition:** Visual regression testing captures screenshots of the UI and compares them pixel-by-pixel (or perceptually) to baseline images to detect unintended visual changes.

**Technical Definition:** Visual regression tests render the UI in a browser (or headless browser), capture a screenshot, and compare it to a stored baseline. Differences are highlighted as visual diffs. This catches layout breaks, styling regressions, and cross-browser rendering differences that DOM-based tests miss. Tools include Playwright's `toHaveScreenshot`, Chromatic, and Percy.

**Beginner-Friendly Explanation:** Visual regression testing takes a picture of your app and compares it to a picture from before. If something moved, changed color, or broke layout, the test shows you exactly what changed.

#### Purposes

- To detect unintended visual changes (layout, color, spacing, typography).
- To verify cross-browser and cross-device rendering consistency.
- To catch CSS regressions that unit and snapshot tests miss.

#### Syntax Rules and Structure

```js
// Playwright visual regression
test('button looks correct', async ({ page }) => {
  await page.goto('/components/button');
  await expect(page.locator('.button')).toHaveScreenshot('button.png');
});
```

**Constraints and Limitations:**

- Sensitive to environment differences (fonts, antialiasing, viewport size).
- Requires baselines to be maintained and updated carefully.
- Can be slow and resource-intensive.

#### Annotated Code Example

```js
// visual.spec.js — Playwright
import { test, expect } from '@playwright/test';

test('dashboard layout is stable', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page).toHaveScreenshot('dashboard.png', {
    threshold: 0.2, // Allow small anti-aliasing differences
  });
});
```

**Expected Output:** On first run, a baseline screenshot is created. On subsequent runs, the test passes if the screenshot matches within the threshold; fails with a visual diff if it differs.

#### Real-World Cases

- **Design systems:** Ensuring components render consistently across browsers.
- **Marketing pages:** Catching layout breaks after CSS changes.
- **Accessibility:** Verifying focus states and high-contrast modes render correctly.

#### References

- Playwright — Visual Comparisons — https://playwright.dev/docs/test-snapshots
- Chromatic — Visual Testing — https://www.chromatic.com/features/visual-test

---

## 3. Test Pyramid

### Definitions

**Core Definition:** The test pyramid is a model for structuring a test suite that recommends many fast unit tests at the base, fewer integration tests in the middle, and a small number of slow end-to-end tests at the top.

**Technical Definition:** Introduced by Mike Cohn and popularized by Martin Fowler, the test pyramid guides the distribution of testing effort across levels of granularity. The base (unit tests) is broad because these tests are fast, cheap, and isolate failures. The middle (integration tests) verifies boundaries. The apex (E2E tests) is narrow because these tests are slow, expensive, and flaky. A healthy distribution is approximately 70% unit, 20% integration, and 10% E2E.

**Beginner-Friendly Explanation:** The test pyramid says: write lots of small, fast tests; write some medium-sized tests that check how parts work together; and write only a few big tests that check the whole system.

### Purposes

- To guide investment toward fast, maintainable tests at the base.
- To avoid the "ice cream cone" anti-pattern (too many slow E2E tests).
- To provide fast feedback while still covering integration and user flows.
- To balance test speed, cost, and confidence.

### Syntax Rules and Structure

```
        /\
       /  \      E2E (10%) — critical user flows, slow
      /────\
     /      \    Integration (20%) — boundaries, medium
    /────────\
   /          \  Unit (70%) — logic, fast, many
  /────────────\
```

**Component Breakdown:**

| Layer | Proportion | Speed | What to test |
|-------|------------|-------|--------------|
| Unit | ~70% | < 50ms | Functions, utilities, state logic |
| Integration | ~20% | 100–500ms | API endpoints, database ops, module wiring |
| E2E | ~10% | 5–30s | Critical user flows (login, checkout) |

**Constraints and Limitations:**

- The classic pyramid is optimized for backend systems; for UI-heavy applications, the "Testing Trophy" (favoring integration tests) is an alternative.
- The exact ratios are guidelines, not laws — adjust to the risk profile of the application.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: A Pyramid-Aligned Suite

```js
// Unit tests (many)
test('calculates tax correctly', () => { /* ... */ });
test('validates email format', () => { /* ... */ });
test('formats currency', () => { /* ... */ });

// Integration tests (some)
test('userRepository saves and retrieves a user', async () => { /* ... */ });
test('API endpoint returns 200 for valid request', async () => { /* ... */ });

// E2E tests (few)
test('user can complete checkout', async ({ page }) => { /* ... */ });
```

**Why this structure works:** The unit tests provide fast feedback on logic. The integration tests catch wiring and contract errors. The E2E tests verify the most critical paths without dominating the suite.

### Real-World Cases

- **Backend services:** API gateways and microservices use the pyramid to balance unit logic tests with integration tests for database and message-queue boundaries.
- **Web applications:** The pyramid (or trophy) guides the split between component tests and Playwright E2E tests.
- **Mobile apps:** The pyramid is adapted for iOS/Android with unit tests for ViewModels, integration tests for API clients, and UI tests for critical flows.

### References

- Martin Fowler — The Practical Test Pyramid — https://martinfowler.com/articles/practical-test-pyramid.html
- Semaphore — Test Pyramid — https://semaphore.io/blog/testing-pyramid

---

## 4. Test-Driven Development (TDD)

### Definitions

**Core Definition:** Test-Driven Development is a software development methodology in which tests are written before the production code, following a short Red-Green-Refactor cycle.

**Technical Definition:** TDD, attributed to Kent Beck, is a disciplined practice where the developer writes a failing test for the next increment of functionality (Red), writes the minimum production code to make it pass (Green), and then refactors the code while keeping the test green (Refactor). TDD drives design from the outside in — the test acts as the first consumer of the API, forcing the developer to think about the interface before the implementation.

**Beginner-Friendly Explanation:** TDD means you write the test first, watch it fail, then write just enough code to make it pass, and finally clean up the code. It's a cycle: Red (failing test), Green (passing test), Refactor (clean up).

### Purposes

- To drive the design of code from its intended usage.
- To ensure every piece of production code has a test.
- To enable safe, incremental refactoring.
- To produce a comprehensive regression suite as a byproduct.

### Syntax Rules and Structure

**The Red-Green-Refactor cycle:**

| Phase | Action | Outcome |
|-------|--------|---------|
| **Red** | Write a test for new functionality | Test fails (code does not exist yet) |
| **Green** | Write the minimum code to pass the test | Test passes |
| **Refactor** | Improve the code without changing behavior | All tests remain green |

**Constraints and Limitations:**

- Requires discipline — skipping the refactor phase leads to messy code.
- Writing meaningful tests before code can be challenging for complex domains.
- TDD is not a substitute for integration and E2E testing.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: TDD Cycle for a FizzBuzz Function

```js
// Step 1 (Red): Write a failing test
test('returns "Fizz" for 3', () => {
  expect(fizzbuzz(3)).toBe('Fizz');
});
// Run → fails (fizzbuzz is not defined)

// Step 2 (Green): Write minimal code to pass
function fizzbuzz(n) {
  return 'Fizz';
}
// Run → passes

// Step 3 (Refactor): Add another test to drive more logic
test('returns the number for non-multiples of 3 or 5', () => {
  expect(fizzbuzz(1)).toBe(1);
});
// Run → fails (returns 'Fizz' for everything)

// Step 4 (Green): Implement the general case
function fizzbuzz(n) {
  if (n % 3 === 0) return 'Fizz';
  return n;
}
// Run → both tests pass

// Step 5 (Refactor): Clean up without changing behavior
// (code is already clean)
```

**Expected Output:** The test suite grows incrementally, and each test passes after the minimal implementation.

#### Real-World Cases

- **API development:** Writing tests for endpoint contracts before implementing handlers.
- **Library development:** Defining the public API through tests before writing internals.
- **Bug fixing:** Writing a failing test that reproduces the bug, then fixing the code to make it pass.

### References

- Kent Beck — Test-Driven Development: By Example — https://www.oreilly.com/library/view/test-driven-development/0321146530/
- Martin Fowler — TestDrivenDevelopment — https://martinfowler.com/bliki/TestDrivenDevelopment.html

---

## 5. Behavior-Driven Development (BDD)

### Definitions

**Core Definition:** Behavior-Driven Development is an extension of TDD that emphasizes collaboration between developers, testers, and business stakeholders by expressing tests in natural-language, behavior-focused scenarios.

**Technical Definition:** BDD uses a domain-specific language (often Gherkin: Given-When-Then) to describe application behavior in terms that non-technical stakeholders can understand. These scenarios serve as both requirements and automated acceptance tests. BDD focuses on the **behavior** of the system from the user's perspective, rather than the technical implementation details that TDD often targets.

**Beginner-Friendly Explanation:** BDD is like TDD but with tests written in plain English (or a structured language) that describes what the user does and what should happen. This helps developers and business people agree on what the software should do.

### Purposes

- To bridge the communication gap between technical and non-technical stakeholders.
- To express requirements as executable acceptance criteria.
- To focus testing on business behavior rather than implementation details.
- To create a shared understanding of expected behavior before development.

### Syntax Rules and Structure

**Gherkin syntax:**

```gherkin
Feature: User login
  Scenario: Successful login with valid credentials
    Given the user is on the login page
    When the user enters valid credentials
    Then the user is redirected to the dashboard
```

**Constraints and Limitations:**

- BDD requires collaboration and buy-in from business stakeholders.
- The Gherkin layer adds overhead; not every test needs to be a BDD scenario.
- BDD tools (Cucumber, SpecFlow) add complexity to the toolchain.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: BDD Scenario with Cucumber (JavaScript)

```gherkin
# features/login.feature
Feature: Login
  Scenario: Valid credentials
    Given I am on the login page
    When I submit username "alice" and password "secret"
    Then I should see the dashboard
```

```js
// step definitions
Given('I am on the login page', function () {
  this.driver.get('/login');
});

When('I submit username {string} and password {string}', function (username, password) {
  this.driver.findElement(By.name('username')).sendKeys(username);
  this.driver.findElement(By.name('password')).sendKeys(password);
  this.driver.findElement(By.css('button[type=submit]')).click();
});

Then('I should see the dashboard', function () {
  assert(this.driver.getCurrentUrl().includes('/dashboard'));
});
```

**Expected Output:** The Cucumber runner executes the scenario, mapping each Gherkin step to a step definition. The test passes if the browser navigates to the dashboard.

### Real-World Cases

- **Enterprise applications:** BDD aligns business analysts and developers on complex workflows.
- **Regulated industries:** BDD scenarios serve as documented, auditable acceptance criteria.
- **Cross-functional teams:** BDD gives product owners a way to contribute directly to test definitions.

### References

- Cucumber — BDD Documentation — https://cucumber.io/docs/bdd/
- Dan North — Introducing BDD — https://dannorth.net/introducing-bdd/

---

## 6. Test Coverage

### Definitions

**Core Definition:** Test coverage is a metric that measures the percentage of code (statements, branches, functions, or lines) executed by the test suite.

**Technical Definition:** Coverage is typically measured by instrumenting the code during test execution and recording which statements, branches, functions, and lines are hit. Common metrics include statement coverage, branch coverage (decision coverage), function coverage, and line coverage. High coverage is necessary but not sufficient for high quality — a test can execute a line without asserting the correct behavior.

**Beginner-Friendly Explanation:** Test coverage tells you how much of your code your tests actually run. If your coverage is 80%, it means 20% of your code is never executed by any test — and could contain bugs you'll never catch.

### Purposes

- To identify untested parts of the codebase.
- To set a measurable quality gate for CI pipelines.
- To guide where additional tests are needed.
- To track testing progress over time.

### Syntax Rules and Structure

**Coverage metrics:**

| Metric | What it measures |
|--------|------------------|
| Statement coverage | Percentage of executable statements run |
| Branch coverage | Percentage of `if`/`else` and `switch` branches taken |
| Function coverage | Percentage of functions called |
| Line coverage | Percentage of lines executed |

**Coverage thresholds (common guidelines):**

- 80%+ is often cited as a reasonable target.
- Critical business logic should approach 100% branch coverage.
- Generated code and configuration files may be excluded.

**Constraints and Limitations:**

- Coverage measures execution, not correctness — 100% coverage does not mean 100% correct.
- Chasing coverage numbers can lead to meaningless tests that execute code without asserting behavior.
- Branch coverage is more meaningful than line coverage.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Interpreting Coverage Reports

```js
// discount.js
export function applyDiscount(price, isMember) {
  if (isMember) {
    return price * 0.9;
  }
  return price;
}
```

```js
// discount.test.js
test('applies discount for members', () => {
  expect(applyDiscount(100, true)).toBe(90);
});
```

**Coverage result:** Statement coverage: 100% (both lines executed). Branch coverage: 50% (only the `isMember === true` branch is tested; the `false` branch is never executed).

**Why this matters:** The function appears fully covered on a line basis, but the non-member path is untested. Branch coverage reveals the gap.

### Real-World Cases

- **CI quality gates:** PRs fail if coverage drops below a threshold.
- **Legacy code assessment:** Coverage reports reveal which parts of an old codebase are safe to refactor.
- **Test gap analysis:** Coverage maps highlight untested edge cases and error-handling paths.

### References

- Istanbul (nyc) — Code Coverage — https://istanbul.js.org/
- Martin Fowler — TestCoverage — https://martinfowler.com/bliki/TestCoverage.html

---

## 7. Test Isolation

### Definitions

**Core Definition:** Test isolation is the practice of ensuring that each test runs independently of other tests and external systems, so that tests can run in any order and produce consistent results.

**Technical Definition:** Isolation operates on two dimensions: (1) **isolation from external dependencies** — unit tests replace databases, APIs, and file systems with test doubles; and (2) **isolation between tests** — no test depends on another test's leftover state. Isolated tests set up their own data, clean up after themselves, and do not share mutable state. Isolation is the "I" in the FIRST principles.

**Beginner-Friendly Explanation:** Test isolation means each test stands on its own. It doesn't depend on what another test did before it. You can run the tests in any order, and they still pass.

### Purposes

- To make test failures easy to diagnose (the failure is in the test's own setup).
- To enable parallel test execution for faster CI.
- To prevent order-dependent flakiness.
- To eliminate hidden coupling between tests.

### Syntax Rules and Structure

**Isolated test pattern:**

```js
beforeEach(() => {
  // Fresh setup for every test
  db = createInMemoryDb();
  service = new Service(db);
});

afterEach(() => {
  // Clean up
  db.close();
});
```

**Constraints and Limitations:**

- True end-to-end tests cannot be fully isolated — they intentionally exercise the whole stack and may share state.
- Isolation adds setup overhead per test (mitigated by fast in-memory fakes).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Non-Isolated vs. Isolated Tests

```js
// ❌ NON-ISOLATED: second test depends on first test's data
let db;
beforeAll(() => { db = createDb(); });

test('creates an order', () => { /* inserts order */ });
test('finds the order', () => { /* relies on previous test's insert */ });

// ✅ ISOLATED: each test sets up its own state
let db;
beforeEach(() => {
  db = createInMemoryDb();
  db.insert({ id: 1, total: 99 }); // Fresh data per test
});
afterEach(() => db.close());

test('creates an order', () => { /* ... */ });
test('finds the order', () => { /* own setup, no dependency */ });
```

**Expected behavior:** The non-isolated tests fail if run out of order or in parallel. The isolated tests pass in any order.

### Real-World Cases

- **Parallel CI:** Isolated tests can be sharded across multiple CI runners.
- **Test randomization:** Running tests in random order surfaces hidden dependencies.
- **Local development:** Isolated tests run reliably on any developer's machine.

### References

- FIRST Principles — Isolated — https://github.com/thapaliyabikendra/ai-artifacts/blob/main/.claude/knowledge/concepts/testing/first.md
- Google Testing Blog — Test Sizes — https://testing.googleblog.com/2010/12/test-sizes.html

---

## 8. Test Doubles

### Definitions

**Core Definition:** A test double is any object that stands in for a real dependency during testing, providing controlled behavior and enabling the system under test to be exercised in isolation.

**Technical Definition:** The term "test double" was coined by Gerard Meszaros in *xUnit Test Patterns* (drawing on the film industry's "stunt double" concept). Test doubles replace real collaborators (databases, APIs, services) so that the unit under test can be verified without the cost, complexity, and nondeterminism of real dependencies. The main types are **dummies, stubs, spies, mocks, and fakes**, each serving a different verification purpose.

**Beginner-Friendly Explanation:** A test double is a stand-in for a real part of your app — like a stunt double in a movie. Instead of using a real database, you use a fake one that behaves the way you need for the test.

### Purposes

- To isolate the system under test from external dependencies.
- To make tests fast and deterministic.
- To simulate hard-to-reproduce conditions (network failures, timeouts, edge-case responses).
- To verify that the system under test interacts correctly with its collaborators.

### Syntax Rules and Structure

**Test double taxonomy:**

| Double | Behavior | Verification style | When to use |
|--------|----------|--------------------|-------------|
| **Dummy** | Empty object, never used | None | Satisfy a parameter that is not exercised |
| **Stub** | Returns canned responses | State verification | Control inputs to the system under test |
| **Spy** | Wraps a real object, records calls | Hybrid | Verify interactions without replacing behavior |
| **Mock** | Programmed with expectations | Behavioral verification | Verify that specific calls were made |
| **Fake** | Working lightweight implementation | State verification | Replace infrastructure (in-memory DB) |

**Constraints and Limitations:**

- Prefer fakes and stubs over mocks for simplicity and resilience.
- Limit mocks to 3–4 per test; over-mocking produces fragile tests that pass even when integration is broken.
- Avoid mocking what you do not own — wrap third-party libraries behind an interface and mock the interface.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Stub vs. Mock

```js
// STUB: returns canned data, no expectations
class StubUserRepository {
  getUsers() { return [{ id: 1, name: 'Alice' }]; }
}

test('viewModel loads users from repository', () => {
  const vm = new ViewModel(new StubUserRepository());
  expect(vm.firstUserName).toBe('Alice');
});

// MOCK: verifies that a method was called
const mockDb = {
  save: vi.fn(),
};

test('service saves the user', () => {
  const service = new UserService(mockDb);
  service.createUser({ name: 'Bob' });
  expect(mockDb.save).toHaveBeenCalledTimes(1);
});
```

**Expected Output:** The stub test verifies state (the correct user is loaded). The mock test verifies behavior (the save method was called exactly once).

#### Example 2: Fake In-Memory Database

```js
class FakeUserRepository {
  constructor() { this.users = new Map(); }
  save(user) { this.users.set(user.id, user); }
  findById(id) { return this.users.get(id) || null; }
}

test('saves and retrieves a user', () => {
  const repo = new FakeUserRepository();
  const user = { id: 1, name: 'Alice' };
  repo.save(user);
  expect(repo.findById(1)).toEqual(user);
});
```

**Expected Output:** The fake provides a lightweight working implementation that behaves like a real repository without a database.

### Real-World Cases

- **HTTP clients:** Stubbing `fetch` or `axios` to return canned responses.
- **Databases:** Using an in-memory database (SQLite, H2) as a fake for integration tests.
- **Time:** Injecting a fake clock to test time-dependent logic deterministically.
- **Third-party APIs:** Using mock servers (MSW, WireMock) to simulate external services.

### References

- Martin Fowler — Mocks Aren't Stubs — https://martinfowler.com/articles/mocksArentStubs.html
- Gerard Meszaros — xUnit Test Patterns — http://xunitpatterns.com/
- Android Developers — Test Doubles — https://developer.android.com/training/testing/fundamentals/test-doubles

---

### 8.1 Mocks

#### Definitions

**Core Definition:** A mock is a test double that is programmed with expectations about how it should be called and verifies that those expectations are met.

**Technical Definition:** A mock is a test double that asserts **behavior** — it fails the test if the expected method calls, arguments, or call counts do not occur. Unlike stubs, which only provide responses, mocks are active verifiers. Mocks are typically created by a mocking framework (e.g., Mockito, Vitest's `vi.fn()`, unittest.mock).

**Beginner-Friendly Explanation:** A mock is a fake object that checks whether your code called it correctly. If you expect your code to call `save()` exactly once, a mock will fail the test if it's called zero times or twice.

#### Purposes

- To verify that the system under test interacts correctly with its collaborators.
- To assert on call counts, argument values, and call order.
- To test side effects that are expressed as method calls (e.g., logging, event emission).

#### Syntax Rules and Structure

```js
const mock = vi.fn();
mock('arg');
expect(mock).toHaveBeenCalledWith('arg');
expect(mock).toHaveBeenCalledTimes(1);
```

**Constraints and Limitations:**

- Mock-heavy tests are brittle — they break when implementation changes even if behavior is correct.
- Limit mocks to I/O boundaries (databases, APIs, file systems).
- Do not mock what you do not own.

#### Annotated Code Example

```js
import { vi, test, expect } from 'vitest';

test('sends a welcome email after registration', async () => {
  const emailService = { send: vi.fn() };
  const authService = new AuthService(emailService);

  await authService.register('alice@example.com');

  expect(emailService.send).toHaveBeenCalledTimes(1);
  expect(emailService.send).toHaveBeenCalledWith('alice@example.com', 'Welcome!');
});
```

**Expected Output:** The test passes if `register` calls `emailService.send` exactly once with the correct arguments. It fails if the call is missing or the arguments are wrong.

#### Real-World Cases

- **Event emitters:** Verifying that a component emits the correct events.
- **Logging:** Asserting that errors are logged at the correct level.
- **Third-party gateways:** Verifying that a payment service calls the gateway with the correct amount.

#### References

- Martin Fowler — Mocks Aren't Stubs — https://martinfowler.com/articles/mocksArentStubs.html
- Vitest — Mock Functions — https://vitest.dev/api/vi.html

---

### 8.2 Stubs

#### Definitions

**Core Definition:** A stub is a test double that provides canned (pre-programmed) responses to calls made during the test, without asserting anything about how it was called.

**Technical Definition:** A stub replaces a dependency with an object that returns fixed values regardless of input (or returns values based on simple input mapping). Stubs are used to control the indirect inputs to the system under test. They enable **state verification** — the test asserts on the final state, not on the interactions that produced it.

**Beginner-Friendly Explanation:** A stub is a fake that always gives the same answer, no matter what you ask it. It lets you control what your code receives from the outside world.

#### Purposes

- To provide controlled inputs to the system under test.
- To simulate specific responses (success, error, edge case) without a real dependency.
- To keep tests fast and deterministic.

#### Syntax Rules and Structure

```js
const stub = { getData: () => ({ id: 1, name: 'Test' }) };
```

**Constraints and Limitations:**

- Stubs do not verify behavior — they only supply data.
- Over-stubbing can hide integration problems if the stub's behavior diverges from the real dependency.

#### Annotated Code Example

```js
test('displays user name from API stub', () => {
  const apiStub = {
    fetchUser: () => Promise.resolve({ name: 'Alice' }),
  };
  const profile = new Profile(apiStub);

  return profile.load().then(() => {
    expect(profile.displayName).toBe('Alice');
  });
});
```

**Expected Output:** The test passes because the stub returns a user with the name "Alice", and the profile displays it correctly.

#### Real-World Cases

- **API clients:** Stubbing an HTTP client to return a fixed JSON response.
- **Feature flags:** Stubbing a flag service to return `true` for a specific test.
- **Error handling:** Stubbing a service to throw an error to test error paths.

#### References

- Martin Fowler — Stub — https://martinfowler.com/bliki/TestDouble.html
- Android Developers — Test Doubles: Stub — https://developer.android.com/training/testing/fundamentals/test-doubles

---

### 8.3 Spies

#### Definitions

**Core Definition:** A spy is a test double that wraps a real object and records information about how it was called while still executing the real logic.

**Technical Definition:** A spy intercepts calls to a real method, records the call details (arguments, return value, call count), and then delegates to the original implementation. It enables verification of interactions without replacing the behavior. Spies are useful when you want to observe a real object without fully mocking it.

**Beginner-Friendly Explanation:** A spy is like a wiretap on a real object. It lets the real code run but records every call so you can check afterward what happened.

#### Purposes

- To observe interactions with a real object without replacing its behavior.
- To verify that a method was called while still exercising the real implementation.
- To record call arguments for later assertions.

#### Syntax Rules and Structure

```js
const realObject = { doWork: () => 'result' };
const spy = vi.spyOn(realObject, 'doWork');
realObject.doWork();
expect(spy).toHaveBeenCalled();
```

**Constraints and Limitations:**

- Spies add complexity and are generally less preferred than fakes or mocks.
- Spying on methods that have side effects can make tests less predictable.

#### Annotated Code Example

```js
import { vi, test, expect } from 'vitest';

test('logger is called during processing', () => {
  const logger = { log: (msg) => console.log(msg) };
  const spy = vi.spyOn(logger, 'log');
  const processor = new Processor(logger);

  processor.process('data');

  expect(spy).toHaveBeenCalledWith('Processing: data');
  spy.mockRestore();
});
```

**Expected Output:** The test passes if `process` calls `logger.log` with the expected message. The real logging behavior still executes.

#### Real-World Cases

- **Legacy code:** Observing calls to methods that cannot easily be replaced.
- **Partial mocking:** Spying on one method of a large class while using the real implementation.
- **Interaction verification:** Confirming that a callback was invoked.

#### References

- Martin Fowler — Test Double: Spy — https://martinfowler.com/bliki/TestDouble.html
- Vitest — vi.spyOn — https://vitest.dev/api/vi.html#vi-spyon

---

### 8.4 Fakes

#### Definitions

**Core Definition:** A fake is a test double that has a working implementation, but one that is simpler or more suitable for testing than the production implementation (e.g., an in-memory database).

**Technical Definition:** A fake implements the same interface as the real dependency but uses a lightweight, test-friendly implementation. For example, a fake repository might store data in a `Map` instead of a database. Fakes provide realistic behavior without the overhead of external infrastructure. They are generally **preferred over mocks and stubs** because they are more realistic and less coupled to implementation details.

**Beginner-Friendly Explanation:** A fake is a real, working replacement — like using a spreadsheet instead of a full database. It actually stores and retrieves data, but in a simple way that's perfect for testing.

#### Purposes

- To replace infrastructure with a lightweight, working implementation.
- To provide realistic behavior without external dependencies.
- To reduce test complexity and improve test speed.
- To avoid the brittleness of mock-heavy tests.

#### Syntax Rules and Structure

```js
class FakeRepository {
  constructor() { this.data = new Map(); }
  save(item) { this.data.set(item.id, item); }
  find(id) { return this.data.get(id) || null; }
}
```

**Constraints and Limitations:**

- Fakes must be kept in sync with the real interface — if the interface changes, the fake must be updated.
- Fakes can drift from real behavior if not carefully designed.

#### Annotated Code Example

```js
class FakeUserRepository {
  constructor() { this.users = new Map(); }
  async save(user) { this.users.set(user.id, user); }
  async findById(id) { return this.users.get(id) || null; }
}

test('service creates and retrieves a user', async () => {
  const repo = new FakeUserRepository();
  const service = new UserService(repo);

  await service.createUser({ id: 1, name: 'Alice' });
  const found = await service.getUser(1);

  expect(found.name).toBe('Alice');
});
```

**Expected Output:** The fake repository stores and retrieves the user correctly, simulating the behavior of a real repository without a database.

#### Real-World Cases

- **In-memory databases:** SQLite in-memory or H2 for integration tests.
- **Fake HTTP clients:** A fake `fetch` that returns predefined responses.
- **Fake clocks:** A controllable clock for testing time-dependent logic.

#### References

- Android Developers — Test Doubles: Fake — https://developer.android.com/training/testing/fundamentals/test-doubles
- Martin Fowler — Fake — https://martinfowler.com/bliki/TestDouble.html

---

## 9. Testing Best Practices

### Definitions

**Core Definition:** Testing best practices are the guidelines and principles that lead to test suites that are fast, reliable, maintainable, and effective at catching defects.

**Technical Definition:** Best practices encompass the FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely), the test pyramid, behavior-focused testing (not implementation-focused), proper use of test doubles, assertion quality, and test maintainability. They aim to maximize the value of testing investment while minimizing flakiness, slow feedback, and false confidence.

**Beginner-Friendly Explanation:** Best practices are the habits that make your tests actually useful — fast enough to run often, reliable enough to trust, and clear enough to tell you exactly what broke.

### Purposes

- To maximize the speed and reliability of feedback from tests.
- To ensure tests remain maintainable as the codebase evolves.
- To avoid false confidence from tests that pass without asserting meaningful behavior.
- To balance test coverage across the pyramid.

### Syntax Rules and Structure

**FIRST Principles:**

| Principle | Meaning | Violation signs |
|-----------|---------|-----------------|
| **F**ast | Tests run in milliseconds | Tests take seconds each |
| **I**solated | Tests are independent | Tests fail only in certain orders |
| **R**epeatable | Same result every time | Tests fail on CI but pass locally |
| **S**elf-Validating | Automated pass/fail | Need to check console output |
| **T**imely | Written at the right time | Tests written long after code |

**Additional best practices:**

| Practice | Description |
|----------|-------------|
| **Test behavior, not implementation** | Assert on what users see, not internal state |
| **One logical concept per test** | Clear, focused assertions |
| **Prefer fakes over mocks** | Less brittle, more realistic |
| **Use accessibility-aware selectors** | Prefer roles and labels over CSS classes |
| **Avoid hardcoded sleeps** | Use dynamic waits (polling) |
| **Avoid mystery guests** | Keep test data self-contained |

**Constraints and Limitations:**

- FIRST is a guideline, not a law — E2E tests intentionally relax Fast and Isolated.
- Chasing 100% coverage can lead to low-value tests that execute code without asserting.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Behavior-Focused vs. Implementation-Focused Test

```js
// ❌ IMPLEMENTATION-FOCUSED: asserts on internal state
test('increments internal counter', () => {
  const counter = new Counter();
  counter.increment();
  expect(counter._count).toBe(1); // Brittle
});

// ✅ BEHAVIOR-FOCUSED: asserts on observable output
test('displays incremented count', () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**Why the second test is better:** It tests what the user experiences (the displayed count), not the internal variable name. It survives refactoring of the internal implementation.

#### Example 2: FIRST-Compliant Test Setup

```js
import { beforeEach, afterEach, test, expect } from 'vitest';
import { createTestDb } from './testing/createTestDb.js';

let db, service;

beforeEach(() => {
  db = createTestDb();        // Fresh, in-memory → Fast, Isolated, Repeatable
  service = new UserService(db);
});

afterEach(() => {
  db.close();                 // Cleanup → Isolated
});

test('creates a user with the given name', async () => {
  const user = await service.create({ name: 'Alice' });
  expect(user.name).toBe('Alice');  // Self-validating
});
```

**Why this follows FIRST:** The in-memory database makes tests fast and isolated. The assertion makes them self-validating. The `beforeEach` setup ensures repeatability.

### Real-World Cases

- **CI/CD pipelines:** FIRST-compliant tests run quickly and reliably on every commit.
- **Code reviews:** Best practices provide a checklist for evaluating test quality.
- **Legacy test suites:** Refactoring slow, flaky tests using FIRST principles restores confidence and speed.

### References

- FIRST Principles — https://github.com/thapaliyabikendra/ai-artifacts/blob/main/.claude/knowledge/concepts/testing/first.md
- Martin Fowler — Test Pyramid — https://martinfowler.com/articles/practical-test-pyramid.html
- Google Testing Blog — Test Sizes — https://testing.googleblog.com/2010/12/test-sizes.html

---

## References

- Martin Fowler — The Practical Test Pyramid — https://martinfowler.com/articles/practical-test-pyramid.html
- Martin Fowler — Mocks Aren't Stubs — https://martinfowler.com/articles/mocksArentStubs.html
- Martin Fowler — TestDouble — https://martinfowler.com/bliki/TestDouble.html
- Martin Fowler — Test-Driven Development — https://martinfowler.com/bliki/TestDrivenDevelopment.html
- Gerard Meszaros — xUnit Test Patterns — http://xunitpatterns.com/
- Kent Beck — Test-Driven Development: By Example — https://www.oreilly.com/library/view/test-driven-development/0321146530/
- Semaphore — TDD vs. BDD: What's the Difference? — https://semaphore.io/blog/tdd-vs-bdd
- Android Developers — Use Test Doubles in Android — https://developer.android.com/training/testing/fundamentals/test-doubles
- FIRST Principles — https://github.com/thapaliyabikendra/ai-artifacts/blob/main/.claude/knowledge/concepts/testing/first.md
- Playwright — Visual Comparisons — https://playwright.dev/docs/test-snapshots
- Playwright — End-to-End Testing — https://playwright.dev/docs/test-intro
- Jest — Snapshot Testing — https://jestjs.io/docs/snapshot-testing
- Cucumber — BDD Documentation — https://cucumber.io/docs/bdd/
- Dan North — Introducing BDD — https://dannorth.net/introducing-bdd/
- Istanbul — Code Coverage — https://istanbul.js.org/
- Google Testing Blog — Just Say No to More End-to-End Tests — https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html
- Google Testing Blog — Test Sizes — https://testing.googleblog.com/2010/12/test-sizes.html
- Flutter — Reviewing Tests (Test Smells) — https://github.com/flutter/agent-plugins/blob/main/.agents/agents/reidbaker-agent/skills/code-review/references/reviewing_tests.md