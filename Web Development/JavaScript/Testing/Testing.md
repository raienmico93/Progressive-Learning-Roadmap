# JavaScript Testing Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Testing is the systematic process of verifying that software behaves as expected, catching defects early, documenting intent, and enabling safe refactoring through automated verification.

**Technical Definition:** Testing in JavaScript encompasses a hierarchy of practices and tools: test types (unit, integration, end-to-end, contract, snapshot), methodologies (TDD, BDD), quality metrics (coverage), isolation techniques (mocks, stubs, spies, fakes), assertions, test runners, and test frameworks. Modern JavaScript testing is supported by standardised assertion interfaces (Node.js `assert`, Chai `expect`, Jest `expect`), test runner APIs (Jest, Vitest, Mocha, Jasmine, Node's built-in `node:test`), and browser automation frameworks (Playwright, Cypress, Puppeteer, Selenium). The testing ecosystem is not specified by ECMAScript itself; it is provided by third-party libraries and runtime features. Testing relies on JavaScript fundamentals: functions, modules, async/await, Promises, and error handling.

**Beginner-Friendly Explanation:** Testing is how you check that your code actually does what you think it does. Instead of manually clicking around your app every time you make a change, you write automated tests that run in seconds and tell you if something broke. Tests catch bugs early, document how your code is supposed to work, and give you confidence to refactor. There are different kinds of tests — some check a single function, others check that multiple parts work together, and some simulate a user clicking through your entire app.

### Key Characteristics

- **Automated:** Tests run without human intervention.
- **Repeatable:** Same input, same output, every time.
- **Fast feedback:** Tests run quickly to support rapid iteration.
- **Isolated:** Each test is independent of others.
- **Descriptive:** Test names describe expected behaviour.
- **Regression-preventing:** Tests catch when old bugs return.
- **Layered:** Unit, integration, and E2E tests complement each other.
- **Ecosystem-driven:** Many tools, no single standard.

### Prerequisites

- Solid JavaScript fundamentals (functions, modules, async/await).
- Understanding of Promises and error handling.
- Familiarity with npm/yarn and `package.json`.
- Awareness of module systems (ESM, CommonJS).
- Basic knowledge of the command line.

### Related Programming Areas

- **Continuous integration:** Running tests on every commit.
- **Test-driven development:** Writing tests before code.
- **Behaviour-driven development:** Tests as specifications.
- **Code coverage:** Measuring test thoroughness.
- **Debugging:** Tests help reproduce bugs.
- **Refactoring:** Tests enable safe change.

---

## Why Testing Matters

### Definitions

**Core Definition:** Testing matters because it catches bugs early, documents expected behaviour, and enables confident change.

**Technical Definition:** Testing reduces defect density, shortens feedback loops, and provides executable documentation. Automated tests are the primary mechanism for regression prevention in modern software engineering. Studies (e.g., from Microsoft, Google) show that defects found earlier are orders of magnitude cheaper to fix than those found in production. Tests also serve as specifications: they describe what the code should do in a form that can be executed and verified.

**Beginner-Friendly Explanation:** Testing matters because bugs are expensive. The earlier you find them, the cheaper they are to fix. Tests also double as documentation — they show how your code is supposed to be used.

### Purposes

- To catch bugs before they reach users.
- To prevent regressions when changing code.
- To document expected behaviour.
- To enable confident refactoring.
- To improve code design (testable code is usually well-structured).
- To reduce debugging time.

### Real-World Cases

- **Open-source libraries:** Tests are essential for contributors.
- **Production apps:** Tests prevent costly outages.
- **Regulated industries:** Tests provide compliance evidence.
- **Legacy code:** Tests enable safe modernisation.

---

## Test Types

### Unit Tests

#### Definitions

- **Core Definition:** Unit tests verify the smallest testable units of code (functions, methods, classes) in isolation.
- **Technical Definition:** A unit test exercises a single "unit" (usually a function or class) with controlled inputs, asserting on outputs and side effects. Dependencies are typically replaced with test doubles. Unit tests are fast (milliseconds), deterministic, and numerous.
- **Beginner-Friendly Explanation:** A unit test checks one small piece of code — like a single function — to make sure it works correctly.

#### Purposes

- To verify individual functions.
- To catch logic errors early.
- To document function contracts.
- To enable fast feedback.

#### Annotated Code Examples

```js
// sum.js
export function sum(a, b) { return a + b; }

// sum.test.js (Vitest/Jest)
import { describe, it, expect } from "vitest";
import { sum } from "./sum.js";

describe("sum", () => {
  it("adds two positive numbers", () => {
    expect(sum(2, 3)).toBe(5);
  });

  it("handles negative numbers", () => {
    expect(sum(-1, 1)).toBe(0);
  });

  it("handles zero", () => {
    expect(sum(0, 0)).toBe(0);
  });
});
```

**Expected Output:**
```
✓ sum > adds two positive numbers
✓ sum > handles negative numbers
✓ sum > handles zero
```

#### Real-World Cases

- **Utility functions:** Date formatting, string manipulation.
- **Pure functions:** Calculations, transformations.
- **Reducers:** Redux reducers are pure and easily unit-tested.

---

### Integration Tests

#### Definitions

- **Core Definition:** Integration tests verify that multiple units work together correctly.
- **Technical Definition:** Integration tests exercise the interaction between two or more components, modules, or services. They often use real dependencies (database, network) or partially mocked ones. Slower than unit tests but faster than E2E.
- **Beginner-Friendly Explanation:** Integration tests check that different parts of your code work together — like a function that calls a database.

#### Purposes

- To verify component interactions.
- To catch interface mismatches.
- To test database queries.
- To validate API contracts.

#### Annotated Code Examples

```js
// userService.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { createUser, getUser } from "./userService.js";
import { db } from "./db.js";

describe("UserService", () => {
  beforeEach(async () => {
    await db.clear();
  });

  it("creates and retrieves a user", async () => {
    const user = await createUser({ name: "Alice", email: "alice@example.com" });
    const fetched = await getUser(user.id);
    expect(fetched.name).toBe("Alice");
    expect(fetched.email).toBe("alice@example.com");
  });
});
```

#### Real-World Cases

- **Database layers:** Testing repositories.
- **API clients:** Testing HTTP interactions.
- **Service layers:** Testing business logic with dependencies.

---

### End-to-End Tests

#### Definitions

- **Core Definition:** End-to-end (E2E) tests simulate real user interactions with the fully deployed application.
- **Technical Definition:** E2E tests drive a real browser (or app) through user flows, from the UI through all layers to the database and back. Tools include Playwright, Cypress, Puppeteer, and Selenium. Slower and more brittle than unit tests, but highest confidence.
- **Beginner-Friendly Explanation:** E2E tests act like a real user — clicking buttons, filling forms, and checking that everything works from start to finish.

#### Purposes

- To verify complete user flows.
- To catch integration issues across the stack.
- To validate deployment.
- To test critical paths (login, checkout).

#### Annotated Code Examples

```js
// Playwright example
import { test, expect } from "@playwright/test";

test("user can log in", async ({ page }) => {
  await page.goto("https://app.example.com/login");
  await page.fill('input[name="email"]', "alice@example.com");
  await page.fill('input[name="password"]', "secret");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
  await expect(page.locator("h1")).toHaveText("Welcome, Alice");
});
```

#### Real-World Cases

- **E-commerce:** Checkout flows.
- **SaaS:** Login, signup, billing.
- **Critical paths:** Any flow that must not break.

---

### Contract Tests

#### Definitions

- **Core Definition:** Contract tests verify that two services (or a service and a consumer) agree on the shape and behaviour of their interactions.
- **Technical Definition:** Contract testing (e.g., Pact) defines a "contract" between a consumer and a provider. Each side is tested against the contract independently, catching mismatches before integration.
- **Beginner-Friendly Explanation:** Contract tests check that two services agree on how they'll talk to each other — like ensuring an API always returns the fields the frontend expects.

#### Purposes

- To decouple service development.
- To catch breaking API changes.
- To enable independent deployment.

#### Real-World Cases

- **Microservices:** Consumer-provider contracts.
- **Frontend/Backend:** API shape agreements.
- **Third-party APIs:** Validating assumptions.

---

### Snapshot Tests

#### Definitions

- **Core Definition:** Snapshot tests capture the output of a function or component and compare it against a stored reference.
- **Technical Definition:** Snapshot testing (e.g., Jest's `toMatchSnapshot`) serialises the output of a render or computation and stores it in a snapshot file. On subsequent runs, the output is compared; differences fail the test unless explicitly updated.
- **Beginner-Friendly Explanation:** Snapshot tests save the output of a component and check that it doesn't change unexpectedly.

#### Purposes

- To detect unintended changes in UI.
- To quickly test component rendering.
- To document current output.

#### Annotated Code Examples

```jsx
// Button.test.jsx
import { render } from "@testing-library/react";
import { Button } from "./Button";

it("renders correctly", () => {
  const { container } = render(<Button label="Click me" />);
  expect(container.firstChild).toMatchSnapshot();
});
```

**Snapshot file (`__snapshots__/Button.test.jsx.snap`):**
```
exports[`renders correctly 1`] = `
<button class="btn">
  Click me
</button>
`;
```

#### Real-World Cases

- **UI components:** Rendering output.
- **Config generation:** Serialised config.
- **API responses:** Recorded responses.

**Warning:** Snapshot tests can become meaningless if blindly updated. Review changes carefully.

---

## Test Pyramid

### Definitions

**Core Definition:** The test pyramid is a heuristic for balancing test types, with many fast unit tests at the base, fewer integration tests in the middle, and few slow E2E tests at the top.

**Technical Definition:** Popularised by Mike Cohn, the pyramid recommends a distribution: ~70% unit, ~20% integration, ~10% E2E. The rationale is speed, cost, and reliability: unit tests are fast and cheap; E2E tests are slow and brittle. The "testing trophy" (Kent C. Dodds) suggests more integration tests for frontend apps.

**Beginner-Friendly Explanation:** Write lots of fast unit tests, some integration tests, and a few end-to-end tests. This gives you good coverage without slow test suites.

### Pyramid Levels

```
    /\
   /E2\      Few, slow, high confidence
  /----\
 / Int  \    Some, medium speed
/--------\
/  Unit   \  Many, fast, focused
```

### Real-World Cases

- **Startups:** Start with unit + key E2E.
- **Enterprises:** Full pyramid with contract tests.
- **Frontend:** "Testing trophy" favours integration.

---

## Test-Driven Development

### Definitions

**Core Definition:** Test-Driven Development (TDD) is a practice where tests are written before the code they test.

**Technical Definition:** TDD follows the "Red-Green-Refactor" cycle: (1) write a failing test (Red), (2) write the minimum code to pass (Green), (3) refactor while keeping tests green. TDD was popularised by Kent Beck and is a core practice of Extreme Programming (XP).

**Beginner-Friendly Explanation:** TDD means writing the test first, then writing code to make it pass. It sounds backwards but leads to better design.

### Red-Green-Refactor

1. **Red:** Write a test that fails (the feature doesn't exist yet).
2. **Green:** Write the simplest code to pass.
3. **Refactor:** Improve the code without breaking tests.

### Annotated Code Examples

```js
// Step 1 (Red): Write a failing test.
test("fizzbuzz returns Fizz for multiples of 3", () => {
  expect(fizzbuzz(3)).toBe("Fizz");
});
// Fails: fizzbuzz is not defined

// Step 2 (Green): Write minimal code.
function fizzbuzz(n) {
  if (n % 3 === 0) return "Fizz";
  return n;
}
// Test passes

// Step 3 (Refactor): Improve without breaking.
function fizzbuzz(n) {
  if (n % 15 === 0) return "FizzBuzz";
  if (n % 3 === 0) return "Fizz";
  if (n % 5 === 0) return "Buzz";
  return n;
}
```

### Real-World Cases

- **Algorithm development:** Clear specs via tests.
- **Bug fixes:** Write a failing test, then fix.
- **API design:** Tests drive interface.

---

## Behavior-Driven Development

### Definitions

**Core Definition:** Behavior-Driven Development (BDD) is an extension of TDD that expresses tests in natural language focused on behaviour.

**Technical Definition:** BDD uses a `describe`/`it` structure (or `given`/`when`/`then`) to describe behaviour from the user's perspective. Popularised by Dan North, BDD emphasizes collaboration between developers, QA, and business stakeholders. Cucumber and Gherkin are common BDD tools.

**Beginner-Friendly Explanation:** BDD is TDD with a focus on describing behaviour in plain language — what the user does and what should happen.

### Annotated Code Examples

```js
describe("Shopping Cart", () => {
  describe("when empty", () => {
    it("has a total of 0", () => {
      const cart = new Cart();
      expect(cart.total()).toBe(0);
    });
  });

  describe("when items are added", () => {
    it("sums the item prices", () => {
      const cart = new Cart();
      cart.add({ price: 10 });
      cart.add({ price: 5 });
      expect(cart.total()).toBe(15);
    });
  });
});
```

### Real-World Cases

- **User stories:** Translate to BDD scenarios.
- **Stakeholder communication:** Readable tests.
- **Acceptance criteria:** BDD scenarios as criteria.

---

## Test Coverage

### Definitions

**Core Definition:** Test coverage measures the percentage of code executed by tests.

**Technical Definition:** Coverage metrics include line coverage, branch coverage, function coverage, and statement coverage. Tools like Istanbul (nyc), c8, and Jest's built-in coverage instrument code to track execution. Coverage is a useful signal but not a goal: 100% coverage does not guarantee correctness.

**Beginner-Friendly Explanation:** Coverage tells you how much of your code is exercised by tests. It's a useful indicator but not a guarantee.

### Coverage Types

- **Line coverage:** Lines executed.
- **Branch coverage:** Branches (if/else) taken.
- **Function coverage:** Functions called.
- **Statement coverage:** Statements executed.

### Annotated Code Examples

```js
// math.js
export function abs(n) {
  if (n < 0) return -n; // Branch 1
  return n;              // Branch 2
}

// math.test.js
import { abs } from "./math.js";

test("abs of positive", () => {
  expect(abs(5)).toBe(5); // Covers branch 2 only
});
```

**Coverage:** Line coverage 100%, branch coverage 50% (missing the negative branch).

### Real-World Cases

- **CI gates:** Enforce minimum coverage.
- **Legacy code:** Measure before refactoring.
- **Critical code:** Aim for high branch coverage.

**Warning:** Coverage is a tool, not a target. High coverage can coexist with bugs.

---

## Test Isolation

### Definitions

**Core Definition:** Test isolation means each test runs independently, with no shared state affecting results.

**Technical Definition:** Isolated tests do not depend on execution order, shared mutable state, or external resources. Isolation is achieved through `beforeEach`/`afterEach` hooks, fresh instances, and test doubles. Non-isolated tests are flaky and hard to debug.

**Beginner-Friendly Explanation:** Each test should be able to run on its own, in any order, without affecting other tests.

### Best Practices

- Reset state in `beforeEach`.
- Use fresh instances per test.
- Avoid shared mutable globals.
- Clean up timers, listeners, and connections.
- Don't depend on test order.

### Annotated Code Examples

```js
let counter;

beforeEach(() => {
  counter = new Counter(); // Fresh instance per test
});

test("increments", () => {
  counter.increment();
  expect(counter.value).toBe(1);
});

test("starts at 0", () => {
  expect(counter.value).toBe(0); // Fresh instance
});
```

### Real-World Cases

- **Parallel execution:** Isolated tests run in parallel.
- **Flaky tests:** Isolation reduces flakiness.
- **Debugging:** Isolated failures are easier to diagnose.

---

## Test Doubles

### Definitions

**Core Definition:** Test doubles are objects that stand in for real dependencies during testing.

**Technical Definition:** The term (from Gerard Meszaros's "xUnit Test Patterns") covers five types: Dummy, Stub, Spy, Mock, and Fake. In practice, "mock" is often used loosely for any double. Each type serves a different purpose: stubs provide canned answers, spies record calls, mocks verify expectations, fakes have working implementations.

**Beginner-Friendly Explanation:** Test doubles are stand-ins for real objects — like stunt doubles in movies. They let you test your code without needing the real database, API, or file system.

---

### Mocks

#### Definitions

- **Core Definition:** Mocks are objects pre-programmed with expectations about which calls they should receive.
- **Technical Definition:** A mock verifies that specific interactions occurred. If expected calls are missing or unexpected calls occur, the test fails. Mocking frameworks: Jest `jest.fn()`, `jest.mock()`, Sinon `sinon.mock()`.
- **Beginner-Friendly Explanation:** A mock is a stand-in that checks whether your code called it correctly.

#### Annotated Code Examples

```js
const mockFn = vi.fn();
mockFn("hello");
expect(mockFn).toHaveBeenCalledWith("hello");
expect(mockFn).toHaveBeenCalledTimes(1);
```

---

### Stubs

#### Definitions

- **Core Definition:** Stubs provide canned responses to calls made during the test.
- **Technical Definition:** A stub replaces a real function with one that returns a predetermined value, without verifying interactions. Used to control the behaviour of dependencies.
- **Beginner-Friendly Explanation:** A stub is a stand-in that always returns the same answer.

#### Annotated Code Examples

```js
const getUser = vi.fn().mockResolvedValue({ id: 1, name: "Alice" });
const user = await getUser(1);
expect(user.name).toBe("Alice");
```

---

### Spies

#### Definitions

- **Core Definition:** Spies record information about how a function was called without changing its behaviour.
- **Technical Definition:** A spy wraps a real function (or replaces it) and records calls, arguments, return values, and `this`. Used to observe interactions without changing behaviour.
- **Beginner-Friendly Explanation:** A spy watches a function and records what it was called with.

#### Annotated Code Examples

```js
const obj = {
  greet(name) { return `Hello, ${name}`; },
};

const spy = vi.spyOn(obj, "greet");
obj.greet("Alice");
expect(spy).toHaveBeenCalledWith("Alice");
expect(spy).toHaveReturnedWith("Hello, Alice");
```

---

### Fakes

#### Definitions

- **Core Definition:** Fakes are working implementations of a dependency, simplified for testing.
- **Technical Definition:** A fake behaves like the real thing but is simpler (e.g., an in-memory database instead of a real one). Fakes are more complex than stubs but more realistic.
- **Beginner-Friendly Explanation:** A fake is a simplified version of a real thing — like using an in-memory database for tests.

#### Annotated Code Examples

```js
class InMemoryUserRepo {
  constructor() { this.users = new Map(); }
  async save(user) { this.users.set(user.id, user); }
  async findById(id) { return this.users.get(id); }
}

const repo = new InMemoryUserRepo();
await repo.save({ id: 1, name: "Alice" });
const user = await repo.findById(1);
expect(user.name).toBe("Alice");
```

---

## Assertions

### Definitions

**Core Definition:** Assertions are statements that check whether a condition is true; if not, the test fails.

**Technical Definition:** Assertions are provided by assertion libraries (Chai, Jest `expect`, Node.js `assert`). Common assertions: equality (`toBe`, `toEqual`), truthiness (`toBeTruthy`, `toBeFalsy`), containment (`toContain`), exceptions (`toThrow`), async (`resolves`, `rejects`).

**Beginner-Friendly Explanation:** Assertions are the "checks" in a test — "this should equal that," "this should throw," "this should be true."

### Common Assertions (Jest/Vitest)

```js
expect(value).toBe(other);           // Strict equality
expect(value).toEqual(other);        // Deep equality
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();
expect(array).toContain(item);
expect(fn).toThrow(Error);
expect(promise).resolves.toBe(value);
expect(promise).rejects.toThrow();
expect(fn).toHaveBeenCalledWith(arg);
expect(fn).toHaveBeenCalledTimes(n);
```

### Annotated Code Examples

```js
test("user object", () => {
  const user = { name: "Alice", age: 30 };

  expect(user.name).toBe("Alice");
  expect(user).toEqual({ name: "Alice", age: 30 });
  expect(user.age).toBeGreaterThan(18);
  expect(user).toHaveProperty("name");
});
```

### Real-World Cases

- **Unit tests:** Assert on return values.
- **API tests:** Assert on status codes and bodies.
- **UI tests:** Assert on rendered text.

---

## Test Runners

### Definitions

**Core Definition:** Test runners are tools that discover, execute, and report test results.

**Technical Definition:** Test runners provide the execution environment: they find test files, run tests, handle async, report results, and integrate with coverage and watch mode. Examples: Jest, Vitest, Mocha, Jasmine, Node.js `node:test`.

**Beginner-Friendly Explanation:** A test runner is the program that runs your tests and shows you the results.

### Common Test Runners

- **Jest:** Batteries-included, popular for React.
- **Vitest:** Vite-native, fast, ESM-first.
- **Mocha:** Flexible, requires separate assertion library.
- **Jasmine:** Batteries-included, older.
- **Node.js `node:test`:** Built-in, no dependencies.
- **AVA:** Concurrent, minimal.

### Annotated Code Examples

```js
// Vitest example
import { describe, it, expect, beforeEach } from "vitest";

describe("Calculator", () => {
  let calc;
  beforeEach(() => { calc = new Calculator(); });

  it("adds", () => {
    expect(calc.add(2, 3)).toBe(5);
  });
});
```

### Real-World Cases

- **React:** Jest + React Testing Library.
- **Vue:** Vitest + Vue Test Utils.
- **Node.js:** `node:test` for zero-dependency projects.
- **Monorepos:** Nx, Turborepo with shared runners.

---

## Test Frameworks

### Definitions

**Core Definition:** Test frameworks are comprehensive libraries that provide test structure, assertions, runners, and utilities.

**Technical Definition:** Test frameworks combine test structure (`describe`/`it`), assertions (`expect`), runners, mocking, snapshots, coverage, and watch mode. Examples: Jest, Vitest, Jasmine, Mocha + Chai + Sinon.

**Beginner-Friendly Explanation:** A test framework is a complete toolkit for writing and running tests.

### Common Frameworks

- **Jest:** Full-featured, widely used.
- **Vitest:** Modern, fast, Vite-native.
- **Jasmine:** Self-contained.
- **Mocha + Chai + Sinon:** Modular.
- **Node.js `node:test` + `assert`:** Built-in.
- **Playwright Test:** E2E framework.
- **Cypress:** E2E framework.

### Annotated Code Examples

```js
// Vitest (full-featured)
import { describe, it, expect, vi, beforeEach } from "vitest";

describe("UserService", () => {
  let service;
  let mockRepo;

  beforeEach(() => {
    mockRepo = { findById: vi.fn() };
    service = new UserService(mockRepo);
  });

  it("returns user from repository", async () => {
    mockRepo.findById.mockResolvedValue({ id: 1, name: "Alice" });
    const user = await service.getUser(1);
    expect(user.name).toBe("Alice");
    expect(mockRepo.findById).toHaveBeenCalledWith(1);
  });
});
```

### Real-World Cases

- **Frontend:** Jest/Vitest + React Testing Library.
- **Backend:** Jest/Vitest + Supertest.
- **E2E:** Playwright, Cypress.
- **Libraries:** Vitest, Jest for broad compatibility.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://tc39.es/ecma262/
- MDN Web Docs – Testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing
- MDN Web Docs – Cross-browser testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing/Cross_browser_testing
- Jest Documentation — https://jestjs.io/docs/getting-started
- Vitest Documentation — https://vitest.dev/
- Mocha Documentation — https://mochajs.org/
- Jasmine Documentation — https://jasmine.github.io/
- Node.js Documentation – Test runner — https://nodejs.org/api/test.html
- Node.js Documentation – assert — https://nodejs.org/api/assert.html
- Chai Documentation — https://www.chaijs.com/
- Sinon.JS Documentation — https://sinonjs.org/
- Testing Library Documentation — https://testing-library.com/
- Playwright Documentation — https://playwright.dev/
- Cypress Documentation — https://docs.cypress.io/
- Puppeteer Documentation — https://pptr.dev/
- Selenium Documentation — https://www.selenium.dev/documentation/
- Martin Fowler – Test Pyramid — https://martinfowler.com/bliki/TestPyramid.html
- Martin Fowler – Mocks Aren't Stubs — https://martinfowler.com/articles/mocksArentStubs.html
- Martin Fowler – Test Double — https://martinfowler.com/bliki/TestDouble.html
- Kent Beck – Test-Driven Development: By Example — https://www.oreilly.com/library/view/test-driven-development/0321146530/
- Dan North – Introducing BDD — https://dannorth.net/introducing-bdd/
- Gerard Meszaros – xUnit Test Patterns — http://xunitpatterns.com/
- Kent C. Dodds – The Testing Trophy — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Mike Cohn – The Forgotten Layer of the Test Automation Pyramid — https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid
- Istanbul (nyc) — https://istanbul.js.org/
- c8 — https://github.com/bcoe/c8
- Pact – Contract Testing — https://pact.io/
- Cucumber – BDD — https://cucumber.io/docs/bdd/