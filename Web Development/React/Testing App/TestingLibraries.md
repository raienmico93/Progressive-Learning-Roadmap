# Testing Libraries — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Testing libraries are software packages that provide the infrastructure, APIs, and utilities needed to write, run, and verify automated tests — including test runners, assertion libraries, DOM utilities, mocking tools, and network interceptors.

**Technical Definition:** A testing ecosystem comprises several distinct layers: **test runners** (Jest, Vitest) that discover, execute, and report on test files; **assertion libraries** (Expect, Chai) that provide the vocabulary for verifying outcomes; **DOM testing utilities** (React Testing Library, Enzyme) that render and query UI components; **user interaction simulators** (`@testing-library/user-event`) that dispatch realistic browser events; **network mocking tools** (MSW, Mirage JS, Nock) that intercept and control HTTP traffic; and **mock/spy libraries** (Sinon) that replace and observe function behavior. Each layer solves a specific problem; a complete test setup combines one tool from each relevant layer.

**Beginner-Friendly Explanation:** Testing libraries are the tools that help you write and run tests. Some run your tests (like Jest or Vitest), some let you check results (like Expect), some let you render components (like React Testing Library), and some let you fake network requests (like MSW). A good test setup picks the right tool for each job.

### Key Characteristics

- **Layered ecosystem:** Runners, assertions, DOM utilities, interaction simulators, and network mocks are distinct categories.
- **Framework-coupled:** React Testing Library is React-specific; Jest and Vitest are framework-agnostic.
- **Philosophy-driven:** Testing Library enforces user-centric testing; Enzyme's shallow rendering reflects a different philosophy.
- **Interchangeable in places:** Vitest and Jest have nearly identical APIs; Chai and Expect serve similar purposes.
- **Deprecation matters:** Enzyme is effectively deprecated for React 18+; `@testing-library/react-hooks` is deprecated in favor of `renderHook` in `@testing-library/react`.

### Prerequisites

- A JavaScript/TypeScript project with Node.js and a package manager.
- A React application (for React-specific tools).
- Understanding of the test pyramid and the distinction between unit, integration, and E2E tests.
- Familiarity with modern JavaScript (Promises, async/await, modules).

### Related Programming Areas

- **Unit testing** — Jest/Vitest runners with Expect/Chai assertions.
- **Component testing** — React Testing Library, user-event, jest-dom.
- **Integration testing** — MSW, custom `renderWithProviders` helpers.
- **E2E testing** — Playwright and Cypress (separate ecosystems, covered elsewhere).
- **Mocking and stubbing** — Sinon, MSW, Nock, Mirage JS.

### Core Concepts / Features

---

## 1. Jest

### Definitions

**Core Definition:** Jest is a JavaScript testing framework — a test runner with built-in assertions, mocking, and snapshot testing — developed by Meta and widely adopted for React applications.

**Technical Definition:** Jest discovers test files by pattern (`*.test.js`, `__tests__/`), executes them in isolated environments (jsdom for DOM tests, node for server tests), and reports results. It includes an assertion library (`expect`), a mocking system (`jest.fn()`, `jest.mock()`, `jest.spyOn()`), snapshot testing, fake timers, and coverage reporting (via Istanbul). Jest runs tests in parallel across worker processes but executes test files serially within each worker.

**Beginner-Friendly Explanation:** Jest is a tool that runs your tests and tells you if they pass or fail. It comes with everything you need — ways to check results, fake functions, and even the ability to "snapshot" UI output.

### Purposes

- To run and report on automated tests in a JavaScript/TypeScript project.
- To provide built-in assertions, mocking, and snapshot testing without additional dependencies.
- To integrate with Babel and TypeScript for transpiling test files.
- To measure test coverage with Istanbul.

### Syntax Rules and Structure

```javascript
// sum.js
export function sum(a, b) { return a + b; }

// sum.test.js
test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `test(name, fn)` / `it(name, fn)` | Defines a test case. |
| `describe(name, fn)` | Groups related tests. |
| `expect(value)` | Creates an assertion. |
| `jest.fn()` | Creates a mock function. |
| `jest.mock('module')` | Auto-mocks a module. |
| `beforeEach` / `afterEach` | Setup/teardown hooks. |

**Constraints and Limitations:**

- Historically required Babel for ESM and TypeScript support; configuration-heavy compared to Vitest.
- Slower startup and watch mode than Vitest, especially in large projects.
- ESM support has improved but remains less seamless than Vitest's native ESM.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing with Mocks and Async

```javascript
// userService.js
export async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}

// userService.test.js
import { fetchUser } from './userService';

test('fetches user by id', async () => {
  global.fetch = jest.fn().mockResolvedValue({
    json: () => Promise.resolve({ id: 1, name: 'Alice' }),
  });

  const user = await fetchUser(1);

  expect(user).toEqual({ id: 1, name: 'Alice' });
  expect(global.fetch).toHaveBeenCalledWith('/api/users/1');
});
```

**Expected Output:** The test passes. `jest.fn()` replaces `fetch`, and the assertions verify both the returned data and the call arguments.

### Real-World Cases

- **Legacy React projects:** Many existing CRA and older React codebases use Jest.
- **Node.js backend testing:** Jest works well for Express/Node services.
- **Snapshot testing:** Jest's snapshot feature is widely used for serialized output.

### References

- Jest — Official Documentation — https://jestjs.io/docs/getting-started
- Jest — API Reference — https://jestjs.io/docs/api

---

## 2. Vitest

### Definitions

**Core Definition:** Vitest is a Vite-native testing framework — a fast test runner with a Jest-compatible API, native ESM and TypeScript support, and built-in browser mode.

**Technical Definition:** Vitest reuses Vite's transform pipeline and module resolution, so tests run against the same code as the application without a separate build step. It provides a Jest-compatible API (`describe`, `it`, `expect`, `vi.fn()`, `vi.mock()`), snapshot testing, coverage (via v8 or Istanbul), and a **browser mode** that runs tests in real browsers via Playwright. It runs tests in worker threads, supports in-source testing, and requires no Babel configuration for TypeScript or JSX.

**Beginner-Friendly Explanation:** Vitest is a modern, fast alternative to Jest. It uses the same API as Jest, so it is easy to switch, but it works natively with Vite and TypeScript — no extra configuration.

### Purposes

- To run tests in a Vite-powered project with native ESM and TypeScript support.
- To provide a faster alternative to Jest with minimal configuration.
- To run tests in real browsers via Vitest Browser Mode.
- To reuse the application's Vite config (plugins, aliases, transforms) in tests.

### Syntax Rules and Structure

```javascript
// sum.test.js
import { describe, it, expect, vi } from 'vitest';
import { sum } from './sum.js';

