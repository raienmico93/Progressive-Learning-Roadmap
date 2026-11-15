# Component Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Component testing is the practice of rendering an individual UI component in isolation, simulating user interactions, and asserting on its observable output to verify that it behaves correctly.

**Technical Definition:** Component testing (also called "unit testing components" in a Testing Library context) mounts a single React component — or a small, tightly-scoped tree — using a DOM testing environment (typically jsdom or happy-dom). The test supplies props and context, queries the rendered DOM using user-centric, accessibility-first selectors, simulates realistic interactions with `user-event`, and asserts on the resulting UI state. It verifies behavior — what the user sees and does — rather than internal state, lifecycle methods, or component instances. Component tests sit between pure unit tests and integration tests: they exercise the component's full render cycle without touching the network or a real browser.

**Beginner-Friendly Explanation:** A component test puts one piece of your UI on a fake screen, clicks and types on it like a real user would, and checks that the screen updates correctly. It doesn't need a browser or a server — just the component and its inputs.

### Key Characteristics

- **User-centric:** Queries reflect how users find elements (roles, labels, text), not CSS classes or `data-testid`.
- **Behavior-focused:** Asserts on rendered output and interaction outcomes, not implementation details.
- **Realistic interactions:** Uses `user-event` to simulate full browser interaction sequences, not single `dispatchEvent` calls.
- **Isolated but integrated:** Renders the component fully (no shallow rendering) while replacing external boundaries (network, time) with test doubles.
- **Accessibility-first:** If a query is hard to write, the UI is likely hard to use — component tests double as a11y smoke tests.

### Prerequisites

- A React project with a test runner (Vitest recommended for new projects in 2026; Jest for existing projects). 
- `@testing-library/react` for rendering and querying.
- `@testing-library/user-event` for realistic interactions.
- `@testing-library/jest-dom` for DOM-specific matchers (`toBeInTheDocument`, `toHaveTextContent`).
- A DOM environment (jsdom or happy-dom) configured in the test runner.

### Related Programming Areas

- **Unit testing hooks** — testing custom Hooks with `renderHook` (the complement to component testing).
- **Integration testing** — testing multiple components together with real context and routing.
- **End-to-end testing** — Playwright/Cypress tests that drive a real browser.
- **Accessibility testing** — using `getByRole` and `jest-axe` to catch a11y issues.
- **Network mocking** — Mock Service Worker (MSW) for component tests that make API calls.

### Core Concepts / Features

---

## 1. Component Testing

### Definitions

**Core Definition:** Component testing is the level of testing that verifies a single UI component's rendering, props handling, state transitions, and user interactions in isolation.

**Technical Definition:** A component test mounts the component under test using `render()` from React Testing Library, which creates a DOM container (typically appended to `document.body`) and invokes React's `createRoot` to render the component tree. The test then queries the resulting DOM with `screen`, interacts via `userEvent`, and asserts with Jest/Vitest matchers. React Testing Library's guiding principle is that "the more your tests resemble the way your software is used, the more confidence they can give you". 

**Beginner-Friendly Explanation:** Component testing means testing one component at a time — giving it some props, checking what it renders, clicking its buttons, and confirming the right things happen. It's the sweet spot between tiny unit tests and slow end-to-end tests.

### Purposes

- To verify that a component renders the correct output for given props and state.
- To confirm that user interactions (clicks, typing, focus) produce the expected UI changes.
- To catch regressions in component behavior before they reach integration or E2E tests.
- To serve as accessible, executable documentation of the component's public contract.
- To provide fast feedback (milliseconds per test) during development and in CI.

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

| Component | Description |
|-----------|-------------|
| `render(<Component />)` | Mounts the component into a jsdom/happy-dom container. |
| `screen` | Queries the entire document — preferred over destructuring from `render`, because it keeps working when the component portals outside its container. |
| `getByRole`, `getByLabelText`, `getByText` | User-centric queries, in priority order. |
| `userEvent.setup()` | Creates a user session that dispatches realistic event sequences. |
| `await user.click(...)` | Simulates a full click (focus, pointer events, click). |
| `expect(...).toBeInTheDocument()` | Asserts on the observable DOM state. |

**Syntax Rules:**

1. Import `render` and `screen` from `@testing-library/react`; use `screen` for all queries.
2. Call `userEvent.setup()` **before** `render()` and always `await` user-event calls. 
3. Follow query priority: `getByRole` → `getByLabelText` → `getByPlaceholderText` → `getByText` → `getByTestId` (last resort). 
4. Use `findBy*` for elements that appear asynchronously; use `queryBy*` to assert absence.
5. Do not test internal state, lifecycle methods, or component instances. 

**Constraints and Limitations:**

- Component tests do not verify cross-component integration or real network behavior — that is the job of integration and E2E tests.
- Snapshot tests should be avoided unless there is a clear, justified reason. 
- Shallow rendering (Enzyme-style) is an anti-pattern in the Testing Library philosophy — render the full component. 

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

**Why this output:** `user.click` dispatches a full user interaction sequence (focus, pointerdown, mousedown, pointerup, mouseup, click). The native `disabled` attribute prevents the click from reaching the handler, and `user-event` respects this browser behavior. 

#### Example 2: Testing a Form with Async Submission

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

**Expected Output:** The test passes. It types into the email field, clicks submit, waits for the success message with `findByText`, and verifies the submit handler was called.

### Real-World Cases

- **Design systems:** Every component in a design system has component tests verifying rendering, props, and interactions.
- **Form components:** Testing validation, submission, and error states without a backend.
- **Conditional UI:** Verifying that loading spinners, error messages, and empty states render correctly.

### References

- React Testing Library — Introduction — https://testing-library.com/docs/react-testing-library/intro
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- Component Testing (tartinerlabs/skills) — https://github.com/tartinerlabs/skills/blob/3a2b1ba97c4f599f5b5ab76c5a2d7914ce9b24b1/plugins/tooling/skills/testing/rules/component-testing.md

---

## 2. Rendering Components

### Definitions

