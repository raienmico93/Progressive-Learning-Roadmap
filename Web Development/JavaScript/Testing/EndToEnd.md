# JavaScript End-to-End Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** End-to-end (E2E) testing is the practice of verifying that an entire application works correctly from the user's perspective, driving a real browser (or client) through complete user flows from the UI through all backend layers to the database and back.

**Technical Definition:** E2E tests exercise the full application stack: browser rendering, client-side JavaScript, HTTP requests, server logic, database queries, and external services. They use browser automation frameworks — Playwright, Cypress, Puppeteer, Selenium/WebDriver — that control a real (or headless) browser via a protocol: the W3C WebDriver standard for Selenium, the Chrome DevTools Protocol (CDP) for Puppeteer and Playwright (which also supports Firefox and WebKit), or a custom in-browser runtime for Cypress. E2E tests are the top of the test pyramid: fewest in number, slowest to run, but highest confidence. They are the only tests that can catch integration issues across the entire stack — misconfigured routing, broken authentication, or CSS that hides a button.

**Beginner-Friendly Explanation:** E2E testing is like having a robot user test your entire app. The robot opens a real browser, navigates to your site, clicks buttons, fills out forms, and checks that everything works — from the button you click all the way to the database that stores the data. It's the closest thing to a real user using your app. Because E2E tests are slow and complex, you write fewer of them — usually covering the most critical user journeys like login, checkout, or signup.

### Key Characteristics

- **Full-stack:** Exercises the entire application, not just one layer.
- **Real browser:** Runs in Chrome, Firefox, WebKit, or Edge.
- **Real network:** Makes real HTTP requests (or intercepts them).
- **User-centric:** Describes flows in user terms (click, type, navigate).
- **Slow and brittle:** Seconds to minutes per test; sensitive to timing and environment.
- **Fewest in number:** Top of the test pyramid (~5–10% of tests).
- **High confidence:** Catches integration issues nothing else can.
- **Environment-dependent:** Requires a running app, database, and services.

### Prerequisites

- Solid understanding of unit and integration testing.
- Familiarity with the DOM, HTML forms, and CSS selectors.
- Knowledge of HTTP, cookies, and authentication.
- Experience with asynchronous code and `async/await`.
- Awareness of CI/CD pipelines.

### Related Programming Areas

- **Unit testing:** The layer below.
- **Integration testing:** The middle layer.
- **Contract testing:** Service agreements.
- **Visual regression:** Screenshot comparisons.
- **Accessibility testing:** a11y checks with axe.
- **Performance testing:** Lighthouse CI, WebPageTest.

---

## E2E Testing

### Definitions

**Core Definition:** E2E testing verifies complete user journeys through the entire application stack.

**Technical Definition:** An E2E test drives a real browser through a sequence of user actions (navigation, clicks, typing, waits), asserting on visible outcomes. The test runs against a deployed or locally served application with real (or containerised) backend services. The browser communicates with the test runner via a protocol (CDP, WebDriver, or a custom channel). Assertions check DOM state, URLs, network responses, and visual appearance.

**Beginner-Friendly Explanation:** E2E testing means testing your whole app end to end — from the user clicking a button to the data being saved and shown back to them.

### Purposes

- To verify critical user journeys.
- To catch full-stack integration issues.
- To validate deployments.
- To prevent regressions in critical flows.
- To ensure cross-browser compatibility.

### Syntax Rules and Structure

```js
import { test, expect } from "@playwright/test";

test("user can log in", async ({ page }) => {
  await page.goto("/login");
  await page.fill('input[name="email"]', "alice@example.com");
  await page.fill('input[name="password"]', "secret");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
  await expect(page.locator("h1")).toHaveText("Welcome, Alice");
});
```

### Annotated Code Examples

#### Example 1: Playwright E2E Test

```js
// tests/login.spec.js
import { test, expect } from "@playwright/test";

test.describe("Authentication", () => {
  test("user can log in with valid credentials", async ({ page }) => {
    await page.goto("https://app.example.com/login");
    await page.fill('input[name="email"]', "alice@example.com");
    await page.fill('input[name="password"]', "correct-password");
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL(/dashboard/);
    await expect(page.locator("h1")).toContainText("Welcome");
  });

  test("shows an error for invalid credentials", async ({ page }) => {
    await page.goto("https://app.example.com/login");
    await page.fill('input[name="email"]', "alice@example.com");
    await page.fill('input[name="password"]', "wrong");
    await page.click('button[type="submit"]');

    await expect(page.locator(".error")).toHaveText("Invalid credentials");
  });
});
```

