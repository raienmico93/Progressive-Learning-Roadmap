# Frontend Testing Strategy & Philosophy: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Frontend Testing Strategy & Philosophy is the set of principles, models, and tooling decisions that determine how a React application's correctness is verified—emphasising integration over isolation, user behaviour over implementation details, and realistic environments over mocked approximations.

**Technical Definition:** Frontend testing strategy encompasses the selection and distribution of test types (static analysis, unit, integration, end-to-end) based on their cost-to-confidence ratio, the philosophical stance that tests should assert on observable user-facing behaviour rather than internal component state, the configuration of execution environments (Node.js with DOM emulation via jsdom or happy-dom) and test runners (Vitest, Jest), and the architectural decision of how much test isolation is appropriate versus wrapping components in realistic application contexts (providers, routers, state stores). The Testing Trophy model formalises this distribution: integration tests form the largest portion (~50%) because they offer the best confidence-per-dollar, followed by E2E tests for critical user journeys (~20%), unit tests for pure logic (~20%), and static analysis as a zero-cost foundation. 

**Beginner-Friendly Explanation:** Testing your app is like checking a bridge before you let cars drive on it. You could test every single bolt in isolation (unit tests), but that doesn't tell you if the bridge actually holds together. Instead, you want to test how the bolts work together (integration tests) and, occasionally, drive a car across the whole bridge (end-to-end tests). The Testing Trophy says: focus most of your effort on testing how things work together, not on testing tiny pieces in isolation. And when you do test, test what the user sees and does—not the secret internal wiring of your components.

### Key Characteristics

- **Confidence-Per-Dollar Focus:** Integration tests provide the highest confidence for the least effort because they exercise real behaviour across real modules, unlike unit tests that often just test mocks. 
- **User-Centric Assertions:** Tests are written from the user's perspective—finding elements by their accessible role or label, interacting with them as a user would, and asserting on visible outcomes. 
- **Refactoring Resilience:** Behaviour-driven tests survive implementation changes (renaming state variables, swapping CSS libraries, restructuring components) and only fail when actual user-facing behaviour regresses. 
- **Realistic Contexts Over Pure Isolation:** Components that depend on routing, theming, or state management are wrapped in their real providers (using a custom `render` utility) rather than mocked, ensuring tests reflect how the component behaves in the actual application. 
- **Modern Tooling Alignment:** Vitest with happy-dom offers 2–4x faster execution than Jest with jsdom for most React component tests, while jsdom remains the standard for Jest and for tests requiring maximum DOM API accuracy. 
- **Accessibility as a Testing Signal:** Querying by role (`getByRole`) is the primary query method because it exposes the element exactly as the accessibility tree does—if you can't query by role, your UI likely lacks accessibility. 

### Prerequisites

- Solid understanding of React components, props, state, and Hooks.
- Familiarity with JavaScript Promises and `async`/`await` for async testing utilities.
- Basic understanding of DOM structure and accessibility semantics (roles, labels).
- Awareness of module mocking and dependency injection concepts.
- Experience with a test runner (Vitest or Jest) and an assertion library.

### Related Programming Areas

- **Test-Driven Development (TDD):** Writing tests before implementation to drive design.
- **Behaviour-Driven Development (BDD):** Describing behaviour in user-facing terms.
- **Accessibility (a11y):** Testing that the UI is usable by assistive technologies.
- **CI/CD Pipelines:** Integrating tests into continuous integration and deployment.
- **Code Coverage:** Measuring which code paths are exercised by tests.
- **Mutation Testing:** Assessing test suite quality by introducing deliberate bugs.

### Core Concepts / Features

1. The Testing Trophy Model
2. The User-Centric Philosophy
3. Test Environments & Runners
4. Test Isolation vs. Setup

---

## Core Concept 1: The Testing Trophy Model

### Definitions

**Core Definition:** The Testing Trophy is a testing distribution model, popularised by Kent C. Dodds, that prioritises integration tests over unit tests, reflecting the modern cost-to-confidence tradeoffs of frontend testing tooling.

**Technical Definition:** The Testing Trophy replaces the traditional Testing Pyramid's emphasis on unit tests with a four-layer model: **Static Analysis** (TypeScript, ESLint) at the base, **Unit Tests** (~20%) for pure logic with no dependencies, **Integration Tests** (~50%) for multiple units working together, and **End-to-End Tests** (~20%) for critical user journeys through the full system.  The rationale is that integration tests provide the best confidence-per-dollar because they test real behaviour across real modules, whereas unit tests in isolation often merely test mocks rather than the actual system.  The model also acknowledges that static analysis catches a large class of bugs at zero runtime cost, and that E2E tests, while highest in confidence, are slow and expensive. 

