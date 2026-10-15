# JavaScript Testing Tools — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Testing tools are the libraries, frameworks, and utilities that support automated testing in JavaScript — providing test structure, assertions, mocking, coverage measurement, browser automation, and continuous integration capabilities.

**Technical Definition:** The JavaScript testing ecosystem is composed of several distinct tool categories: **test runners** (Jest, Vitest, Mocha, AVA, Tape, `node:test`) that discover and execute tests; **assertion libraries** (Chai, Jest/Vitest `expect`, Node.js `assert`) that verify expectations; **mocking libraries** (Sinon, Vitest/Jest built-in) that replace dependencies; **UI testing frameworks** (Testing Library, Enzyme) that render and query components; **E2E frameworks** (Cypress, Playwright, Puppeteer, Selenium) that drive real browsers; and **coverage tools** (Istanbul/nyc, c8) that instrument code to measure execution. Modern tools like Vitest and Jest bundle multiple categories into a single package. The ecosystem is not standardised by ECMAScript; it evolves rapidly and is shaped by community adoption, framework integration, and developer experience. Key standards include the W3C WebDriver protocol (Selenium), the Chrome DevTools Protocol (Puppeteer, Playwright), and the CommonJS/ESM module specifications.

**Beginner-Friendly Explanation:** Testing tools are the software you use to write and run tests. Some tools just run your tests (runners), some check that results are correct (assertions), some fake out dependencies (mocks), some render UI components (Testing Library, Enzyme), some drive real browsers (Cypress, Playwright, Puppeteer), and some measure how much of your code is tested (Istanbul, c8). Modern tools like Jest and Vitest bundle many of these capabilities together, while others like Mocha are more modular and let you pick your own assertion and mocking libraries.

### Key Characteristics

- **Modular ecosystem:** Many tools, each with a specific role.
- **Bundled vs modular:** Jest/Vitest are all-in-one; Mocha/Chai/Sinon are modular.
- **Framework-agnostic and framework-specific:** Some tools target React, Vue, etc.
- **Modern vs legacy:** Vitest, Playwright are modern; Enzyme, Jasmine are older.
- **Speed varies:** Vitest and AVA are fast; Jest is slower but mature.
- **ESM support varies:** Vitest and `node:test` are ESM-first; older tools require configuration.
- **Coverage tools differ:** Istanbul/nyc for Jest, c8 for Vitest and Node.js.
- **Rapid evolution:** Tools change frequently; check compatibility.

### Prerequisites

- Solid understanding of JavaScript fundamentals.
- Familiarity with unit, integration, and E2E testing concepts.
- Knowledge of npm/yarn and `package.json`.
- Awareness of ESM vs CommonJS.
- Basic knowledge of the DOM and browser APIs.

### Related Programming Areas

- **Testing fundamentals:** Unit, integration, E2E.
- **Test-driven development:** Writing tests first.
- **CI/CD:** Running tests on commits.
- **Code coverage:** Measuring test thoroughness.
- **Browser automation:** Driving real browsers.
- **Component testing:** Rendering UI components.

---

## Jest

### Definitions

**Core Definition:** Jest is a batteries-included JavaScript testing framework developed by Meta (Facebook), providing test running, assertions, mocking, snapshots, and coverage in one package.

**Technical Definition:** Jest uses its own test runner, `jest-circus` (or `jest-jasmine2`), and provides a `describe`/`it`/`expect` API. It includes built-in mocking (`jest.fn`, `jest.mock`, `jest.spyOn`), snapshot testing (`toMatchSnapshot`), fake timers, and coverage via Istanbul. Jest uses a `jsdom` environment by default for DOM testing. It is widely used with React and other frameworks.

**Beginner-Friendly Explanation:** Jest is a complete testing toolkit. You install it, write tests with `describe` and `it`, and it handles the rest — running, assertions, mocking, and coverage.

### Purposes

- To provide an all-in-one testing solution.
- To test React components (via `react-test-renderer` or Testing Library).
- To snapshot-test UI output.
- To mock dependencies easily.

### Syntax Rules and Structure