**Expected Output:**
```
✓ Authentication > user can log in with valid credentials
✓ Authentication > shows an error for invalid credentials
```

**Why This Output:** Playwright drives a real browser; assertions wait for the DOM to reach the expected state.

#### Example 2: Checkout Flow

```js
test("user can complete checkout", async ({ page }) => {
  await page.goto("/products");
  await page.click('button[data-product="widget"]');
  await page.click("text=Add to cart");
  await page.click("text=Checkout");

  await page.fill('input[name="card"]', "4242424242424242");
  await page.fill('input[name="expiry"]', "12/30");
  await page.fill('input[name="cvc"]', "123");
  await page.click("text=Pay");

  await expect(page.locator(".confirmation")).toContainText("Order confirmed");
});
```

### Real-World Cases

- **Authentication:** Login, signup, password reset.
- **E-commerce:** Product browse, cart, checkout.
- **SaaS:** Onboarding, subscription, settings.
- **Critical flows:** Anything that must not break.

---

## Cypress

### Definitions

**Core Definition:** Cypress is a JavaScript E2E testing framework that runs tests inside the browser, providing fast, reliable, and debuggable tests.

**Technical Definition:** Cypress executes tests in the same run loop as the application, inside the browser. It uses a custom architecture (not WebDriver) with a Node.js server that proxies the browser. Cypress provides automatic waiting, time-travel debugging, real-time reloads, and network stubbing. It supports Chrome, Edge, Electron, and Firefox. Cypress is JavaScript-only.

**Beginner-Friendly Explanation:** Cypress is an E2E testing tool that runs inside your browser. It's famous for its great debugging experience and automatic waiting.

### Purposes

- To write fast, reliable E2E tests.
- To debug tests with time-travel.
- To stub network requests easily.
- To run component tests as well.

### Syntax Rules and Structure

```js
describe("Login", () => {
  it("logs in", () => {
    cy.visit("/login");
    cy.get('input[name="email"]').type("alice@example.com");
    cy.get('input[name="password"]').type("secret");
    cy.get('button[type="submit"]').click();
    cy.url().should("include", "/dashboard");
  });
});
```

### Annotated Code Examples

```js
// cypress/e2e/login.cy.js
describe("Authentication", () => {
  beforeEach(() => {
    cy.visit("/login");
  });

  it("logs in with valid credentials", () => {
    cy.get('input[name="email"]').type("alice@example.com");
    cy.get('input[name="password"]').type("correct-password");
    cy.get('button[type="submit"]').click();
    cy.url().should("include", "/dashboard");
    cy.contains("Welcome, Alice").should("be.visible");
  });

  it("shows an error for invalid credentials", () => {
    cy.get('input[name="email"]').type("alice@example.com");
    cy.get('input[name="password"]').type("wrong");
    cy.get('button[type="submit"]').click();
    cy.get(".error").should("contain", "Invalid credentials");
  });
});
```

**Expected Output:**
```
Authentication
  ✓ logs in with valid credentials
  ✓ shows an error for invalid credentials
```

### Real-World Cases

- **Frontend-heavy apps:** SPAs, dashboards.
- **Rapid iteration:** Fast feedback in dev.
- **Debugging:** Time-travel through test steps.

---

## Playwright

### Definitions

**Core Definition:** Playwright is a cross-browser E2E testing framework from Microsoft that supports Chromium, Firefox, and WebKit with a single API.

**Technical Definition:** Playwright drives browsers via the Chrome DevTools Protocol (for Chromium) and equivalent protocols for Firefox and WebKit. It supports headless and headed modes, multiple browser contexts, network interception, tracing, video recording, and auto-waiting. Playwright is available for JavaScript/TypeScript, Python, Java, and .NET. It is the most feature-complete E2E framework as of 2025.

**Beginner-Friendly Explanation:** Playwright is a modern E2E tool that tests Chrome, Firefox, and Safari with the same code. It's fast and has great debugging tools.

### Purposes

- To test across all major browsers.
- To run tests in parallel.
- To debug with traces and videos.
- To test mobile viewports.
- To test APIs alongside UI.

### Syntax Rules and Structure

