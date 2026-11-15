# React Custom Hooks — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Custom Hooks are JavaScript functions whose names begin with "use" and that call other React Hooks, allowing you to extract and reuse stateful logic across multiple components.

**Technical Definition:** A custom Hook is a function that encapsulates stateful logic by composing built-in React Hooks (such as `useState`, `useEffect`, `useRef`, `useCallback`, etc.). Custom Hooks are not a React feature per se — they are a convention built on top of the existing Hooks API. React identifies a function as a Hook by its `use` prefix, which signals to the linter and React DevTools that the function may contain Hook calls and must follow the Rules of Hooks. Custom Hooks may call other Hooks, including other custom Hooks, enabling composition and abstraction of complex logic.

**Beginner-Friendly Explanation:** A custom Hook is just a function that starts with "use" and lets you reuse logic that involves state and side effects. Instead of copying and pasting the same `useState` and `useEffect` code into five different components, you write it once in a custom Hook and call that Hook from each component.

### Key Characteristics

- **Naming convention:** Must start with `use` followed by a capital letter (e.g., `useFetch`, `useLocalStorage`).
- **Composition:** Can call other Hooks (built-in or custom) at the top level.
- **Reusability:** Encapsulates stateful logic, not state itself — each component using a custom Hook gets its own independent state.
- **Rules of Hooks apply:** Custom Hooks must only be called at the top level of a function component or another custom Hook — never inside conditions, loops, or nested functions.
- **Single responsibility:** Each custom Hook should have one clear purpose.

### Prerequisites

- A React application (v16.8+ for Hooks support).
- Basic understanding of built-in React Hooks (`useState`, `useEffect`, `useRef`, `useCallback`, `useMemo`).
- Familiarity with JavaScript closures and async operations (Promises, async/await).
- For testing: `@testing-library/react` (with `renderHook`) or an equivalent testing library.

### Related Programming Areas

- **React component architecture** — Custom Hooks separate stateful logic from rendering.
- **Hooks composition** — Building complex Hooks from simpler ones.
- **Testing** — `renderHook` and `act` from `@testing-library/react` enable isolated Hook testing.
- **Browser APIs** — Many common Hooks wrap Web APIs (LocalStorage, IntersectionObserver, WebSocket, etc.).
- **Performance optimization** — Hooks like `useDebounce` and `useThrottle` optimize event handling.

### Core Concepts / Features

---

## 1. Custom Hooks

### Definitions

**Core Definition:** A custom Hook is a reusable JavaScript function prefixed with `use` that encapsulates stateful logic and can call other Hooks.

**Technical Definition:** A custom Hook is a function that satisfies three criteria: (1) its name starts with `use`, (2) it may call built-in React Hooks, and (3) it is called only from a function component or another custom Hook. Custom Hooks do not share state between components — each call creates an independent state instance. They are a mechanism for logic reuse, not state reuse.

**Beginner-Friendly Explanation:** A custom Hook is like a recipe that tells React how to manage a piece of state or a side effect. You write the recipe once, and any component can follow it — but each component gets its own kitchen (state).

### Purposes

- To extract and reuse stateful logic across multiple components without higher-order components or render props.
- To simplify complex components by moving non-rendering logic into dedicated functions.
- To test stateful logic in isolation from the UI.
- To create a shared library of behaviors (e.g., data fetching, storage, DOM observation) that can be composed across an application.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function useCustomHook(parameters) {
  // Call built-in or custom Hooks at the top level
  const [state, setState] = useState(initialValue);

  useEffect(() => {
    // Side effect logic
    return () => { /* cleanup */ };
  }, [dependencies]);

  // Return whatever the consumer needs
  return { state, setState };
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useCustomHook` | Must start with `use` + capital letter. |
| Parameters | Inputs the Hook needs (optional). |
| Hook calls | Must be at the top level, unconditionally. |
| Return value | State, callbacks, refs, or any API the consumer needs. |

**Syntax Rules:**

1. Custom Hook names must start with `use`.
2. Call Hooks only at the top level — never inside conditions, loops, or nested functions.
3. Custom Hooks may call other Hooks (that's their purpose).
4. Call custom Hooks only from function components or other custom Hooks.
5. Each custom Hook should have a single, concrete purpose.

**Constraints and Limitations:**

- Custom Hooks cannot be called conditionally — the same Hooks must be called in the same order on every render.
- State is not shared between components using the same custom Hook; each call is independent.
- Custom Hooks do not replace components; they cannot return JSX (though they can return values that components render).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Extracting a Counter Hook

```jsx
// Step 1: Define the custom Hook
import { useState, useCallback } from 'react';

function useCounter(initialValue = 0, step = 1) {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount((c) => c + step), [step]);
  const decrement = useCallback(() => setCount((c) => c - step), [step]);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset };
}

// Step 2: Use the Hook in components
function CounterA() {
  const { count, increment } = useCounter(0, 1);
  return <button onClick={increment}>A: {count}</button>;
}

function CounterB() {
  const { count, increment } = useCounter(100, 10);
  return <button onClick={increment}>B: {count}</button>;
}
```

**Expected Output:** `CounterA` starts at 0 and increments by 1; `CounterB` starts at 100 and increments by 10. Each component has independent state.

**Why this output:** Each call to `useCounter` creates its own `useState` instance. The Hook encapsulates the counter logic, but the state is per-component.

### Real-World Cases

- **Data fetching:** `useFetch` wraps the Fetch API with loading/error states.
- **Persistent state:** `useLocalStorage` syncs state with `localStorage`.
- **DOM observation:** `useIntersectionObserver` observes element visibility.
- **Real-time communication:** `useWebSocket` manages WebSocket connections.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- Patterns.dev — Hooks Pattern — https://www.patterns.dev/react/hooks-pattern

---

## 2. Hook Naming

### Definitions

**Core Definition:** Hook naming is the convention of prefixing custom Hook functions with `use` followed by a descriptive name in PascalCase (e.g., `useFetch`, `useLocalStorage`).

**Technical Definition:** The `use` prefix is a signal to React and its tooling (ESLint, React DevTools) that the function follows the Rules of Hooks. Without the prefix, React cannot distinguish a custom Hook from a regular function, and the linter will not enforce Hook rules. The name after `use` should describe the Hook's purpose clearly and concisely.

**Beginner-Friendly Explanation:** Always name your custom Hooks `useSomething` — like `useFetch` or `useToggle`. The `use` part tells React "this function follows hook rules," and the rest of the name tells other developers what it does.

### Purposes

- To signal to React and tooling that the function is a Hook and must follow the Rules of Hooks.
- To make the Hook's purpose immediately clear from its name.
- To enable ESLint rules (like `eslint-plugin-react-hooks`) to validate Hook usage.
- To improve discoverability and consistency across a codebase.

### Syntax Rules and Structure

```jsx
// ✅ Good names
useFetch
useLocalStorage
useDebounce
useIntersectionObserver
useWindowSize

// ❌ Bad names
fetchData      // Missing use prefix
use_fetch      // Underscore instead of PascalCase
UseFetch       // Capital U
use-fetch      // Hyphen
```

**Component Breakdown:**

| Part | Description |
|------|-------------|
| `use` | Required prefix (lowercase). |
| `Name` | PascalCase descriptive name (e.g., `Fetch`, `LocalStorage`). |

**Syntax Rules:**

1. Must start with `use` (lowercase).
2. The character after `use` should be a capital letter (PascalCase).
3. The name should describe the Hook's purpose (what it does, what it returns, or what it manages).
4. Avoid generic names like `useData` or `useLogic` — be specific.

**Constraints and Limitations:**

- A function without the `use` prefix that calls Hooks will violate the Rules of Hooks and may cause runtime errors.
- The `use` prefix is a convention enforced by tooling, not a language keyword.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Good vs. Bad Naming

```jsx
// ✅ Good: clear, descriptive, use prefix
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  // ...
  return size;
}

// ❌ Bad: no use prefix — React will not treat it as a Hook
function windowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  // ...
  return size;
}

// ❌ Bad: vague name
function useStuff() {
  // What does this do?
}
```

**Why this matters:** Without the `use` prefix, React cannot detect that `windowSize` contains Hooks. Calling it from a component will violate the Rules of Hooks and cause errors.

### Real-World Cases

- **Library authors:** Published Hook libraries (e.g., `react-use`, `@mantine/hooks`) follow strict naming conventions so consumers can immediately identify Hooks.
- **Team codebases:** Consistent naming makes code reviews and onboarding easier.
- **Linting:** `eslint-plugin-react-hooks` relies on the `use` prefix to detect Hook calls and enforce rules.

### References

- React — Rules of Hooks: Only call Hooks from React functions — https://react.dev/reference/rules/rules-of-hooks
- ESLint Plugin React Hooks — https://www.npmjs.com/package/eslint-plugin-react-hooks

---

## 3. Hook Composition

### Definitions

**Core Definition:** Hook composition is the practice of building custom Hooks by combining simpler built-in Hooks or other custom Hooks.

**Technical Definition:** Composition allows a custom Hook to call other Hooks (built-in or custom) at its top level. This creates a dependency chain where a complex Hook is built from smaller, focused Hooks. The key constraint is that all Hook calls must remain at the top level — composition does not permit conditional Hook calls. Composing independent Hooks at the component level (rather than deeply nesting them) makes the dependency graph explicit and each Hook independently replaceable.

**Beginner-Friendly Explanation:** Hook composition means building bigger Hooks out of smaller ones. For example, a `useUserProfile` Hook might use `useFetch` to get data and `useLocalStorage` to cache it. Each Hook does one thing, and you combine them like building blocks.

### Purposes

- To build complex behaviors from small, focused, testable units.
- To avoid monolithic Hooks that try to do too much.
- To make dependency chains explicit and maintainable.
- To enable reuse of intermediate Hooks across different composite Hooks.

### Syntax Rules and Structure

```jsx
// Smaller Hook: useDebounce
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debouncedValue;
}

// Smaller Hook: useFetch
function useFetch(url) {
  const [data, setData] = useState(null);
  // ...fetch logic
  return data;
}

// Composed Hook: useDebouncedSearch
function useDebouncedSearch(query, delay = 500) {
  const debouncedQuery = useDebounce(query, delay); // Composes useDebounce
  const results = useFetch(`/api/search?q=${debouncedQuery}`); // Composes useFetch
  return results;
}
```

**Component Breakdown:**

| Hook | Description |
|------|-------------|
| `useDebounce` | Delays value updates. |
| `useFetch` | Fetches data from a URL. |
| `useDebouncedSearch` | Composes both to create debounced search. |

**Syntax Rules:**

1. Composed Hooks must be called at the top level of the custom Hook.
2. Each sub-Hook should have a single responsibility.
3. Prefer composing at the component level rather than deeply nesting Hooks to keep dependencies visible.
4. Return values should be predictable and composable (tuples or objects).

**Constraints and Limitations:**

- Deep nesting of Hooks can obscure the dependency chain and make debugging harder.
- Composed Hooks share the same state instance per call — there is no memoization between components.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Composing useDebounce with useFetch

```jsx
import { useState, useEffect } from 'react';

// Step 1: useDebounce (small Hook)
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}

// Step 2: useFetch (small Hook)
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  useEffect(() => {
    let cancelled = false;
    fetch(url)
      .then((r) => r.json())
      .then((d) => { if (!cancelled) setData(d); })
      .finally(() => { if (!cancelled) setLoading(false); });
    return () => { cancelled = true; };
  }, [url]);
  return { data, loading };
}

// Step 3: Composed Hook
function useDebouncedSearch(query, delay = 300) {
  const debouncedQuery = useDebounce(query, delay);
  const { data, loading } = useFetch(
    debouncedQuery ? `/api/search?q=${debouncedQuery}` : null
  );
  return { results: data, isLoading: loading, debouncedQuery };
}