**Core Definition:** Rendering a component in a test means mounting it into a simulated DOM environment so it can be queried and interacted with.

**Technical Definition:** `render(ui, options?)` from React Testing Library creates a container element, appends it to `document.body`, and uses React's `createRoot` to render the provided React element. It returns an object containing `container`, `baseElement`, `debug()`, `rerender()`, `unmount()`, and `asFragment()`. Automatic cleanup runs after each test (when the test framework's `afterEach` global is available), removing the rendered tree from the DOM.

**Beginner-Friendly Explanation:** Rendering a component in a test means putting it on a simulated screen so you can look at it and interact with it. The `render()` function does this for you.

### Purposes

- To mount the component under test into a DOM environment for querying and interaction.
- To provide a fresh, isolated container for each test.
- To enable re-rendering with new props via `rerender()`.
- To enable unmount testing via `unmount()`.

### Syntax Rules and Structure

```jsx
import { render, screen } from '@testing-library/react';

const { container, rerender, unmount } = render(<Component prop="initial" />);
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| `container` | The DOM node containing the rendered component. |
| `baseElement` | The document body (default); useful for portals. |
| `rerender(ui)` | Re-renders the component with new props/elements. |
| `unmount()` | Unmounts the component to test cleanup. |
| `debug()` | Pretty-prints the DOM for debugging. |
| `asFragment()` | Returns the rendered output as a `DocumentFragment`. |

**Syntax Rules:**

1. Prefer `screen` over destructuring queries from `render` — it queries the whole document and survives portals. 
2. React Testing Library automatically cleans up after each test — do not call manual `cleanup()`. 
3. Use `rerender()` to test prop changes without unmounting.
4. Use `unmount()` to verify that cleanup functions (timers, listeners) run correctly. 

**Constraints and Limitations:**

- `container` queries are scoped to the component's container and will not see portal content — use `baseElement` or `screen` instead. 
- Do not assert on `container.innerHTML` — query the DOM with accessible queries.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Rendering and Re-rendering with Props

```jsx
import { render, screen } from '@testing-library/react';
import { test, expect } from 'vitest';
import { Greeting } from './Greeting.jsx';

test('updates greeting when name prop changes', () => {
  const { rerender } = render(<Greeting name="Alice" />);
  expect(screen.getByText('Hello, Alice!')).toBeInTheDocument();

  rerender(<Greeting name="Bob" />);
  expect(screen.getByText('Hello, Bob!')).toBeInTheDocument();
});
```

**Expected Output:** Both assertions pass. The component initially renders "Hello, Alice!", then "Hello, Bob!" after re-rendering with a new prop.

### Real-World Cases

- **Prop-change testing:** Verifying that a component correctly re-renders when its props change.
- **Unmount cleanup:** Testing that timers and event listeners are removed when a component unmounts.

### References

- React Testing Library — render — https://testing-library.com/docs/react-testing-library/api#render
- React Testing Library — Setup (automatic cleanup) — https://testing-library.com/docs/react-testing-library/setup

---

## 3. Querying Elements

### Definitions

**Core Definition:** Querying elements means locating DOM nodes in the rendered output using Testing Library's query functions, prioritizing user-visible attributes.

**Technical Definition:** Testing Library provides three query families — `getBy*` (throws if not found or found multiple times), `queryBy*` (returns `null` if not found, throws if multiple), and `findBy*` (returns a Promise, retrying until the element appears or times out). Each family has singular and plural (`getAllBy*`, `queryAllBy*`, `findAllBy*`) variants. Queries are prioritized by accessibility: `getByRole` first, then `getByLabelText`, `getByPlaceholderText`, `getByText`, and `getByTestId` only as a last resort. 

**Beginner-Friendly Explanation:** Querying is how you find the element you want to interact with or check. You should find elements the same way a user would — by their role (button, heading) or label, not by CSS class.

### Purposes

- To locate elements in the rendered output for interaction or assertion.
- To encourage accessible markup by prioritizing role and label queries.
- To distinguish between "element must exist" (`getBy`), "element may not exist" (`queryBy`), and "element will appear" (`findBy`) scenarios.
- To write tests that survive refactoring of CSS classes and internal markup.

### Syntax Rules and Structure

**Query families:**

| Family | Not found | Found multiple | Async? |
|--------|-----------|----------------|--------|
| `getBy*` | Throws | Throws | No |
| `queryBy*` | Returns `null` | Throws | No |
| `findBy*` | Rejects (after timeout) | Rejects | Yes (returns Promise) |

**Query priority (highest to lowest):**

1. `getByRole` — accessible roles (button, heading, textbox, link)
2. `getByLabelText` — form fields with labels
3. `getByPlaceholderText` — inputs without labels
4. `getByText` — non-interactive text content
5. `getByTestId` — last resort only

**Component Breakdown:**

| Query | Use case |
|-------|----------|
| `getByRole('button', { name: /submit/i })` | Find a button by its accessible name. |
| `getByLabelText(/email/i)` | Find an input by its associated label. |
| `queryByText('Error')` | Assert that an error message is absent. |
| `findByText('Loaded')` | Wait for async content to appear. |
| `within(element).getByRole(...)` | Scope a query to a subtree. |

**Syntax Rules:**

1. Always prefer `getByRole` — it is the most accessible and robust query. 
2. Use `queryBy*` only to assert absence (`expect(...).not.toBeInTheDocument()`).
3. Use `findBy*` for async content; always `await` it. 
4. Use `within()` to scope queries to a specific container when multiple similar elements exist.

**Constraints and Limitations:**

- `getByTestId` should be rare — if you need it frequently, the UI may lack accessible roles or labels. 
- `container.querySelector()` is an anti-pattern in Testing Library — it couples tests to markup structure.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Query Priority in Practice

```jsx
// ❌ BAD: CSS selector query
const { container } = render(<LoginForm />);
const input = container.querySelector('input[name="email"]');

