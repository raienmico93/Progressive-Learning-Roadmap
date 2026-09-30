# Component & Integration Testing (React Testing Library): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Component and integration testing with React Testing Library (RTL) is the practice of verifying React component behaviour by rendering components into a simulated DOM, querying elements the way users perceive them, and asserting on observable outcomes rather than internal implementation details.

**Technical Definition:** React Testing Library is a lightweight testing utility built on top of `@testing-library/dom` that provides a `render` function, a suite of DOM queries (`getByRole`, `findByText`, `queryByLabelText`, etc.), and user interaction simulation via `@testing-library/user-event`. RTL operates on the accessibility tree rather than the internal component tree, meaning queries reflect how assistive technologies and users perceive the rendered output. The library exports `renderHook` for testing custom hooks in isolation, `waitFor` for asynchronous assertions, and integrates with test runners like Vitest and Jest. RTL v16 supports React 19 and requires `@testing-library/dom` as a peer dependency.

**Beginner-Friendly Explanation:** React Testing Library is a tool that lets you test your React components the way a real user would use them. Instead of looking at the internal state of a component or its CSS classes, you render the component, find elements the way a person would (by their label, their role, or their visible text), interact with them (type, click, submit), and check what appears on screen. This makes your tests more reliable because they don't break when you rename a variable or change a `<div>` to a `<section>` — they only break when the actual user-facing behaviour changes.

### Key Characteristics

- **User-Centric Queries:** The primary query method is `getByRole`, which matches elements by their ARIA role and accessible name — the same signals used by screen readers. This enforces semantic HTML and accessibility by default. 
- **Accessibility as a Testing Signal:** If you cannot query an element by its role or label, it likely lacks proper accessibility semantics. RTL's query priority (role > label > text > test ID) makes accessibility a first-class testing concern. 
- **Realistic User Interactions:** `@testing-library/user-event` simulates full browser interaction lifecycles — pointer events, focus, keyboard events, and change events — rather than dispatching a single synthetic event like `fireEvent`. 
- **Asynchronous Awareness:** `findBy*` queries and `waitFor` handle dynamic DOM mutations, delays, and async data loading without manual polling. 
- **Hook Isolation:** `renderHook` allows custom hooks to be tested independently of any UI component, with support for provider wrappers. 
- **Implementation Agnostic:** Tests assert on what the user sees and does, not on component state, internal methods, or lifecycle hooks. 

### Prerequisites

- Solid understanding of React components, props, state, and Hooks.
- Familiarity with JavaScript Promises and `async`/`await` syntax.
- Basic knowledge of DOM structure and accessibility semantics (roles, labels, accessible names).
- A configured test runner (Vitest or Jest) with a DOM environment (jsdom or happy-dom).
- Installation of `@testing-library/react`, `@testing-library/user-event`, and `@testing-library/jest-dom`.

### Related Programming Areas

- **Accessibility (a11y):** Testing that the UI is usable by assistive technologies.
- **Behaviour-Driven Development (BDD):** Describing behaviour in user-facing terms.
- **Integration Testing:** Testing multiple units working together.
- **Test-Driven Development (TDD):** Writing tests before implementation to drive design.
- **Mock Service Worker (MSW):** Mocking API calls at the network level while keeping the rest of the application real.

### Core Concepts / Features

1. DOM Query Strategies
2. Simulating Real User Interactions
3. Asynchronous Assertions
4. State, Context, & Hook Testing

---

## Core Concept 1: DOM Query Strategies

### Definitions

**Core Definition:** DOM Query Strategies in RTL are the methods used to locate elements in the rendered output, prioritised by their accessibility to users and assistive technologies.

**Technical Definition:** React Testing Library exposes three query families — `getBy*` (throws if not found), `queryBy*` (returns `null` if not found), and `findBy*` (returns a Promise that resolves when the element appears) — each available in single-element (`getBy`) and multi-element (`getAllBy`) variants. The queries are prioritised by accessibility: `getByRole` matches elements exposed in the accessibility tree by their ARIA role and accessible name; `getByLabelText` matches form controls by their associated label; `getByPlaceholderText` matches inputs by placeholder text; `getByText` matches non-interactive elements by visible text; `getByDisplayValue` matches form elements by their current value; `getByAltText` matches images by their alt text; `getByTitle` matches elements by their title attribute; and `getByTestId` matches elements by a `data-testid` attribute as a last resort. 

**Beginner-Friendly Explanation:** When you test a component, you need to find the elements you want to interact with. RTL gives you many different ways to find things. The best way is to find them the same way a user would: by their role (like "button" or "textbox"), by their label (like "Email"), or by their visible text. If you can't find something that way, you can fall back to test IDs, but that should be rare — if an element can't be found by its role or label, it probably isn't accessible to screen reader users either.

### Purposes

- To enforce semantic HTML and accessibility by requiring elements to be queryable by their accessible role or label.
- To make tests resilient to implementation changes (CSS classes, DOM structure) by coupling to user-visible semantics.
- To provide a consistent query API across all test files and projects.
- To catch accessibility regressions: if an element loses its accessible name, `getByRole` fails.
- To distinguish between elements that must exist (`getBy`), may not exist (`queryBy`), and appear asynchronously (`findBy`).
- To align testing with the accessibility tree — the same representation used by screen readers and assistive technologies.

