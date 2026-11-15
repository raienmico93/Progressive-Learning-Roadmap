# Unit Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Unit testing is the practice of verifying the behavior of the smallest testable units of software — individual functions, components, hooks, or modules — in isolation from their dependencies.

**Technical Definition:** A unit test exercises a single unit of software (a function, method, class, or component) with controlled inputs, replaces all external dependencies with test doubles, and asserts that the observed output or behavior matches the expected outcome. Unit tests are characterized by speed (milliseconds), isolation (no shared state, no external systems), determinism (same result every run), and self-validation (automated pass/fail). They form the broad base of the test pyramid and provide fast feedback during development and continuous integration.

**Beginner-Friendly Explanation:** Unit testing means checking one small piece of your code — like a single function or component — all by itself. You give it some input, see what comes out, and check if it matches what you expected. Because these tests are tiny and don't touch databases or networks, they run in milliseconds and tell you immediately if something broke.

### Key Characteristics

- **Granular:** Tests one logical unit per test case.
- **Isolated:** Replaces external dependencies (APIs, databases, file systems) with test doubles.
- **Fast:** Executes in milliseconds, enabling frequent runs and watch mode.
- **Deterministic:** Same input always produces the same result — no randomness, no real timers, no network flakiness.
- **Self-validating:** Automated assertions produce a clear pass/fail without manual inspection.
- **Behavior-focused:** Asserts on what the unit does, not how it does it internally.

### Prerequisites

- A testing framework (Vitest, Jest, Mocha, pytest, JUnit, etc.).
- An assertion library (built into most modern frameworks).
- For React: `@testing-library/react` (with `renderHook`, `act`, and `user-event`).
- For async code: `waitFor`, `findBy*` queries, and fake timers.
- A CI/CD pipeline for running tests automatically (recommended).

### Related Programming Areas

- **Integration testing** — verifies that units work together (the layer above unit tests).
- **Test-Driven Development (TDD)** — writing tests before code to drive design.
- **Test doubles** — mocks, stubs, spies, and fakes that isolate the unit under test.
- **Refactoring** — unit tests serve as a safety net for restructuring code.
- **Code coverage** — measuring which parts of the unit are exercised by tests.

### Core Concepts / Features

---

## 1. Unit Testing Components

### Definitions

**Core Definition:** Unit testing a component means rendering it in isolation and asserting that it produces the correct output and responds correctly to user interactions.

**Technical Definition:** A component unit test mounts the component using a testing library (e.g., `render` from `@testing-library/react`), queries the rendered DOM using user-visible attributes (roles, labels, text), simulates user interactions with `user-event`, and asserts on the resulting UI state. The test should verify **behavior** — what the user sees and does — rather than internal implementation details like state variable names or private methods. External dependencies (API calls, context providers) are mocked or wrapped appropriately.

**Beginner-Friendly Explanation:** A component unit test checks that a single React component renders the right HTML and responds correctly when a user clicks or types. It doesn't test the whole app — just one component, with its dependencies replaced by fakes.

### Purposes

- To verify that a component renders the correct output for given props.
- To confirm that user interactions (clicks, typing, selections) trigger the expected behavior.
- To catch regressions when component logic changes.
- To document the component's public contract (props and visible behavior).

### Syntax Rules and Structure

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { expect, test } from 'vitest';

test('component behavior description', async () => {
  const user = userEvent.setup();

  // Arrange: render with props
  render(<Component prop="value" />);

  // Act: interact as a user
  await user.click(screen.getByRole('button', { name: /submit/i }));

  // Assert: verify observable output
  expect(screen.getByText('Success')).toBeInTheDocument();
});
```

**Component Breakdown:**

| Step | Description |
|------|-------------|
| `render(<Component />)` | Mounts the component into a jsdom (or browser) environment. |
| `screen.getByRole(...)` | Queries the DOM by accessibility role — the most user-centric query. |
| `userEvent.setup()` | Creates a user interaction session with realistic event dispatching. |
| `await user.click(...)` | Simulates a real user click (focus, mousedown, mouseup, click). |
| `expect(...).toBeInTheDocument()` | Asserts on the observable DOM state. |

**Constraints and Limitations:**

- Avoid querying by CSS class or `data-testid` when a user-visible query (role, label, text) exists. 
- Do not assert on internal state (`component.state`, hook values) — assert on what the user sees.
- Use `findBy*` for elements that appear asynchronously, `queryBy*` to assert absence. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Button Component

```jsx
// Button.jsx
export function Button({ label, onClick, disabled = false }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// Button.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi, test, expect } from 'vitest';
import { Button } from './Button.jsx';

test('calls onClick when clicked', async () => {
  const user = userEvent.setup();
  const handleClick = vi.fn();

  render(<Button label="Submit" onClick={handleClick} />);

  await user.click(screen.getByRole('button', { name: /submit/i }));

  expect(handleClick).toHaveBeenCalledTimes(1);
});

test('does not call onClick when disabled', async () => {
  const user = userEvent.setup();
  const handleClick = vi.fn();

  render(<Button label="Submit" onClick={handleClick} disabled />);

  await user.click(screen.getByRole('button', { name: /submit/i }));

  expect(handleClick).not.toHaveBeenCalled();
});
```

**Expected Output:** Both tests pass. The first confirms the click handler fires; the second confirms it does not fire when the button is disabled.

**Why this output:** `user.click` dispatches a full user interaction sequence. The `disabled` attribute natively prevents click events in browsers, and jsdom respects that behavior.

#### Example 2: Testing a Form with Asynchronous Submission

```jsx
// LoginForm.jsx
export function LoginForm({ onSubmit }) {
  const [status, setStatus] = useState('idle');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    await onSubmit(new FormData(e.target));
    setStatus('success');
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" type="email" />
      <button type="submit" disabled={status === 'submitting'}>
        {status === 'submitting' ? 'Submitting...' : 'Log In'}
      </button>
      {status === 'success' && <p>Welcome!</p>}
    </form>
  );
}

// LoginForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi, test, expect } from 'vitest';
import { LoginForm } from './LoginForm.jsx';

test('submits the form and shows success', async () => {
  const user = userEvent.setup();
  const handleSubmit = vi.fn().mockResolvedValue(undefined);

  render(<LoginForm onSubmit={handleSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(await screen.findByText('Welcome!')).toBeInTheDocument();
  expect(handleSubmit).toHaveBeenCalledTimes(1);
});
```

**Expected Output:** The test passes. It types into the email field, clicks submit, waits for the success message, and verifies the submit handler was called.

### Real-World Cases

- **Design systems:** Every component in a design system has unit tests verifying its rendering, prop handling, and interaction behavior.
- **Form components:** Testing validation, submission, and error states without a backend.
- **Conditional UI:** Verifying that loading spinners, error messages, and empty states render correctly.

### References

- Scrimba — How to Test React Apps [2026]: A Practical Guide — https://scrimba.com/articles/how-to-test-react-apps-2026/
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority

---

## 2. Unit Testing Hooks

### Definitions

**Core Definition:** Unit testing a hook means exercising a custom React Hook in isolation — without rendering a full component — and asserting on its returned values and side effects.

**Technical Definition:** The `renderHook` utility from `@testing-library/react` renders a hook inside a minimal test component, providing a stable environment for React's Hooks system. The test reads the hook's return value via `result.current`, wraps all state updates in `act()` to flush React's update queue, uses `rerender()` to test prop changes, and `unmount()` to verify cleanup logic. The original `react-hooks-testing-library` is deprecated in favor of the built-in `renderHook`. 

**Beginner-Friendly Explanation:** A hook test checks that a custom hook — like `useCounter` or `useLocalStorage` — returns the right values and updates correctly when you call its functions. Instead of building a whole component, you use `renderHook` to test the hook by itself.

### Purposes

- To verify the logic of reusable custom Hooks independently of any component.
- To test state updates, side effects, cleanup, and error conditions in isolation.
- To ensure that Hooks behave correctly when props change or when the component unmounts.
- To document the contract of a custom Hook for consumers.

### Syntax Rules and Structure

```jsx
import { renderHook, act } from '@testing-library/react';

const { result } = renderHook(() => useCustomHook(initialArg));

// Read state
result.current.value;

// Update state (must wrap in act)
act(() => {
  result.current.updateFunction(newValue);
});

// Re-render with new props
const { rerender } = renderHook(({ arg }) => useCustomHook(arg), {
  initialProps: { arg: 1 },
});
rerender({ arg: 2 });

// Test cleanup
const { unmount } = renderHook(() => useCustomHook());
unmount();
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `renderHook(fn)` | Renders the hook inside a test harness component. |
| `result.current` | The hook's latest return value. |
| `act(cb)` | Flushes all state updates and effects before assertions. |
| `rerender(newProps)` | Re-renders the hook with new arguments. |
| `unmount()` | Unmounts the harness to test cleanup logic. |

**Constraints and Limitations:**

- Every state mutation must be inside `act()`, or the test reads stale values and emits warnings. 
- `renderHook` is best for complex, reusable Hooks; for simple Hooks, testing through a real component may be more natural. 
- Timer-based Hooks require `vi.useFakeTimers()` / `jest.useFakeTimers()` and timer advancement.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Counter Hook

```jsx
// useCounter.js
import { useState, useCallback } from 'react';

export function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = useCallback(() => setCount((c) => c + 1), []);
  const decrement = useCallback(() => setCount((c) => c - 1), []);
  const reset = useCallback(() => setCount(initial), [initial]);
  return { count, increment, decrement, reset };
}

// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter.js';

describe('useCounter', () => {
  it('starts with the initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  it('increments', () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => result.current.increment());
    expect(result.current.count).toBe(1);
  });

  it('decrements', () => {
    const { result } = renderHook(() => useCounter(5));
    act(() => result.current.decrement());
    expect(result.current.count).toBe(4);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    act(() => result.current.increment());
    act(() => result.current.reset());
    expect(result.current.count).toBe(10);
  });
});
```

**Expected Output:** All four tests pass. The hook's initial value, increment, decrement, and reset behaviors are verified in isolation.

**Why this output:** `renderHook` provides a stable environment for the hook. Each `act()` flushes the state update, and `result.current` reads the fresh value.

#### Example 2: Testing Timer-Based Hook Cleanup

```jsx
// useInterval.js
import { useEffect, useRef } from 'react';

export function useInterval(callback, delay) {
  const savedCallback = useRef(callback);
  useEffect(() => { savedCallback.current = callback; }, [callback]);

  useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

// useInterval.test.js
import { renderHook } from '@testing-library/react';
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';
import { useInterval } from './useInterval.js';

describe('useInterval', () => {
  beforeEach(() => vi.useFakeTimers());
  afterEach(() => vi.useRealTimers());

  it('fires the callback on each interval', () => {
    const callback = vi.fn();
    renderHook(() => useInterval(callback, 1000));

    vi.advanceTimersByTime(3000);

    expect(callback).toHaveBeenCalledTimes(3);
  });

  it('clears the interval on unmount', () => {
    const clearSpy = vi.spyOn(global, 'clearInterval');
    const { unmount } = renderHook(() => useInterval(vi.fn(), 1000));

    unmount();

    expect(clearSpy).toHaveBeenCalledTimes(1);
  });
});
```

**Expected Output:** The first test confirms the callback fires three times after 3 seconds of fake time. The second confirms the interval is cleared on unmount.

### Real-World Cases

- **Data-fetching Hooks:** Testing `useFetch` with mocked network responses and `waitFor`.
- **Storage Hooks:** Testing `useLocalStorage` reads and writes against a mocked `localStorage`.
- **Event Hooks:** Testing `useKeyPress`, `useWindowSize`, and `useMediaQuery` with simulated browser events.

### References

- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- ReactUse — hook-test — https://github.com/childrentime/reactuse/blob/7081b1810215818882c739be8685cdacdb6ffb16/.claude/skills/hook-test/SKILL.md
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/

---

## 3. Unit Testing Utilities

### Definitions

**Core Definition:** Unit testing utility functions means verifying that small, reusable helper functions — formatters, validators, parsers, calculators — produce correct outputs for a range of inputs, including edge cases.

**Technical Definition:** Utility functions are typically pure (or nearly pure) functions that transform inputs to outputs. Their tests supply representative inputs (normal, boundary, invalid) and assert on the returned values. Because utilities are decoupled from UI and business state, their tests are trivial to write, fast to run, and highly maintainable. Complex utilities should target ≥80% coverage.

**Beginner-Friendly Explanation:** Utility tests check that helper functions like `formatPrice` or `normalizeUrl` give the right answer. You call the function with some inputs and assert that the result matches what you expect.

### Purposes

- To verify the correctness of reusable helper functions across their input domain.
- To catch edge-case bugs (empty input, null, negative numbers, Unicode).
- To document the expected behavior of utility APIs.
- To provide fast, low-maintenance regression protection.

### Syntax Rules and Structure

```js
import { describe, it, expect } from 'vitest';
import { utilityFn } from './utils.js';

describe('utilityFn', () => {
  it('handles typical input', () => {
    expect(utilityFn('input')).toBe('expected');
  });

  it('handles edge cases', () => {
    expect(utilityFn('')).toBe('');
    expect(utilityFn(null)).toBeNull();
  });
});
```

**Constraints and Limitations:**

- Cover normal, boundary, and invalid inputs — not just the happy path.
- Avoid testing implementation details (internal variables) — assert on return values.
- Utilities that depend on browser APIs should wrap those APIs in a function for easy mocking. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a URL Normalizer

```js
// urlHelpers.js
export function normalizeUrl(url, base) {
  if (!url) return '';
  if (url.startsWith('http://') || url.startsWith('https://')) return url;
  if (url.startsWith('/')) return `${base}${url}`;
  return `${base}/${url}`;
}

// urlHelpers.test.js
import { describe, it, expect } from 'vitest';
import { normalizeUrl } from './urlHelpers.js';

describe('normalizeUrl', () => {
  const base = 'https://example.com';

  it('returns empty string for null/undefined', () => {
    expect(normalizeUrl(null, base)).toBe('');
    expect(normalizeUrl(undefined, base)).toBe('');
  });

  it('returns absolute URLs unchanged', () => {
    expect(normalizeUrl('https://other.com/path', base)).toBe('https://other.com/path');
  });

  it('prefixes relative paths with the base', () => {
    expect(normalizeUrl('/about', base)).toBe('https://example.com/about');
    expect(normalizeUrl('about', base)).toBe('https://example.com/about');
  });
});
```

**Expected Output:** All tests pass. The utility handles null, absolute URLs, and relative paths correctly.

#### Example 2: Testing a Price Formatter

```js
// formatPrice.js
export function formatPrice(cents, currency = 'USD') {
  const formatter = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  });
  return formatter.format(cents / 100);
}

// formatPrice.test.js
import { describe, it, expect } from 'vitest';
import { formatPrice } from './formatPrice.js';

describe('formatPrice', () => {
  it('formats cents to USD by default', () => {
    expect(formatPrice(1999)).toBe('$19.99');
  });

  it('handles zero', () => {
    expect(formatPrice(0)).toBe('$0.00');
  });

  it('supports different currencies', () => {
    expect(formatPrice(1999, 'EUR')).toContain('€');
  });

  it('handles large numbers', () => {
    expect(formatPrice(100000000)).toBe('$1,000,000.00');
  });
});
```

**Expected Output:** All tests pass, covering typical, zero, multi-currency, and large-number cases.

### Real-World Cases

- **Date formatting:** `formatDate(date, format)` across time zones and locales.
- **Validation:** `validateEmail`, `validatePassword` with valid and invalid inputs.
- **Parsing:** `parseQueryString`, `parseCurrency` with malformed inputs.

### References

- Adobe — Unit Testing Guide — https://github.com/adobe/skills/blob/4130a3c138c88ae29802fbc153d3e7beea2a0637/plugins/aem/edge-delivery-services/skills/testing-blocks/references/unit-testing.md
- @sop-cli/utils — Utility function testing principles — https://www.npmjs.com/package/@sop-cli/utils

---

## 4. Unit Testing Reducers

### Definitions

**Core Definition:** Unit testing a reducer means calling the reducer function with a given state and action, and asserting that it returns the correctly updated state.

**Technical Definition:** A reducer is a pure function `(state, action) => newState`. Testing a reducer involves supplying a known state and action, calling the reducer, and asserting on the returned state. Because reducers are pure, no mocking is required — the test is simply input-to-output verification. Every action case declared by a slice should have a corresponding test. The Redux team recommends that reducers be tested only if they contain complex logic; otherwise, integration tests cover them adequately.

**Beginner-Friendly Explanation:** A reducer test checks that a Redux reducer updates the state correctly when given an action. It's simple: give the reducer a state and an action, and check that the new state is what you expect.

### Purposes

- To verify that each action case in a reducer produces the correct state transition.
- To catch logic errors in complex state updates (nested data, arrays, normalization).
- To assert immutability — the reducer must return a new state object, not mutate the old one.
- To document the reducer's behavior for each action type.

### Syntax Rules and Structure

```js
import reducer, { actionCreator } from './slice.js';

const initialState = { /* ... */ };
const action = actionCreator(payload);
const newState = reducer(initialState, action);

expect(newState).toEqual({ /* expected */ });
expect(newState).not.toBe(initialState); // Immutability check
```

**Constraints and Limitations:**

- Do not mock reducers — they are pure functions and should be tested directly.
- Redux recommends integration tests over isolated reducer tests for simple reducers; unit-test complex reducers. 
- Always verify immutability for reducers that handle nested or array state.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Slice Reducer

```js
// counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
    reset: () => ({ value: 0 }),
  },
});

export const { increment, decrement, reset } = counterSlice.actions;
export default counterSlice.reducer;

// counterSlice.test.js
import { describe, it, expect } from 'vitest';
import reducer, { increment, decrement, reset } from './counterSlice.js';

describe('counter reducer', () => {
  it('returns the initial state', () => {
    expect(reducer(undefined, { type: 'unknown' })).toEqual({ value: 0 });
  });

  it('handles increment', () => {
    const previous = { value: 5 };
    const next = reducer(previous, increment());
    expect(next.value).toBe(6);
    expect(next).not.toBe(previous); // Immutability
  });

  it('handles decrement', () => {
    expect(reducer({ value: 5 }, decrement()).value).toBe(4);
  });

  it('handles reset', () => {
    expect(reducer({ value: 99 }, reset()).value).toBe(0);
  });
});
```

**Expected Output:** All tests pass. Each action produces the correct state, and the reducer returns a new object rather than mutating the input.

### Real-World Cases

- **Shopping cart:** Testing add, remove, and quantity-update reducer cases.
- **Authentication:** Testing login, logout, and token-refresh state transitions.
- **Form wizards:** Testing next-step, previous-step, and reset reducer logic.

### References

- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- Redux Toolkit — Testing Slice Reducers — https://github.com/agents-inc/skills/blob/main/skills/src/skills/web-state-redux-toolkit/examples/testing.md

---

## 5. Unit Testing Selectors

### Definitions

**Core Definition:** Unit testing a selector means supplying a known state object and asserting that the selector returns the correct derived value.

**Technical Definition:** A selector is a function `(state) => derivedValue`. Testing a selector involves constructing a representative state, calling the selector with that state, and asserting on the result. Memoized selectors (created with `createSelector` from Reselect or RTK) expose `.recomputations()` and `.resetRecomputations()` methods for verifying memoization behavior. Like reducers, selectors are pure and should be tested directly — never mocked. 

**Beginner-Friendly Explanation:** A selector test checks that a function correctly extracts or derives data from the Redux state. You give the selector a fake state and check that it returns the right value.

### Purposes

- To verify that derived data (filtered lists, computed totals, formatted values) is correct.
- To test memoization behavior for selectors created with `createSelector`.
- To catch bugs in complex state-derivation logic.
- To document the shape and meaning of derived state.

### Syntax Rules and Structure

```js
import { selector } from './selectors.js';

const state = { /* representative state */ };
expect(selector(state)).toBe(expectedValue);
```

**For memoized selectors:**

```js
const selector = createSelector(inputSelector, (input) => derived);
selector(state); // First call — computes
selector(state); // Second call — memoized
expect(selector.recomputations()).toBe(1);
```

**Constraints and Limitations:**

- Do not mock selector functions — test them with real state objects. 
- Memoized selectors should be tested for both correctness and memoization (recomputation count).
- Selectors that depend on multiple reducers require a composite state fixture.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Simple Selector

```js
// selectors.js
export const selectTodos = (state) => state.todos.items;
export const selectCompletedTodos = (state) =>
  state.todos.items.filter((t) => t.completed);
export const selectTodoCount = (state) => state.todos.items.length;

// selectors.test.js
import { describe, it, expect } from 'vitest';
import { selectCompletedTodos, selectTodoCount } from './selectors.js';

const mockState = {
  todos: {
    items: [
      { id: 1, text: 'Learn testing', completed: true },
      { id: 2, text: 'Write tests', completed: false },
      { id: 3, text: 'Refactor', completed: true },
    ],
  },
};

describe('todo selectors', () => {
  it('returns only completed todos', () => {
    const result = selectCompletedTodos(mockState);
    expect(result).toHaveLength(2);
    expect(result.map((t) => t.id)).toEqual([1, 3]);
  });

  it('returns the total count', () => {
    expect(selectTodoCount(mockState)).toBe(3);
  });
});
```

**Expected Output:** Both tests pass. The selectors correctly filter and count from the mock state.

#### Example 2: Testing a Memoized Selector

```js
// selectors.js
import { createSelector } from '@reduxjs/toolkit';

const selectItems = (state) => state.cart.items;
const selectTaxRate = (state) => state.cart.taxRate;

export const selectTotalWithTax = createSelector(
  [selectItems, selectTaxRate],
  (items, taxRate) => {
    const subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0);
    return subtotal * (1 + taxRate);
  }
);

// selectors.test.js
import { describe, it, expect } from 'vitest';
import { selectTotalWithTax } from './selectors.js';

const state = {
  cart: {
    items: [
      { price: 10, qty: 2 },
      { price: 5, qty: 1 },
    ],
    taxRate: 0.1,
  },
};

describe('selectTotalWithTax', () => {
  it('computes the total with tax', () => {
    expect(selectTotalWithTax(state)).toBeCloseTo(27.5);
  });

  it('memoizes the result', () => {
    selectTotalWithTax(state);
    selectTotalWithTax(state);
    expect(selectTotalWithTax.recomputations()).toBe(1);
  });
});
```

**Expected Output:** The first test verifies the total is `(10×2 + 5×1) × 1.1 = 27.5`. The second verifies that calling the selector twice with the same state recomputes only once.

### Real-World Cases

- **E-commerce carts:** Computing subtotals, tax, and shipping costs from cart state.
- **Filtered lists:** Deriving visible items from filters and search terms.
- **Dashboard metrics:** Aggregating counts and averages from normalized state.

### References

- Redux — Writing Tests: Selectors — https://redux.js.org/usage/writing-tests
- Egghead — Unit Testing a Memoized Selector — https://egghead.io/lessons/react-unit-testing-a-memoized-selector-that-derives-data-from-multiple-reducers

---

## 6. Testing Pure Functions

### Definitions

**Core Definition:** A pure function is a function that always returns the same output for the same input and produces no side effects; testing it requires only supplying inputs and asserting on outputs.

**Technical Definition:** Purity means referential transparency: the function's return value depends solely on its arguments, and calling it has no observable effect beyond returning a value (no mutation, no I/O, no randomness, no reliance on external mutable state). Pure functions are trivially testable because the test setup is just "call the function with inputs" — no mocks, no fixtures, no cleanup. Writing logic in pure functions whenever possible is the single biggest factor in making code easy to test. 

**Beginner-Friendly Explanation:** A pure function is predictable: give it the same input, and it always gives the same output, without changing anything else. Because of that, testing it is simple — just call it and check the result.

### Purposes

- To verify deterministic input-to-output mapping across representative and edge-case inputs.
- To catch logic errors in calculations, transformations, and validations.
- To provide the fastest, most maintainable form of unit test.
- To encourage designing code with pure functions at its core.

### Syntax Rules and Structure

```js
import { describe, it, expect } from 'vitest';
import { pureFn } from './logic.js';

describe('pureFn', () => {
  it('maps input to expected output', () => {
    expect(pureFn(input)).toBe(expectedOutput);
  });
});
```

**Constraints and Limitations:**

- Pure functions cannot perform I/O, mutate arguments, or depend on `Date.now()`, `Math.random()`, or global state.
- If a function must touch the outside world, isolate the pure logic into a separate function and test that. 
- Impure dependencies (timers, browser APIs) should be wrapped in functions that can be mocked.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Pure Discount Calculator

```js
// discount.js
export function applyDiscount(price, isMember) {
  if (price < 0) throw new Error('Price cannot be negative');
  return isMember ? price * 0.9 : price;
}

// discount.test.js
import { describe, it, expect } from 'vitest';
import { applyDiscount } from './discount.js';

describe('applyDiscount', () => {
  it('applies 10% discount for members', () => {
    expect(applyDiscount(100, true)).toBe(90);
  });

  it('returns full price for non-members', () => {
    expect(applyDiscount(100, false)).toBe(100);
  });

  it('throws on negative price', () => {
    expect(() => applyDiscount(-1, true)).toThrow('Price cannot be negative');
  });
});
```

**Expected Output:** All three tests pass. The pure function maps every input case to the correct output or error.

#### Example 2: Extracting Pure Logic from a Component

```jsx
// shipping.js — pure logic extracted from a component
export function calculateShippingTier(weightKg) {
  if (weightKg < 1) return { tier: 'light', baseCost: 4.99 };
  if (weightKg < 10) return { tier: 'standard', baseCost: 7.99 };
  return { tier: 'heavy', baseCost: 12.99 };
}

// shipping.test.js
import { describe, it, expect } from 'vitest';
import { calculateShippingTier } from './shipping.js';

describe('calculateShippingTier', () => {
  it('returns light tier for under 1kg', () => {
    expect(calculateShippingTier(0.5)).toEqual({ tier: 'light', baseCost: 4.99 });
  });

  it('returns standard tier for 1–10kg', () => {
    expect(calculateShippingTier(5)).toEqual({ tier: 'standard', baseCost: 7.99 });
  });

  it('returns heavy tier for 10kg and above', () => {
    expect(calculateShippingTier(10)).toEqual({ tier: 'heavy', baseCost: 12.99 });
  });
});
```

**Expected Output:** All tests pass. The pure function has three clear branches, each verified with a boundary-appropriate input.

### Real-World Cases

- **Financial calculations:** Tax, discounts, interest, currency conversion.
- **Validation logic:** Email, password, and form-field validation rules.
- **Data transformation:** Sorting, filtering, normalizing, and mapping data.
- **Game logic:** Score calculation, collision detection, turn resolution.

### References

- Galaxy Project — Implement logic in pure functions when possible — https://github.com/galaxyproject/galaxy/blob/a73355ea2e5c47c62c5c79547b0b5e330e0695f5/client/docs/unit-testing/strategies.md
- Steve Kinney — Implementing Some Basic Tests — https://stevekinney.com/courses/testing/implementing-some-basic-tests

---

## 7. Testing Side Effects

### Definitions

**Core Definition:** Testing side effects means verifying that a function or component correctly interacts with external systems — APIs, timers, DOM APIs, logging — without actually performing the real side effect.

**Technical Definition:** Side effects are operations that affect the world outside the function: network requests, `localStorage` writes, timer scheduling, DOM manipulation, and event emission. To test them in isolation, the external dependency is replaced with a test double (mock, stub, spy, or fake). The test asserts that the side effect was **requested correctly** — with the right arguments, at the right time, the right number of times — rather than that it actually occurred. Dependency injection (passing the dependency as a parameter) makes side effects easy to substitute.

**Beginner-Friendly Explanation:** A side effect is anything your code does besides returning a value — like calling an API or setting a timer. To test it, you replace the real API with a fake, then check that your code called the fake correctly.

### Purposes

- To verify that external interactions occur with the correct arguments and timing.
- To test error handling for failed external operations.
- To keep tests fast and deterministic by avoiding real I/O.
- To isolate the unit under test from infrastructure concerns.

### Syntax Rules and Structure

**Mock a dependency:**

```js
const mockApi = { save: vi.fn() };
const service = new Service(mockApi);

service.createUser({ name: 'Alice' });

expect(mockApi.save).toHaveBeenCalledWith({ name: 'Alice' });
```

**Mock a module:**

```js
vi.mock('./emailService.js');
import { sendEmail } from './emailService.js';

sendEmail.mockResolvedValue(undefined);
```

**Constraints and Limitations:**

- Mock only direct dependencies — avoid mocking everything. 
- Prefer dependency injection (constructor parameters) over module mocking for testability.
- Wrap native browser APIs (e.g., `window.location`) in functions so they can be easily mocked. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing an Email Side Effect with a Mock

```js
// authService.js
export class AuthService {
  constructor(emailService) {
    this.emailService = emailService;
  }

  async register(email) {
    await this.emailService.send(email, 'Welcome!');
    return { success: true };
  }
}

// authService.test.js
import { describe, it, expect, vi } from 'vitest';
import { AuthService } from './authService.js';

describe('AuthService.register', () => {
  it('sends a welcome email after registration', async () => {
    const emailService = { send: vi.fn().mockResolvedValue(undefined) };
    const auth = new AuthService(emailService);

    await auth.register('alice@example.com');

    expect(emailService.send).toHaveBeenCalledTimes(1);
    expect(emailService.send).toHaveBeenCalledWith('alice@example.com', 'Welcome!');
  });
});
```

**Expected Output:** The test passes. The mock records the call, and the assertions verify the side effect was requested with the correct arguments.

#### Example 2: Testing a Timer Side Effect with Fake Timers

```js
// debounce.js
export function createDebouncedFn(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// debounce.test.js
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { createDebouncedFn } from './debounce.js';

describe('createDebouncedFn', () => {
  beforeEach(() => vi.useFakeTimers());
  afterEach(() => vi.useRealTimers());

  it('calls the function once after the delay', () => {
    const fn = vi.fn();
    const debounced = createDebouncedFn(fn, 300);

    debounced('a');
    debounced('b');
    debounced('c');

    expect(fn).not.toHaveBeenCalled();
    vi.advanceTimersByTime(300);
    expect(fn).toHaveBeenCalledTimes(1);
    expect(fn).toHaveBeenCalledWith('c');
  });
});
```

**Expected Output:** The test passes. Despite three rapid calls, the debounced function fires only once — with the last arguments — after the fake timer advances by 300ms.

### Real-World Cases

- **Analytics:** Verifying that page-view and click events are sent with the correct payload.
- **Error logging:** Asserting that errors are reported to a logging service.
- **Local storage:** Testing that preferences are written to `localStorage` with the correct key and value.
- **Navigation:** Testing that a redirect function is called after login.

### References

- Galaxy Project — Wrap native browser resources in a function so they can be easily mocked — https://github.com/galaxyproject/galaxy/blob/a73355ea2e5c47c62c5c79547b0b5e330e0695f5/client/docs/unit-testing/strategies.md
- NestJS — Use Testing Module for Unit Tests (mocked dependencies) — https://github.com/nestjs/docs.nestjs.com/blob/master/content/fundamentals/testing.md

---

## 8. Testing Async Code

### Definitions

**Core Definition:** Testing async code means verifying that functions returning Promises, using `async`/`await`, or relying on timers resolve, reject, and update state correctly.

**Technical Definition:** Async tests must `await` the promise chain so that assertions execute after the asynchronous operation settles. Frameworks provide `async`/`await` in test functions, `rejects.toThrow` (or equivalent) for rejection assertions, `waitFor` and `findBy*` for polling UI updates, and fake timers for time-dependent logic. The most common failure mode is a **forgotten `await`**, which produces a false-green test where assertions run before the async operation completes. Synchronous blocking (`.Result`, `GetAwaiter().GetResult()`) is an anti-pattern that can deadlock. 

**Beginner-Friendly Explanation:** Async testing means waiting for your code to finish before checking the result. If you forget to `await`, your test might pass without actually testing anything — or fail with a mysterious timeout.

### Purposes

- To verify that asynchronous operations produce the correct result after they settle.
- To test error and rejection paths explicitly.
- To ensure the UI reflects loading, success, and error states at the right times.
- To avoid false-green tests caused by unawaited promises.

### Syntax Rules and Structure

**Async/await pattern:**

```js
test('fetches data', async () => {
  const data = await fetchData();
  expect(data).toEqual({ id: 1 });
});
```

**Rejection pattern:**

```js
await expect(fetchData('bad')).rejects.toThrow('Not found');
```

**UI polling pattern:**

```js
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});
```

**Constraints and Limitations:**

- Always `await` async test bodies and async assertions — unawaited promises produce false greens. 
- Use fake timers (`vi.useFakeTimers()`) instead of real delays for time-dependent code. 
- Handle `AbortError` separately when testing request cancellation.
- Do not use synchronous blocking on promises — it can deadlock. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing an Async Data Fetcher

```js
// userService.js
export async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// userService.test.js
import { describe, it, expect, vi } from 'vitest';
import { fetchUser } from './userService.js';

describe('fetchUser', () => {
  it('returns user data on success', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ id: 1, name: 'Alice' }),
    });

    const user = await fetchUser(1);

    expect(user).toEqual({ id: 1, name: 'Alice' });
  });

  it('throws on HTTP error', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 404,
    });

    await expect(fetchUser(999)).rejects.toThrow('HTTP 404');
  });
});
```

**Expected Output:** Both tests pass. The success path returns the parsed user; the error path throws the expected error.

**Why this output:** `await` ensures the assertions run after the promise resolves. The rejection test uses `rejects.toThrow` to assert the error without letting it propagate as an unhandled rejection.

#### Example 2: Testing Async UI with `findBy`

```jsx
// UserProfile.jsx
export function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`/api/users/${userId}`).then((r) => r.json()).then(setUser);
  }, [userId]);

  if (!user) return <p>Loading...</p>;
  return <h1>{user.name}</h1>;
}

// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import { vi, test, expect } from 'vitest';
import { UserProfile } from './UserProfile.jsx';

test('displays the user name after loading', async () => {
  global.fetch = vi.fn().mockResolvedValue({
    json: () => Promise.resolve({ name: 'Alice' }),
  });

  render(<UserProfile userId={1} />);

  expect(screen.getByText('Loading...')).toBeInTheDocument();
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The loading state is visible initially, and `findByText` waits for the async update to render the user name.

### Real-World Cases

- **API integration:** Testing service functions that call HTTP endpoints.
- **Debounced search:** Testing that rapid input produces a single search request.
- **Polling hooks:** Testing `useInterval`-based data refresh with fake timers.
- **Form submission:** Testing async submit handlers that show loading and success states.

### References

- ai-skills — test-async-functions — https://github.com/turkeryildirim/ai-skills/blob/main/ai-skills/javascript/rules/test-async-functions.md
- Vitest/Jest Patterns for Accurate Tests — https://github.com/your-org/your-repo/blob/main/docs/testing/vitest-jest-patterns.md
- Steve Kinney — Using Async/Await — https://stevekinney.com/courses/testing/using-async-await

---

## 9. Testing Best Practices

### Definitions

**Core Definition:** Testing best practices are the principles and habits that make unit tests fast, reliable, maintainable, and effective at catching regressions.

**Technical Definition:** Best practices encompass the FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely), behavior-focused assertions (not implementation details), accessible queries, proper async handling, minimal and intentional mocking, and the test pyramid distribution. They aim to maximize the value of the test suite while minimizing flakiness, maintenance burden, and false confidence.

**Beginner-Friendly Explanation:** Best practices are the rules of thumb that keep your tests useful — fast enough to run constantly, clear enough to tell you exactly what broke, and stable enough to trust.

### Purposes

- To keep tests fast enough to run on every save and every commit.
- To ensure tests fail for the right reason and pass for the right reason.
- To minimize brittleness so tests survive refactoring.
- To balance test coverage across the pyramid (many unit tests, fewer integration, few E2E).

### Syntax Rules and Structure

**FIRST Principles:**

| Principle | Meaning | Violation signs |
|-----------|---------|-----------------|
| **F**ast | Milliseconds per test | Seconds per test |
| **I**solated | No shared state between tests | Tests fail in certain orders |
| **R**epeatable | Same result every run | Fails on CI but passes locally |
| **S**elf-validating | Automated pass/fail | Manual inspection needed |
| **T**imely | Written alongside the code | Tests written long after |

**Additional practices:**

| Practice | Description |
|----------|-------------|
| **Test behavior, not implementation** | Assert on user-visible output, not internal state. |
| **Use accessible queries** | Prefer `getByRole`, `getByLabelText`, `getByText` over `data-testid` or CSS selectors. |
| **Await async assertions** | Use `async`/`await` and `findBy*`/`waitFor`; never leave promises unawaited. |
| **Mock only boundaries** | Mock network, storage, and time — not internal modules. |
| **Prefer fakes over mocks** | Fakes are less brittle and more realistic. |
| **One logical concept per test** | A failing test should point to one clear cause. |
| **Avoid hardcoded sleeps** | Use dynamic waits or fake timers. |
| **Keep test data self-contained** | No "mystery guests" — create all needed data in the test. |

**Constraints and Limitations:**

- 100% coverage does not guarantee correctness — coverage measures execution, not assertion quality.
- Over-mocking produces tests that pass even when integration is broken. 
- Disabling `eslint` rules for hooks or testing should be a red flag, not a routine fix.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Behavior-Focused vs. Implementation-Focused Test

```jsx
// ❌ IMPLEMENTATION-FOCUSED: asserts on internal state
test('increments internal counter', () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1); // This is the public API — okay
});

