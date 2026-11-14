# React Concurrent Rendering — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrent rendering is React's ability to prepare a new user interface in the background while keeping the current interface responsive, interrupting or pausing rendering work when more urgent updates arrive.

**Technical Definition:** Concurrent rendering is a scheduling architecture built on React's Fiber engine that allows the framework to interrupt, pause, resume, and abandon rendering work. It uses time-slicing and priority-based scheduling to split rendering into small work units, yielding to the browser between units so that urgent updates (typing, clicking) preempt non-urgent ones (filtering, data visualisation). In React 18, concurrent rendering was opt-in — it activated only when you used a concurrent API like `startTransition` or `useDeferredValue`. React 19 makes concurrent rendering the default scheduling behaviour for all components, with every app receiving interruptible rendering, priority-based scheduling, and approximately 5ms yield windows out of the box.

**Beginner-Friendly Explanation:** Concurrent rendering is React's multitasking ability. Imagine you are cooking several dishes at once. Before concurrent rendering, React would finish one dish completely before starting the next — even if a pot was boiling over. With concurrent rendering, React can pause a slow dish, attend to something urgent (like a user clicking a button), and come back to the slow dish later. This keeps the app responsive even when React is doing heavy work in the background. The features described in this cheat sheet are the tools that let you tell React which work is urgent and which can wait.

### Key Characteristics

- **Interruptible rendering:** React can pause, resume, or abandon rendering work at Fiber boundaries.
- **Priority-based scheduling:** Urgent updates (typing, clicking) preempt non-urgent updates (filtering, charting).
- **Time slicing:** Rendering work is split into ~5ms windows, yielding to the browser between windows to check for urgent tasks.
- **Default in React 19:** Concurrent rendering is the default scheduling behaviour, not an opt-in mode.
- **Cooperative multitasking:** The render phase is interruptible; the commit phase remains synchronous and atomic.
- **Opt-in API surface:** `useTransition`, `useDeferredValue`, and `Suspense` are the developer-facing entry points.
- **Reorders, does not reduce:** Concurrent features reorder work; they do not make the underlying work cheaper. Combine with memoization to reduce work itself.

### Prerequisites

- React function components, state, and `useEffect`.
- Understanding of the render → reconcile → commit pipeline.
- Familiarity with Fiber architecture and priority scheduling.
- Basic knowledge of JavaScript Promises and asynchronous programming.

### Related Programming Areas

- **React Fiber** — the architecture that enables concurrent rendering.
- **Code Splitting** — `Suspense` and `React.lazy` for lazy loading.
- **Streaming SSR and Selective Hydration** — server-rendering improvements built on concurrent features.
- **Core Web Vitals** — INP (Interaction to Next Paint) is the primary metric that concurrent features improve.

---

## 1. Concurrent Rendering

### Definitions

**Core Definition:** Concurrent rendering is React's interruptible rendering model that allows the framework to prepare multiple UI versions simultaneously and prioritise urgent updates.

**Technical Definition:** Concurrent rendering is a scheduling architecture where React can pause, interrupt, and resume renders. It prepares a new UI in the background while keeping the current UI interactive. Urgent updates (typing, clicking) stay responsive while expensive updates (filtering thousands of rows, navigating) render without blocking input. The key difference from traditional rendering is that the render phase is interruptible — React can yield to the browser between work units to process input and paint frames.

**Beginner-Friendly Explanation:** Concurrent rendering means React can work on a slow update in the background while you keep using the app. If something urgent happens, React pauses the slow update and handles the urgent thing first.

### Purposes

- To keep the UI responsive during expensive renders.
- To prioritise urgent updates (typing, clicking) over non-urgent ones (filtering, data visualisation).
- To prevent long tasks from blocking the main thread and degrading Interaction to Next Paint (INP).
- To enable features like `Suspense`, transitions, streaming SSR, and selective hydration.

### Syntax Rules and Structure

**Concurrent Rendering Behaviour:**

| Scenario | React Behaviour |
|----------|-----------------|
| Urgent update arrives | Pause current render, switch to urgent work |
| Urgent work completes | Resume or restart interrupted work |
| Work becomes stale | Abandon the interrupted render |
| Commit phase | Cannot be interrupted (atomic) |

**Syntax Rules:**

1. Concurrent rendering is the default in React 19; no opt-in flag is required.
2. Rendering work is sliced into ~5ms windows with yield points.
3. The commit phase is synchronous and atomic; only the render phase is interruptible.
4. Components must remain pure — React may pause, abandon, and restart a render.

**Constraints and Limitations:**

- Rendering must be pure; side effects during render can double-fire under concurrent scheduling.
- Concurrent rendering reorders work; it does not reduce the cost of work itself.
- Impure renders (mutations, analytics during render) are correctness bugs, not just performance issues.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Concurrent Rendering Enables Interruptible Search

```jsx
import { useState, useTransition } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setQuery(e.target.value); // Urgent: update input immediately
    startTransition(() => {
      // Non-urgent: expensive search can be interrupted
      setResults(expensiveSearch(e.target.value));
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Searching…</p>}
      <ResultsList results={results} />
    </>
  );
}
```

**Expected Output:** The input remains responsive while the search runs in the background. `isPending` indicates background work.

**Why this output:** `startTransition` marks the search update as non-urgent. Concurrent rendering allows React to interrupt the search to process the urgent input update.

