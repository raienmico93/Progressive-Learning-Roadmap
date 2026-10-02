# Custom Hook Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A custom Hook is a JavaScript function whose name starts with `use` and that calls other React Hooks, allowing you to extract and reuse stateful logic across multiple components without duplicating code or changing the component hierarchy.

**Technical Definition:** Custom Hooks are functions that begin with `use`, call one or more built-in Hooks (`useState`, `useEffect`, `useRef`, `useContext`, etc.), and return any value the caller needs. They are not a separate React API—they are a convention backed by the Rules of Hooks, which React's linter enforces. Custom Hooks let you share stateful logic (not state itself) between components: each call to a custom Hook creates an independent instance of its state, so two components using the same Hook do not share state unless that state is lifted or comes from an external store. Custom Hooks compose: a custom Hook can call other custom Hooks, and a component can call multiple custom Hooks. They are React's answer to the "mixin"/"higher-order component" problems, replacing inheritance and wrapper hell with plain functions that follow two simple rules: the name must start with `use`, and the Hook must be called at the top level of a component or another Hook.

**Beginner-Friendly Explanation:** A custom Hook is a reusable function for logic that needs state, effects, or refs. If you find yourself writing the same `useState` + `useEffect` pattern in three components—say, tracking the window size or fetching data—you extract it into a `useWindowSize` or `useFetch` function. Each component that calls the Hook gets its own copy of the state. You are not sharing the state; you are sharing the logic. This keeps components focused on what they render, while the Hook handles how the logic works.

### Key Characteristics

- **Name Must Start with `use`:** React's linter and runtime rely on this to identify Hooks and enforce the Rules of Hooks.
- **Plain Functions:** Custom Hooks are ordinary functions; they have no special syntax or API.
- **Isolated State per Call:** Each call to a custom Hook creates a fresh state instance; two components using the same Hook do not share state.
- **Composable:** Custom Hooks can call built-in Hooks and other custom Hooks, forming a tree of logic.
- **Not a Component:** Custom Hooks do not render; they return values the caller uses.
- **Lint-Enforced Rules:** The `eslint-plugin-react-hooks` plugin enforces the `use` prefix and the Rules of Hooks.
- **Dependency Arrays Matter:** Custom Hooks that use `useEffect`, `useMemo`, or `useCallback` must manage dependencies correctly or expose them to callers.
- **Testable in Isolation:** Custom Hooks can be tested with `@testing-library/react`'s `renderHook` without a full component tree.

### Prerequisites

- Solid understanding of React function components, JSX, and the primary Hooks (`useState`, `useEffect`, `useRef`, `useContext`).
- Working knowledge of JavaScript closures, functions, and the event loop.
- Familiarity with the Rules of Hooks and why they exist.
- Basic understanding of dependency arrays and referential equality.
- Awareness of `eslint-plugin-react-hooks` and its `exhaustive-deps` rule.

### Related Programming Areas

- **React Composition:** Reusing logic without inheritance or HOCs.
- **State Management:** Local state, shared state, and external stores.
- **Effects:** Synchronising with external systems.
- **Dependency Management:** Stable references and `exhaustive-deps`.
- **Testing:** `renderHook`, `act`, and hook-level unit tests.

### Core Concepts / Features

1. Hook Extraction
2. Reusable Stateful Logic
3. Hook Naming Conventions & Rules
4. Hook Composition
5. Dependency Management

---

## Core Concept 1: Hook Extraction

### Definitions

**Core Definition:** Hook extraction is the process of identifying duplicate stateful behaviour across components—or separating UI from logic within a single component—and moving that logic into a custom Hook.

**Technical Definition:** Hook extraction is a refactoring pattern in which a cohesive unit of stateful logic (state declarations, effects, refs, derived values, and the functions that operate on them) is moved from a component into a function named `useX`. The component then calls the Hook and uses its return value. Extraction is justified when: (1) the same logic appears in two or more components; (2) a component's logic is complex enough to obscure its rendering; (3) the logic can be tested independently; or (4) the logic would benefit from a clear name that documents its purpose. React's documentation recommends starting with a Hook when you write a `useEffect` whose purpose is not obvious from a single glance, or when the same `useEffect` appears in multiple components. Extraction does not change the component's behaviour; it reorganises the code so that state, effects, and handlers travel together.

**Beginner-Friendly Explanation:** If you notice the same pattern in two components—like tracking a window size, syncing to `localStorage`, or debouncing an input—that is a signal to extract a custom Hook. You move the state and effects into a function that starts with `use`, give it a descriptive name, and call it from both components. The components get shorter and clearer; the logic lives in one place and can be tested on its own.

### Purposes

- To remove duplicate stateful logic across components.
- To separate rendering concerns from stateful behaviour.
- To give a name to a piece of logic that would otherwise be an anonymous `useEffect`.
- To make logic independently testable.
- To improve readability by reducing the size of component bodies.
- To encapsulate related state, effects, and handlers in one unit.

### Syntax Rules and Structure

**Before Extraction (Duplicated Logic):**
```jsx
function WindowSizeA() {
  const [width, setWidth] = React.useState(window.innerWidth);

  React.useEffect(() => {
    function handleResize() { setWidth(window.innerWidth); }
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return <p>Width A: {width}</p>;
}

function WindowSizeB() {
  const [width, setWidth] = React.useState(window.innerWidth);

  React.useEffect(() => {
    function handleResize() { setWidth(window.innerWidth); }
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return <p>Width B: {width}</p>;
}
```

**After Extraction:**
```jsx
function useWindowWidth() {
  const [width, setWidth] = React.useState(window.innerWidth);

  React.useEffect(() => {
    function handleResize() { setWidth(window.innerWidth); }
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}

function WindowSizeA() {
  const width = useWindowWidth();
  return <p>Width A: {width}</p>;
}

function WindowSizeB() {
  const width = useWindowWidth();
  return <p>Width B: {width}</p>;
}
```

**Component Breakdown:**
- `useWindowWidth`: The custom Hook; contains the state and effect.
- Both components call `useWindowWidth()` and receive their own `width`.
- The duplication is eliminated; the logic lives in one place.