describe('sum', () => {
  it('adds 1 + 2 to equal 3', () => {
    expect(sum(1, 2)).toBe(3);
  });

  it('uses a mock', () => {
    const fn = vi.fn();
    fn(42);
    expect(fn).toHaveBeenCalledWith(42);
  });
});
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `describe`, `it`, `test` | Same as Jest. |
| `expect` | Same as Jest. |
| `vi.fn()` | Mock function (Vitest equivalent of `jest.fn()`). |
| `vi.mock()` | Module mocking. |
| `vi.advanceTimersByTime()` | Fake timer advancement. |
| `vi.useFakeTimers()` | Enable fake timers. |

**Constraints and Limitations:**

- Smaller ecosystem than Jest — some Jest plugins have no direct equivalent.
- Browser Mode requires Playwright providers and additional setup.
- Some Jest-specific APIs (e.g., `jest.requireActual`) differ slightly in Vitest.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Hook with Vitest and RenderHook

```javascript
// useCounter.test.js
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('increments the count', () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => result.current.increment());
    expect(result.current.count).toBe(1);
  });
});
```

**Expected Output:** The test passes. `renderHook` renders the Hook, `act` flushes the state update, and `expect` verifies the new count.

### Real-World Cases

- **Vite/React projects:** Vitest is the natural choice for new Vite-powered applications in 2026.
- **Monorepos:** Vitest's speed and workspace support make it suitable for large monorepos.
- **Browser-mode testing:** Vitest Browser Mode runs tests in real browsers without a separate E2E setup.

### References

- Vitest — Official Documentation — https://vitest.dev/guide/
- Vitest — Jest Compatibility — https://vitest.dev/guide/migration.html
- Bunnyshell — E2E Testing Frameworks & Tools Compared for 2026 — https://www.bunnyshell.com/blog/e2e-testing-frameworks-tools/

---

## 3. React Testing Library

### Definitions

**Core Definition:** React Testing Library (RTL) is a DOM testing utility for React that renders components and queries the resulting DOM using user-centric selectors, encouraging tests that resemble how users interact with the application.

**Technical Definition:** RTL is built on top of `@testing-library/dom` and provides React-specific rendering (`render`), cleanup, and `renderHook`. Its guiding principle is that "the more your tests resemble the way your software is used, the more confidence they can give you". It queries the DOM via `screen` using accessible roles, labels, and text rather than implementation details, and it works with any test runner (Jest, Vitest, Mocha, etc.).

**Beginner-Friendly Explanation:** React Testing Library helps you render components and find elements the way a user would — by their role, label, or text. It encourages tests that check what users see, not how the component is built inside.

### Purposes

- To render React components into a DOM environment for testing.
- To query the DOM using accessible, user-centric selectors.
- To encourage tests that survive refactoring of implementation details.
- To integrate with user-event for realistic interactions and jest-dom for DOM assertions.

### Syntax Rules and Structure

```jsx
import { render, screen } from '@testing-library/react';

render(<Button label="Submit" />);
expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `render(ui, options)` | Renders a component into a container. |
| `screen` | Queries the document; preferred over destructured queries. |
| `getByRole`, `getByLabelText`, `getByText` | User-centric queries. |
| `findBy*`, `queryBy*` | Async and absence queries. |
| `renderHook` | Renders a custom Hook in isolation. |
| `cleanup` | Automatically called after each test. |

**Constraints and Limitations:**

- Does not provide a test runner or assertion library — pair with Jest/Vitest and jest-dom.
- Shallow rendering (Enzyme-style) is not supported — RTL renders the full component tree.
- Snapshot tests are discouraged unless justified.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Component with Accessible Queries

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect } from 'vitest';
import { LoginForm } from './LoginForm';

test('submits the form', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn().mockResolvedValue(undefined);

  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(onSubmit).toHaveBeenCalledWith({ email: 'alice@example.com' });
});
```

**Expected Output:** The test passes. It queries the email input by its label and the submit button by its role, then verifies the submit handler.

### Real-World Cases

- **Component libraries:** Every component test in a design system uses RTL's accessible queries.
- **Form testing:** `getByLabelText` ensures every input has a properly associated label.
- **Accessibility regression:** If a query fails, the UI may be inaccessible to screen readers.

### References

- React Testing Library — Introduction — https://testing-library.com/docs/react-testing-library/intro
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority

---

## 4. Enzyme

### Definitions

**Core Definition:** Enzyme is a JavaScript testing utility for React — developed by Airbnb — that provides shallow, mount, and render methods for testing component output, state, and lifecycle behavior.

**Technical Definition:** Enzyme renders React components in three modes: `shallow` (renders only the top-level component, not children), `mount` (full DOM rendering, requires jsdom), and `render` (static HTML rendering). It exposes component internals — `state()`, `props()`, `setState()`, `instance()` — allowing tests to inspect and manipulate implementation details. Enzyme has official adapters only up to React 16. Enzyme is effectively unmaintained for React 17+ and has no React 18+ support; Airbnb has archived the repository.

**Beginner-Friendly Explanation:** Enzyme is an older tool for testing React components. It let you "shallow render" a component without its children and inspect its internal state directly. However, it is no longer maintained for modern React — most projects have migrated to React Testing Library.

### Purposes

- To shallow-render components and test them in isolation from children (historical use case).
- To inspect component state and props directly (implementation-coupled testing).
- To maintain legacy test suites written before React Testing Library became the standard.

### Syntax Rules and Structure

```javascript
// Legacy Enzyme example — NOT recommended for new projects
import { shallow } from 'enzyme';
import { Button } from './Button';

const wrapper = shallow(<Button label="Click" />);
expect(wrapper.find('button').text()).toBe('Click');
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `shallow(<Component />)` | Renders only the component, not its children. |
| `mount(<Component />)` | Full DOM rendering (requires jsdom). |
| `wrapper.find(selector)` | Finds elements by CSS selector or component. |
| `wrapper.state()` | Reads internal component state. |
| `wrapper.simulate('click')` | Simulates an event (basic, not realistic). |

**Constraints and Limitations:**

- **Deprecated for modern React:** No official adapter for React 17+; Airbnb archived the repository.
- Shallow rendering tests implementation, not behavior — tests break on refactors.
- `simulate()` does not simulate realistic browser events (no focus, no pointer events).
- The Enzyme community has largely migrated to React Testing Library.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Legacy Enzyme Test (For Reference Only)

```javascript
import { shallow } from 'enzyme';
import { Counter } from './Counter';