// Step 4: Use in component
function SearchBox() {
  const [query, setQuery] = useState('');
  const { results, isLoading } = useDebouncedSearch(query);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {isLoading ? <p>Searching...</p> : <ResultsList results={results} />}
    </div>
  );
}
```

**Expected Output:** Typing in the search box debounces the query by 300ms, then fetches results. The loading state shows during the fetch.

**Why this output:** `useDebouncedSearch` composes `useDebounce` (to delay the query) and `useFetch` (to fetch results). Each Hook handles one responsibility, making the composed Hook easy to understand and test.

### Real-World Cases

- **Search interfaces:** `useDebouncedSearch` combines `useDebounce` and `useFetch`.
- **Authentication flows:** `useAuth` composes `useLocalStorage` (for tokens) and `useFetch` (for login/logout APIs).
- **UI state machines:** Composing `useToggle`, `usePrevious`, and `useTimeout` for complex interactive components.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- GitHub — React Hooks Composition Patterns — https://github.com/pproenca/dot-skills/blob/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/react-refactor/references/hook-composition-over-nesting.md
- GitHub — Components, composition, and TypeScript — https://github.com/vercel/next.js/blob/canary/docs/01-app/03-api-reference/01-directives/use-client.mdx

---

## 4. Hook Reuse

### Definitions

**Core Definition:** Hook reuse is the ability to call the same custom Hook from multiple components, with each call receiving its own independent state.

**Technical Definition:** Custom Hooks are reusable logic containers, not state containers. When a component calls `useCounter()`, it gets a new `useState` instance. Two components calling the same Hook do not share state. Reuse is achieved by extracting the logic into a function that can be imported and called across the codebase.

**Beginner-Friendly Explanation:** You can use the same custom Hook in many components. Each component gets its own copy of the state — they do not interfere with each other.

### Purposes

- To avoid duplicating stateful logic across components.
- To create a shared library of behaviors (e.g., `useToggle`, `useFetch`) that any component can use.
- To ensure consistency in how common patterns (storage, fetching, event listeners) are implemented.
- To make logic testable once and trusted everywhere.

### Syntax Rules and Structure

```jsx
// Define once
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

// Reuse in multiple components
function Modal() {
  const [isOpen, toggleOpen] = useToggle(false);
  // ...
}

function Sidebar() {
  const [isExpanded, toggleExpanded] = useToggle(true);
  // ...
}
```

**Component Breakdown:**

| Usage | Description |
|-------|-------------|
| `useToggle(false)` | Independent state instance for `Modal`. |
| `useToggle(true)` | Independent state instance for `Sidebar`. |

**Syntax Rules:**

1. Each call to a custom Hook creates a new, independent state instance.
2. Custom Hooks do not share state between components by default.
3. To share state, use Context or a state management library alongside the Hook.

**Constraints and Limitations:**

- Reuse is of logic, not state. If shared state is needed, the Hook should read from a Context or external store.
- Hooks that depend on browser APIs (e.g., `window`) may need SSR guards.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Reusing useLocalStorage Across Components

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setStoredValue = (newValue) => {
    setValue(newValue);
    window.localStorage.setItem(key, JSON.stringify(newValue));
  };

  return [value, setStoredValue];
}

// Component A: theme preference
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>{theme}</button>;
}

// Component B: username preference
function NameInput() {
  const [name, setName] = useLocalStorage('username', '');
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

**Expected Output:** `ThemeToggle` persists the theme under the `"theme"` key; `NameInput` persists the username under the `"username"` key. Each component has independent state, but both use the same reusable Hook.

### Real-World Cases

- **Design systems:** Reusing `useDisclosure` (open/close state) across modals, dropdowns, and drawers.
- **Form libraries:** Reusing `useForm` across all forms in an application.
- **Data fetching:** Reusing `useFetch` across dozens of components.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- 30 Seconds of Code — Persisting state in React — https://github.com/Chalarangelo/30-seconds-of-code/blob/master/content/snippets/react/s/persist-state-to-storage.md

---

## 5. Hook Testing

### Definitions

**Core Definition:** Hook testing is the practice of writing unit tests for custom Hooks to verify that their state, side effects, and returned values behave correctly.

**Technical Definition:** Hook testing uses `renderHook` from `@testing-library/react` to render a Hook in isolation, `act` to wrap state updates, and `waitFor` for async assertions. The `result.current` property holds the Hook's latest return value. For Hooks that depend on Context or Router, a `wrapper` option provides the necessary providers. The original `react-hooks-testing-library` is now deprecated in favor of `@testing-library/react`'s built-in `renderHook`.

**Beginner-Friendly Explanation:** Hook testing means writing small tests that check your custom Hook works correctly — like testing that `useCounter` actually increments the count. You use special tools to run the Hook by itself, without a full component.

### Purposes

- To verify that stateful logic is correct independently of the UI.
- To test edge cases (error states, cleanup, rapid updates) that are hard to trigger in a full component.
- To prevent regressions when refactoring Hook logic.
- To document the expected behavior of a Hook through executable tests.

### Syntax Rules and Structure

```jsx
import { renderHook, act, waitFor } from '@testing-library/react';

const { result } = renderHook(() => useCounter(0));

act(() => {
  result.current.increment();
});

expect(result.current.count).toBe(1);
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `renderHook(fn)` | Renders the Hook inside a test harness. |
| `result.current` | The Hook's latest return value. |
| `act(cb)` | Wraps state updates to flush effects. |
| `rerender(newProps)` | Re-renders the Hook with new props. |
| `unmount()` | Unmounts the Hook to test cleanup. |

**Syntax Rules:**

1. Use `renderHook` from `@testing-library/react` (React 18+).
2. Wrap state-changing calls in `act`.
3. Use `waitFor` for async state updates.
4. Read `result.current` after `act` or `waitFor`.
5. Provide a `wrapper` for Hooks that need Context or Router.

**Constraints and Limitations:**

- Prefer testing via a real component when the Hook is simple and easy to exercise through the UI.
- `renderHook` is best for complex, reusable Hooks that are hard to test through a single component.
- Never mock React built-ins — use MSW or similar tools for data-fetching Hooks.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing useCounter

```jsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('increments and decrements the counter', () => {
  const { result } = renderHook(() => useCounter(10, 2));

  expect(result.current.count).toBe(10);

  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(12);

  act(() => {
    result.current.decrement();
  });
  expect(result.current.count).toBe(10);

  act(() => {
    result.current.reset();
  });
  expect(result.current.count).toBe(10);
});
```

**Expected Output:** All assertions pass. The test verifies that `useCounter` correctly increments by 2, decrements by 2, and resets to the initial value.

**Why this output:** `renderHook` runs the Hook in isolation. `act` wraps each state update so React processes it. `result.current` always holds the latest return value.

#### Example 2: Testing Async Hook with waitFor

```jsx
import { renderHook, waitFor } from '@testing-library/react';

function useUser(id) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`/api/users/${id}`).then((r) => r.json()).then(setUser);
  }, [id]);
  return user;
}

test('fetches user data', async () => {
  global.fetch = jest.fn(() =>
    Promise.resolve({ json: () => Promise.resolve({ name: 'Alice' }) })
  );

  const { result } = renderHook(() => useUser(1));

  await waitFor(() => expect(result.current).toEqual({ name: 'Alice' }));
});
```

**Expected Output:** The test passes after the mocked fetch resolves. `waitFor` retries until the Hook's state updates.

### Real-World Cases

- **Library development:** Testing Hooks like `useDebounce` for rapid input cancellation and cleanup.
- **Data-fetching Hooks:** Testing `useFetch` with MSW (Mock Service Worker) for network-level mocking.
- **Context-dependent Hooks:** Testing Hooks that read from Context by providing a `wrapper` with the provider.

### References

- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/
- Testing Library — renderHook — https://testing-library.com/docs/react-testing-library/api/#renderhook

---

## 6. Common Custom Hooks

### 6.1 `useFetch`

#### Definitions

**Core Definition:** `useFetch` is a custom Hook that wraps the Fetch API to handle data fetching, loading states, error handling, and request cancellation.

**Technical Definition:** `useFetch(url, options)` manages the lifecycle of an HTTP request. It returns `{ data, loading, error }` (or similar), and typically handles race conditions via `AbortController`, response parsing, and cleanup on unmount.

**Beginner-Friendly Explanation:** `useFetch` fetches data from a URL and tells you when it's loading, when it's done, and if something went wrong — all in one line.

#### Purposes

- To eliminate manual `useState` + `useEffect` fetching boilerplate.
- To handle loading and error states consistently across components.
- To prevent race conditions and memory leaks via request cancellation.
- To provide a declarative API for server data.

#### Syntax Rules and Structure

```jsx
const { data, loading, error } = useFetch(url, options);
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `data` | Parsed response body. |
| `loading` | Boolean indicating pending state. |
| `error` | Error object if the request failed. |

**Syntax Rules:**

1. Call at the top level of a component.
2. The `url` should be stable or memoized to avoid unnecessary re-fetches.
3. Handle cleanup with `AbortController` to prevent state updates on unmounted components.

**Constraints and Limitations:**

- Does not handle caching or deduplication (use a library like React Query for that).
- SSR requires guards for `window` and `fetch`.

#### Annotated Code Example

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!url) return;
    const controller = new AbortController();
    setLoading(true);
    setError(null);

    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(setData)
      .catch((err) => {
        if (err.name !== 'AbortError') setError(err);
      })
      .finally(() => setLoading(false));

    return () => controller.abort(); // Cancel on unmount or URL change
  }, [url]);

  return { data, loading, error };
}
```

**Expected Output:** While fetching, `loading` is `true`. On success, `data` contains the parsed JSON. On failure, `error` contains the error. If the component unmounts mid-fetch, the request is aborted.

#### Real-World Cases

- **User profiles:** Fetching user data from `/api/users/:id`.
- **Product catalogs:** Loading product lists with pagination or filters.
- **Search results:** Combining `useFetch` with `useDebounce` for search-as-you-type.

#### References

- GitHub — Mastering HTTP Requests with the useFetch Hook — https://github.com/small-lab-io/usehooks.io/blob/main/apps/www/posts/mastering-http-requests-with-the-usefetch-hook.mdx
- GitHub — useFetch (nxtedition) — https://github.com/nxtedition/graphics-kit/blob/main/src/hooks/useFetch.md

---

### 6.2 `useLocalStorage`

#### Definitions

**Core Definition:** `useLocalStorage` is a custom Hook that syncs a stateful value with `window.localStorage`, persisting it across page reloads.

**Technical Definition:** `useLocalStorage(key, initialValue)` returns a tuple `[storedValue, setValue]`, similar to `useState`. It lazily initializes from `localStorage`, falls back to `initialValue`, and writes to `localStorage` on every update. It typically uses `try/catch` for SSR safety and JSON serialization.

**Beginner-Friendly Explanation:** `useLocalStorage` works like `useState`, but the value survives page refreshes because it is saved in the browser's local storage.

#### Purposes

- To persist user preferences (theme, language, layout) across sessions.
- To cache form data so users do not lose progress on reload.
- To provide a `useState`-like API with automatic persistence.

#### Syntax Rules and Structure

```jsx
const [value, setValue] = useLocalStorage(key, initialValue);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `key` | The `localStorage` key. |
| `initialValue` | Fallback if no stored value exists. |
| `value` | Current state (synced with storage). |
| `setValue` | Updates state and storage. |

**Syntax Rules:**

1. Initialize lazily with a function to read `localStorage` once.
2. Use `try/catch` for SSR and privacy-mode safety.
3. Serialize with `JSON.stringify` and parse with `JSON.parse`.

**Constraints and Limitations:**

- Does not automatically sync across tabs (listen to the `storage` event for that).
- Limited storage quota (typically 5–10 MB).
- Not available in server-side rendering environments without guards.

#### Annotated Code Example

```jsx
function useLocalStorage(key, defaultValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const value = window.localStorage.getItem(key);
      if (value) return JSON.parse(value);
      window.localStorage.setItem(key, JSON.stringify(defaultValue));
      return defaultValue;
    } catch {
      return defaultValue;
    }
  });

  const setValue = (newValue) => {
    try {
      window.localStorage.setItem(key, JSON.stringify(newValue));
    } catch {}
    setStoredValue(newValue);
  };

  return [storedValue, setValue];
}
```

**Expected Output:** The initial render reads from `localStorage` if a value exists. Calling `setValue` updates both React state and `localStorage`.

#### Real-World Cases

- **Theme persistence:** Saving dark/light mode preference.
- **Form drafts:** Preserving unsaved form data across reloads.
- **User settings:** Storing language, font size, or layout preferences.

#### References

- 30 Seconds of Code — Persisting state in React — https://github.com/Chalarangelo/30-seconds-of-code/blob/master/content/snippets/react/s/persist-state-to-storage.md

---

### 6.3 `useSessionStorage`

#### Definitions

**Core Definition:** `useSessionStorage` is a custom Hook that syncs a stateful value with `window.sessionStorage`, persisting it for the duration of the page session.

**Technical Definition:** `useSessionStorage(key, initialValue)` works identically to `useLocalStorage` but uses `sessionStorage` instead. Data stored in `sessionStorage` is cleared when the browser tab or window is closed.

**Beginner-Friendly Explanation:** `useSessionStorage` is like `useLocalStorage`, but the data disappears when you close the tab. It is useful for temporary state like multi-step form progress.

#### Purposes

- To persist data for a single tab session without long-term storage.
- To store multi-step form progress that should reset when the tab closes.
- To isolate state between tabs (each tab gets its own `sessionStorage`).

#### Syntax Rules and Structure

```jsx
const [value, setValue] = useSessionStorage(key, initialValue);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `key` | The `sessionStorage` key. |
| `initialValue` | Fallback if no stored value exists. |