**Separating Logic from UI (Complex Component):**
```jsx
// Before: state, effects, and rendering mixed together
function ProductSearch() {
  const [query, setQuery] = React.useState('');
  const [results, setResults] = React.useState([]);
  const [isLoading, setIsLoading] = React.useState(false);

  React.useEffect(() => {
    if (!query) { setResults([]); return; }
    let ignore = false;
    setIsLoading(true);
    fetch(`/api/search?q=${query}`)
      .then((r) => r.json())
      .then((data) => { if (!ignore) { setResults(data); setIsLoading(false); } });
    return () => { ignore = true; };
  }, [query]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {isLoading && <p>Searching...</p>}
      <ul>{results.map((r) => <li key={r.id}>{r.title}</li>)}</ul>
    </div>
  );
}
```
```jsx
// After: logic in a Hook, UI in the component
function useSearch(query) {
  const [results, setResults] = React.useState([]);
  const [isLoading, setIsLoading] = React.useState(false);

  React.useEffect(() => {
    if (!query) { setResults([]); return; }
    let ignore = false;
    setIsLoading(true);
    fetch(`/api/search?q=${query}`)
      .then((r) => r.json())
      .then((data) => { if (!ignore) { setResults(data); setIsLoading(false); } });
    return () => { ignore = true; };
  }, [query]);

  return { results, isLoading };
}

function ProductSearch() {
  const [query, setQuery] = React.useState('');
  const { results, isLoading } = useSearch(query);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {isLoading && <p>Searching...</p>}
      <ul>{results.map((r) => <li key={r.id}>{r.title}</li>)}</ul>
    </div>
  );
}
```

**Component Breakdown:**
- `useSearch(query)`: Encapsulates the search effect and its loading/results state.
- The component keeps only the query state (which drives the UI) and the rendering.
- The logic is now testable in isolation.

**Syntax Rules:**
- Extract when the same stateful logic appears in two or more components.
- Extract when a component's logic obscures its rendering.
- Name the Hook `use` + a descriptive noun or verb phrase (`useWindowWidth`, `useSearch`, `useLocalStorage`).
- Move only stateful logic; pure helpers (formatters, constants) do not need a Hook.
- Keep the Hook focused on one concern; do not create a "god hook" that does everything.
- Return only what the caller needs; do not expose internal state unnecessarily.
- Preserve the original behaviour exactly; extraction should be behaviour-preserving.

**Constraints and Limitations:**
- Extraction adds indirection; do not extract logic used only once (the "three uses" heuristic applies).
- Custom Hooks share logic, not state; do not expect two components to share the same state instance.
- Extracting UI into a Hook is not possible; Hooks do not render.
- Over-extraction fragments logic into too many files; keep related concerns together.
- A Hook's dependencies become part of its contract; changing them is a breaking change.

### Annotated Code Examples

**Example 1: Extracting a Debounced Value Hook**

```jsx
function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = React.useState(value);

  React.useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}

function SearchBox() {
  const [query, setQuery] = React.useState('');
  const debouncedQuery = useDebouncedValue(query, 500);

  React.useEffect(() => {
    if (debouncedQuery) console.log('Searching for', debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Expected Output:** The component logs "Searching for ..." only after the user pauses typing for 500ms. The `useDebouncedValue` Hook encapsulates the debounce logic and can be reused wherever a debounced value is needed.

**Why This Output Occurs:** The Hook manages a `debounced` state that updates after `delay` ms of inactivity. The cleanup clears the previous timer, so only the last value in a burst of changes is applied.

**Example 2: Extracting a `useLocalStorage` Hook**

```jsx
function useLocalStorage(key, initialValue) {
  const [stored, setStored] = React.useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = React.useCallback(
    (value) => {
      try {
        setStored(value);
        window.localStorage.setItem(key, JSON.stringify(value));
      } catch (error) {
        console.warn(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key]
  );

  return [stored, setValue];
}

function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Theme: {theme}
    </button>
  );
}
```

**Expected Output:** The theme toggle persists the user's choice in `localStorage`. Reloading the page restores the saved theme. The Hook is reusable for any persisted value.

**Why This Output Occurs:** The lazy initialiser reads `localStorage` on mount. `setValue` writes to both state and `localStorage`. The `[key]` dependency on `setValue` ensures the correct key is used if the key changes.

### Real-World Cases

- **Window/scroll tracking:** `useWindowWidth`, `useScrollPosition`.
- **Network:** `useFetch`, `useOnlineStatus`, `useSearch`.
- **Storage:** `useLocalStorage`, `useSessionStorage`.
- **Timers:** `useInterval`, `useTimeout`, `useDebouncedValue`.
- **Media:** `useMediaQuery`, `usePrefersReducedMotion`.
- **Forms:** `useFormDraft`, `useFieldArray` (from RHF).

### References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Extracting a Custom Hook - https://react.dev/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component
- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- eslint-plugin-react-hooks - https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks

---

## Core Concept 2: Reusable Stateful Logic

### Definitions

**Core Definition:** Reusable stateful logic is the property of custom Hooks that lets multiple components reuse the same logic while each maintaining its own independent state instance.

**Technical Definition:** A custom Hook is a function that calls other Hooks. Each time it is called, React associates the Hooks inside it with the calling component's fiber. This means state declared inside a custom Hook is scoped to the component instance that called it—not shared across components. Two components calling `useWindowWidth` each get their own `width` state; changing one does not affect the other. The logic (the effect, the resize listener, the state updater) is shared; the state itself is not. To share state between components, you must lift it to a common ancestor, use Context, or use an external store (Zustand, Redux). This distinction is crucial: "Custom Hooks share logic, not state." React's official documentation states: "You can't use a custom Hook to share state between two components. Each call to a Hook gets an isolated state."

**Beginner-Friendly Explanation:** Think of a custom Hook as a recipe. Two chefs can use the same recipe (the logic), but each chef has their own ingredients (the state). If one chef adds salt, the other chef's dish is unaffected. If you want two components to share the same state, a custom Hook is not the tool—you need to lift the state up, use Context, or use a store. Understanding this distinction prevents one of the most common misunderstandings about custom Hooks.

### Purposes

- To reuse stateful logic across components without sharing state.
- To isolate each component's state instance so updates do not leak.
- To encapsulate effects, refs, and derived values in a reusable unit.
- To make components independent while sharing behaviour.
- To avoid prop drilling by keeping state local to each consumer.
- To enable testing of logic in isolation from UI.

### Syntax Rules and Structure

**Independent State per Component:**
```jsx
function useCounter(initial = 0) {
  const [count, setCount] = React.useState(initial);
  const increment = () => setCount((c) => c + 1);
  return { count, increment };
}

