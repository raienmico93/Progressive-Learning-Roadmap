# Network, Error, & Async Boundaries in React Testing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Network, Error, and Async Boundaries in React testing refers to the set of techniques and tools used to intercept network requests, validate asynchronous UI state transitions, verify error-handling boundaries, and stub out non-JavaScript dependencies during component and integration tests.

**Technical Definition:** This domain encompasses four interlocking testing concerns: (1) **network mocking** via Mock Service Worker (MSW), which intercepts requests at the network level (browser or Node.js) rather than stubbing `window.fetch`; (2) **UI state validation**, which asserts on transitions between skeleton loading states, successful data renders, empty states, and error banners using React Testing Library's async utilities (`findBy*`, `waitFor`, `waitForElementToBeRemoved`); (3) **boundary and failure testing**, which intentionally throws network and runtime errors to verify that Error Boundaries and error banners render correctly; and (4) **dependency and asset mocking**, which replaces third-party modules with spies, mocks CSS and image imports, and stubs non-JavaScript assets so tests focus on component logic rather than styling or asset resolution.

**Beginner-Friendly Explanation:** When you test a React component that loads data from an API, you don't want to make real network requests — they're slow, unreliable, and might fail. Instead, you intercept those requests and return fake data. MSW lets you do this at the network level, so your component code stays exactly the same. Then you check that the component shows a loading spinner, then the data, then an empty state or an error message if something goes wrong. You also need to make sure that if the API returns an error, your Error Boundary catches it and shows a friendly message. Finally, you stub out things like CSS files and images so your tests don't break just because Vitest can't read a `.css` file.

### Key Characteristics

- **Network-Level Interception:** MSW intercepts requests at the network layer (via Service Worker in the browser or `@mswjs/interceptors` in Node.js), so the application code makes real `fetch` or `axios` calls that are transparently mocked. This is more realistic than stubbing `window.fetch` and does not break isolation.
- **Async-Aware Assertions:** React Testing Library's `findBy*` queries (`findByRole`, `findByText`) are asynchronous versions of `getBy*` that return a Promise, waiting for the element to appear before resolving. `waitFor` retries a callback until it passes or times out. `waitForElementToBeRemoved` waits for an element to disappear.
- **Boundary Verification:** Error Boundaries are tested by rendering a component that throws under controlled conditions (e.g., a `Bomb` component with a `shouldThrow` flag) and asserting that the fallback UI appears instead of the crashed component.
- **Asset Stubbing:** CSS, images, and other non-JavaScript assets are mocked or ignored in the test environment so that import statements do not throw `Unknown file extension ".css"` errors.
- **Spy vs. Mock Distinction:** `jest.spyOn` wraps an existing method and tracks calls while preserving the original implementation; `jest.mock` (or `vi.mock`) replaces an entire module with a mock implementation.
- **Shared Handlers, Per-Test Overrides:** MSW handlers are defined once in a shared `handlers.ts` file and reused across tests. Individual tests override them with `server.use()` for error simulation or custom responses.

### Prerequisites

- Solid understanding of React components, Hooks, and the render/commit lifecycle.
- Familiarity with JavaScript Promises and `async`/`await`.
- A configured test runner (Vitest or Jest) with a DOM environment (`jsdom` or `happy-dom`).
- React Testing Library and `@testing-library/user-event` installed.
- Basic understanding of HTTP status codes (200, 404, 500) and error handling.

### Related Programming Areas

- **Integration Testing:** Testing multiple units working together with realistic network behaviour.
- **Error Boundary Testing:** Verifying that React Error Boundaries catch rendering and async errors.
- **Accessibility Testing:** Using `getByRole` to enforce semantic HTML.
- **Mock Service Worker (MSW):** The recommended library for declarative API mocking.
- **Mocking and Spying:** Replacing modules and tracking function calls with `vi.mock`, `vi.spyOn`, `jest.mock`, and `jest.spyOn`.
- **Test Isolation:** Ensuring tests don't leak state or depend on execution order.

### Core Concepts / Features

1. Network Mocking with MSW
2. UI State Validation
3. Boundary & Failure Testing
4. Dependency & Asset Mocking

---

## Core Concept 1: Network Mocking with MSW (Mock Service Worker)

### Definitions

**Core Definition:** MSW (Mock Service Worker) is a library that intercepts network requests at the browser or Node.js level, allowing tests to return controlled API responses without modifying application code.