**Syntax Rules:**

1. Same as `useLocalStorage` but uses `window.sessionStorage`.
2. Apply the same `try/catch` and JSON serialization patterns.

**Constraints and Limitations:**

- Data is not shared between tabs.
- Cleared when the tab closes.
- Same storage quota constraints as `localStorage`.

#### Annotated Code Example

```jsx
function useSessionStorage(key, defaultValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const value = window.sessionStorage.getItem(key);
      if (value) return JSON.parse(value);
      window.sessionStorage.setItem(key, JSON.stringify(defaultValue));
      return defaultValue;
    } catch {
      return defaultValue;
    }
  });

  const setValue = (newValue) => {
    try {
      window.sessionStorage.setItem(key, JSON.stringify(newValue));
    } catch {}
    setStoredValue(newValue);
  };

  return [storedValue, setValue];
}
```

**Expected Output:** The value persists across page reloads within the same tab, but is cleared when the tab is closed.

#### Real-World Cases

- **Checkout flows:** Preserving cart or form data during a single shopping session.
- **Wizard forms:** Keeping multi-step form state as the user navigates between steps.
- **Temporary filters:** Storing search filters that should reset on a new visit.

#### References

- 30 Seconds of Code — Persisting state in React — https://github.com/Chalarangelo/30-seconds-of-code/blob/master/content/snippets/react/s/persist-state-to-storage.md

---

### 6.4 `useDebounce`

#### Definitions

**Core Definition:** `useDebounce` is a custom Hook that delays the update of a value until a specified time has passed without another change.

**Technical Definition:** `useDebounce(value, delay)` returns a debounced version of `value`. It sets a timer on every change and only updates the debounced value when the timer completes without being reset. It cleans up the timer on change or unmount.

**Beginner-Friendly Explanation:** `useDebounce` waits until the user stops typing for a certain amount of time before updating the value. It is perfect for search boxes so you do not fire an API request on every keystroke.

#### Purposes

- To reduce the frequency of expensive operations (API calls, filtering) triggered by rapid input.
- To improve performance in search, resize, and scroll handlers.
- To group multiple rapid changes into a single final update.

#### Syntax Rules and Structure

```jsx
const debouncedValue = useDebounce(value, delay);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `value` | The rapidly changing value. |
| `delay` | Milliseconds to wait after the last change. |
| `debouncedValue` | The delayed value. |

**Syntax Rules:**

1. Use `useState` to hold the debounced value.
2. Use `useEffect` with `setTimeout` and `clearTimeout`.
3. Clean up the timer when `value` or `delay` changes.

**Constraints and Limitations:**

- Adds latency to the final update (the delay).
- Not suitable for values that must update immediately (use `useThrottle` instead).

#### Annotated Code Example

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) {
      fetch(`/api/search?q=${debouncedQuery}`);
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Expected Output:** Typing "React" fires the fetch only once, 500ms after the last keystroke.

#### Real-World Cases

- **Search boxes:** Debouncing user input before API calls.
- **Window resize:** Debouncing expensive layout recalculations.
- **Form validation:** Debouncing validation checks as the user types.

#### References

- Toss React Simplikit — useDebounce — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useDebounce/useDebounce.md
- use-debounce (npm) — https://www.npmjs.com/package/use-debounce

---

### 6.5 `useThrottle`

#### Definitions

**Core Definition:** `useThrottle` is a custom Hook that limits the rate at which a value or function is updated to at most once per specified time interval.

**Technical Definition:** `useThrottle(value, interval)` ensures that the throttled value updates at most once every `interval` milliseconds. Unlike debounce, which waits for a pause, throttle guarantees regular updates during continuous changes.

**Beginner-Friendly Explanation:** `useThrottle` updates a value at most once every X milliseconds — even if the input changes constantly. It is useful for scroll or resize events where you want regular updates, not just one after the user stops.

#### Purposes

- To limit the frequency of updates during continuous events (scroll, resize, mousemove).
- To improve performance without completely delaying updates.
- To provide regular, predictable updates instead of waiting for a pause.

#### Syntax Rules and Structure

```jsx
const throttledValue = useThrottle(value, interval);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `value` | The rapidly changing value. |
| `interval` | Minimum time between updates (ms). |

**Syntax Rules:**

1. Track the last update time with a ref.
2. Only update if `Date.now() - lastUpdate >= interval`.
3. Can be implemented with `useState` + `useRef` or a timer-based approach.

**Constraints and Limitations:**

- May miss intermediate values (only samples at the interval).
- Not ideal for values that must update on every change.

#### Annotated Code Example

```jsx
function useThrottle(value, interval = 200) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastUpdated = useRef(Date.now());

  useEffect(() => {
    const now = Date.now();
    if (now - lastUpdated.current >= interval) {
      lastUpdated.current = now;
      setThrottledValue(value);
    } else {
      const timer = setTimeout(() => {
        lastUpdated.current = Date.now();
        setThrottledValue(value);
      }, interval - (now - lastUpdated.current));
      return () => clearTimeout(timer);
    }
  }, [value, interval]);

  return throttledValue;
}
```

**Expected Output:** During rapid scroll events, the throttled value updates at most once every 200ms.

#### Real-World Cases

- **Scroll position tracking:** Throttling `scrollY` for parallax or progress indicators.
- **Resize handlers:** Updating layout at a controlled rate.
- **Mouse movement:** Throttling cursor tracking for performance.

#### References

- @usefy/use-throttle — https://www.npmjs.com/package/@usefy/use-throttle
- @react-hook/throttle — https://www.npmjs.com/package/@react-hook/throttle

---

### 6.6 `useMediaQuery`

#### Definitions

**Core Definition:** `useMediaQuery` is a custom Hook that tracks whether a CSS media query matches the current viewport.

**Technical Definition:** `useMediaQuery(query)` uses `window.matchMedia(query)` to subscribe to changes and returns a boolean indicating whether the query matches. It is SSR-safe when using `useSyncExternalStore` or a default server snapshot.

**Beginner-Friendly Explanation:** `useMediaQuery` tells you if the screen matches a media query — like `(max-width: 768px)` for mobile. It updates automatically when the window is resized.

#### Purposes

- To conditionally render UI based on viewport size (mobile vs. desktop).
- To sync component logic with CSS breakpoints.
- To avoid duplicated media query logic across components.

#### Syntax Rules and Structure

```jsx
const isMobile = useMediaQuery('(max-width: 768px)');
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `query` | A valid CSS media query string. |
| Return | Boolean indicating whether the query matches. |

**Syntax Rules:**

1. Use `window.matchMedia` to create a `MediaQueryList`.
2. Subscribe to `change` events.
3. Provide a server snapshot (`false`) for SSR compatibility.

**Constraints and Limitations:**

- SSR may render a different initial value (hydration mismatch).
- Requires a browser environment.

#### Annotated Code Example

```jsx
import { useSyncExternalStore } from 'react';

function useMediaQuery(query) {
  const subscribe = (callback) => {
    const media = window.matchMedia(query);
    media.addEventListener('change', callback);
    return () => media.removeEventListener('change', callback);
  };

  const getSnapshot = () => window.matchMedia(query).matches;
  const getServerSnapshot = () => false;

  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

// Usage
function ResponsiveLayout() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  return isMobile ? <MobileNav /> : <DesktopNav />;
}
```

**Expected Output:** On screens ≤768px, `isMobile` is `true` and `MobileNav` renders. On wider screens, `DesktopNav` renders.

#### Real-World Cases

- **Responsive navigation:** Switching between mobile hamburger menu and desktop nav.
- **Conditional rendering:** Showing/hiding sidebars based on screen size.
- **Adaptive layouts:** Adjusting grid columns or spacing based on breakpoints.

#### References

- GitHub — useMediaQuery (SSR-safe with useSyncExternalStore) — https://github.com/barso/hooks/blob/main/packages/hooks/src/useMediaQuery.md
- MUI — useMediaQuery — https://mui.com/material-ui/react-use-media-query/

---

### 6.7 `useIntersectionObserver`

#### Definitions

**Core Definition:** `useIntersectionObserver` is a custom Hook that detects whether a specific DOM element is visible within the viewport (or a root container).

**Technical Definition:** `useIntersectionObserver(callback, options)` wraps the `IntersectionObserver` API. It returns a ref setter that, when attached to an element, observes visibility changes and invokes the callback with an `IntersectionObserverEntry`.

**Beginner-Friendly Explanation:** `useIntersectionObserver` tells you when an element enters or leaves the screen. It is perfect for lazy loading images or triggering animations when a section becomes visible.

#### Purposes

- To implement lazy loading of images and content as they scroll into view.
- To trigger animations or analytics when elements become visible.
- To detect scroll-based visibility without listening to scroll events.

#### Syntax Rules and Structure

```jsx
const ref = useIntersectionObserver(callback, options);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `callback` | Function called with `IntersectionObserverEntry`. |
| `options` | `root`, `rootMargin`, `threshold`. |
| Return | A ref setter to attach to an element. |

**Syntax Rules:**

1. Create an `IntersectionObserver` in an effect.
2. Observe the target element via the returned ref.
3. Clean up the observer on unmount.

**Constraints and Limitations:**

- Requires the Intersection Observer API (widely supported).
- Threshold values control when the callback fires (0 = any pixel, 1 = fully visible).

#### Annotated Code Example

```jsx
function useIntersectionObserver(callback, options = {}) {
  const ref = useRef(null);
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;
    const observer = new IntersectionObserver(
      (entries) => entries.forEach((entry) => callbackRef.current(entry)),
      options
    );
    observer.observe(element);
    return () => observer.disconnect();
  }, [options.root, options.rootMargin, options.threshold]);

  return ref;
}

// Usage
function LazyImage({ src }) {
  const ref = useIntersectionObserver((entry) => {
    if (entry.isIntersecting) {
      entry.target.src = src;
    }
  }, { threshold: 0.1 });

  return <img ref={ref} alt="Lazy loaded" />;
}
```

**Expected Output:** The image `src` is set only when the image enters the viewport.

#### Real-World Cases

- **Infinite scroll:** Detecting when a sentinel element at the bottom of a list becomes visible.
- **Lazy loading images:** Loading images only when they scroll into view.
- **Analytics:** Tracking which sections of a page users actually see.

#### References

- Toss React Simplikit — useIntersectionObserver — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useIntersectionObserver/useIntersectionObserver.md
- MDN — Intersection Observer API — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

---

### 6.8 `useResizeObserver`

#### Definitions

**Core Definition:** `useResizeObserver` is a custom Hook that observes changes to an element's size using the `ResizeObserver` API.

**Technical Definition:** `useResizeObserver(ref, callback)` or `useResizeObserver()` returns a ref and size information. It subscribes to size changes of the observed element and provides the current `width` and `height`.

**Beginner-Friendly Explanation:** `useResizeObserver` tells you when an element changes size — not the window, but a specific `div` or `section`.

#### Purposes

- To respond to element size changes (not just window resizes).
- To update layouts or charts when their container resizes.
- To provide precise dimensions for canvas or SVG rendering.

#### Syntax Rules and Structure