### Syntax Rules and Structure

**Query Priority (Best to Worst):**

| Priority | Query | Use For |
|----------|-------|---------|
| 1 | `getByRole` | Buttons, links, headings, inputs, dialogs |
| 2 | `getByLabelText` | Form inputs with associated labels |
| 3 | `getByPlaceholderText` | Inputs without visible labels |
| 4 | `getByText` | Non-interactive text content |
| 5 | `getByDisplayValue` | Form elements by current value |
| 6 | `getByAltText` | Images by alt text |
| 7 | `getByTitle` | Elements by title attribute (low accessibility) |
| 8 | `getByTestId` | Last resort only |

**Query Variant Behaviour:**

| Variant | No Match | 1 Match | >1 Match | Async |
|---------|----------|---------|----------|-------|
| `getBy*` | throw | return | throw | No |
| `queryBy*` | null | return | throw | No |
| `findBy*` | throw | return | throw | Yes |
| `getAllBy*` | throw | array | array | No |
| `queryAllBy*` | [] | array | array | No |
| `findAllBy*` | throw | array | array | Yes |

**General Syntax:**

```jsx
import { render, screen } from '@testing-library/react';

test('query examples', () => {
  render(<LoginForm />);

  // ✅ BEST: Query by accessible role and name
  const submitButton = screen.getByRole('button', { name: /log in/i });
  const emailInput = screen.getByRole('textbox', { name: /email/i });

  // ✅ GOOD: Query form fields by label
  const passwordInput = screen.getByLabelText(/password/i);

  // ✅ OK: Query non-interactive text
  const heading = screen.getByText(/welcome back/i);

  // ✅ Check element doesn't exist
  expect(screen.queryByRole('dialog')).not.toBeInTheDocument();

  // ✅ Wait for async element
  const modal = await screen.findByRole('dialog');

  // ⚠️ LAST RESORT: Query by test ID
  const customWidget = screen.getByTestId('custom-widget');
});
```

**Component Breakdown:**
- `screen.getByRole('button', { name: /log in/i })`: Finds the button by its ARIA role and accessible name.
- `screen.getByLabelText(/password/i)`: Finds the input by its associated `<label>`.
- `screen.queryByRole('dialog')`: Returns `null` if no dialog exists (for negative assertions).
- `screen.findByRole('dialog')`: Returns a Promise that resolves when the dialog appears.

**Syntax Rules:**
- Always start with `getByRole`. It is the primary recommended query because it matches how assistive technologies perceive the component. 
- Use `getByLabelText` for form fields. Every form control should have an associated label.
- Use `queryBy*` for negative assertions (`expect(...).not.toBeInTheDocument()`). `getBy*` throws when the element is not found, making it unsuitable for absence checks. 
- Use `findBy*` for elements that appear asynchronously. It combines `waitFor` with `getBy` in a single, readable call. 
- Use `getByTestId` only when no accessible query is feasible, and document why.
- The `name` option in `getByRole` refers to the accessible name, not the HTML `name` attribute. 
- Use `getAllBy*` when multiple elements are expected; `getBy*` throws if more than one match is found.

**Constraints and Limitations:**
- Some elements may not have implicit ARIA roles; explicit `role` attributes may be required.
- `getByText` may match multiple elements if the text appears in multiple places; use `getAllByText` or scope the query.
- `getByRole` with `hidden: true` includes elements normally excluded from the accessibility tree. 
- `getByTestId` is an escape hatch and should not become the default query strategy.
- Querying by CSS class or DOM structure is not supported by RTL's primary API and is an anti-pattern.

### Annotated Code Examples

**Example 1: Query Priority in a Login Form**

```jsx
import React from 'react';
import { render, screen } from '@testing-library/react';

function LoginForm({ onSubmit }) {
  return (
    <form onSubmit={onSubmit}>
      <h1>Log In</h1>

      <label htmlFor="email">Email</label>
      <input id="email" type="email" placeholder="you@example.com" />

      <label htmlFor="password">Password</label>
      <input id="password" type="password" />

      <button type="submit">Log In</button>

      <a href="/forgot-password">Forgot password?</a>
    </form>
  );
}

test('finds elements using accessible queries', () => {
  render(<LoginForm onSubmit={vi.fn()} />);

  // ✅ Query by role and accessible name
  const heading = screen.getByRole('heading', { level: 1, name: /log in/i });
  const emailInput = screen.getByRole('textbox', { name: /email/i });
  const passwordInput = screen.getByLabelText(/password/i);
  const submitButton = screen.getByRole('button', { name: /log in/i });
  const forgotLink = screen.getByRole('link', { name: /forgot password/i });

  expect(heading).toBeInTheDocument();
  expect(emailInput).toHaveAttribute('type', 'email');
  expect(passwordInput).toHaveAttribute('type', 'password');
  expect(submitButton).toBeInTheDocument();
  expect(forgotLink).toHaveAttribute('href', '/forgot-password');
});
```

**Expected Output:** All assertions pass. The heading is found by its role (`heading`) and level (`1`). The email input is found by its role (`textbox`) and accessible name (`/email/i`). The password input is found by its label. The submit button is found by its role and accessible name. The link is found by its role (`link`) and accessible name.