```js
import { describe, it, expect, jest, beforeEach } from "@jest/globals";

describe("Calculator", () => {
  let calc;
  beforeEach(() => { calc = new Calculator(); });

  it("adds", () => {
    expect(calc.add(2, 3)).toBe(5);
  });

  it("mocks functions", () => {
    const mockFn = jest.fn().mockReturnValue(42);
    expect(mockFn()).toBe(42);
    expect(mockFn).toHaveBeenCalled();
  });
});
```

### Annotated Code Examples

```js
// calculator.test.js
import { describe, it, expect, jest } from "@jest/globals";
import { Calculator } from "./calculator.js";
import { logger } from "./logger.js";

jest.mock("./logger.js"); // Auto-mock the logger module

describe("Calculator", () => {
  let calc;
  beforeEach(() => {
    calc = new Calculator();
    jest.clearAllMocks();
  });

  it("adds two numbers", () => {
    expect(calc.add(2, 3)).toBe(5);
  });

  it("logs the operation", () => {
    calc.add(2, 3);
    expect(logger.log).toHaveBeenCalledWith("add", 2, 3, 5);
  });

  it("snapshots the result object", () => {
    expect(calc.add(2, 3)).toMatchSnapshot();
  });
});
```

**Expected Output:**
```
PASS  ./calculator.test.js
  Calculator
    ✓ adds two numbers
    ✓ logs the operation
    ✓ snapshots the result object

Snapshots:   1 passed, 1 total
Tests:       3 passed, 3 total
```

### Real-World Cases

- **React apps:** Create React App, Next.js.
- **Node.js libraries:** Many popular packages.
- **Snapshot testing:** UI components.
- **Legacy projects:** Wide adoption.

---

## Vitest

### Definitions

**Core Definition:** Vitest is a modern, Vite-native testing framework that is fast, ESM-first, and Jest-compatible.

**Technical Definition:** Vitest runs on Vite's dev server, providing instant watch mode and HMR. It uses `vite-node` for transforming ESM and TypeScript. Vitest provides a Jest-compatible API (`describe`, `it`, `expect`, `vi.fn`, `vi.mock`, `vi.spyOn`), snapshot testing, fake timers, and coverage via c8 or Istanbul. It supports browser mode via Playwright/WebDriver.

**Beginner-Friendly Explanation:** Vitest is a modern, fast testing framework that works seamlessly with Vite. It's like Jest but faster and better for ESM and TypeScript.

### Purposes

- To test Vite-based projects.
- To test ESM and TypeScript natively.
- To run tests fast in watch mode.
- To replace Jest in modern projects.

### Syntax Rules and Structure

```js
import { describe, it, expect, vi, beforeEach } from "vitest";

describe("Calculator", () => {
  let calc;
  beforeEach(() => { calc = new Calculator(); });

  it("adds", () => {
    expect(calc.add(2, 3)).toBe(5);
  });

  it("mocks functions", () => {
    const mockFn = vi.fn().mockReturnValue(42);
    expect(mockFn()).toBe(42);
  });
});
```

### Annotated Code Examples

```js
// calculator.test.js
import { describe, it, expect, vi, beforeEach } from "vitest";
import { Calculator } from "./calculator.js";
import { logger } from "./logger.js";

vi.mock("./logger.js");

describe("Calculator", () => {
  let calc;
  beforeEach(() => {
    calc = new Calculator();
    vi.clearAllMocks();
  });

  it("adds two numbers", () => {
    expect(calc.add(2, 3)).toBe(5);
  });

  it("logs the operation", () => {
    calc.add(2, 3);
    expect(logger.log).toHaveBeenCalledWith("add", 2, 3, 5);
  });

  it("snapshots the result", () => {
    expect(calc.add(2, 3)).toMatchSnapshot();
  });
});
```

**Expected Output:**
```
✓ calculator.test.js (3 tests)
  ✓ Calculator > adds two numbers
  ✓ Calculator > logs the operation
  ✓ Calculator > snapshots the result

Test Files  1 passed (1)
Tests       3 passed (3)
```

### Real-World Cases

- **Vite projects:** Vue, React, Svelte, Solid.
- **Modern Node.js:** ESM-first backends.
- **Monorepos:** Fast watch mode.
- **Replacing Jest:** Migration is straightforward.