```jsx
const { ref, width, height } = useResizeObserver();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `ref` | Attach to the element to observe. |
| `width`, `height` | Current dimensions. |

**Syntax Rules:**

1. Create a `ResizeObserver` in an effect.
2. Observe the target element via `ref`.
3. Disconnect on cleanup.

**Constraints and Limitations:**

- `ResizeObserver` is not supported in very old browsers (polyfill available).
- Does not work on elements with `display: none`.

#### Annotated Code Example

```jsx
function useResizeObserver() {
  const ref = useRef(null);
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        const { width, height } = entry.contentRect;
        setSize({ width, height });
      }
    });

    observer.observe(element);
    return () => observer.disconnect();
  }, []);

  return { ref, ...size };
}
```

**Expected Output:** When the observed element resizes, `width` and `height` update with the new dimensions.

#### Real-World Cases

- **Charts:** Redrawing charts when their container resizes.
- **Virtualized lists:** Recalculating item heights when the viewport changes.
- **Dashboard widgets:** Adjusting widget layouts based on available space.

#### References

- ZeeCoder/use-resize-observer — https://github.com/ZeeCoder/use-resize-observer
- MDN — ResizeObserver — https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver

---

### 6.9 `useMutationObserver`

#### Definitions

**Core Definition:** `useMutationObserver` is a custom Hook that observes changes to the DOM tree (attributes, child list, text content) using the `MutationObserver` API.

**Technical Definition:** `useMutationObserver(ref, callback, options)` subscribes to DOM mutations on a target element. The callback receives an array of `MutationRecord` objects describing the changes.

**Beginner-Friendly Explanation:** `useMutationObserver` watches for changes inside an HTML element — like when a child is added, an attribute changes, or text is updated.

#### Purposes

- To react to third-party DOM changes (e.g., widgets, iframes).
- To detect when dynamic content is inserted or removed.
- To synchronize React state with external DOM mutations.

#### Syntax Rules and Structure

```jsx
useMutationObserver(ref, callback, { attributes: true, childList: true });
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `ref` | The element to observe. |
| `callback` | Function receiving `MutationRecord[]`. |
| `options` | `attributes`, `childList`, `subtree`, etc. |

**Syntax Rules:**

1. Create a `MutationObserver` in an effect.
2. Observe the target element with the specified options.
3. Disconnect on cleanup.

**Constraints and Limitations:**

- Overuse can cause performance issues (observing large subtrees).
- Does not observe React state changes — only actual DOM mutations.

#### Annotated Code Example

```jsx
function useMutationObserver(ref, callback, options = {}) {
  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new MutationObserver(callback);
    observer.observe(element, options);

    return () => observer.disconnect();
  }, [ref, callback, options]);
}

// Usage
function DynamicContent() {
  const ref = useRef(null);
  useMutationObserver(ref, (mutations) => {
    mutations.forEach((m) => console.log('Mutation:', m.type));
  }, { childList: true, subtree: true });

  return <div ref={ref}>{/* dynamic content */}</div>;
}
```

**Expected Output:** Every time a child is added or removed, the callback logs the mutation type.

#### Real-World Cases

- **Integrating third-party widgets:** Detecting when a widget injects content into the DOM.
- **Rich text editors:** Observing content changes for auto-save.
- **Accessibility monitoring:** Detecting when ARIA attributes change.

#### References

- @siberiacancode/reactuse — useMutationObserver — https://github.com/siberiacancode/reactuse/blob/main/packages/reactuse/src/hooks/useMutationObserver/useMutationObserver.md
- MDN — MutationObserver — https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver

---

### 6.10 `useClickOutside`

#### Definitions

**Core Definition:** `useClickOutside` is a custom Hook that invokes a callback when the user clicks outside a referenced element.

**Technical Definition:** `useClickOutside(ref, callback)` attaches a `mousedown` (or `click`) listener to the document and calls `callback` when the event target is outside the element referenced by `ref`. It is commonly used for dropdowns, modals, and popovers.

**Beginner-Friendly Explanation:** `useClickOutside` tells you when the user clicks somewhere else on the page. It is used to close dropdowns or modals when you click outside them.

#### Purposes

- To close dropdowns, modals, and popovers when the user clicks outside.
- To implement dismissible UI components.
- To avoid manual event listener management.

#### Syntax Rules and Structure

```jsx
useClickOutside(ref, callback);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `ref` | The element to monitor. |
| `callback` | Function called on outside click. |

**Syntax Rules:**

1. Attach a `mousedown` or `click` listener to `document`.
2. Check if `ref.current` contains the event target.
3. Clean up the listener on unmount.

**Constraints and Limitations:**

- Does not work with iframes or elements in shadow DOM without additional handling.
- Should be disabled when the dropdown/modal is closed to avoid unnecessary listeners.

#### Annotated Code Example

```jsx
function useClickOutside(ref, callback) {
  useEffect(() => {
    function handleClick(event) {
      if (ref.current && !ref.current.contains(event.target)) {
        callback(event);
      }
    }

    document.addEventListener('mousedown', handleClick);
    return () => document.removeEventListener('mousedown', handleClick);
  }, [ref, callback]);
}

// Usage
function Dropdown() {
  const [open, setOpen] = useState(false);
  const ref = useRef(null);
  useClickOutside(ref, () => setOpen(false));

  return (
    <div ref={ref}>
      <button onClick={() => setOpen(!open)}>Toggle</button>
      {open && <ul><li>Item 1</li><li>Item 2</li></ul>}
    </div>
  );
}
```

**Expected Output:** Clicking outside the dropdown closes it.

#### Real-World Cases

- **Dropdown menus:** Closing when the user clicks elsewhere.
- **Modals:** Dismissing when clicking the backdrop.
- **Popovers and tooltips:** Hiding when focus moves outside.

#### References

- GitHub — useClickOutside (reactuse) — https://github.com/siberiacancode/reactuse/blob/main/packages/reactuse/src/hooks/useClickOutside/useClickOutside.md
- @reactuses/core — useClickOutside — https://www.npmjs.com/package/@reactuses/core

---

### 6.11 `useKeyPress`

#### Definitions

**Core Definition:** `useKeyPress` is a custom Hook that tracks whether a specific keyboard key is currently pressed.

**Technical Definition:** `useKeyPress(targetKey)` returns a boolean indicating whether the target key is held down. It listens to `keydown` and `keyup` events on the document (or a target element) and updates state accordingly.

**Beginner-Friendly Explanation:** `useKeyPress` tells you if a specific key (like "Enter" or "Escape") is being pressed right now.

#### Purposes

- To detect keyboard shortcuts (e.g., `Escape` to close a modal).
- To track modifier keys (`Shift`, `Control`, `Alt`) for combined interactions.
- To implement keyboard navigation or accessibility features.

#### Syntax Rules and Structure

```jsx
const isPressed = useKeyPress('Enter');
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `targetKey` | The key to track (e.g., `'a'`, `'Enter'`, `'ArrowLeft'`). |
| Return | Boolean indicating pressed state. |

**Syntax Rules:**

1. Add `keydown` and `keyup` listeners.
2. Set pressed state on keydown; reset on keyup.
3. Clean up listeners on unmount.

**Constraints and Limitations:**

- Key names must match the `KeyboardEvent.key` values.
- Does not handle key combinations natively (compose with multiple `useKeyPress` calls).

#### Annotated Code Example

```jsx
function useKeyPress(targetKey) {
  const [isPressed, setIsPressed] = useState(false);

  useEffect(() => {
    const handleDown = (e) => { if (e.key === targetKey) setIsPressed(true); };
    const handleUp = (e) => { if (e.key === targetKey) setIsPressed(false); };

    window.addEventListener('keydown', handleDown);
    window.addEventListener('keyup', handleUp);

    return () => {
      window.removeEventListener('keydown', handleDown);
      window.removeEventListener('keyup', handleUp);
    };
  }, [targetKey]);

  return isPressed;
}

// Usage
function Modal({ onClose }) {
  const isEscapePressed = useKeyPress('Escape');
  useEffect(() => { if (isEscapePressed) onClose(); }, [isEscapePressed, onClose]);
  return <div>Modal content (press Escape to close)</div>;
}
```

**Expected Output:** The modal closes when the Escape key is pressed.

#### Real-World Cases

- **Keyboard shortcuts:** Ctrl+S to save, Escape to close modals.
- **Games:** Detecting arrow keys for player movement.
- **Accessibility:** Tracking modifier keys for custom keyboard interactions.

#### References

- react-use — useKeyPress — https://github.com/streamich/react-use/blob/master/docs/useKeyPress.md
- @zl-asica/react — useKeyPress — https://jsr.io/@zl-asica/react/doc/use-key-press

---

### 6.12 `useHover`

#### Definitions

**Core Definition:** `useHover` is a custom Hook that tracks whether the mouse is hovering over a referenced element.

**Technical Definition:** `useHover()` returns a ref and a boolean `hovered` state. It attaches `mouseenter` and `mouseleave` listeners to the referenced element and updates the state accordingly.

**Beginner-Friendly Explanation:** `useHover` tells you when the mouse is over a specific element. It is useful for tooltips, hover effects, and interactive UI.

#### Purposes

- To show tooltips or popovers on hover.
- To apply hover-based styling or animations.
- To track hover state for analytics or conditional rendering.

#### Syntax Rules and Structure

```jsx
const { ref, hovered } = useHover();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `ref` | Attach to the element to observe. |
| `hovered` | Boolean indicating hover state. |

**Syntax Rules:**

1. Attach `mouseenter` and `mouseleave` listeners to the ref'd element.
2. Update state on enter/leave.
3. Clean up listeners on unmount.

**Constraints and Limitations:**

- Does not work on touch devices (no hover state).
- Requires a DOM element ref.

#### Annotated Code Example

```jsx
function useHover() {
  const ref = useRef(null);
  const [hovered, setHovered] = useState(false);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const enter = () => setHovered(true);
    const leave = () => setHovered(false);

    element.addEventListener('mouseenter', enter);
    element.addEventListener('mouseleave', leave);

    return () => {
      element.removeEventListener('mouseenter', enter);
      element.removeEventListener('mouseleave', leave);
    };
  }, []);

  return { ref, hovered };
}

// Usage
function Tooltip({ text, children }) {
  const { ref, hovered } = useHover();
  return (
    <span ref={ref} style={{ position: 'relative' }}>
      {children}
      {hovered && <span className="tooltip">{text}</span>}
    </span>
  );
}
```

**Expected Output:** The tooltip appears when the mouse enters the span and disappears on leave.

#### Real-World Cases

- **Tooltips:** Showing additional information on hover.
- **Hover cards:** Displaying user profile previews on hover.
- **Interactive charts:** Highlighting data points on hover.

#### References

- GitHub — useHover (siberiacancode/reactuse) — https://github.com/siberiacancode/reactuse/blob/main/packages/reactuse/src/hooks/useHover/useHover.md
- @usefy/use-hover — https://www.npmjs.com/package/@usefy/use-hover

---

### 6.13 `useFocus`

#### Definitions

**Core Definition:** `useFocus` is a custom Hook that tracks and controls the focus state of a specific DOM element.

**Technical Definition:** `useFocus(ref)` returns a tuple `[isFocused, setFocused]`. It listens to `focus` and `blur` events on the referenced element and provides a setter that calls `element.focus()` or `element.blur()` to programmatically control focus.

**Beginner-Friendly Explanation:** `useFocus` tells you if an input is focused and lets you focus or blur it programmatically.

#### Purposes

- To track focus state for styling or validation.
- To programmatically focus an input (e.g., after a modal opens).
- To implement focus management in forms.

#### Syntax Rules and Structure

```jsx
const [isFocused, setFocused] = useFocus(ref);
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `isFocused` | Boolean indicating focus state. |
| `setFocused` | Function to focus (`true`) or blur (`false`). |

**Syntax Rules:**

1. Attach `focus` and `blur` listeners to the referenced element.
2. The setter calls `element.focus()` or `element.blur()`.
3. Clean up listeners on unmount.

**Constraints and Limitations:**

- Does not work with elements that are not focusable (e.g., `<div>` without `tabIndex`).
- Focus state can be lost when the element is unmounted.

#### Annotated Code Example