**Beginner-Friendly Explanation:** The Testing Trophy is a way of deciding how much of each type of test to write. Imagine you're building a car. You could test every screw individually (unit tests), but that's slow and doesn't prove the car drives. Instead, you test how the engine and transmission work together (integration tests), and occasionally you take the whole car for a drive (E2E tests). The Trophy says: spend most of your testing time on integration tests because they give you the most confidence for the least effort.

### Purposes

- To allocate testing effort where it provides the highest confidence per unit of cost.
- To shift focus from isolated unit tests (which often test mocks) toward integration tests that exercise real behaviour.
- To provide a pragmatic framework for right-sizing test coverage across different project types.
- To acknowledge the value of static analysis as a zero-cost foundation.
- To guide teams in deciding when to write unit tests (pure logic, algorithms) versus integration tests (component interactions).
- To reduce the maintenance burden of brittle unit tests that break on refactors without catching real bugs.

### Syntax Rules and Structure

**The Trophy Distribution:**

```text
┌───────────────────────┐
│         E2E           │  ~20% — Critical user journeys only
├───────────────────────┤
│     Integration       │  ~50% — Best confidence-per-dollar
├───────────────────────┤
│        Unit           │  ~20% — Pure logic, algorithms, edge cases
├───────────────────────┤
│   Static Analysis     │  Always on — Zero-cost, catches trivial bugs
└───────────────────────┘
```

**Component Breakdown:**
- **Static Analysis:** TypeScript, ESLint, and security scanners. Runs at write time, catches syntax errors, type mismatches, and common bugs before any code executes. 
- **Unit Tests:** Test individual functions or classes in isolation, with dependencies mocked. Fast and cheap, but only useful for pure logic without collaborators. 
- **Integration Tests:** Test multiple units working together (e.g., a form component with its validation logic and state management). Medium speed, high confidence. 
- **E2E Tests:** Test full user flows through the real system (browser, server, database). Slow and expensive, but provide the highest confidence. 

**General Syntax for an Integration Test (Testing Library):**

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { SearchForm } from './SearchForm';

test('should filter results when the user types a query', async () => {
  const user = userEvent.setup();
  render(<SearchForm />);

  // Arrange: find the search input by its accessible role
  const input = screen.getByRole('textbox', { name: /search/i });

  // Act: type a query as a user would
  await user.type(input, 'react');

  // Assert: verify that the filtered results are visible
  const results = await screen.findAllByText(/react/i);
  expect(results.length).toBeGreaterThan(0);
});
```

**Component Breakdown:**
- `render(<SearchForm />)`: Renders the component in a simulated DOM.
- `screen.getByRole('textbox', { name: /search/i })`: Finds the input by its accessible role and name (not by CSS class or test ID).
- `userEvent.setup()`: Creates a user event instance that simulates real browser interactions.
- `user.type(input, 'react')`: Types into the input as a user would.
- `screen.findAllByText(/react/i)`: Asynchronously waits for and finds elements containing "react".

**Syntax Rules:**
- Write integration tests for any component that combines multiple concerns (state, rendering, user interaction).
- Reserve unit tests for pure functions (utility helpers, formatters, validators) with no dependencies.
- Write E2E tests only for critical user journeys (login, checkout, sign-up).
- Always include static analysis in CI; it catches bugs that tests might miss.
- Do not write unit tests just to hit coverage numbers; they provide false confidence if they only test mocks. 

**Constraints and Limitations:**
- Integration tests are slower than unit tests; a large suite can take minutes to run.
- E2E tests are flaky and expensive to maintain; use sparingly.
- The 20/50/20 distribution is a guideline, not a strict rule; project type influences the balance. 
- Some logic (complex algorithms, state machines) genuinely benefits from focused unit tests.
- Static analysis cannot catch runtime logic errors; integration tests are still required.

### Annotated Code Examples

**Example 1: Unit Test vs. Integration Test for a Shopping Cart**

**cart-utils.js**
Contains the pure mathematical logic for calculating totals.
```jsx
// Pure logic (unit-testable)
export function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
```

**cart-utils.test.js**
The isolated unit test for verifying the total calculation.
```jsx
// Unit test (fast, isolated)
import { calculateTotal } from './cart-utils';

test('calculateTotal sums item prices with quantities', () => {
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 3 },
  ];
  expect(calculateTotal(items)).toBe(35);
});
```

**Cart.jsx**
The UI component that manages the shopping cart state.
```jsx
// Component with state and rendering (integration-testable)
import React, { useState } from 'react';
import { calculateTotal } from './cart-utils';

export function Cart() {
  const [items, setItems] = useState([]);

  function addItem() {
    setItems((prev) => [...prev, { price: 10, quantity: 1 }]);
  }

  return (
    <div>
      <button onClick={addItem}>Add Item</button>
      <p>Total: ${calculateTotal(items)}</p>
    </div>
  );
}
```

**Cart.test.jsx**
The integration test to ensure the UI component correctly responds to user interactions and updates the state.
```jsx
// Integration test (real component, real state)
import React from 'react';

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Cart } from './Cart';