---

## Mocha

### Definitions

**Core Definition:** Mocha is a flexible, modular JavaScript test framework that provides the test structure (`describe`/`it`) but leaves assertions and mocking to other libraries.

**Technical Definition:** Mocha is a test runner that executes tests in Node.js and browsers. It does not include assertions or mocking; users typically pair it with Chai (assertions) and Sinon (mocks). Mocha supports BDD, TDD, and QUnit interfaces. It is highly configurable and widely used in legacy projects.

**Beginner-Friendly Explanation:** Mocha runs your tests but doesn't include assertions or mocks — you pick your own.

### Purposes

- To provide a flexible test runner.
- To work with any assertion library.
- To test in Node.js and browsers.

### Annotated Code Examples

```js
// calculator.test.js
import { describe, it, beforeEach } from "mocha";
import { expect } from "chai";
import sinon from "sinon";
import { Calculator } from "./calculator.js";
import { logger } from "./logger.js";

describe("Calculator", () => {
  let calc;
  let logSpy;
  beforeEach(() => {
    calc = new Calculator();
    logSpy = sinon.spy(logger, "log");
  });

  it("adds two numbers", () => {
    expect(calc.add(2, 3)).to.equal(5);
  });

  it("logs the operation", () => {
    calc.add(2, 3);
    expect(logSpy.calledWith("add", 2, 3, 5)).to.be.true;
  });
});
```

### Real-World Cases

- **Node.js libraries:** Many npm packages.
- **Legacy projects:** Long-standing infrastructure.
- **Custom setups:** When you want full control.

---

## Jasmine

### Definitions

**Core Definition:** Jasmine is a behavior-driven JavaScript testing framework with built-in assertions, spies, and mocking.

**Technical Definition:** Jasmine provides `describe`/`it`/`expect`, a rich matcher library (`toBe`, `toEqual`, `toContain`), spies (`spyOn`), and async support. It runs in Node.js and browsers. Jasmine is older than Jest but still used, especially in Angular projects.

**Beginner-Friendly Explanation:** Jasmine is a self-contained testing framework — you don't need additional assertion or mocking libraries.

### Annotated Code Examples

```js
describe("Calculator", () => {
  let calc;
  beforeEach(() => { calc = new Calculator(); });

  it("adds two numbers", () => {
    expect(calc.add(2, 3)).toBe(5);
  });

  it("spies on a method", () => {
    const spy = spyOn(calc, "add").and.returnValue(10);
    expect(calc.add(2, 3)).toBe(10);
    expect(spy).toHaveBeenCalledWith(2, 3);
  });
});
```

### Real-World Cases

- **Angular:** Default test framework (older versions).
- **Legacy projects:** Long-standing use.
- **Browser testing:** Works without a build step.

---

## AVA

### Definitions

**Core Definition:** AVA is a fast, concurrent test runner for Node.js with built-in assertions and ESM support.

**Technical Definition:** AVA runs tests concurrently, provides `test`/`t` assertion API, and supports ESM natively. It does not use `describe`/`it`; instead, each test is a top-level `test()` call. AVA is minimal and fast but less popular than Jest/Vitest.

**Beginner-Friendly Explanation:** AVA is a fast, minimal test runner that runs tests concurrently.

### Annotated Code Examples

```js
import test from "ava";

test("adds two numbers", (t) => {
  t.is(add(2, 3), 5);
});

test("handles async", async (t) => {
  const result = await fetchData();
  t.is(result, "done");
});
```

### Real-World Cases

- **Node.js libraries:** Fast, minimal test suites.
- **ESM projects:** Native ESM support.
- **Concurrent testing:** Speedy test runs.

---

## Tape

### Definitions

**Core Definition:** Tape is a minimal, tap-producing test harness for Node.js and browsers.

**Technical Definition:** Tape produces TAP (Test Anything Protocol) output, has no globals, and uses a simple API (`test`, `t.equal`, `t.deepEqual`, `t.plan`). It is extremely minimal and often used for small libraries.

**Beginner-Friendly Explanation:** Tape is a tiny test library that outputs TAP-compatible results.

