# Testing Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Testing patterns are reusable, proven structures and techniques for organizing test code so that it is readable, maintainable, deterministic, and effective at catching defects.

**Technical Definition:** Testing patterns are architectural and stylistic conventions applied to automated tests. They include **structural patterns** (Arrange-Act-Assert, Given-When-Then) that organize test bodies; **domain-specific patterns** for testing React hooks, effects, context, reducers, selectors, forms, routing, and data fetching; **verification patterns** (snapshot, visual regression, accessibility) that define how outcomes are checked; and **meta-patterns** (best practices) that govern isolation, query priority, async handling, and maintainability. Patterns reduce cognitive load, make failures diagnostic, and ensure tests remain stable as production code evolves.

**Beginner-Friendly Explanation:** Testing patterns are the "recipes" for writing good tests. They tell you how to structure a test (arrange, act, assert), how to test each kind of code (hooks, forms, routing), and how to check results (snapshots, accessibility). Following patterns keeps your tests clean and reliable.

### Key Characteristics

- **Structural consistency:** Every test follows the same shape, making the suite predictable.
- **Behavior-first:** Patterns emphasize user-visible outcomes over implementation details.
- **Deterministic:** Patterns include explicit async handling, fake timers, and isolation guarantees.
- **Composable:** Domain patterns (hooks, forms, routing) combine with structural patterns (AAA) and verification patterns (accessibility).
- **Tool-agnostic:** Patterns apply across Jest, Vitest, Playwright, and Cypress, even when syntax differs.

### Prerequisites

- A test runner (Vitest or Jest).
- React Testing Library (`render`, `screen`, `renderHook`).
- `@testing-library/user-event` for realistic interactions.
- `@testing-library/jest-dom` for semantic DOM matchers.
- MSW for network-level mocking (data-fetching patterns).
- Understanding of the testing pyramid and FIRST principles.

### Related Programming Areas

- **Unit testing** — hooks, reducers, selectors, pure functions.
- **Component testing** — props, state, context, forms.
- **Integration testing** — routing, data fetching, error handling.
- **Visual and accessibility testing** — snapshot, visual regression, a11y audits.
- **End-to-end testing** — Playwright/Cypress patterns (referenced for routing/forms).

### Core Concepts / Features

---

## 1. Arrange-Act-Assert (AAA)

### Definitions

**Core Definition:** Arrange-Act-Assert is a structural pattern that divides every test into three clear phases: set up the conditions, execute the behavior, and verify the outcome.

**Technical Definition:** AAA enforces the single-responsibility principle within a test body. **Arrange** prepares inputs, dependencies, and the system under test. **Act** invokes exactly one behavior. **Assert** verifies the observable outcome. Mixing phases (e.g., asserting inside the Act phase or performing multiple Acts) obscures the test's intent and makes failures harder to diagnose.

**Beginner-Friendly Explanation:** Every test should have three parts: set things up (Arrange), do the thing (Act), and check the result (Assert). This makes tests easy to read and debug.

### Purposes

- To make test intent immediately readable.
- To isolate the behavior under test from setup and verification.
- To ensure each test verifies exactly one outcome.
- To make failures point to a specific phase.

### Syntax Rules and Structure

```javascript
test('descriptive behavior', () => {
  // Arrange
  const input = 5;
  const expected = 25;

  // Act
  const actual = square(input);

  // Assert
  expect(actual).toBe(expected);
});
```

**Component Breakdown:**

| Phase | Responsibility |
|-------|----------------|
| Arrange | Create inputs, mocks, render components, seed state. |
| Act | Call the function, click the button, submit the form. |
| Assert | Verify the output or DOM state. |

**Constraints and Limitations:**

- Do not place assertions in the Arrange or Act phases.
- Do not perform multiple Acts in one test — split into separate tests.
- Setup shared across tests belongs in `beforeEach`, not in the Arrange phase of every test.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: AAA in a Component Test

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect, vi } from 'vitest';
import { Button } from './Button';