#### Example 2: Concurrent-Safe Render Purity

```jsx
// ❌ Impure: side effect during render — may double-fire under concurrent rendering
let globalId = 0;
function ImpureCard({ user }) {
  const id = globalId++; // Side effect during render
  analytics.track('view', user.id); // Called multiple times!
  return <div id={`card-${id}`}>{user.name}</div>;
}

// ✅ Pure: side effects in effects, stable IDs from useId
import { useId, useEffect } from 'react';

function PureCard({ user }) {
  const id = useId();
  useEffect(() => {
    analytics.track('view', user.id); // Runs once per commit
  }, [user.id]);
  return <div id={id}>{user.name}</div>;
}
```

**Expected Output:** The impure version may fire analytics multiple times. The pure version fires once per commit.

**Why this output:** React may pause, abandon, and restart a render under concurrent rendering. Anything observable from the outside belongs in an effect or ref, never in render.

### Real-World Cases

- **Search autocomplete:** Typing must remain responsive while results are computed.
- **Tab switching:** Navigating between heavy tabs should not freeze the input.
- **Data dashboards:** Charts can render in the background while filters remain interactive.

### References

- React Concurrent Features (Claude Dev Suite) — https://github.com/claude-dev-suite/claude-dev-suite/blob/133ba99731744ee6c84fd892181471580f84d315/skills/frontend-frameworks/react-concurrent/SKILL.md
- Concurrent Rendering (Bobmatnyc Skills) — https://github.com/bobmatnyc/claude-mpm-skills/blob/718070a7d622921b01687799a1f9613f36c6f615/toolchains/javascript/frameworks/react/react-advanced/references/concurrent-rendering.md
- Concurrent React and Scheduling (Steve Kinney) — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 2. Concurrent Mode

### Definitions

**Core Definition:** Concurrent Mode is the historical name for React's concurrent rendering architecture, which became the default behaviour in React 18 and was further integrated in React 19.

**Technical Definition:** Concurrent Mode was the experimental umbrella term for React's interruptible rendering capabilities during the React 16–17 era. In React 18, the concept was formalised: concurrent rendering became an opt-in behaviour activated by concurrent APIs, and the separate "mode" was removed. React 19 makes concurrent rendering the default for all apps, without requiring any mode switch. The features once described as "Concurrent Mode" are now simply "Concurrent Features" — `useTransition`, `useDeferredValue`, and `Suspense`.

**Beginner-Friendly Explanation:** "Concurrent Mode" was the old name for React's ability to multitask. It used to be a special mode you had to turn on. Now it is just how React works by default.

### Purposes

- To understand the historical evolution of React's concurrent capabilities.
- To clarify that "Concurrent Mode" is no longer a separate mode in modern React.
- To recognise that concurrent features are the current API surface.

### Syntax Rules and Structure

**Evolution of Concurrent Rendering:**

| Era | Status |
|-----|--------|
| React 16–17 | Experimental "Concurrent Mode" (opt-in, unstable) |
| React 18 | Concurrent rendering available via opt-in APIs |
| React 19 | Concurrent rendering is the default for all components |

**Syntax Rules:**

1. Do not look for a "Concurrent Mode" flag in React 18+; it does not exist.
2. Concurrent features are the API surface: `useTransition`, `useDeferredValue`, `Suspense`.
3. React 19 apps get concurrent rendering automatically.

### Annotated Code Examples

```jsx
// React 18+: no "Concurrent Mode" wrapper needed
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />); // Concurrent rendering is available by default
```

**Expected Output:** The app renders with concurrent rendering capabilities; no special mode is required.

**Why this output:** React 18 removed the separate Concurrent Mode; concurrent rendering is activated by concurrent APIs and is the default in React 19.

### Real-World Cases

- **Migration from React 17:** Understanding that no mode switch is needed.
- **React 19 adoption:** Concurrent rendering is automatic.
- **Legacy documentation:** Recognising outdated "Concurrent Mode" references.

### References

- Modern React (18–19.2) Reference — https://github.com/... (Concurrent features opt-in in 18, default in 19)
- React 19 Async Rendering Changes — https://blog.openreplay.com

---

## 3. Concurrent Features

### Definitions

**Core Definition:** Concurrent features are the developer-facing APIs that let you opt into concurrent rendering behaviour — `useTransition`, `useDeferredValue`, `Suspense`, and the `use` Hook.

**Technical Definition:** Concurrent features are the APIs that expose React's concurrent rendering capabilities. They include `useTransition` (marks state updates as non-urgent transitions), `useDeferredValue` (defers a value so expensive renders lag behind), `Suspense` (shows a fallback while children load), and `use` (reads a promise or context, integrating with Suspense). In React 19, `startTransition` also accepts async functions (Actions), and `use` is stable.

**Beginner-Friendly Explanation:** Concurrent features are the tools you use to tell React what is urgent and what can wait. They are the switches that activate React's multitasking ability.

### Purposes

- To mark updates as non-urgent and interruptible.
- To defer expensive renders without blocking urgent input.
- To show loading states declaratively with `Suspense`.
- To read asynchronous data with `use()` and integrate with Suspense.
- To enable concurrent behaviour without a separate mode.

### Syntax Rules and Structure

**Concurrent Features at a Glance:**