test('increments internal state', () => {
  const wrapper = shallow(<Counter />);
  expect(wrapper.state('count')).toBe(0);
  wrapper.find('button').simulate('click');
  expect(wrapper.state('count')).toBe(1);
});
```

**Expected Output:** The test passes on Enzyme-supported React versions, but it asserts on internal state (`count`) — a brittle pattern that breaks when the implementation changes.

**Migration note:** The equivalent React Testing Library test asserts on the displayed count, not the internal state.

### Real-World Cases

- **Legacy codebases:** Projects with extensive Enzyme suites are migrating to RTL incrementally.
- **Historical context:** Understanding Enzyme helps maintain older React projects.

### References

- Enzyme — GitHub Repository (archived) — https://github.com/enzymejs/enzyme
- Enzyme — Migration to React Testing Library — https://testing-library.com/docs/react-testing-library/migrate-from-enzyme/

---

## 5. Testing Library

### Definitions

**Core Definition:** Testing Library is a family of DOM testing utilities — for React, Vue, Angular, Svelte, and more — that share a common philosophy: test the software the way users use it.

**Technical Definition:** The Testing Library family is built on `@testing-library/dom`, which provides framework-agnostic queries (`getByRole`, `getByLabelText`, etc.) and the `screen` object. Framework adapters — `@testing-library/react`, `@testing-library/vue`, `@testing-library/angular`, `@testing-library/svelte` — add framework-specific rendering. All adapters share the same query API and guiding principles: accessible queries, user-centric tests, and avoidance of implementation details.

**Beginner-Friendly Explanation:** Testing Library is a collection of tools for different frameworks that all work the same way. Whether you use React, Vue, or Angular, you query by role and label, and your tests look similar.

### Purposes

- To provide a consistent, user-centric testing API across frameworks.
- To encourage accessible markup through role and label queries.
- To decouple DOM queries from framework-specific rendering.
- To standardize testing practices across a multi-framework organization.

### Syntax Rules and Structure

```jsx
// React
import { render, screen } from '@testing-library/react';
render(<Button />);
screen.getByRole('button');

// Vue
import { render, screen } from '@testing-library/vue';
render(Button);
screen.getByRole('button');
```

**Component Breakdown:**

| Package | Framework |
|---------|-----------|
| `@testing-library/dom` | Framework-agnostic queries. |
| `@testing-library/react` | React adapter. |
| `@testing-library/vue` | Vue adapter. |
| `@testing-library/angular` | Angular adapter. |
| `@testing-library/svelte` | Svelte adapter. |

**Constraints and Limitations:**

- Each adapter has framework-specific rendering APIs, but the query API is shared.
- Not all framework adapters are equally mature or actively maintained.

### Real-World Cases

- **Multi-framework organizations:** Teams using React and Vue can share testing conventions.
- **Design systems:** Component libraries across frameworks use the same query philosophy.

### References

- Testing Library — Official Documentation — https://testing-library.com/docs/
- Testing Library — Framework Adapters — https://testing-library.com/docs/dom-testing-library/intro

---

## 6. `@testing-library/react`

### Definitions

**Core Definition:** `@testing-library/react` is the React adapter for Testing Library — it provides `render`, `screen`, `renderHook`, and automatic cleanup for React components and Hooks.

**Technical Definition:** This package wraps `@testing-library/dom` with React-specific rendering. `render()` mounts the component using React's `createRoot`, appends the container to `document.body`, and returns query utilities. `renderHook()` renders a custom Hook in a minimal test harness. Automatic cleanup runs after each test via the test framework's `afterEach` global, unmounting the rendered tree.

**Beginner-Friendly Explanation:** `@testing-library/react` is the package you import when testing React components. It gives you `render` to mount components and `screen` to find elements.

### Purposes

- To render React components and Hooks into a DOM environment.
- To provide the `screen` query object for user-centric DOM queries.
- To enable `renderHook` for testing custom Hooks in isolation.
- To ensure automatic cleanup between tests.

### Syntax Rules and Structure

```jsx
import { render, screen, renderHook } from '@testing-library/react';

render(<Component />);
screen.getByRole('button');

const { result } = renderHook(() => useCounter());
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `render(ui, options)` | Mounts a component; returns queries and `rerender`/`unmount`. |
| `screen` | Queries the entire document (works with portals). |
| `renderHook(callback, options)` | Renders a custom Hook. |
| `cleanup()` | Automatic after each test — no manual call needed. |

**Constraints and Limitations:**

- Does not include `user-event` — install `@testing-library/user-event` separately.
- Does not include DOM matchers — install `@testing-library/jest-dom` separately.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Rendering and Querying a Component

```jsx
import { render, screen } from '@testing-library/react';
import { Greeting } from './Greeting';

test('renders the greeting', () => {
  render(<Greeting name="Alice" />);
  expect(screen.getByText('Hello, Alice!')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. `render` mounts the component, and `screen.getByText` finds the greeting.

### Real-World Cases

- **Component tests:** The foundation of every React component test.
- **Hook tests:** `renderHook` for custom Hooks.
- **Portal testing:** `screen` queries the whole document, including portal content.

### References

- React Testing Library — render API — https://testing-library.com/docs/react-testing-library/api#render
- React Testing Library — Setup — https://testing-library.com/docs/react-testing-library/setup

---

## 7. `@testing-library/user-event`

### Definitions

**Core Definition:** `@testing-library/user-event` is a user interaction simulation library that dispatches the full sequence of events a real browser produces for interactions like clicking, typing, and keyboard navigation.

**Technical Definition:** Unlike `fireEvent` (which dispatches a single DOM event), `user-event` simulates the complete interaction sequence. For example, `user.click()` fires focus, pointerdown, mousedown, pointerup, mouseup, and click, and respects element visibility and disabled state. `user.type()` fires keydown, keypress, input, and keyup for each character, respecting `maxLength`, `disabled`, and keyboard layouts. `userEvent.setup()` must be called before rendering, and all interactions must be awaited.

**Beginner-Friendly Explanation:** `user-event` simulates user actions the way a real browser does — a full click, character-by-character typing, and proper keyboard events. It catches bugs that a simple `fireEvent.click` would miss.

### Purposes

- To simulate realistic user interactions that exercise full event sequences.
- To catch bugs in focus management, keyboard handlers, and input behavior.
- To respect element state (disabled, hidden, read-only) as a real browser would.
- To provide a higher-fidelity alternative to `fireEvent`.

### Syntax Rules and Structure

```jsx
import userEvent from '@testing-library/user-event';

