# JavaScript Test Automation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Test automation is the practice of running tests automatically — without human intervention — as part of a continuous integration (CI) pipeline, with support for parallelism, reporting, coverage measurement, and advanced techniques like mutation, property-based, fuzz, and visual regression testing.

**Technical Definition:** Test automation integrates testing tools (Jest, Vitest, Playwright, Cypress) into CI/CD pipelines (GitHub Actions, GitLab CI, CircleCI, Jenkins) to run on every commit or pull request. Key infrastructure: **CI integration** (webhooks, runners, artifacts), **test pipelines** (stages: lint → unit → integration → E2E → deploy), **parallel testing** (sharding across workers), **test reporting** (JUnit XML, HTML reports, dashboards), **code coverage** (Istanbul/nyc, c8), **mutation testing** (Stryker), **property-based testing** (fast-check), **fuzz testing** (jsfuzz, jazzer.js), and **visual regression testing** (Percy, Applitools, Playwright screenshots). These techniques go beyond ordinary testing to increase confidence, find edge cases, and prevent regressions.

**Beginner-Friendly Explanation:** Test automation means your tests run by themselves — every time you push code, a CI server runs your tests and tells you if something broke. Automation goes beyond just running tests: you can run them in parallel to save time, measure how much code they cover, verify that tests actually catch bugs (mutation testing), generate random inputs to find edge cases (property-based and fuzz testing), and compare screenshots to catch visual changes (visual regression). Together, these techniques make your test suite fast, thorough, and reliable.

### Key Characteristics

- **Automated:** Runs without human intervention on every commit.
- **Layered:** Unit → integration → E2E → visual regression.
- **Parallel:** Sharded across workers for speed.
- **Measured:** Coverage, mutation score, and visual diffs.
- **Reported:** Results surface in CI dashboards and PRs.
- **Progressive:** Fast tests first, slow tests later.
- **Configurable:** Thresholds gate merges.
- **Observable:** Failures produce artifacts (logs, screenshots, videos).

### Prerequisites

- Solid understanding of unit, integration, and E2E testing.
- Familiarity with Git and pull-request workflows.
- Knowledge of CI/CD concepts.
- Experience with test runners (Jest, Vitest, Playwright).
- Basic understanding of coverage and test metrics.

### Related Programming Areas

- **CI/CD:** Pipelines, runners, artifacts.
- **Test runners:** Jest, Vitest, Playwright.
- **Coverage tools:** Istanbul, c8.
- **Mutation testing:** Stryker.
- **Property-based testing:** fast-check.
- **Fuzz testing:** jsfuzz, jazzer.js.
- **Visual regression:** Percy, Applitools.

---

## CI Integration

### Definitions

**Core Definition:** CI integration is the practice of running automated tests as part of a Continuous Integration pipeline on every commit, pull request, or merge.

**Technical Definition:** CI integration connects a version-control system (GitHub, GitLab, Bitbucket) to a CI service (GitHub Actions, GitLab CI, CircleCI, Jenkins, Travis CI) that runs scripts in response to events. Test scripts run in isolated runners, produce artifacts (reports, screenshots, videos), and report status back to the VCS. Branch protection rules can require passing tests before merging. Key concepts: workflows, jobs, steps, runners, artifacts, caches, secrets.

**Beginner-Friendly Explanation:** CI integration means your tests run automatically whenever you push code or open a pull request. If they fail, you know before merging.

### Purposes

- To catch regressions before merging.
- To enforce quality gates.
- To run tests consistently across environments.
- To provide fast feedback to developers.

### Syntax Rules and Structure

**GitHub Actions workflow example:**
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage
      - run: npm run test:e2e