| API | Purpose | Returns |
|-----|---------|---------|
| `useTransition()` | Mark updates non-blocking + track pending | `[isPending, startTransition]` |
| `startTransition()` | Mark updates non-blocking (no pending) | `void` |
| `useDeferredValue(value)` | Defer re-renders for a value | deferred value |
| `<Suspense fallback>` | Show fallback while children load | JSX boundary |
| `use(resource)` | Read promise or context (React 19) | resolved value |

**Syntax Rules:**

1. `useTransition` and `startTransition` mark updates as non-urgent.
2. `useDeferredValue` defers a value; React controls when to update it.
3. `Suspense` declares a loading boundary for lazy components and Suspense-enabled data.
4. `use()` reads promises and integrates with `Suspense` for data fetching.

### Annotated Code Examples

```jsx
import { useState, useTransition, useDeferredValue, Suspense } from 'react';

function App() {
  const [tab, setTab] = useState('home');
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const [isPending, startTransition] = useTransition();

  return (
    <Suspense fallback={<PageSkeleton />}>
      <nav>
        <button onClick={() => startTransition(() => setTab('posts'))}>
          Posts
        </button>
      </nav>
      {isPending && <p>Loading tab…</p>}
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <SearchResults query={deferredQuery} />
    </Suspense>
  );
}
```

**Expected Output:** Tab navigation is non-urgent and shows a pending indicator; search input is urgent and results use the deferred value.

**Why this output:** `startTransition` marks the tab update as non-urgent; `useDeferredValue` lets the results lag behind the input; `Suspense` provides a loading boundary.

### Real-World Cases

- **Search interfaces:** `useDeferredValue` for results, urgent input.
- **Tab navigation:** `startTransition` for heavy content.
- **Code splitting:** `Suspense` + `lazy`.
- **Data fetching:** `use()` + `Suspense` in React 19.

### References

- Concurrent Features (React 18+) — https://github.com/syngrisi/syngrisi/.../concurrent-features.md
- Concurrent React (TypeScript Cheatsheets) — https://github.com/typescript-cheatsheets/react/.../concurrent.md
- React 19 Concurrent Patterns — https://github.com/github/awesome-copilot/.../react19-concurrent-patterns

---

## 4. Transitions

### Definitions

**Core Definition:** A transition is a state update that React treats as non-urgent, allowing it to be interrupted by more important updates.

**Technical Definition:** A transition marks an update as non-urgent. State updates inside `startTransition` are assigned to a lower-priority lane in React's scheduler. If an urgent update arrives (typing, clicking), React abandons the in-progress background render and restarts it with the latest state. Transitions are created with `startTransition` (standalone or from `useTransition`). Transitions reorder work; they do not make work cheaper.

**Beginner-Friendly Explanation:** A transition is a way of telling React: "This update can wait. If the user does something more important, drop this and handle that first."

### Purposes

- To mark expensive state updates as non-urgent.
- To keep the UI responsive during background rendering.
- To allow urgent updates to interrupt and preempt non-urgent work.
- To prevent non-urgent updates from blocking user input.

### Syntax Rules and Structure

**Using `startTransition`:**

```jsx
import { startTransition } from 'react';

startTransition(() => {
  setExpensiveState(newValue);
});
```

**Using `useTransition`:**

```jsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setExpensiveState(newValue);
});
```

**Syntax Rules:**

1. Wrap state updates in `startTransition` to mark them as transitions.
2. Updates inside a transition are interruptible and non-blocking.
3. `isPending` indicates whether a transition is in progress.
4. Only updates issued synchronously within the transition scope belong to the transition. Updates after an `await` must be re-wrapped in another `startTransition`.

**Constraints and Limitations:**

- You can only wrap updates if you have access to the state's set function.
- State updates in `setTimeout` are not marked as transitions.
- Transitions reorder work; they do not reduce the cost of the work itself.

### Annotated Code Examples

```jsx
import { useState, useTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <div>
      <button onClick={() => selectTab('about')}>About</button>
      <button onClick={() => selectTab('posts')}>Posts</button>
      {isPending && <Spinner />}
      {tab === 'about' ? <About /> : <Posts />}
    </div>
  );
}
```

**Expected Output:** Clicking a tab updates the active tab in the background. The UI remains responsive, and a spinner appears while the transition is pending.

**Why this output:** `startTransition` marks `setTab` as non-urgent. React renders the new tab in the background and commits when ready, without blocking input.

### Real-World Cases

- **Tab navigation:** Switching between heavy tabs without blocking clicks.
- **Search filtering:** Updating results without interrupting typing.
- **Route changes:** Navigating between screens while keeping the current screen interactive.

### References

- Concurrent Rendering (Bobmatnyc) — https://github.com/bobmatnyc/claude-mpm-skills/.../concurrent-rendering.md
- Concurrent Features (React 18+) — https://github.com/syngrisi/syngrisi/.../concurrent-features.md

---

## 5. Suspense

### Definitions

**Core Definition:** `Suspense` is a React component that displays a fallback UI while its children are loading, and it serves as the boundary for streaming SSR and selective hydration.

**Technical Definition:** `<Suspense fallback={...}>` catches suspended rendering in its subtree. When used with `React.lazy`, it shows the fallback while a lazy component's code loads. When used with Suspense-enabled data sources (`use()`, framework data layers), it shows the fallback while data is fetched. `Suspense` is integrated with streaming server rendering and selective hydration: each `Suspense` boundary becomes an independently streamable and hydratable chunk.