const user = userEvent.setup();
await user.click(screen.getByRole('button', { name: /submit/i }));
await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
await user.keyboard('{Enter}');
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `userEvent.setup()` | Creates a user session — call before render. |
| `user.click(element)` | Full click sequence. |
| `user.type(element, text)` | Character-by-character typing. |
| `user.keyboard(keys)` | Keyboard events (e.g., `'{Enter}'`). |
| `user.tab()` | Moves focus to the next element. |
| `user.hover(element)` | Simulates hover. |

**Constraints and Limitations:**

- All calls must be awaited.
- `user-event` is slower than `fireEvent` because it dispatches more events — this is intentional.
- Some low-level events (e.g., `scroll`) are not supported — use `fireEvent` for those.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typing and Keyboard Interaction

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { SearchBox } from './SearchBox';

test('searches on Enter', async () => {
  const user = userEvent.setup();
  const onSearch = vi.fn();

  render(<SearchBox onSearch={onSearch} />);

  await user.type(screen.getByRole('searchbox'), 'react');
  await user.keyboard('{Enter}');

  expect(onSearch).toHaveBeenCalledWith('react');
});
```

**Expected Output:** The test passes. `user.type` fires character-by-character events, and `user.keyboard('{Enter}')` triggers the search.

### Real-World Cases

- **Form testing:** Typing, clearing, and submitting forms with realistic events.
- **Keyboard navigation:** Testing tab order and Enter/Escape handlers.
- **Disabled states:** Verifying that user-event refuses to interact with disabled elements.

### References

- Testing Library — user-event Introduction — https://testing-library.com/docs/user-event/intro
- Testing Library — fireEvent vs user-event — https://testing-library.com/docs/user-event/intro#differences-from-fireevent

---

## 8. `@testing-library/jest-dom`

### Definitions

**Core Definition:** `@testing-library/jest-dom` is a collection of custom Jest/Vitest matchers that make DOM assertions more readable and semantically meaningful (e.g., `toBeInTheDocument`, `toBeDisabled`, `toHaveTextContent`).

**Technical Definition:** jest-dom extends the `expect` object with DOM-specific matchers that understand HTML semantics. For example, `toBeVisible()` accounts for CSS display/visibility and ancestor hidden states; `toBeDisabled()` checks not just the `disabled` attribute but inherited disabled state from fieldset; `toHaveValue()` reads the actual input value. It works with both Jest and Vitest (via `@testing-library/jest-dom/vitest`).

**Beginner-Friendly Explanation:** jest-dom adds helpful matchers like `toBeInTheDocument()` and `toBeDisabled()` so your assertions read like plain English instead of checking raw DOM properties.

### Purposes

- To provide readable, semantic DOM assertions.
- To handle HTML edge cases (visible/disabled/value) correctly.
- To reduce boilerplate in DOM assertions.
- To work seamlessly with Jest and Vitest.

### Syntax Rules and Structure

```jsx
import '@testing-library/jest-dom';

expect(screen.getByText('Hello')).toBeInTheDocument();
expect(screen.getByRole('button')).toBeDisabled();
expect(screen.getByRole('textbox')).toHaveValue('Alice');
expect(screen.getByRole('checkbox')).toBeChecked();
```

**Component Breakdown:**

| Matcher | Assertion |
|---------|-----------|
| `toBeInTheDocument()` | Element is in the DOM. |
| `toBeVisible()` | Element is visible (CSS-aware). |
| `toBeDisabled()` / `toBeEnabled()` | Form control state (inherits fieldset). |
| `toHaveTextContent(text)` | Text content. |
| `toHaveValue(value)` | Input/textarea value. |
| `toBeChecked()` | Checkbox/radio state. |
| `toHaveAttribute(name, value)` | Attribute value. |

**Constraints and Limitations:**

- Must be imported in the test setup file (or per test file) to register the matchers.
- For Vitest, use `@testing-library/jest-dom/vitest` to register matchers correctly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Semantic DOM Assertions

```jsx
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import { SubmitButton } from './SubmitButton';

test('button is disabled while submitting', () => {
  render(<SubmitButton isSubmitting={true} />);
  expect(screen.getByRole('button', { name: /submit/i })).toBeDisabled();
});
```

**Expected Output:** The test passes. `toBeDisabled()` correctly detects the disabled state, including inherited disabled state from a fieldset.

### Real-World Cases

- **Form states:** Asserting disabled, checked, and value states with semantic matchers.
- **Visibility:** `toBeVisible()` for conditional UI.
- **Accessibility:** Combining `toHaveAccessibleName()` with role queries.

### References

- jest-dom — GitHub Repository — https://github.com/testing-library/jest-dom
- jest-dom — Custom Matchers — https://github.com/testing-library/jest-dom#custom-matchers

---

## 9. `@testing-library/react-hooks`

### Definitions

**Core Definition:** `@testing-library/react-hooks` is a deprecated library that provided `renderHook` and `act` for testing React Hooks before those utilities were built into `@testing-library/react`.

**Technical Definition:** The library was created to test Hooks in isolation when React's own testing ecosystem lacked a first-class solution. It provided `renderHook`, `act`, and utilities for testing hooks with `Suspense` and error boundaries. In React 18, `@testing-library/react` added its own `renderHook` and `act`, and the `react-hooks` package was deprecated and archived. Projects should migrate to `@testing-library/react`'s built-in `renderHook`.

**Beginner-Friendly Explanation:** `@testing-library/react-hooks` was an older package for testing Hooks. It is no longer maintained — you should use the `renderHook` that comes with `@testing-library/react` instead.

### Purposes

- **Historical:** To test custom Hooks in isolation before React Testing Library supported it natively.
- **Migration:** To understand legacy test code and migrate it to the current `renderHook`.

### Syntax Rules and Structure

```javascript
// DEPRECATED — do not use in new projects
import { renderHook, act } from '@testing-library/react-hooks';

const { result } = renderHook(() => useCounter());
act(() => result.current.increment());
expect(result.current.count).toBe(1);
```

**Migration to current API:**

```javascript
// RECOMMENDED — use @testing-library/react
import { renderHook, act } from '@testing-library/react';