```js
import { test, expect } from "@playwright/test";

test("homepage has title", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveTitle(/My App/);
});
```

### Annotated Code Examples

#### Example 1: Multi-Browser Test

```js
// playwright.config.js
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit", use: { ...devices["Desktop Safari"] } },
    { name: "Mobile Chrome", use: { ...devices["Pixel 5"] } },
  ],
});
```

```js
// tests/home.spec.js
import { test, expect } from "@playwright/test";

test("homepage renders", async ({ page }) => {
  await page.goto("/");
  await expect(page.locator("h1")).toHaveText("Welcome");
});
```

**Expected Output:** Runs the same test across Chromium, Firefox, WebKit, and mobile viewports.

#### Example 2: Network Interception

```js
test("shows error when API fails", async ({ page }) => {
  await page.route("/api/users", (route) =>
    route.fulfill({ status: 500, body: "Server error" })
  );
  await page.goto("/users");
  await expect(page.locator(".error")).toContainText("Failed to load");
});
```

#### Example 3: Tracing for Debugging

```js
// playwright.config.js
export default defineConfig({
  use: {
    trace: "on-first-retry", // Record trace on first retry
    video: "retain-on-failure",
    screenshot: "only-on-failure",
  },
});
```

### Real-World Cases

- **Cross-browser testing:** Verify Chrome, Firefox, Safari.
- **CI pipelines:** Parallel, headless execution.
- **Debugging:** Trace viewer, videos.
- **Accessibility:** Integrate axe-core.

---

## Puppeteer

### Definitions

**Core Definition:** Puppeteer is a Node.js library that provides a high-level API to control Chrome or Chromium over the Chrome DevTools Protocol.

**Technical Definition:** Puppeteer launches or connects to a Chrome/Chromium browser and controls it via CDP. It provides APIs for navigation, evaluation, screenshots, PDFs, network interception, and performance tracing. Puppeteer is not a test runner; it's a browser automation library that can be combined with Jest, Vitest, or `node:test`.

**Beginner-Friendly Explanation:** Puppeteer is a library for controlling Chrome programmatically. It's often used for scraping, automation, and testing.

### Purposes

- To automate Chrome/Chromium.
- To scrape or generate screenshots/PDFs.
- To test Chrome-specific behaviour.
- To run performance audits.

### Annotated Code Examples

```js
import puppeteer from "puppeteer";
import { describe, it, expect, beforeAll, afterAll } from "vitest";

describe("Homepage", () => {
  let browser, page;
  beforeAll(async () => {
    browser = await puppeteer.launch({ headless: true });
    page = await browser.newPage();
  });
  afterAll(async () => { await browser.close(); });

  it("renders the title", async () => {
    await page.goto("http://localhost:3000");
    const title = await page.title();
    expect(title).toBe("My App");
  });

  it("can click a button", async () => {
    await page.click("#submit");
    const text = await page.$eval("#result", (el) => el.textContent);
    expect(text).toBe("Submitted");
  });
});
```

### Real-World Cases

- **Scraping:** Extract data from pages.
- **PDF generation:** Render HTML to PDF.
- **Performance:** Run Lighthouse.
- **Testing:** Chrome-specific tests.

---

## Selenium

### Definitions

**Core Definition:** Selenium is the original browser automation framework, providing bindings for many languages including JavaScript (via `selenium-webdriver`).

**Technical Definition:** Selenium uses the W3C WebDriver standard to control browsers. Each browser has a WebDriver implementation (ChromeDriver, GeckoDriver, SafariDriver). Selenium supports Java, Python, C#, Ruby, JavaScript, and Kotlin. It is the most widely adopted E2E framework in enterprise environments.

**Beginner-Friendly Explanation:** Selenium is the oldest and most widely used browser automation tool. It works with many languages and browsers.

### Purposes

- To automate browsers across languages.
- To test in enterprise environments.
- To integrate with existing test infrastructure.

### Annotated Code Examples

```js
import { Builder, By, until } from "selenium-webdriver";
import { describe, it, expect, beforeAll, afterAll } from "vitest";

describe("Homepage", () => {
  let driver;
  beforeAll(async () => {
    driver = await new Builder().forBrowser("chrome").build();
  });
  afterAll(async () => { await driver.quit(); });

  it("renders the title", async () => {
    await driver.get("http://localhost:3000");
    expect(await driver.getTitle()).toBe("My App");
  });

  it("can click a button", async () => {
    await driver.findElement(By.id("submit")).click();
    const result = await driver.wait(until.elementLocated(By.id("result")));
    expect(await result.getText()).toBe("Submitted");
  });
});
```