function CounterA() {
  const { count, increment } = useCounter(0);
  return <button onClick={increment}>A: {count}</button>;
}

function CounterB() {
  const { count, increment } = useCounter(100);
  return <button onClick={increment}>B: {count}</button>;
}
```

**Component Breakdown:**
- `useCounter(0)` in `CounterA` creates one state instance starting at `0`.
- `useCounter(100)` in `CounterB` creates a separate state instance starting at `100`.
- Incrementing `CounterA` does not affect `CounterB`.

**Sharing State via Lifting (Not via a Hook):**
```jsx
function useCounter(initial = 0) {
  const [count, setCount] = React.useState(initial);
  return { count, setCount };
}

function Parent() {
  const { count, setCount } = useCounter(0);

  return (
    <div>
      <CounterA count={count} onIncrement={() => setCount((c) => c + 1)} />
      <CounterB count={count} onIncrement={() => setCount((c) => c + 1)} />
    </div>
  );
}

function CounterA({ count, onIncrement }) {
  return <button onClick={onIncrement}>A: {count}</button>;
}

function CounterB({ count, onIncrement }) {
  return <button onClick={onIncrement}>B: {count}</button>;
}
```

**Component Breakdown:**
- The state is lifted to `Parent`, which calls `useCounter` once.
- Both `CounterA` and `CounterB` receive the same `count` and `onIncrement`.
- This is how you share state; the Hook is used once, at the owner.

**Isolated Effects:**
```jsx
function useInterval(callback, delay) {
  const savedCallback = React.useRef(callback);

  React.useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  React.useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

function ClockA() {
  const [time, setTime] = React.useState(new Date());
  useInterval(() => setTime(new Date()), 1000);
  return <p>A: {time.toLocaleTimeString()}</p>;
}

function ClockB() {
  const [time, setTime] = React.useState(new Date());
  useInterval(() => setTime(new Date()), 1000);
  return <p>B: {time.toLocaleTimeString()}</p>;
}
```

**Component Breakdown:**
- Each `Clock` calls `useInterval` with its own `setTime`.
- Each interval is independent; the cleanup of one does not affect the other.

**Syntax Rules:**
- Use custom Hooks to share logic, not state.
- Each call to a Hook creates a new state instance scoped to the calling component.
- To share state, lift it to a common ancestor, use Context, or use an external store.
- Effects inside a Hook are also scoped per component; each instance manages its own subscriptions.
- Refs inside a Hook are per component; two instances do not share a ref.
- Return only what the caller needs; the caller owns the returned state.
- Never expect a custom Hook to memoise or deduplicate across instances.

**Constraints and Limitations:**
- State is not shared; if two components need the same state, lift it.
- Effects run per instance; N calls to the Hook mean N subscriptions.
- Custom Hooks cannot replace state management libraries; they are a logic-reuse tool.
- A Hook's state resets when the component unmounts; persistence requires `localStorage` or an external store.
- Sharing logic across unrelated components can lead to duplicated effects and wasted work; consider a shared store.

### Annotated Code Examples

**Example 1: Independent Counter Instances**

```jsx
function useCounter(initial = 0, step = 1) {
  const [count, setCount] = React.useState(initial);
  const increment = React.useCallback(() => setCount((c) => c + step), [step]);
  const decrement = React.useCallback(() => setCount((c) => c - step), [step]);
  const reset = React.useCallback(() => setCount(initial), [initial]);
  return { count, increment, decrement, reset };
}

function ShoppingCart() {
  const { count: items, increment: addItem, decrement: removeItem } = useCounter(0);
  return (
    <div>
      <p>Items in cart: {items}</p>
      <button onClick={addItem}>Add</button>
      <button onClick={removeItem}>Remove</button>
    </div>
  );
}

function Notifications() {
  const { count: unread, increment: markUnread, reset: markAllRead } = useCounter(0);
  return (
    <div>
      <p>Unread: {unread}</p>
      <button onClick={markUnread}>New notification</button>
      <button onClick={markAllRead}>Mark all read</button>
    </div>
  );
}
```

**Expected Output:** The cart and notification counters are independent. Adding items to the cart does not affect unread notifications, and vice versa.

**Why This Output Occurs:** Each component calls `useCounter`, creating its own `count` state. The `useCallback` wrappers preserve the handlers' identities, but the state is per instance.

**Example 2: Per-Component Fetch Instances**

```jsx
function useFetch(url) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState(null);

  React.useEffect(() => {
    const controller = new AbortController();
    setLoading(true);
    fetch(url, { signal: controller.signal })
      .then((res) => res.json())
      .then(setData)
      .catch((err) => { if (err.name !== 'AbortError') setError(err); })
      .finally(() => setLoading(false));
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

function UserProfile({ userId }) {
  const { data: user, loading } = useFetch(`/api/users/${userId}`);
  if (loading) return <p>Loading user...</p>;
  return <h1>{user.name}</h1>;
}

function UserPosts({ userId }) {
  const { data: posts, loading } = useFetch(`/api/users/${userId}/posts`);
  if (loading) return <p>Loading posts...</p>;
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

**Expected Output:** The profile and posts components each fetch their own data. A slow posts request does not block the profile, and each request is cancelled when the component unmounts or the URL changes.

**Why This Output Occurs:** Each call to `useFetch` creates its own `data`, `loading`, and `error` state, plus its own `AbortController`. The two components are independent.

### Real-World Cases

- **Multiple forms on a page:** Each form uses the same `useFormDraft` Hook but has its own draft.
- **Multiple charts:** Each chart uses the same `useChartData` Hook but fetches its own data.
- **Multiple modals:** Each modal uses the same `useFocusTrap` Hook but has its own focus state.
- **Timers:** Each component using `useInterval` gets its own interval.
- **Subscriptions:** Each component using `useEventListener` gets its own listener.

### References

- React Official Documentation – Custom Hooks Share Logic, Not State - https://react.dev/learn/reusing-logic-with-custom-hooks#custom-hooks-share-logic-not-state
- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Sharing State Between Components - https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Passing Data Deeply with Context - https://react.dev/learn/passing-data-deeply-with-context

---

## Core Concept 3: Hook Naming Conventions & Rules

### Definitions

**Core Definition:** Hook naming conventions require that every custom Hook and every function that calls a Hook begins with `use`; the Rules of Hooks require that Hooks be called only at the top level of a component or another Hook, never inside conditions, loops, or nested functions.

**Technical Definition:** React identifies Hooks by their name: any function whose name starts with `use` and that calls other Hooks is treated as a custom Hook. The `use` prefix is not a stylistic choice—it is a signal to React's linter and runtime that the function follows the Rules of Hooks. The Rules of Hooks are: (1) **Only call Hooks at the top level**—not inside conditions, loops, nested functions, or `try/catch` blocks; (2) **Only call Hooks from React function components or custom Hooks**—not from regular JavaScript functions, class components, or event handlers. These rules exist because React relies on the *order* of Hook calls to associate state with the correct Hook: React maintains a linked list of Hooks per component, and every render must call the same Hooks in the same order. Violating the rules causes "Rendered fewer Hooks than expected" or "Rendered more Hooks than during the previous render" errors. The `eslint-plugin-react-hooks` plugin enforces both rules and the `exhaustive-deps` rule for dependency arrays.

**Beginner-Friendly Explanation:** React tracks Hooks by the order they are called. If you call `useState` inside an `if` statement, React cannot know which state belongs to which render—it loses track. That is why Hooks must always be at the top level, in the same order, every render. The `use` prefix tells React (and your linter) that a function is a Hook, so it can enforce these rules. If you name a function `fetchData` instead of `useFetchData`, React will not treat it as a Hook, and if it calls `useState` inside, you will get a runtime error.

### Purposes

- To signal to React and the linter that a function is a Hook.
- To enforce the Rules of Hooks consistently across a codebase.
- To prevent "Rendered fewer/more Hooks than expected" runtime errors.
- To ensure Hook call order is stable across renders.
- To make dependencies explicit and lintable.
- To make code readable and predictable for other developers.

### Syntax Rules and Structure

**Correct Hook Naming:**
```jsx
// ✅ Custom Hook: name starts with `use`
function useLocalStorage(key, initialValue) {
  const [stored, setStored] = React.useState(initialValue);
  // ...
  return [stored, setStored];
}

// ✅ Component: name starts with a capital letter
function UserProfile() {
  const [user, setUser] = useLocalStorage('user', null);
  return <h1>{user?.name}</h1>;
}
```

**Incorrect Hook Naming:**
```jsx
// ❌ Does not start with `use`; React will not treat it as a Hook
function getLocalStorage(key) {
  const [stored, setStored] = React.useState(null); // Runtime error
  return stored;
}

// ❌ Lowercase component name; React will not treat it as a component
function userProfile() {
  return <h1>Profile</h1>;
}
```

**Rules of Hooks — Top Level Only:**
```jsx
// ❌ Conditional Hook call
function Component({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = React.useState(null); // Error
  }
  // ...
}

// ✅ Hooks at the top level
function Component({ isLoggedIn }) {
  const [user, setUser] = React.useState(null);
  if (!isLoggedIn) return <LoginPrompt />;
  // ...
}
```

**Rules of Hooks — Components or Custom Hooks Only:**
```jsx
// ❌ Calling a Hook from a regular function
function fetchUser(id) {
  const [user, setUser] = React.useState(null); // Error
}

// ✅ Calling a Hook from a custom Hook
function useUser(id) {
  const [user, setUser] = React.useState(null);
  // ...
  return user;
}
```

**Correct Conditional Logic Inside Hooks:**
```jsx
function useUser(id) {
  const [user, setUser] = React.useState(null);

  React.useEffect(() => {
    if (!id) return; // Conditional logic inside the effect is fine
    fetch(`/api/users/${id}`).then((r) => r.json()).then(setUser);
  }, [id]);

  return user;
}
```

**Component Breakdown:**
- The Hook is called unconditionally at the top of `useUser`.
- The condition is inside the effect, not around the Hook call.
- The Hook order is stable across renders.

**Lint Configuration:**
```json
// .eslintrc.json
{
  "extends": ["plugin:react-hooks/recommended"],
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

**Component Breakdown:**
- `rules-of-hooks`: Errors on conditional or nested Hook calls.
- `exhaustive-deps`: Warns on missing or incorrect dependencies.
- Both rules are part of `plugin:react-hooks/recommended`.

**Syntax Rules:**
- Name every custom Hook `use` + a capitalised word (`useFetch`, `useLocalStorage`).
- Call Hooks only at the top level of a component or custom Hook.
- Never call Hooks inside `if`, `for`, `while`, `switch`, or nested functions.
- Never call Hooks inside `try/catch`; if you must, put the `try/catch` inside the Hook.
- Never call Hooks from event handlers, class components, or regular functions.
- Use `eslint-plugin-react-hooks` to enforce both rules.
- Keep the `use` prefix even for Hooks that only return values (not state).
- Name the returned state and setters descriptively; consumers destructure by name.

**Constraints and Limitations:**
- The `use` prefix is a convention enforced by the linter; React cannot detect a Hook that does not follow it.
- The Rules of Hooks are not optional; violating them causes runtime errors.
- Hooks cannot be called conditionally, even if the condition is stable across renders.
- Hooks cannot be called in class components.
- The linter can be disabled, but doing so is strongly discouraged.
- Hook order is per component instance; two instances of the same component have independent Hook lists.

### Annotated Code Examples

**Example 1: Correcting a Conditional Hook**

```jsx
// ❌ Broken: Hook inside an if statement
function UserProfile({ userId }) {
  if (userId) {
    const [user, setUser] = React.useState(null);
    React.useEffect(() => {
      fetch(`/api/users/${userId}`).then((r) => r.json()).then(setUser);
    }, [userId]);
    return <h1>{user?.name}</h1>;
  }
  return <p>No user selected</p>;
}

// ✅ Fixed: Hook at the top level, condition inside the effect
function UserProfile({ userId }) {
  const [user, setUser] = React.useState(null);

  React.useEffect(() => {
    if (!userId) { setUser(null); return; }
    fetch(`/api/users/${userId}`).then((r) => r.json()).then(setUser);
  }, [userId]);

  if (!userId) return <p>No user selected</p>;
  return <h1>{user?.name}</h1>;
}
```

**Expected Output:** The fixed version works correctly for both `userId` values. The broken version throws "Rendered fewer Hooks than expected" when `userId` changes from truthy to falsy.

**Why This Output Occurs:** React tracks Hooks by order. In the broken version, the number of Hooks changes between renders (one render calls `useState` and `useEffect`, the next calls none), so React cannot associate state with the correct Hook. The fixed version always calls the same Hooks in the same order.

**Example 2: Correctly Naming a Custom Hook**

```jsx
// ✅ Correct: `use` prefix signals a Hook
function useDocumentTitle(title) {
  React.useEffect(() => {
    const previous = document.title;
    document.title = title;
    return () => { document.title = previous; };
  }, [title]);
}

function Page({ title }) {
  useDocumentTitle(title);
  return <h1>{title}</h1>;
}

// ❌ Incorrect: no `use` prefix; if this function called Hooks, it would error
function setDocumentTitle(title) {
  React.useEffect(() => { // Runtime error if called from a component
    document.title = title;
  }, [title]);
}
```

**Expected Output:** `useDocumentTitle` works because it follows the naming convention. `setDocumentTitle` would throw if it called `useEffect`, because React does not treat it as a Hook.

**Why This Output Occurs:** The `use` prefix tells React and the linter that the function is a Hook. Without it, React does not associate the Hook calls with the component's Hook list.

### Real-World Cases

- **All custom Hooks:** The `use` prefix is non-negotiable.
- **Lint configuration:** Every React project should enable `react-hooks/rules-of-hooks`.
- **Code review:** Reviewers should flag functions that call Hooks without the `use` prefix.
- **Library authoring:** Custom Hook packages must follow the convention.
- **Refactoring:** Extracting a Hook from a component requires renaming it with the `use` prefix.

### References

- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – eslint-plugin-react-hooks - https://react.dev/reference/eslint-plugin-react-hooks
- React Official Documentation – `exhaustive-deps` - https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- eslint-plugin-react-hooks - https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks
- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks

---

## Core Concept 4: Hook Composition

### Definitions

**Core Definition:** Hook composition is the practice of calling built-in Hooks and other custom Hooks inside a custom Hook, and calling multiple custom Hooks inside a component, to build complex logic from smaller, focused units.

**Technical Definition:** Custom Hooks compose in the same way components do: a custom Hook can call `useState`, `useEffect`, `useRef`, `useContext`, `useMemo`, `useCallback`, and other custom Hooks. A component can call multiple custom Hooks, and a custom Hook can call multiple custom Hooks, forming a tree of logic. Composition is React's answer to the "mixin" and "higher-order component" patterns: instead of wrapping a component in layers of HOCs, you compose Hooks inside the component. Each Hook returns values that the caller combines. Composition preserves the Rules of Hooks because all Hook calls happen at the top level of the component or Hook, in a stable order. A common pattern is to build a "facade" Hook that composes smaller Hooks into a single, cohesive API—for example, `useAuth` composes `useUser`, `useToken`, and `usePermissions` into one Hook that the component calls.

**Beginner-Friendly Explanation:** Composition means building bigger Hooks from smaller ones. Just as you compose components (a `Page` contains a `Header`, a `Sidebar`, and a `Content`), you compose Hooks (a `useAuth` Hook contains `useUser`, `useToken`, and `usePermissions`). The component calls one Hook and gets a clean API. Composition is how you avoid "god hooks" that try to do everything—instead, you build small Hooks that each do one thing, then combine them.

### Purposes

- To build complex logic from small, focused Hooks.
- To avoid duplicating logic across Hooks.
- To provide a single, cohesive API to components.
- To isolate concerns (auth, data, UI) in separate Hooks.
- To make Hooks easier to test and maintain.
- To replace HOC and mixin patterns with composable functions.

### Syntax Rules and Structure

**Composing Built-in Hooks:**
```jsx
function useToggle(initial = false) {
  const [value, setValue] = React.useState(initial);
  const toggle = React.useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

function useBoolean(initial = false) {
  const [value, setValue] = React.useState(initial);
  const setTrue = React.useCallback(() => setValue(true), []);
  const setFalse = React.useCallback(() => setValue(false), []);
  const toggle = React.useCallback(() => setValue((v) => !v), []);
  return { value, setTrue, setFalse, toggle };
}
```

**Composing Custom Hooks:**
```jsx
function useUser() {
  const [user, setUser] = React.useState(null);
  React.useEffect(() => {
    fetch('/api/me').then((r) => r.json()).then(setUser);
  }, []);
  return user;
}

function useToken() {
  const [token, setToken] = React.useState(() => localStorage.getItem('token'));
  React.useEffect(() => {
    if (token) localStorage.setItem('token', token);
    else localStorage.removeItem('token');
  }, [token]);
  return [token, setToken];
}

function usePermissions(user) {
  return React.useMemo(() => {
    if (!user) return [];
    return user.roles.flatMap((role) => role.permissions);
  }, [user]);
}

// Facade Hook: composes the above into one API
function useAuth() {
  const user = useUser();
  const [token, setToken] = useToken();
  const permissions = usePermissions(user);

  const login = React.useCallback(async (credentials) => {
    const res = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
    });
    const { user, token } = await res.json();
    setToken(token);
    return user;
  }, [setToken]);

  const logout = React.useCallback(() => {
    setToken(null);
  }, [setToken]);

  return { user, token, permissions, login, logout };
}
```

**Component Breakdown:**
- `useUser`, `useToken`, `usePermissions`: Small, focused Hooks.
- `useAuth`: A facade Hook that composes them into one API.
- Components call `useAuth()` and get `{ user, token, permissions, login, logout }`.

**Composing Multiple Hooks in a Component:**
```jsx
function Dashboard() {
  const { user, permissions } = useAuth();
  const windowWidth = useWindowWidth();
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const online = useOnlineStatus();

  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      <p>Window: {windowWidth}px</p>
      <p>Theme: {theme}</p>
      <p>Status: {online ? 'Online' : 'Offline'}</p>
    </div>
  );
}
```

**Component Breakdown:**
- The component calls four Hooks, each independent.
- Each Hook manages its own state and effects.
- The component composes the returned values into its UI.

**Composing with Parameters and Conditional Logic:**
```jsx
function useSearch(query, options = {}) {
  const debouncedQuery = useDebouncedValue(query, options.delay ?? 300);
  const [results, setResults] = React.useState([]);
  const [loading, setLoading] = React.useState(false);

  React.useEffect(() => {
    if (!debouncedQuery) { setResults([]); return; }
    let ignore = false;
    setLoading(true);
    fetch(`/api/search?q=${debouncedQuery}`)
      .then((r) => r.json())
      .then((data) => { if (!ignore) { setResults(data); setLoading(false); } });
    return () => { ignore = true; };
  }, [debouncedQuery]);

  return { results, loading };
}
```

**Component Breakdown:**
- `useSearch` composes `useDebouncedValue` with `useState` and `useEffect`.
- The debouncing logic is encapsulated; the caller only sees `results` and `loading`.
- Options are passed through to the composed Hook.

**Syntax Rules:**
- Call custom Hooks at the top level of a custom Hook or component.
- Compose small, focused Hooks into larger facade Hooks.
- Pass parameters through to composed Hooks.
- Return only the values the caller needs; hide internal Hooks.
- Keep each Hook focused on one concern; compose rather than accumulate.
- Use `useMemo` and `useCallback` to stabilise values passed between Hooks.
- Document the composed Hook's API; the internal composition is an implementation detail.

**Constraints and Limitations:**
- Composition can obscure which Hook owns which state; document the facade.
- Deeply composed Hooks are harder to debug; use React DevTools to inspect Hook state.
- A facade Hook inherits the constraints of its composed Hooks (e.g., dependency requirements).
- Over-composition fragments logic into too many layers; keep the tree shallow.
- Composed Hooks re-render the component when any of their state changes; isolate volatile state.

### Annotated Code Examples

**Example 1: Facade Hook for Form Logic**

```jsx
function useField(initialValue = '') {
  const [value, setValue] = React.useState(initialValue);
  const [touched, setTouched] = React.useState(false);
  const onChange = React.useCallback((e) => setValue(e.target.value), []);
  const onBlur = React.useCallback(() => setTouched(true), []);
  const reset = React.useCallback(() => { setValue(initialValue); setTouched(false); }, [initialValue]);
  return { value, touched, onChange, onBlur, reset };
}

function useForm(initialValues) {
  const fields = React.useMemo(
    () => Object.fromEntries(
      Object.entries(initialValues).map(([key, value]) => [key, value])
    ),
    [initialValues]
  );

  const [values, setValues] = React.useState(fields);
  const [touched, setTouched] = React.useState({});

  const handleChange = React.useCallback((name) => (e) => {
    setValues((v) => ({ ...v, [name]: e.target.value }));
  }, []);

  const handleBlur = React.useCallback((name) => () => {
    setTouched((t) => ({ ...t, [name]: true }));
  }, []);

  const reset = React.useCallback(() => {
    setValues(fields);
    setTouched({});
  }, [fields]);

  return { values, touched, handleChange, handleBlur, reset };
}

function LoginForm() {
  const { values, handleChange, handleBlur, reset } = useForm({ email: '', password: '' });

  return (
    <form onSubmit={(e) => { e.preventDefault(); console.log(values); }}>
      <input
        value={values.email}
        onChange={handleChange('email')}
        onBlur={handleBlur('email')}
      />
      <input
        type="password"
        value={values.password}
        onChange={handleChange('password')}
        onBlur={handleBlur('password')}
      />
      <button type="submit">Log In</button>
      <button type="button" onClick={reset}>Reset</button>
    </form>
  );
}
```

**Expected Output:** A login form with email and password fields, touched tracking, and a reset button. The `useForm` Hook composes `useState`, `useMemo`, and `useCallback` to provide a clean API. The `useField` Hook is also available for individual fields.

**Why This Output Occurs:** `useForm` composes built-in Hooks to manage values, touched state, change handlers, and reset logic. The component receives a single API and focuses on rendering.

**Example 2: Composing Data-Fetching and Polling**

```jsx
function useFetch(url, options = {}) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState(null);

  const refetch = React.useCallback(() => {
    setLoading(true);
    return fetch(url, options)
      .then((r) => { if (!r.ok) throw new Error(`HTTP ${r.status}`); return r.json(); })
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url, JSON.stringify(options)]);

  React.useEffect(() => { refetch(); }, [refetch]);

  return { data, loading, error, refetch };
}

function usePolling(url, interval) {
  const { data, loading, error, refetch } = useFetch(url);

  React.useEffect(() => {
    if (!interval) return;
    const id = setInterval(refetch, interval);
    return () => clearInterval(id);
  }, [interval, refetch]);

  return { data, loading, error, refetch };
}

function LiveMetrics() {
  const { data, loading } = usePolling('/api/metrics', 5000);
  if (loading && !data) return <p>Loading metrics...</p>;
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

**Expected Output:** The metrics component fetches data on mount and refetches every 5 seconds. The `usePolling` Hook composes `useFetch` with a polling effect, and the component only calls `usePolling`.

**Why This Output Occurs:** `usePolling` composes `useFetch` (which itself composes `useState`, `useEffect`, and `useCallback`). The polling effect calls `refetch` on an interval. The component is unaware of the internal composition.

### Real-World Cases

- **Auth:** `useAuth` composing `useUser`, `useToken`, `usePermissions`.
- **Forms:** `useForm` composing `useField`, `useValidation`, `useSubmission`.
- **Data fetching:** `usePolling` composing `useFetch` and `useInterval`.
- **UI state:** `useDisclosure` composing `useBoolean` and `useCallback`.
- **Media:** `useMediaQuery` composing `useEffect` and `useState`.
- **Storage:** `useSyncedState` composing `useLocalStorage` and `useEffect`.

### References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Building Your Own Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks#building-your-own-hooks
- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – `useMemo` - https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback` - https://react.dev/reference/react/useCallback

---

## Core Concept 5: Dependency Management

### Definitions

**Core Definition:** Dependency management in custom Hooks is the practice of correctly declaring the reactive values that a Hook's effects and callbacks depend on, and preserving stable references so that effects do not re-run or callbacks do not change identity unnecessarily.

**Technical Definition:** Custom Hooks that use `useEffect`, `useMemo`, or `useCallback` inherit the dependency-array contract. The Hook must include every reactive value it reads inside the effect or callback in the dependency array, or the effect will use a stale closure. The Hook must also avoid unstable dependencies: if a dependency is a new object, array, or function on every render, the effect or memo will never be reused. Two patterns help: (1) **Stable callbacks** via `useCallback` (with correct dependencies) preserve the identity of functions passed to effects or returned to callers; (2) **Latest-ref pattern** (`useRef` to hold the latest callback) stabilises the effect's dependency while keeping the callback fresh—this is the manual equivalent of the proposed `useEffectEvent` (formerly `useEvent`). React 19's `useEffectEvent` is the future standard for extracting non-reactive logic from effects. Custom Hooks should document their dependency requirements and expose the minimal reactive surface to callers, often returning stable functions (`useCallback`) so consumers can safely use them in their own dependency arrays.

**Beginner-Friendly Explanation:** Dependency arrays are how React knows when to re-run an effect or recompute a value. If you forget a dependency, the effect uses an old value—a bug called a stale closure. If you include an unstable dependency (a new function or object every render), the effect re-runs on every render—a performance problem. Custom Hooks must get this right internally and expose stable functions to their consumers, so the consumer's effects do not re-run unnecessarily. The "latest-ref" pattern (and, in React 19, `useEffectEvent`) is the modern solution for callbacks that need the latest values without re-running the effect.

### Purposes

- To ensure effects inside custom Hooks re-run exactly when their inputs change.
- To prevent stale closures where the Hook reads outdated values.
- To preserve the identity of functions returned by the Hook.
- To allow consumers to use the Hook's return values in their own dependency arrays.
- To avoid infinite loops caused by unstable dependencies.
- To adopt the latest-ref pattern or `useEffectEvent` for callbacks that need fresh values.

### Syntax Rules and Structure

**Correct Dependencies in a Custom Hook:**
```jsx
function useInterval(callback, delay) {
  React.useEffect(() => {
    if (delay === null) return;
    const id = setInterval(callback, delay);
    return () => clearInterval(id);
  }, [callback, delay]); // Both dependencies listed
}

// Consumer must pass a stable callback, or the interval resets on every render
function Clock() {
  const [time, setTime] = React.useState(new Date());
  const tick = React.useCallback(() => setTime(new Date()), []);
  useInterval(tick, 1000);
  return <p>{time.toLocaleTimeString()}</p>;
}
```

**Component Breakdown:**
- The Hook lists `callback` and `delay` as dependencies.
- The consumer must stabilise `callback` with `useCallback` to avoid resetting the interval.

**Latest-Ref Pattern (Stabilising the Dependency):**
```jsx
function useInterval(callback, delay) {
  const savedCallback = React.useRef(callback);

  React.useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  React.useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]); // Only `delay` is a dependency; callback is read from the ref
}