const { result } = renderHook(() => useCounter());
act(() => result.current.increment());
expect(result.current.count).toBe(1);
```

**Constraints and Limitations:**

- **Deprecated and archived** — no new releases, no React 18+ support.
- The migration is straightforward: change the import path.
- Some legacy utilities (e.g., `waitForNextUpdate`) have no direct equivalent — use `waitFor` from `@testing-library/react`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Migration from react-hooks to Testing Library

```javascript
// BEFORE (deprecated)
import { renderHook, act, waitForNextUpdate } from '@testing-library/react-hooks';

test('async hook', async () => {
  const { result, waitForNextUpdate } = renderHook(() => useAsyncData());
  await waitForNextUpdate();
  expect(result.current.data).toBeDefined();
});

// AFTER (current)
import { renderHook, waitFor } from '@testing-library/react';

test('async hook', async () => {
  const { result } = renderHook(() => useAsyncData());
  await waitFor(() => expect(result.current.data).toBeDefined());
});
```

**Expected Output:** Both versions pass, but the current version uses `waitFor` from `@testing-library/react`.

### Real-World Cases

- **Legacy migration:** Updating old test suites to use the current `renderHook`.
- **Documentation:** Understanding why older tutorials import from `@testing-library/react-hooks`.

### References

- React Hooks Testing Library — Deprecation Notice — https://github.com/testing-library/react-hooks-testing-library
- React Testing Library — renderHook — https://testing-library.com/docs/react-testing-library/api#renderhook

---

## 10. MSW (Mock Service Worker)

### Definitions

**Core Definition:** Mock Service Worker (MSW) is a network-level API mocking library that intercepts HTTP requests using a Service Worker (in the browser) or request interception (in Node.js), returning controlled responses without modifying application code.

**Technical Definition:** MSW registers a Service Worker in the browser that intercepts `fetch` and `XMLHttpRequest` calls at the network layer. In Node.js (tests, SSR), it uses `@mswjs/interceptors` to patch HTTP modules. Handlers are defined with `http.get`, `http.post`, `graphql.query`, etc., and return `HttpResponse` objects. Because interception happens at the network layer, the application code makes real `fetch` calls — the mock is transparent. This makes MSW framework-agnostic and suitable for tests, development, and debugging.

**Beginner-Friendly Explanation:** MSW fakes your API server. Your code makes real network requests, but MSW intercepts them and returns fake responses. This means you do not have to change your code or mock `fetch` directly — the mock happens at the network level.

### Purposes

- To mock API responses at the network layer without modifying application code.
- To provide realistic, framework-agnostic API mocking for tests and development.
- To simulate error responses, delays, and edge cases.
- To reuse the same mock handlers across tests, Storybook, and local development.

### Syntax Rules and Structure

```javascript
// handlers.js
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users/:id', ({ params }) =>
    HttpResponse.json({ id: params.id, name: 'Alice' })
  ),
];

// server.js (Node.js test setup)
import { setupServer } from 'msw/node';
import { handlers } from './handlers';
export const server = setupServer(...handlers);

// test setup
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `http.get/post/...` | Define request handlers by method and URL. |
| `HttpResponse.json()` | Return a JSON response. |
| `setupServer(...handlers)` | Create a Node.js mock server for tests. |
| `server.use(handler)` | Override handlers for a specific test. |
| `server.resetHandlers()` | Restore default handlers after each test. |

**Constraints and Limitations:**

- Service Worker registration requires HTTPS in production (not an issue in tests).
- Handlers must match the exact method and URL pattern.
- MSW does not mock `XMLHttpRequest` in all environments — check compatibility.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Component with MSW

```jsx
import { render, screen } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { UserProfile } from './UserProfile';

test('displays user data from the API', async () => {
  server.use(
    http.get('/api/users/1', () =>
      HttpResponse.json({ id: 1, name: 'Alice' })
    )
  );

  render(<UserProfile userId="1" />);

  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The component makes a real `fetch` call, MSW intercepts it, and the UI renders the mocked data.

### Real-World Cases

- **Integration tests:** Mocking API responses for component and integration tests.
- **Storybook:** Using the same handlers for mocked API data in stories.
- **Local development:** Running the frontend against mocked APIs before the backend is ready.

### References

- MSW — Official Documentation — https://mswjs.io/docs/
- Using Mock Service Worker to Improve Jest Unit Tests — https://www.wwt.com/article/using-mock-service-worker-to-improve-jest-unit-tests

---

## 11. Mirage JS

### Definitions

**Core Definition:** Mirage JS is a client-side API mocking library that provides an in-memory database, relationship modeling, and route handlers for simulating a backend during development and testing.

**Technical Definition:** Mirage JS intercepts HTTP requests (via Pretender in the browser) and routes them to handlers defined in its server configuration. It provides a full in-memory ORM with models, relationships, factories, and serializers, allowing realistic CRUD simulation. It was created by Sam Selikoff and is commonly used in Ember and React applications. Unlike MSW (which focuses on request interception), Mirage JS provides a complete backend simulation with data persistence during the session.

**Beginner-Friendly Explanation:** Mirage JS fakes an entire backend — including a database — right in the browser. You define models and routes, and it simulates a real API with relationships and CRUD operations.

### Purposes

- To simulate a complete backend (with a database) during development and testing.
- To model relationships between resources (users, posts, comments).
- To provide realistic CRUD behavior without a real server.
- To seed test data with factories.

### Syntax Rules and Structure

```javascript
import { createServer, Model } from 'miragejs';

const server = createServer({
  models: { user: Model },

  seeds(server) {
    server.create('user', { name: 'Alice' });
  },

  routes() {
    this.namespace = 'api';

    this.get('/users', (schema) => schema.users.all());
    this.post('/users', (schema, request) => {
      const attrs = JSON.parse(request.requestBody);
      return schema.users.create(attrs);
    });
  },
});
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `createServer(config)` | Creates the mock server. |
| `models` | Defines in-memory data models. |
| `seeds(server)` | Seeds initial data. |
| `routes()` | Defines route handlers with `this.get`, `this.post`, etc. |
| `schema.users.all()` | Queries the in-memory database. |

**Constraints and Limitations:**

- Larger API surface than MSW — more learning curve.
- Data is in-memory and resets on page reload.
- Less commonly used in React ecosystems than MSW; more prevalent in Ember.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Mirage JS Server with Relationships