**Beginner-Friendly Explanation:** `Suspense` is the loading boundary. You wrap a part of the UI in `Suspense` and give it a fallback. If that part is not ready — because its code or data is still loading — React shows the fallback. When it is ready, React replaces the fallback with the real content.

### Purposes

- To provide a loading state for lazy-loaded components and Suspense-enabled data.
- To define independently streamable and hydratable chunks for SSR.
- To prevent slow components from blocking the entire page.
- To coordinate loading states across a subtree.

### Syntax Rules and Structure

```jsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

**Syntax Rules:**

1. `Suspense` requires a `fallback` prop; the fallback can be any `ReactNode`, including `null`.
2. Children can be any React node; if they suspend, the fallback is rendered.
3. For SSR, `Suspense` boundaries define chunks that stream independently.
4. Only Suspense-enabled data sources (frameworks, `use()`, `lazy()`) can trigger `Suspense`.

**Constraints and Limitations:**

- State is not preserved for a tree that suspends before its first mount.
- If a visible tree suspends again, the fallback is shown unless the update was a transition or deferred value.
- Suspense does not detect data fetching in event handlers or `useEffect`.

### Annotated Code Examples

```jsx
import { Suspense } from 'react';
import Albums from './Albums';

function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>Loading albums…</h2>;
}
```

**Expected Output:** The artist name renders immediately. The albums section shows "Loading albums…" until the data or code is ready, then renders the albums.

**Why this output:** `Albums` suspends while its data loads. The nearest `Suspense` boundary renders the fallback, while the rest of the page remains visible.

### Real-World Cases

- **Lazy-loaded routes:** Showing a page skeleton while the route chunk loads.
- **Slow data sections:** Streaming in comments or recommendations after the main content.
- **Modals and editors:** Deferring heavy component code until the user opens them.

### References

- Suspense, Error Boundaries, and Concurrent Features (DeepWiki) — https://deepwiki.com/facebook/react
- Concurrent React (TypeScript Cheatsheets) — https://github.com/typescript-cheatsheets/react/.../concurrent.md

---

## 6. `useTransition`

### Definitions

**Core Definition:** `useTransition` is a React Hook that returns a pending flag and a function to mark state updates as non-urgent transitions.

**Technical Definition:** `const [isPending, startTransition] = useTransition()` returns a tuple. `isPending` is a boolean indicating whether a transition is in progress. `startTransition` is a function that accepts an action (a synchronous or async function that calls state setters) and marks all state updates scheduled synchronously during that action as transitions. In React 19, `startTransition` supports async functions (Actions), and `isPending` remains `true` for the entire duration of the async operation, including awaited work.

**Beginner-Friendly Explanation:** `useTransition` gives you two things: a way to start a background update (`startTransition`) and a flag (`isPending`) that tells you whether that background update is still running.

### Purposes

- To mark state updates as non-urgent transitions.
- To track whether a transition is pending with `isPending`.
- To keep the UI responsive during expensive updates.
- To avoid showing fallbacks for non-urgent updates.

### Syntax Rules and Structure

```jsx
const [isPending, startTransition] = useTransition();
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `isPending` | `true` while a transition is in progress. |
| `startTransition` | Function that marks its synchronous state updates as transitions. |

**Syntax Rules:**

1. Call `useTransition` at the top level of a component or custom Hook.
2. `startTransition` calls its action immediately and marks synchronous state updates as transitions.
3. State updates after an `await` must be wrapped in another `startTransition` unless using React 19 async actions.
4. `isPending` can be used to disable buttons or show subtle pending indicators.

**Constraints and Limitations:**

- Cannot be used to wrap updates whose `set` function you do not control.
- If you need to defer a value you do not control, use `useDeferredValue` instead.
- Never use `startTransition` or `useTransition` for controlled text input updates — transitions are interruptible and will cause input lag.

### Annotated Code Examples

```jsx
import { useState, useTransition } from 'react';

function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (value) => {
    setQuery(value); // Urgent
    startTransition(() => {
      setResults(filterItems(value)); // Non-urgent
    });
  };

  return (
    <div>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <span>Updating results…</span>}
      <ResultsList results={results} />
    </div>
  );
}
```

**Expected Output:** The input updates immediately. The results update in the background, and the pending indicator appears while the transition runs.

**Why this output:** `setQuery` is urgent and processed immediately. `setResults` is inside `startTransition`, so it is interruptible and does not block input.

### Real-World Cases

- **Search and filtering:** Keeping input responsive while results update.
- **Tab containers:** Switching tabs without blocking navigation.
- **Form submissions:** Showing pending states without blocking the form.

### References

- Concurrent React Features (Claude Dev Suite) — https://github.com/claude-dev-suite/...
- React-Claude-Skill-Package (Concurrent) — https://github.com/Impertio-Studio/React-Claude-Skill-Package/.../SKILL.md

---

## 7. `useDeferredValue`

### Definitions

**Core Definition:** `useDeferredValue` is a React Hook that returns a deferred version of a value, allowing expensive UI updates to lag behind urgent updates.

**Technical Definition:** `const deferredValue = useDeferredValue(value)` returns a version of `value` that “lags behind” during updates. On the initial render, the deferred value equals the provided value. During updates, React first re-renders with the old deferred value, then schedules a background re-render with the new value. The background re-render is interruptible. `useDeferredValue` is integrated with `Suspense`: if the background update suspends, the user continues to see the old deferred value rather than a fallback.