// ✅ GOOD: accessible queries
render(<LoginForm />);
const input = screen.getByRole('textbox', { name: /email/i });
const button = screen.getByRole('button', { name: /sign in/i });
```

**Why this matters:** The CSS selector breaks if the input is wrapped in a custom component that changes its tag. `getByRole` finds the element by its accessible semantics, which remain stable as long as the UI remains usable.

#### Example 2: Using `within` and `queryBy`

```jsx
import { render, screen, within } from '@testing-library/react';

test('shows an error and no success message on failure', () => {
  render(<Form status="error" />);

  const alert = screen.getByRole('alert');
  expect(within(alert).getByText('Invalid email')).toBeInTheDocument();
  expect(screen.queryByText('Success')).not.toBeInTheDocument();
});
```

**Expected Output:** The test passes. It finds the alert element, scopes a text query inside it with `within`, and asserts that the success message is absent with `queryBy`.

### Real-World Cases

- **Accessibility auditing:** `getByRole` queries double as a11y checks — if you cannot find an element by its role, screen reader users may not find it either.
- **Form testing:** `getByLabelText` ensures every input has a properly associated label.
- **Conditional rendering:** `queryBy` for absence, `findBy` for async appearance.

### References

- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- Testing Library — getBy, queryBy, findBy — https://testing-library.com/docs/queries/about

---

## 4. Firing Events

### Definitions

**Core Definition:** Firing events means simulating user interactions — clicks, typing, keyboard presses, focus — on rendered elements.

**Technical Definition:** Testing Library offers two interaction APIs: `fireEvent` (a lightweight wrapper around `dispatchEvent` that fires a single DOM event) and `user-event` (a higher-fidelity simulator that dispatches the full sequence of events a real browser produces for an interaction, including focus, pointer events, and value manipulation). For example, `fireEvent.click` fires only a `click` event, while `userEvent.click` fires focus, pointerdown, mousedown, pointerup, mouseup, and click, and refuses to click hidden or disabled elements. `userEvent.setup()` must be called before rendering, and all interactions must be awaited. 

**Beginner-Friendly Explanation:** Firing events is how you simulate a user clicking or typing. Use `user-event` — it behaves like a real browser, so it catches bugs that a single synthetic click would miss.

### Purposes

- To simulate realistic user interactions that exercise the component's event handlers.
- To catch bugs that only appear with full event sequences (e.g., keyboard handlers, focus management).
- To verify that disabled or hidden elements cannot be interacted with.
- To test form input, submission, and validation flows.

### Syntax Rules and Structure

```jsx
import userEvent from '@testing-library/user-event';

const user = userEvent.setup();
render(<Component />);

await user.click(screen.getByRole('button', { name: /submit/i }));
await user.type(screen.getByRole('textbox', { name: /email/i }), 'alice@example.com');
await user.keyboard('{Enter}');
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `userEvent.setup()` | Creates a user session; call before render. |
| `user.click(element)` | Full click sequence. |
| `user.type(element, text)` | Types text character by character, firing keyboard and input events. |
| `user.keyboard(keys)` | Fires keyboard events (e.g., `'{Enter}'`, `'{Shift>}a{/Shift}'`). |
| `user.tab()` | Moves focus to the next focusable element. |
| `user.hover(element)` | Simulates hover. |

**Syntax Rules:**

1. Always `await` user-event calls. 
2. Call `userEvent.setup()` before `render()`. 
3. Prefer `user-event` over `fireEvent` in almost all cases. Use `fireEvent` only for low-level events that `user-event` does not model (e.g., a raw `scroll` event). 
4. Do not wrap `user-event` calls in manual `act()` — React Testing Library handles it automatically. 

**Constraints and Limitations:**

- `fireEvent` is still needed for events `user-event` does not support (e.g., `scroll`, `load`). 
- `user-event` adds visibility and interactability checks — a test may fail if the element is not actually user-accessible, which is usually the correct behavior.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: `fireEvent` vs. `userEvent` — The Fidelity Difference

```jsx
// fireEvent.click fires ONLY a click event
fireEvent.click(button);

// userEvent.click fires focus, pointerdown, mousedown, pointerup, mouseup, click
await user.click(button);
```

**Why `userEvent` is better:** If the component relies on `onMouseDown` or focus events, `fireEvent.click` will miss those handlers. `userEvent.click` exercises the full interaction, catching real bugs. 

#### Example 2: Typing and Keyboard Interaction

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { SearchBox } from './SearchBox.jsx';

test('searches on Enter key', async () => {
  const user = userEvent.setup();
  const onSearch = vi.fn();

  render(<SearchBox onSearch={onSearch} />);

  await user.type(screen.getByRole('searchbox'), 'react');
  await user.keyboard('{Enter}');

  expect(onSearch).toHaveBeenCalledWith('react');
});
```

**Expected Output:** The test passes. `user.type` fires character-by-character keyboard and input events, and `user.keyboard('{Enter}')` triggers the Enter handler.

### Real-World Cases

- **Form submission:** Testing that clicking submit or pressing Enter calls the submit handler.
- **Keyboard navigation:** Testing tab order, Escape-to-close, and arrow-key navigation.
- **Disabled states:** Verifying that `user-event` refuses to interact with disabled or hidden elements (which is the correct browser behavior).

### References

- Testing Library — user-event Introduction — https://testing-library.com/docs/user-event/intro
- Testing Library — fireEvent vs user-event — https://testing-library.com/docs/user-event/intro#differences-from-fireevent
- React Testing Library Best Practices (gohypergiant) — https://github.com/gohypergiant/standard-toolkit/blob/88ef844bf587a9fd43f75ee0503dc471569a2cd1/.agents/skills/accelint-react-testing/README.md

---

## 5. Asserting Output

### Definitions

**Core Definition:** Asserting output means using matchers to verify that the rendered DOM matches the expected state after rendering or interaction.

**Technical Definition:** Assertions combine Jest/Vitest `expect` with `@testing-library/jest-dom` matchers (e.g., `toBeInTheDocument`, `toHaveTextContent`, `toBeDisabled`, `toHaveValue`, `toHaveAttribute`) or with query-based assertions (`expect(screen.getByText(...)).toBeInTheDocument()`). Assertions should be **user-visible** — text content, roles, accessible names, ARIA states — not implementation details like internal state or CSS classes. Async assertions use `waitFor` or `findBy*` with `await`. 

**Beginner-Friendly Explanation:** Asserting output means checking that what the user sees is correct — the right text, the right button state, the right error message. You check the screen, not the component's private variables.

### Purposes

- To verify that the rendered output matches the expected UI after rendering or interaction.
- To assert on accessibility attributes (roles, labels, `aria-*`).
- To confirm that conditional UI (errors, loading states, empty states) appears correctly.
- To provide a readable, executable specification of the component's behavior.

### Syntax Rules and Structure

```jsx
// DOM presence
expect(screen.getByText('Hello')).toBeInTheDocument();