```javascript
import { createServer, Model, hasMany, belongsTo } from 'miragejs';

createServer({
  models: {
    user: Model.extend({ posts: hasMany() }),
    post: Model.extend({ author: belongsTo('user') }),
  },

  seeds(server) {
    const alice = server.create('user', { name: 'Alice' });
    server.create('post', { title: 'Hello', author: alice });
  },

  routes() {
    this.namespace = 'api';
    this.get('/users', (schema) => schema.users.all());
    this.get('/users/:id/posts', (schema, request) =>
      schema.users.find(request.params.id).posts
    );
  },
});
```

**Expected Output:** `GET /api/users/1/posts` returns the posts belonging to user 1.

### Real-World Cases

- **Ember applications:** Mirage JS originated in the Ember ecosystem.
- **Full backend simulation:** Apps that need realistic CRUD and relationships without a server.
- **Development before backend:** Frontend teams building against a simulated backend.

### References

- Mirage JS — Official Documentation — https://miragejs.com/docs/getting-started/introduction/
- Mirage JS — Models and Relationships — https://miragejs.com/docs/main-concepts/models/

---

## 12. Nock

### Definitions

**Core Definition:** Nock is an HTTP server mocking library for Node.js that intercepts `http` and `https` requests and returns predefined responses.

**Technical Definition:** Nock works by overriding Node's `http.ClientRequest` and `https.ClientRequest` modules. It intercepts outgoing requests, matches them against configured interceptors (method, URL, headers, body), and returns the configured response without making a real network call. It is primarily used for testing Node.js backends and services — not browser-based frontend tests (where MSW is preferred).

**Beginner-Friendly Explanation:** Nock fakes HTTP requests in Node.js. It is useful for testing backend code that makes API calls, like a server that talks to a third-party service.

### Purposes

- To mock HTTP requests in Node.js tests.
- To test backend services that call external APIs.
- To assert that the correct requests were made (method, URL, headers, body).

### Syntax Rules and Structure

```javascript
const nock = require('nock');

const scope = nock('https://api.example.com')
  .get('/users/1')
  .reply(200, { id: 1, name: 'Alice' });

// Request is intercepted
await fetch('https://api.example.com/users/1');

expect(scope.isDone()).toBe(true);
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `nock(baseUrl)` | Creates a scope for the given base URL. |
| `.get(path)` / `.post(path)` | Defines the request matcher. |
| `.reply(status, body)` | Defines the response. |
| `.isDone()` | Verifies all interceptors were used. |

**Constraints and Limitations:**

- Works only in Node.js — not in browsers.
- Does not intercept `fetch` natively in all Node versions — may need `node-fetch` or an interceptor.
- Less realistic than MSW for frontend testing.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Mocking an External API Call

```javascript
import nock from 'nock';
import { getUser } from './userService';

test('fetches user from external API', async () => {
  nock('https://api.example.com')
    .get('/users/1')
    .reply(200, { id: 1, name: 'Alice' });

  const user = await getUser(1);

  expect(user).toEqual({ id: 1, name: 'Alice' });
});
```

**Expected Output:** The test passes. Nock intercepts the request and returns the configured response.

### Real-World Cases

- **Backend API clients:** Testing services that call third-party APIs.
- **Node.js microservices:** Mocking inter-service HTTP calls.
- **CLI tools:** Testing HTTP interactions in command-line tools.

### References

- Nock — GitHub Repository — https://github.com/nock/nock
- Nock — Usage Documentation — https://github.com/nock/nock#usage

---

## 13. Sinon

### Definitions

**Core Definition:** Sinon is a standalone JavaScript library for creating mocks, stubs, spies, and fake timers — independent of any test framework.

**Technical Definition:** Sinon provides `sinon.spy()` (wraps a function, records calls), `sinon.stub()` (replaces a function with programmable behavior), `sinon.mock()` (defines expectations on an object), `sinon.fake()` (creates fake implementations), and `sinon.useFakeTimers()` (controls `Date`, `setTimeout`, `setInterval`). It also includes a built-in assertion API (`sinon.assert.calledWith()`, etc.). It works with any test runner (Jest, Mocha, Vitest) and is commonly used with Mocha/Chai.

**Beginner-Friendly Explanation:** Sinon is a tool for faking and spying on functions. It works with any test framework and provides flexible ways to replace functions, verify calls, and control time.

### Purposes

- To create spies, stubs, and mocks independently of the test runner.
- To control time with fake timers.
- To verify function calls and arguments.
- To replace functions with programmable fakes.

### Syntax Rules and Structure

```javascript
import sinon from 'sinon';

const spy = sinon.spy();
spy('arg');
sinon.assert.calledWith(spy, 'arg');

const stub = sinon.stub(obj, 'method').returns(42);
const clock = sinon.useFakeTimers();
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `sinon.spy()` | Wraps a function and records calls. |
| `sinon.stub(obj, 'method')` | Replaces a method with a configurable fake. |
| `sinon.mock(obj)` | Defines expectations on an object. |
| `sinon.useFakeTimers()` | Controls time-based functions. |
| `sinon.assert` | Assertion helpers. |

**Constraints and Limitations:**

- Jest and Vitest provide similar functionality with `jest.fn()` / `vi.fn()` — Sinon is redundant in those ecosystems.
- Sinon mocks are less commonly used in modern React testing — prefer fakes and MSW.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Sinon Stub and Fake Timers

```javascript
import sinon from 'sinon';
import { createDebouncedFn } from './debounce';

test('debounces calls', () => {
  const clock = sinon.useFakeTimers();
  const fn = sinon.spy();
  const debounced = createDebouncedFn(fn, 300);

  debounced('a');
  debounced('b');
  clock.tick(300);

  sinon.assert.calledOnce(fn);
  sinon.assert.calledWith(fn, 'b');
  clock.restore();
});
```

**Expected Output:** The test passes. The debounced function is called only once — with the last argument — after the fake clock advances by 300ms.

### Real-World Cases

- **Mocha/Chai test suites:** Sinon is a common companion to Mocha.
- **Legacy test code:** Projects that predate Jest's built-in mocking.
- **Complex stubbing:** Scenarios requiring flexible stub behavior across multiple calls.

### References

- Sinon — Official Documentation — https://sinonjs.org/
- Sinon — Fake Timers — https://sinonjs.org/releases/latest/fake-timers/

---

## 14. Chai

### Definitions

**Core Definition:** Chai is a BDD/TDD assertion library for Node.js and browsers that provides expressive assertion styles (`expect`, `should`, `assert`).