### Annotated Code Examples

```js
import test from "tape";

test("addition", (t) => {
  t.equal(add(2, 3), 5);
  t.end();
});

test("async", (t) => {
  t.plan(1);
  fetchData().then((result) => {
    t.equal(result, "done");
  });
});
```

### Real-World Cases

- **Small libraries:** Minimal dependencies.
- **TAP consumers:** CI systems that parse TAP.

---

## Node.js Test Runner

### Definitions

**Core Definition:** The Node.js test runner (`node:test`) is a built-in test framework in Node.js 18+, providing `test`/`describe`/`it` structure with no external dependencies.

**Technical Definition:** `node:test` provides `test()`, `describe()`, `it()`, `before`/`after` hooks, `mock` utilities (`t.mock.fn`), and assertions via `node:assert`. It supports ESM and CommonJS, runs tests concurrently, and integrates with `node --test`. Coverage is available via `--experimental-test-coverage`.

**Beginner-Friendly Explanation:** Node.js has a built-in test runner — no need to install Jest or Mocha for simple Node projects.

### Annotated Code Examples

```js
// calculator.test.js
import { test, describe, beforeEach, mock } from "node:test";
import assert from "node:assert/strict";
import { Calculator } from "./calculator.js";

describe("Calculator", () => {
  let calc;
  beforeEach(() => { calc = new Calculator(); });

  test("adds two numbers", () => {
    assert.strictEqual(calc.add(2, 3), 5);
  });

  test("mocks a function", () => {
    const mockFn = mock.fn(() => 42);
    assert.strictEqual(mockFn(), 42);
    assert.strictEqual(mockFn.mock.callCount(), 1);
  });

  test("async", async () => {
    const result = await calc.asyncAdd(2, 3);
    assert.strictEqual(result, 5);
  });
});
```

**Run with:**
```bash
node --test
```

### Real-World Cases

- **Node.js libraries:** Zero dependencies.
- **Scripts:** Simple tests without setup.
- **Modern Node.js:** ESM-first.

---

## Cypress

### Definitions

**Core Definition:** Cypress is a JavaScript E2E testing framework that runs tests inside the browser with a focus on developer experience.

**Technical Definition:** Cypress executes tests in the same run loop as the application, providing automatic waiting, time-travel debugging, network stubbing, and a rich interactive UI. It runs Chrome, Edge, Electron, and Firefox. It supports E2E and component testing. Cypress is JavaScript-only and does not use WebDriver.

**Beginner-Friendly Explanation:** Cypress is an E2E testing tool with excellent debugging and automatic waiting.

### Annotated Code Examples

```js
// cypress/e2e/login.cy.js
describe("Login", () => {
  beforeEach(() => cy.visit("/login"));

  it("logs in successfully", () => {
    cy.get('[data-testid="email"]').type("alice@example.com");
    cy.get('[data-testid="password"]').type("secret");
    cy.get('[data-testid="submit"]').click();
    cy.url().should("include", "/dashboard");
    cy.contains("Welcome, Alice").should("be.visible");
  });

  it("shows error on invalid credentials", () => {
    cy.get('[data-testid="email"]').type("alice@example.com");
    cy.get('[data-testid="password"]').type("wrong");
    cy.get('[data-testid="submit"]').click();
    cy.get(".error").should("contain", "Invalid credentials");
  });
});
```

### Real-World Cases

- **SPAs:** React, Vue, Angular.
- **Fast iteration:** Excellent local dev experience.
- **Component testing:** In-browser component tests.

---

## Playwright

### Definitions

**Core Definition:** Playwright is a modern cross-browser E2E testing framework from Microsoft, supporting Chromium, Firefox, and WebKit with a single API.

**Technical Definition:** Playwright drives browsers via CDP (Chromium) and equivalent protocols (Firefox, WebKit). It provides auto-waiting, network interception, tracing, video, and multi-context support. Available for JavaScript/TypeScript, Python, Java, and .NET. It is the most feature-complete E2E framework as of 2025.

**Beginner-Friendly Explanation:** Playwright is a modern E2E tool that tests all major browsers with the same code.

### Annotated Code Examples