```

### Annotated Code Examples

#### Example 1: GitHub Actions with Multiple Jobs

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run lint

  unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: coverage/

  e2e:
    runs-on: ubuntu-latest
    needs: [unit]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

**Expected Output:** CI runs lint, unit, and E2E jobs; failures block the PR; artifacts are uploaded.

#### Example 2: Branch Protection

```yaml
# Require "unit" and "e2e" jobs to pass before merging.
# Configure in repository settings → Branches → Branch protection rules.
```

### Real-World Cases

- **Open-source:** PRs require passing CI.
- **Enterprises:** Multiple environments (staging, prod).
- **Monorepos:** Affected-only test runs (Nx, Turborepo).

---

## Test Pipelines

### Definitions

**Core Definition:** A test pipeline is the ordered sequence of test stages that run in CI, from fast checks to slow, comprehensive tests.

**Technical Definition:** Pipelines organise tests into stages that run in order, with early stages providing fast feedback and later stages providing thorough coverage. Typical pipeline: lint → type-check → unit → integration → build → E2E → deploy → smoke tests. Failures stop the pipeline early to save resources. Pipelines can be defined in YAML (GitHub Actions, GitLab CI) or as code (Jenkinsfile, CircleCI config).

**Beginner-Friendly Explanation:** A test pipeline is the order in which your tests run — fast ones first, slow ones later — so you get feedback quickly.

### Purposes

- To fail fast on cheap checks.
- To save CI resources.
- To provide progressive confidence.
- To isolate failures to stages.

### Annotated Code Examples

#### Example 1: Multi-Stage Pipeline

```yaml
# .github/workflows/pipeline.yml
name: Pipeline

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck

  unit:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test -- --coverage
      - run: npx c8 check-coverage --lines 80

  integration:
    needs: unit
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: test }
        ports: ["5432:5432"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/test

  e2e:
    needs: integration
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
```

**Expected Output:** Pipeline runs stages in order: lint → unit → integration → e2e. Each stage gates the next.

### Real-World Cases

- **Fast feedback:** Lint and unit tests first.
- **Resource optimisation:** Slow tests only run if fast ones pass.
- **Progressive confidence:** Each stage adds assurance.

---

## Parallel Testing

### Definitions

**Core Definition:** Parallel testing runs multiple tests simultaneously across workers, shards, or machines to reduce total execution time.

**Technical Definition:** Parallelism in testing is achieved at several levels: **within a runner** (Jest/Vitest run test files in worker processes), **across shards** (split test files across CI jobs), and **across browsers/devices** (Playwright projects). Playwright supports `--shard=1/4` for sharding; Jest supports `--maxWorkers`; Vitest supports `--threads`. CI matrices can run shards in parallel jobs.

**Beginner-Friendly Explanation:** Parallel testing means running many tests at the same time instead of one after another, so your test suite finishes faster.

### Purposes

- To reduce test execution time.
- To utilise CI resources efficiently.
- To enable fast feedback on large suites.
- To scale E2E across browsers.

### Syntax Rules and Structure

**Jest:**
```bash
jest --maxWorkers=4
jest --shard=1/3
```

**Vitest:**
```bash
vitest --threads
vitest --shard=1/3
```

**Playwright:**
```bash
npx playwright test --workers=4
npx playwright test --shard=1/4
```

**GitHub Actions matrix:**
```yaml
strategy:
  matrix:
    shard: [1, 2, 3, 4]
steps:
  - run: npx playwright test --shard=${{ matrix.shard }}/4
```

### Annotated Code Examples

#### Example 1: Playwright Sharding in CI

```yaml
# .github/workflows/e2e.yml
name: E2E
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test --shard=${{ matrix.shard }}/4
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report-${{ matrix.shard }}
          path: playwright-report/
```

**Expected Output:** E2E tests are split across 4 shards, running in parallel jobs; total time reduced by ~4×.

#### Example 2: Jest Workers

```bash
# Use 4 workers.
jest --maxWorkers=4

# Use 50% of CPU cores.
jest --maxWorkers=50%
```

#### Example 3: Vitest Threads

```js
// vitest.config.js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    pool: "threads", // or "forks"
    poolOptions: {
      threads: { maxThreads: 4, minThreads: 1 },
    },
  },
});
```

### Real-World Cases

- **Large suites:** Thousands of tests run in minutes.
- **Cross-browser:** Playwright projects run in parallel.
- **Monorepos:** Affected-only testing per package.

---

## Test Reporting

### Definitions

**Core Definition:** Test reporting is the practice of producing structured output from test runs that developers and CI systems can consume.

**Technical Definition:** Test reports come in several formats: **console** (human-readable), **JUnit XML** (CI systems), **HTML** (interactive), **JSON** (machine-readable), **TAP** (Test Anything Protocol). CI systems parse JUnit XML to display test results, failures, and durations. Coverage reports use formats like LCOV, Cobertura, and HTML. Tools: Jest reporters, Vitest reporters, Playwright HTML reporter, Allure, Mochawesome.

**Beginner-Friendly Explanation:** Test reporting means presenting test results in a useful format — console output, HTML reports, or JUnit XML for CI.

### Purposes

- To communicate test results.
- To surface failures with context.
- To track test health over time.
- To integrate with CI dashboards.

### Annotated Code Examples

#### Example 1: Playwright HTML Reporter

```js
// playwright.config.js
import { defineConfig } from "@playwright/test";