// ✅ BEHAVIOR-FOCUSED: asserts on what the user sees
test('displays the incremented count', async () => {
  const user = userEvent.setup();
  render(<Counter />);

  await user.click(screen.getByRole('button', { name: /increment/i }));

  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**Why the second test is better:** It verifies the user-observable result, not the internal hook value. It survives refactoring that changes how the count is stored.

#### Example 2: FIRST-Compliant Test Suite

```js
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { createTestDb } from './testing/createTestDb.js';

let db, service;

beforeEach(() => {
  db = createTestDb(); // Fresh in-memory DB: Fast, Isolated, Repeatable
  service = new UserService(db);
});

afterEach(() => {
  db.close(); // Cleanup: Isolated
});

describe('UserService', () => {
  it('creates a user with the given name', async () => {
    const user = await service.create({ name: 'Alice' });
    expect(user.name).toBe('Alice'); // Self-validating
  });

  it('returns null for a missing user', async () => {
    const user = await service.getUser(999);
    expect(user).toBeNull();
  });
});
```

**Why this follows FIRST:** The in-memory database makes tests fast and isolated. The assertions are self-validating. The `beforeEach`/`afterEach` setup ensures repeatability. The tests are written alongside the service (timely).

### Real-World Cases

- **CI/CD pipelines:** FIRST-compliant tests run quickly and reliably on every push.
- **Code reviews:** Best practices provide a checklist for evaluating test quality.
- **Legacy test suites:** Refactoring slow, flaky, implementation-coupled tests using these principles restores speed and trust.

### References

- Scrimba — How to Test React Apps [2026] — https://scrimba.com/articles/how-to-test-react-apps-2026/
- Adobe — Unit Testing Guide — https://github.com/adobe/skills/blob/4130a3c138c88ae29802fbc153d3e7beea2a0637/plugins/aem/edge-delivery-services/skills/testing-blocks/references/unit-testing.md
- Redux — Writing Tests: Guiding Principles — https://redux.js.org/usage/writing-tests

---

## References

- Scrimba — How to Test React Apps [2026]: A Practical Guide — https://scrimba.com/articles/how-to-test-react-apps-2026/
- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- ReactUse — hook-test — https://github.com/childrentime/reactuse/blob/7081b1810215818882c739be8685cdacdb6ffb16/.claude/skills/hook-test/SKILL.md
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/
- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- Redux Toolkit — Testing Slice Reducers — https://github.com/agents-inc/skills/blob/main/skills/src/skills/web-state-redux-toolkit/examples/testing.md
- Egghead — Unit Testing a Memoized Selector — https://egghead.io/lessons/react-unit-testing-a-memoized-selector-that-derives-data-from-multiple-reducers
- Galaxy Project — Implement logic in pure functions when possible — https://github.com/galaxyproject/galaxy/blob/a73355ea2e5c47c62c5c79547b0b5e330e0695f5/client/docs/unit-testing/strategies.md
- Steve Kinney — Implementing Some Basic Tests — https://stevekinney.com/courses/testing/implementing-some-basic-tests
- Adobe — Unit Testing Guide — https://github.com/adobe/skills/blob/4130a3c138c88ae29802fbc153d3e7beea2a0637/plugins/aem/edge-delivery-services/skills/testing-blocks/references/unit-testing.md
- ai-skills — test-async-functions — https://github.com/turkeryildirim/ai-skills/blob/main/ai-skills/javascript/rules/test-async-functions.md
- Vitest/Jest Patterns for Accurate Tests — https://github.com/your-org/your-repo/blob/main/docs/testing/vitest-jest-patterns.md
- Steve Kinney — Using Async/Await — https://stevekinney.com/courses/testing/using-async-await
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- NestJS — Use Testing Module for Unit Tests — https://github.com/nestjs/docs.nestjs.com/blob/master/content/fundamentals/testing.md
- @sop-cli/utils — Utility function testing principles — https://www.npmjs.com/package/@sop-cli/utils