**Beginner-Friendly Explanation:** `useDeferredValue` lets you say: “Use the latest value for the urgent part of the UI, but use the old value for the expensive part until React has time to update it.” It is useful when you do not control the state setter.

### Purposes

- To defer updating an expensive part of the UI.
- To keep high-priority updates (like typing) responsive.
- To show stale content while fresh content renders in the background.
- To avoid Suspense fallbacks for background updates.

### Syntax Rules and Structure

```jsx
const deferredValue = useDeferredValue(value);
```

**Syntax Rules:**

1. Call `useDeferredValue` at the top level of a component.
2. The value can be of any type; primitives and objects created outside rendering are safest.
3. On initial render, the deferred value equals the provided value.
4. During updates, the deferred value lags behind and catches up in a background render.

**Constraints and Limitations:**

- Creating a new object during render and passing it to `useDeferredValue` causes unnecessary background re-renders.
- `useDeferredValue` does not prevent network requests; it only defers rendering.
- It is not a substitute for memoization when the expensive component does not re-render with the deferred value.

### Annotated Code Examples

```jsx
import { useState, useDeferredValue, useMemo } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const results = useMemo(
    () => expensiveSearch(deferredQuery),
    [deferredQuery]
  );

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ResultsList results={results} />
    </div>
  );
}
```

**Expected Output:** The input updates immediately. The results list updates after the deferred value catches up, keeping typing smooth.

**Why this output:** `deferredQuery` lags behind `query`. The expensive search uses `deferredQuery` and is memoized, so it only recomputes when the deferred value changes — not on every keystroke.

### Real-World Cases

- **Charts and data visualisation:** Deferring expensive chart re-renders while controls remain responsive.
- **Search results:** Showing stale results while new results render in the background.
- **Large lists:** Deferring rendering of off-screen or expensive list sections.

### References

- Concurrent Rendering (Bobmatnyc) — https://github.com/bobmatnyc/claude-mpm-skills/.../concurrent-rendering.md
- Concurrent React and Scheduling (Steve Kinney) — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 8. Streaming SSR

### Definitions

**Core Definition:** Streaming SSR is a server-rendering technique that sends HTML to the browser in chunks as they become ready, rather than waiting for the entire page to render.

**Technical Definition:** Traditional SSR renders the full tree on the server and sends one HTML response. Streaming SSR uses `renderToPipeableStream` (Node) or `renderToReadableStream` (Web Streams) to send the document shell first, then stream in `Suspense` boundary content as it resolves. This improves Time to First Byte (TTFB) and allows the browser to start parsing and rendering while slower sections are still being prepared. Streaming SSR is built on concurrent rendering and is paired with selective hydration.

**Beginner-Friendly Explanation:** Streaming SSR is like a restaurant serving courses as they are ready instead of waiting for the entire meal to be cooked. The browser receives the page shell immediately, and slower sections arrive later without blocking the rest of the page.

### Purposes

- To improve TTFB by sending the shell immediately.
- To let users see and interact with fast content while slow content streams in.
- To prevent slow data dependencies from blocking the entire page.
- To enable selective hydration of interactive components before the full page has loaded.

### Syntax Rules and Structure

**Server (Node):**

```js
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/client.js'],
  onShellReady() {
    res.setHeader('Content-Type', 'text/html');
    pipe(res);
  },
});
```

**Client:**

```js
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document, <App />);
```

**Syntax Rules:**

1. Use `renderToPipeableStream` (Node) or `renderToReadableStream` (edge).
2. Start piping in `onShellReady` so the shell reaches the browser immediately.
3. Wrap slow sections in `Suspense` to stream them separately.
4. Use `hydrateRoot` on the client to enable selective hydration.

**Constraints and Limitations:**

- Requires a server runtime that supports streaming.
- Framework support (Next.js, etc.) is often required for Suspense-enabled data fetching.

### Annotated Code Examples

```jsx
// Server streaming setup
import { renderToPipeableStream } from 'react-dom/server';

function handleRequest(req, res) {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/client.js'],
    onShellReady() {
      res.setHeader('Content-Type', 'text/html');
      pipe(res);
    },
  });
}
```

```jsx
// App with Suspense boundaries
function App() {
  return (
    <div>
      <Header />
      <Suspense fallback={<UserSkeleton />}>
        <UserProfile />
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <PostList />
      </Suspense>
    </div>
  );
}
```

**Expected Output:** The shell (header) is sent immediately. `UserProfile` and `PostList` stream in independently as their data resolves.

**Why this output:** Each `Suspense` boundary is a streaming chunk. React sends the shell first, then streams each boundary's HTML when its content is ready.

### Real-World Cases

- **E-commerce product pages:** Streaming recommendations and reviews after the main product info.
- **Dashboards:** Sending the shell and streaming widgets as their data loads.
- **Content sites:** Streaming comments and related articles after the article body.

### References

- React Rendering Architecture: Fiber, Lanes, Streaming SSR, RSC — https://github.com/... (streaming SSR with Suspense)
- Selective Hydration (oakoss/agent-skills) — https://github.com/oakoss/agent-skills/.../selective-hydration.md

---

## 9. Selective Hydration

### Definitions

**Core Definition:** Selective hydration is React's ability to hydrate different parts of a server-rendered page independently, prioritising the parts the user interacts with first.