**Technical Definition:** MSW uses a Service Worker in the browser and `@mswjs/interceptors` in Node.js to intercept outgoing HTTP requests. Handlers defined with `http.get`, `http.post`, etc. match requests by URL and method, and return `HttpResponse` objects. The `setupServer` function from `msw/node` creates a mock server for test environments, while `setupWorker` from `msw/browser` is used for browser-based development. MSW v2 changed the API from `rest.get` (v1) to `http.get` (v2), and from Express-style response resolvers to the Fetch API `Response` class. Testing Library officially recommends MSW for API mocking instead of stubbing `window.fetch`.

**Beginner-Friendly Explanation:** Instead of replacing `fetch` with a fake function, MSW sits between your app and the real network. When your app makes a request, MSW catches it and returns whatever fake response you've defined. Your app doesn't know the difference — it makes the same `fetch` call it would in production. MSW is recommended by Testing Library because it doesn't break isolation and lets you test the actual request/response cycle.

### Purposes

- To intercept API requests at the network level without stubbing `fetch` or `axios`.
- To define reusable, declarative handlers for API endpoints.
- To simulate success, error, loading, and empty states with controlled responses.
- To override handlers per-test for error simulation and edge cases.
- To keep application code unchanged during tests — the same `fetch` calls run.
- To test loading states, error banners, and data rendering realistically.

### Syntax Rules and Structure

**General Syntax with MSW v2:**

```javascript
// handlers.ts — shared handlers
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/user', () => {
    return HttpResponse.json({ id: 1, name: 'Alice' });
  }),
];

// server.ts — Node.js test server
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// setupTests.ts
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Component Breakdown:**
- `http.get('/api/user', resolver)`: Defines a GET handler for `/api/user`.
- `HttpResponse.json(data)`: Returns a JSON response with status 200 by default.
- `setupServer(...handlers)`: Creates a Node.js mock server.
- `server.listen()`: Starts intercepting requests.
- `server.resetHandlers()`: Resets handlers to their original state after each test.
- `server.close()`: Stops the server after all tests.

**Per-Test Handler Override with `server.use()`:**

```javascript
import { server } from './mocks/server';
import { http, HttpResponse } from 'msw';

test('shows error banner on 500', async () => {
  server.use(
    http.get('/api/user', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );

  render(<UserProfile />);

  const errorBanner = await screen.findByRole('alert');
  expect(errorBanner).toHaveTextContent(/failed to load/i);
});
```

**Component Breakdown:**
- `server.use(http.get(...))`: Overrides the default handler for this test only.
- `new HttpResponse(null, { status: 500 })`: Returns a 500 response with no body.
- The override is automatically reset by `server.resetHandlers()` in `afterEach`.

**Syntax Rules:**
- Define shared handlers in a dedicated `handlers.ts` file for reuse across tests.
- Always call `server.listen()` in `beforeAll`, `server.resetHandlers()` in `afterEach`, and `server.close()` in `afterAll`.
- Use `server.use()` for per-test overrides — it takes precedence over shared handlers.
- MSW v2 uses `http.get` (not `rest.get`) and `HttpResponse.json` (not `res(ctx.json(...))`).
- For network errors, use `HttpResponse.error()` to simulate a connection failure.
- For delayed responses, use `await delay(ms)` from MSW to simulate slow networks.

**Constraints and Limitations:**
- MSW requires Node.js 18 or later.
- MSW v2 changed the API significantly from v1 — older tutorials may show `rest.get` and `res(ctx.json(...))`, which no longer work.
- In Next.js Server Components, MSW interception may not work as expected for server-side fetches; test these with E2E tests.
- MSW adds a small performance overhead to test startup, but this is negligible for most suites.

### Annotated Code Examples

**Example 1: Complete MSW Setup with Success and Error Tests**

```javascript
// mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/greeting', () => {
    return HttpResponse.json({ greeting: 'hello there' });
  }),
];

// mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/setupTests.ts
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

```jsx
// Greeting.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { Greeting } from './Greeting';

test('loads and displays greeting', async () => {
  const user = userEvent.setup();
  render(<Greeting />);

  await user.click(screen.getByText('Load Greeting'));

  const heading = await screen.findByRole('heading');
  expect(heading).toHaveTextContent('hello there');
  expect(screen.getByRole('button')).toBeDisabled();
});

test('handles server error', async () => {
  server.use(
    http.get('/api/greeting', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );

  const user = userEvent.setup();
  render(<Greeting />);

  await user.click(screen.getByText('Load Greeting'));

  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent('Oops, failed to fetch!');
  expect(screen.getByRole('button')).not.toBeDisabled();
});
```

**Expected Output:** The first test passes: the greeting "hello there" appears, and the button is disabled. The second test passes: the error alert "Oops, failed to fetch!" appears, and the button is re-enabled.