```jsx
function useFocus(ref) {
  const [isFocused, setIsFocused] = useState(false);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const onFocus = () => setIsFocused(true);
    const onBlur = () => setIsFocused(false);

    element.addEventListener('focus', onFocus);
    element.addEventListener('blur', onBlur);

    return () => {
      element.removeEventListener('focus', onFocus);
      element.removeEventListener('blur', onBlur);
    };
  }, [ref]);

  const setFocused = (shouldFocus) => {
    if (shouldFocus) ref.current?.focus();
    else ref.current?.blur();
  };

  return [isFocused, setFocused];
}
```

**Expected Output:** `isFocused` is `true` when the input is focused. Calling `setFocused(true)` focuses the input programmatically.

#### Real-World Cases

- **Form validation:** Showing error styles only when a field is focused and invalid.
- **Auto-focus:** Focusing a search input when a modal opens.
- **Focus traps:** Managing focus in modals and dialogs.

#### References

- GitHub — useFocus — https://github.com/siberiacancode/reactuse/blob/main/packages/reactuse/src/hooks/useFocus/useFocus.md
- GitHub — React useFocus Hook: Track & Control Element Focus State — https://github.com/reactive-react/use-focus/blob/main/README.md

---

### 6.14 `useForm`

#### Definitions

**Core Definition:** `useForm` is a custom Hook (often provided by a library like React Hook Form) that manages form state, validation, and submission.

**Technical Definition:** `useForm()` returns `register`, `handleSubmit`, `formState`, and other utilities. It minimizes re-renders by using uncontrolled inputs with refs, and provides built-in validation rules. The most popular implementation is from `react-hook-form`.

**Beginner-Friendly Explanation:** `useForm` handles all the boring parts of forms — tracking values, validating inputs, and handling submission — so you can focus on the UI.

#### Purposes

- To manage form state without writing `useState` for every field.
- To provide performant validation with minimal re-renders.
- To handle form submission and error display consistently.

#### Syntax Rules and Structure

```jsx
const { register, handleSubmit, formState: { errors } } = useForm();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `register` | Connects an input to the form. |
| `handleSubmit` | Wraps the submit handler with validation. |
| `formState` | Contains `errors`, `isSubmitting`, etc. |

**Syntax Rules:**

1. Call `useForm` at the top of the component.
2. Spread `{...register('fieldName')}` onto inputs.
3. Pass the submit handler through `handleSubmit`.

**Constraints and Limitations:**

- `react-hook-form` is an external library, not a built-in React Hook.
- Validation rules are library-specific.

#### Annotated Code Example

```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: 'Email is required' })} />
      {errors.email && <p>{errors.email.message}</p>}

      <input {...register('password', { required: 'Password is required' })} />
      {errors.password && <p>{errors.password.message}</p>}

      <button type="submit">Log In</button>
    </form>
  );
}
```

**Expected Output:** Submitting the form with empty fields displays the validation errors. With valid data, `onSubmit` logs the form values.

#### Real-World Cases

- **Login/signup forms:** Validation and submission handling.
- **Settings forms:** Managing many input fields with minimal boilerplate.
- **Dynamic forms:** Adding/removing fields with `useFieldArray`.

#### References

- React Hook Form — useForm — https://react-hook-form.com/docs/useform
- CoreUI — How to use React Hook Form — https://coreui.io/blog/how-to-use-react-hook-form/

---

### 6.15 `useToggle`

#### Definitions

**Core Definition:** `useToggle` is a custom Hook that manages a boolean state with a toggle function.

**Technical Definition:** `useToggle(initialValue)` returns `[value, toggle]`, where `toggle` flips the boolean. It is a lightweight abstraction over `useState` for the common toggle pattern.

**Beginner-Friendly Explanation:** `useToggle` is a simple Hook for true/false state — like opening and closing a modal or switching a theme.

#### Purposes

- To manage boolean UI state (open/closed, on/off) without writing repetitive toggle logic.
- To provide a consistent API for toggles across components.
- To reduce boilerplate in modals, dropdowns, and switches.

#### Syntax Rules and Structure

```jsx
const [value, toggle] = useToggle(false);
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `value` | Boolean state. |
| `toggle` | Function that flips the state. |

**Syntax Rules:**

1. Use `useState` internally.
2. Return a stable toggle function with `useCallback`.

**Constraints and Limitations:**

- Only manages boolean state — for multi-value toggles, use a different Hook.

#### Annotated Code Example

```jsx
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

// Usage
function Modal() {
  const [isOpen, toggleOpen] = useToggle(false);
  return (
    <>
      <button onClick={toggleOpen}>Toggle Modal</button>
      {isOpen && <div className="modal">Modal content</div>}
    </>
  );
}
```

**Expected Output:** Clicking the button toggles the modal open and closed.

#### Real-World Cases

- **Modals and dialogs:** Open/close state.
- **Theme switching:** Light/dark mode toggle.
- **Accordions:** Expand/collapse sections.

#### References

- Educative — useToggle custom hook — https://www.educative.io/answers/how-to-create-a-custom-toggle-hook-in-react
- react-simplikit — useToggle — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useToggle/useToggle.md

---

### 6.16 `useCounter`

#### Definitions

**Core Definition:** `useCounter` is a custom Hook that manages a numeric counter with increment, decrement, and reset operations.

**Technical Definition:** `useCounter(initialValue, options)` returns `{ count, increment, decrement, reset }`. It encapsulates `useState` with `useCallback` for stable action functions.

**Beginner-Friendly Explanation:** `useCounter` gives you a number and functions to increase, decrease, or reset it. It is useful for quantity selectors, pagination, and scoreboards.

#### Purposes

- To manage numeric state with common operations.
- To avoid duplicating increment/decrement logic.
- To provide stable action functions for memoized components.

#### Syntax Rules and Structure

```jsx
const { count, increment, decrement, reset } = useCounter(0, { step: 1 });
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `count` | Current numeric value. |
| `increment` | Increases by `step`. |
| `decrement` | Decreases by `step`. |
| `reset` | Resets to initial value. |

**Syntax Rules:**

1. Use `useState` for the count.
2. Wrap actions in `useCallback`.

**Constraints and Limitations:**

- Step size and bounds should be configurable for flexibility.

#### Annotated Code Example

```jsx
function useCounter(initial = 0, step = 1) {
  const [count, setCount] = useState(initial);

  const increment = useCallback(() => setCount((c) => c + step), [step]);
  const decrement = useCallback(() => setCount((c) => c - step), [step]);
  const reset = useCallback(() => setCount(initial), [initial]);

  return { count, increment, decrement, reset };
}

// Usage
function QuantitySelector() {
  const { count, increment, decrement } = useCounter(1, 1);
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
    </div>
  );
}
```

**Expected Output:** The quantity increases and decreases by 1.

#### Real-World Cases

- **E-commerce:** Quantity selectors for cart items.
- **Pagination:** Page number counters.
- **Games:** Score or life counters.

#### References

- Toss React Simplikit — useCounter — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useCounter/useCounter.md
- react-use — useCounter — https://github.com/streamich/react-use/blob/master/docs/useCounter.md

---

### 6.17 `useInterval`

#### Definitions

**Core Definition:** `useInterval` is a custom Hook that executes a callback repeatedly at a specified interval, without stale closures.

**Technical Definition:** `useInterval(callback, delay)` uses a ref to store the latest callback and sets up `setInterval` only when the delay changes. Passing `null` as the delay pauses the interval. This pattern solves the stale closure problem that occurs with `useEffect` + `setInterval`.

**Beginner-Friendly Explanation:** `useInterval` runs a function every X milliseconds. Unlike a raw `setInterval`, it always uses the latest version of your function and cleans up automatically.

#### Purposes

- To run periodic tasks (polling, timers, animations).
- To avoid stale closures in interval callbacks.
- To provide declarative pause/resume via the delay parameter.

#### Syntax Rules and Structure

```jsx
useInterval(callback, delay);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `callback` | Function to run on each interval. |
| `delay` | Milliseconds between runs; `null` to pause. |

**Syntax Rules:**

1. Store the latest callback in a ref.
2. Set up `setInterval` in an effect that depends only on `delay`.
3. Clear the interval on cleanup.

**Constraints and Limitations:**

- The first execution occurs after the first delay (not immediately).
- Intervals are not guaranteed to be exact (browser throttling).

#### Annotated Code Example

```jsx
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

// Usage
function Timer() {
  const [count, setCount] = useState(0);
  useInterval(() => setCount((c) => c + 1), 1000);
  return <p>Seconds: {count}</p>;
}
```

**Expected Output:** The counter increments every second without stale closure issues.

#### Real-World Cases

- **Polling:** Fetching data every N seconds.
- **Timers:** Countdown or stopwatch components.
- **Animations:** Updating animation frames at regular intervals.

#### References

- Overreacted — Making setInterval Declarative with React Hooks — https://overreacted.io/making-setinterval-declarative-with-react-hooks/
- react-use — useInterval — https://github.com/streamich/react-use/blob/master/docs/useInterval.md

---

### 6.18 `useTimeout`

#### Definitions

**Core Definition:** `useTimeout` is a custom Hook that executes a callback after a specified delay, with automatic cleanup.

**Technical Definition:** `useTimeout(callback, delay)` sets a `setTimeout` that runs the callback after `delay` milliseconds. It cleans up the timer on unmount or when dependencies change, preventing memory leaks and state updates on unmounted components.

**Beginner-Friendly Explanation:** `useTimeout` runs a function once after a delay — like showing a notification after 3 seconds. It cleans up after itself so you do not get errors.

#### Purposes

- To run a one-time delayed action (notifications, redirects).
- To avoid manual `setTimeout` cleanup in components.
- To provide declarative timeout management.

#### Syntax Rules and Structure

```jsx
useTimeout(callback, delay);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `callback` | Function to run after the delay. |
| `delay` | Milliseconds; `null` to cancel. |

**Syntax Rules:**

1. Store the timer ID in a ref.
2. Clear the timer on cleanup and dependency change.

**Constraints and Limitations:**

- The callback is not re-run if the delay changes before it fires (depending on implementation).
- `setTimeout` is not exact.

#### Annotated Code Example

```jsx
function useTimeout(callback, delay) {
  const savedCallback = useRef(callback);

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay === null) return;
    const id = setTimeout(() => savedCallback.current(), delay);
    return () => clearTimeout(id);
  }, [delay]);
}

// Usage
function Notification({ message }) {
  const [visible, setVisible] = useState(true);
  useTimeout(() => setVisible(false), 3000);
  if (!visible) return null;
  return <div className="notification">{message}</div>;
}
```

**Expected Output:** The notification disappears 3 seconds after it appears.

#### Real-World Cases

- **Toast notifications:** Auto-dismiss after a few seconds.
- **Redirects:** Navigating after a delay.
- **Debounced UI elements:** Delayed show/hide.

#### References

- Toss React Simplikit — useTimeout — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useTimeout/useTimeout.md
- beautiful-react-hooks — useTimeout — https://github.com/beautifulinteractions/beautiful-react-hooks/blob/master/src/useTimeout.md

---

### 6.19 `usePrevious`

#### Definitions

**Core Definition:** `usePrevious` is a custom Hook that returns the value from the previous render.

**Technical Definition:** `usePrevious(value)` stores the current value in a ref and returns the value from the previous render. It uses a ref to persist the value across renders without triggering re-renders.

**Beginner-Friendly Explanation:** `usePrevious` tells you what a value was on the last render. It is useful for comparisons, animations, and detecting changes.

#### Purposes

- To compare current and previous values (e.g., detecting direction of change).
- To trigger animations based on state transitions.
- To avoid redundant effects when a value has not changed.

#### Syntax Rules and Structure

```jsx
const previousValue = usePrevious(value);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `value` | Current value. |
| Return | Previous render's value (or `undefined` initially). |

**Syntax Rules:**

1. Use a ref to store the previous value.
2. Update the ref in an effect after render.

**Constraints and Limitations:**

- The previous value is `undefined` on the first render.
- Does not track changes across component unmounts.

#### Annotated Code Example

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}, Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

**Expected Output:** After clicking "+1", the display shows the current count and the previous count.

#### Real-World Cases

- **Animation direction:** Determining whether a value increased or decreased.
- **Analytics:** Tracking state transitions.
- **Form validation:** Comparing current and previous input values.

#### References

- react-use — usePrevious — https://github.com/streamich/react-use/blob/master/docs/usePrevious.md
- @reactuses/core — usePrevious — https://www.npmjs.com/package/@reactuses/core