**Why This Output Occurs:** Each element has proper semantic HTML: `<h1>` has an implicit `heading` role, `<input type="email">` has an implicit `textbox` role, `<label>` provides the accessible name, `<button>` has a `button` role, and `<a>` has a `link` role. The test queries reflect how a user (and a screen reader) would perceive these elements, making the test resilient to CSS or DOM structure changes. 

**Example 2: Query Variants for Different Scenarios**

```jsx
import React, { useState } from 'react';
import { render, screen } from '@testing-library/react';

function TogglePanel() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen((prev) => !prev)}>
        {isOpen ? 'Close' : 'Open'}
      </button>
      {isOpen && <div role="dialog">Panel content</div>}
    </div>
  );
}

test('query variants in action', async () => {
  render(<TogglePanel />);

  // ✅ queryBy — element does not exist initially
  expect(screen.queryByRole('dialog')).not.toBeInTheDocument();

  // ✅ getBy — button exists now
  const toggleButton = screen.getByRole('button', { name: /open/i });
  expect(toggleButton).toBeInTheDocument();

  // Act: click the button
  await userEvent.click(toggleButton);

  // ✅ findBy — dialog appears asynchronously
  const dialog = await screen.findByRole('dialog');
  expect(dialog).toHaveTextContent('Panel content');

  // ✅ getAllBy — multiple elements
  const buttons = screen.getAllByRole('button');
  expect(buttons).toHaveLength(1);
});
```

**Expected Output:** The test passes. `queryByRole` returns `null` initially (no dialog). `getByRole` finds the toggle button. After clicking, `findByRole` waits for the dialog to appear. `getAllByRole` returns an array of buttons.

**Why This Output Occurs:** Each query variant serves a distinct purpose: `queryBy*` is used for negative assertions (element should not exist), `getBy*` is used for elements that exist synchronously, `findBy*` is used for elements that appear asynchronously, and `getAllBy*` is used when multiple elements are expected. Mixing these up is the most common source of both flakes and misleading failure messages. 

### Real-World Cases

- **Login forms:** Querying email and password inputs by their labels, the submit button by its role and name, and error messages by their role (`alert`).
- **Navigation menus:** Querying links by their role (`link`) and accessible name, ensuring keyboard navigability.
- **Data tables:** Querying rows by their role (`row`) and cells by their role (`cell`), enforcing semantic table structure.
- **Modals and dialogs:** Querying dialogs by their role (`dialog`) and close buttons by their accessible name.
- **Form validation:** Querying error messages by their role (`alert`) and asserting on their text content.
- **Accessibility audits:** Using `getByRole` as a proxy for accessibility — if an element cannot be found by role, it likely lacks proper semantics.

---

## Core Concept 2: Simulating Real User Interactions

### Definitions

**Core Definition:** Simulating real user interactions in RTL is the practice of using `@testing-library/user-event` to trigger the full lifecycle of browser events — pointer events, focus, keyboard, and change events — that occur when a real user interacts with the UI.

**Technical Definition:** `@testing-library/user-event` is a companion library to RTL that simulates user interactions at a higher level of fidelity than `fireEvent`. Where `fireEvent` dispatches a single synthetic DOM event (e.g., `fireEvent.click` dispatches only a `click` event), `user-event` simulates the complete sequence of events a browser fires during a real interaction: for a click, this includes `pointerover`, `pointerenter`, `pointerdown`, `mousedown`, `focus`, `pointerup`, `mouseup`, and `click`. Similarly, `user.type` simulates `keydown`, `keypress`, `keyup`, and `change` events for each character. Since v14, all `user-event` methods are asynchronous and must be awaited, and `userEvent.setup()` must be called before rendering to create a session that tracks state across interactions. 

**Beginner-Friendly Explanation:** When a real user clicks a button, the browser doesn't just fire one "click" event — it fires a whole sequence of events: the mouse moves over the button, the button gets focus, the mouse button goes down, the mouse button comes up, and finally the click event fires. The old `fireEvent` method only fires the final click, which can miss bugs. `user-event` simulates the whole sequence, so your tests behave much more like a real browser. This means you need to `await` every interaction, because the events happen over time.

### Purposes

- To simulate interactions with the same fidelity as a real browser, catching bugs that `fireEvent` misses.
- To test complete event lifecycles (pointer, focus, keyboard, change) rather than isolated synthetic events.
- To ensure that interactions requiring focus, keyboard modifiers, or pointer state work correctly.
- To reduce test flakiness caused by incomplete event simulation.
- To test keyboard accessibility (Tab, Shift+Tab, Escape, Enter) realistically.
- To provide a single, consistent API for all user interactions (`click`, `type`, `keyboard`, `hover`, `paste`, `upload`).

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('user can log in', async () => {
  // ✅ ALWAYS call setup() before render
  const user = userEvent.setup();

  render(<LoginForm />);

  // ✅ Await every interaction
  await user.type(screen.getByLabelText(/email/i), 'user@example.com');
  await user.type(screen.getByLabelText(/password/i), 'password123');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  // Assert on the outcome
  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});