test('should update total when an item is added', async () => {
  const user = userEvent.setup();
  render(<Cart />);

  // Initially zero
  expect(screen.getByText(/total: \$0/i)).toBeInTheDocument();

  // Act: click the add button
  await user.click(screen.getByRole('button', { name: /add item/i }));

  // Assert: total updates
  expect(screen.getByText(/total: \$10/i)).toBeInTheDocument();
});
```

**Expected Output:** The unit test passes instantly (calculates 35). The integration test passes after simulating a user click, verifying that the button, state, and rendering all work together.

**Why This Output Occurs:** The unit test verifies the pure `calculateTotal` function in isolation. The integration test verifies the `Cart` component's behaviour: clicking the button updates state, which recalculates the total and re-renders the output. The integration test provides higher confidence because it exercises the actual component logic and rendering, while the unit test provides fast feedback on the math. 

**Example 2: E2E Test for a Login Flow (Playwright)**

```javascript
// e2e/login.spec.js — E2E test (full system)
import { test, expect } from '@playwright/test';

test('user can log in with valid credentials', async ({ page }) => {
  // Navigate to the login page
  await page.goto('/login');

  // Fill in credentials
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');

  // Submit
  await page.getByRole('button', { name: /log in/i }).click();

  // Assert: redirected to dashboard and user name is visible
  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByText('Welcome, Alice')).toBeVisible();
});
```

**Expected Output:** The E2E test launches a real browser, navigates to the login page, fills in credentials, submits the form, and verifies that the dashboard loads with the user's name.

**Why This Output Occurs:** The E2E test exercises the full stack: browser, React app, API, and database. It provides the highest confidence that the login flow works end-to-end, but it is slower and more expensive to run than integration tests. This is why E2E tests are reserved for critical journeys only. 

### Real-World Cases

- **Design systems:** Using integration tests to verify that components render correctly with real props and interactions, rather than unit-testing every prop permutation.
- **E-commerce checkout:** Writing E2E tests for the checkout journey (cart → address → payment → confirmation) while writing integration tests for individual form steps.
- **SaaS dashboards:** Writing integration tests for each widget (loading, data display, interaction) and unit tests for data transformation utilities.
- **API clients:** Writing unit tests for request/response serialisation and integration tests for retry logic and error handling.
- **Form-heavy applications:** Writing integration tests for form submission, validation, and error display, with unit tests for validation schemas.

---

## Core Concept 2: The User-Centric Philosophy

### Definitions

**Core Definition:** The User-Centric Philosophy is the testing principle that tests should assert on observable user-facing behaviour rather than internal implementation details such as component state, CSS class names, or DOM structure.

**Technical Definition:** The User-Centric Philosophy, embodied by Testing Library, states that "the more your tests resemble the way your software is used, the more confidence they can give you."  Tests should query elements by their accessible role (`getByRole`), label (`getByLabelText`), or visible text—the same signals a real user (including one using assistive technology) would use.  Tests should avoid asserting on internal state values, hook return values, CSS class names, `data-testid` attributes (except as a last resort), or DOM structure, because these are implementation details that change during refactoring without affecting user-visible behaviour.  A test coupled to implementation is a tax on every refactor and provides false confidence when behaviour actually breaks. 

**Beginner-Friendly Explanation:** When you test a login form, you don't care whether the component uses `useState` or `useReducer` internally. You care that when a user types their email and password and clicks "Log In," they either see the dashboard or an error message. User-centric testing means writing tests that mimic what a real person does—find the email field by its label, type into it, click the button, and check what appears on screen. This way, if you refactor the component's internals (rename a variable, swap a `<div>` for a `<section>`), your tests still pass because the behaviour didn't change.

### Purposes

- To write tests that survive refactoring without breaking on non-behavioural changes.
- To ensure tests provide genuine confidence that the UI works for real users.
- To catch accessibility regressions by requiring elements to be queryable by role or label.
- To reduce test maintenance burden by coupling tests to behaviour, not implementation.
- To align testing with the actual user experience, including keyboard navigation and screen reader compatibility.
- To make tests readable and understandable by describing user actions and outcomes.

### Syntax Rules and Structure

**Query Priority (Best to Worst):** 

1. **Queries accessible to everyone (preferred):** `getByRole` (primary choice), `getByLabelText` (form fields), `getByPlaceholderText`, `getByText`, `getByDisplayValue`.
2. **Semantic queries:** `getByAltText`, `getByTitle`.
3. **Test IDs (last resort):** `getByTestId`—use only when no accessible query is possible.

**General Syntax for Behaviour-Driven Testing:**

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

test('should show an error when submitting an invalid email', async () => {
  const user = userEvent.setup();
  render(<LoginForm />);

  // Arrange: find the email input by its label
  const emailInput = screen.getByLabelText(/email/i);
  const submitButton = screen.getByRole('button', { name: /log in/i });

  // Act: type an invalid email and submit
  await user.type(emailInput, 'not-an-email');
  await user.click(submitButton);

  // Assert: an error message is visible
  const errorMessage = screen.getByText(/please enter a valid email/i);
  expect(errorMessage).toBeInTheDocument();
});
```

