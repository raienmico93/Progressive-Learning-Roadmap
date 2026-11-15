# `use` Hook (React 19) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `use` hook is a React 19 API that reads the value of a resource — either a Promise or a Context — directly during component render, integrating with Suspense and error boundaries to handle loading and error states declaratively.

**Technical Definition:** `use<T>(resource: Usable<T>): T` is a React API that accepts a Promise or a Context object. When passed a Promise, React suspends the component until the Promise settles; the resolved value is returned, and rejections propagate to the nearest Error Boundary. When passed a Context, it behaves like `useContext` but is exempt from the top-level-only rule of Hooks, allowing it to be called conditionally and inside loops. `use` is technically not a Hook under the traditional Rules of Hooks — it is a React API that deliberately breaks the conditional-call rule while still requiring invocation inside a Component or Hook.

**Beginner-Friendly Explanation:** The `use` hook lets you "unwrap" a Promise or read a Context value right inside your component — no need for `useEffect` + `useState` boilerplate. If the data isn't ready yet, React automatically shows a loading fallback (via Suspense); if something goes wrong, an Error Boundary handles it. The best part: unlike every other React hook, you can call `use` inside `if` statements and loops.

### Key Characteristics

- **Dual-purpose:** Reads either a Promise (async data) or a Context value (shared state).
- **Conditional calling allowed:** Exempt from the top-level rule — can be called inside conditionals, loops, and after early returns.
- **Suspense-powered:** When reading a pending Promise, the component suspends and the nearest `<Suspense>` fallback renders.
- **Error Boundary integration:** Rejected Promises bubble to the nearest Error Boundary — `use` does not provide a local `catch`.
- **Render-only:** Must be called during render inside a Component or Hook — not in event handlers, effects, or regular functions.
- **Stable Promise requirement:** Promises passed to `use` must be created outside the component or memoized; creating them during render causes infinite suspense loops.

### Prerequisites

- React 19+ (the `use` API is stable in React 19; previously available in Canary/experimental channels).
- A React project with Suspense configured (Suspense boundaries are required when reading Promises).
- An Error Boundary (or `react-error-boundary`) for handling rejected Promises.
- Basic understanding of Promises, async/await, and React Context.

### Related Programming Areas

- **Suspense and concurrent rendering** — `use` is the primary client-side API for suspending on Promises.
- **React Server Components (RSC)** — Server Components use `async`/`await`; `use` is preferred in Client Components for reading Promises passed from the server.
- **Data fetching patterns** — `use` replaces the `useEffect` + `useState` + loading-state triplet for simple async reads.
- **Error boundaries** — Required to handle Promise rejections read by `use`.
- **Context API** — `use(Context)` is a more flexible alternative to `useContext`.

### Core Concepts / Features

---

## 1. `use` Hook

### Definitions

**Core Definition:** The `use` hook is a React 19 API for reading Promise values and Context values during render.

**Technical Definition:** `use` accepts a `Usable<T>` resource (Promise or Context) and returns the resolved value. For Promises, it suspends the component until the Promise settles, leveraging React's Suspense mechanism. For Context, it returns the current context value from the nearest provider. Unlike `useContext`, `use` can be called conditionally and within loops, making it the first React API to deliberately bypass the top-level rule of Hooks.

**Beginner-Friendly Explanation:** `use` is like a universal reader: give it a Promise and it waits for the data (showing a loading screen); give it a Context and it gives you the shared value. And you can use it inside `if` statements — something no other hook allows.

### Purposes

- To read the resolved value of a Promise directly in a component body without `useEffect` and `useState`. 
- To read a Context value with the flexibility of conditional and loop-based access.
- To delegate loading states to Suspense boundaries and error states to Error Boundaries.
- To simplify async data consumption in Client Components receiving Promises from Server Components.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { use } from 'react';

const value = use(resource);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `use` | The React API function imported from `'react'`. |
| `resource` | A Promise or a Context object. |
| `value` | The resolved Promise value or the current Context value. |

**Syntax Rules:**