---

### 6.20 `useIsMounted`

#### Definitions

**Core Definition:** `useIsMounted` is a custom Hook that returns whether the component is currently mounted.

**Technical Definition:** `useIsMounted()` returns a ref or boolean that is `true` after the component mounts and `false` after it unmounts. It uses `useEffect` with a cleanup function to set the flag. It is commonly used to prevent state updates on unmounted components.

**Beginner-Friendly Explanation:** `useIsMounted` tells you if the component is still on the screen. It helps avoid errors when async operations finish after the component has been removed.

#### Purposes

- To prevent "Can't perform a state update on an unmounted component" warnings.
- To guard async callbacks that may resolve after unmount.
- To conditionally run logic only while mounted.

#### Syntax Rules and Structure

```jsx
const isMounted = useIsMounted();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `isMounted` | `true` while mounted, `false` after unmount. |

**Syntax Rules:**

1. Use a ref initialized to `false`.
2. Set `ref.current = true` in an effect and `false` in cleanup.

**Constraints and Limitations:**

- React 18+ no longer warns about state updates on unmounted components, but the Hook is still useful for avoiding unnecessary work.
- The ref-based approach does not trigger re-renders.

#### Annotated Code Example

```jsx
function useIsMounted() {
  const isMounted = useRef(false);

  useEffect(() => {
    isMounted.current = true;
    return () => { isMounted.current = false; };
  }, []);

  return isMounted;
}

// Usage
function DataFetcher() {
  const isMounted = useIsMounted();
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data')
      .then((r) => r.json())
      .then((d) => {
        if (isMounted.current) setData(d);
      });
  }, [isMounted]);

  return <div>{data ? data.name : 'Loading...'}</div>;
}
```

**Expected Output:** The state update is skipped if the component unmounts before the fetch resolves.

#### Real-World Cases

- **Async data fetching:** Guarding `setState` after unmount.
- **Timers and intervals:** Preventing callbacks from running after unmount.
- **Event listeners:** Checking mount status before handling events.

#### References

- GitHub — useIsMounted — https://github.com/jmlweb/isMounted
- @utilityjs/use-is-mounted — https://www.npmjs.com/package/@utilityjs/use-is-mounted

---

### 6.21 `useAsync`

#### Definitions

**Core Definition:** `useAsync` is a custom Hook that wraps an async function and provides `data`, `loading`, and `error` states.

**Technical Definition:** `useAsync(fn, deps)` executes the async function and tracks its lifecycle. It returns `{ data, loading, error, execute }` and handles race conditions by ignoring stale results. It is similar to `useFetch` but more generic — it can wrap any async function.

**Beginner-Friendly Explanation:** `useAsync` runs an async function (like an API call) and tells you when it's loading, when it's done, and if it failed. It works with any async function, not just `fetch`.

#### Purposes

- To manage any async operation (API calls, database queries, file reads).
- To provide consistent loading/error states for async logic.
- To prevent race conditions with stale async results.

#### Syntax Rules and Structure

```jsx
const { data, loading, error, execute } = useAsync(asyncFn, deps);
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `data` | Resolved value. |
| `loading` | Boolean pending state. |
| `error` | Error if rejected. |
| `execute` | Manual trigger function. |

**Syntax Rules:**

1. Use `useState` for data, loading, and error.
2. Use `useEffect` with a cancellation flag to ignore stale results.
3. Memoize the async function or use a ref to avoid infinite loops.

**Constraints and Limitations:**

- Does not cache results across components.
- Requires careful dependency management.

#### Annotated Code Example

```jsx
function useAsync(asyncFn, deps = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    setError(null);

    asyncFn()
      .then((result) => {
        if (!cancelled) setData(result);
      })
      .catch((err) => {
        if (!cancelled) setError(err);
      })
      .finally(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; };
  }, deps);

  return { data, loading, error };
}
```

**Expected Output:** While the async function runs, `loading` is `true`. On success, `data` is set. On failure, `error` is set. Stale results are ignored.

#### Real-World Cases

- **API calls:** Wrapping `fetch` or `axios` calls.
- **Database queries:** Running IndexedDB or SQLite queries.
- **File operations:** Reading files in Electron or Node.js environments.

#### References

- react-async — useAsync — https://docs.react-async.com/api/use-async
- JSR — useAsync — https://jsr.io/@zap-studio/react-hooks/doc/use-async

---

### 6.22 `usePagination`

#### Definitions

**Core Definition:** `usePagination` is a custom Hook that manages pagination state and computes page ranges.

**Technical Definition:** `usePagination({ currentPage, totalPages, ... })` returns page numbers, navigation handlers (`next`, `previous`, `goTo`), and flags (`hasNext`, `hasPrevious`). It encapsulates the arithmetic for pagination UI.

**Beginner-Friendly Explanation:** `usePagination` handles the logic of page numbers, next/previous buttons, and page ranges so you can build pagination UIs quickly.

#### Purposes

- To manage current page state and navigation.
- To compute page ranges with ellipses for large page counts.
- To provide consistent pagination logic across components.

#### Syntax Rules and Structure

```jsx
const { pages, next, previous, goTo, hasNext, hasPrevious } = usePagination({
  totalPages: 10,
  currentPage: 1,
});
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `pages` | Array of page numbers (or ranges). |
| `next`, `previous` | Navigation functions. |
| `goTo(page)` | Jump to a specific page. |
| `hasNext`, `hasPrevious` | Boundary flags. |

**Syntax Rules:**

1. Manage current page with `useState` or accept it as a controlled prop.
2. Use `useMemo` to compute the page range.

**Constraints and Limitations:**

- Does not fetch data — only manages pagination state.
- Range computation can be customized for different UI patterns.

#### Annotated Code Example

```jsx
function usePagination(totalPages, initialPage = 1) {
  const [currentPage, setCurrentPage] = useState(initialPage);

  const pages = useMemo(() => {
    return Array.from({ length: totalPages }, (_, i) => i + 1);
  }, [totalPages]);

  const next = useCallback(() => {
    setCurrentPage((p) => Math.min(p + 1, totalPages));
  }, [totalPages]);

  const previous = useCallback(() => {
    setCurrentPage((p) => Math.max(p - 1, 1));
  }, []);

  const goTo = useCallback((page) => {
    setCurrentPage(Math.max(1, Math.min(page, totalPages)));
  }, [totalPages]);

  return {
    currentPage,
    pages,
    next,
    previous,
    goTo,
    hasNext: currentPage < totalPages,
    hasPrevious: currentPage > 1,
  };
}
```

**Expected Output:** Calling `next` advances the page; `previous` goes back; `goTo(5)` jumps to page 5.

#### Real-World Cases

- **Table pagination:** Navigating through table pages.
- **Search results:** Paging through search results.
- **Image galleries:** Paginating through photo collections.

#### References

- GitHub — usePagination — https://github.com/mantinedev/mantine/blob/master/packages/%40mantine/hooks/src/use-pagination/use-pagination.ts
- Mantine — usePagination — https://mantine.dev/hooks/use-pagination/

---

### 6.23 `useInfiniteScroll`

#### Definitions

**Core Definition:** `useInfiniteScroll` is a custom Hook that triggers a callback when the user scrolls near the bottom of a container.

**Technical Definition:** `useInfiniteScroll(callback, options)` returns a `ref` (sentinel element) and uses the Intersection Observer API to detect when the sentinel becomes visible. It calls `callback` to load more data.

**Beginner-Friendly Explanation:** `useInfiniteScroll` automatically loads more content when you scroll to the bottom of a list — like social media feeds.

#### Purposes

- To implement infinite scrolling feeds without manual scroll listeners.
- To load data lazily as the user reaches the end of the content.
- To provide a smooth, app-like browsing experience.

#### Syntax Rules and Structure

```jsx
const { sentinelRef } = useInfiniteScroll({ loadMore, hasMore });
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `loadMore` | Async function to load more items. |
| `hasMore` | Boolean indicating if more data exists. |
| `sentinelRef` | Attach to a sentinel element at the end of the list. |

**Syntax Rules:**

1. Use Intersection Observer to watch the sentinel element.
2. Call `loadMore` when the sentinel becomes visible.
3. Stop observing when `hasMore` is `false`.

**Constraints and Limitations:**

- Requires a sentinel element at the end of the list.
- Does not manage data fetching — only triggers the callback.

#### Annotated Code Example

```jsx
function useInfiniteScroll(loadMore, hasMore) {
  const sentinelRef = useRef(null);

  useEffect(() => {
    if (!hasMore || !sentinelRef.current) return;
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) loadMore();
      },
      { threshold: 0.1 }
    );
    observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [loadMore, hasMore]);

  return { sentinelRef };
}

// Usage
function Feed() {
  const [items, setItems] = useState([]);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = useCallback(async () => {
    const newItems = await fetchMoreItems();
    setItems((prev) => [...prev, ...newItems]);
    if (newItems.length === 0) setHasMore(false);
  }, []);

  const { sentinelRef } = useInfiniteScroll(loadMore, hasMore);

  return (
    <ul>
      {items.map((item) => <li key={item.id}>{item.name}</li>)}
      <div ref={sentinelRef} />
    </ul>
  );
}
```

**Expected Output:** When the user scrolls to the sentinel, `loadMore` is called and new items are appended.

#### Real-World Cases

- **Social feeds:** Loading more posts as the user scrolls.
- **Product lists:** Infinite product grids in e-commerce.
- **Chat history:** Loading older messages when scrolling up.

#### References

- @bongsik/infinite-scroll — https://www.npmjs.com/package/@bongsik/infinite-scroll
- ReactUse — useInfiniteScroll — https://reactuse.org/hooks/use-infinite-scroll

---

### 6.24 `useCopyToClipboard`

#### Definitions

**Core Definition:** `useCopyToClipboard` is a custom Hook that copies text to the clipboard and tracks whether the copy succeeded.

**Technical Definition:** `useCopyToClipboard()` returns `[copiedText, copy]` or `{ copied, copy }`. It uses `navigator.clipboard.writeText()` with a fallback for older browsers, and typically resets the copied state after a timeout.

**Beginner-Friendly Explanation:** `useCopyToClipboard` copies text to the clipboard when you call `copy('text')` and tells you if it worked.

#### Purposes

- To provide a reusable copy-to-clipboard interaction.
- To handle the Clipboard API with fallbacks and error handling.
- To show feedback ("Copied!") after copying.

#### Syntax Rules and Structure

```jsx
const [copiedText, copy] = useCopyToClipboard();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `copiedText` | The last copied text (or `null`). |
| `copy(text)` | Copies the text to the clipboard. |

**Syntax Rules:**

1. Use `navigator.clipboard.writeText()` with a try/catch.
2. Fallback to `document.execCommand('copy')` for older browsers.
3. Reset the copied state after a timeout.

**Constraints and Limitations:**

- The Clipboard API requires a secure context (HTTPS or localhost).
- User activation is required (copy must be triggered by a user gesture).

#### Annotated Code Example

```jsx
function useCopyToClipboard(resetDelay = 2000) {
  const [copiedText, setCopiedText] = useState(null);

  const copy = useCallback(async (text) => {
    if (!navigator?.clipboard) {
      console.warn('Clipboard not supported');
      return;
    }
    try {
      await navigator.clipboard.writeText(text);
      setCopiedText(text);
      setTimeout(() => setCopiedText(null), resetDelay);
    } catch (err) {
      console.error('Copy failed:', err);
      setCopiedText(null);
    }
  }, [resetDelay]);

  return [copiedText, copy];
}