**Technical Definition:** Chai offers three assertion interfaces: `expect` (BDD style, e.g., `expect(value).to.equal(3)`), `should` (extends `Object.prototype` with assertion methods), and `assert` (TDD style, e.g., `assert.equal(value, 3)`). It is commonly paired with Mocha and Sinon and supports plugins (e.g., `chai-dom`, `chai-as-promised`) for extended matchers. In modern React ecosystems, Jest/Vitest's built-in `expect` often replaces Chai.

**Beginner-Friendly Explanation:** Chai is a way to write test assertions in a readable style, like `expect(value).to.equal(3)`. It is commonly used with Mocha, but modern frameworks like Jest and Vitest include their own assertions.

### Purposes

- To provide expressive assertions in Mocha/Node.js test suites.
- To support BDD-style readable assertions.
- To extend with plugins for DOM, Promises, and other domains.

### Syntax Rules and Structure

```javascript
import { expect } from 'chai';

expect(1 + 2).to.equal(3);
expect({ a: 1 }).to.deep.equal({ a: 1 });
expect(() => fn()).to.throw('error');
```

**Component Breakdown:**

| Interface | Style |
|-----------|-------|
| `expect(value).to.equal(x)` | BDD style. |
| `value.should.equal(x)` | Should style (extends Object.prototype). |
| `assert.equal(value, x)` | TDD style. |

**Constraints and Limitations:**

- Jest and Vitest include their own `expect` — Chai is redundant in those ecosystems.
- The `should` interface can conflict with other libraries that modify `Object.prototype`.
- Plugin ecosystem is smaller than Jest's matcher ecosystem.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Chai Assertions with Mocha

```javascript
import { expect } from 'chai';
import { sum } from './sum.js';

describe('sum', () => {
  it('adds numbers', () => {
    expect(sum(1, 2)).to.equal(3);
  });

  it('throws on non-numbers', () => {
    expect(() => sum('a', 1)).to.throw('Invalid input');
  });
});
```

**Expected Output:** Both tests pass with Mocha + Chai.

### Real-World Cases

- **Mocha test suites:** Chai is the standard assertion library for Mocha.
- **Legacy Node.js projects:** Many older projects use Chai + Mocha + Sinon.
- **Custom plugins:** `chai-as-promised` for Promise assertions.

### References

- Chai — Official Documentation — https://www.chaijs.com/
- Chai — API Reference — https://www.chaijs.com/api/

---

## 15. Expect

### Definitions

**Core Definition:** Expect is the built-in assertion API provided by Jest and Vitest, offering a wide range of matchers for values, DOM elements (with jest-dom), and asynchronous code.

**Technical Definition:** `expect(value)` returns an object with matcher methods (`.toBe()`, `.toEqual()`, `.toHaveBeenCalled()`, `.rejects.toThrow()`, etc.). Jest and Vitest implement the same core `expect` API, with `.extend()` for custom matchers. When combined with `@testing-library/jest-dom`, `expect` gains DOM-specific matchers (`toBeInTheDocument`, `toBeDisabled`, etc.).

**Beginner-Friendly Explanation:** Expect is the function you use to check results in Jest and Vitest — like `expect(2 + 2).toBe(4)`. It comes built into those frameworks.

### Purposes

- To provide a consistent assertion API in Jest and Vitest.
- To support value, function, and DOM assertions.
- To support async assertions via `.resolves` and `.rejects`.
- To allow custom matchers via `.extend()`.

### Syntax Rules and Structure

```javascript
expect(2 + 2).toBe(4);
expect({ a: 1 }).toEqual({ a: 1 });
expect(fn).toHaveBeenCalledWith('arg');
await expect(promise).rejects.toThrow('error');
expect(2 + 2).not.toBe(5);
```

**Component Breakdown:**

| Matcher | Purpose |
|---------|---------|
| `.toBe()` | Strict equality (`Object.is`). |
| `.toEqual()` | Deep equality. |
| `.toHaveBeenCalledWith()` | Mock call arguments. |
| `.resolves` / `.rejects` | Async assertions. |
| `.not` | Negation. |

**Constraints and Limitations:**

- `.toBe()` uses `Object.is` — objects with the same shape are not equal; use `.toEqual()`.
- DOM matchers require `@testing-library/jest-dom`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Value and Async Assertions

```javascript
import { test, expect } from 'vitest';

test('value and async assertions', async () => {
  expect(2 + 2).toBe(4);
  expect({ a: 1 }).toEqual({ a: 1 });

  await expect(Promise.reject(new Error('boom'))).rejects.toThrow('boom');
});
```

**Expected Output:** The test passes. `.toBe` checks primitives, `.toEqual` checks deep equality, and `.rejects` asserts the rejection.

### Real-World Cases

- **All Jest/Vitest tests:** `expect` is the universal assertion function.
- **Custom matchers:** Extending `expect` with domain-specific assertions.

### References

- Jest — Expect — https://jestjs.io/docs/expect
- Vitest — Expect — https://vitest.dev/api/expect.html

---

## 16. Testing Library Comparison

### Definitions

**Core Definition:** Testing library comparison is the evaluation of testing tools across dimensions such as philosophy, API surface, performance, maintenance status, and ecosystem fit.

**Technical Definition:** A comparison evaluates tools on: **rendering approach** (full DOM vs. shallow), **query philosophy** (user-centric vs. implementation-centric), **interaction fidelity** (`user-event` vs. `fireEvent` vs. `simulate`), **maintenance status** (active vs. deprecated), **framework coupling** (React-only vs. framework-agnostic), and **integration with runners and assertions**.

**Beginner-Friendly Explanation:** Comparing testing libraries means looking at what each tool does well and picking the one that fits your project — considering things like whether it is still maintained, how realistic its interactions are, and whether it works with your test runner.

### Comparison Table

| Tool | Category | Rendering | Query style | Interactions | Status |
|------|----------|-----------|-------------|--------------|--------|
| **Jest** | Test runner | N/A | N/A | N/A | Active |
| **Vitest** | Test runner | N/A | N/A | N/A | Active |
| **React Testing Library** | DOM utility | Full DOM | User-centric | Via user-event | Active |
| **Enzyme** | DOM utility | Shallow/mount | Implementation-centric | `simulate()` | Deprecated |
| **@testing-library/react** | React adapter | Full DOM | User-centric | Via user-event | Active |
| **@testing-library/user-event** | Interaction | N/A | N/A | Full browser sequences | Active |
| **@testing-library/jest-dom** | Matchers | N/A | N/A | N/A | Active |
| **@testing-library/react-hooks** | Hook utility | N/A | N/A | N/A | Deprecated |
| **MSW** | Network mock | N/A | N/A | N/A | Active |
| **Mirage JS** | Backend sim | N/A | N/A | N/A | Active |
| **Nock** | Network mock | N/A | N/A | N/A | Active |
| **Sinon** | Mocks/spies | N/A | N/A | N/A | Active |
| **Chai** | Assertions | N/A | N/A | N/A | Active |
| **Expect** | Assertions | N/A | N/A | N/A | Active (built-in) |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Same Test in Different Libraries