**Why This Output Occurs:** The shared handler returns a 200 response with the greeting. `findByRole('heading')` waits for the heading to appear. The per-test override with `server.use()` returns a 500 response, causing the component to render an error alert. `findByRole('alert')` waits for the alert to appear. The `server.resetHandlers()` in `afterEach` ensures the override does not leak into other tests.

**Example 2: Simulating Network Errors and Delays**

```javascript
import { http, HttpResponse, delay } from 'msw';

// Network error (connection failure)
http.get('/api/data', () => {
  return HttpResponse.error();
});

// Timeout simulation
http.get('/api/slow', async () => {
  await delay(5000);
  return HttpResponse.json({ data: 'too late' });
});

// Delayed success
http.get('/api/medium', async () => {
  await delay(500);
  return HttpResponse.json({ data: 'just in time' });
});
```

**Expected Output:** The network error causes `fetch` to reject with a `TypeError: Failed to fetch`. The timeout simulation delays the response by 5 seconds. The delayed success returns after 500ms.

**Why This Output Occurs:** `HttpResponse.error()` simulates a low-level network failure, which causes `fetch` to reject. `await delay(ms)` suspends the resolver for the specified duration before returning the response. This allows testing of loading spinners, timeout logic, and retry mechanisms.

### Real-World Cases

- **Dashboard data loading:** Mocking multiple API endpoints with different response times to test loading sequences.
- **E-commerce product pages:** Simulating 404 responses for missing products and 500 responses for server errors.
- **Authentication flows:** Mocking login success, invalid credentials (401), and server errors (500).
- **Search interfaces:** Testing empty results, populated results, and error states.
- **Real-time feeds:** Simulating delayed responses to test loading spinners and skeletons.

---

## Core Concept 2: UI State Validation

### Definitions

**Core Definition:** UI State Validation is the practice of asserting that a component correctly transitions between loading (skeleton), success (data rendered), empty, and error states based on asynchronous data-fetching outcomes.

**Technical Definition:** React Testing Library provides three async utilities for state validation: `findBy*` queries (async versions of `getBy*` that wait for an element to appear), `waitFor` (retries a callback until it passes or times out), and `waitForElementToBeRemoved` (waits for an element to disappear from the DOM). The recommended pattern is to assert the loading state synchronously with `getBy*`, then use `findBy*` to wait for the data or error state to appear. For negative assertions (element should not exist), use `queryBy*` with `not.toBeInTheDocument()`. The `waitFor` utility should contain exactly one assertion per call to avoid ambiguous failure messages.

**Beginner-Friendly Explanation:** When your component loads data, it usually shows a spinner first, then the data, or an error, or an empty state. Your test needs to check each of these. You use `getByText('Loading...')` to check the spinner immediately. Then you use `await screen.findByText('Alice')` to wait for the data to appear. If you want to check that the spinner is gone, you use `queryByText('Loading...')` and assert it's `null`. The key is: `getBy` for things that should already be there, `findBy` for things that will appear, and `queryBy` for things that should not exist.

### Purposes

- To verify that the loading skeleton appears before data is fetched.
- To assert that data renders correctly after a successful API response.
- To confirm that an empty state (e.g., "No results found") appears when the API returns no data.
- To validate that an error banner or alert appears when the API fails.
- To ensure that the loading indicator is removed once data or an error appears.
- To test the complete state transition lifecycle of an async component.

### Syntax Rules and Structure

**Query Variant Behaviour:**

| Variant | No Match | 1 Match | >1 Match | Async |
|---------|----------|---------|----------|-------|
| `getBy*` | throw | return | throw | No |
| `queryBy*` | null | return | throw | No |
| `findBy*` | throw | return | throw | Yes |

**General Syntax for State Transition Testing:**

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('loading → data transition', async () => {
  const user = userEvent.setup();
  render(<DataFetcher />);

  // ✅ Assert loading state synchronously
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Act: trigger the fetch
  await user.click(screen.getByRole('button', { name: /fetch/i }));

  // ✅ findBy waits for data to appear
  const data = await screen.findByText('Alice');
  expect(data).toBeInTheDocument();

  // ✅ Assert loading indicator is removed
  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
});
```

**Component Breakdown:**
- `getByText('Loading...')`: Synchronous query for the loading state.
- `findByText('Alice')`: Asynchronous query that waits for the data to appear.
- `queryByText('Loading...')`: Returns `null` if the loading indicator is gone.
- `waitFor`: Retries the assertion until the loading indicator is removed.

**General Syntax for Empty State Testing:**

```jsx
test('displays empty state when API returns no data', async () => {
  server.use(
    http.get('/api/items', () => {
      return HttpResponse.json([]);
    })
  );

  render(<ItemList />);

  const emptyMessage = await screen.findByText(/no items found/i);
  expect(emptyMessage).toBeInTheDocument();
  expect(screen.queryByRole('listitem')).not.toBeInTheDocument();
});
```

**Component Breakdown:**
- `server.use(...)`: Overrides the handler to return an empty array.
- `findByText(/no items found/i)`: Waits for the empty state message.
- `queryByRole('listitem')`: Returns `null` because no items exist.

**Syntax Rules:**
- Use `getBy*` for elements that should already be present (loading skeletons).
- Use `findBy*` for elements that appear asynchronously (data, error messages).
- Use `queryBy*` with `not.toBeInTheDocument()` for elements that should not exist.
- Use `waitForElementToBeRemoved` to wait for a loading indicator to disappear.
- Never put multiple assertions inside a single `waitFor` — use separate `waitFor` calls for each assertion.
- `findBy*` has a default timeout of 1000ms; use the third argument to extend it for slow paths.

**Constraints and Limitations:**
- `findBy*` queries do not work for asserting absence — use `queryBy*` with `waitFor`.
- `waitFor` retries on a 50ms interval by default; very fast updates may still require a small delay.
- Overusing `waitFor` can make tests slower; prefer `findBy*` for element appearance.
- The loading state may be too brief to catch synchronously if the API responds instantly; use MSW's `delay()` to control timing.

### Annotated Code Examples

**Example 1: Full State Transition — Loading, Data, Empty, Error**

```jsx
// DataView.jsx
import React, { useState, useEffect } from 'react';

export function DataView() {
  const [state, setState] = useState({ status: 'idle', data: [], error: null });

  async function fetchData() {
    setState({ status: 'loading', data: [], error: null });
    try {
      const res = await fetch('/api/items');
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      setState({
        status: data.length === 0 ? 'empty' : 'success',
        data,
        error: null,
      });
    } catch (err) {
      setState({ status: 'error', data: [], error: err.message });
    }
  }

  useEffect(() => { fetchData(); }, []);

  if (state.status === 'loading') return <p>Loading...</p>;
  if (state.status === 'error') return <p role="alert">Error: {state.error}</p>;
  if (state.status === 'empty') return <p>No items found.</p>;

  return (
    <ul>
      {state.data.map((item) => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

```jsx
// DataView.test.jsx
import { render, screen, waitForElementToBeRemoved } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { DataView } from './DataView';

test('shows loading, then data', async () => {
  server.use(
    http.get('/api/items', async () => {
      await delay(100);
      return HttpResponse.json([{ id: 1, name: 'Item A' }]);
    })
  );

  render(<DataView />);

  // Loading appears synchronously
  expect(screen.getByText('Loading...')).toBeInTheDocument();

  // Wait for loading to be removed
  await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

  // Data appears
  expect(screen.getByText('Item A')).toBeInTheDocument();
});

test('shows empty state when API returns empty array', async () => {
  server.use(
    http.get('/api/items', () => HttpResponse.json([]))
  );

  render(<DataView />);

  expect(await screen.findByText('No items found.')).toBeInTheDocument();
});

test('shows error banner on 500', async () => {
  server.use(
    http.get('/api/items', () => new HttpResponse(null, { status: 500 }))
  );

  render(<DataView />);

  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent(/error/i);
});
```

**Expected Output:** The first test passes: "Loading..." appears, then is removed, then "Item A" appears. The second test passes: "No items found." appears. The third test passes: an alert with an error message appears.

**Why This Output Occurs:** The `delay(100)` ensures the loading state is visible long enough to assert synchronously. `waitForElementToBeRemoved` waits for the loading text to disappear. `findByText` and `findByRole` wait for the empty and error states to appear. The per-test handler overrides control the API response for each scenario.

### Real-World Cases

- **User profile pages:** Testing loading skeleton → user data → error on 404.
- **Search results:** Testing loading → results → empty state → error banner.
- **Data tables:** Testing loading → rows → "No data" message → error alert.
- **Dashboard widgets:** Testing loading spinner → chart data → empty state → error fallback.
- **Infinite scroll:** Testing loading indicator → additional items → "No more items" message.

---

## Core Concept 3: Boundary & Failure Testing

### Definitions

**Core Definition:** Boundary and Failure Testing is the practice of intentionally throwing network and runtime errors during tests to verify that Error Boundaries and error banners render correctly instead of crashing the application.

**Technical Definition:** React Error Boundaries are class components that implement `getDerivedStateFromError` and/or `componentDidCatch` to catch errors in their child component tree. In tests, Error Boundaries are verified by rendering a child component that throws under controlled conditions (e.g., a `Bomb` component with a `shouldThrow` flag), then asserting that the fallback UI appears instead of the crashed component. For network errors, MSW handlers return 4xx/5xx responses or `HttpResponse.error()`, and the test asserts that an error banner or alert appears. Error Boundaries do not catch errors in event handlers, asynchronous code, or server-side rendering — these must be tested separately.

**Beginner-Friendly Explanation:** Your app has safety nets called Error Boundaries. If a component crashes, the Error Boundary shows a friendly message instead of a blank screen. To test this, you write a component that deliberately throws an error, wrap it in an Error Boundary, and check that the friendly message appears. You also test network errors: if the API returns a 500, your app should show an error banner, not crash. These tests ensure your error handling actually works when things go wrong.

### Purposes

- To verify that Error Boundaries catch rendering errors and display the fallback UI.
- To confirm that error banners appear when the API returns 4xx or 5xx responses.
- To test the "Try again" button resets the Error Boundary and re-renders children.
- To ensure that errors in one component do not crash the entire application.
- To validate that network errors (connection failures, timeouts) are handled gracefully.
- To test that error logging (e.g., `console.error`) is called when an error occurs.

### Syntax Rules and Structure

**General Syntax for Error Boundary Testing:**

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

let shouldThrow = false;

function Bomb() {
  if (shouldThrow) throw new Error('kaboom');
  return <div>Recovered content</div>;
}

test('ErrorBoundary catches render error and shows fallback', () => {
  shouldThrow = true;
  render(
    <ErrorBoundary>
      <Bomb />
    </ErrorBoundary>
  );

  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
  expect(screen.getByText('kaboom')).toBeInTheDocument();
});

test('re-renders children after "Try again"', async () => {
  shouldThrow = true;
  render(
    <ErrorBoundary>
      <Bomb />
    </ErrorBoundary>
  );

  shouldThrow = false;
  await userEvent.click(screen.getByRole('button', { name: /try again/i }));

  expect(screen.getByText('Recovered content')).toBeInTheDocument();
});
```

**Component Breakdown:**
- `Bomb`: A component that throws when `shouldThrow` is `true`.
- `shouldThrow = true`: Triggers the error condition before rendering.
- `screen.getByText(/something went wrong/i)`: Asserts that the fallback UI appears.
- `shouldThrow = false`: Clears the error condition before clicking "Try again".
- `screen.getByText('Recovered content')`: Asserts that the child re-renders successfully.

**General Syntax for Network Error Boundary Testing:**

```jsx
test('error banner appears on server error', async () => {
  server.use(
    http.get('/api/user', () => new HttpResponse(null, { status: 500 }))
  );

  render(<UserProfile />);

  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent(/failed to load/i);
});
```

**Component Breakdown:**
- `server.use(...)`: Overrides the handler to return a 500 response.
- `findByRole('alert')`: Waits for the error banner to appear.
- `toHaveTextContent(/failed to load/i)`: Asserts on the error message.

**Syntax Rules:**
- Use a controlled `Bomb` component with a `shouldThrow` flag to trigger errors deterministically.
- Spy on `console.error` in `beforeEach` and restore it in `afterEach` to suppress expected error noise.
- For Error Boundaries using `react-error-boundary`, the fallback component receives `error` and `resetErrorBoundary` props.
- Test both the error state and the recovery path (clicking "Try again").
- For network errors, use MSW to return 500, 404, or `HttpResponse.error()`.
- Assert that the error banner has `role="alert"` for accessibility.

**Constraints and Limitations:**
- Error Boundaries do not catch errors in event handlers, `setTimeout`, or `requestAnimationFrame` callbacks. Test these with `try`/`catch` in the handler and assert on the error state.
- Error Boundaries do not catch errors during server-side rendering.
- React logs caught errors to `console.error`; spying on it prevents test output noise.
- The `shouldThrow` flag must be reset between tests to avoid state leakage.
- `react-error-boundary`'s `ErrorBoundary` component re-throws errors if no fallback is provided.

### Annotated Code Examples

**Example 1: Error Boundary with Recovery**

```jsx
// ErrorBoundary.jsx
import React from 'react';

export class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Caught error:', error, errorInfo);
  }

  reset = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <p>{this.state.error.message}</p>
          <button onClick={this.reset}>Try again</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

```jsx
// ErrorBoundary.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ErrorBoundary } from './ErrorBoundary';

let shouldThrow = true;

function Bomb() {
  if (shouldThrow) throw new Error('kaboom in render');
  return <div>recovered content</div>;
}

beforeEach(() => {
  vi.spyOn(console, 'error').mockImplementation(() => {});
  shouldThrow = true;
});

afterEach(() => {
  vi.restoreAllMocks();
  shouldThrow = false;
});

test('renders children when nothing throws', () => {
  shouldThrow = false;
  render(
    <ErrorBoundary>
      <div>all good</div>
    </ErrorBoundary>
  );
  expect(screen.getByText('all good')).toBeInTheDocument();
});

test('catches a render error and shows the message', () => {
  render(
    <ErrorBoundary>
      <Bomb />
    </ErrorBoundary>
  );
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
  expect(screen.getByText('kaboom in render')).toBeInTheDocument();
  expect(console.error).toHaveBeenCalled();
});

test('re-renders the children after "Try again"', async () => {
  const user = userEvent.setup();
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

**Expected Output:** All three tests pass. The first test renders children normally. The second test catches the error and shows "Something went wrong" with "kaboom in render". The third test recovers after clicking "Try again" and shows "recovered content".

**Why This Output Occurs:** The `ErrorBoundary` class component implements `getDerivedStateFromError` to update state and render the fallback. The `Bomb` component throws when `shouldThrow` is `true`. The `console.error` spy suppresses the expected error log. The `reset` method clears the error state, allowing the children to re-render.

**Example 2: Network Error Banner Testing**

```jsx
// UserProfile.jsx
import React, { useState, useEffect } from 'react';

export function UserProfile() {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/user')
      .then((res) => {
        if (!res.ok) throw new Error('Failed to load user');
        return res.json();
      })
      .then(setUser)
      .catch((err) => setError(err.message));
  }, []);

  if (error) return <div role="alert">{error}</div>;
  if (!user) return <p>Loading...</p>;
  return <h2>{user.name}</h2>;
}
```

```jsx
// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { UserProfile } from './UserProfile';

test('shows error banner when API returns 500', async () => {
  server.use(
    http.get('/api/user', () => new HttpResponse(null, { status: 500 }))
  );

  render(<UserProfile />);

  const alert = await screen.findByRole('alert');
  expect(alert).toHaveTextContent('Failed to load user');
});
```

**Expected Output:** The test passes. The error banner with "Failed to load user" appears.

**Why This Output Occurs:** The MSW handler override returns a 500 response. The component's `fetch` rejects with an error, which is caught and stored in the `error` state. The component renders the `role="alert"` div with the error message. `findByRole('alert')` waits for the alert to appear.

### Real-World Cases

- **E-commerce checkout:** Testing that a failed payment API shows an error banner and allows retry.
- **Social media feeds:** Testing that a failed post fetch shows an error message without crashing the feed.
- **Dashboard widgets:** Testing that a failed chart data fetch shows a widget-level error fallback.
- **Authentication:** Testing that a failed login shows an inline error message.
- **Multi-step forms:** Testing that a failed step submission shows an error and allows the user to retry.

---

## Core Concept 4: Dependency & Asset Mocking

### Definitions

**Core Definition:** Dependency and Asset Mocking is the practice of replacing third-party modules, tracking function calls with spies, and stubbing out non-JavaScript assets (CSS, images, fonts) so that tests focus on component logic rather than external dependencies or file resolution.

**Technical Definition:** `vi.mock` (Vitest) and `jest.mock` (Jest) replace an entire module with a mock implementation. `vi.spyOn` and `jest.spyOn` wrap an existing method on an object, allowing you to track calls while optionally preserving the original implementation. For CSS and asset imports, Vitest and Jest need to be configured to ignore or mock files with extensions like `.css`, `.png`, `.jpg`, and `.svg` — otherwise, the test runner throws `Unknown file extension ".css"` because it tries to parse CSS as JavaScript. Vitest handles CSS and assets similarly to Vite: when using `jsdom` or `happy-dom`, it follows Vite's rules for CSS and asset imports. Configuration options include `test.css` for CSS modules and `test.alias` for stubbing asset imports.

**Beginner-Friendly Explanation:** Your components import things like CSS files, images, and third-party libraries. In a test environment, those imports can cause problems — Vitest doesn't know how to read a `.css` file, and you don't want to make real calls to a third-party API. So you mock them. You replace the third-party module with a fake version, you track whether a function was called using a spy, and you tell Vitest to ignore CSS and image imports. This keeps your tests focused on your component's logic.

### Purposes

- To replace third-party modules with controlled mock implementations.
- To track whether a function was called and with what arguments using spies.
- To stub out CSS and image imports so the test runner does not throw errors.
- To mock Next.js components (e.g., `next/image`) that rely on loaders not available in tests.
- To test component logic in isolation from external dependencies.
- To verify that callbacks (e.g., `onSubmit`, `onClick`) are called with the correct arguments.

### Syntax Rules and Structure

**General Syntax for Module Mocking (Vitest):**

```javascript
// vi.mock replaces the entire module
import { vi } from 'vitest';

vi.mock('./api-client', () => ({
  fetchUser: vi.fn().mockResolvedValue({ name: 'Alice' }),
}));

// vi.spyOn wraps an existing method
import * as apiClient from './api-client';

const spy = vi.spyOn(apiClient, 'fetchUser');
spy.mockResolvedValue({ name: 'Alice' });

// Assertions
expect(spy).toHaveBeenCalledTimes(1);
expect(spy).toHaveBeenCalledWith('user-123');
```

**Component Breakdown:**
- `vi.mock('./api-client', factory)`: Replaces the module with the factory result.
- `vi.fn().mockResolvedValue(...)`: Creates a mock function that resolves with a value.
- `vi.spyOn(obj, 'method')`: Wraps the method, preserving the original implementation by default.
- `spy.mockResolvedValue(...)`: Overrides the spy's return value.
- `toHaveBeenCalledWith(...)`: Asserts on the arguments passed to the spy.

**General Syntax for CSS and Asset Mocking (Vitest):**

```javascript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    // Option 1: Configure CSS handling
    css: {
      modules: {
        classNameStrategy: 'non-scoped',
      },
    },
    // Option 2: Alias asset imports to a mock module
    alias: {
      '\\.(css|less|scss|sass|styl)$': 'identity-obj-proxy',
      '\\.(png|jpg|jpeg|gif|svg|webp|avif)$': 'vitest-asset-mock',
    },
  },
});
```

**Component Breakdown:**
- `css.modules.classNameStrategy`: Controls how CSS module class names are generated.
- `alias`: Maps asset imports to mock modules.
- `identity-obj-proxy`: Returns the class name as-is (useful for CSS modules).
- `vitest-asset-mock`: A simple mock that returns a placeholder string for image imports.

**General Syntax for Mocking Next.js Components:**

```javascript
vi.mock('next/image', () => ({
  __esModule: true,
  default: (props) => <img {...props} />,
}));

vi.mock('next/link', () => ({
  __esModule: true,
  default: ({ children, href }) => <a href={href}>{children}</a>,
}));
```

**Component Breakdown:**
- `vi.mock('next/image', factory)`: Replaces the Next.js Image component with a standard `<img>`.
- `__esModule: true`: Indicates that the module uses ES module syntax.
- `default: (props) => <img {...props} />`: The mock implementation.

**Syntax Rules:**
- Use `vi.mock` (Vitest) or `jest.mock` (Jest) for module-level mocking. The factory function is hoisted to the top of the file.
- Use `vi.spyOn` when you want to track calls to an existing method without replacing the entire module.
- Configure CSS and asset mocking in `vitest.config.ts` or `jest.config.ts` to avoid `Unknown file extension` errors.
- Use `identity-obj-proxy` for CSS modules so that `styles.container` returns `'container'`.
- Mock Next.js components (`next/image`, `next/link`) that rely on loaders or browser APIs not available in tests.
- Always restore mocks in `afterEach` with `vi.restoreAllMocks()` or `jest.restoreAllMocks()`.

**Constraints and Limitations:**
- `vi.mock` is hoisted to the top of the file; variables referenced in the factory must be prefixed with `mock` (Vitest) or defined inside the factory.
- `vi.mock` does not support mocking modules imported with `require()`.
- CSS mocking does not test actual styles — it only prevents import errors.
- Mocking `next/image` replaces the component with a standard `<img>`, so tests do not verify Next.js-specific image optimisation.
- Spy functions must be restored to avoid leaking state between tests.

### Annotated Code Examples

**Example 1: Mocking a Third-Party Module and Spying on a Callback**

```jsx
// analytics.js — third-party module
export function trackEvent(eventName, properties) {
  // Sends data to analytics service
}

// Button.jsx
import { trackEvent } from './analytics';

export function Button({ onClick, children }) {
  function handleClick() {
    trackEvent('button_click', { label: children });
    onClick?.();
  }
  return <button onClick={handleClick}>{children}</button>;
}
```

```jsx
// Button.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';
import * as analytics from './analytics';
import { Button } from './Button';

// ✅ Spy on the analytics module
const trackSpy = vi.spyOn(analytics, 'trackEvent').mockImplementation(() => {});

test('calls analytics and onClick when clicked', async () => {
  const user = userEvent.setup();
  const handleClick = vi.fn();

  render(<Button onClick={handleClick}>Submit</Button>);

  await user.click(screen.getByRole('button', { name: /submit/i }));

  expect(trackSpy).toHaveBeenCalledWith('button_click', { label: 'Submit' });
  expect(handleClick).toHaveBeenCalledTimes(1);
});

afterEach(() => {
  vi.restoreAllMocks();
});
```

**Expected Output:** The test passes. `trackEvent` is called with `('button_click', { label: 'Submit' })`, and `onClick` is called once.

**Why This Output Occurs:** `vi.spyOn` wraps `trackEvent` and replaces its implementation with a no-op. The spy records the call and its arguments. `vi.fn()` creates a mock callback for `onClick`. The test asserts on both the analytics call and the callback call, verifying the component's complete behaviour.

**Example 2: Mocking CSS and Image Imports in Vitest**

```javascript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    alias: {
      '\\.(css|less|scss|sass|styl)$': 'identity-obj-proxy',
      '\\.(png|jpg|jpeg|gif|svg|webp|avif)$': 'vitest-asset-mock',
    },
  },
});
```

```jsx
// Card.jsx
import styles from './Card.module.css';
import logo from './logo.png';

export function Card({ title }) {
  return (
    <div className={styles.card}>
      <img src={logo} alt="Logo" />
      <h2>{title}</h2>
    </div>
  );
}
```

```jsx
// Card.test.jsx
import { render, screen } from '@testing-library/react';
import { Card } from './Card';

test('renders card with title and image', () => {
  render(<Card title="Hello" />);

  expect(screen.getByRole('heading', { name: /hello/i })).toBeInTheDocument();
  expect(screen.getByRole('img', { name: /logo/i })).toBeInTheDocument();
});
```

**Expected Output:** The test passes without any CSS or image import errors. The heading "Hello" and the image with alt text "Logo" are found.

**Why This Output Occurs:** The `alias` configuration maps CSS imports to `identity-obj-proxy` (which returns the class name as a string) and image imports to `vitest-asset-mock` (which returns a placeholder). The component renders normally, and the test queries the heading and image by their accessible roles.

### Real-World Cases

- **Analytics tracking:** Spying on `trackEvent` to verify that user interactions are logged correctly.
- **Feature flags:** Mocking a feature flag module to test both enabled and disabled states.
- **Next.js applications:** Mocking `next/image` and `next/link` to avoid loader-related errors.
- **CSS modules:** Using `identity-obj-proxy` so that `styles.card` returns `'card'` in tests.
- **Image imports:** Stubbing image imports so the test runner does not try to parse binary files.
- **Third-party SDKs:** Mocking payment SDKs (Stripe, PayPal) to test form submission without real transactions.

---

## References

- Example – Testing Library: https://testing-library.com/docs/react-testing-library/example-intro
- Async Utilities Reference – itechmeat/llm-code: https://github.com/itechmeat/llm-code/blob/HEAD/skills/react-testing-library/references/async.md
- Error Boundary Test Example – Hugging Face: https://huggingface.co/spaces/Ma-Ri-Ba-Ku/alto-llm-corrector/blob/a94837f5c02ea105429f5c9a7d7e5f6e821113f6/frontend/src/components/ErrorBoundary.test.tsx
- CSS Mocking in Vitest – Code Examples: https://code-examples.net/en/q/4be7c4e/css-mocking-in-vitest-a-guide-to-handling-mui-component-imports-in-react-vite-testing
- Mock Service Worker (MSW) – Official Documentation: https://mswjs.io/docs/
- MSW v2 Integration with React Testing Library – GitHub Discussion: https://github.com/vercel/next.js/discussions/77373
- Vitest – Test Environment Configuration: https://vitest.dev/config/environment
- Vitest – Mocking Modules: https://vitest.dev/guide/mocking/modules
- React Testing Library – Async Utilities: https://testing-library.com/docs/dom-testing-library/api-async
- React Testing Library – renderHook API: https://testing-library.com/docs/react-testing-library/api#renderhook
- Kent C. Dodds – The Testing Trophy and Testing Classifications: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Steve Kinney – FireEvent vs UserEvent in Testing: https://stevekinney.com/courses/testing/user-event
- Testing Library – Guiding Principles: https://testing-library.com/docs/guiding-principles
- Vitest – Coverage Configuration: https://vitest.dev/config/#coverage
- MSW – Emulate Network Errors: https://egghead.io/lessons/msw-emulate-network-errors-in-msw
- Jest – Mock Functions: https://jestjs.io/docs/mock-functions
- Vitest – Mocking: https://vitest.dev/guide/mocking