test('calls onClick when clicked', async () => {
  // Arrange
  const user = userEvent.setup();
  const handleClick = vi.fn();
  render(<Button label="Submit" onClick={handleClick} />);

  // Act
  await user.click(screen.getByRole('button', { name: /submit/i }));

  // Assert
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

**Expected Output:** The test passes. The three phases are clearly separated, and the assertion verifies the single behavior.

### Real-World Cases

- **Every unit and component test:** AAA is the default structure for well-written tests.
- **Code reviews:** Reviewers check that tests follow AAA to ensure clarity.
- **Test debugging:** When a test fails, the phase indicates whether the bug is in setup, behavior, or expectation.

### References

- Semaphore — AAA (Arrange-Act-Assert) testing pattern — https://semaphore.io/blog/aaa-pattern-test-automation
- Martin Fowler — GivenWhenThen — https://martinfowler.com/bliki/GivenWhenThen.html

---

## 2. Given-When-Then

### Definitions

**Core Definition:** Given-When-Then is a BDD-style structural pattern that describes a test in terms of its initial context (Given), the action performed (When), and the expected outcome (Then).

**Technical Definition:** GWT is the natural-language equivalent of AAA, often used with BDD frameworks (Cucumber, SpecFlow) but equally applicable to plain unit tests. **Given** establishes the preconditions, **When** describes the event or action, and **Then** specifies the observable outcome. GWT emphasizes behavior from the user's perspective and is well-suited to acceptance tests and stakeholder-readable specifications.

**Beginner-Friendly Explanation:** Given-When-Then is another way to structure a test: "Given this situation, when I do this, then this should happen." It reads like a story and is easy for non-developers to understand.

### Purposes

- To express tests in business-readable language.
- To align tests with acceptance criteria and user stories.
- To bridge the communication gap between technical and non-technical stakeholders.
- To structure BDD scenarios consistently.

### Syntax Rules and Structure

```gherkin
Given the user is on the login page
When the user enters valid credentials
Then the user is redirected to the dashboard
```

**Component Breakdown:**

| Keyword | Maps to | Meaning |
|---------|---------|---------|
| Given | Arrange | Preconditions and context. |
| When | Act | The action or event. |
| Then | Assert | The expected outcome. |

**Constraints and Limitations:**

- GWT can become verbose for low-level unit tests — AAA is often more concise there.
- BDD frameworks add toolchain complexity; plain comments suffice for most tests.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: GWT with Plain Vitest

```javascript
test('user logs in successfully', async () => {
  // Given: a registered user on the login page
  const user = userEvent.setup();
  render(<LoginPage />);
  const credentials = { email: 'alice@example.com', password: 'secret' };

  // When: the user submits valid credentials
  await user.type(screen.getByLabelText(/email/i), credentials.email);
  await user.type(screen.getByLabelText(/password/i), credentials.password);
  await user.click(screen.getByRole('button', { name: /log in/i }));

  // Then: the dashboard is displayed
  expect(await screen.findByRole('heading', { name: /dashboard/i })).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The GWT comments make the scenario readable even without a BDD framework.

#### Example 2: Gherkin with Cucumber

```gherkin
Feature: Login
  Scenario: Valid credentials
    Given I am on the login page
    When I submit username "alice" and password "secret"
    Then I should see the dashboard
```

**Expected Output:** The Cucumber runner maps each step to a step definition and executes the scenario.

### Real-World Cases

- **Acceptance criteria:** GWT scenarios directly mirror user story acceptance criteria.
- **Cross-functional teams:** Product owners can read and validate GWT scenarios.
- **Regulated industries:** GWT scenarios serve as auditable documentation.

### References

- Martin Fowler — GivenWhenThen — https://martinfowler.com/bliki/GivenWhenThen.html
- Cucumber — Gherkin Reference — https://cucumber.io/docs/gherkin/reference/

---

## 3. Testing Hooks

### Definitions

**Core Definition:** Testing hooks means verifying that built-in React Hooks (`useState`, `useEffect`, `useContext`, etc.) behave correctly within a component by observing their effects on the rendered output.

**Technical Definition:** Hooks cannot be called outside a React component or custom Hook, so testing them requires either rendering a component that uses them or using `renderHook` for custom Hooks. The test asserts on the observable consequences of the Hook — the rendered UI, the callback invocations, or the side effects — rather than on the Hook's internal state. For built-in Hooks used inside components, this is covered by component tests; for extracted custom Hooks, `renderHook` provides isolation.

**Beginner-Friendly Explanation:** You cannot test a hook directly — you test what it does. Render a component that uses the hook, interact with it, and check that the screen updates correctly.

### Purposes

- To verify that state, effects, and context behavior produce the correct UI.
- To catch bugs in Hook usage (missing dependencies, stale closures).
- To test extracted custom Hooks in isolation with `renderHook`.
- To ensure cleanup and side effects work correctly.

### Syntax Rules and Structure

```jsx
// Testing a built-in Hook via a component
render(<Counter />);
await user.click(screen.getByRole('button', { name: /increment/i }));
expect(screen.getByText('Count: 1')).toBeInTheDocument();

// Testing a custom Hook directly
const { result } = renderHook(() => useCounter());
act(() => result.current.increment());
expect(result.current.count).toBe(1);
```

**Constraints and Limitations:**

- Never assert on internal Hook state in a component test — assert on the UI.
- Custom Hooks that depend on context require a `wrapper` in `renderHook`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing `useState` Through the UI

```jsx
function Toggle() {
  const [on, setOn] = useState(false);
  return <button onClick={() => setOn(!on)}>{on ? 'ON' : 'OFF'}</button>;
}

test('toggles state when clicked', async () => {
  const user = userEvent.setup();
  render(<Toggle />);

  expect(screen.getByRole('button')).toHaveTextContent('OFF');
  await user.click(screen.getByRole('button'));
  expect(screen.getByRole('button')).toHaveTextContent('ON');
});
```

**Expected Output:** The test passes. The state change is verified through the button's text, not by inspecting the `on` variable.

### Real-World Cases

- **Interactive components:** Toggles, counters, accordions, and tabs.
- **Effect-driven behavior:** Components that fetch data or subscribe to events on mount.
- **Context consumers:** Components that read from a provider.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- Testing Library — renderHook — https://testing-library.com/docs/react-testing-library/api#renderhook

---

## 4. Testing Custom Hooks

### Definitions

**Core Definition:** Testing a custom Hook means rendering it in isolation with `renderHook`, exercising its returned functions, and asserting on its returned values and side effects.

**Technical Definition:** `renderHook` from `@testing-library/react` mounts the Hook inside a minimal test component, exposing its return value via `result.current`. State updates must be wrapped in `act()`. Props can be changed via `rerender()`, and cleanup behavior can be tested via `unmount()`. Context-dependent Hooks require a `wrapper` option that provides the necessary providers.

**Beginner-Friendly Explanation:** A custom Hook test uses `renderHook` to run the Hook by itself, then calls its functions and checks its return values — no full component needed.

### Purposes

- To verify the logic of reusable custom Hooks independently of UI components.
- To test state transitions, side effects, and cleanup in isolation.
- To test Hooks with changing arguments via `rerender`.
- To ensure Hooks that use context receive the correct provider.

### Syntax Rules and Structure

```jsx
import { renderHook, act } from '@testing-library/react';

const { result } = renderHook(() => useCustomHook(initial));
act(() => result.current.update(newValue));
expect(result.current.value).toBe(newValue);
```

**Constraints and Limitations:**

- Every state update must be inside `act()`.
- Hooks that need context require the `wrapper` option.
- Timer-based Hooks require fake timers.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Custom Hook with Rerender

```jsx
import { renderHook } from '@testing-library/react';
import { useDebounce } from './useDebounce';

test('debounces value changes', () => {
  vi.useFakeTimers();
  const { result, rerender } = renderHook(
    ({ value }) => useDebounce(value, 300),
    { initialProps: { value: 'a' } }
  );

  rerender({ value: 'ab' });
  expect(result.current).toBe('a');

  vi.advanceTimersByTime(300);
  expect(result.current).toBe('ab');
  vi.useRealTimers();
});
```

**Expected Output:** The Hook initially returns `'a'`; after `rerender` with `'ab'` and advancing fake timers, it returns `'ab'`.

### Real-World Cases

- **Data-fetching Hooks:** `useFetch`, `useAsync`, `useQuery` wrappers.
- **Storage Hooks:** `useLocalStorage`, `useSessionStorage`.
- **Event Hooks:** `useKeyPress`, `useWindowSize`, `useMediaQuery`.

### References

- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/

---

## 5. Testing Async Hooks

### Definitions

**Core Definition:** Testing async hooks means verifying that custom Hooks which perform asynchronous operations (data fetching, timers, subscriptions) correctly transition through loading, success, and error states.

**Technical Definition:** Async Hooks are tested with `renderHook`, `waitFor`, and `act` (or `waitForNextUpdate` in the deprecated `react-hooks` library). The test renders the Hook, waits for the async operation to settle via `waitFor`, and asserts on `result.current`. Network calls are mocked with MSW, and timers are controlled with fake timers. Async state updates that occur outside `act` produce warnings and flaky assertions.

**Beginner-Friendly Explanation:** Async hook tests check that a hook that fetches data shows loading first, then the data, or an error if something fails. You wait for the async operation to finish before checking.

### Purposes

- To verify loading, success, and error states of async Hooks.
- To ensure that async state updates do not cause warnings or race conditions.
- To test cleanup of pending operations on unmount.
- To validate that Hook dependencies trigger re-fetches correctly.

### Syntax Rules and Structure

```jsx
const { result } = renderHook(() => useAsyncData(url));

await waitFor(() => {
  expect(result.current.data).toBeDefined();
});
```

**Constraints and Limitations:**

- Do not use `setTimeout` with real delays — use `waitFor` or fake timers.
- Async state updates must be wrapped in `act` or awaited via `waitFor`.
- Unmounted async updates should be guarded (e.g., with a `cancelled` flag).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing an Async Data Hook with MSW

```jsx
import { renderHook, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { useUser } from './useUser';

test('loads user data', async () => {
  server.use(
    http.get('/api/users/1', () => HttpResponse.json({ name: 'Alice' }))
  );

  const { result } = renderHook(() => useUser('1'));

  expect(result.current.loading).toBe(true);

  await waitFor(() => {
    expect(result.current.loading).toBe(false);
    expect(result.current.data).toEqual({ name: 'Alice' });
  });
});
```

**Expected Output:** The Hook initially reports loading; after the mocked API resolves, it reports the data.

### Real-World Cases

- **Data fetching:** Testing `useFetch`, `useQuery` wrappers, and custom API Hooks.
- **Polling:** Testing `useInterval`-based data refresh with fake timers.
- **Debounced search:** Testing that async search results update after the debounce delay.

### References

- Test Async Hooks — https://raw.githubusercontent.com/djankies/claude-configs/master/skills/testing/react-hooks-testing/SKILL.md
- Testing Library — Async Methods — https://testing-library.com/docs/dom-testing-library/api-async

---

## 6. Testing Effects

### Definitions

**Core Definition:** Testing effects means verifying that `useEffect` side effects — subscriptions, timers, network requests, DOM manipulation — execute, clean up, and respond to dependency changes correctly.

**Technical Definition:** Effects are tested indirectly through their observable consequences: a subscription updates the UI, a timer triggers a state change, a fetch populates data, or a cleanup removes a listener. For custom Hooks that wrap effects, `renderHook` with `rerender` and `unmount` verifies that effects re-run when dependencies change and clean up on unmount. React Testing Library automatically wraps rendering and interactions in `act`, flushing effects after each update.

**Beginner-Friendly Explanation:** Effects are things your component does after rendering — like setting a timer or subscribing to events. You test them by checking their results: does the UI update? Is the listener removed on unmount?

### Purposes

- To verify that effects run when expected (on mount, on dependency change).
- To confirm that cleanup functions run on unmount and dependency change.
- To test effect-driven data fetching, subscriptions, and timers.
- To catch missing-dependency and stale-closure bugs.

### Syntax Rules and Structure

```jsx
const { result, rerender, unmount } = renderHook(({ id }) => useSubscription(id), {
  initialProps: { id: 1 },
});

rerender({ id: 2 }); // Effect should re-run
unmount();           // Cleanup should run
```

**Constraints and Limitations:**

- Effects are not tested directly — only through their consequences.
- Testing cleanup requires `unmount()` or re-rendering with changed dependencies.
- Fake timers are needed for time-based effects.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Effect Re-run and Cleanup

```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    document.title = title;
    return () => { document.title = 'Default'; };
  }, [title]);
}