**Component Breakdown:**
- `screen.getByLabelText(/email/i)`: Finds the email input by its associated label—the same way a screen reader would.
- `screen.getByRole('button', { name: /log in/i })`: Finds the submit button by its accessible role and name.
- `user.type(emailInput, 'not-an-email')`: Types into the input as a user would.
- `screen.getByText(/please enter a valid email/i)`: Finds the error message by its visible text.

**Bad Example (Brittle, Implementation-Coupled):**

```jsx
// ❌ BAD: Coupled to implementation details
const { container } = render(<LoginForm />);
const input = container.querySelector('.email-input');
fireEvent.change(input, { target: { value: 'not-an-email' } });
// This test breaks if you rename the CSS class or change the DOM structure
```

**Syntax Rules:**
- Always prefer `getByRole` as the primary query method. 
- Use `getByLabelText` for form fields; every input should have an associated label.
- Use `getByText` for non-interactive elements (headings, paragraphs).
- Use `getByTestId` only when no accessible query is feasible, and document why.
- Never assert on component state directly (e.g., `component.state.isValid`).
- Never assert on CSS class names or DOM structure unless the test is specifically about styling.
- Use `userEvent` over `fireEvent` for simulating real user interactions. 

**Constraints and Limitations:**
- Some elements may not have implicit ARIA roles; explicit `role` attributes may be required.
- Testing complex animations or transitions may require waiting utilities (`waitFor`, `findBy*`).
- Testing third-party components that don't expose accessible roles may require test IDs as a fallback.
- User-centric tests are slower than implementation-coupled tests because they simulate real interactions.
- Full user-centric testing requires a genuine understanding of accessibility semantics.

### Annotated Code Examples

**Example 1: User-Centric Login Form Test**

```jsx
import React, { useState } from 'react';

export function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  async function handleSubmit(e) {
    e.preventDefault();
    setError('');

    if (!email.includes('@')) {
      setError('Please enter a valid email address');
      return;
    }

    try {
      await onSubmit({ email, password });
    } catch {
      setError('Invalid email or password');
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />

      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />

      {error && <p role="alert">{error}</p>}

      <button type="submit">Log In</button>
    </form>
  );
}
```

```jsx
// LoginForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

test('should show a validation error for an invalid email', async () => {
  const user = userEvent.setup();
  render(<LoginForm onSubmit={jest.fn()} />);

  // Arrange: find fields by their labels
  const emailInput = screen.getByLabelText(/email/i);
  const submitButton = screen.getByRole('button', { name: /log in/i });

  // Act: type an invalid email and submit
  await user.type(emailInput, 'invalid');
  await user.click(submitButton);

  // Assert: the error message is visible
  const errorMessage = screen.getByRole('alert');
  expect(errorMessage).toHaveTextContent(/please enter a valid email/i);
});
```

**Expected Output:** The test passes because the component renders an error message with `role="alert"` when the email is invalid. The test finds the email input by its label (`/email/i`), the button by its role (`button`, name `/log in/i`), and the error by its role (`alert`).

**Why This Output Occurs:** The test queries elements the way a user (and assistive technology) would: the email field has an associated `<label>`, the button has the text "Log In", and the error message uses `role="alert"` to announce itself to screen readers. If the component's internals change (e.g., `useState` replaced with `useReducer`), the test still passes because the user-visible behaviour is identical. 

**Example 2: Bad vs. Good Test Comparison**

```jsx
// ❌ BAD: Implementation-coupled test
test('should set isValid to false for invalid email', () => {
  const wrapper = shallow(<LoginForm />);
  wrapper.find('input').simulate('change', { target: { value: 'invalid' } });
  expect(wrapper.state('isValid')).toBe(false); // Tests internal state!
});

// ✅ GOOD: User-centric test
test('should show an error message for an invalid email', async () => {
  const user = userEvent.setup();
  render(<LoginForm />);

  await user.type(screen.getByLabelText(/email/i), 'invalid');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(screen.getByRole('alert')).toHaveTextContent(/valid email/i);
});
```

**Expected Output:** The bad test breaks if the component switches from `useState` to `useReducer` or if the state variable is renamed. The good test continues to pass because it asserts on the visible error message, which is the actual user-facing behaviour.