1. `use` must be called inside a React Component or a custom Hook.
2. `use` can be called conditionally (inside `if`), inside loops, and after early returns — it is exempt from the top-level rule.
3. When reading a Promise, the component must be wrapped in a `<Suspense>` boundary to handle the pending state.
4. When reading a Promise, rejections must be handled by an Error Boundary — `use` cannot be wrapped in `try/catch`.
5. Promises passed to `use` must be stable across renders (created outside the component, memoized, or passed from a parent/Server Component).

**Constraints and Limitations:**

- `use` cannot be used in event handlers, `useEffect`, class methods, or regular JavaScript functions — it only works during render.
- Creating a Promise inside the component render body causes an infinite suspense loop (a new Promise every render).
- In Server Components, prefer `async`/`await` over `use` for data fetching.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Reading a Promise

```jsx
// Step 1: Import use and Suspense
import { use, Suspense } from 'react';

// Step 2: Define a child component that reads the Promise
function Message({ messagePromise }) {
  // use() unwraps the Promise; React suspends until it resolves
  const message = use(messagePromise);
  return <p>{message}</p>;
}

// Step 3: Parent creates the Promise (stable, outside render) and wraps in Suspense
function App() {
  const messagePromise = fetchMessage(); // Created once, outside the component

  return (
    <Suspense fallback={<p>Loading message...</p>}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}
```

**Expected Output (while pending):**

```html
<p>Loading message...</p>
```

**Expected Output (after resolution):**

```html
<p>Hello from the server!</p>
```

**Why this output:** `use(messagePromise)` suspends `Message` until `messagePromise` resolves. The Suspense fallback renders during the pending phase. Once resolved, `use` returns the value and `Message` renders with the data.

#### Example 2: Reading Context Conditionally

```jsx
import { use, createContext } from 'react';

const ThemeContext = createContext('light');

function ThemedButton({ show }) {
  if (show) {
    const theme = use(ThemeContext); // ✅ Conditional use is allowed
    return <button className={theme}>Themed Button</button>;
  }
  return <button>Plain Button</button>;
}
```

**Expected Output (when `show` is `true`):**

```html
<button class="light">Themed Button</button>
```

**Expected Output (when `show` is `false`):**

```html
<button>Plain Button</button>
```

**Why this output:** `use(ThemeContext)` is called only when `show` is truthy. If `show` is false, the hook is never invoked — this is legal because `use` is exempt from the top-level rule.

### Real-World Cases

- **Server-to-client data streaming:** A Server Component fetches data and passes a Promise to a Client Component, which reads it with `use` inside a Suspense boundary.
- **Conditional feature flags:** Reading a feature-flag Context only when a certain condition is met, avoiding unnecessary context reads.
- **Suspense-based data fetching:** Replacing `useEffect` + `useState` fetch patterns with declarative `use` + Suspense.

### References

- React — `use` — https://react.dev/reference/react/use
- React — Canary `use` API — https://18.react.dev/reference/react/use
- GitHub — React 19 Platform Hooks & APIs — https://github.com/bobmatnyc/claude-mpm-skills/blob/718070a7d622921b01687799a1f9613f36c6f615/toolchains/javascript/frameworks/react/react-advanced/references/react19-actions-and-apis.md

---

## 2. Reading Promises

### Definitions

**Core Definition:** Reading a Promise with `use` means passing a Promise object to the hook and receiving its resolved value, while React suspends the component until the Promise settles.

**Technical Definition:** When `use(promise)` is called, React inspects the Promise's status. If pending, React throws the Promise (suspends the component) and the nearest Suspense boundary renders its fallback. When the Promise resolves, React re-renders the component with the resolved value. If the Promise rejects, React throws the rejection reason, which propagates to the nearest Error Boundary.

**Beginner-Friendly Explanation:** Give `use` a Promise and it will wait for the data to arrive. While waiting, a loading screen (Suspense fallback) appears. When the data is ready, the component renders with the actual value.

### Purposes

- To consume async data (API responses, database reads) declaratively during render.
- To eliminate manual loading-state management (`useState` + `useEffect`).
- To integrate async data fetching with Suspense and Error Boundaries.
- To stream data from Server Components to Client Components via Promises.

### Syntax Rules and Structure