```

**Component Breakdown:**
- `userEvent.setup()`: Creates a user session that tracks state (keyboard state, pointer state) across interactions. Must be called before `render`. 
- `user.type(element, text)`: Simulates typing each character, firing keyboard and change events.
- `user.click(element)`: Simulates a full pointer and mouse event sequence.
- `await`: Every `user-event` method returns a Promise and must be awaited.

**Keyboard Modifiers:**

```jsx
const user = userEvent.setup();

// Press and hold Shift, then click
await user.keyboard('[<ShiftLeft>]'); // Press Shift (without releasing)
await user.click(element); // Click with shiftKey: true
await user.keyboard('[/<ShiftLeft>]'); // Release Shift

// Press Enter
await user.keyboard('{Enter}');

// Tab to next element
await user.tab();

// Select text
await user.tripleClick(element);
```

**Component Breakdown:**
- `user.keyboard('[<ShiftLeft>]')`: Presses and holds the left Shift key.
- `user.keyboard('[/<ShiftLeft>]')`: Releases the left Shift key.
- `user.tab()`: Simulates pressing Tab to move focus.
- `user.tripleClick(element)`: Selects the element's text content.

**Syntax Rules:**
- Always call `userEvent.setup()` before `render()`. 
- Always `await` every `user-event` method; they are asynchronous. 
- Prefer `user-event` over `fireEvent` for almost all interactions. `fireEvent` is a low-level escape hatch for edge cases (e.g., simulating a specific event that `user-event` does not support). 
- Use `user.keyboard()` for keyboard interactions, `user.pointer()` for low-level pointer control, and `user.type()` for text input.
- `user-event` respects `pointer-events: none` and will throw if you try to click an unclickable element. Use `skipPointerEventsCheck` only when necessary. 
- Tests using `user-event` may require overriding the test framework's default timeout (5 seconds) for long interaction sequences.

**Constraints and Limitations:**
- `user-event` is slower than `fireEvent` because it simulates more events.
- Some third-party components (e.g., Radix UI Tabs) may rely on specific events (`onMouseDown`) that require `fireEvent` as a fallback. 
- `user-event` does not simulate browser-native behaviours like drag-and-drop file uploads perfectly; `user.upload()` handles file inputs. 
- The `pointerEventsCheck` can cause failures for components with complex CSS; use `skipPointerEventsCheck` sparingly. 
- `user-event` v14 changed many APIs from synchronous to asynchronous; older tutorials may show outdated patterns.

### Annotated Code Examples

**Example 1: Complete Login Interaction with user-event**

```jsx
import React, { useState } from 'react';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function LoginForm({ onLogin }) {
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
      await onLogin({ email, password });
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

test('should show validation error for invalid email', async () => {
  const user = userEvent.setup();
  render(<LoginForm onLogin={vi.fn()} />);

  // Act: type an invalid email and submit
  await user.type(screen.getByLabelText(/email/i), 'invalid-email');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  // Assert: error message appears
  const errorMessage = screen.getByRole('alert');
  expect(errorMessage).toHaveTextContent(/please enter a valid email/i);
});

test('should call onLogin with valid credentials', async () => {
  const user = userEvent.setup();
  const handleLogin = vi.fn().mockResolvedValue(undefined);

  render(<LoginForm onLogin={handleLogin} />);

  await user.type(screen.getByLabelText(/email/i), 'user@example.com');
  await user.type(screen.getByLabelText(/password/i), 'password123');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(handleLogin).toHaveBeenCalledWith({
    email: 'user@example.com',
    password: 'password123',
  });
});
```

**Expected Output:** Both tests pass. In the first test, the invalid email triggers the validation error, which is found by `getByRole('alert')`. In the second test, valid credentials cause `onLogin` to be called with the correct payload.

**Why This Output Occurs:** `user.type()` simulates the full keyboard and change event lifecycle for each character, so the input's `onChange` handler fires correctly. `user.click()` simulates the full pointer and mouse event sequence, so the form's `onSubmit` handler fires. The tests await every interaction, ensuring the UI has settled before assertions run. 

**Example 2: fireEvent vs user-event Comparison**

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Counter() {
  const [count, setCount] = React.useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <p>Count: {count}</p>
    </div>
  );
}

// ❌ fireEvent: dispatches only the click event
test('fireEvent increments the counter', () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
});

// ✅ user-event: simulates the full pointer + mouse + click sequence
test('user-event increments the counter', async () => {
  const user = userEvent.setup();
  render(<Counter />);

  await user.click(screen.getByRole('button', { name: /increment/i }));

  expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
});
```

**Expected Output:** Both tests pass for this simple counter. However, for components that rely on focus, pointer state, or keyboard modifiers, the `fireEvent` test would fail while the `user-event` test would pass.

**Why This Output Occurs:** `fireEvent.click` dispatches a single synthetic `click` event. `user.click` simulates `pointerover`, `pointerenter`, `pointerdown`, `mousedown`, `focus`, `pointerup`, `mouseup`, and `click` in sequence. For a simple button, the outcome is the same. For a component that checks `event.shiftKey` or relies on focus being set, `fireEvent` would miss the context. 

### Real-World Cases

- **Form submission:** Simulating typing into inputs, clicking submit, and asserting on the result.
- **Keyboard navigation:** Testing Tab order, Escape to close modals, and Enter to submit forms.
- **Dropdown menus:** Simulating hover, click, and keyboard selection of menu items.
- **File uploads:** Using `user.upload()` to simulate selecting a file.
- **Text selection:** Using `user.tripleClick()` and `user.pointer()` to simulate text selection.
- **Clipboard operations:** Using `user.copy()`, `user.paste()`, and `user.cut()` for clipboard interactions.

---

## Core Concept 3: Asynchronous Assertions

### Definitions

**Core Definition:** Asynchronous assertions in RTL are the mechanisms — `findBy*` queries, `waitFor`, and `waitForElementToBeRemoved` — used to wait for dynamic DOM mutations, data loading, or delayed state updates before asserting on the result.

**Technical Definition:** `findBy*` queries return a Promise that resolves with the found element(s) or rejects if the element is not found within a default timeout of 1000ms. They are the async equivalent of `getBy*`, combining `waitFor` with a `getBy` query in a single readable call. `waitFor` accepts a callback and retries it until it stops throwing or the timeout expires (default 1000ms). It is intended for non-element assertions, such as waiting for a state value to change or for a function to be called. `waitForElementToBeRemoved` waits for an element to be removed from the DOM. The key principle is: use `findBy*` to wait for an element to appear, use `waitFor` for assertions that are not about element presence, and never put side effects or multiple assertions inside `waitFor`. 

**Beginner-Friendly Explanation:** Sometimes your app takes time to load data or update the screen. You can't just check immediately — the data might not be there yet. `findBy*` queries wait for an element to appear (up to 1 second by default) and then return it. `waitFor` waits for any assertion to become true, like "the loading spinner is gone" or "the count is now 3." The rule of thumb is: if you're waiting for an element, use `findBy*`; if you're waiting for something else, use `waitFor`. Never put multiple assertions inside `waitFor` — if the first one fails, you won't know which one caused the problem. 

### Purposes

- To handle dynamic DOM mutations caused by asynchronous data fetching, timers, or state updates.
- To provide readable, intention-revealing waiting mechanisms (`findBy*` vs `waitFor`).
- To avoid manual polling loops or arbitrary `setTimeout` delays in tests.
- To produce clear failure messages when expected elements never appear.
- To distinguish between element queries (use `findBy*`) and non-element assertions (use `waitFor`).
- To manage timeouts for slow paths without polluting the test body with delay logic.

### Syntax Rules and Structure

**General Syntax for `findBy*`:**

```jsx
import { render, screen } from '@testing-library/react';

test('waits for async content to appear', async () => {
  render(<UserList />);

  // ✅ findByRole returns a Promise that resolves when the element appears
  const heading = await screen.findByRole('heading', { name: /users/i });
  expect(heading).toBeInTheDocument();

  // ✅ findAllBy returns an array
  const items = await screen.findAllByRole('listitem');
  expect(items).toHaveLength(3);
});
```

**General Syntax for `waitFor`:**

```jsx
import { render, screen, waitFor } from '@testing-library/react';

test('waits for a non-element assertion', async () => {
  render(<Counter />);

  await user.click(screen.getByRole('button', { name: /increment/i }));

  // ✅ waitFor for assertions that are not about element presence
  await waitFor(() => {
    expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
  });
});

test('waits for multiple conditions', async () => {
  render(<RegistrationForm />);

  await user.click(screen.getByRole('button', { name: /submit/i }));

  // ✅ One assertion per waitFor
  await waitFor(() => {
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
  });
  await waitFor(() => {
    expect(screen.getByText(/password is required/i)).toBeInTheDocument();
  });
});
```

**Component Breakdown:**
- `screen.findByRole(...)`: Returns a Promise that resolves with the element or rejects after the timeout.
- `await waitFor(() => { expect(...) })`: Retries the callback until it stops throwing or the timeout expires.
- `waitFor` with one assertion per call: Ensures the first failure reports immediately without ambiguity.

**General Syntax for `waitForElementToBeRemoved`:**

```jsx
import { render, screen, waitForElementToBeRemoved } from '@testing-library/react';

test('waits for loading spinner to disappear', async () => {
  render(<DataView />);

  // Assert loading state is present
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // ✅ Wait for the loading indicator to be removed
  await waitForElementToBeRemoved(() => screen.queryByText(/loading/i));

  // Assert data is now visible
  expect(screen.getByText(/data loaded/i)).toBeInTheDocument();
});
```

**Component Breakdown:**
- `waitForElementToBeRemoved(() => screen.queryByText(/loading/i))`: Waits for the element to be removed from the DOM.
- The callback must return the element or `null`; when it returns `null`, the element has been removed.

**Syntax Rules:**
- Use `findBy*` to wait for an element to appear; it is `getBy*` plus `waitFor` under the hood. 
- Use `waitFor` only for non-element assertions (state changes, function calls, etc.). 
- Never put multiple assertions inside a single `waitFor`; use separate `waitFor` calls. 
- Never perform side effects inside `waitFor`; it may run the callback multiple times. 
- Use `findBy*` with a timeout option for slow paths: `await screen.findByText(/slow/i, {}, { timeout: 3000 })`. 
- `findBy*` automatically retries until the element appears or the timeout expires (default 1000ms).
- Prefer `findBy*` over `waitFor` + `getBy*` because it produces better error messages and is more concise. 

**Constraints and Limitations:**
- `findBy*` has a default timeout of 1000ms; slow API calls may require a longer timeout.
- `waitFor` retries the callback on a timer (default 50ms interval); for very fast updates, this may add latency.
- `waitFor` does not work with fake timers unless configured (use `vi.useFakeTimers()` with `advanceTimers` option). 
- `findBy*` queries are not suitable for asserting absence — use `queryBy*` with `waitFor` for that.
- Multiple `findBy*` calls in sequence can be slower than a single `waitFor` with a combined assertion, but they produce clearer failure messages.

### Annotated Code Examples

**Example 1: Search Results with Async Loading**

```jsx
import React, { useState } from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  async function handleSearch() {
    setLoading(true);
    // Simulate API call
    await new Promise((resolve) => setTimeout(resolve, 300));
    setResults(['React Testing', 'React Hooks', 'React Suspense']);
    setLoading(false);
  }

  return (
    <div>
      <input
        role="searchbox"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button onClick={handleSearch}>Search</button>

      {loading && <p>Searching...</p>}

      <ul>
        {results.map((result) => (
          <li key={result}>{result}</li>
        ))}
      </ul>
    </div>
  );
}

test('displays search results asynchronously', async () => {
  const user = userEvent.setup();
  render(<SearchResults />);

  // Act: type and search
  await user.type(screen.getByRole('searchbox'), 'react');
  await user.click(screen.getByRole('button', { name: /search/i }));

  // Assert: loading indicator is present synchronously
  expect(screen.getByText(/searching/i)).toBeInTheDocument();

  // ✅ findBy waits for results to appear
  const results = await screen.findAllByRole('listitem');
  expect(results).toHaveLength(3);
  expect(screen.queryByText(/searching/i)).not.toBeInTheDocument();
});

test('raises the timeout for a slow path', async () => {
  const user = userEvent.setup();
  render(<SearchResults />);

  await user.type(screen.getByRole('searchbox'), 'slow-query');
  await user.click(screen.getByRole('button', { name: /search/i }));

  // ✅ Third argument is the waitFor options, not the query options
  const result = await screen.findByText(
    /react suspense/i,
    {},
    { timeout: 3000 }
  );
  expect(result).toBeInTheDocument();
});
```

**Expected Output:** The first test passes: "Searching..." appears synchronously, then `findAllByRole('listitem')` waits for the results to appear (after 300ms) and returns three items. The second test passes with an extended timeout of 3000ms.

**Why This Output Occurs:** `getByText(/searching/i)` finds the loading indicator synchronously because it is rendered immediately after the click. `findAllByRole('listitem')` returns a Promise that resolves when the list items appear after the simulated API delay. `queryByText(/searching/i)` returns `null` because the loading indicator is removed when results arrive. The third argument to `findByText` is the `waitFor` options object, not the query options. 

**Example 2: waitFor for Non-Element Assertions**

```jsx
import React, { useState } from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function DebouncedCounter() {
  const [count, setCount] = useState(0);
  const [debouncedCount, setDebouncedCount] = useState(0);

  React.useEffect(() => {
    const timer = setTimeout(() => setDebouncedCount(count), 500);
    return () => clearTimeout(timer);
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
      <p>Count: {count}</p>
      <p>Debounced: {debouncedCount}</p>
    </div>
  );
}

test('counter settles after debounce', async () => {
  const user = userEvent.setup();
  render(<DebouncedCounter />);

  // Act: click three times rapidly
  await user.click(screen.getByRole('button', { name: /increment/i }));
  await user.click(screen.getByRole('button', { name: /increment/i }));
  await user.click(screen.getByRole('button', { name: /increment/i }));

  // ✅ waitFor for the debounced value to settle
  await waitFor(() => {
    expect(screen.getByText(/debounced: 3/i)).toBeInTheDocument();
  });
});
```

**Expected Output:** The test passes. The immediate count shows 3, and after the 500ms debounce, the debounced count also shows 3.

**Why This Output Occurs:** `waitFor` retries the callback until the assertion passes or the timeout expires. It is used here because the assertion is about the debounced count reaching a specific value, which is not an element query but a state-derived text assertion. `findBy*` would not be appropriate because the element (`<p>Debounced: 3</p>`) exists from the start — its text content changes. 

### Real-World Cases

- **Data fetching:** Waiting for API responses to populate lists, tables, or detail views.
- **Form submission:** Waiting for success messages or error alerts after submitting a form.
- **Debounced search:** Waiting for debounced results to settle after rapid typing.
- **Modal transitions:** Waiting for a modal to appear after clicking a trigger, or disappear after clicking close.
- **Loading states:** Waiting for a loading spinner to be removed and content to appear.
- **Optimistic updates:** Waiting for the server response to confirm an optimistic UI change.

---

## Core Concept 4: State, Context, & Hook Testing

### Definitions

**Core Definition:** State, Context, and Hook testing in RTL is the practice of validating custom hook behaviour, context provider integration, and state management logic either in isolation using `renderHook` or through component integration tests.

**Technical Definition:** `renderHook` is a utility exported by `@testing-library/react` (since v13.1.0) that renders a test component which calls the provided hook callback and returns an object containing `result` (the current return value of the hook) and utilities like `rerender` and `unmount`. It supports a `wrapper` option for wrapping the hook in providers (e.g., `QueryClientProvider`, `AuthProvider`, `MemoryRouter`). The `act` function from RTL (re-exported from `react-dom/test-utils`) is used to wrap state updates that occur outside of `render` or `renderHook`. For context testing, the recommended approach is to render a consumer component within the real provider and assert on the rendered output, rather than mocking the context. 

**Beginner-Friendly Explanation:** Custom hooks are functions that contain state and logic, but they can't be rendered on their own — they need to be called inside a component. `renderHook` lets you test a hook without creating a full component for it. You call `renderHook(() => useCounter())`, and it returns an object with `result.current` that gives you the hook's current return value. You can call functions from the hook inside `act()` to trigger state updates, and you can wrap the hook in providers using the `wrapper` option. For context, instead of mocking the context, you render a component that consumes the context inside the real provider and assert on what the component renders.

### Purposes

- To test custom hook logic (state updates, effects, return values) independently of any UI component.
- To validate context providers by rendering consumer components within them.
- To test hooks that depend on providers (e.g., `useQuery` requiring `QueryClientProvider`) by supplying a `wrapper`.
- To isolate hook behaviour from component rendering logic, making tests faster and more focused.
- To verify that state updates propagate correctly through context to consuming components.
- To test hook error handling and edge cases without the overhead of a full component tree.

### Syntax Rules and Structure

**General Syntax for `renderHook`:**

```jsx
import { renderHook, act } from '@testing-library/react';

function useCounter(initialValue = 0) {
  const [count, setCount] = React.useState(initialValue);
  const increment = () => setCount((c) => c + 1);
  const decrement = () => setCount((c) => c - 1);
  return { count, increment, decrement };
}

test('useCounter increments and decrements', () => {
  const { result } = renderHook(() => useCounter(10));

  // Initial value
  expect(result.current.count).toBe(10);

  // ✅ Wrap state updates in act()
  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(11);

  act(() => {
    result.current.decrement();
  });
  expect(result.current.count).toBe(10);
});
```

**Component Breakdown:**
- `renderHook(() => useCounter(10))`: Renders the hook in a test component and returns `{ result, rerender, unmount }`.
- `result.current`: The current return value of the hook.
- `act(() => { result.current.increment() })`: Wraps the state update so React flushes it before assertions.

**General Syntax for `renderHook` with Provider Wrapper:**

```jsx
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

test('useFetchUser returns user data', async () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  // ✅ Wrapper provides the necessary context
  const wrapper = ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );

  const { result } = renderHook(() => useFetchUser('123'), { wrapper });

  await waitFor(() => {
    expect(result.current.data).toEqual({ name: 'John' });
  });
});
```

**Component Breakdown:**
- `wrapper`: A component that wraps the hook in the required providers.
- `renderHook(() => useFetchUser('123'), { wrapper })`: Passes the wrapper as an option.
- `waitFor`: Waits for the async hook to resolve.

**General Syntax for Context Testing Through Components:**

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { AuthProvider, useAuth } from './AuthContext';

function Consumer() {
  const { user, login } = useAuth();
  return (
    <div>
      <p>{user ? `Welcome, ${user.name}` : 'Not logged in'}</p>
      <button onClick={() => login({ name: 'Alice' })}>Log in</button>
    </div>
  );
}

test('context provides user and login function', async () => {
  const user = userEvent.setup();

  // ✅ Render the consumer inside the real provider
  render(
    <AuthProvider>
      <Consumer />
    </AuthProvider>
  );

  expect(screen.getByText(/not logged in/i)).toBeInTheDocument();

  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(screen.getByText(/welcome, alice/i)).toBeInTheDocument();
});
```

**Component Breakdown:**
- `<AuthProvider>`: The real provider component.
- `<Consumer>`: A component that consumes the context.
- `screen.getByText(...)`: Asserts on the rendered output of the consumer.

**Syntax Rules:**
- Use `renderHook` for custom hooks that are not tightly coupled to a specific UI component. 
- Use component testing when the hook's behaviour is best observed through the UI it renders. 
- Always wrap state updates in `act()` when calling hook methods outside of `render` or `renderHook`. 
- Use the `wrapper` option to supply providers required by the hook. 
- For context testing, render a consumer component inside the real provider rather than mocking the context. 
- Create a fresh provider instance (e.g., `QueryClient`) for each test to ensure isolation. 
- `renderHook` returns `result`, `rerender`, and `unmount`; use `rerender` to test changes in hook arguments.

**Constraints and Limitations:**
- `renderHook` should not be used when the hook is tightly coupled to a specific UI component; test through the component instead. 
- `act()` is required for state updates but does not work with asynchronous state updates in all cases; use `waitFor` for async.
- The `wrapper` option applies the same providers to all renders; if different tests need different providers, create separate `renderHook` calls.
- `renderHook` does not support Server Components; use E2E tests for those.
- Context testing through components requires more boilerplate than mocking, but provides higher confidence.

### Annotated Code Examples

**Example 1: Custom Hook Testing with `renderHook`**

```jsx
import { renderHook, act } from '@testing-library/react';

function useCounter(initialValue = 0) {
  const [count, setCount] = React.useState(initialValue);

  const increment = () => setCount((c) => c + 1);
  const decrement = () => setCount((c) => c - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

test('useCounter manages count state', () => {
  const { result } = renderHook(() => useCounter(5));

  expect(result.current.count).toBe(5);

  act(() => result.current.increment());
  expect(result.current.count).toBe(6);

  act(() => result.current.decrement());
  expect(result.current.count).toBe(5);

  act(() => result.current.reset());
  expect(result.current.count).toBe(5);
});

test('useCounter rerenders with new initial value', () => {
  const { result, rerender } = renderHook(
    ({ initial }) => useCounter(initial),
    { initialProps: { initial: 0 } }
  );

  expect(result.current.count).toBe(0);

  rerender({ initial: 10 });
  expect(result.current.count).toBe(10);
});
```

**Expected Output:** Both tests pass. The first test verifies increment, decrement, and reset. The second test verifies that the hook responds to changes in its initial value via `rerender`.

**Why This Output Occurs:** `renderHook` renders a test component that calls the hook. `result.current` always reflects the latest return value. `act()` ensures that state updates are flushed before assertions. `rerender` re-renders the hook with new props, allowing testing of hooks that depend on external inputs. 

**Example 2: Context Testing Through a Consumer Component**

```jsx
import React, { createContext, useContext, useState } from 'react';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

const ThemeContext = createContext(null);

function ThemeProvider({ children, initialTheme = 'light' }) {
  const [theme, setTheme] = useState(initialTheme);
  const toggleTheme = () => setTheme((t) => (t === 'light' ? 'dark' : 'light'));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  return useContext(ThemeContext);
}

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>Toggle theme</button>
    </div>
  );
}

test('theme toggles between light and dark', async () => {
  const user = userEvent.setup();

  render(
    <ThemeProvider initialTheme="light">
      <ThemeToggle />
    </ThemeProvider>
  );

  expect(screen.getByText(/current theme: light/i)).toBeInTheDocument();

  await user.click(screen.getByRole('button', { name: /toggle theme/i }));

  expect(screen.getByText(/current theme: dark/i)).toBeInTheDocument();
});
```

**Expected Output:** The test passes. The initial theme is "light", and after clicking the toggle button, the theme changes to "dark".

**Why This Output Occurs:** The `ThemeToggle` component consumes the `ThemeContext` and renders the current theme. The test renders the component inside the real `ThemeProvider`, so the context provides the actual state and toggle function. This tests the integration between the provider, the context, and the consumer, which is more valuable than mocking the context. 

### Real-World Cases

- **Form state hooks:** Testing `useForm`, `useField`, or custom form hooks with `renderHook` and `act`.
- **Authentication hooks:** Testing `useAuth` with a wrapper that provides the `AuthProvider`.
- **Data fetching hooks:** Testing `useQuery`, `useMutation`, or custom fetch hooks with a `QueryClientProvider` wrapper.
- **Theme hooks:** Testing `useTheme` through a component that consumes the theme context.
- **Router hooks:** Testing `useNavigate` or `useParams` by wrapping the hook in a `MemoryRouter`.
- **Local storage hooks:** Testing `useLocalStorage` by verifying read, write, and sync behaviour.

---

## References

- React Testing Library – Official Documentation: https://testing-library.com/docs/react-testing-library/intro
- React Testing Library – Query Priority: https://testing-library.com/docs/queries/about/#priority
- React Testing Library – About Queries: https://testing-library.com/docs/queries/about
- Testing Library – Guiding Principles: https://testing-library.com/docs/guiding-principles
- user-event – Official Documentation: https://testing-library.com/docs/user-event/intro
- user-event – Setup: https://testing-library.com/docs/user-event/setup
- user-event – Pointer API: https://testing-library.com/docs/user-event/pointer
- React Testing Library – Async Utilities: https://testing-library.com/docs/dom-testing-library/api-async
- React Testing Library – renderHook API: https://testing-library.com/docs/react-testing-library/api#renderhook
- React Testing Library – act: https://testing-library.com/docs/react-testing-library/api#act
- Kent C. Dodds – The Testing Trophy and Testing Classifications: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Kent C. Dodds – Write Tests. Not Too Many. Mostly Integration.: https://kentcdodds.com/blog/write-tests
- Steve Kinney – FireEvent vs UserEvent in Testing: https://stevekinney.com/courses/testing/user-event
- Steve Kinney – TypeScript Patterns for React Testing: https://stevekinney.com/courses/react-typescript/typescript-react-testing
- Testing Library – Accessibility: https://testing-library.com/docs/queries/byrole
- MDN Web Docs – ARIA Roles: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles
- WAI-ARIA Authoring Practices: https://www.w3.org/WAI/ARIA/apg/
- Vitest – Environment Configuration: https://vitest.dev/config/environment
- Vitest – Test Environment Guide: https://vitest.dev/guide/environment
- TanStack Query – Testing: https://tanstack.com/query/latest/docs/framework/react/guides/testing