test('updates and resets the document title', () => {
  const { rerender, unmount } = renderHook(
    ({ title }) => useDocumentTitle(title),
    { initialProps: { title: 'Page A' } }
  );

  expect(document.title).toBe('Page A');

  rerender({ title: 'Page B' });
  expect(document.title).toBe('Page B');

  unmount();
  expect(document.title).toBe('Default');
});
```

**Expected Output:** The title updates on each render and resets on unmount, verifying both the effect and its cleanup.

### Real-World Cases

- **Subscriptions:** WebSocket or event-emitter subscriptions that must clean up on unmount.
- **Timers:** Intervals that must be cleared on unmount.
- **Document metadata:** Updating `<title>` or meta tags based on the current route.

### References

- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect

---

## 7. Testing Context

### Definitions

**Core Definition:** Testing context means rendering a component with a specific Context value via a provider wrapper and asserting that the component reads and uses that value correctly.

**Technical Definition:** Components that call `useContext` require a matching provider ancestor. In tests, the provider is supplied via the `wrapper` option of `render()`/`renderHook()`, or via a custom `renderWithProviders` helper. Each test supplies the context value relevant to its scenario — authenticated vs. anonymous user, dark vs. light theme, feature flag on vs. off. Context is never mocked directly; the real provider is used with controlled values.

**Beginner-Friendly Explanation:** If a component reads data from a Context, you wrap it in the matching Provider when testing, give the Provider a fake value, and check that the component displays it correctly.

### Purposes

- To test components that depend on Context without building the full app tree.
- To verify rendering for different context values (auth, theme, locale).
- To test custom Hooks that call `useContext`.
- To ensure context-dependent conditional rendering is correct.

### Syntax Rules and Structure

```jsx
render(<Component />, {
  wrapper: ({ children }) => (
    <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
  ),
});
```

**Constraints and Limitations:**

- Never mock `useContext` — provide the real provider with a controlled value.
- Multiple providers require nesting in the wrapper.
- A custom `renderWithProviders` reduces boilerplate across a suite.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Different Context Values

```jsx
const ThemeContext = createContext('light');

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={`btn-${theme}`}>Click</button>;
}