**Technical Definition:** In traditional SSR, React hydrates the entire tree once all JavaScript has loaded. Selective hydration, enabled by `hydrateRoot` and `Suspense` boundaries, allows React to hydrate independently hydratable chunks as their code arrives. If the user interacts with a part of the page that has not yet hydrated, React prioritises hydrating that part first. This makes the page interactive sooner and improves Interaction to Next Paint (INP).

**Beginner-Friendly Explanation:** Selective hydration means React does not wait for every piece of JavaScript to load before making the page interactive. It hydrates the parts that are ready, and if you click on a part that is not ready yet, React rushes to hydrate that part first.

### Purposes

- To make parts of an SSR page interactive before all JavaScript has loaded.
- To prioritise hydration of components the user interacts with.
- To prevent slow components from blocking the hydration of the entire page.
- To improve INP on server-rendered pages.

### Syntax Rules and Structure

```jsx
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document, <App />);
```

**Syntax Rules:**

1. Use `hydrateRoot` (React 18+) instead of `hydrate`.
2. Wrap independently hydratable sections in `Suspense` boundaries.
3. Do not place critical interactive components inside long-lived loading fallbacks.
4. Use streaming SSR APIs (`renderToPipeableStream`) on the server.

**Constraints and Limitations:**

- Requires `hydrateRoot` and `Suspense` boundaries.
- Slow components inside `Suspense` can still delay their own hydration, but they no longer block siblings.

### Annotated Code Examples

```jsx
// Client entry
import { hydrateRoot } from 'react-dom/client';
import App from './App';

hydrateRoot(document, <App />);
```

```jsx
// App with independently hydratable sections
function App() {
  return (
    <div>
      <Header /> {/* Hydrates immediately */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments /> {/* Hydrates when its code/data is ready */}
      </Suspense>
      <Footer /> {/* Hydrates immediately */}
    </div>
  );
}
```

**Expected Output:** `Header` and `Footer` hydrate as soon as their code is available. `Comments` hydrates later, without blocking the rest of the page.

**Why this output:** `hydrateRoot` enables selective hydration. Each `Suspense` boundary is an independently hydratable chunk. React prioritises the boundary the user interacts with.

### Real-World Cases

- **E-commerce:** Hydrating the “Add to Cart” button before the reviews section.
- **Dashboards:** Making filters interactive before heavy charts hydrate.
- **Content sites:** Hydrating navigation and search before comments.

### References

- Selective Hydration (oakoss/agent-skills) — https://github.com/oakoss/agent-skills/.../selective-hydration.md
- Selective Hydration (PatternsDev/skills) — https://github.com/PatternsDev/skills/.../react-selective-hydration/SKILL.md

---

## 10. Time Slicing

### Definitions

**Core Definition:** Time slicing is the technique of splitting rendering work into small time windows so React can yield to the browser between windows and keep the UI responsive.

**Technical Definition:** Fiber processes the work-in-progress tree one Fiber at a time. After each unit of work, React evaluates whether it still has time in the current frame (approximately 5ms windows) or should yield to the browser. This mechanism underpins time slicing and concurrent rendering, allowing React to maintain UI responsiveness even during large or expensive renders.

**Beginner-Friendly Explanation:** Time slicing means React works in short bursts. After each burst, it checks if the browser needs to do something else, like respond to a click or draw a frame.

### Purposes

- To prevent long tasks from blocking the main thread.
- To keep the UI responsive during heavy renders.
- To enable concurrent rendering.
- To allow urgent work to interrupt background work.

### Syntax Rules and Structure

**Time Slicing Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| Window size | ~5ms per work slice |
| Yield check | After each Fiber unit |
| Resumption | Work continues when the browser is free |

**Syntax Rules:**

1. React processes work in ~5ms windows.
2. After each window, React yields to the browser.
3. Work resumes when the browser is idle or urgent work is complete.
4. The granularity of the component tree affects yield flexibility.

### Annotated Code Examples

```jsx
// Conceptual: time slicing yields after each unit
function workLoop(deadline) {
  while (nextUnitOfWork && deadline.timeRemaining() > 1) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
  }
  if (nextUnitOfWork) {
    scheduleCallback(workLoop); // Resume later
  }
}
```

**Expected Output:** Work is processed in small slices, yielding to the browser between slices.

**Why this output:** The work loop checks remaining time after each Fiber unit and yields if the frame budget is exhausted.

### Real-World Cases

- **Large lists:** Time slicing keeps scrolling smooth while items render.
- **Complex dashboards:** Heavy charts do not block input.
- **Animations:** Yielding keeps animation frames on schedule.

### References

- Concurrent Rendering and Automatic Batching (Educative) — https://www.educative.io
- Virtual DOM & Fiber Architecture (Steve Kinney) — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture

---

## 11. Priority Scheduling

### Definitions

**Core Definition:** Priority scheduling is React's mechanism for assigning every update a priority level and processing the highest-priority work first.

**Technical Definition:** React's concurrent scheduler uses a priority hierarchy: ImmediatePriority (user input, focus events), UserBlockingPriority (click handlers, form inputs), NormalPriority (data fetching), LowPriority (analytics, logging), and IdlePriority (background cleanup). The scheduler can interrupt lower-priority work when higher-priority work arrives, ensuring the app stays responsive to user interactions. React 18 replaced scheduler priorities with the lane model — a 31-bit bitmask that allows fine-grained concurrent prioritisation.