export default defineConfig({
  reporter: [
    ["list"],
    ["html", { outputFolder: "playwright-report", open: "never" }],
    ["junit", { outputFile: "test-results/junit.xml" }],
  ],
});
```

#### Example 2: Vitest Reporters

```js
// vitest.config.js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    reporters: ["default", "junit", "html"],
    outputFile: {
      junit: "./test-results/junit.xml",
      html: "./test-results/index.html",
    },
  },
});
```

#### Example 3: JUnit XML Upload to CI

```yaml
- run: npm test
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: test-results
    path: test-results/
- uses: dorny/test-reporter@v1
  if: always()
  with:
    name: Test Results
    path: test-results/junit.xml
    reporter: jest-junit
```

### Real-World Cases

- **PR checks:** JUnit XML surfaces failures.
- **Dashboards:** Allure, ReportPortal.
- **Historical tracking:** Store reports per commit.

---

## Code Coverage

### Definitions

**Core Definition:** Code coverage measures the percentage of source code executed by tests.

**Technical Definition:** Coverage is measured by instrumenting source code (Istanbul) or using V8's built-in coverage (c8). Metrics: **statement**, **branch**, **function**, and **line** coverage. Reports include HTML (interactive), LCOV (CI tools), Cobertura (Jenkins), and text (console). Tools: Istanbul (nyc), c8, Jest's built-in coverage. Coverage thresholds can gate merges.

**Beginner-Friendly Explanation:** Coverage tells you how much of your code is exercised by tests. It's a useful signal but not a guarantee of correctness.

### Purposes

- To identify untested code.
- To track coverage trends.
- To enforce minimum thresholds.
- To guide testing effort.

### Annotated Code Examples

#### Example 1: Vitest Coverage with c8

```js
// vitest.config.js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    coverage: {
      provider: "v8",
      reporter: ["text", "html", "lcov"],
      include: ["src/**/*.js"],
      exclude: ["src/**/*.test.js"],
      thresholds: {
        lines: 80,
        branches: 75,
        functions: 80,
        statements: 80,
      },
    },
  },
});
```

**Run with:**
```bash
vitest run --coverage
```

**Expected Output (text):**
```
% Coverage report from v8
------------------|---------|----------|---------|---------|
File              | % Stmts | % Branch | % Funcs | % Lines |
------------------|---------|----------|---------|---------|
All files         |   85.2  |   78.4   |   90.0  |   85.2  |
 src/             |   85.2  |   78.4   |   90.0  |   85.2  |
  calculator.js   |   100   |   80     |   100   |   100   |
  userService.js  |   72.5  |   65.0   |   80.0  |   72.5  |
------------------|---------|----------|---------|---------|
```

#### Example 2: Jest Coverage

```bash
jest --coverage --coverageThreshold='{"global":{"lines":80,"branches":75}}'
```

#### Example 3: nyc with Mocha

```json
// package.json
{
  "scripts": {
    "test": "mocha",
    "coverage": "nyc --reporter=html --reporter=text mocha"
  },
  "nyc": {
    "check-coverage": true,
    "lines": 80,
    "branches": 75,
    "functions": 80,
    "statements": 80
  }
}
```

### Real-World Cases

- **CI gates:** Fail if coverage drops.
- **Coverage badges:** README status.
- **Trend tracking:** Codecov, Coveralls.

---

## Mutation Testing

### Definitions

**Core Definition:** Mutation testing evaluates test quality by introducing small changes (mutations) to source code and checking whether tests fail.

**Technical Definition:** Mutation testing tools (Stryker for JavaScript) systematically mutate source code — replacing `+` with `-`, `<` with `>`, removing statements, changing return values. For each mutant, the test suite is run. If tests fail, the mutant is "killed" (good). If tests pass, the mutant "survives" (bad — tests didn't catch the change). The mutation score is the percentage of killed mutants. Mutation testing is slow but reveals weak tests that high coverage misses.

**Beginner-Friendly Explanation:** Mutation testing checks whether your tests actually catch bugs. The tool deliberately breaks your code in small ways and checks if your tests notice. If they don't, your tests are weak.

### Purposes

- To measure test effectiveness.
- To find gaps in assertions.
- To improve test quality beyond coverage.
- To catch false confidence from high coverage.

### Syntax Rules and Structure

**Stryker configuration:**
```json
{
  "mutate": ["src/**/*.js", "!src/**/*.test.js"],
  "testRunner": "vitest",
  "reporters": ["html", "clear-text", "progress"],
  "thresholds": { "high": 80, "low": 60, "break": 50 }
}
```

### Annotated Code Examples

#### Example 1: Running Stryker

```bash
npx stryker run
```

**Expected Output:**
```
Mutation testing report:
Files: 5
Mutants: 120
Killed: 100 (83.3%)
Survived: 15 (12.5%)
No coverage: 5 (4.2%)
Mutation score: 83.3%
```

**Why This Helps:** A mutation score of 83.3% means 83.3% of introduced bugs were caught. Surviving mutants indicate weak tests.

#### Example 2: Interpreting Surviving Mutants

```js
// Original code
function isAdult(age) {
  return age >= 18;
}