// Consumer does NOT need useCallback; the latest callback is always used
function Clock() {
  const [time, setTime] = React.useState(new Date());
  useInterval(() => setTime(new Date()), 1000);
  return <p>{time.toLocaleTimeString()}</p>;
}
```

**Component Breakdown:**
- `savedCallback` ref holds the latest callback.
- The first effect updates the ref on every render (safe because it has no dependencies that trigger work).
- The second effect depends only on `delay`, so the interval is not reset when the callback changes.
- The consumer can pass an inline arrow function without `useCallback`.

**React 19 `useEffectEvent` (The Modern Solution):**
```jsx
import { useEffectEvent } from 'react';

function useInterval(callback, delay) {
  const onTick = useEffectEvent(callback);

  React.useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => onTick(), delay);
    return () => clearInterval(id);
  }, [delay]); // `onTick` is not a dependency
}
```

**Component Breakdown:**
- `useEffectEvent(callback)` extracts the non-reactive logic.
- The effect does not depend on `onTick`; the effect event always sees the latest `callback`.
- This is the future standard, replacing the manual latest-ref pattern.

**Stable Return Values:**
```jsx
function useDisclosure(initial = false) {
  const [isOpen, setIsOpen] = React.useState(initial);

  const open = React.useCallback(() => setIsOpen(true), []);
  const close = React.useCallback(() => setIsOpen(false), []);
  const toggle = React.useCallback(() => setIsOpen((v) => !v), []);

  return { isOpen, open, close, toggle };
}
```

**Component Breakdown:**
- `open`, `close`, and `toggle` are stable (empty dependency arrays).
- Consumers can safely use them in their own `useEffect` dependencies.
- `isOpen` changes; consumers re-render when it does.

**Consumer Using the Hook's Return in a Dependency Array:**
```jsx
function useDisclosureWithLogging(label) {
  const disclosure = useDisclosure();

  React.useEffect(() => {
    console.log(`${label} is ${disclosure.isOpen ? 'open' : 'closed'}`);
  }, [label, disclosure.isOpen]); // Only isOpen is a dependency; functions are stable

  return disclosure;
}
```

**Component Breakdown:**
- The consumer includes `disclosure.isOpen` (a value) but not `open`/`close`/`toggle` (stable functions).
- If the functions were not stable, the effect would re-run on every render.

**Syntax Rules:**
- Include every reactive value read inside an effect in the dependency array.
- Use the `exhaustive-deps` lint rule to catch missing dependencies.
- Stabilise functions with `useCallback` before passing them to effects or returning them.
- Use the latest-ref pattern or `useEffectEvent` for callbacks that need fresh values without resetting effects.
- Return stable functions from custom Hooks so consumers can use them in dependency arrays.
- Do not lie to the linter; restructure the code instead.
- Use `useMemo` for objects and arrays that are dependencies of other Hooks.
- Document whether the Hook's return values are stable.

**Constraints and Limitations:**
- The latest-ref pattern requires an extra effect and a ref; it is easy to get wrong.
- `useEffectEvent` is experimental in some React versions; check your version.
- Stable functions are not a substitute for correct dependencies; the values they close over must be current.
- The `exhaustive-deps` rule may suggest dependencies that cause infinite loops; restructure to avoid.
- Custom Hooks that return unstable values force consumers to memoise them, adding boilerplate.
- Overusing `useCallback` for every function adds overhead without benefit.

### Annotated Code Examples

**Example 1: Stable `useInterval` with Latest-Ref Pattern**

```jsx
function useInterval(callback, delay) {
  const savedCallback = React.useRef(callback);

  // Keep the ref up to date with the latest callback
  React.useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Set up the interval; only `delay` is a dependency
  React.useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

function AnimatedCounter() {
  const [count, setCount] = React.useState(0);
  const [isRunning, setIsRunning] = React.useState(true);
  const [step, setStep] = React.useState(1);

  useInterval(
    () => setCount((c) => c + step), // Always uses the latest `step`
    isRunning ? 1000 : null
  );

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setIsRunning((r) => !r)}>Toggle</button>
      <button onClick={() => setStep((s) => s + 1)}>Step: {step}</button>
    </div>
  );
}
```

**Expected Output:** The counter increments every second when running. Changing `step` does not reset the interval; the next tick uses the new step. Toggling `isRunning` pauses and resumes the interval.

**Why This Output Occurs:** The latest-ref pattern keeps `savedCallback.current` up to date without adding `callback` to the interval effect's dependencies. Changing `step` re-renders the component and updates the ref, but the interval is only cleared and re-created when `delay` changes.

**Example 2: Stable Functions Returned from a Hook**

```jsx
function useCounter(initial = 0) {
  const [count, setCount] = React.useState(initial);

  const increment = React.useCallback(() => setCount((c) => c + 1), []);
  const decrement = React.useCallback(() => setCount((c) => c - 1), []);
  const reset = React.useCallback(() => setCount(initial), [initial]);

  return { count, increment, decrement, reset };
}