**Beginner-Friendly Explanation:** Priority scheduling is React's way of deciding what to work on first. Urgent things like typing and clicking get priority; background things like analytics and data visualisation wait their turn.

### Purposes

- To process urgent updates before non-urgent ones.
- To enable concurrent rendering features.
- To keep the UI responsive under heavy load.
- To provide predictable scheduling behaviour.

### Syntax Rules and Structure

**Priority Hierarchy:**

| Priority | Typical Triggers |
|----------|------------------|
| Immediate | User input, focus events |
| User Blocking | Click handlers, form inputs |
| Normal | Data fetching, network requests |
| Low | Analytics, logging |
| Idle | Background cleanup tasks |

**Syntax Rules:**

1. Every update enters the scheduler with a priority.
2. The highest-priority work is processed first.
3. High-priority tasks can interrupt low-priority tasks.
4. React 18+ uses lanes (bitmasks) for fine-grained prioritisation.

### Annotated Code Examples

```jsx
// Conceptual: priority levels (internal to React)
enum Priority {
  ImmediatePriority = 1,    // User input, focus
  UserBlockingPriority = 2, // Click, form input
  NormalPriority = 3,       // Data fetching
  LowPriority = 4,          // Analytics
  IdlePriority = 5,         // Background cleanup
}
```

**Expected Output:** React processes higher-priority work first.

**Why this output:** The scheduler walks the highest-priority bit set first, enabling urgent work to preempt background work.

### Real-World Cases

- **Typing:** Input updates are UserBlocking priority and processed immediately.
- **Transitions:** `startTransition` marks updates as lower priority.
- **Data fetching:** Normal or Low priority, deferred until urgent work is done.

### References

- Concurrent React and Scheduling (Steve Kinney) — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- Concurrent Rendering and Automatic Batching (Educative) — https://www.educative.io

---

## 12. Interruptible Rendering

### Definitions

**Core Definition:** Interruptible rendering is React's ability to pause, resume, or abandon rendering work when higher-priority updates arrive.

**Technical Definition:** Because Fiber represents work as heap-allocated Fiber nodes rather than call-stack frames, React can pause rendering at Fiber boundaries, switch to a higher-priority subtree, and later resume or discard the interrupted work. This is the core capability that distinguishes concurrent rendering from the legacy stack reconciler. Concurrent features such as `useTransition`, `useDeferredValue`, and `Suspense` use interruptible rendering to keep the UI responsive.

**Beginner-Friendly Explanation:** Interruptible rendering means React can stop what it is doing, handle something urgent, and come back later. It is like being able to pause a video game when someone calls your name.

### Purposes

- To keep the UI responsive during heavy rendering.
- To allow urgent updates to preempt background work.
- To enable concurrent features.
- To discard stale work and restart with the latest state.

### Syntax Rules and Structure

**Interruptible Rendering Behaviour:**

| Scenario | React Behaviour |
|----------|-----------------|
| Urgent update arrives | Pause current render, switch to urgent work |
| Urgent work completes | Resume or restart interrupted work |
| Work becomes stale | Abandon the interrupted render |
| Commit phase | Cannot be interrupted (atomic) |

**Syntax Rules:**

1. Rendering can be paused at Fiber boundaries.
2. Urgent updates can preempt lower-priority work.
3. Interrupted work can be resumed or discarded.
4. The commit phase remains atomic.

### Annotated Code Examples

```jsx
// Conceptual: interruptible rendering (internal)
function renderRoot(root, lanes) {
  while (workInProgress !== null) {
    if (shouldYield()) {
      return; // Pause and yield to browser
    }
    workInProgress = performUnitOfWork(workInProgress);
  }
  commitRoot(root); // Atomic commit
}
```

**Expected Output:** Rendering pauses at Fiber boundaries and resumes later.

**Why this output:** The work loop checks `shouldYield()` and returns to the browser, preserving the work-in-progress tree for resumption.

### Real-World Cases

- **Search autocomplete:** Typing interrupts result rendering.
- **Tab navigation:** Clicking a tab preempts heavy tab content rendering.
- **Animations:** Rendering yields to keep animations smooth.

### References

- Virtual DOM & Fiber Architecture (Steve Kinney) — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture
- Concurrent React and Scheduling (Steve Kinney) — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 13. Concurrent Rendering Best Practices

### Definitions

**Core Definition:** Concurrent rendering best practices are the guidelines for writing components that cooperate with React's interruptible, priority-driven scheduler rather than fighting it.

**Technical Definition:** Best practices combine render purity (no side effects, mutations, or external reads during render), correct cleanup (cancelling requests, timers, and subscriptions in effects), appropriate use of transitions and deferred values (only for genuinely non-urgent work), and Suspense boundary placement (wrapping slow sections without trapping critical interactions). In React 19, concurrent rendering is the default, so these practices are correctness requirements, not optional enhancements.