```jsx
const data = use(promise);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `promise` | A stable Promise object (created outside render or memoized). |
| `data` | The resolved value of the Promise. |

**Syntax Rules:**

1. The Promise must be created outside the component or memoized with `useMemo` to ensure stability across renders.
2. The component calling `use(promise)` must be wrapped in a `<Suspense>` boundary.
3. Rejections are handled by Error Boundaries, not `try/catch`. 
4. In Client Components, Promises are typically passed as props from a parent or Server Component.

**Constraints and Limitations:**

- **Never create the Promise inside the component render body** — this causes an infinite suspense loop because every render creates a new Promise.
- The resolved value must be serializable when passing Promises from Server Components to Client Components.
- `use` does not provide a local error state; errors bubble to Error Boundaries.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Stable Promise Passed as Prop

```jsx
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Async function (can be from an API module)
async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error('User not found');
  return res.json();
}

// Child component reads the Promise
function UserProfile({ userPromise }) {
  const user = use(userPromise); // Suspends until resolved
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// Parent creates the Promise once and wraps in Suspense + ErrorBoundary
function App() {
  const userPromise = fetchUser(123); // Stable: created outside the child

  return (
    <ErrorBoundary fallback={<p>Failed to load user.</p>}>
      <Suspense fallback={<p>Loading user...</p>}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Expected Output (while pending):**

```html
<p>Loading user...</p>
```

**Expected Output (after resolution):**

```html
<div>
  <h1>Ada Lovelace</h1>
  <p>ada@example.com</p>
</div>
```

**Expected Output (on rejection):**

```html
<p>Failed to load user.</p>
```

**Why this output:** The Promise is created once in `App` (not inside `UserProfile`). `use(userPromise)` suspends `UserProfile` until resolution. Suspense handles the loading state; the Error Boundary catches rejections.

#### Example 2: Common Mistake — Promise Created During Render

```jsx
// ⚠️ THIS CAUSES AN INFINITE SUSPENSE LOOP
function Profile({ id }) {
  const user = use(fetchUser(id)); // New Promise every render → loops
  return <h1>{user.name}</h1>;
}
```

**Why this is wrong:** Every render creates a new `fetchUser(id)` Promise. `use` suspends the component, React re-renders to resolve, creates another new Promise, and the cycle repeats infinitely.

**Correct approach:** Hoist the Promise outside the component or memoize it with `useMemo`:

```jsx
function Profile({ userPromise }) {
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}
```

### Real-World Cases

- **User profile pages:** Fetching user data via a Promise passed from a parent route component.
- **Product detail pages:** Reading a product Promise from a Server Component in a Client Component.
- **Dashboard widgets:** Each widget receives its own Promise and renders independently inside its own Suspense boundary.

### References

- React — `use` with Promises — https://react.dev/reference/react/use
- SitePoint — React 19 `use()` Hook: Data Fetching Patterns That Actually Work — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/

---

## 3. Reading Context

### Definitions

**Core Definition:** Reading Context with `use` means passing a Context object to the hook to retrieve the current context value, with the ability to call it conditionally or inside loops.

**Technical Definition:** `use(SomeContext)` returns the current value of `SomeContext` from the nearest provider, functioning similarly to `useContext(SomeContext)`. The key difference is that `use` is not bound by the top-level-only rule — it can be invoked inside `if` statements, `for` loops, and after early returns. This makes it the preferred API for conditional context reading in React 19.

**Beginner-Friendly Explanation:** `use(ThemeContext)` gives you the current theme value, just like `useContext`. But unlike `useContext`, you can call it inside an `if` statement. So if you only need the theme when a certain condition is true, you can read it only then.

### Purposes

- To read context values with conditional logic (e.g., only when a feature is enabled).
- To read context inside loops where the context may vary.
- To replace `useContext` with a more flexible API that aligns with React 19's concurrent patterns.
- To avoid unnecessary context reads when the value is not needed.

### Syntax Rules and Structure

```jsx
import { use } from 'react';

const value = use(SomeContext);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SomeContext` | A Context object created with `createContext()`. |
| `value` | The current context value from the nearest provider. |

**Syntax Rules:**

1. `use(Context)` works like `useContext(Context)` but can be called conditionally and inside loops.
2. The Context provider must be an ancestor of the component calling `use`.
3. `use` returns the default context value if no provider is found above.

**Constraints and Limitations:**

- Conditional context reading should be based on component logic (props, state), not on the context value itself (e.g., don't conditionally call `use` based on whether a value is cached).
- `use(Context)` is not a replacement for all `useContext` use cases; for unconditional top-level reads, either works.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Conditional Context Reading

```jsx
import { use, createContext } from 'react';

const ThemeContext = createContext('light');

function Button({ showTheme, children }) {
  if (showTheme) {
    const theme = use(ThemeContext); // ✅ Called conditionally
    return <button className={`btn-${theme}`}>{children}</button>;
  }
  return <button>{children}</button>;
}
```

**Expected Output (when `showTheme` is `true`):**

```html
<button class="btn-light">Click me</button>
```

**Expected Output (when `showTheme` is `false`):**

```html
<button>Click me</button>
```

**Why this output:** `use(ThemeContext)` is only called when `showTheme` is truthy. This conditional call is legal because `use` is exempt from the top-level rule.

#### Example 2: Context in a Loop

```jsx
import { use, createContext } from 'react';

const ItemContext = createContext({ color: 'gray' });

function ItemList({ showColors }) {
  const items = ['A', 'B', 'C'];

  return (
    <ul>
      {items.map((item) => {
        if (showColors) {
          const { color } = use(ItemContext); // ✅ Inside a loop
          return <li key={item} style={{ color }}>{item}</li>;
        }
        return <li key={item}>{item}</li>;
      })}
    </ul>
  );
}
```

**Expected Output (when `showColors` is `true`):**

```html
<ul>
  <li style="color: gray;">A</li>
  <li style="color: gray;">B</li>
  <li style="color: gray;">C</li>
</ul>
```

**Why this output:** `use(ItemContext)` is called inside a `map` loop. While unusual, this is permitted because `use` can be called in loops. Each iteration reads the same context value.

### Real-World Cases

- **Feature-flagged theming:** Reading a theme context only when a feature flag is enabled.
- **Conditional localization:** Reading a locale context only for components that display localized text.
- **Dynamic list rendering:** Reading a context inside a list iteration when each item needs context-derived styling.

### References

- React — `use` with Context — https://react.dev/reference/react/use
- Mintlify — `use` Hook (Context) — https://mintlify.wiki/facebook/react/api/hooks/use
- DEV Community — Use Hook with Context — https://dev.to/kkr0423/reactjs-hook-pattern-use-hook-with-context-595k

---

## 4. Conditional Use

### Definitions

**Core Definition:** Conditional use refers to the ability to call `use` inside `if` statements, loops, and after early returns — a deliberate exception to React's Rules of Hooks.

**Technical Definition:** The Rules of Hooks normally require all hooks to be called at the top level of a component, unconditionally, in the same order on every render. `use` is exempt from this rule because it does not rely on hook slot ordering in the same way. React permits conditional `use` calls, provided the condition is based on component props, state, or other render-time logic — not on the promise's internal status (e.g., `promise.status`).

**Beginner-Friendly Explanation:** Normally, React hooks must always be called in the exact same order. `use` is special: you can put it inside an `if` statement or a loop. This means you can skip reading data or context when you don't need it.

### Purposes

- To avoid unnecessary data fetching or context reads when a condition is not met.
- To read context or Promises only for certain branches of rendering logic.
- To simplify components that previously required workarounds (like always calling `useContext` and then checking the value).

### Syntax Rules and Structure

```jsx
// Conditional use with a Promise
if (shouldFetch) {
  const data = use(dataPromise);
}

// Conditional use with Context
if (showTheme) {
  const theme = use(ThemeContext);
}
```

**Syntax Rules:**

1. Conditional `use` is allowed when the condition is based on props, state, or render-time logic.
2. Do not conditionally call `use` based on the promise's `.status` property (e.g., `if (promise.status === 'fulfilled')`) — let `use` handle the status internally.
3. `use` can appear after early returns, inside loops, and in nested blocks (but still inside the component function).

**Constraints and Limitations:**

- Conditional `use` is still only valid during render — not in event handlers, effects, or callbacks.
- While conditionally reading a Promise is legal, the Promise itself must still be stable; conditional creation of Promises can lead to subtle bugs.
- React may emit a development warning if `use` is called conditionally based on cached value status.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Conditional Data Fetching

```jsx
import { use } from 'react';

function SearchResults({ query, searchPromise }) {
  if (!query) {
    return <p>Enter a search term to begin.</p>;
  }

  // ✅ use is called conditionally — only when query is truthy
  const results = use(searchPromise);

  return (
    <ul>
      {results.map((r) => <li key={r.id}>{r.title}</li>)}
    </ul>
  );
}
```

**Expected Output (when `query` is empty):**

```html
<p>Enter a search term to begin.</p>
```

**Expected Output (when `query` is non-empty):**

```html
<ul>
  <li>Result 1</li>
  <li>Result 2</li>
</ul>
```

**Why this output:** When `query` is empty, the early return prevents `use` from being called. When `query` is non-empty, `use(searchPromise)` reads the Promise. The conditional call is legal because `use` is exempt from the top-level rule.

#### Example 2: What NOT to Do — Condition Based on Promise Status

```jsx
// ⚠️ AVOID: Conditionally calling use based on promise.status
function BadExample({ promise }) {
  if (promise.status === 'fulfilled') {
    const data = use(promise); // React discourages this pattern
    return <p>{data}</p>;
  }
  return <p>Loading...</p>;
}
```

**Why this is discouraged:** React's internal Promise handling manages the status. Manually checking `promise.status` bypasses React's suspense coordination and can lead to incorrect behavior. Always pass the Promise directly to `use` and let React handle the status.

### Real-World Cases

- **Search-as-you-type:** Only reading a search results Promise when the query is non-empty.
- **Conditional context providers:** Reading a feature-flag context only for users in an experiment group.
- **Lazy-loaded sections:** Reading a Promise for a section's data only when the section is expanded or visible.

### References

- React — `use` Caveats — https://react.dev/reference/react/use
- GitHub — Conditional `use()` warning (PR #34030) — https://github.com/facebook/react/pull/34030
- ByteChalk — React 19 `use` API: Promises, Context & Suspense — https://www.bytechalk.com/react-19-use-api-promises-context-suspense/

---

## 5. Async Components

### Definitions

**Core Definition:** Async components are React components that perform asynchronous operations (typically awaiting data) before rendering their JSX, enabled in client components through the `use` hook and Suspense.

**Technical Definition:** In React 19, **Client Components cannot be declared as `async function`** — the `async` keyword is reserved for Server Components, which use `await` directly. Client Components achieve async data reading through `use(promise)` inside a synchronous component wrapped in `<Suspense>`. The Promise is typically created by a parent or Server Component and passed down as a prop. This pattern is sometimes called "async components" in the client context, but technically the component itself remains synchronous; `use` handles the suspension.

**Beginner-Friendly Explanation:** You might hear "async components" and think of a component that `await`s data. In client React 19, you don't write `async function MyComponent()`. Instead, you write a normal component that calls `use(promise)` — React automatically suspends it until the data is ready. The component code stays clean and synchronous-looking.

### Purposes

- To consume asynchronous data in Client Components without `useEffect` and state management.
- To enable Suspense-based loading states for client-side data fetching.
- To stream Promises from Server Components to Client Components for progressive rendering.
- To keep component logic declarative and free of manual loading/error state machines.

### Syntax Rules and Structure

**Correct Pattern (Client Component):**

```jsx
// ✅ Synchronous component using use() with a Promise
function UserProfile({ userPromise }) {
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}
```

**Incorrect Pattern (Client Component):**

```jsx
// ⚠️ Do NOT declare Client Components as async
'use client';
async function UserProfile() {
  const user = await fetchUser(); // Not supported in Client Components
  return <h1>{user.name}</h1>;
}
```

**Component Breakdown:**

| Pattern | Environment | How async data is read |
|---------|-------------|------------------------|
| `async function Component() { await ... }` | Server Components | Native `async`/`await`. |
| `function Component() { const data = use(promise); }` | Client Components | `use` + Suspense. |

**Syntax Rules:**

1. Client Components should be synchronous functions; use `use` to read Promises inside them.
2. The Promise should be created outside the Client Component (in a parent, Server Component, or event handler).
3. Wrap the component in a `<Suspense>` boundary to handle the loading state.
4. In Server Components, prefer `async`/`await` over `use` for data fetching.

**Constraints and Limitations:**

- `async function` components are not supported in Client Components (with `'use client'`). 
- Creating Promises inside Client Component render bodies causes infinite suspense loops.
- Hooks (including `use`) cannot be called inside `async` functions that are not React components.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Server-to-Client Promise Streaming

```jsx
// Server Component (no 'use client')
import { fetchUser } from './api';

export default async function Page() {
  const userPromise = fetchUser(123); // Created on the server
  return (
    <Suspense fallback={<p>Loading user...</p>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

```jsx
// Client Component ('use client')
'use client';
import { use } from 'react';

export function UserProfile({ userPromise }) {
  const user = use(userPromise); // Reads the Promise passed from the server
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**Expected Output (while pending):**

```html
<p>Loading user...</p>
```

**Expected Output (after resolution):**

```html
<div>
  <h1>Ada Lovelace</h1>
  <p>ada@example.com</p>
</div>
```

**Why this output:** The Server Component creates the Promise and passes it to the Client Component. The Client Component uses `use` to read the Promise. Suspense on the server side handles the loading state, and the resolved data streams to the client.

### Real-World Cases

- **Server-driven UIs:** Server Components fetch data and stream Promises to Client Components for interactive rendering.
- **Progressive loading:** Parts of the page render as their Promises resolve, thanks to Suspense boundaries.
- **Third-party API integration:** Client Components read Promises from API calls initiated by parent components.

### References

- React — Server Components and `use` — https://react.dev/reference/rsc/use-client
- GitHub — Hooks are not supported inside an async component — https://errors.standardbeagle.com
- GitHub — Async components with Suspense — https://github.com/namnm/rntwsc/blob/master/docs/async-components.md

---

## 6. Suspense Integration

### Definitions

**Core Definition:** Suspense integration means that when `use` reads a pending Promise, React suspends the component and renders the nearest `<Suspense>` fallback until the Promise resolves.

**Technical Definition:** `use` leverages React's Suspense mechanism by "throwing" the Promise when it is pending. React catches this thrown Promise, walks up the component tree to the nearest `<Suspense>` boundary, and renders its `fallback` prop. When the Promise resolves, React re-renders the suspended component with the resolved value, replacing the fallback. For rejected Promises, React throws the rejection reason, which is caught by the nearest Error Boundary.

**Beginner-Friendly Explanation:** Suspense is React's built-in loading screen system. When `use` is waiting for data, React shows the `<Suspense fallback={...}>` content. When the data arrives, the real content appears. If the data fetch fails, an Error Boundary catches it.

### Purposes

- To declaratively handle loading states without manual `isLoading` flags.
- To coordinate multiple async data reads under a single loading boundary.
- To enable progressive rendering where parts of the UI appear as their data resolves.
- To integrate error handling through Error Boundaries for failed Promises.

### Syntax Rules and Structure

```jsx
<Suspense fallback={<Loading />}>
  <ComponentThatUsesPromise />
</Suspense>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<Suspense>` | The React component that provides the fallback UI. |
| `fallback` | The UI to render while the child component is suspended. |
| `ComponentThatUsesPromise` | A component that calls `use(promise)`. |

**Syntax Rules:**

1. A `<Suspense>` boundary must wrap any component that calls `use` with a pending Promise.
2. Multiple components can share one Suspense boundary; the fallback renders until all suspended children resolve.
3. Suspense boundaries can be nested for granular loading control.
4. Error Boundaries must wrap Suspense boundaries to handle rejected Promises (or be placed above them).

**Constraints and Limitations:**

- Suspense does not catch errors — rejected Promises require Error Boundaries.
- If no Suspense boundary exists, React will throw an error when a component suspends.
- Suspense fallbacks should be meaningful and accessible; avoid blank screens.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Suspense + Error Boundary

```jsx
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function Message({ messagePromise }) {
  const message = use(messagePromise); // Suspends until resolved
  return <p>{message}</p>;
}

function App() {
  const messagePromise = fetchMessage();

  return (
    <ErrorBoundary fallback={<p>Error loading message.</p>}>
      <Suspense fallback={<p>Loading message...</p>}>
        <Message messagePromise={messagePromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Expected Output (pending):**

```html
<p>Loading message...</p>
```

**Expected Output (resolved):**

```html
<p>Hello!</p>
```

**Expected Output (rejected):**

```html
<p>Error loading message.</p>
```

**Why this output:** `use` throws the pending Promise to trigger Suspense. The fallback renders. On resolution, the component re-renders with data. On rejection, the Error Boundary catches the thrown reason and renders its fallback.

#### Example 2: Nested Suspense Boundaries

```jsx
function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Header />
      <Suspense fallback={<ContentSkeleton />}>
        <MainContent />
      </Suspense>
    </Suspense>
  );
}
```

**Why this is useful:** The outer Suspense handles the overall page loading. The inner Suspense allows the header to appear while the main content is still loading, improving perceived performance.

### Real-World Cases

- **Page-level loading states:** Wrapping entire route components in Suspense for initial data loading.
- **Progressive dashboards:** Multiple Suspense boundaries around different dashboard widgets, each resolving independently.
- **Streaming SSR:** Suspense boundaries on the server stream fallbacks first, then replace them as data resolves.

### References

- React — `<Suspense>` — https://react.dev/reference/react/Suspense
- React — `use` and Suspense — https://18.react.dev/reference/react/use
- Epic React — How React Suspense Works Under the Hood — https://www.epicreact.dev/how-react-suspense-works-under-the-hood

---

## 7. `use` Best Practices

### Definitions

**Core Definition:** `use` best practices are guidelines for safely and effectively using the `use` hook with Promises and Context in React 19 applications.

**Technical Definition:** Best practices encompass Promise stability (never creating Promises during render), proper Suspense and Error Boundary usage, correct conditional calling patterns, avoiding mixing data-fetching paradigms, and understanding when `use` is appropriate versus when `useEffect` or Server Component `async`/`await` is preferred.

**Beginner-Friendly Explanation:** Best practices are the "rules of thumb" that keep your `use` code bug-free and performant — like always creating Promises outside your component and always wrapping `use` calls in Suspense.

### Purposes

- To prevent infinite suspense loops caused by unstable Promises.
- To ensure loading and error states are handled gracefully.
- To maintain clean, maintainable component architecture.
- To choose the right data-reading approach for each scenario (Client vs. Server, Promise vs. Context).

### Syntax Rules and Structure

**Best Practices Checklist:**

| Practice | Description |
|----------|-------------|
| **Stable Promises only** | Create Promises outside the component, memoize with `useMemo`, or pass from a parent/Server Component. |
| **Never wrap `use` in `try/catch`** | Rejections surface through Error Boundaries, not `catch` blocks. |
| **Always wrap Promise reads in Suspense** | A Suspense boundary is required to handle pending states. |
| **Pair with Error Boundaries** | Wrap Suspense in an Error Boundary to handle rejected Promises. |
| **Don't conditionally call `use` based on `promise.status`** | Let React manage the Promise status internally. |
| **Don't mix `use` and `useEffect` for the same data** | Pick one source of truth for each piece of data. |
| **Prefer `use` in Client Components; prefer `async`/`await` in Server Components** | Server Components use native async/await; Client Components use `use`. |
| **Keep conditional `use` based on props/state, not cached values** | Conditional calling is allowed, but the condition should be deterministic. |
| **Use `use(Context)` for conditional context reads** | `useContext` must be top-level; `use` can be conditional. |

**Constraints and Limitations:**

- `use` cannot replace all `useEffect` use cases — it reads values, it does not execute side effects.
- Creating Promises in the render body of Client Components is the most common pitfall.
- Error Boundaries must be class components (or use `react-error-boundary`) — function components cannot be Error Boundaries.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Correct Data Fetching Pattern

```jsx
import { use, Suspense, useMemo } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
}

// Child: reads the Promise
function UserProfile({ userPromise }) {
  const user = use(userPromise); // ✅ Stable Promise passed from parent
  return <h1>{user.name}</h1>;
}

// Parent: creates stable Promise with useMemo
function App({ userId }) {
  const userPromise = useMemo(() => fetchUser(userId), [userId]);

  return (
    <ErrorBoundary fallback={<p>Error loading user.</p>}>
      <Suspense fallback={<p>Loading...</p>}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Why this follows best practices:**

- The Promise is memoized with `useMemo`, ensuring stability across renders.
- `use` is wrapped in Suspense (for pending) and an Error Boundary (for rejections).
- The child component reads the Promise declaratively without any `useEffect` or state. 

#### Example 2: Mixing Paradigms (Anti-Pattern)

```jsx
// ⚠️ ANTI-PATTERN: Fetching the same data twice
function BadComponent({ userPromise }) {
  const userFromUse = use(userPromise);
  const [userFromEffect, setUserFromEffect] = useState(null);

  useEffect(() => {
    fetchUser().then(setUserFromEffect); // Duplicate fetching!
  }, []);

  // Two sources of truth for the same data
  return <h1>{userFromUse.name || userFromEffect?.name}</h1>;
}
```

**Why this is wrong:** Reading the same data with both `use` and `useEffect` creates two sources of truth, potential race conditions, and unnecessary network requests. Pick one paradigm.

### Real-World Cases

- **Production apps:** Following these practices prevents the most common `use`-related bugs (infinite loops, unhandled errors, duplicate fetches).
- **Team codebases:** Consistent patterns make code reviews and onboarding easier.
- **Performance-critical UIs:** Stable Promises and proper Suspense boundaries ensure smooth, predictable rendering.

### References

- React — `use` Caveats — https://react.dev/reference/react/use
- GitHub — React 19 Platform Hooks & APIs (Rules and pitfalls) — https://github.com/bobmatnyc/claude-mpm-skills/blob/718070a7d622921b01687799a1f9613f36c6f615/toolchains/javascript/frameworks/react/react-advanced/references/react19-actions-and-apis.md
- SitePoint — React 19 `use()` Hook: Common Pitfalls — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/
- DEV Community — `use()` hook and Suspense: when it replaces `useEffect` — https://dev.to/jtorchia/react-19-use-hook-and-suspense-when-it-replaces-useeffect-and-when-it-throws-you-into-a-worse-eei

---

## References

- React — `use` — https://react.dev/reference/react/use
- React — Canary `use` API — https://18.react.dev/reference/react/use
- React — `<Suspense>` — https://react.dev/reference/react/Suspense
- React — Server Components and Client Components — https://react.dev/reference/rsc/use-client
- Mintlify — `use` Hook — https://mintlify.wiki/facebook/react/api/hooks/use
- GitHub — React 19 Platform Hooks & APIs — https://github.com/bobmatnyc/claude-mpm-skills/blob/718070a7d622921b01687799a1f9613f36c6f615/toolchains/javascript/frameworks/react/react-advanced/references/react19-actions-and-apis.md
- GitHub — React 19 `use()` Hook Pattern Reference — https://github.com/github/awesome-copilot/blob/4725261086aa385231fb2accf3764a535b422275/skills/react19-concurrent-patterns/references/react19-use.md
- SitePoint — React 19 `use()` Hook: Data Fetching Patterns That Actually Work — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/
- DEV Community — React 19 `use()` hook and Suspense — https://dev.to/jtorchia/react-19-use-hook-and-suspense-when-it-replaces-useeffect-and-when-it-throws-you-into-a-worse-eei
- Steve Kinney — The `use()` Hook — https://stevekinney.com/courses/react-performance/the-use-hook
- ByteChalk — React 19 `use` API: Promises, Context & Suspense — https://www.bytechalk.com/react-19-use-api-promises-context-suspense/