**Why This Output Occurs:** The bad test couples itself to the component's internal state shape (`isValid`), which is an implementation detail. The good test couples itself to the user-visible outcome (an error message appears), which is stable across refactors. This is the core principle of user-centric testing. 

### Real-World Cases

- **Form validation:** Testing that invalid inputs produce visible error messages, not that the internal validation function returns `false`.
- **Navigation:** Testing that clicking a link navigates to the correct page, not that the router's internal state updated.
- **Data loading:** Testing that a loading spinner disappears and data appears, not that the `isLoading` state variable changed.
- **Modal interactions:** Testing that pressing Escape closes the modal and focus returns to the trigger element.
- **Accessibility:** Testing that every interactive element is reachable by keyboard and has an accessible name.

---

## Core Concept 3: Test Environments & Runners

### Definitions

**Core Definition:** Test Environments & Runners are the execution infrastructure—the test runner (Vitest or Jest), the DOM emulation environment (jsdom, happy-dom, or Node.js), and the configuration that determines how tests execute and what APIs are available.

**Technical Definition:** A test runner is a program that discovers test files, executes test cases, and reports results. Vitest is a Vite-native test runner with first-class TypeScript and ESM support, optional browser mode via Playwright, and in-source testing.  Jest is a mature, widely adopted test runner with a large ecosystem and `ts-jest` for TypeScript. The DOM environment determines which browser APIs are available: `node` (default, no DOM), `jsdom` (emulates the browser by providing Browser API, uses the `jsdom` package), `happy-dom` (emulates the browser API, considered faster than jsdom but lacks some API), and `edge-runtime` (emulates Vercel's edge runtime).  Vitest's default DOM environment is `happy-dom` in recent versions, while Jest uses `jsdom`. 

**Beginner-Friendly Explanation:** Your tests need a place to run. Vitest and Jest are the two main test runners—they find your tests, run them, and tell you which ones passed or failed. But your React components expect a browser environment (with `document`, `window`, etc.), so you also need a DOM emulator. jsdom is the most accurate but slower; happy-dom is faster but less complete. Vitest is newer, faster, and works seamlessly with Vite; Jest is older, more established, and has a larger ecosystem.

### Purposes

- To execute tests in an environment that provides the APIs your React components need.
- To choose the right balance of DOM API accuracy versus execution speed.
- To configure TypeScript, JSX, and module resolution for the test environment.
- To integrate with CI/CD pipelines for automated testing.
- To enable code coverage reporting and threshold enforcement.
- To support different environments for different test files (e.g., Node for utility tests, jsdom for component tests).

### Syntax Rules and Structure

**Vitest Configuration (Recommended for New Projects):** 

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    environment: 'jsdom', // or 'happy-dom' for speed
    globals: true,         // use describe/it/expect without imports
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
});
```

**Component Breakdown:**
- `plugins: [react()]`: Enables JSX transformation via `@vitejs/plugin-react`.
- `environment: 'jsdom'`: Sets the DOM emulation environment for all test files.
- `globals: true`: Makes `describe`, `it`, and `expect` available globally.
- `setupFiles`: Runs setup code (e.g., importing `@testing-library/jest-dom`) before tests.
- `coverage.thresholds`: Enforces minimum coverage percentages.

**Jest Configuration (Standard for Established Projects):** 

```typescript
// jest.config.ts
import type { Config } from '@jest/types';

const config: Config.InitialOptions = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEach: ['<rootDir>/src/test/setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  transform: {
    '^.+\\.tsx?$': ['ts-jest', { tsconfig: { jsx: 'react-jsx' } }],
  },
  collectCoverageFrom: ['src/**/*.{ts,tsx}', '!src/**/*.d.ts'],
};

export default config;
```

**Component Breakdown:**
- `preset: 'ts-jest'`: Uses `ts-jest` to transform TypeScript files.
- `testEnvironment: 'jsdom'`: Uses jsdom for DOM emulation.
- `moduleNameMapper`: Maps path aliases and mocks CSS imports.
- `transform`: Configures `ts-jest` with the React JSX transform.

**DOM Environment Comparison:**

| Feature | jsdom | happy-dom |
|---------|-------|-----------|
| **DOM API Accuracy** | High (more complete) | Lower (lacks some APIs) |
| **Speed** | Slower | 2–4x faster |
| **Jest Support** | Standard | Not standard |
| **Vitest Support** | Yes | Yes (default in newer versions) |
| **Best For** | Tests requiring accurate DOM behaviour | Most React component tests |

**Syntax Rules:**
- Use Vitest for new projects; it has native ESM support, faster execution, and optional browser mode. 
- Use Jest for existing projects with large test suites or heavy ecosystem dependencies.
- Use `jsdom` when your tests depend on accurate DOM behaviour (e.g., focus management, selection ranges). 
- Use `happy-dom` when speed is the priority and your tests don't require obscure DOM APIs. 
- Set `environment: 'jsdom'` or `'happy-dom'` in the config for React component tests.
- Use `environment: 'node'` (default) for utility tests that don't need DOM APIs.
- Use per-file environment control comments (`// @vitest-environment jsdom`) for mixed test suites. 