// Mutant: >= replaced with >
function isAdult(age) {
  return age > 18;
}

// If tests only check isAdult(20) === true, both pass.
// Add isAdult(18) === true to kill the mutant.
```

### Real-World Cases

- **Critical code:** Payments, auth, medical.
- **Test quality:** Find weak assertions.
- **CI (optional):** Run nightly.

---

## Property-Based Testing

### Definitions

**Core Definition:** Property-based testing verifies that properties (invariants) hold for randomly generated inputs, rather than checking specific examples.

**Technical Definition:** Property-based testing tools (fast-check for JavaScript) generate random inputs and check that a property holds. When a property fails, the tool shrinks the input to find a minimal failing case. Properties are statements like "reversing a list twice returns the original" or "sorting is idempotent."

**Beginner-Friendly Explanation:** Instead of testing specific examples, property-based testing checks general rules with random inputs — and finds edge cases you wouldn't think of.

### Purposes

- To find edge cases.
- To reduce example-based test maintenance.
- To verify invariants.
- To complement example-based tests.

### Syntax Rules and Structure

```js
import { test, expect } from "vitest";
import fc from "fast-check";

test("reverse twice returns original", () => {
  fc.assert(
    fc.property(fc.array(fc.integer()), (arr) => {
      const reversed = [...arr].reverse().reverse();
      expect(reversed).toEqual(arr);
    })
  );
});
```

### Annotated Code Examples

#### Example 1: Property of `sort`

```js
import { describe, it, expect } from "vitest";
import fc from "fast-check";

describe("sort", () => {
  it("is idempotent", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = [...arr].sort((a, b) => a - b);
        const sortedAgain = [...sorted].sort((a, b) => a - b);
        expect(sortedAgain).toEqual(sorted);
      })
    );
  });

  it("preserves length", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = [...arr].sort((a, b) => a - b);
        expect(sorted.length).toBe(arr.length);
      })
    );
  });
});
```

**Expected Output:**
```
✓ sort > is idempotent
✓ sort > preserves length
```

#### Example 2: Property of `encode`/`decode`

```js
fc.assert(
  fc.property(fc.string(), (s) => {
    expect(decodeURIComponent(encodeURIComponent(s))).toBe(s);
  })
);
```

**Note:** This property fails for strings with lone surrogates; fast-check shrinks to a minimal failing example.

### Real-World Cases

- **Parsers:** Encode/decode round-trips.
- **Data structures:** Invariants (e.g., heap order).
- **Algorithms:** Correctness properties.
- **Math libraries:** Algebraic laws.

---

## Fuzz Testing

### Definitions

**Core Definition:** Fuzz testing feeds random or malformed inputs to a program to find crashes, hangs, and unexpected behaviour.

**Technical Definition:** Fuzz testing generates many random inputs (often via mutation of seed corpora) and monitors the program for crashes, exceptions, or hangs. Coverage-guided fuzzing (libFuzzer, jazzer.js) uses code coverage to guide input generation. Fuzzing is common for parsers, decoders, and security-sensitive code. JavaScript fuzzers: jsfuzz, jazzer.js.

**Beginner-Friendly Explanation:** Fuzz testing throws random or malformed data at your code to see if it crashes.

### Purposes

- To find crashes and unhandled exceptions.
- To test parsers and decoders.
- To find security vulnerabilities.
- To complement property-based testing.

### Annotated Code Examples

#### Example 1: Simple Fuzz Test

```js
import { fuzz } from "@jazzer.js/core";
import { parseJSON } from "./parser.js";

fuzz({
  target: (data) => {
    try {
      parseJSON(data.toString("utf-8"));
    } catch (err) {
      // Expected for invalid input
      if (!(err instanceof SyntaxError)) throw err;
    }
  },
});
```

#### Example 2: Coverage-Guided Fuzzing with jazzer.js

```js
// fuzz/parser.fuzz.js
import { fuzz } from "@jazzer.js/core";
import { parse } from "../src/parser.js";