### Real-World Cases

- **Enterprise:** Large-scale cross-browser testing.
- **Multi-language teams:** Java, Python, C#.
- **Legacy systems:** Existing Selenium infrastructure.

---

## WebDriver

### Definitions

**Core Definition:** WebDriver is a W3C standard protocol for controlling browsers programmatically.

**Technical Definition:** The W3C WebDriver specification defines a REST-like protocol where a client (test code) sends HTTP commands to a WebDriver server (browser driver) that controls the browser. Commands include navigation, element lookup, interaction, and script execution. Selenium uses WebDriver. Playwright and Puppeteer use CDP instead, but Playwright also supports connecting via WebDriver BiDi (a bidirectional WebDriver protocol).

**Beginner-Friendly Explanation:** WebDriver is the standard "language" that test tools use to talk to browsers. Selenium speaks WebDriver; Playwright and Puppeteer speak CDP.

### Purposes

- To provide a standard browser automation protocol.
- To decouple test code from browser internals.
- To enable cross-browser automation.

### Real-World Cases

- **Selenium:** Primary WebDriver client.
- **WebDriver BiDi:** Next-gen bidirectional protocol.
- **Appium:** WebDriver for mobile apps.

---

## Test Scenarios

### Definitions

**Core Definition:** A test scenario is a description of a user journey or use case that an E2E test verifies.

**Technical Definition:** Test scenarios are high-level descriptions of what a user does and what should happen. They are decomposed into steps (Given/When/Then or Arrange/Act/Assert). Scenarios focus on user-visible behaviour, not implementation details.

**Beginner-Friendly Explanation:** A test scenario is a story about what a user does — like "A user logs in, adds an item to the cart, and checks out."

### Purposes

- To define what to test.
- To prioritise critical flows.
- To communicate with stakeholders.
- To structure test suites.

### Annotated Code Examples

```js
// Scenario: User registers a new account
// Given I am on the signup page
// When I fill in valid details and submit
// Then I see the dashboard

test("User registers a new account", async ({ page }) => {
  await page.goto("/signup");
  await page.fill('input[name="name"]', "Alice");
  await page.fill('input[name="email"]', "alice@example.com");
  await page.fill('input[name="password"]', "secure-password");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
});
```

### Real-World Cases

- **Critical paths:** Login, checkout, payment.
- **Edge cases:** Invalid input, network failure.
- **User stories:** Map to E2E scenarios.

---

## Page Objects

### Definitions

**Core Definition:** The Page Object pattern encapsulates page structure and interactions in a class or module, decoupling tests from selectors.

**Technical Definition:** A Page Object represents a page (or component) and exposes methods for user actions (`login(email, password)`) and queries (`getErrorMessage()`). Selectors are defined once inside the Page Object. When the UI changes, only the Page Object needs updating. Page Objects improve maintainability and readability.

**Beginner-Friendly Explanation:** A Page Object is a class that represents a page in your app. Instead of scattering selectors throughout tests, you put them in one place.

### Purposes

- To reduce selector duplication.
- To improve test maintainability.
- To abstract page structure.
- To make tests readable.

### Annotated Code Examples

```js
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.locator('input[name="email"]');
    this.passwordInput = page.locator('input[name="password"]');
    this.submitButton = page.locator('button[type="submit"]');
    this.errorMessage = page.locator(".error");
  }

  async goto() { await this.page.goto("/login"); }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getError() { return this.errorMessage.textContent(); }
}
```

```js
// tests/login.spec.js
import { test, expect } from "@playwright/test";
import { LoginPage } from "../pages/LoginPage.js";

test("logs in successfully", async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login("alice@example.com", "secret");
  await expect(page).toHaveURL("/dashboard");
});

test("shows error on invalid credentials", async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login("alice@example.com", "wrong");
  expect(await loginPage.getError()).toBe("Invalid credentials");
});
```

**Expected Output:**
```
✓ logs in successfully
✓ shows error on invalid credentials
```

### Real-World Cases

- **Large test suites:** Centralise selectors.
- **Frequent UI changes:** Update one file.
- **Team collaboration:** Shared page abstractions.

---

## Test Data

### Definitions