function renderWithTheme(theme) {
  return render(
    <ThemeContext.Provider value={theme}>
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

test('renders with the dark theme', () => {
  renderWithTheme('dark');
  expect(screen.getByRole('button')).toHaveClass('btn-dark');
});

test('renders with the light theme', () => {
  renderWithTheme('light');
  expect(screen.getByRole('button')).toHaveClass('btn-light');
});
```

**Expected Output:** Both tests pass, verifying that the component respects the provided theme value.

### Real-World Cases

- **Authentication context:** Testing authenticated vs. unauthenticated UI.
- **Feature flags:** Testing that feature-gated UI appears only when the flag is enabled.
- **Localization:** Testing that components render the correct language.

### References

- Testing Library — Setup: Custom Render with providers — https://testing-library.com/docs/react-testing-library/setup
- Souvlaki — Composable test wrappers for context-heavy components — https://www.npmjs.com/package/souvlaki

---

## 8. Testing Reducers

### Definitions

**Core Definition:** Testing a reducer means calling the reducer with a given state and action and asserting that it returns the correct new state, without mutating the original.

**Technical Definition:** A reducer is a pure function `(state, action) => newState`. Its test supplies a known state and action, calls the reducer, and asserts on the returned state. Reducers are tested directly — no rendering, no mocking. For each action type declared by the reducer, there should be at least one test covering the expected transition, plus tests for unknown actions (which should return the current state) and immutability.

**Beginner-Friendly Explanation:** A reducer test gives the reducer a state and an action, and checks that the new state is correct. It is simple, fast, and needs no rendering.

### Purposes

- To verify that each action produces the correct state transition.
- To ensure that the reducer does not mutate the original state.
- To cover default/unknown action behavior.
- To test complex nested state updates.

### Syntax Rules and Structure

```javascript
const newState = reducer(previousState, action);
expect(newState).toEqual(expectedState);
expect(newState).not.toBe(previousState); // Immutability
```

**Constraints and Limitations:**

- Test each action type separately.
- Assert immutability for reducers that handle nested or array state.
- Redux recommends integration tests over isolated reducer tests for simple reducers; unit-test complex ones.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Slice Reducer

```javascript
import reducer, { increment, decrement } from './counterSlice';

test('handles increment', () => {
  const previous = { value: 5 };
  const next = reducer(previous, increment());
  expect(next.value).toBe(6);
  expect(next).not.toBe(previous);
});

test('returns current state for unknown action', () => {
  const state = { value: 5 };
  expect(reducer(state, { type: 'unknown' })).toBe(state);
});
```

**Expected Output:** Both tests pass. The reducer increments correctly and returns the same state for unknown actions.

### Real-World Cases

- **Shopping carts:** Add, remove, and update-quantity actions.
- **Authentication:** Login, logout, and token-refresh transitions.
- **Form wizards:** Next-step, previous-step, and reset actions.

### References

- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- Redux Toolkit — Testing Slice Reducers — https://github.com/agents-inc/skills/blob/main/skills/src/skills/web-state-redux-toolkit/examples/testing.md

---

## 9. Testing Selectors

### Definitions

**Core Definition:** Testing a selector means supplying a known state object and asserting that the selector returns the correct derived value — and, for memoized selectors, that recomputation is minimized.

**Technical Definition:** Selectors are pure functions `(state) => derivedValue`. Their tests construct a representative state fixture, call the selector, and assert on the result. Memoized selectors created with `createSelector` (Reselect/RTK) expose `.recomputations()` and `.resetRecomputations()`, allowing tests to verify that repeated calls with the same state do not recompute. Selectors are never mocked — they are tested directly with real state.

**Beginner-Friendly Explanation:** A selector test gives the selector a fake state and checks that it extracts or computes the right value. For memoized selectors, you also check that it does not recompute unnecessarily.

### Purposes

- To verify derived data (filtered lists, totals, formatted values).
- To test memoization behavior.
- To catch bugs in complex state derivation.
- To document the shape and meaning of derived state.

### Syntax Rules and Structure

```javascript
const state = { items: [...], taxRate: 0.1 };
expect(selectTotal(state)).toBe(27.5);
expect(selectTotal.recomputations()).toBe(1);
```

**Constraints and Limitations:**

- Do not mock selectors — test them with real state fixtures.
- Memoized selectors require recomputation-count assertions.
- Selectors that depend on multiple reducers need composite state fixtures.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Memoized Selector

```javascript
import { createSelector } from '@reduxjs/toolkit';

const selectItems = (state) => state.cart.items;
const selectTaxRate = (state) => state.cart.taxRate;

export const selectTotalWithTax = createSelector(
  [selectItems, selectTaxRate],
  (items, taxRate) => items.reduce((sum, i) => sum + i.price * i.qty, 0) * (1 + taxRate)
);

test('computes total and memoizes', () => {
  const state = { cart: { items: [{ price: 10, qty: 2 }, { price: 5, qty: 1 }], taxRate: 0.1 } };
  expect(selectTotalWithTax(state)).toBeCloseTo(27.5);

  selectTotalWithTax(state);
  expect(selectTotalWithTax.recomputations()).toBe(1);
});
```

**Expected Output:** The total is `27.5`, and calling the selector twice with the same state recomputes only once.

### Real-World Cases

- **E-commerce carts:** Subtotal, tax, and shipping calculations.
- **Filtered lists:** Deriving visible items from filters and search terms.
- **Dashboard metrics:** Aggregating counts and averages from normalized state.

### References

- Redux — Writing Tests: Selectors — https://redux.js.org/usage/writing-tests
- Egghead — Unit Testing a Memoized Selector — https://egghead.io/lessons/react-unit-testing-a-memoized-selector-that-derives-data-from-multiple-reducers

---

## 10. Testing Forms

### Definitions

**Core Definition:** Testing forms means verifying that form inputs capture user data, validation rules fire correctly, submission handlers receive the right data, and success/error states render as expected.

**Technical Definition:** Form tests render the form (with its real validation logic), fill fields with `user.type`, submit with `user.click`, and assert on the resulting UI or the submit handler's arguments. Validation errors are asserted via `role="alert"` or accessible error messages. Async submission is verified with `waitFor` or `findBy*`, and the API is mocked with MSW. The submit button's disabled state during submission is a common assertion.

**Beginner-Friendly Explanation:** Form tests fill in fields like a user would, click submit, and check that the right data is sent and the right success or error message appears.

### Purposes

- To verify that inputs capture and submit the correct data.
- To confirm that validation errors appear and block submission.
- To test loading and disabled states during submission.
- To test success and error handling after the API responds.

### Syntax Rules and Structure

```jsx
const user = userEvent.setup();
render(<Form onSubmit={mockSubmit} />);

await user.type(screen.getByLabelText(/email/i), 'test@example.com');
await user.click(screen.getByRole('button', { name: /submit/i }));

await waitFor(() => {
  expect(mockSubmit).toHaveBeenCalledWith({ email: 'test@example.com' });
});
```

**Constraints and Limitations:**

- Do not assert on internal form state — assert on submitted data and visible UI.
- Async validation requires `waitFor` or `findBy*`.
- Mock the API with MSW, not the form's submit handler (unless testing the handler in isolation).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Validation and Submission

```jsx
function LoginForm({ onSubmit }) {
  const [error, setError] = useState('');
  async function handleSubmit(e) {
    e.preventDefault();
    const email = e.target.email.value;
    if (!email.includes('@')) {
      setError('Invalid email');
      return;
    }
    setError('');
    await onSubmit({ email });
  }
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" />
      <button type="submit">Submit</button>
      {error && <p role="alert">{error}</p>}
    </form>
  );
}

test('shows validation error for invalid email', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'invalid');
  await user.click(screen.getByRole('button', { name: /submit/i }));

  expect(screen.getByRole('alert')).toHaveTextContent('Invalid email');
  expect(onSubmit).not.toHaveBeenCalled();
});
```

**Expected Output:** The test passes. The invalid email triggers the validation error, and the submit handler is not called.

### Real-World Cases

- **Login/signup forms:** Testing validation and submission with mocked API responses.
- **Checkout forms:** Verifying that shipping and payment data are submitted correctly.
- **Profile editing:** Testing that updated user data is saved and reflected in the UI.

### References

- Testing TanStack Form — https://github.com/fusengine/agents/blob/main/plugins/nextjs-expert/skills/nextjs-tanstack-form/references/testing.md
- How to mock API requests in React tests — https://coreui.io/answers/how-to-mock-api-requests-in-react-tests/

---

## 11. Testing Routing

### Definitions

**Core Definition:** Testing routing means verifying that navigation renders the correct components, route parameters are read correctly, and navigation actions (links, redirects) produce the expected URL and UI state.

**Technical Definition:** React Router relies on context, so routing components must be wrapped in a router during tests. `MemoryRouter` is the standard choice: it stores navigation history in memory, accepts `initialEntries` to set the starting route, and does not depend on the browser URL. For data routers, `createMemoryRouter` is used. Route parameters are tested by rendering the route with a parameterized path and asserting that `useParams` returns the expected value. Programmatic navigation is tested by mocking `useNavigate` or by asserting on the UI after a navigation-triggering interaction.

**Beginner-Friendly Explanation:** Routing tests check that navigating to a URL shows the right page, and that clicking links or buttons takes you where you expect.

### Purposes

- To verify that each route renders the correct component.
- To test route parameters and query strings.
- To confirm that navigation actions change the rendered UI.
- To test protected routes and redirects.

### Syntax Rules and Structure

```jsx
render(
  <MemoryRouter initialEntries={['/users/42']}>
    <Routes>
      <Route path="/users/:id" element={<UserProfile />} />
    </Routes>
  </MemoryRouter>
);
```

**Constraints and Limitations:**

- Always wrap routing components in a router.
- `MemoryRouter` does not change the browser URL — that is intentional.
- Mock `useNavigate` only when verifying programmatic navigation calls.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Route Parameters

```jsx
import { MemoryRouter, Routes, Route } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  return <h1>User ID: {id}</h1>;
}