```js
// tests/login.spec.js
import { test, expect } from "@playwright/test";

test.describe("Login", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/login");
  });

  test("logs in successfully", async ({ page }) => {
    await page.fill('[data-testid="email"]', "alice@example.com");
    await page.fill('[data-testid="password"]', "secret");
    await page.click('[data-testid="submit"]');
    await expect(page).toHaveURL(/dashboard/);
    await expect(page.locator("h1")).toContainText("Welcome");
  });

  test("shows error on invalid credentials", async ({ page }) => {
    await page.fill('[data-testid="email"]', "alice@example.com");
    await page.fill('[data-testid="password"]', "wrong");
    await page.click('[data-testid="submit"]');
    await expect(page.locator(".error")).toHaveText("Invalid credentials");
  });
});
```

### Real-World Cases

- **Cross-browser testing:** Chrome, Firefox, Safari.
- **CI pipelines:** Parallel, headless.
- **Debugging:** Trace viewer, videos.

---

## Puppeteer

### Definitions

**Core Definition:** Puppeteer is a Node.js library for controlling Chrome/Chromium via the Chrome DevTools Protocol.

**Technical Definition:** Puppeteer launches or connects to Chrome, providing APIs for navigation, evaluation, screenshots, PDFs, and network interception. It is not a test runner; combine with Jest, Vitest, or `node:test`. Puppeteer is Chrome-only (though `puppeteer-firefox` existed historically).

**Beginner-Friendly Explanation:** Puppeteer is a library for controlling Chrome programmatically — for scraping, screenshots, PDFs, and testing.

### Annotated Code Examples

```js
import puppeteer from "puppeteer";
import { test, expect, beforeAll, afterAll } from "vitest";

let browser, page;
beforeAll(async () => {
  browser = await puppeteer.launch({ headless: true });
  page = await browser.newPage();
});
afterAll(async () => { await browser.close(); });

test("renders the homepage", async () => {
  await page.goto("http://localhost:3000");
  const title = await page.title();
  expect(title).toBe("My App");
});
```

### Real-World Cases

- **Scraping:** Extract data.
- **PDF generation:** HTML → PDF.
- **Performance audits:** Lighthouse.
- **Chrome-specific testing.**

---

## Testing Library

### Definitions

**Core Definition:** Testing Library is a family of libraries for testing UI components in a user-centric way, querying the DOM by roles, labels, and text rather than implementation details.

**Technical Definition:** Testing Library provides `render`, `screen`, `fireEvent`, `userEvent`, and queries (`getByRole`, `getByLabelText`, `getByText`, `getByTestId`). It is framework-agnostic at its core, with adapters for React, Vue, Svelte, Angular, and more. It encourages testing accessibility and user behaviour, not internal state.

**Beginner-Friendly Explanation:** Testing Library tests components the way a user would — by looking for text, labels, and roles — not by inspecting internal state.

### Annotated Code Examples

```jsx
// LoginForm.test.jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { LoginForm } from "./LoginForm";

test("submits the form", async () => {
  const onSubmit = vi.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  await userEvent.type(screen.getByLabelText("Email"), "alice@example.com");
  await userEvent.type(screen.getByLabelText("Password"), "secret");
  await userEvent.click(screen.getByRole("button", { name: /log in/i }));

  expect(onSubmit).toHaveBeenCalledWith({
    email: "alice@example.com",
    password: "secret",
  });
});
```

### Real-World Cases

- **React:** `@testing-library/react`.
- **Vue:** `@testing-library/vue`.
- **Svelte:** `@testing-library/svelte`.
- **Accessibility:** Encourages accessible markup.

---

## Enzyme

### Definitions

**Core Definition:** Enzyme is a React testing utility that allows shallow, mount, and render rendering of components.

**Technical Definition:** Enzyme provides `shallow`, `mount`, and `render` methods, plus a jQuery-like API for traversing and asserting on the component tree. It was widely used before Testing Library. Enzyme is not actively maintained for React 17+ (though `@wojtekmaj/enzyme-adapter-react-17` exists). It is considered legacy.

**Beginner-Friendly Explanation:** Enzyme was the old way to test React components; Testing Library is now preferred.