**Core Definition:** Test data is the input values and state used by E2E tests.

**Technical Definition:** Test data includes user credentials, form values, database seeds, and API fixtures. Best practices: use factories or builders for dynamic data, isolate test data per test, seed the database before tests, and clean up after. Avoid hardcoded production data.

**Beginner-Friendly Explanation:** Test data is the fake data your tests use — like a fake user account.

### Purposes

- To provide consistent inputs.
- To isolate tests from production data.
- To enable reproducibility.
- To test edge cases.

### Annotated Code Examples

#### Example 1: Data Factories

```js
// factories/userFactory.js
export function createUser(overrides = {}) {
  const id = Math.random().toString(36).slice(2, 8);
  return {
    name: `User ${id}`,
    email: `user-${id}@example.com`,
    password: "test-password",
    ...overrides,
  };
}
```

```js
test("creates a user", async ({ page }) => {
  const user = createUser();
  await page.goto("/signup");
  await page.fill('input[name="name"]', user.name);
  await page.fill('input[name="email"]', user.email);
  await page.fill('input[name="password"]', user.password);
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
});
```

#### Example 2: Database Seeding

```js
// fixtures/seed.js
export async function seedUsers(db, count = 3) {
  const users = [];
  for (let i = 0; i < count; i++) {
    users.push(await db.users.create(createUser()));
  }
  return users;
}
```

#### Example 3: API Fixtures

```js
// fixtures/api.js
export const mockUsers = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob", email: "bob@example.com" },
];
```

### Real-World Cases

- **Database seeding:** Pre-populate test DB.
- **API fixtures:** MSW, json-server.
- **Form data:** Generate unique values per test.

---

## Test Environments

### Definitions

**Core Definition:** A test environment is the infrastructure (browser, server, database, services) in which E2E tests run.

**Technical Definition:** Test environments typically include: a browser (headless or headed), a running application server, a database (real or in-memory), external service stubs, and a test runner. Environments may be local, containerised (Docker Compose), or cloud-based (GitHub Actions, CircleCI, BrowserStack, Sauce Labs). Best practices: use a dedicated test environment, ensure isolation, and reset state between runs.

**Beginner-Friendly Explanation:** A test environment is the setup where your E2E tests run — the browser, the app, the database, and the services.

### Components

- **Browser:** Chrome, Firefox, WebKit.
- **App server:** Dev server or production build.
- **Database:** PostgreSQL, MongoDB, SQLite.
- **External services:** Stripe, SendGrid (stubbed).
- **CI/CD:** GitHub Actions, GitLab CI.

### Annotated Code Examples

#### Example 1: Docker Compose for E2E

```yaml
# docker-compose.test.yml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://test:test@db:5432/test
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: test
    ports:
      - "5432:5432"
```

#### Example 2: Playwright Config

```js
// playwright.config.js
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  timeout: 30000,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  use: {
    baseURL: process.env.BASE_URL ?? "http://localhost:3000",
    headless: true,
    screenshot: "only-on-failure",
    video: "retain-on-failure",
    trace: "on-first-retry",
  },
  webServer: {
    command: "npm run start",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

### Real-World Cases

- **Local dev:** Docker Compose for full stack.
- **CI:** GitHub Actions + Playwright.
- **Cloud:** BrowserStack, Sauce Labs.
- **Cross-browser:** Playwright projects per browser.

---

## E2E Best Practices

### Definitions

**Core Definition:** E2E best practices are the guidelines for writing reliable, maintainable, and fast end-to-end tests.

**Technical Definition:** Best practices include: test critical flows only, use stable selectors (data-testid), avoid flakiness with proper waits, isolate tests, seed data, run in parallel, use Page Objects, debug with traces/videos, and integrate with CI. Follow the test pyramid: keep E2E tests few and focused.

**Beginner-Friendly Explanation:** These are the habits that make E2E tests useful instead of a source of frustration.

### Best Practices

1. **Test critical flows only.**
   - Login, signup, checkout, payment.
2. **Use stable selectors.**
   - Prefer `data-testid` over CSS classes and text.
   ```html
   <button data-testid="submit-button">Submit</button>
   ```
   ```js
   await page.click('[data-testid="submit-button"]');
   ```
3. **Avoid brittle waits.**
   - Use auto-waiting (`await expect(...).toBeVisible()`) instead of `waitForTimeout`.
4. **Isolate tests.**
   - Each test starts from a clean state.
5. **Seed data deterministically.**
   - Use factories with controlled randomness.
6. **Run tests in parallel.**
   - Playwright and Cypress support parallelism.
7. **Use Page Objects.**
   - Centralise selectors and actions.
8. **Debug with traces, videos, screenshots.**
   - Playwright's trace viewer is excellent.
9. **Keep tests fast.**
   - Mock external services where appropriate.
10. **Run in CI on every commit.**
    - Prevent regressions.
11. **Retry flaky tests.**
    - `retries: 2` in Playwright.
12. **Test across browsers.**
    - At least Chrome and one other.
13. **Test mobile viewports.**
    - Playwright device emulation.
14. **Integrate accessibility checks.**
    - `@axe-core/playwright`.
15. **Follow the test pyramid.**
    - Few E2E tests, many unit tests.

### Annotated Code Examples

#### Example 1: Using `data-testid`

```html
<form>
  <input name="email" data-testid="email-input" />
  <input name="password" data-testid="password-input" />
  <button type="submit" data-testid="login-button">Log in</button>