// Absence
expect(screen.queryByText('Error')).not.toBeInTheDocument();

// Async appearance
expect(await screen.findByText('Loaded')).toBeInTheDocument();

// Attribute / state
expect(screen.getByRole('button', { name: /submit/i })).toBeDisabled();
expect(screen.getByRole('textbox')).toHaveValue('alice');
expect(screen.getByRole('checkbox')).toBeChecked();
```

**Component Breakdown:**

| Matcher | Assertion |
|---------|-----------|
| `toBeInTheDocument()` | Element is present in the DOM. |
| `toHaveTextContent(text)` | Element contains the given text. |
| `toBeDisabled()` / `toBeEnabled()` | Form control state. |
| `toHaveValue(value)` | Input/textarea value. |
| `toBeChecked()` | Checkbox/radio state. |
| `toHaveAttribute(name, value)` | Attribute value. |
| `toHaveAccessibleName(name)` | Computed accessible name. |

**Syntax Rules:**

1. Use `@testing-library/jest-dom` matchers for DOM assertions. 
2. Assert on user-visible output (text, roles, ARIA states), not CSS classes or internal variables. 
3. Use `await expect(...).resolves` or `findBy*` for async assertions.
4. Use `queryBy*` for absence assertions — `getBy*` throws when the element is missing. 

**Constraints and Limitations:**

- `toHaveTextContent` is substring-based — use exact matching when necessary.
- Do not assert on `container.innerHTML` or `container.firstChild` — these are implementation details.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Asserting Conditional Rendering

```jsx
import { render, screen } from '@testing-library/react';
import { Alert } from './Alert.jsx';

test('shows the error message when variant is error', () => {
  render(<Alert variant="error" message="Invalid email" />);

  const alert = screen.getByRole('alert');
  expect(alert).toHaveTextContent('Invalid email');
  expect(alert).toHaveClass('alert-error'); // Only if the class is part of the public API
});

test('does not render an alert for the info variant', () => {
  render(<Alert variant="info" message="Saved" />);
  expect(screen.queryByRole('alert')).not.toBeInTheDocument();
});
```

**Expected Output:** Both tests pass. The error variant renders an alert with the message; the info variant does not render an alert role.

#### Example 2: Async Assertion with `findBy`

```jsx
import { render, screen } from '@testing-library/react';

test('displays the loaded user name', async () => {
  render(<UserProfile userId="1" />);

  expect(screen.getByText('Loading...')).toBeInTheDocument();
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The loading state is visible immediately, and `findByText` waits for the async update to render "Alice".

### Real-World Cases

- **Error states:** Asserting that error messages appear in an alert role.
- **Loading states:** Asserting that a spinner or skeleton is visible during loading.
- **Form validation:** Asserting that inputs are marked invalid or that error text is present.
- **Disabled states:** Asserting that a submit button is disabled while submitting.

### References

- Testing Library — Jest DOM matchers — https://github.com/testing-library/jest-dom
- React Testing Library Best Practices (gohypergiant) — https://github.com/gohypergiant/standard-toolkit/blob/88ef844bf587a9fd43f75ee0503dc471569a2cd1/.agents/skills/accelint-react-testing/README.md

---

## 6. Testing Props

### Definitions

**Core Definition:** Testing props means rendering a component with different prop values and asserting that the rendered output correctly reflects each set of props.

**Technical Definition:** Because React components are functions of props, testing props is equivalent to testing the function's input-to-output mapping: `render(<Component prop={value} />)` followed by assertions on the rendered DOM. Prop testing covers required props, optional props with defaults, conditional rendering based on prop values, and callback props (verified with `vi.fn()`/`jest.fn()`). It does not test internal state or implementation.

**Beginner-Friendly Explanation:** Testing props means checking that when you give a component different inputs, it shows the right output. For example, if you pass `variant="error"`, does it render a red error box?

### Purposes

- To verify that a component renders correctly for each meaningful combination of props.
- To confirm that optional props use the correct defaults.
- To verify that callback props are called with the correct arguments.
- To document the component's public API contract.

### Syntax Rules and Structure

```jsx
render(<Component requiredProp="value" optionalProp={true} />);
expect(screen.getByText('Expected output')).toBeInTheDocument();
```

**Component Breakdown:**

| Prop type | Test approach |
|-----------|---------------|
| Required string/number | Render with the value; assert on rendered text. |
| Optional with default | Render without the prop; assert that the default output appears. |
| Conditional (variant, size) | Render with each variant; assert on the corresponding output. |
| Callback (`onClick`, `onChange`) | Pass a `vi.fn()`; interact; assert the call count and arguments. |

**Constraints and Limitations:**

- Do not test TypeScript prop types at runtime — that is TypeScript's job.
- Do not assert on prop values directly (e.g., `component.props.variant`) — assert on the rendered output.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Variants and Defaults

```jsx
// Badge.jsx
export function Badge({ label, variant = 'default' }) {
  return <span className={`badge badge-${variant}`}>{label}</span>;
}

// Badge.test.jsx
import { render, screen } from '@testing-library/react';
import { test, expect } from 'vitest';
import { Badge } from './Badge.jsx';

test('renders the label', () => {
  render(<Badge label="New" />);
  expect(screen.getByText('New')).toBeInTheDocument();
});

test('uses the default variant when none is provided', () => {
  render(<Badge label="New" />);
  expect(screen.getByText('New')).toHaveClass('badge-default');
});

test('renders the specified variant', () => {
  render(<Badge label="Sale" variant="success" />);
  expect(screen.getByText('Sale')).toHaveClass('badge-success');
});
```

**Expected Output:** All three tests pass. The label renders, the default variant applies when omitted, and the specified variant applies when provided.

### Real-World Cases

- **Design system variants:** Testing that `variant`, `size`, and `color` props produce the correct classes and styles.
- **Conditional rendering:** Testing that a `loading` prop shows a spinner and a `data` prop shows content.
- **Callback props:** Testing that `onChange`, `onSubmit`, and `onClose` are called with the expected arguments.

### References

- React Testing Library — Component Testing Patterns — https://github.com/mintuz/skills/blob/main/data/skills-md/mintuz/claude-plugins/react-testing/SKILL.md
- Steve Kinney — Testing React Components with TypeScript — https://stevekinney.com/courses/react-typescript/testing-react-typescript

---

## 7. Testing State

### Definitions

**Core Definition:** Testing state means verifying that a component's internal state changes produce the correct UI updates, without directly inspecting the state variable itself.

**Technical Definition:** In React Testing Library, state is tested **indirectly** — by interacting with the component (via `user-event`) and asserting on the resulting DOM. For example, clicking an "increment" button should update the displayed count. The internal `useState` value is never accessed directly; doing so would couple the test to implementation details. For complex state logic extracted into reducers or custom Hooks, those units are tested separately (reducer tests, `renderHook` tests).

**Beginner-Friendly Explanation:** You do not test state directly — you test what state produces. Click the button, then check that the screen shows the new count. That is how you verify state works.

### Purposes

- To verify that user interactions trigger the correct state transitions and UI updates.
- To ensure that state updates are reflected in the rendered output.
- To catch bugs where state changes but the UI does not re-render correctly.
- To avoid brittle tests that break when internal state variable names change.

### Syntax Rules and Structure

```jsx
const user = userEvent.setup();
render(<Counter />);

await user.click(screen.getByRole('button', { name: /increment/i }));
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

**Component Breakdown:**

| Step | Description |
|------|-------------|
| Render | Mount the component in its initial state. |
| Interact | Use `user-event` to trigger the state change. |
| Assert | Verify the UI reflects the new state. |

**Constraints and Limitations:**

- Never access `component.state`, `result.current`, or internal variables in a component test. 
- State updates that cause async effects require `findBy*` or `waitFor`.
- State logic that is complex should be extracted into a reducer or custom Hook and tested separately.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Counter Component's State via UI

```jsx
// Counter.jsx
export function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </div>
  );
}