// Usage
function CopyButton({ text }) {
  const [copiedText, copy] = useCopyToClipboard();
  return (
    <button onClick={() => copy(text)}>
      {copiedText ? 'Copied!' : 'Copy'}
    </button>
  );
}
```

**Expected Output:** Clicking the button copies `text` and shows "Copied!" for 2 seconds.

#### Real-World Cases

- **Code blocks:** "Copy code" buttons in documentation.
- **Share links:** Copying URLs to the clipboard.
- **Invite codes:** Copying referral codes.

#### References

- JSR — useCopyToClipboard — https://jsr.io/@pokactx/react-cliptext/doc/use-copy-to-clipboard
- @koobiq/react-core — useCopyToClipboard — https://github.com/koobiq/react/blob/main/packages/react-core/src/hooks/useCopyToClipboard/useCopyToClipboard.md

---

### 6.25 `useDarkMode`

#### Definitions

**Core Definition:** `useDarkMode` is a custom Hook that manages a dark/light theme state, persisting it to `localStorage` and syncing with the system preference.

**Technical Definition:** `useDarkMode()` returns `[isDark, toggle]`. It reads the system preference via `prefers-color-scheme` when no stored value exists, applies a CSS class to `<html>`, and persists the choice to `localStorage`.

**Beginner-Friendly Explanation:** `useDarkMode` handles the entire dark mode setup — detecting the system preference, saving the user's choice, and applying the theme.

#### Purposes

- To implement dark mode with system preference detection.
- To persist theme choice across sessions.
- To apply the theme globally via a CSS class.

#### Syntax Rules and Structure

```jsx
const [isDark, toggle] = useDarkMode();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `isDark` | Boolean indicating dark mode. |
| `toggle` | Function to switch modes. |

**Syntax Rules:**

1. Read `localStorage` for stored preference.
2. Fall back to `window.matchMedia('(prefers-color-scheme: dark)')`.
3. Apply `dark` class to `document.documentElement`.

**Constraints and Limitations:**

- Requires CSS that responds to the `dark` class.
- System preference changes require a listener on `matchMedia`.

#### Annotated Code Example

```jsx
function useDarkMode() {
  const [isDark, setIsDark] = useState(() => {
    const stored = localStorage.getItem('theme');
    if (stored) return stored === 'dark';
    return window.matchMedia('(prefers-color-scheme: dark)').matches;
  });

  useEffect(() => {
    document.documentElement.classList.toggle('dark', isDark);
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [isDark]);

  const toggle = useCallback(() => setIsDark((d) => !d), []);

  return [isDark, toggle];
}
```

**Expected Output:** The `<html>` element gets the `dark` class when dark mode is enabled, and the preference is persisted.

#### Real-World Cases

- **Websites:** Theme toggles in the header.
- **Apps:** Automatic dark mode based on system settings.
- **Dashboards:** User-selectable themes.

#### References

- GitHub — useDarkMode — https://github.com/reactuses/reactuse/blob/main/packages/core/src/useDarkMode/useDarkMode.md
- use-dark-mode — https://github.com/donavon/use-dark-mode

---

### 6.26 `useScrollPosition`

#### Definitions

**Core Definition:** `useScrollPosition` is a custom Hook that tracks the current scroll position (`x` and `y`) of the window or a target element.

**Technical Definition:** `useScrollPosition()` returns `{ x, y }` and listens to `scroll` events, optionally throttled for performance.

**Beginner-Friendly Explanation:** `useScrollPosition` tells you how far the user has scrolled on the page.

#### Purposes

- To track scroll position for progress indicators.
- To trigger effects based on scroll depth.
- To implement sticky headers or parallax.

#### Syntax Rules and Structure

```jsx
const { x, y } = useScrollPosition();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `x`, `y` | Horizontal and vertical scroll positions. |

**Syntax Rules:**

1. Add a `scroll` listener to `window` or a target element.
2. Throttle the handler for performance.
3. Clean up on unmount.

**Constraints and Limitations:**

- Scroll events fire frequently — throttle or debounce for performance.
- Does not account for scroll position in iframes.

#### Annotated Code Example

```jsx
function useScrollPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleScroll = () => {
      setPosition({ x: window.scrollX, y: window.scrollY });
    };

    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return position;
}
```

**Expected Output:** `y` updates as the user scrolls down the page.

#### Real-World Cases

- **Reading progress bars:** Showing how far the user has scrolled.
- **Sticky navigation:** Adding a shadow to the header after scrolling.
- **Back-to-top buttons:** Showing the button after a certain scroll depth.

#### References

- @n8tb1t/use-scroll-position — https://www.npmjs.com/package/@n8tb1t/use-scroll-position
- JSR — useScrollPosition — https://jsr.io/@zap-studio/react-hooks/doc/use-scroll-position

---

### 6.27 `useWindowSize`

#### Definitions

**Core Definition:** `useWindowSize` is a custom Hook that tracks the browser window's `innerWidth` and `innerHeight`.

**Technical Definition:** `useWindowSize()` returns `{ width, height }` and updates on `resize` events. It is typically SSR-safe with default dimensions for the server.

**Beginner-Friendly Explanation:** `useWindowSize` tells you the current width and height of the browser window.

#### Purposes

- To build responsive layouts in JavaScript.
- To conditionally render components based on window size.
- To compute positions or dimensions dynamically.

#### Syntax Rules and Structure

```jsx
const { width, height } = useWindowSize();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `width` | Window inner width. |
| `height` | Window inner height. |

**Syntax Rules:**

1. Read `window.innerWidth` and `window.innerHeight` on mount.
2. Add a `resize` listener.
3. Clean up on unmount.

**Constraints and Limitations:**

- `resize` fires frequently — throttle for performance.
- SSR requires default values.

#### Annotated Code Example

```jsx
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}
```

**Expected Output:** `width` and `height` update when the window is resized.

#### Real-World Cases

- **Responsive components:** Switching layouts based on window size.
- **Canvas sizing:** Setting canvas dimensions to match the window.
- **Modal positioning:** Centering modals based on viewport size.

#### References

- GitHub — useWindowSize — https://github.com/hamo-org/hamo/blob/main/docs/use-window-size.md
- react-use — useWindowSize — https://github.com/streamich/react-use/blob/master/docs/useWindowSize.md

---

### 6.28 `useOnlineStatus`

#### Definitions

**Core Definition:** `useOnlineStatus` is a custom Hook that tracks whether the browser is online or offline.

**Technical Definition:** `useOnlineStatus()` returns a boolean using `navigator.onLine` and listens to `online`/`offline` events.

**Beginner-Friendly Explanation:** `useOnlineStatus` tells you if the user has an internet connection.

#### Purposes

- To show offline banners or indicators.
- To disable actions that require a network connection.
- To trigger data sync when connectivity is restored.

#### Syntax Rules and Structure

```jsx
const isOnline = useOnlineStatus();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `isOnline` | `true` if online, `false` if offline. |

**Syntax Rules:**

1. Initialize from `navigator.onLine`.
2. Listen to `online` and `offline` events.
3. Clean up listeners.

**Constraints and Limitations:**

- `navigator.onLine` may report `true` even without internet access (e.g., connected to a local network without internet).
- Not a replacement for actual network request validation.

#### Annotated Code Example

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

**Expected Output:** The boolean updates when the browser goes online or offline.

#### Real-World Cases

- **Offline banners:** Showing "You are offline" messages.
- **PWA features:** Caching data when offline.
- **Retry logic:** Re-fetching data when connectivity is restored.

#### References

- react-use-online-status — https://www.npmjs.com/package/react-use-online-status
- JSR — useOnlineStatus — https://jsr.io/@zap-studio/react-hooks/doc/use-online-status

---

### 6.29 `useGeolocation`

#### Definitions

**Core Definition:** `useGeolocation` is a custom Hook that retrieves and tracks the user's geographic location using the Geolocation API.

**Technical Definition:** `useGeolocation(options)` returns `{ coordinates, error, loading, isSupported }`. It supports one-time position retrieval (`getCurrentPosition`) and continuous tracking (`watchPosition`).

**Beginner-Friendly Explanation:** `useGeolocation` tells you where the user is (with their permission) — latitude, longitude, and accuracy.

#### Purposes

- To display user location on a map.
- To provide location-based services (store finders, weather).
- To calculate distances or directions.

#### Syntax Rules and Structure

```jsx
const { coordinates, error, loading } = useGeolocation();
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `coordinates` | `{ latitude, longitude, accuracy }`. |
| `error` | Error object if retrieval failed. |
| `loading` | Boolean pending state. |

**Syntax Rules:**

1. Call `navigator.geolocation.getCurrentPosition` or `watchPosition`.
2. Handle permission errors.
3. Clean up the watcher on unmount.

**Constraints and Limitations:**

- Requires user permission (prompted by the browser).
- Not available in insecure contexts (HTTP).
- Accuracy varies by device and environment.

#### Annotated Code Example

```jsx
function useGeolocation() {
  const [coordinates, setCoordinates] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!navigator.geolocation) {
      setError(new Error('Geolocation not supported'));
      setLoading(false);
      return;
    }

    const watcher = navigator.geolocation.watchPosition(
      (position) => {
        setCoordinates(position.coords);
        setLoading(false);
      },
      (err) => {
        setError(err);
        setLoading(false);
      }
    );

    return () => navigator.geolocation.clearWatch(watcher);
  }, []);

  return { coordinates, error, loading };
}
```

**Expected Output:** `coordinates` contains latitude/longitude once the user grants permission.

#### Real-World Cases

- **Store locators:** Finding nearby stores.
- **Weather apps:** Showing local weather based on position.
- **Delivery apps:** Tracking user location for delivery estimates.

#### References

- GitHub — useGeolocation — https://github.com/reactuses/reactuse/blob/main/packages/core/src/useGeolocation/useGeolocation.md
- JSR — useGeolocation — https://jsr.io/@zap-studio/react-hooks/doc/use-geolocation

---

### 6.30 `usePermission`

#### Definitions

**Core Definition:** `usePermission` is a custom Hook that queries the status of a browser permission (e.g., microphone, camera, notifications).

**Technical Definition:** `usePermission(permissionName)` uses the Permissions API (`navigator.permissions.query`) to return the permission state (`"granted"`, `"denied"`, or `"prompt"`). It listens to the permission's `change` event for updates.

**Beginner-Friendly Explanation:** `usePermission` tells you if the user has granted permission to use the camera, microphone, or other browser features.

#### Purposes

- To check permission status before requesting access.
- To show UI based on whether permission is granted, denied, or prompt.
- To react to permission changes.

#### Syntax Rules and Structure

```jsx
const state = usePermission({ name: 'microphone' });
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `state` | `"granted"`, `"denied"`, `"prompt"`, or `"unsupported"`. |

**Syntax Rules:**

1. Use `navigator.permissions.query({ name })`.
2. Listen to `change` events on the permission status.
3. Handle unsupported browsers gracefully.

**Constraints and Limitations:**

- Permissions API support varies by browser.
- Some permissions require user gesture to query.

#### Annotated Code Example

```jsx
function usePermission(name) {
  const [state, setState] = useState('prompt');

  useEffect(() => {
    if (!navigator.permissions) {
      setState('unsupported');
      return;
    }

    let permissionStatus;
    navigator.permissions.query({ name }).then((status) => {
      permissionStatus = status;
      setState(status.state);
      status.onchange = () => setState(status.state);
    });

    return () => { if (permissionStatus) permissionStatus.onchange = null; };
  }, [name]);

  return state;
}
```

**Expected Output:** `state` reflects the current permission status and updates when the user changes it.

#### Real-World Cases

- **Camera/microphone access:** Checking before starting a video call.
- **Notifications:** Showing a "Enable notifications" button only when permission is `"prompt"`.
- **Location services:** Checking geolocation permission status.

#### References

- react-use — usePermission — https://github.com/streamich/react-use/blob/master/docs/usePermission.md
- JSR — usePermission — https://jsr.io/@zap-studio/react-hooks/doc/use-permission

---

### 6.31 `useBroadcastChannel`

#### Definitions

**Core Definition:** `useBroadcastChannel` is a custom Hook that enables communication between browser tabs, windows, or iframes using the Broadcast Channel API.

**Technical Definition:** `useBroadcastChannel(channelName, onMessage)` creates a `BroadcastChannel` instance and invokes `onMessage` when messages arrive on that channel. It returns a `postMessage` function for sending messages.

**Beginner-Friendly Explanation:** `useBroadcastChannel` lets different tabs of your app talk to each other — like syncing a shopping cart across tabs.

#### Purposes

- To synchronize state across browser tabs.
- To implement cross-tab communication (logout, theme changes).
- To coordinate background tasks across windows.

#### Syntax Rules and Structure

```jsx
const { postMessage } = useBroadcastChannel('channel-name', onMessage);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `channelName` | The broadcast channel name. |
| `onMessage` | Callback for incoming messages. |
| `postMessage` | Function to send messages. |

**Syntax Rules:**