test('renders the user id from the route parameter', () => {
  render(
    <MemoryRouter initialEntries={['/users/42']}>
      <Routes>
        <Route path="/users/:id" element={<UserProfile />} />
      </Routes>
    </MemoryRouter>
  );

  expect(screen.getByText('User ID: 42')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. `MemoryRouter` starts at `/users/42`, the route matches, and `useParams` extracts `id = "42"`.

### Real-World Cases

- **User profile pages:** Testing `/users/:id` renders the correct user.
- **Protected routes:** Testing that unauthenticated users are redirected to login.
- **Navigation menus:** Testing that clicking a link renders the target page.

### References

- MemoryRouter — React Router — https://reactrouter.com/6.30.3/router-components/memory-router
- What are the considerations for testing components that utilize React Router — https://www.edureka.co/community/309032/considerations-testing-components-utilize-router-navigation

---

## 12. Testing Data Fetching

### Definitions

**Core Definition:** Testing data fetching means verifying that a component requests data from the correct endpoint, renders loading and success states, and handles errors appropriately.

**Technical Definition:** Data-fetching tests render the component, mock the network with MSW at the HTTP layer, assert on the loading state, wait for the resolved data with `findBy*` or `waitFor`, and assert on the rendered output. Error states are tested by returning error responses. The component's real data-fetching logic (fetch, axios, React Query) runs — only the network is mocked. MSW is preferred over mocking `fetch` directly because it exercises the real HTTP client and request configuration.

**Beginner-Friendly Explanation:** Data-fetching tests check that your component asks the server for data, shows a loading state, then displays the data — or an error if the request fails. MSW fakes the server.

### Purposes

- To verify that the correct endpoint is called.
- To confirm loading, success, and error states render correctly.
- To test that fetched data is transformed and displayed correctly.
- To catch bugs without depending on a real backend.

### Syntax Rules and Structure

```jsx
server.use(http.get('/api/users/1', () => HttpResponse.json({ name: 'Alice' })));
render(<UserProfile id="1" />);
expect(screen.getByText('Loading...')).toBeInTheDocument();
expect(await screen.findByText('Alice')).toBeInTheDocument();
```

**Constraints and Limitations:**

- Mock at the network layer with MSW — do not mock `fetch` directly.
- Test error states explicitly with error responses.
- Do not assert on internal loading state variables — assert on visible loading UI.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Loading, Success, and Error

```jsx
test('shows loading then data', async () => {
  server.use(http.get('/api/users/1', () => HttpResponse.json({ name: 'Alice' })));
  render(<UserProfile userId="1" />);
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});

test('shows an error when the API fails', async () => {
  server.use(http.get('/api/users/1', () => HttpResponse.json({}, { status: 500 })));
  render(<UserProfile userId="1" />);
  expect(await screen.findByRole('alert')).toHaveTextContent('Error');
});
```

**Expected Output:** The first test verifies loading followed by data; the second verifies the error alert.

### Real-World Cases

- **Dashboard widgets:** Each widget fetches its own data with MSW handlers.
- **Search results:** Testing that the query is sent and results render.
- **Pagination:** Testing that page changes trigger new API calls.

### References

- How to mock API requests in React tests — https://coreui.io/answers/how-to-mock-api-requests-in-react-tests/
- Using Mock Service Worker to Improve Jest Unit Tests — https://www.wwt.com/article/using-mock-service-worker-to-improve-jest-unit-tests

---

## 13. Testing Error Handling

### Definitions

**Core Definition:** Testing error handling means verifying that components and functions gracefully handle errors — displaying fallback UI, showing error messages, or logging — when operations fail.

**Technical Definition:** Error-handling tests simulate failure conditions: rejected Promises, HTTP error responses, thrown exceptions, and invalid inputs. They assert that the component renders an error message (often with `role="alert"`), that an Error Boundary catches render errors, and that the app does not crash. Error Boundaries are tested by rendering a child that throws and asserting on the fallback UI. Async errors are tested with rejected Promises and `waitFor`. React logs caught errors to `console.error`, which should be silenced in tests.

**Beginner-Friendly Explanation:** Error-handling tests check that when something goes wrong — a failed API call, a crash in a component — the user sees a helpful message instead of a broken page.

### Purposes

- To verify that errors produce user-visible messages.
- To confirm that Error Boundaries catch render errors.
- To test that failed API calls are handled gracefully.
- To ensure that invalid input produces validation errors.

### Syntax Rules and Structure

```jsx
test('handles API failure', async () => {
  server.use(http.get('/api/data', () => HttpResponse.json({}, { status: 500 })));
  render(<DataComponent />);
  expect(await screen.findByRole('alert')).toHaveTextContent('Failed to load');
});

test('Error Boundary catches a render error', () => {
  vi.spyOn(console, 'error').mockImplementation(() => {});
  render(<ErrorBoundary><Bomb /></ErrorBoundary>);
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
});
```

**Constraints and Limitations:**

- Error Boundaries do not catch errors in event handlers or async code.
- Silence `console.error` in tests to keep output clean.
- Async errors require `findBy*` or `waitFor`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Error Boundary

```jsx
function Bomb() { throw new Error('kaboom'); }

test('shows fallback when child throws', () => {
  vi.spyOn(console, 'error').mockImplementation(() => {});
  render(<ErrorBoundary><Bomb /></ErrorBoundary>);
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The Error Boundary catches the thrown error and renders the fallback.

### Real-World Cases

- **Route-level boundaries:** Showing a friendly error page when a route crashes.
- **Widget-level boundaries:** Preventing one broken widget from crashing the dashboard.
- **API error handling:** Showing retry UI when a request fails.

### References

- Error Boundary Testing Example (HuggingFace) — https://huggingface.co/spaces/Ma-Ri-Ba-Ku/alto-llm-corrector/blob/a94837f5c02ea105429f5c9a7d7e5f6e821113f6/frontend/src/components/ErrorBoundary.test.tsx
- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

---

## 14. Snapshot Testing

### Definitions

**Core Definition:** Snapshot testing captures the serialized output of a component or value and compares it to a stored baseline; the test fails when the output changes, prompting the developer to review and approve the diff.

**Technical Definition:** On first run, the snapshot is written to a `__snapshots__` directory. On subsequent runs, the serialized output is compared to the baseline. Changes cause a failure with a diff. Snapshots can capture rendered HTML (`toMatchSnapshot`), inline values (`toMatchInlineSnapshot`), or serialized data. The primary value of snapshots is detecting **unintended changes**; their primary risk is **accidental approval** of regressions without review.

**Beginner-Friendly Explanation:** A snapshot test takes a "photo" of your component's output and compares future renders to it. If something changes, the test fails so you can decide whether the change was intended.

### Purposes

- To detect unintended changes in rendered output.
- To quickly add broad coverage for components without writing explicit assertions.
- To document the structure of serialized data.

### Syntax Rules and Structure

```jsx
test('renders correctly', () => {
  const { container } = render(<Button label="Click" />);
  expect(container.firstChild).toMatchSnapshot();
});
```

**Constraints and Limitations:**

- Snapshots are easy to accidentally approve without reviewing — treat snapshot updates as code changes.
- Large snapshots are hard to review.
- Snapshots are brittle — they fail on whitespace and class-name order changes.
- Avoid snapshots for component tests unless there is a clear, justified reason.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Snapshot Test with Review

```jsx
test('Button renders with the correct structure', () => {
  const { container } = render(<Button label="Click me" />);
  expect(container.firstChild).toMatchSnapshot();
});
```

**Expected Output:** The first run creates the snapshot file. Subsequent runs pass if the output matches; fail if it differs.

**Why use sparingly:** The snapshot captures the full rendered HTML. A change in a CSS class name causes a failure even if behavior is unchanged. Use snapshots for serialized data (e.g., configuration objects) more often than for components.

### Real-World Cases

- **Configuration serialization:** Snapshotting generated configs or error messages.
- **Serialized API responses:** Detecting changes in normalized data shapes.
- **UI component libraries:** Catching unintended markup changes (with careful review).

### References

- Jest — Snapshot Testing — https://jestjs.io/docs/snapshot-testing
- Vitest — Snapshot — https://vitest.dev/guide/snapshot

---

## 15. Visual Regression Testing

### Definitions

**Core Definition:** Visual regression testing captures screenshots of rendered UI and compares them to baseline images to detect unintended visual changes.

**Technical Definition:** Visual regression tools render the UI in a browser (often headless), capture a screenshot, and compare it pixel-by-pixel or perceptually to a stored baseline. Differences are highlighted as visual diffs. This catches layout breaks, styling regressions, and cross-browser rendering differences that DOM-based assertions miss. Playwright's `toHaveScreenshot` and commercial tools like Chromatic and Percy are common implementations.

**Beginner-Friendly Explanation:** Visual regression testing takes a picture of your UI and compares it to a previous picture. If something moved or changed color unintentionally, the test shows you exactly what changed.

### Purposes

- To detect unintended visual changes (layout, color, spacing, typography).
- To verify cross-browser and cross-device rendering consistency.
- To catch CSS regressions that functional tests miss.

### Syntax Rules and Structure

```javascript
test('button looks correct', async ({ page }) => {
  await page.goto('/components/button');
  await expect(page.locator('.button')).toHaveScreenshot('button.png');
});
```

**Constraints and Limitations:**

- Sensitive to environment differences (fonts, antialiasing, viewport).
- Baselines require careful maintenance and review.
- Can be slow and resource-intensive.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Playwright Visual Comparison

```javascript
test('dashboard layout is stable', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page).toHaveScreenshot('dashboard.png', {
    threshold: 0.2,
  });
});
```

**Expected Output:** The first run creates the baseline; subsequent runs pass if the screenshot matches within the threshold, or fail with a visual diff.

### Real-World Cases

- **Design systems:** Ensuring components render consistently across browsers.
- **Marketing pages:** Catching layout breaks after CSS changes.
- **Accessibility:** Verifying focus states and high-contrast modes.

### References

- Playwright — Visual Comparisons — https://playwright.dev/docs/test-snapshots
- Chromatic — Visual Testing — https://www.chromatic.com/features/visual-test

---

## 16. Accessibility Testing

### Definitions

**Core Definition:** Accessibility testing verifies that the UI is usable by people with disabilities — screen reader users, keyboard-only users, and users with visual impairments.

**Technical Definition:** Automated accessibility testing combines **semantic querying** (Testing Library's `getByRole`, `getByLabelText`) with **automated audits** (`jest-axe` or `axe-core`) that check for violations of WCAG rules. Semantic queries double as a11y checks: if an element cannot be found by its role or label, screen reader users likely cannot find it either. `jest-axe` scans the rendered DOM for issues like missing alt text, insufficient color contrast (limited), and invalid ARIA attributes.

**Beginner-Friendly Explanation:** Accessibility tests check that your app works for everyone — including people using screen readers or keyboards. Tools like `jest-axe` automatically scan for common accessibility problems.

### Purposes

- To catch accessibility violations before they reach users.
- To ensure interactive elements have accessible names and roles.
- To verify keyboard navigation and focus management.
- To enforce a11y standards in CI.

### Syntax Rules and Structure

```jsx
import { axe } from 'jest-axe';

test('has no accessibility violations', async () => {
  const { container } = render(<Form />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

**Constraints and Limitations:**

- Automated tools catch ~30–50% of accessibility issues — manual testing is still required.
- Color contrast checks are limited in jsdom.
- Keyboard navigation and screen reader behavior require manual or E2E testing.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Automated a11y Audit with jest-axe

```jsx
import { render, screen } from '@testing-library/react';
import { axe } from 'jest-axe';
import { LoginForm } from './LoginForm';

test('login form has no accessibility violations', async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

test('every input has an accessible label', () => {
  render(<LoginForm />);
  expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
  expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
});
```

**Expected Output:** Both tests pass. The form has no automated a11y violations, and each input is accessible by its label.

### Real-World Cases

- **Design systems:** Every component is audited for a11y violations.
- **Form accessibility:** Ensuring every input has a label and every error is announced.
- **CI a11y gates:** Failing the build if new violations are introduced.

### References

- jest-axe — GitHub Repository — https://github.com/nickcolley/jest-axe
- Testing Library — Accessibility Queries — https://testing-library.com/docs/queries/about/#priority
- Deque — axe-core — https://github.com/dequelabs/axe-core

---

## 17. Testing Best Practices

### Definitions

**Core Definition:** Testing best practices are the principles that make test suites fast, reliable, maintainable, and effective at catching regressions.

**Technical Definition:** Best practices encompass FIRST principles (Fast, Isolated, Repeatable, Self-validating, Timely), the test pyramid, behavior-focused assertions, accessible queries, explicit async handling, minimal mocking, and consistent structure (AAA/GWT). They aim to maximize confidence per test while minimizing flakiness, maintenance cost, and false positives.

**Beginner-Friendly Explanation:** Best practices are the habits that keep tests useful — test what users see, keep tests independent, wait for async things properly, and do not mock more than necessary.

### Purposes

- To keep tests fast enough to run on every change.
- To ensure failures are diagnostic and meaningful.
- To minimize brittleness so tests survive refactoring.
- To maintain a healthy balance across the test pyramid.

### Syntax Rules and Structure

**Best practices checklist:**

| Practice | Description |
|----------|-------------|
| **Follow AAA or GWT** | Consistent structure in every test. |
| **Test behavior, not implementation** | Assert on user-visible output. |
| **Query by role/label first** | Accessible queries are robust and catch a11y issues. |
| **Use `user-event`** | Realistic interactions over `fireEvent`. |
| **Always await async** | `await` user-event, `findBy*`, `waitFor`. |
| **Mock only boundaries** | Network (MSW), time (fake timers), browser APIs. |
| **Prefer fakes over mocks** | Less brittle, more realistic. |
| **Isolate tests** | Fresh state, fresh store, no order dependencies. |
| **No hard sleeps** | Use dynamic waits or fake timers. |
| **One behavior per test** | Clear failures, easy debugging. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Best-Practice-Compliant Test

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect, vi } from 'vitest';
import { LoginForm } from './LoginForm';

test('submits valid credentials', async () => {
  // Arrange
  const user = userEvent.setup();
  const onSubmit = vi.fn().mockResolvedValue(undefined);
  render(<LoginForm onSubmit={onSubmit} />);

  // Act
  await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
  await user.type(screen.getByLabelText(/password/i), 'secret');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  // Assert
  expect(onSubmit).toHaveBeenCalledWith({
    email: 'alice@example.com',
    password: 'secret',
  });
  expect(await screen.findByText('Welcome')).toBeInTheDocument();
});
```

**Why this follows best practices:** It uses AAA structure, accessible queries, `user-event`, `await` for all async operations, and asserts on user-visible outcomes.

### Real-World Cases

- **CI pipelines:** Best-practice-compliant tests run fast and reliably on every pull request.
- **Code reviews:** Best practices provide a checklist for evaluating test quality.
- **Legacy suites:** Refactoring slow, flaky, implementation-coupled tests restores trust and speed.

### References

- Semaphore — AAA (Arrange-Act-Assert) testing pattern — https://semaphore.io/blog/aaa-pattern-test-automation
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- Redux — Writing Tests: Guiding Principles — https://redux.js.org/usage/writing-tests

---

## References

- Semaphore — AAA (Arrange-Act-Assert) testing pattern — https://semaphore.io/blog/aaa-pattern-test-automation
- Martin Fowler — GivenWhenThen — https://martinfowler.com/bliki/GivenWhenThen.html
- Cucumber — Gherkin Reference — https://cucumber.io/docs/gherkin/reference/
- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- Testing Library — renderHook — https://testing-library.com/docs/react-testing-library/api#renderhook
- Testing Library — Async Methods — https://testing-library.com/docs/dom-testing-library/api-async
- Testing Library — Setup: Custom Render with providers — https://testing-library.com/docs/react-testing-library/setup
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/
- Test Async Hooks — https://raw.githubusercontent.com/djankies/claude-configs/master/skills/testing/react-hooks-testing/SKILL.md
- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- Redux Toolkit — Testing Slice Reducers — https://github.com/agents-inc/skills/blob/main/skills/src/skills/web-state-redux-toolkit/examples/testing.md
- Egghead — Unit Testing a Memoized Selector — https://egghead.io/lessons/react-unit-testing-a-memoized-selector-that-derives-data-from-multiple-reducers
- Testing TanStack Form — https://github.com/fusengine/agents/blob/main/plugins/nextjs-expert/skills/nextjs-tanstack-form/references/testing.md
- How to mock API requests in React tests — https://coreui.io/answers/how-to-mock-api-requests-in-react-tests/
- Using Mock Service Worker to Improve Jest Unit Tests — https://www.wwt.com/article/using-mock-service-worker-to-improve-jest-unit-tests
- MemoryRouter — React Router — https://reactrouter.com/6.30.3/router-components/memory-router
- Error Boundary Testing Example (HuggingFace) — https://huggingface.co/spaces/Ma-Ri-Ba-Ku/alto-llm-corrector/blob/a94837f5c02ea105429f5c9a7d7e5f6e821113f6/frontend/src/components/ErrorBoundary.test.tsx
- Jest — Snapshot Testing — https://jestjs.io/docs/snapshot-testing
- Vitest — Snapshot — https://vitest.dev/guide/snapshot
- Playwright — Visual Comparisons — https://playwright.dev/docs/test-snapshots
- Chromatic — Visual Testing — https://www.chromatic.com/features/visual-test
- jest-axe — GitHub Repository — https://github.com/nickcolley/jest-axe
- Deque — axe-core — https://github.com/dequelabs/axe-core
- Souvlaki — Composable test wrappers for context-heavy components — https://www.npmjs.com/package/souvlaki