fuzz({
  target: (data) => {
    const input = data.toString("utf-8");
    parse(input); // Should not crash (may throw SyntaxError)
  },
  maxLength: 1024,
});
```

**Run with:**
```bash
npx jazzer fuzz/parser.fuzz.js
```

### Real-World Cases

- **JSON/XML parsers:** Malformed inputs.
- **Image decoders:** Malformed image files.
- **Compression libraries:** Corrupt archives.
- **Security-critical code:** Input validation.

---

## Visual Regression Testing

### Definitions

**Core Definition:** Visual regression testing compares screenshots of UI before and after changes to detect unintended visual differences.

**Technical Definition:** Visual regression tools capture baseline screenshots, then compare new screenshots pixel-by-pixel after changes. Differences are flagged for review. Tools: Percy, Applitools, Chromatic, Playwright's `toHaveScreenshot`, BackstopJS. Challenges: false positives from font rendering, animations, and dynamic content. Best practices: disable animations, mask dynamic regions, use consistent viewports.

**Beginner-Friendly Explanation:** Visual regression testing takes screenshots of your UI and compares them to catch unintended visual changes.

### Purposes

- To catch unintended visual changes.
- To verify responsive layouts.
- To test across browsers.
- To prevent style regressions.

### Syntax Rules and Structure

**Playwright:**
```js
await expect(page).toHaveScreenshot("homepage.png");
```

**Percy:**
```js
import percySnapshot from "@percy/playwright";
await percySnapshot(page, "Homepage");
```

### Annotated Code Examples

#### Example 1: Playwright Visual Regression

```js
// tests/visual.spec.js
import { test, expect } from "@playwright/test";

test("homepage looks correct", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveScreenshot("homepage.png", {
    fullPage: true,
    animations: "disabled",
    mask: [page.locator(".timestamp")],
  });
});
```

**First run:** Creates baseline screenshot.
**Subsequent runs:** Compares against baseline; fails if different.

**Run with:**
```bash
npx playwright test tests/visual.spec.js
npx playwright test --update-snapshots  # Update baselines
```

#### Example 2: Percy Integration

```js
// tests/percy.spec.js
import { test } from "@playwright/test";
import percySnapshot from "@percy/playwright";

test("homepage", async ({ page }) => {
  await page.goto("/");
  await percySnapshot(page, "Homepage");
});

test("product page", async ({ page }) => {
  await page.goto("/products/1");
  await percySnapshot(page, "Product Page");
});
```

**Run with:**
```bash
npx percy exec -- playwright test
```

### Real-World Cases

- **Design systems:** Component visual consistency.
- **E-commerce:** Product pages, checkout.
- **Marketing sites:** Landing pages.
- **Responsive design:** Multiple viewports.

---

## References

- Jest Documentation — https://jestjs.io/docs/getting-started
- Vitest Documentation — https://vitest.dev/
- Playwright Documentation — https://playwright.dev/
- Playwright – Sharding — https://playwright.dev/docs/test-sharding
- Playwright – Visual comparisons — https://playwright.dev/docs/test-snapshots
- Playwright – Reporters — https://playwright.dev/docs/test-reporters
- Cypress Documentation — https://docs.cypress.io/
- Mocha Documentation — https://mochajs.org/
- Node.js Documentation – Test runner — https://nodejs.org/api/test.html
- Istanbul Documentation — https://istanbul.js.org/
- c8 Documentation — https://github.com/bcoe/c8
- nyc Documentation — https://github.com/istanbuljs/nyc
- Stryker Mutator — https://stryker-mutator.io/
- fast-check — https://fast-check.dev/
- jazzer.js — https://github.com/CodeIntelligenceTesting/jazzer.js
- jsfuzz — https://github.com/fuzitorg/JsFuzz
- Percy — https://percy.io/
- Applitools — https://applitools.com/
- Chromatic — https://www.chromatic.com/
- BackstopJS — https://github.com/garris/BackstopJS
- GitHub Actions Documentation — https://docs.github.com/en/actions
- GitLab CI Documentation — https://docs.gitlab.com/ee/ci/
- CircleCI Documentation — https://circleci.com/docs/
- Jenkins Documentation — https://www.jenkins.io/doc/
- Codecov — https://about.codecov.io/
- Coveralls — https://coveralls.io/
- Allure Report — https://allurereport.org/
- Martin Fowler – Continuous Integration — https://martinfowler.com/articles/continuousIntegration.html
- Martin Fowler – Test Pyramid — https://martinfowler.com/bliki/TestPyramid.html
- Google Testing Blog – Test Sizes — https://testing.googleblog.com/2010/12/test-sizes.html
- Kent C. Dodds – Testing Trophy — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications