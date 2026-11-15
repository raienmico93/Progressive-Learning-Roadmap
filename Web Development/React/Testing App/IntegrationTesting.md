# Integration Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Integration testing is the practice of verifying that multiple units of a software system — components, modules, services, or layers — work together correctly as a combined whole, rather than in isolation.

**Technical Definition:** Integration testing exercises the **seams** between software units: the interfaces, data contracts, and interaction protocols through which components collaborate. In a React application, integration tests render a component tree (multiple components, context providers, state stores) using React Testing Library, mock network boundaries at the HTTP layer with Mock Service Worker (MSW), and assert that the composed system produces the correct user-visible behavior. Unlike unit tests, which replace all collaborators with test doubles, integration tests use **real** application code for internal modules and mock only genuine external boundaries (network, time, browser APIs). Integration tests sit in the middle of the testing pyramid — more realistic than unit tests, faster and less brittle than end-to-end tests.

**Beginner-Friendly Explanation:** Integration testing checks that different parts of your app work together. Instead of testing one function alone, you test a whole feature — like a login form that calls an API and updates the screen — to make sure the pieces fit together properly.

### Key Characteristics

- **Seam-focused:** Targets the boundaries and contracts between units, not the internals of each unit.
- **Real internal code, mocked external boundaries:** Uses real components, hooks, context, and state logic; mocks only network, time, and other genuine external systems.
- **User-visible assertions:** Verifies observable behavior (what the user sees and does), not internal state or implementation details.
- **Moderate speed and cost:** Slower than unit tests (seconds), much faster than E2E tests (tens of seconds).
- **High confidence per test:** Catches bugs that unit tests miss — wiring errors, contract mismatches, provider integration failures.

### Prerequisites

- React Testing Library (`@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom`).
- Mock Service Worker (MSW) for network-level API mocking.
- A test runner (Vitest recommended for new projects in 2026; Jest for existing projects).
- Familiarity with React Router's `MemoryRouter`, Redux/state management test utilities, and provider wrappers.
- Understanding of the testing pyramid and the distinction between unit, integration, and E2E tests.

### Related Programming Areas

- **Unit testing** — the layer below; tests individual units in isolation.
- **End-to-end (E2E) testing** — the layer above; drives a real browser through full user journeys.
- **Component testing** — a subset of integration testing that focuses on a single component with its dependencies.
- **Test doubles** — mocks, stubs, spies, and fakes used at integration boundaries.
- **State management** — Redux, Zustand, React Query, and Context providers integrated into the test tree.

### Core Concepts / Features

---

## 1. Integration Testing

### Definitions

**Core Definition:** Integration testing verifies that multiple software units — components, modules, or layers — collaborate correctly when combined.

**Technical Definition:** Integration tests render a meaningful slice of the application (a feature, a page, or a component tree) with its real internal dependencies (components, hooks, context, state stores) and controlled external dependencies (network mocked via MSW, time via fake timers). The test then simulates user interactions and asserts that the integrated system produces the expected UI state. Integration tests form 25–35% of a healthy React test suite, sitting between unit tests (50–60%) and E2E tests (10–15%). The Redux team explicitly recommends integration-style tests: "Prefer writing integration tests with everything working together" — render a `<Provider>` with a real store, use real Redux logic, and mock only API calls.

**Beginner-Friendly Explanation:** Integration testing checks that a whole feature works — not just one function. You put the real components together, give them a fake API, and check that clicking and typing produces the right result.

### Purposes

- To verify that multiple units collaborate correctly across their interfaces.
- To catch wiring, contract, and integration bugs that unit tests miss.
- To test realistic user flows (forms, routing, data fetching) without the cost of a full browser.
- To validate that context providers, state stores, and routing are correctly connected.
- To provide high confidence per test at moderate speed.

### Syntax Rules and Structure

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect } from 'vitest';
import { renderWithProviders } from './test-utils';
import { FeaturePage } from './FeaturePage';

test('integration test description', async () => {
  const user = userEvent.setup();

  // Arrange: render the feature with providers and mocked API
  renderWithProviders(<FeaturePage />);

  // Act: interact with the integrated UI
  await user.click(screen.getByRole('button', { name: /load data/i }));

  // Assert: verify the integrated system's visible output
  expect(await screen.findByText('Loaded item')).toBeInTheDocument();
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `renderWithProviders` | Custom render that wraps the UI in all required providers (Router, Redux, Theme, QueryClient). |
| Real internal modules | The actual components, hooks, and state logic — not mocked. |
| MSW handlers | Mock network responses at the HTTP level, intercepting real `fetch` calls. |
| `user-event` | Simulates realistic user interactions. |
| `findBy*` | Waits for async updates triggered by the integrated data flow. |

**Syntax Rules:**

1. Use real internal code — do not mock your own modules unless they are genuine external dependencies.
2. Mock network requests with MSW, not by mocking `fetch` or `axios` directly.
3. Render with a custom `renderWithProviders` that includes all app-wide providers.
4. Assert on user-visible output, not internal state or implementation details.
5. Use `findBy*` and `waitFor` for asynchronous data flows.

**Constraints and Limitations:**

- Integration tests are slower than unit tests — do not test every edge case at this level; reserve them for meaningful collaborations.
- Over-mocking defeats the purpose — if you mock your own modules, you are writing a unit test, not an integration test.
- Integration tests do not replace E2E tests for critical user journeys — they run in jsdom, not a real browser.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Feature Integration Test

```jsx
// feature.jsx — real internal components
export function FeaturePage() {
  const [data, setData] = useState(null);

  async function loadData() {
    const res = await fetch('/api/items');
    setData(await res.json());
  }

  return (
    <div>
      <button onClick={loadData}>Load Data</button>
      {data && <ul>{data.map((item) => <li key={item.id}>{item.name}</li>)}</ul>}
    </div>
  );
}

// feature.test.jsx — integration test
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { FeaturePage } from './feature';

test('loads and displays items from the API', async () => {
  const user = userEvent.setup();

  server.use(
    http.get('/api/items', () =>
      HttpResponse.json([{ id: 1, name: 'Keyboard' }, { id: 2, name: 'Mouse' }])
    )
  );

  render(<FeaturePage />);
  await user.click(screen.getByRole('button', { name: /load data/i }));

  expect(await screen.findByText('Keyboard')).toBeInTheDocument();
  expect(screen.getByText('Mouse')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. Clicking the button triggers a real `fetch` call, MSW intercepts it at the network layer, and the UI renders the returned items.

**Why this output:** No component is mocked. The real `FeaturePage` component, its real `fetch` call, and its real state update are exercised. MSW intercepts the HTTP request at the boundary, simulating the server. This is a true integration test.

### Real-World Cases

- **Feature slices:** Testing an entire feature (e.g., a shopping cart) with its components, context, and API integration.
- **Page-level testing:** Rendering a full page with routing, state, and data fetching — the approach recommended for client-heavy applications.
- **Redux-connected components:** Rendering a `<Provider>` with a real store and testing the full interaction flow.

### References

- Testing Strategies for React and Next.js Applications — https://github.com/alirezarezvani/claude-skills/blob/9ca476d8401f8750f92f4a7b31ec68efaa61ab02/engineering-team/skills/senior-qa/references/testing_strategies.md
- The Modern Frontend Testing Pyramid & Strategy — https://feature-sliced.design/ja/blog/frontend-testing-strategy
- Redux — Writing Tests: Guiding Principles — https://redux.js.org/usage/writing-tests

---

## 2. Testing Component Interactions

### Definitions

**Core Definition:** Testing component interactions verifies that when one component communicates with another — through props, callbacks, context, or events — the combined behavior is correct.

**Technical Definition:** Component interaction testing renders a parent-child component tree (or a set of sibling components sharing state or context) and exercises the communication paths between them. This includes prop drilling, callback invocation, context propagation, and custom event dispatch. The test simulates a user action on one component and asserts that the resulting update propagates correctly to another. Interaction tests often use `within()` to scope queries to specific subtrees and `user-event` to simulate realistic interactions.

**Beginner-Friendly Explanation:** Component interaction testing checks that different components talk to each other correctly. For example, if a user clicks a button in one component, does the correct data appear in another component?

### Purposes

- To verify that parent-child component communication works (props down, callbacks up).
- To confirm that context values propagate correctly through the component tree.
- To test that sibling components sharing state through a parent or store update correctly.
- To catch wiring bugs that unit tests miss because each component works in isolation but fails in composition.

### Syntax Rules and Structure

```jsx
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('child updates when parent state changes', async () => {
  const user = userEvent.setup();
  render(<ParentWithChildComponents />);

  await user.click(screen.getByRole('button', { name: /update/i }));

  const childRegion = screen.getByRole('region', { name: /child/i });
  expect(within(childRegion).getByText('Updated')).toBeInTheDocument();
});
```

**Component Breakdown:**

| Concern | Approach |
|---------|----------|
| Prop communication | Render parent with child; assert that prop values appear in the child. |
| Callback communication | Pass a `vi.fn()` from parent to child; interact with child; assert the callback fired. |
| Context communication | Wrap the tree in a provider; assert that the consumer reads the value. |
| Sibling communication | Render the shared parent; interact with one child; assert the other child updates. |

**Constraints and Limitations:**

- Do not mock child components — that defeats the purpose of interaction testing.
- Use `within()` to scope assertions when multiple similar elements exist.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Parent-Child Interaction Through Context

```jsx
// ThemeToggle.jsx — child that updates context
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

export function ThemeToggle() {
  const { theme, toggle } = useContext(ThemeContext);
  return <button onClick={toggle}>Theme: {theme}</button>;
}

// ThemedCard.jsx — child that reads context
export function ThemedCard() {
  const { theme } = useContext(ThemeContext);
  return <div className={`card-${theme}`}>Card content</div>;
}

// ThemePage.jsx — parent providing context
export function ThemePage() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, toggle: () => setTheme(t => t === 'light' ? 'dark' : 'light') }}>
      <ThemeToggle />
      <ThemedCard />
    </ThemeContext.Provider>
  );
}

// ThemePage.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect } from 'vitest';
import { ThemePage } from './ThemePage';

test('toggling the theme updates both components', async () => {
  const user = userEvent.setup();
  render(<ThemePage />);

  expect(screen.getByText('Theme: light')).toBeInTheDocument();
  expect(screen.getByText('Card content')).toHaveClass('card-light');

  await user.click(screen.getByRole('button', { name: /theme/i }));

  expect(screen.getByText('Theme: dark')).toBeInTheDocument();
  expect(screen.getByText('Card content')).toHaveClass('card-dark');
});
```

**Expected Output:** The test passes. Clicking the toggle updates the context, and both the toggle button and the card reflect the new theme.

### Real-World Cases

- **Form wizards:** A multi-step form where one step's inputs affect the next step's display.
- **Filter panels:** A filter component updates a results list component through shared state.
- **Theme switchers:** A toggle updates the entire UI through context.

### References

- Testing Studio React Components — https://www.sanity.io/learn/course/testing-sanity-studio/testing-studio-react-component
- Testing component interactions with RTL — https://github.com/alirezarezvani/claude-skills/blob/9ca476d8401f8750f92f4a7b31ec68efaa61ab02/engineering-team/skills/senior-qa/references/testing_strategies.md

---

## 3. Testing Form Submissions

### Definitions

**Core Definition:** Testing form submissions verifies that a form captures user input, validates it, submits it to the correct handler or API, and displays the appropriate success or error state.

**Technical Definition:** A form submission integration test renders the form component (with any real validation libraries like React Hook Form or TanStack Form), types into the fields using `user-event`, clicks the submit button, and asserts on the resulting UI — a success message, an error alert, a disabled button during submission, or a navigation. Network calls triggered by submission are mocked with MSW. The test covers the full flow: input → validation → submission → response handling.

**Beginner-Friendly Explanation:** Form submission testing checks that when a user fills out a form and clicks submit, everything works — the data goes to the right place, and the user sees the right result (success or error).

### Purposes

- To verify that form inputs are captured correctly and passed to the submit handler or API.
- To confirm that validation errors appear and prevent submission when appropriate.
- To test that the submit button is disabled during submission.
- To verify success and error states after the API responds.

### Syntax Rules and Structure

```jsx
const user = userEvent.setup();
render(<RegistrationForm onSubmit={mockSubmit} />);

await user.type(screen.getByLabelText(/email/i), 'test@example.com');
await user.click(screen.getByRole('button', { name: /submit/i }));

await waitFor(() => {
  expect(mockSubmit).toHaveBeenCalledWith({ email: 'test@example.com' });
});
```

**Component Breakdown:**

| Step | Description |
|------|-------------|
| Fill fields | `user.type` into labeled inputs. |
| Click submit | `user.click` the submit button. |
| Assert call | `waitFor` the submit handler or MSW-mocked API call. |
| Assert UI | Check for success message, error alert, or navigation. |

**Syntax Rules:**

1. Use `getByLabelText` to find form fields by their labels.
2. Use `user-event` (not `fireEvent`) for realistic typing and clicking.
3. Use `waitFor` or `findBy*` for async submission results.
4. Mock the API at the network layer with MSW — do not mock the form's submit handler unless testing the handler in isolation.
5. Test validation errors by submitting invalid input and asserting on `role="alert"` elements.

**Constraints and Limitations:**

- Do not assert on internal form state (e.g., `form.values.email`) — assert on user-visible outcomes.
- Async validation requires `waitFor` or `findBy*`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Full Form Submission with MSW

```jsx
// RegistrationForm.jsx
export function RegistrationForm({ onSubmit }) {
  const [status, setStatus] = useState('idle');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    const data = new FormData(e.target);
    try {
      await onSubmit(Object.fromEntries(data));
      setStatus('success');
    } catch {
      setStatus('error');
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" type="email" required />
      <button type="submit" disabled={status === 'submitting'}>
        {status === 'submitting' ? 'Submitting...' : 'Register'}
      </button>
      {status === 'success' && <p>Registration successful!</p>}
      {status === 'error' && <p role="alert">Registration failed.</p>}
    </form>
  );
}

// RegistrationForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { test, expect } from 'vitest';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { RegistrationForm } from './RegistrationForm';

test('submits the form and shows success', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn().mockResolvedValue(undefined);

  server.use(
    http.post('/api/register', () => HttpResponse.json({ success: true }))
  );

  render(<RegistrationForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
  await user.click(screen.getByRole('button', { name: /register/i }));

  expect(await screen.findByText('Registration successful!')).toBeInTheDocument();
  expect(onSubmit).toHaveBeenCalledWith({ email: 'alice@example.com' });
});
```

**Expected Output:** The test passes. The form captures the email, calls the submit handler, and displays the success message after the mocked API responds.

### Real-World Cases

- **Login and signup forms:** Testing the full authentication flow with mocked API responses.
- **Checkout forms:** Verifying that shipping and payment forms submit correct data.
- **Profile editing:** Testing that updated user data is submitted and the UI reflects the change.

### References

- Testing TanStack Form — https://github.com/fusengine/agents/blob/main/plugins/nextjs-expert/skills/nextjs-tanstack-form/references/testing.md
- How to mock API requests in React tests — https://coreui.io/answers/how-to-mock-api-requests-in-react-tests/

---

## 4. Testing Routing

### Definitions

**Core Definition:** Testing routing verifies that navigation between routes renders the correct components, that route parameters and query strings are handled correctly, and that programmatic navigation works.

**Technical Definition:** React Router relies on context, so components that use `useNavigate`, `useParams`, `Link`, or `Route` must be wrapped in a router during testing. `MemoryRouter` is the standard choice for tests because it stores navigation history in memory without depending on the browser URL. The `initialEntries` prop sets the starting route, enabling tests to verify route matching, parameter extraction, and conditional rendering. For data routers, `createMemoryRouter` is used instead. Programmatic navigation can be verified by mocking `useNavigate` or by asserting on the rendered output after a navigation-triggering interaction.

**Beginner-Friendly Explanation:** Routing tests check that when you navigate to a URL like `/users/123`, the correct page shows up, and that clicking links or buttons takes you to the right place.

### Purposes

- To verify that each route renders the correct component.
- To test that route parameters (`:id`) and query strings are read correctly.
- To confirm that navigation (via `Link`, `useNavigate`, or form submission) changes the rendered UI.
- To test protected routes and redirects.

### Syntax Rules and Structure

```jsx
import { MemoryRouter, Routes, Route } from 'react-router-dom';

render(
  <MemoryRouter initialEntries={['/users/123']}>
    <Routes>
      <Route path="/users/:id" element={<UserProfile />} />
    </Routes>
  </MemoryRouter>
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `MemoryRouter` | In-memory router for tests — no browser dependency. |
| `initialEntries` | Array of initial route paths. |
| `Routes` / `Route` | Define the route tree. |
| `useParams` | Reads the `:id` from the route. |

**Syntax Rules:**

1. Always wrap routing components in a router (`MemoryRouter` for tests).
2. Use `initialEntries` to set the starting URL.
3. Mock `useNavigate` with `vi.mock` when verifying programmatic navigation calls.
4. For data routers, use `createMemoryRouter` with route objects.

**Constraints and Limitations:**

- `MemoryRouter` does not update the real browser URL — that is intentional for tests.
- Do not mock React Router's internals — use the real `MemoryRouter` and assert on rendered output.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Route Parameters and Navigation

```jsx
// UserProfile.jsx
import { useParams } from 'react-router-dom';

export function UserProfile() {
  const { id } = useParams();
  return <h1>User ID: {id}</h1>;
}

// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Routes, Route } from 'react-router-dom';
import { test, expect } from 'vitest';
import { UserProfile } from './UserProfile';

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

**Expected Output:** The test passes. `MemoryRouter` starts at `/users/42`, the route matches, `useParams` extracts `id = "42"`, and the component renders it.

### Real-World Cases

- **User profile pages:** Testing that `/users/:id` renders the correct user's data.
- **Protected routes:** Testing that unauthenticated users are redirected to login.
- **Navigation menus:** Testing that clicking a `NavLink` renders the target page.

### References

- MemoryRouter — React Router — https://reactrouter.com/6.30.3/router-components/memory-router
- What are the considerations for testing components that utilize React Router — https://www.edureka.co/community/309032/considerations-testing-components-utilize-router-navigation

---

## 5. Testing Data Fetching

### Definitions

**Core Definition:** Testing data fetching verifies that a component or feature correctly requests data from an API, handles loading and error states, and renders the fetched data.

**Technical Definition:** Data-fetching integration tests render the component that performs the fetch, mock the HTTP layer with MSW, and assert on the loading state, the rendered data, and the error state. MSW intercepts actual network requests (via a Service Worker in the browser or an interceptor in Node.js) and returns controlled responses, allowing the real `fetch` call to execute while the server is simulated. This is preferred over mocking `fetch` or `axios` directly because it exercises the real data-fetching logic and HTTP client configuration.

**Beginner-Friendly Explanation:** Data fetching tests check that when your component asks the server for data, it shows a loading spinner, then displays the data correctly — or shows an error if something goes wrong. A tool called MSW fakes the server so no real network call is made.

### Purposes

- To verify that the correct API endpoint is called with the correct parameters.
- To confirm that loading, success, and error states render correctly.
- To test that the fetched data is transformed and displayed as expected.
- To catch bugs in data-fetching logic without depending on a real backend.

### Syntax Rules and Structure

```jsx
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';

server.use(
  http.get('/api/users/1', () =>
    HttpResponse.json({ id: 1, name: 'Alice' })
  )
);

render(<UserProfile userId="1" />);
expect(screen.getByText('Loading...')).toBeInTheDocument();
expect(await screen.findByText('Alice')).toBeInTheDocument();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| MSW `http.get` handler | Intercepts the request at the network layer. |
| `HttpResponse.json` | Returns a controlled JSON response. |
| `render(<Component />)` | Renders the component that performs the fetch. |
| `findBy*` | Waits for the async data to appear. |

**Syntax Rules:**

1. Use MSW as the single source of truth for API mocking — do not mock `fetch` or `axios` directly.
2. Define handlers per test (`server.use`) or in a shared `handlers.js` file.
3. Assert on the loading state first, then use `findBy*` for the resolved data.
4. Test error states by returning error responses (`HttpResponse.json({}, { status: 500 })`).
5. Do not mock the component's data-fetching hook — let the real hook run with MSW providing the data.

**Constraints and Limitations:**

- MSW handlers must match the exact request (method + URL) for the mock to apply.
- For GraphQL, MSW provides `graphql.query` and `graphql.mutation` handlers.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing Loading, Success, and Error States

```jsx
// UserProfile.jsx
export function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(setUser)
      .catch((err) => setError(err.message))
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (error) return <p role="alert">Error: {error}</p>;
  return <h1>{user.name}</h1>;
}

// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { test, expect } from 'vitest';
import { UserProfile } from './UserProfile';

test('shows loading then user data', async () => {
  server.use(
    http.get('/api/users/1', () => HttpResponse.json({ name: 'Alice' }))
  );

  render(<UserProfile userId="1" />);
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});

test('shows an error when the API fails', async () => {
  server.use(
    http.get('/api/users/1', () => HttpResponse.json({}, { status: 500 }))
  );

  render(<UserProfile userId="1" />);
  expect(await screen.findByRole('alert')).toHaveTextContent('Error');
});
```

**Expected Output:** The first test passes with the loading state followed by "Alice". The second passes with the error alert.

### Real-World Cases

- **Dashboard widgets:** Each widget fetches its own data; MSW provides per-widget responses.
- **Search results:** Testing that the search query is sent to the API and results render.
- **Pagination:** Testing that page changes trigger new API calls and render the correct page of data.

### References

- How to mock API requests in React tests — https://coreui.io/answers/how-to-mock-api-requests-in-react-tests/
- Using Mock Service Worker to Improve Jest Unit Tests — https://www.wwt.com/article/using-mock-service-worker-to-improve-jest-unit-tests
- Cart flow (MSW integration example) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/vitest/project-cart-flow.md

---

## 6. Testing State Management

### Definitions

**Core Definition:** Testing state management integration verifies that a component correctly reads from and writes to an external state store (Redux, Zustand, React Query, or Context) when rendered with a real store instance.

**Technical Definition:** The Redux team's recommended approach is to render the component inside a real `<Provider>` with a real store instance, use real Redux logic (reducers, selectors, thunks), mock only API calls, and assert that the UI updates appropriately when state changes. The store is created fresh per test (or per test suite) with a `preloadedState` to set up specific state scenarios. Selectors and reducers are **not** mocked — they are part of the integration under test. For Zustand, the store is reset between tests; for React Query, a fresh `QueryClient` with retries disabled is provided.

**Beginner-Friendly Explanation:** State management testing checks that your components correctly use the global state — reading values from the store, updating them through actions, and re-rendering when the state changes. You use a real store in the test, not a fake one.

### Purposes

- To verify that components correctly read state from the store.
- To confirm that user interactions dispatch the correct actions and update the store.
- To test that selectors derive the correct values from the state.
- To ensure that the UI re-renders when the store state changes.

### Syntax Rules and Structure

```jsx
import { render } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import reducer from './reducer';

const store = configureStore({
  reducer,
  preloadedState: { users: [{ id: 1, name: 'Alice' }] },
});

render(
  <Provider store={store}>
    <Users />
  </Provider>
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Provider` | Wraps the component with the Redux store. |
| `configureStore` | Creates a real store with the app's real reducer. |
| `preloadedState` | Sets up the initial state for the test scenario. |
| Real reducer/selectors | Not mocked — the integration under test. |

**Syntax Rules:**

1. Use a real store instance with the app's real reducers.
2. Do not mock selectors or React-Redux hooks — mocking library internals is fragile and provides false confidence.
3. Use `preloadedState` to set up specific state scenarios.
4. Create a fresh store per test or test suite to ensure isolation.
5. Assert on the rendered UI, not on the store state directly.

**Constraints and Limitations:**

- Redux store state is an implementation detail — assert on what the user sees, not on `store.getState()`.
- For React Query, disable retries in tests to avoid flaky async behavior.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Redux-Connected Component

```jsx
// Users.jsx
import { useSelector, useDispatch } from 'react-redux';
import { addUser } from './usersSlice';

export function Users() {
  const users = useSelector((state) => state.users.items);
  const dispatch = useDispatch();

  return (
    <div>
      <button onClick={() => dispatch(addUser({ id: 2, name: 'Bob' }))}>
        Add Bob
      </button>
      <ul>{users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>
    </div>
  );
}

// Users.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { test, expect } from 'vitest';
import reducer from './usersSlice';
import { Users } from './Users';

test('adds a user to the store and renders it', async () => {
  const user = userEvent.setup();
  const store = configureStore({
    reducer: { users: reducer },
    preloadedState: { users: { items: [{ id: 1, name: 'Alice' }] } },
  });

  render(
    <Provider store={store}>
      <Users />
    </Provider>
  );

  expect(screen.getByText('Alice')).toBeInTheDocument();

  await user.click(screen.getByRole('button', { name: /add bob/i }));

  expect(screen.getByText('Bob')).toBeInTheDocument();
});
```

**Expected Output:** The test passes. "Alice" renders from the preloaded state; clicking "Add Bob" dispatches the action, updates the store, and "Bob" appears in the UI.

### Real-World Cases

- **Todo lists:** Testing that adding, toggling, and deleting todos updates the UI through the real store.
- **Shopping carts:** Testing that cart state updates propagate to the cart badge and summary.
- **User authentication:** Testing that login and logout actions update the UI across components.

### References

- Writing Tests — Redux — https://redux.js.org/usage/writing-tests
- @suerg/test-kit — https://www.npmjs.com/package/@suerg/test-kit
- @render-with/redux — https://www.npmjs.com/package/@render-with/redux

---

## 7. Testing with Real Dependencies

### Definitions

**Core Definition:** Testing with real dependencies means using the actual implementations of internal modules, state stores, routers, and context providers in integration tests, while mocking only genuine external boundaries (network, time, browser APIs).

**Technical Definition:** The distinction between unit and integration testing hinges on dependency treatment. Unit tests replace all collaborators with test doubles. Integration tests use **real** internal dependencies — real components, real hooks, real reducers, real selectors, real routers — and mock only **external** systems that the application does not own: HTTP servers (via MSW), the system clock (via fake timers), and browser APIs that jsdom does not implement. The guiding principle is: "Do not mock your own modules unless the module is a genuine external dependency". The Redux team reinforces this: "Prefer writing integration tests with everything working together".

**Beginner-Friendly Explanation:** Integration tests use your real code — your real components, your real store, your real router — and fake only things outside your app, like the server. This gives you confidence that your actual code works together, not just a bunch of fakes.

### Purposes

- To maximize confidence by exercising real application code.
- To catch integration bugs that unit tests miss because they mock too much.
- To reduce test maintenance by avoiding mocks that drift from real interfaces.
- To test the same code paths that run in production (minus the network).

### Syntax Rules and Structure

**Real dependencies (use as-is):**

| Dependency | Treatment |
|------------|-----------|
| Components | Real |
| Custom hooks | Real |
| Redux reducers/selectors | Real |
| React Router | Real (`MemoryRouter`) |
| Context providers | Real (with controlled values) |
| React Query | Real (with a fresh `QueryClient`) |

**Mocked dependencies (test doubles):**

| Dependency | Treatment |
|------------|-----------|
| HTTP server | MSW (network-level mock) |
| System time | Fake timers (`vi.useFakeTimers()`) |
| Browser APIs (e.g., `localStorage`) | Stub or fake |
| Third-party services (e.g., Stripe) | Mock at the boundary |

**Syntax Rules:**

1. Render with a `renderWithProviders` helper that includes all real app-wide providers (Router, Redux, Theme, QueryClient).
2. Use MSW handlers for network mocking — not `vi.mock('axios')` or `global.fetch = vi.fn()`.
3. Use fake timers for time-dependent logic.
4. Do not mock your own modules — if you must, extract the logic and test it separately.

**Constraints and Limitations:**

- Some third-party libraries (e.g., payment SDKs) may need to be mocked at the boundary if they cannot run in jsdom.
- Real dependencies make tests slightly slower but far more trustworthy.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Custom `renderWithProviders` with All Real Providers

```jsx
// test-utils.jsx
import { render } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import reducer from './reducers';

export function renderWithProviders(ui, {
  initialEntries = ['/'],
  preloadedState = {},
  ...options
} = {}) {
  const store = configureStore({ reducer, preloadedState });
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  function Wrapper({ children }) {
    return (
      <Provider store={store}>
        <QueryClientProvider client={queryClient}>
          <MemoryRouter initialEntries={initialEntries}>
            {children}
          </MemoryRouter>
        </QueryClientProvider>
      </Provider>
    );
  }

  return { store, queryClient, ...render(ui, { wrapper: Wrapper, ...options }) };
}
```

**Why this follows the real-dependencies principle:** The helper provides the real store, real router, and real QueryClient — not mocks. Each test gets fresh instances, ensuring isolation while exercising the real integration.

### Real-World Cases

- **Full feature pages:** Rendering a page with routing, state, and data fetching — all real except the API.
- **Redux-connected components:** Using a real store with preloaded state.
- **React Query components:** Using a real `QueryClient` with retries disabled.

### References

- The Modern Frontend Testing Pyramid & Strategy — https://feature-sliced.design/ja/blog/frontend-testing-strategy
- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- Use Wrapper Option for Context Providers — https://raw.githubusercontent.com/pproenca/dot-skills/refs/heads/master/skills/.experimental/react-testing-library/references/setup-wrapper-providers.md

---

## 8. Integration Testing Best Practices

### Definitions

**Core Definition:** Integration testing best practices are the guidelines that make integration tests reliable, maintainable, fast enough to run frequently, and effective at catching real integration bugs.

**Technical Definition:** Best practices encompass: using real internal code and mocking only genuine external boundaries; mocking at the network layer with MSW rather than mocking `fetch` or `axios`; using a custom `renderWithProviders` helper for consistent provider setup; asserting on user-visible behavior rather than implementation details; using `findBy*` and `waitFor` for async flows; keeping tests isolated with fresh stores and clean MSW handlers; and targeting 25–35% of the test suite at the integration level. Snapshot tests are discouraged in component tests, and query priority (role → label → text → testId) should be followed.

**Beginner-Friendly Explanation:** Best practices keep integration tests useful — use real code, fake only the server, wrap everything in the right providers, check what the user sees, and keep each test independent.

### Purposes

- To ensure integration tests fail for the right reason and pass for the right reason.
- To maximize confidence per test while keeping maintenance costs manageable.
- To keep tests fast enough to run on every pull request.
- To provide a stable foundation for the feature's regression safety net.

### Syntax Rules and Structure

**Best practices checklist:**

| Practice | Description |
|----------|-------------|
| **Real internal code, mock external boundaries** | Use real components, hooks, and stores; mock only network, time, and browser APIs. |
| **MSW over fetch mocks** | Intercept at the HTTP layer for realistic, framework-agnostic mocking. |
| **Custom `renderWithProviders`** | Centralize provider setup; import from `test-utils`, not RTL directly. |
| **Assert on user-visible behavior** | Query roles, labels, and text — not internal state or CSS classes. |
| **Use `findBy*` for async data** | Wait for data to appear; never use hardcoded sleeps. |
| **Keep tests isolated** | Fresh store, fresh QueryClient, fresh MSW handlers per test. |
| **Mock at the correct boundary** | Mock `fetch` with MSW, not by replacing the global function. |
| **Test the happy path and key error states** | Integration tests are expensive — do not test every edge case here. |
| **Follow query priority** | `getByRole` → `getByLabelText` → `getByText` → `getByTestId`. |
| **No snapshots in component/integration tests** | Snapshots are brittle and hide regressions. |

**Constraints and Limitations:**

- Over-mocking turns an integration test into a unit test — if you mock your own modules, you lose integration confidence.
- Integration tests run in jsdom, not a real browser — do not rely on layout, scrolling, or real timers.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: A Best-Practice-Compliant Integration Test

```jsx
import { renderWithProviders, screen } from './test-utils';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';
import { test, expect } from 'vitest';
import { ShopPage } from './ShopPage';

test('user adds products to the cart and sees the total', async () => {
  const user = userEvent.setup();

  server.use(
    http.get('/api/products', () =>
      HttpResponse.json([
        { id: 1, name: 'Keyboard', price: 2199 },
        { id: 2, name: 'Headphones', price: 1299 },
      ])
    )
  );

  renderWithProviders(<ShopPage />);

  const keyboard = await screen.findByRole('article', { name: /keyboard/i });
  await user.click(within(keyboard).getByRole('button', { name: /add to cart/i }));
  await user.click(within(keyboard).getByRole('button', { name: /add to cart/i }));

  const headphones = screen.getByRole('article', { name: /headphones/i });
  await user.click(within(headphones).getByRole('button', { name: /add to cart/i }));

  expect(screen.getByRole('link', { name: /cart \(3\)/i })).toBeInTheDocument();

  const summary = screen.getByRole('region', { name: /cart summary/i });
  expect(within(summary).getByText('Keyboard × 2')).toBeInTheDocument();
  expect(within(summary).getByText('Total: €5,697.00')).toBeInTheDocument();
});
```

**Why this follows best practices:**

- Uses `renderWithProviders` for real Router and Context providers.
- Uses MSW to mock the API at the network layer — no `fetch` mocking.
- Queries by accessible roles (`article`, `region`, `link`) with accessible names.
- Uses `user-event` for realistic interactions.
- Asserts on user-visible output (cart count, line items, total).
- Uses `findByRole` for async data appearance.
- The expected value is computed by hand, not by copying the code's formula.

### Real-World Cases

- **E-commerce flows:** Adding products to a cart, updating quantities, and verifying totals.
- **Authentication flows:** Login, logout, and protected route access with mocked API responses.
- **Data dashboards:** Loading data from multiple endpoints and verifying the combined UI.

### References

- Testing Strategies for React and Next.js Applications — https://github.com/alirezarezvani/claude-skills/blob/9ca476d8401f8750f92f4a7b31ec68efaa61ab02/engineering-team/skills/senior-qa/references/testing_strategies.md
- The Modern Frontend Testing Pyramid & Strategy — https://feature-sliced.design/ja/blog/frontend-testing-strategy
- Cart flow (MSW integration example) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/vitest/project-cart-flow.md
- Writing Tests — Redux — https://redux.js.org/usage/writing-tests

---

## References

- Testing Strategies for React and Next.js Applications — https://github.com/alirezarezvani/claude-skills/blob/9ca476d8401f8750f92f4a7b31ec68efaa61ab02/engineering-team/skills/senior-qa/references/testing_strategies.md
- The Modern Frontend Testing Pyramid & Strategy — https://feature-sliced.design/ja/blog/frontend-testing-strategy
- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- How to mock API requests in React tests — https://coreui.io/answers/how-to-mock-api-requests-in-react-tests/
- Using Mock Service Worker to Improve Jest Unit Tests — https://www.wwt.com/article/using-mock-service-worker-to-improve-jest-unit-tests
- Cart flow (MSW integration example) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/vitest/project-cart-flow.md
- Testing TanStack Form — https://github.com/fusengine/agents/blob/main/plugins/nextjs-expert/skills/nextjs-tanstack-form/references/testing.md
- MemoryRouter — React Router — https://reactrouter.com/6.30.3/router-components/memory-router
- What are the considerations for testing components that utilize React Router — https://www.edureka.co/community/309032/considerations-testing-components-utilize-router-navigation
- Use Wrapper Option for Context Providers — https://raw.githubusercontent.com/pproenca/dot-skills/refs/heads/master/skills/.experimental/react-testing-library/references/setup-wrapper-providers.md
- Testing Studio React Components — https://www.sanity.io/learn/course/testing-sanity-studio/testing-studio-react-component
- @suerg/test-kit — https://www.npmjs.com/package/@suerg/test-kit
- @render-with/redux — https://www.npmjs.com/package/@render-with/redux
- Testing Playbook — https://raw.githubusercontent.com/Paldom/node-skills/6e17f63b7b67614f9e3386fdec59e4f15443c2f2/skills/node-testing/references/testing-playbook.md
- vibe-coding-tools (React testing guide) — https://github.com/legioncodeinc/vibe-coding-tools/blob/c1e05d022d865e32f4cb93d313d5f51ad0175a0b/plugins/web-development-tools/skills/react-stinger/guides/08-testing.md