### Status

**Legacy:** Enzyme's official adapter is not maintained for React 18+. Migrate to Testing Library.

### Annotated Code Examples

```jsx
import { shallow } from "enzyme";
import { Button } from "./Button";

it("renders the label", () => {
  const wrapper = shallow(<Button label="Click" />);
  expect(wrapper.text()).toBe("Click");
});
```

### Real-World Cases

- **Legacy React projects:** Pre-2021 codebases.
- **Migration targets:** Moving to Testing Library.

---

## Sinon

### Definitions

**Core Definition:** Sinon is a standalone mocking library providing spies, stubs, mocks, and fake timers.

**Technical Definition:** Sinon provides `sinon.spy()`, `sinon.stub()`, `sinon.mock()`, `sinon.useFakeTimers()`, and `sinon.createSandbox()`. It is framework-agnostic and works with any test runner. It is often paired with Mocha and Chai.

**Beginner-Friendly Explanation:** Sinon is a library for creating spies, stubs, and mocks in any test runner.

### Annotated Code Examples

```js
import { describe, it, beforeEach, afterEach } from "mocha";
import { expect } from "chai";
import sinon from "sinon";
import { UserService } from "./userService.js";

describe("UserService", () => {
  let sandbox, mockRepo, service;

  beforeEach(() => {
    sandbox = sinon.createSandbox();
    mockRepo = { findById: sandbox.stub() };
    service = new UserService(mockRepo);
  });

  afterEach(() => sandbox.restore());

  it("returns a user", async () => {
    mockRepo.findById.resolves({ id: 1, name: "Alice" });
    const user = await service.getUser(1);
    expect(user.name).to.equal("Alice");
    expect(mockRepo.findById.calledWith(1)).to.be.true;
  });
});
```

### Real-World Cases

- **Mocha projects:** Default mocking library.
- **Framework-agnostic:** Any runner.
- **Fake timers:** Testing timers.

---

## Chai

### Definitions

**Core Definition:** Chai is a BDD/TDD assertion library with `expect`, `should`, and `assert` styles.

**Technical Definition:** Chai provides chainable assertions (`expect(x).to.be.a("string")`, `expect(x).to.equal(5)`). It has plugins like `chai-as-promised` for Promises and `chai-dom` for DOM. It is often paired with Mocha.

**Beginner-Friendly Explanation:** Chai is an assertion library that works with any test runner.

### Annotated Code Examples

```js
import { expect } from "chai";
import chaiAsPromised from "chai-as-promised";
import chai from "chai";
chai.use(chaiAsPromised);

expect(5).to.equal(5);
expect("hello").to.be.a("string");
expect([1, 2, 3]).to.include(2);
await expect(fetchData()).to.eventually.equal("done");
```

### Real-World Cases

- **Mocha projects:** Standard pairing.
- **Custom assertions:** Plugins.
- **BDD style:** Readable assertions.

---

## Expect

### Definitions

**Core Definition:** "Expect" refers to the assertion style used by Jest, Vitest, Chai, and others.

**Technical Definition:** `expect(value)` creates an expectation object with matchers (`.toBe`, `.toEqual`, `.toThrow`, `.toHaveBeenCalledWith`). Jest/Vitest provide their own `expect`; Chai provides an `expect` style. The name is also used loosely to mean "assertion library."

**Beginner-Friendly Explanation:** "Expect" is the `expect(...).toBe(...)` style of asserting in tests.

### Annotated Code Examples

```js
// Jest/Vitest expect
expect(5).toBe(5);
expect({ a: 1 }).toEqual({ a: 1 });
expect(() => { throw new Error("boom"); }).toThrow("boom");
expect(fn).toHaveBeenCalledWith("arg");
```

### Real-World Cases

- **Jest/Vitest:** Built-in.
- **Chai:** `expect` style.

---

## Istanbul

### Definitions

**Core Definition:** Istanbul is a JavaScript code coverage tool that instruments code to track which lines, branches, and functions are executed.

**Technical Definition:** Istanbul (via `nyc` or Jest's built-in coverage) instruments source code and produces coverage reports (HTML, LCOV, text, JSON). It measures statements, branches, functions, and lines. Modern alternatives include c8 (which uses V8's built-in coverage).