**Beginner-Friendly Explanation:** Concurrent React can pause and restart your components. That means your components must be predictable: they should not do anything with side effects while rendering, and they must clean up after themselves. Use `useTransition` and `useDeferredValue` only when an update can genuinely wait, and place `Suspense` boundaries around slow sections, not around critical buttons.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Keep render pure** | No mutations, subscriptions, analytics, or external state reads during render. |
| **Put side effects in effects** | DOM work, analytics, and subscriptions belong after commit. |
| **Never use transitions for text inputs** | Transitions are interruptible and will cause input lag. |
| **Re-wrap post-await updates** | State updates after `await` must be wrapped in another `startTransition`. |
| **Pair `useDeferredValue` with `memo`** | Without memoization on the receiving component, deferral is defeated. |
| **Don't wrap urgent updates in transitions** | Button clicks and form submissions should remain urgent. |
| **Use nested Suspense boundaries** | Create progressive loading experiences for independent sections. |
| **Use `startTransition` for navigation** | Prevents hiding already-visible content behind a fallback. |
| **Remember: concurrency reorders, it does not reduce** | Combine with `React.memo`, `useMemo`, and `useCallback` to make work cheaper. |
| **Profile before and after** | Verify that concurrent features actually improve responsiveness for your use case. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
import { useState, useDeferredValue, memo, useMemo } from 'react';

const SearchResults = memo(function SearchResults({ query }) {
  const results = useMemo(
    () => expensiveSearch(query),
    [query]
  );
  return <ResultsList results={results} />;
});

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SearchResults query={deferredQuery} />
      </div>
    </>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ Transition for controlled input: causes input lag
startTransition(() => setQuery(value));

// ❌ Deferred value without memo: deferral is defeated
const deferredQuery = useDeferredValue(query);
<SearchResults query={deferredQuery} />; // SearchResults not memoized

// ❌ Post-await update outside transition
startTransition(async () => {
  const data = await fetchData();
  setResult(data); // Escapes the transition!
});

// ❌ Urgent update wrapped in transition
startTransition(() => setTab(nextTab)); // Tab click should be urgent
```

**Expected Output:** The good practices keep the input responsive, effectively defer the expensive results, and preserve transition boundaries. The bad practices cause input lag, defeat deferral, or lose transition context.

**Why this output:** Transitions are interruptible and must not be used for controlled inputs. Deferred values require memoization to be effective. Post-`await` updates escape the transition scope. Urgent updates must remain outside transitions.

### Real-World Cases

- **All React 19 applications:** Concurrent rendering is the default, so these practices apply universally.
- **Search interfaces:** Pair `useDeferredValue` with `memo` for effective deferral.
- **Navigation:** Use `startTransition` for route changes that suspend.
- **Async actions:** Re-wrap post-`await` state updates in `startTransition`.

### References

- React-Claude-Skill-Package (Concurrent) — https://github.com/Impertio-Studio/React-Claude-Skill-Package/.../SKILL.md
- Concurrent React and Scheduling (Steve Kinney) — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- Concurrent Rendering (Bobmatnyc) — https://github.com/bobmatnyc/claude-mpm-skills/.../concurrent-rendering.md
- React 19 Concurrent Patterns — https://github.com/github/awesome-copilot/.../react19-concurrent-patterns

---

## References

- React Concurrent Features (Claude Dev Suite) — https://github.com/claude-dev-suite/claude-dev-suite/blob/133ba99731744ee6c84fd892181471580f84d315/skills/frontend-frameworks/react-concurrent/SKILL.md
- Concurrent Rendering (Bobmatnyc Skills) — https://github.com/bobmatnyc/claude-mpm-skills/blob/718070a7d622921b01687799a1f9613f36c6f615/toolchains/javascript/frameworks/react/react-advanced/references/concurrent-rendering.md
- Concurrent Features (React 18+) (Syngrisi) — https://github.com/syngrisi/syngrisi/blob/7f44c5a24cbb451b75a7ff25a1b16e5acf95162f/.agents/skills/react-performance-optimization/references/concurrent-features.md
- Virtual DOM & Fiber Architecture (Steve Kinney) — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture
- Concurrent React and Scheduling (Steve Kinney) — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- Concurrent React (TypeScript Cheatsheets) — https://github.com/typescript-cheatsheets/react/blob/c550e41b98f7ec20d993eb3c2acc4cbf6eac2e29/docs/basic/getting-started/concurrent.md
- React-Claude-Skill-Package (Concurrent) — https://github.com/Impertio-Studio/React-Claude-Skill-Package/blob/main/skills/source/react-core/react-core-concurrent/SKILL.md
- React 19 Concurrent Patterns (awesome-copilot) — https://github.com/github/awesome-copilot/blob/e986f49695491311df2774030ebe11efabd0fb77/skills/react19-concurrent-patterns/SKILL.md
- Suspense, Error Boundaries, and Concurrent Features (DeepWiki) — https://deepwiki.com/facebook/react
- React Rendering Architecture: Fiber, Lanes, Streaming SSR, RSC — https://github.com/...
- Selective Hydration (oakoss/agent-skills) — https://github.com/oakoss/agent-skills/blob/main/skills/hydration-guardian/references/selective-hydration.md
- Selective Hydration (PatternsDev/skills) — https://github.com/PatternsDev/skills/blob/main/skills/react/react-selective-hydration/SKILL.md
- A deep dive into React Fiber (LogRocket, updated 2026) — https://blog.logrocket.com/deep-dive-react-fiber/
- Concurrent Rendering and Automatic Batching (Educative) — https://www.educative.io
- Modern React (18–19.2) + Compiler 1.0 Reference — https://github.com/...
- React 19 Async Rendering Changes (OpenReplay) — https://blog.openreplay.com
- Feature-Sliced Design — Progressive Hydration — https://feature-sliced.design