// Counter.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect } from 'vitest';
import { Counter } from './Counter.jsx';

test('increments the displayed count when clicked', async () => {
  const user = userEvent.setup();
  render(<Counter />);

  expect(screen.getByText('Count: 0')).toBeInTheDocument();

  await user.click(screen.getByRole('button', { name: /increment/i }));

  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The initial count is 0; after clicking, the displayed count is 1.

**Why this output:** The test interacts with the component as a user would and asserts on the visible output — never on the internal `count` variable.

### Real-World Cases

- **Toggle buttons:** Verifying that clicking a dark-mode toggle changes the displayed theme.
- **Accordions:** Verifying that clicking a header expands the panel.
- **Multi-step forms:** Verifying that clicking "Next" advances to the next step.

### References

- React Testing Library — Core Principles: Test behavior, not implementation — https://github.com/mintuz/skills/blob/main/data/skills-md/mintuz/claude-plugins/react-testing/SKILL.md
- Component Testing (ledgerhq/lumen) — https://github.com/ledgerhq/lumen/blob/1d4929f3eb762cce283c87eea0f3ebc1f74dc8ba/.claude/skills/component-testing/SKILL.md

---

## 8. Testing Context

### Definitions

**Core Definition:** Testing context means providing a component with the Context values it consumes — via a wrapper component — and asserting that it renders correctly for different context states.

**Technical Definition:** A component that calls `useContext` requires a matching Provider ancestor. In tests, you wrap the component with the Provider using either the `wrapper` option of `render()` or a custom render function that includes all global providers. For each test, you supply the specific context value you want to exercise — e.g., an authenticated user, a theme, or a feature flag. Do not mock `useContext` directly; provide the real provider with controlled values. 

**Beginner-Friendly Explanation:** If your component needs data from a Context (like the current user or theme), you have to wrap it in the corresponding Provider when you test it. You give the Provider a fake value and check that the component uses it correctly.

### Purposes

- To test components that depend on React Context without needing the real app's provider tree.
- To verify that the component renders correctly for different context values (e.g., admin vs. viewer, dark vs. light theme).
- To keep tests focused by providing only the context values the component actually needs.
- To enable reusable, composable test wrappers for context-heavy components.

### Syntax Rules and Structure

**Option 1: `wrapper` option:**

```jsx
render(<Component />, {
  wrapper: ({ children }) => (
    <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
  ),
});
```

**Option 2: Custom render function:**

```jsx
// test-utils.jsx
import { render } from '@testing-library/react';
import { ThemeProvider } from './ThemeContext';

function AllProviders({ children }) {
  return <ThemeProvider value="light">{children}</ThemeProvider>;
}

export function renderWithProviders(ui, options) {
  return render(ui, { wrapper: AllProviders, ...options });
}
```

**Component Breakdown:**

| Approach | Description |
|----------|-------------|
| `wrapper` option | Pass a wrapper component to `render()`; it wraps the UI. |
| Custom render | A `test-utils` file that re-exports everything from RTL and overrides `render` with the providers baked in. |
| Composable wrappers | Build small wrapper helpers (e.g., `withTheme`, `withAuth`) and compose them per test. |

**Syntax Rules:**

1. Wrap the component in the real Provider — do not mock `useContext`. 
2. Provide only the context values the component needs for the specific test.
3. Use a custom `render` from `test-utils` to avoid repeating provider setup in every test. 
4. For multiple providers, nest them in the wrapper — order matters if providers depend on each other.

**Constraints and Limitations:**

- The `wrapper` option only wraps the rendered UI — it does not apply to `renderHook` unless passed explicitly.
- A single "mega-wrapper" can become unwieldy — prefer composable per-context wrappers. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Context-Consuming Component

```jsx
// UserBadge.jsx
import { useContext } from 'react';
import { UserContext } from './UserContext';

export function UserBadge() {
  const user = useContext(UserContext);
  return <span>{user ? user.name : 'Guest'}</span>;
}

// UserBadge.test.jsx
import { render, screen } from '@testing-library/react';
import { UserContext } from './UserContext';
import { UserBadge } from './UserBadge.jsx';

function renderWithUser(user) {
  return render(
    <UserContext.Provider value={user}>
      <UserBadge />
    </UserContext.Provider>
  );
}

test('shows the user name when authenticated', () => {
  renderWithUser({ name: 'Alice' });
  expect(screen.getByText('Alice')).toBeInTheDocument();
});

test('shows "Guest" when no user is provided', () => {
  renderWithUser(null);
  expect(screen.getByText('Guest')).toBeInTheDocument();
});
```

**Expected Output:** Both tests pass. The component reads the context value and renders the correct name or fallback.

### Real-World Cases

- **Authentication context:** Testing that protected UI renders only for authenticated users.
- **Theme context:** Testing that components render with the correct theme tokens.
- **Feature flags:** Testing that feature-gated UI appears only when the flag context value is enabled.

### References

- Testing Library — Setup: Custom Render with providers — https://testing-library.com/docs/react-testing-library/setup
- Souvlaki — Composable React test wrappers for context-heavy components — https://www.npmjs.com/package/souvlaki
- React Testing Library: Advanced Patterns for Complex UIs — https://codeloomdevv.co.in/blog/react/react-testing-library-advanced-patterns

---

## 9. Testing Portals

### Definitions

**Core Definition:** Testing portals means rendering a component that uses `ReactDOM.createPortal` and asserting on the content that is rendered outside the component's normal DOM hierarchy.

**Technical Definition:** `createPortal` renders children into a different DOM node (typically a sibling of the app root, like `document.body` or a dedicated `#modal-root`). By default, React Testing Library renders into a container inside `document.body`, so portal content may end up outside the `container`. The solution is to use `screen` (which queries the entire `document.body`) or the `baseElement` returned by `render()`, rather than `container.querySelector()`. The portal's target DOM node must exist in the test environment — create it in a `beforeEach` or in the test setup file. 

**Beginner-Friendly Explanation:** A portal renders content somewhere else in the DOM — like a modal that is attached to `<body>` instead of inside its parent component. To test it, you have to look at the whole page, not just the component's container.

### Purposes

- To verify that portal-rendered content (modals, tooltips, dropdowns) appears correctly.
- To test the interaction and focus behavior of portal-based UI.
- To ensure that portal content is accessible and queryable in tests.

### Syntax Rules and Structure

```jsx
// Ensure the portal target exists
beforeEach(() => {
  const portalRoot = document.createElement('div');
  portalRoot.setAttribute('id', 'portal-root');
  document.body.appendChild(portalRoot);
});

afterEach(() => {
  document.getElementById('portal-root')?.remove();
});

// Render the component
render(<Modal isOpen={true} />);

// Query with screen (queries the whole document, including portals)
expect(screen.getByRole('dialog')).toBeInTheDocument();
```

**Component Breakdown:**

| Concern | Solution |
|---------|----------|
| Portal target missing | Create the target node in `beforeEach` or test setup. |
| Content outside `container` | Use `screen` or `baseElement` instead of `container`. |
| Cleanup | Remove the portal target in `afterEach`. |
| Focus management | Use `user.tab()` and assert on `document.activeElement`. |

**Constraints and Limitations:**

- `container.querySelector()` will not find portal content — this is the most common portal testing mistake. 
- If the portal target is not created before render, the portal will fail silently or throw. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Modal Rendered Through a Portal

```jsx
// Modal.jsx
import { createPortal } from 'react-dom';

export function Modal({ isOpen, children }) {
  if (!isOpen) return null;
  return createPortal(
    <div role="dialog" aria-modal="true">{children}</div>,
    document.getElementById('modal-root')
  );
}

// Modal.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { beforeEach, afterEach, test, expect } from 'vitest';
import { Modal } from './Modal.jsx';

beforeEach(() => {
  const root = document.createElement('div');
  root.id = 'modal-root';
  document.body.appendChild(root);
});

afterEach(() => {
  document.getElementById('modal-root')?.remove();
});

test('renders modal content in a portal and closes on Escape', async () => {
  const user = userEvent.setup();
  const onClose = vi.fn();

  render(<Modal isOpen={true} onClose={onClose}><p>Confirm?</p></Modal>);

  const dialog = screen.getByRole('dialog');
  expect(dialog).toHaveTextContent('Confirm?');

  await user.keyboard('{Escape}');
  // If the modal listens for Escape, assert onClose was called
});
```

**Expected Output:** The test passes. The modal content is found via `screen.getByRole('dialog')` even though it is rendered into `#modal-root` outside the component's container.

### Real-World Cases

- **Modals and dialogs:** Testing that modal content renders, traps focus, and closes on Escape.
- **Dropdown menus:** Testing that menu items rendered in a portal are clickable.
- **Tooltips and popovers:** Testing that tooltip content appears on hover.

### References

- Mantine — Testing portals with React Testing Library — https://github.com/mantinedev/mantine/blob/3646de9f/apps/help.mantine.dev/src/pages/q/portals-testing.mdx
- React Testing Library — Advanced Patterns: Portals — https://codeloomdevv.co.in/blog/react/react-testing-library-advanced-patterns

---

## 10. Testing Error Boundaries

### Definitions

**Core Definition:** Testing an error boundary means rendering a component that throws an error inside an Error Boundary and asserting that the boundary's fallback UI appears instead of the crashed component.

**Technical Definition:** React Error Boundaries are class components that implement `componentDidCatch` and/or `getDerivedStateFromError`. To test them, render a child component that throws (either unconditionally or conditionally) inside the boundary, and assert that the fallback UI is rendered. React logs the caught error to `console.error` — silence this in the test with `vi.spyOn(console, 'error').mockImplementation(() => {})` to keep test output clean. Error boundaries do not catch errors in event handlers, async code, or server-side rendering — those require separate handling. 

**Beginner-Friendly Explanation:** An Error Boundary catches crashes in its child components and shows a fallback UI instead of a broken page. To test it, you deliberately make a child component throw, then check that the fallback appears.

### Purposes

- To verify that the Error Boundary catches render errors and displays the correct fallback.
- To test the fallback UI (error message, retry button) renders and behaves correctly.
- To ensure that children render normally when no error occurs.
- To verify that errors in event handlers or async code are not caught by the boundary (they require different handling).

### Syntax Rules and Structure

```jsx
// Suppress console.error noise from React
beforeEach(() => {
  vi.spyOn(console, 'error').mockImplementation(() => {});
});
afterEach(() => {
  vi.restoreAllMocks();
});

// A component that throws
function Bomb() {
  throw new Error('kaboom');
}

// Render inside the boundary
render(
  <ErrorBoundary fallback={<p>Something went wrong</p>}>
    <Bomb />
  </ErrorBoundary>
);

expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
```

**Component Breakdown:**

| Concern | Solution |
|---------|----------|
| Error noise | Spy on `console.error` and mock it. |
| Triggering an error | Render a child that throws during render. |
| Fallback assertion | Query for the fallback text. |
| Retry behavior | Reset the throwing condition and click "Try again". |

**Constraints and Limitations:**

- Error boundaries only catch errors during rendering, in lifecycle methods, and in constructors — not in event handlers, async code, or the boundary itself. 
- React 19 may log additional error information — suppress `console.error` in the test.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Error Boundary Fallback and Recovery

```jsx
// ErrorBoundary.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { beforeEach, afterEach, test, expect, vi } from 'vitest';
import { ErrorBoundary } from './ErrorBoundary';

let shouldThrow = true;

function Bomb() {
  if (shouldThrow) throw new Error('kaboom in render');
  return <div>recovered content</div>;
}

beforeEach(() => {
  vi.spyOn(console, 'error').mockImplementation(() => {});
});
afterEach(() => {
  vi.restoreAllMocks();
});

test('renders children when nothing throws', () => {
  render(
    <ErrorBoundary>
      <div>all good</div>
    </ErrorBoundary>
  );
  expect(screen.getByText('all good')).toBeInTheDocument();
});

test('catches a render error and shows the message', () => {
  shouldThrow = true;
  render(
    <ErrorBoundary>
      <Bomb />
    </ErrorBoundary>
  );
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
  expect(screen.getByText('kaboom in render')).toBeInTheDocument();
  expect(console.error).toHaveBeenCalled();
});

test('re-renders children after "Try again"', async () => {
  const user = userEvent.setup();
  shouldThrow = true;
  render(
    <ErrorBoundary>
      <Bomb />
    </ErrorBoundary>
  );
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();

  shouldThrow = false;
  await user.click(screen.getByRole('button', { name: /try again/i }));

  expect(screen.getByText('recovered content')).toBeInTheDocument();
  expect(screen.queryByText(/something went wrong/i)).not.toBeInTheDocument();
});
```

**Expected Output:** All three tests pass. The boundary renders children normally, catches the render error and shows the fallback, and recovers after the retry button is clicked.

### Real-World Cases

- **Route-level error boundaries:** Testing that a failed route displays a friendly error page.
- **Widget-level boundaries:** Testing that one broken widget does not crash the entire dashboard.
- **Retry flows:** Testing that the fallback's retry button re-renders the children successfully.

### References

- Error Boundary Testing Example (HuggingFace) — https://huggingface.co/spaces/Ma-Ri-Ba-Ku/alto-llm-corrector/blob/a94837f5c02ea105429f5c9a7d7e5f6e821113f6/frontend/src/components/ErrorBoundary.test.tsx
- React Testing Library: Advanced Patterns — Error Boundaries — https://codeloomdevv.co.in/blog/react/react-testing-library-advanced-patterns

---

## 11. Component Testing Best Practices

### Definitions

**Core Definition:** Component testing best practices are the guidelines that make component tests fast, reliable, maintainable, and aligned with the Testing Library philosophy of user-centric testing.

**Technical Definition:** Best practices encompass query priority (accessibility first), realistic interaction simulation (`user-event` over `fireEvent`), explicit async handling (`await` + `findBy*`/`waitFor`), custom render functions for context providers, avoidance of implementation-detail assertions (no state inspection, no shallow rendering, no snapshot tests without justification), and automatic cleanup reliance (no manual `act()` or `cleanup()`). They aim to maximize test confidence while minimizing brittleness and false positives. 

**Beginner-Friendly Explanation:** Best practices are the habits that keep component tests useful — test what users see, interact realistically, wait for async things properly, and don't peek at the component's private parts.

### Purposes

- To ensure tests fail for the right reason and pass for the right reason.
- To minimize brittleness so tests survive refactoring of styles and internal structure.
- To maximize accessibility coverage by testing through accessible roles and labels.
- To keep tests fast and deterministic with proper async handling and no manual cleanup.

### Syntax Rules and Structure

**Best practices checklist:**

| Practice | Why |
|----------|-----|
| **Query by role first** | Most accessible and robust; catches a11y issues. |
| **Use `screen` for all queries** | Queries the whole document; works with portals. |
| **Prefer `userEvent` over `fireEvent`** | Simulates real browser interaction sequences. |
| **Always `await` user-event and async queries** | Prevents false greens. |
| **Use `findBy*` for async content** | Retries until the element appears. |
| **Use `queryBy*` to assert absence** | `getBy*` throws when absent. |
| **Create a custom `render` with providers** | Avoids repeating context setup. |
| **Do not test internal state** | Test the UI, not the implementation. |
| **Do not use shallow rendering** | Render the full component. |
| **No manual `act()` or `cleanup()`** | React Testing Library handles it automatically. |
| **No snapshot tests without justification** | Snapshots are brittle and hide regressions. |
| **Structure with `describe`/`it`** | Group by concern (Rendering, Interactions, States). |

**Constraints and Limitations:**

- Over-mocking context values produces tests that pass even when the provider is broken — prefer real providers with controlled values. 
- Accessibility queries can reveal real a11y problems — do not bypass them with `getByTestId` just to make the test pass. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: A Fully Best-Practice-Compliant Test

```jsx
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { ContactForm } from './ContactForm.jsx';

describe('ContactForm', () => {
  it('submits the form with user input', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();

    render(<ContactForm onSubmit={onSubmit} />);

    // Query by accessible labels
    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/message/i), 'Hello world');

    // Interact realistically
    await user.click(screen.getByRole('button', { name: /submit/i }));

    // Assert on user-visible outcomes
    expect(await screen.findByText(/thank you/i)).toBeInTheDocument();
    expect(onSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      message: 'Hello world',
    });
  });
});
```

**Why this follows best practices:**

- Uses `userEvent.setup()` and `await` for all interactions.
- Queries by accessible labels and roles — no `data-testid` or CSS selectors.
- Uses `findByText` for the async success message.
- Asserts on user-visible output and callback arguments, not internal state.
- No manual `act()` or `cleanup()` — React Testing Library handles them. 

#### Example 2: Custom Render with Providers (test-utils pattern)

```jsx
// test-utils.jsx
import { render } from '@testing-library/react';
import { ThemeProvider } from './ThemeContext';
import { AuthProvider } from './AuthContext';

function AllProviders({ children }) {
  return (
    <ThemeProvider value="light">
      <AuthProvider initialUser={null}>{children}</AuthProvider>
    </ThemeProvider>
  );
}

export function renderWithProviders(ui, options) {
  return render(ui, { wrapper: AllProviders, ...options });
}

export * from '@testing-library/react';

// Component.test.jsx
import { renderWithProviders, screen } from './test-utils';
import { Dashboard } from './Dashboard';

test('shows the login prompt for unauthenticated users', () => {
  renderWithProviders(<Dashboard />);
  expect(screen.getByText(/please log in/i)).toBeInTheDocument();
});
```

**Why this follows best practices:** The custom `render` centralizes provider setup, so individual tests stay focused on behavior. Tests import from `test-utils` instead of `@testing-library/react` directly.

### Real-World Cases

- **Design systems:** Enforce query priority and `userEvent` usage via ESLint rules and code review checklists.
- **CI pipelines:** Best-practice-compliant tests run fast and reliably, catching regressions without flaking.
- **Legacy migrations:** Moving from Enzyme/shallow rendering to React Testing Library requires adopting these practices wholesale.

### References

- React Testing Library Best Practices (gohypergiant) — https://github.com/gohypergiant/standard-toolkit/blob/88ef844bf587a9fd43f75ee0503dc471569a2cd1/.agents/skills/accelint-react-testing/README.md
- Component Testing (ledgerhq/lumen) — https://github.com/ledgerhq/lumen/blob/1d4929f3eb762cce283c87eea0f3ebc1f74dc8ba/.claude/skills/component-testing/SKILL.md
- React Testing Library — Core Principles — https://github.com/mintuz/skills/blob/main/data/skills-md/mintuz/claude-plugins/react-testing/SKILL.md
- Testing Library — Setup: Custom Render — https://testing-library.com/docs/react-testing-library/setup

---

## References

- React Testing Library — Introduction — https://testing-library.com/docs/react-testing-library/intro
- React Testing Library — render API — https://testing-library.com/docs/react-testing-library/api#render
- React Testing Library — Setup (Custom Render, automatic cleanup) — https://testing-library.com/docs/react-testing-library/setup
- Testing Library — Query Priority — https://testing-library.com/docs/queries/about/#priority
- Testing Library — user-event Introduction — https://testing-library.com/docs/user-event/intro
- Testing Library — Jest DOM matchers — https://github.com/testing-library/jest-dom
- Component Testing (tartinerlabs/skills) — https://github.com/tartinerlabs/skills/blob/3a2b1ba97c4f599f5b5ab76c5a2d7914ce9b24b1/plugins/tooling/skills/testing/rules/component-testing.md
- React Testing Library Best Practices (gohypergiant) — https://github.com/gohypergiant/standard-toolkit/blob/88ef844bf587a9fd43f75ee0503dc471569a2cd1/.agents/skills/accelint-react-testing/README.md
- React Testing Library Skill (mintuz) — https://github.com/mintuz/skills/blob/main/data/skills-md/mintuz/claude-plugins/react-testing/SKILL.md
- Component Testing (ledgerhq/lumen) — https://github.com/ledgerhq/lumen/blob/1d4929f3eb762cce283c87eea0f3ebc1f74dc8ba/.claude/skills/component-testing/SKILL.md
- React Testing Library: Advanced Patterns for Complex UIs (Codeloom) — https://codeloomdevv.co.in/blog/react/react-testing-library-advanced-patterns
- Souvlaki — Composable test wrappers for context-heavy components — https://www.npmjs.com/package/souvlaki
- ErrorBoundary.test.tsx (HuggingFace example) — https://huggingface.co/spaces/Ma-Ri-Ba-Ku/alto-llm-corrector/blob/a94837f5c02ea105429f5c9a7d7e5f6e821113f6/frontend/src/components/ErrorBoundary.test.tsx
- Mantine — Testing portals with React Testing Library — https://github.com/mantinedev/mantine/blob/3646de9f/apps/help.mantine.dev/src/pages/q/portals-testing.mdx
- Steve Kinney — Testing React Components with TypeScript — https://stevekinney.com/courses/react-typescript/testing-react-typescript
- Scrimba — How to Test React Apps [2026]: A Practical Guide — https://scrimba.com/articles/how-to-test-react-apps-2026/