</form>
```

```js
await page.fill('[data-testid="email-input"]', "alice@example.com");
await page.fill('[data-testid="password-input"]', "secret");
await page.click('[data-testid="login-button"]');
```

**Why This Helps:** `data-testid` is stable across style and copy changes.

#### Example 2: Auto-Waiting Instead of Timeouts

```js
// BAD: Fixed timeout.
await page.click("#submit");
await page.waitForTimeout(2000);
expect(await page.textContent(".result")).toBe("Done");

// GOOD: Auto-waiting.
await page.click("#submit");
await expect(page.locator(".result")).toHaveText("Done");
```

#### Example 3: Isolated Tests

```js
test.beforeEach(async ({ page, request }) => {
  // Reset database via API.
  await request.post("/api/test/reset");
  await page.goto("/");
});
```

#### Example 4: Accessibility Check

```js
import AxeBuilder from "@axe-core/playwright";

test("homepage has no accessibility violations", async ({ page }) => {
  await page.goto("/");
  const results = await new AxeBuilder({ page }).analyze();
  expect(results.violations).toEqual([]);
});
```

#### Example 5: CI Configuration

```yaml
# .github/workflows/e2e.yml
name: E2E
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

### Real-World Cases

- **E-commerce:** Checkout flow tested across browsers.
- **SaaS:** Login and dashboard tested in CI.
- **Startups:** Fast feedback with Playwright + GitHub Actions.
- **Enterprises:** Selenium Grid, BrowserStack.

---

## References

- Playwright Documentation — https://playwright.dev/
- Playwright – Best Practices — https://playwright.dev/docs/best-practices
- Playwright – Page Object Models — https://playwright.dev/docs/pom
- Playwright – Test Configuration — https://playwright.dev/docs/test-configuration
- Cypress Documentation — https://docs.cypress.io/
- Cypress – Best Practices — https://docs.cypress.io/guides/references/best-practices
- Puppeteer Documentation — https://pptr.dev/
- Selenium Documentation — https://www.selenium.dev/documentation/
- Selenium – WebDriver — https://www.selenium.dev/documentation/webdriver/
- W3C WebDriver Specification — https://www.w3.org/TR/webdriver2/
- Chrome DevTools Protocol — https://chromedevtools.github.io/devtools-protocol/
- MDN Web Docs – Cross-browser testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing/Cross_browser_testing
- MDN Web Docs – Introduction to automated testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing/Automated_testing
- Testing Library Documentation — https://testing-library.com/
- MSW (Mock Service Worker) — https://mswjs.io/
- @axe-core/playwright — https://github.com/dequelabs/axe-core-npm/tree/develop/packages/playwright
- BrowserStack — https://www.browserstack.com/
- Sauce Labs — https://saucelabs.com/
- Testcontainers — https://testcontainers.com/
- Martin Fowler – PageObject — https://martinfowler.com/bliki/PageObject.html
- Martin Fowler – Test Pyramid — https://martinfowler.com/bliki/TestPyramid.html
- Kent C. Dodds – Testing Trophy — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Google Testing Blog – Test Sizes — https://testing.googleblog.com/2010/12/test-sizes.html
- GitHub Actions Documentation — https://docs.github.com/en/actions
- Docker Compose Documentation — https://docs.docker.com/compose/