**Constraints and Limitations:**
- Neither jsdom nor happy-dom is a full browser; some APIs may be missing or behave differently. 
- happy-dom is faster but has known compatibility gaps (e.g., focus management, selection ranges). 
- Vitest does not currently support `async` Server Components; use E2E tests for those. 
- Jest's ESM support requires additional configuration and is less seamless than Vitest's.
- Browser mode (real browser testing) is available in Vitest but requires Playwright.

### Annotated Code Examples

**Example 1: Vitest Configuration with happy-dom and Setup File**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'happy-dom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
    css: true,
  },
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
  cleanup();
});
```

```jsx
// Button.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

test('should call onClick when clicked', async () => {
  const user = userEvent.setup();
  const handleClick = vi.fn(); // Vitest's mock function

  render(<Button onClick={handleClick}>Click me</Button>);

  await user.click(screen.getByRole('button', { name: /click me/i }));

  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

**Expected Output:** The test passes. `happy-dom` provides a fast DOM environment, `@testing-library/jest-dom/vitest` extends `expect` with DOM matchers, and the `cleanup` function ensures the DOM is reset between tests.

**Why This Output Occurs:** `happy-dom` is Vitest's default in recent versions and provides a fast, lightweight DOM. The setup file imports the Vitest-compatible version of `jest-dom` matchers and ensures cleanup after each test. `vi.fn()` is Vitest's equivalent of Jest's `jest.fn()`. 

**Example 2: Jest Configuration with jsdom for Focus Management Tests**

```typescript
// jest.config.ts
import type { Config } from '@jest/types';

const config: Config.InitialOptions = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEach: ['<rootDir>/src/test/setup.ts'],
  transform: {
    '^.+\\.tsx?$': ['ts-jest', { tsconfig: { jsx: 'react-jsx' } }],
  },
};

export default config;
```

```jsx
// Modal.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Modal } from './Modal';

test('should focus the modal when opened', async () => {
  const user = userEvent.setup();
  render(<Modal isOpen={true} />);

  // jsdom provides accurate focus behaviour
  const modal = screen.getByRole('dialog');
  expect(modal).toHaveFocus();
});
```

**Expected Output:** The test passes. jsdom provides accurate focus management, which happy-dom may not fully support. This is why jsdom remains the preferred environment for tests that depend on focus, selection, or other advanced DOM behaviour. 

**Why This Output Occurs:** jsdom implements a more complete DOM API, including accurate focus management, selection ranges, and event dispatching. When a test depends on these behaviours, jsdom is the safer choice. The `@testing-library/user-event` library uses these APIs to simulate real user interactions.

### Real-World Cases

- **Next.js applications:** Using Vitest with `jsdom` and `vite-tsconfig-paths` for path alias support. 
- **Create React App migrations:** Migrating from Jest + jsdom to Vitest + happy-dom for faster test execution.
- **Design system libraries:** Using jsdom for accurate focus and selection testing, and happy-dom for faster rendering tests.
- **Large monorepos:** Using Vitest's workspace feature to run tests across multiple packages with different environments.
- **CI pipelines:** Configuring `coverage.thresholds` in Vitest to enforce minimum coverage and fail the build if thresholds are not met. 

---

## Core Concept 4: Test Isolation vs. Setup

### Definitions

**Core Definition:** Test Isolation vs. Setup is the architectural decision of how much a component test should be isolated from its dependencies versus how much it should be wrapped in realistic application contexts such as providers, routers, and state stores.

**Technical Definition:** Pure isolation in component testing means rendering a component with mocked dependencies (mocked API calls, mocked context values, mocked router hooks) to test its logic in a controlled environment. Realistic setup means wrapping the component in its real providers—`MemoryRouter` for routing, `ThemeProvider` for theming, `AuthProvider` for authentication, `QueryClientProvider` for data fetching—using a custom `render` utility (often called `renderWithProviders` or `setupRender`) that encapsulates the application's provider tree.  The tradeoff is between control (mocking gives you precise control over dependencies) and confidence (real providers give you confidence that the component works in the actual application).  The recommended approach is to use real providers by default and mock only at the network boundary (e.g., with Mock Service Worker). 

**Beginner-Friendly Explanation:** When you test a component, you have two choices. You can isolate it completely—mock everything it depends on so you control exactly what happens. Or you can set up a realistic context—wrap it in the same providers your real app uses, so it behaves just like it would for a real user. The problem with pure isolation is that you end up testing your mocks, not your app. The problem with full realism is that it's harder to control. The best approach is to use real providers but mock the network layer—so your component talks to a fake API, but everything else is real.

### Purposes

- To ensure components render correctly with the same providers they use in production.
- To avoid the false confidence of tests that only verify mocked behaviour.
- To provide a reusable setup utility that eliminates repetitive provider wrapping.
- To balance test isolation (control over dependencies) with test realism (confidence in production behaviour).
- To prevent tests from failing due to missing context providers (e.g., "useRouter must be used within a Router").
- To mock only at the network boundary, keeping the rest of the application real.

### Syntax Rules and Structure

**General Syntax for a Custom Render Utility:** 

```jsx
// test-utils.tsx
import React, { ReactElement, ReactNode } from 'react';
import { render, RenderOptions, RenderResult } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import { ThemeProvider } from './theme';
import { AuthProvider } from './auth';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  theme?: 'light' | 'dark';
  user?: User | null;
  initialEntries?: string[];
}

function customRender(
  ui: ReactElement,
  options?: CustomRenderOptions
): RenderResult {
  const {
    theme = 'light',
    user = null,
    initialEntries = ['/'],
    ...renderOptions
  } = options ?? {};

  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  function Wrapper({ children }: { children: ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        <AuthProvider user={user}>
          <ThemeProvider theme={theme}>
            <MemoryRouter initialEntries={initialEntries}>
              {children}
            </MemoryRouter>
          </ThemeProvider>
        </AuthProvider>
      </QueryClientProvider>
    );
  }

  return render(ui, { wrapper: Wrapper, ...renderOptions });
}

// Re-export everything from Testing Library
export * from '@testing-library/react';
export { customRender as render };
```

**Component Breakdown:**
- `CustomRenderOptions`: Extends Testing Library's `RenderOptions` with app-specific options (theme, user, initial route).
- `QueryClient`: A fresh instance per test to ensure isolation. 
- `MemoryRouter`: Provides routing context without touching the browser's URL. 
- `AuthProvider` / `ThemeProvider`: Real application providers with configurable values.
- `render(ui, { wrapper: Wrapper })`: Testing Library's built-in wrapper option. 

**General Syntax for Mocking at the Network Boundary (MSW):**

```javascript
// mocks/handlers.js
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/user', () => {
    return HttpResponse.json({ id: 1, name: 'Alice' });
  }),
];

// mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/test/setup.ts
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Component Breakdown:**
- `http.get('/api/user', ...)`: Intercepts network requests at the HTTP level.
- `setupServer(...handlers)`: Creates an MSW server for Node.js (test environment).
- `server.listen()` / `server.resetHandlers()` / `server.close()`: Lifecycle hooks for the mock server. 

**Syntax Rules:**
- Create a single `test-utils.tsx` file that exports a custom `render` function with all application providers.
- Always create a fresh `QueryClient` (or equivalent state store) per test to ensure isolation. 
- Use `MemoryRouter` for components that depend on routing; it provides routing context without browser URL manipulation. 
- Mock at the network boundary (MSW, `fetch` mock) rather than mocking internal modules or functions. 
- Avoid mocking internal application modules; if you mock a function that Component A imports from `utils.js`, your test is coupled to the internal architecture. 
- For context values, prefer wrapping in the real provider with a fixed value over mocking the context itself. 

**Constraints and Limitations:**
- A large provider wrapper can slow down tests if the providers do heavy initialisation.
- MSW adds a dependency and requires handler setup; for simple cases, `vi.fn()` or `jest.fn()` on `fetch` may suffice.
- Some providers (e.g., `QueryClientProvider`) require a fresh instance per test to avoid state leakage. 
- Over-wrapping can make tests slow; include only the providers the component actually needs.
- Testing Library's `wrapper` option applies to the entire render; if different tests need different providers, use multiple custom render functions.

### Annotated Code Examples

**Example 1: Custom Render with Providers vs. Pure Isolation**

```jsx
// ❌ PURE ISOLATION: Mocking everything
test('should display user name', () => {
  // Mock the auth context
  vi.mock('./AuthContext', () => ({
    useAuth: () => ({ user: { name: 'Alice' } }),
  }));

  render(<UserProfile />);
  expect(screen.getByText('Alice')).toBeInTheDocument();
});
// This tests the mock, not the real component

// ✅ REALISTIC SETUP: Real providers, mocked network
import { render, screen } from './test-utils'; // custom render
import { server } from './mocks/server';
import { http, HttpResponse } from 'msw';

test('should display the logged-in user name', async () => {
  // Override the default handler for this test
  server.use(
    http.get('/api/user', () => {
      return HttpResponse.json({ name: 'Alice' });
    })
  );

  render(<UserProfile />);

  // Wait for the async data to load and render
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

**Expected Output:** The pure isolation test passes even if the `AuthContext` integration is broken, because it replaces the context entirely with a mock. The realistic setup test passes only if the component correctly fetches from the API and renders the user's name, providing genuine confidence. 

**Why This Output Occurs:** The pure isolation test mocks `useAuth` directly, so it never exercises the real authentication flow. The realistic test uses the real `AuthProvider`, real routing, and real rendering, intercepting only the network request with MSW. This means the test verifies the actual integration between the component, the auth context, and the data layer. 

**Example 2: Reusable `renderWithProviders` Utility**

```jsx
// test-utils.tsx
import React from 'react';
import { render } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { MemoryRouter } from 'react-router-dom';
import { AuthProvider } from '../context/AuthContext';

export function renderWithProviders(
  ui,
  {
    user = null,
    route = '/',
    queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    }),
    ...renderOptions
  } = {}
) {
  function Wrapper({ children }) {
    return (
      <QueryClientProvider client={queryClient}>
        <AuthProvider user={user}>
          <MemoryRouter initialEntries={[route]}>
            {children}
          </MemoryRouter>
        </AuthProvider>
      </QueryClientProvider>
    );
  }

  return render(ui, { wrapper: Wrapper, ...renderOptions });
}

// UserProfile.test.jsx
import { screen } from '@testing-library/react';
import { renderWithProviders } from './test-utils';
import { UserProfile } from './UserProfile';

test('should render the user profile with the authenticated user', () => {
  renderWithProviders(<UserProfile />, {
    user: { name: 'Alice', email: 'alice@example.com' },
  });

  expect(screen.getByText('Alice')).toBeInTheDocument();
  expect(screen.getByText('alice@example.com')).toBeInTheDocument();
});

test('should redirect to login when not authenticated', () => {
  renderWithProviders(<UserProfile />, {
    user: null,
    route: '/profile',
  });

  // The component should render a redirect or login prompt
  expect(screen.getByText(/log in/i)).toBeInTheDocument();
});
```

**Expected Output:** Both tests pass. The first test renders the profile with a mock user, verifying that the name and email appear. The second test renders the profile with no user and verifies that a login prompt appears.

**Why This Output Occurs:** The `renderWithProviders` utility wraps every component in the real providers (`QueryClientProvider`, `AuthProvider`, `MemoryRouter`) with configurable values. This ensures that the component under test behaves exactly as it would in the real application, while still allowing per-test customisation (user, route, query client). The fresh `QueryClient` per test ensures isolation. 

### Real-World Cases

- **Next.js applications:** Wrapping components in `QueryClientProvider` and `SessionProvider` for integration tests.
- **React Router applications:** Using `MemoryRouter` with `initialEntries` to test components that depend on route parameters.
- **Redux applications:** Wrapping components in a fresh Redux store with preloaded state for each test. 
- **Multi-provider applications:** Creating a single `renderWithProviders` utility that composes all application providers, eliminating repetitive setup.
- **MSW integration:** Using Mock Service Worker to mock API calls at the network level while keeping all other providers real. 

---

## References

- Testing Strategy — The Test Trophy – Tyler-R-Kendrick (GitHub): https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/tyler-r-kendrick/agent-skills/testing/SKILL.md
- React Testing Best Practices – RT Camp: https://rtcamp.com/handbook/react-best-practices/testing/
- Vitest – Test Environment Configuration: https://main.vitest.dev/config/environment
- Vitest – Test Environment Guide: https://vitest.dev/guide/environment
- Page Navigation – Epic Web (React Component Testing with Vitest): https://www.epicweb.dev/workshops/react-component-testing-with-vitest/best-practices/page-navigation/solution
- Vitest and Testing Library Unit/Component Review – Vincent Chu Wai Chow (GitHub): https://raw.githubusercontent.com/VincentChuWaiChow/vanguard-frontier-agentic/refs/heads/master/skills/frontend/frontend-testing-strategy-review/references/unit-component-framework-review.md
- How to set up Vitest with Next.js – Next.js Documentation: https://nextjs.org/docs/15/pages/guides/testing/vitest
- TypeScript Patterns for React Testing – Steve Kinney: https://stevekinney.com/courses/react-typescript/typescript-react-testing
- Testing Library – Guiding Principles: https://testing-library.com/docs/guiding-principles
- Kent C. Dodds – Write Tests. Not Too Many. Mostly Integration.: https://kentcdodds.com/blog/write-tests
- Kent C. Dodds – The Testing Trophy and Testing Classifications: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Martin Fowler – The Practical Test Pyramid: https://martinfowler.com/articles/practical-test-pyramid.html
- React Testing Library – Query Priority: https://testing-library.com/docs/queries/about/#priority
- happy-dom – GitHub: https://github.com/capricorn86/happy-dom
- jsdom – GitHub: https://github.com/jsdom/jsdom
- Mock Service Worker (MSW) – Documentation: https://mswjs.io/docs/
- Vitest – Coverage Configuration: https://vitest.dev/config/#coverage