**Beginner-Friendly Explanation:** Istanbul measures how much of your code is executed by tests.

### Annotated Code Examples

```bash
# With nyc (Istanbul CLI)
nyc --reporter=html --reporter=text npm test

# With Jest
jest --coverage
```

**Expected Output (text):**
```
File         | % Stmts | % Branch | % Funcs | % Lines
-------------|---------|----------|---------|--------
calculator.js|   100   |   80     |   100   |   100
```

### Real-World Cases

- **Jest coverage:** Built-in.
- **nyc:** CLI for any test runner.
- **CI gates:** Enforce minimum coverage.

---

## c8

### Definitions

**Core Definition:** c8 is a code coverage tool that uses V8's built-in coverage instrumentation, avoiding source-code instrumentation.

**Technical Definition:** c8 uses the `NODE_V8_COVERAGE` environment variable and V8's built-in coverage data. It is faster and more accurate than Istanbul for Node.js. It is the default coverage tool for Vitest and recommended for `node:test`.

**Beginner-Friendly Explanation:** c8 measures coverage using V8's built-in capabilities, making it faster than Istanbul.

### Annotated Code Examples

```bash
c8 node --test
c8 --reporter=html --reporter=text npm test
```

**Expected Output (text):**
```
File         | % Stmts | % Branch | % Funcs | % Lines
-------------|---------|----------|---------|--------
calculator.js|   100   |   80     |   100   |   100
```

### Real-World Cases

- **Node.js projects:** Fast coverage.
- **Vitest:** Default coverage provider.
- **`node:test`:** Recommended coverage tool.

---

## nyc

### Definitions

**Core Definition:** nyc is the command-line interface for Istanbul, providing coverage for any test runner.

**Technical Definition:** nyc wraps a test command, instruments source files, and produces coverage reports. It supports subprocess coverage, caching, and many reporters. It is widely used with Mocha but works with any runner.

**Beginner-Friendly Explanation:** nyc is the CLI for Istanbul coverage — it works with any test runner.

### Annotated Code Examples

```json
// package.json
{
  "scripts": {
    "test": "mocha",
    "coverage": "nyc npm test"
  },
  "nyc": {
    "reporter": ["text", "html", "lcov"],
    "exclude": ["**/*.test.js", "**/node_modules/**"]
  }
}
```

### Real-World Cases

- **Mocha projects:** Standard coverage.
- **Any runner:** Works with Jest, Vitest, `node:test`.
- **CI gates:** Enforce coverage thresholds.

---

## References

- Jest Documentation — https://jestjs.io/docs/getting-started
- Vitest Documentation — https://vitest.dev/
- Mocha Documentation — https://mochajs.org/
- Jasmine Documentation — https://jasmine.github.io/
- AVA Documentation — https://github.com/avajs/ava
- Tape Documentation — https://github.com/ljharb/tape
- Node.js Documentation – Test runner — https://nodejs.org/api/test.html
- Cypress Documentation — https://docs.cypress.io/
- Playwright Documentation — https://playwright.dev/
- Puppeteer Documentation — https://pptr.dev/
- Testing Library Documentation — https://testing-library.com/
- Enzyme Documentation — https://enzymejs.github.io/enzyme/
- Sinon.JS Documentation — https://sinonjs.org/
- Chai Documentation — https://www.chaijs.com/
- Istanbul Documentation — https://istanbul.js.org/
- c8 Documentation — https://github.com/bcoe/c8
- nyc Documentation — https://github.com/istanbuljs/nyc
- MDN Web Docs – Testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing
- W3C WebDriver Specification — https://www.w3.org/TR/webdriver2/
- Chrome DevTools Protocol — https://chromedevtools.github.io/devtools-protocol/
- Testing Library – Priority of Queries — https://testing-library.com/docs/queries/about/#priority
- Kent C. Dodds – Testing Trophy — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Martin Fowler – Test Double — https://martinfowler.com/bliki/TestDouble.html
- Martin Fowler – Mocks Aren't Stubs — https://martinfowler.com/articles/mocksArentStubs.html