function CounterWithLogging() {
  const { count, increment } = useCounter(0);

  // Safe: `increment` is stable, so the effect runs only once
  React.useEffect(() => {
    const id = setInterval(() => increment(), 1000);
    return () => clearInterval(id);
  }, [increment]);

  return <p>Count: {count}</p>;
}
```

**Expected Output:** The counter increments every second, and the effect runs only once (because `increment` is stable). Without `useCallback`, the effect would re-run on every render, clearing and re-creating the interval repeatedly.

**Why This Output Occurs:** `useCallback(() => setCount((c) => c + 1), [])` returns the same function reference across renders. The consumer's effect depends on `increment`, which never changes, so the interval is created once.

### Real-World Cases

- **Timers:** `useInterval`, `useTimeout` with stable callbacks.
- **Event listeners:** `useEventListener` with the latest-ref pattern.
- **Data fetching:** `useFetch` with a stable `refetch` function.
- **Debouncing:** `useDebouncedCallback` with a stable debounced function.
- **Polling:** `usePolling` with a stable polling function.
- **WebSocket:** `useWebSocket` with a stable send function.

### References

- React Official Documentation – Removing Effect Dependencies - https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – `useEffectEvent` - https://react.dev/reference/react/useEffectEvent
- React Official Documentation – `useCallback` - https://react.dev/reference/react/useCallback
- React Official Documentation – `exhaustive-deps` ESLint Rule - https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- Dan Abramov – A Complete Guide to useEffect - https://overreacted.io/a-complete-guide-to-useeffect/

---

## Comparison and Decision Guidance

| Concept | When to Use | When to Avoid | Key Risk |
|---|---|---|---|
| **Hook extraction** | Duplicate logic in 2+ components; logic obscures UI | One-off logic; trivial state | Over-extraction |
| **Reusable logic** | Sharing behaviour across components | Sharing state across components | Expecting shared state |
| **Naming conventions** | Every custom Hook | Functions that do not call Hooks | Runtime errors |
| **Rules of Hooks** | Every component and Hook | Never | Conditional Hook calls |
| **Composition** | Building complex Hooks from small ones | Simple, single-purpose Hooks | Over-composition |
| **Dependency management** | Hooks with `useEffect`/`useMemo`/`useCallback` | Hooks with no reactive values | Stale closures, infinite loops |
| **Latest-ref pattern** | Callbacks that need fresh values without resetting effects | When `useEffectEvent` is available | Incorrect ref updates |
| **`useEffectEvent`** | React 19+ projects | Older React versions | Experimental status |

**Decision Guidance:**
- **Extract** when the same logic appears twice, or when a component's logic is too complex to read.
- **Do not expect** custom Hooks to share state; lift state or use Context/store to share.
- **Always prefix** custom Hooks with `use`; enforce with `eslint-plugin-react-hooks`.
- **Compose** small Hooks into facade Hooks; keep each Hook focused on one concern.
- **Stabilise** functions returned from Hooks with `useCallback` so consumers can use them in dependency arrays.
- **Use the latest-ref pattern** (or `useEffectEvent` in React 19) for callbacks that need fresh values without resetting effects.
- **Satisfy `exhaustive-deps`**; do not disable it.
- **Document** the Hook's return values and whether they are stable.
- **Test** custom Hooks with `renderHook` from `@testing-library/react`.

---

## References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – `exhaustive-deps` ESLint Rule - https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React Official Documentation – `useEffectEvent` - https://react.dev/reference/react/useEffectEvent
- React Official Documentation – Removing Effect Dependencies - https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – `useCallback` - https://react.dev/reference/react/useCallback
- React Official Documentation – `useMemo` - https://react.dev/reference/react/useMemo
- React Official Documentation – `useRef` - https://react.dev/reference/react/useRef
- React Official Documentation – Sharing State Between Components - https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Passing Data Deeply with Context - https://react.dev/learn/passing-data-deeply-with-context
- eslint-plugin-react-hooks - https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks
- Dan Abramov – A Complete Guide to useEffect - https://overreacted.io/a-complete-guide-to-useeffect/
- Testing Library – `renderHook` - https://testing-library.com/docs/react-testing-library/api#renderhook
- Kent C. Dodds – When to useMemo and useCallback - https://kentcdodds.com/blog/usememo-and-usecallback