1. Create a `BroadcastChannel` in an effect.
2. Set `onmessage` to the callback.
3. Close the channel on cleanup.

**Constraints and Limitations:**

- Not supported in all browsers (polyfill available).
- Messages are not persisted — late-joining tabs miss previous messages.

#### Annotated Code Example

```jsx
function useBroadcastChannel(channelName, onMessage) {
  const channelRef = useRef(null);

  useEffect(() => {
    const channel = new BroadcastChannel(channelName);
    channelRef.current = channel;
    channel.onmessage = (event) => onMessage(event.data);
    return () => channel.close();
  }, [channelName, onMessage]);

  const postMessage = useCallback((data) => {
    channelRef.current?.postMessage(data);
  }, []);

  return { postMessage };
}
```

**Expected Output:** Messages posted in one tab are received by `onMessage` in other tabs with the same channel name.

#### Real-World Cases

- **Shopping carts:** Syncing cart state across tabs.
- **Authentication:** Logging out all tabs when one logs out.
- **Theme sync:** Applying theme changes across all open tabs.

#### References

- use-broadcast-channel — https://www.npmjs.com/package/use-broadcast-channel
- GitHub — react-broadcast-channel — https://github.com/GuptaSiddhant/react-broadcast-channel

---

### 6.32 `useWebSocket`

#### Definitions

**Core Definition:** `useWebSocket` is a custom Hook that manages a WebSocket connection, including connection status, message handling, and cleanup.

**Technical Definition:** `useWebSocket(url)` returns `{ messages, status, send }`. It creates a `WebSocket`, updates status on `open`/`close`/`error`, appends incoming messages, and closes the connection on unmount.

**Beginner-Friendly Explanation:** `useWebSocket` connects to a WebSocket server and gives you the messages as they arrive, plus a function to send messages back.

#### Purposes

- To build real-time features (chat, live dashboards, notifications).
- To manage the WebSocket lifecycle (connect, reconnect, cleanup).
- To provide a simple API for sending and receiving messages.

#### Syntax Rules and Structure

```jsx
const { messages, status, send } = useWebSocket(url);
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `messages` | Array of received messages. |
| `status` | `"connecting"`, `"connected"`, `"disconnected"`, `"error"`. |
| `send(data)` | Sends a message. |

**Syntax Rules:**

1. Create the `WebSocket` in an effect.
2. Set event handlers (`onopen`, `onmessage`, `onclose`, `onerror`).
3. Close the socket on cleanup.
4. Use `useRef` to access the socket in `send` without dependencies.

**Constraints and Limitations:**

- Reconnection logic must be implemented manually or via a library like `react-use-websocket`.
- WebSocket connections can be interrupted by network changes.

#### Annotated Code Example

```jsx
function useWebSocket(url) {
  const [messages, setMessages] = useState([]);
  const [status, setStatus] = useState('connecting');
  const wsRef = useRef(null);

  useEffect(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;

    ws.onopen = () => setStatus('connected');
    ws.onclose = () => setStatus('disconnected');
    ws.onerror = () => setStatus('error');
    ws.onmessage = (event) => {
      try {
        setMessages((prev) => [...prev, JSON.parse(event.data)]);
      } catch {
        setMessages((prev) => [...prev, event.data]);
      }
    };

    return () => ws.close();
  }, [url]);

  const send = useCallback((data) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(typeof data === 'string' ? data : JSON.stringify(data));
    }
  }, []);

  return { messages, status, send };
}
```

**Expected Output:** Incoming messages are appended to `messages`; `status` reflects the connection state; `send` posts data to the server.

#### Real-World Cases

- **Chat applications:** Real-time messaging.
- **Live dashboards:** Streaming metrics and updates.
- **Collaborative editing:** Syncing changes across users.

#### References

- CoreUI — How to use WebSockets in React — https://coreui.io/answers/how-to-use-websockets-in-react/
- react-use-websocket — https://www.npmjs.com/package/react-use-websocket

---

### 6.33 `useEventSource`

#### Definitions

**Core Definition:** `useEventSource` is a custom Hook that subscribes to Server-Sent Events (SSE) from a server endpoint.

**Technical Definition:** `useEventSource(url, eventNames)` wraps the `EventSource` API. It listens for named SSE events and returns the latest data and connection status. It handles reconnection automatically (built into the `EventSource` API).

**Beginner-Friendly Explanation:** `useEventSource` listens to a stream of updates from the server — like live scores or deployment logs — without needing WebSockets.

#### Purposes

- To receive real-time updates from a server via SSE.
- To handle server-sent events with automatic reconnection.
- To stream logs, notifications, or live data.

#### Syntax Rules and Structure

```jsx
const { data, status } = useEventSource(url, eventNames);
```

**Component Breakdown:**

| Return | Description |
|--------|-------------|
| `data` | Latest event data. |
| `status` | Connection status. |

**Syntax Rules:**

1. Create an `EventSource` in an effect.
2. Add listeners for named events (`addEventListener`).
3. Close the source on cleanup.

**Constraints and Limitations:**

- SSE is one-way (server to client) — use WebSocket for bidirectional communication.
- Limited to text data (JSON is common).
- Browser connection limits may apply.

#### Annotated Code Example

```jsx
function useEventSource(url, eventNames = []) {
  const [data, setData] = useState(null);
  const [status, setStatus] = useState('connecting');

  useEffect(() => {
    const source = new EventSource(url);

    source.onopen = () => setStatus('open');
    source.onerror = () => setStatus('error');

    eventNames.forEach((name) => {
      source.addEventListener(name, (event) => {
        setData(JSON.parse(event.data));
      });
    });

    return () => source.close();
  }, [url, eventNames]);

  return { data, status };
}
```

**Expected Output:** `data` updates whenever the server sends an event with one of the specified names; `status` reflects the connection state.

#### Real-World Cases

- **Live deployment logs:** Streaming build or deploy output.
- **Live scores:** Real-time sports or game updates.
- **Notifications:** Server-pushed alerts without polling.

#### References

- @heecheon92/use-event-source — https://www.npmjs.com/package/@heecheon92/use-event-source
- JSR — useEventSource — https://jsr.io/@zap-studio/react-hooks/doc/use-event-source

---

## 7. Custom Hook Best Practices

### Definitions

**Core Definition:** Custom Hook best practices are guidelines for designing, composing, testing, and maintaining custom Hooks to ensure they are reusable, performant, and easy to reason about.

**Technical Definition:** Best practices encompass single responsibility, stable return types, proper dependency management, composition over deep nesting, testing strategies, and naming conventions. They aim to prevent common pitfalls such as stale closures, infinite loops, unnecessary re-renders, and obscured dependency chains.

**Beginner-Friendly Explanation:** Best practices are the rules of thumb that keep your custom Hooks clean, predictable, and bug-free.

### Purposes

- To prevent stale closures and dependency bugs.
- To keep Hooks focused and testable.
- To make dependency chains explicit and maintainable.
- To ensure consistent APIs across a codebase.
- To optimize performance by avoiding unnecessary re-renders.

### Syntax Rules and Structure

**Best Practices Checklist:**

| Practice | Description |
|----------|-------------|
| **One concern per Hook** | Each Hook should do one thing well (e.g., `useLocalStorage`, not `useEverything`). |
| **Use `useCallback` for returned functions** | Stabilize action functions to prevent unnecessary re-renders. |
| **Use `useMemo` for computed values** | Memoize expensive derivations. |
| **Prefer composition over deep nesting** | Compose Hooks at the component level to keep dependencies visible. |
| **Return tuples for state-like APIs** | `[value, setValue]` allows callers to rename freely. |
| **Return objects for compound APIs** | `{ data, loading, error }` is clearer for multiple values. |
| **Test through the public API** | Use `renderHook`, `act`, and `waitFor`. |
| **Prefer real component tests for simple Hooks** | Use `renderHook` only for complex/reusable Hooks. |
| **Never mock React built-ins** | Use MSW for network mocking. |
| **Always clean up side effects** | Return cleanup functions from effects (intervals, listeners, observers). |

### Constraints and Limitations

- Deeply nested Hook composition makes debugging harder — keep the chain shallow.
- Returning new objects/arrays on every render can break memoization — memoize return values when necessary.
- Hooks that depend on browser APIs require SSR guards.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Well-Designed Hook (Single Responsibility + Stable Returns)

```jsx
import { useState, useCallback } from 'react';

// ✅ Single responsibility: toggle state
// ✅ Stable toggle function with useCallback
// ✅ Tuple return for state-like API
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

// ✅ Composed at the component level, not deeply nested
function Modal() {
  const [isOpen, toggleOpen] = useToggle(false);
  return (
    <>
      <button onClick={toggleOpen}>Toggle</button>
      {isOpen && <div>Modal</div>}
    </>
  );
}
```

**Why this follows best practices:** The Hook does one thing, returns a stable function, and uses a tuple return that callers can rename.

#### Example 2: Testing a Custom Hook

```jsx
import { renderHook, act } from '@testing-library/react';
import { useToggle } from './useToggle';

test('toggles boolean state', () => {
  const { result } = renderHook(() => useToggle(false));

  expect(result.current[0]).toBe(false);

  act(() => {
    result.current[1](); // toggle
  });
  expect(result.current[0]).toBe(true);

  act(() => {
    result.current[1](); // toggle back
  });
  expect(result.current[0]).toBe(false);
});
```

**Why this follows best practices:** The test uses `renderHook` to test the Hook in isolation, wraps state updates in `act`, and reads `result.current` after each update.

### Real-World Cases

- **Design systems:** Custom Hooks follow strict best practices for consistency and reusability.
- **Open-source libraries:** Published Hooks (react-use, Mantine Hooks) adhere to these practices for reliability.
- **Large codebases:** Consistent Hook design reduces bugs and onboarding time.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- GitHub — React Hooks Composition Patterns — https://github.com/pproenca/dot-skills/blob/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/react-refactor/references/hook-composition-over-nesting.md
- GitHub — Testing Custom React Hooks — https://github.com/testing-library/react-testing-library/blob/main/docs/api.md
- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/

---

## References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- ESLint Plugin React Hooks — https://www.npmjs.com/package/eslint-plugin-react-hooks
- Testing Library — renderHook — https://testing-library.com/docs/react-testing-library/api/#renderhook
- Keploy — Testing Library React Hooks: A Complete Guide — https://keploy.io/blog/community/testing-library-react-hooks
- CoreUI — How to test hooks in React — https://coreui.io/blog/how-to-test-hooks-in-react/
- GitHub — React Hooks Composition Patterns — https://github.com/pproenca/dot-skills/blob/cf93c57cac89d6fc3e4194686000411567f5caf3/skills/.experimental/react-refactor/references/hook-composition-over-nesting.md
- GitHub — Mastering HTTP Requests with the useFetch Hook — https://github.com/small-lab-io/usehooks.io/blob/main/apps/www/posts/mastering-http-requests-with-the-usefetch-hook.mdx
- 30 Seconds of Code — Persisting state in React — https://github.com/Chalarangelo/30-seconds-of-code/blob/master/content/snippets/react/s/persist-state-to-storage.md
- Toss React Simplikit — useDebounce — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useDebounce/useDebounce.md
- Toss React Simplikit — useIntersectionObserver — https://github.com/toss/react-simplikit/blob/main/packages/react-simplikit/src/hooks/useIntersectionObserver/useIntersectionObserver.md
- CoreUI — How to use WebSockets in React — https://coreui.io/answers/how-to-use-websockets-in-react/
- use-broadcast-channel — https://www.npmjs.com/package/use-broadcast-channel
- JSR — useEventSource — https://jsr.io/@zap-studio/react-hooks/doc/use-event-source
- react-use — useKeyPress — https://github.com/streamich/react-use/blob/master/docs/useKeyPress.md
- react-use — usePermission — https://github.com/streamich/react-use/blob/master/docs/usePermission.md
- react-use — useInterval — https://github.com/streamich/react-use/blob/master/docs/useInterval.md
- Overreacted — Making setInterval Declarative with React Hooks — https://overreacted.io/making-setinterval-declarative-with-react-hooks/
- React Hook Form — useForm — https://react-hook-form.com/docs/useform
- Mantine — usePagination — https://mantine.dev/hooks/use-pagination/
- use-dark-mode — https://github.com/donavon/use-dark-mode