```javascript
// Enzyme (legacy, shallow)
const wrapper = shallow(<Counter />);
expect(wrapper.state('count')).toBe(0);
wrapper.find('button').simulate('click');
expect(wrapper.state('count')).toBe(1);

// React Testing Library (modern, user-centric)
render(<Counter />);
await user.click(screen.getByRole('button', { name: /increment/i }));
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

**Why RTL is preferred:** The RTL test asserts on the user-visible count, not the internal state. It survives refactoring of the component's implementation.

### Real-World Cases

- **Migration projects:** Comparing Enzyme to RTL helps plan a migration.
- **Tool selection:** Teams compare Jest vs. Vitest, MSW vs. Nock, and Chai vs. Expect.
- **Ecosystem consistency:** Choosing tools that integrate well with the existing stack.

### References

- Testing Library — Migrate from Enzyme — https://testing-library.com/docs/react-testing-library/migrate-from-enzyme/
- MSW — Comparison with other mocking libraries — https://mswjs.io/docs/comparison/

---

## 17. Testing Library Selection

### Definitions

**Core Definition:** Testing library selection is the process of choosing the right combination of test runner, assertion library, DOM utility, interaction simulator, and mocking tool for a project's specific needs.

**Technical Definition:** Selection criteria include: **runner compatibility** (Jest vs. Vitest vs. Mocha), **framework** (React vs. Vue vs. Angular), **testing philosophy** (user-centric vs. implementation-centric), **maintenance status** (active vs. deprecated), **performance** (native ESM vs. Babel), **network mocking needs** (MSW vs. Nock vs. Mirage), and **team familiarity**.

**Beginner-Friendly Explanation:** Choosing testing libraries means picking the right tool for each job — which runner, which assertion library, which DOM utility, and which mocking tool — based on your project and team.

### Selection Guide

| Scenario | Recommended stack |
|----------|-------------------|
| **New React + Vite project** | Vitest + React Testing Library + user-event + jest-dom + MSW |
| **Existing React + Jest project** | Jest + React Testing Library + user-event + jest-dom + MSW |
| **Legacy Enzyme project** | Migrate to RTL; keep Jest until migration is complete |
| **Node.js backend** | Vitest or Jest + Expect + Nock (for HTTP) or MSW |
| **Mocha-based project** | Mocha + Chai + Sinon + Nock or MSW |
| **Ember application** | Mirage JS + QUnit or Jest |
| **Full backend simulation** | Mirage JS |
| **Network-level mocking** | MSW (browser + Node) or Nock (Node only) |
| **Non-React framework** | Vitest/Jest + Testing Library adapter + MSW |

**Constraints and Limitations:**

- Do not mix deprecated tools (Enzyme, `@testing-library/react-hooks`) with modern React.
- Avoid redundant tools (e.g., Sinon alongside Jest/Vitest's built-in mocks).
- Prefer MSW over direct `fetch` mocking for integration tests.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Recommended Stack for a New React Project

```json
// package.json (devDependencies)
{
  "vitest": "^3.0.0",
  "jsdom": "^25.0.0",
  "@testing-library/react": "^16.0.0",
  "@testing-library/user-event": "^14.0.0",
  "@testing-library/jest-dom": "^6.0.0",
  "msw": "^2.0.0"
}
```

**Why this stack:** Vitest is fast and Vite-native; RTL provides user-centric queries; user-event provides realistic interactions; jest-dom adds semantic matchers; MSW provides network-level API mocking. This is the standard modern React testing stack in 2026.

### Real-World Cases

- **Greenfield projects:** The Vitest + RTL + MSW stack is the default for new React projects.
- **Legacy migrations:** Jest + RTL + MSW for existing CRA projects; incremental Enzyme migration.
- **Backend services:** Vitest + Expect + MSW or Nock.

### References

- Testing Library — Official Documentation — https://testing-library.com/docs/
- Vitest — Official Documentation — https://vitest.dev/guide/
- MSW — Official Documentation — https://mswjs.io/docs/

---

## References

- Jest — Official Documentation — https://jestjs.io/docs/getting-started
- Jest — API Reference — https://jestjs.io/docs/api
- Vitest — Official Documentation — https://vitest.dev/guide/
- Vitest — Jest Compatibility — https://vitest.dev/guide/migration.html
- React Testing Library — Introduction — https://testing-library.com/docs/react-testing-library/intro
- React Testing Library — render API — https://testing-library.com/docs/react-testing-library/api#render
- React Testing Library — Setup — https://testing-library.com/docs/react-testing-library/setup
- Testing Library — Official Documentation — https://testing-library.com/docs/
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- Testing Library — user-event Introduction — https://testing-library.com/docs/user-event/intro
- Testing Library — fireEvent vs user-event — https://testing-library.com/docs/user-event/intro#differences-from-fireevent
- jest-dom — GitHub Repository — https://github.com/testing-library/jest-dom
- React Hooks Testing Library — Deprecation Notice — https://github.com/testing-library/react-hooks-testing-library
- Enzyme — GitHub Repository — https://github.com/enzymejs/enzyme
- Testing Library — Migrate from Enzyme — https://testing-library.com/docs/react-testing-library/migrate-from-enzyme/
- MSW — Official Documentation — https://mswjs.io/docs/
- Using Mock Service Worker to Improve Jest Unit Tests — https://www.wwt.com/article/using-mock-service-worker-to-improve-jest-unit-tests
- Mirage JS — Official Documentation — https://miragejs.com/docs/getting-started/introduction/
- Nock — GitHub Repository — https://github.com/nock/nock
- Sinon — Official Documentation — https://sinonjs.org/
- Chai — Official Documentation — https://www.chaijs.com/
- Jest — Expect — https://jestjs.io/docs/expect
- Vitest — Expect — https://vitest.dev/api/expect.html
- Bunnyshell — E2E Testing Frameworks & Tools Compared for 2026 — https://www.bunnyshell.com/blog/e2e-testing-frameworks-tools/