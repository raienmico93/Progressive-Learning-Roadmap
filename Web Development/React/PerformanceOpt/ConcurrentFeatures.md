# React Concurrent Features — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrent features are a set of React capabilities that allow the framework to prepare multiple versions of the user interface simultaneously, interrupt and resume rendering work, and prioritise urgent updates over non-urgent ones — all to keep the application responsive during expensive operations.

**Technical Definition:** Concurrent rendering is a behind-the-scenes mechanism introduced in React 18 that enables React to prepare multiple versions of the UI at the same time. Rendering is interruptible: React may start rendering an update, pause in the middle, abandon it, or continue later. Concurrent features — `useTransition`, `useDeferredValue`, `Suspense` with streaming SSR, and selective hydration — are built on this foundation. In React 18, concurrent rendering was opt-in, enabled only for updates triggered by these features. In React 19, concurrent rendering is the default scheduling behaviour for all components, with the scheduler slicing rendering work into approximately 5ms chunks and yielding to the browser between them.

**Beginner-Friendly Explanation:** Imagine you are cooking several dishes at once. Before concurrent features, React would finish one dish completely before starting the next — even if a pot was boiling over. With concurrent features, React can pause a slow dish, attend to something urgent (like a user clicking a button), and come back to the slow dish later. This means the app stays responsive even when React is doing heavy work in the background. The features described in this cheat sheet are the tools that let you tell React which work is urgent and which can wait.

### Key Characteristics

- **Interruptible rendering:** React can pause, resume, or abandon rendering work.
- **Priority-based scheduling:** Urgent updates (typing, clicking) preempt non-urgent updates (filtering, charting).
- **Default in React 19:** Concurrent rendering is the default scheduling behaviour, not an opt-in mode.
- **Cooperative multitasking:** Rendering work is sliced into small chunks with yield points, keeping the main thread available for input.
- **Atomic commits:** While rendering is interruptible, the commit phase remains synchronous and atomic.
- **Opt-in API surface:** `useTransition`, `useDeferredValue`, and `Suspense` are the developer-facing entry points.

### Prerequisites

- React function components, state, and `useEffect`.
- Understanding of the render → reconcile → commit pipeline.
- Familiarity with JavaScript Promises and asynchronous programming.
- Basic knowledge of server-side rendering and hydration concepts.

### Related Programming Areas

- **Rendering Performance** — long tasks and main-thread blocking.
- **Code Splitting** — `Suspense` and `React.lazy` for lazy loading.
- **Streaming SSR and Selective Hydration** — server-rendering improvements built on concurrent features.
- **Core Web Vitals** — INP is the primary metric that concurrent features improve.

---

## 1. Concurrent Rendering

### Definitions

**Core Definition:** Concurrent rendering is React's interruptible rendering model that allows the framework to prepare multiple UI versions simultaneously and prioritise urgent updates.

**Technical Definition:** Concurrent rendering is a new behind-the-scenes mechanism that enables React to prepare multiple versions of the UI at the same time. A key property is that rendering is interruptible: React may start rendering an update, pause in the middle, then continue later. It can even abandon an in-progress render entirely. Concurrent rendering uses a cooperative scheduling model with time-slicing and a priority hierarchy (Immediate → UserBlocking → Normal → Low → Idle). The commit phase remains atomic — React only commits when the render is complete and consistent.

**Beginner-Friendly Explanation:** Concurrent rendering is React's ability to multitask. Instead of freezing while it finishes a slow render, React can pause and let the browser respond to the user, then resume where it left off. It is like reading a book and being able to put it down when someone calls your name, then picking it up again.

### Purposes

- To keep the UI responsive during expensive renders.
- To prioritise urgent updates (typing, clicking) over non-urgent ones (data visualisation, filtering).
- To prevent long tasks from blocking the main thread and degrading Interaction to Next Paint (INP).
- To enable features like `Suspense`, transitions, streaming SSR, and selective hydration.

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

1. Concurrent rendering is the default in React 19; no opt-in flag is required.
2. Rendering work is sliced into ~5ms chunks with yield points.
3. The commit phase is synchronous and atomic; only the render phase is interruptible.
4. Components must remain pure — React may pause, abandon, and restart a render.

**Constraints and Limitations:**

- Rendering must be pure; side effects during render can double-fire under concurrent scheduling.
- Refs must not be read or written during render; their values may not be stable across interrupted renders.
- Concurrent rendering reorders work; it does not make individual work cheaper.

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
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Searching…</p>}
      <ResultsList results={results} />
    </div>
  );
}
```

**Expected Output:** The input remains responsive while the search runs in the background. `isPending` indicates background work.

**Why this output:** `startTransition` marks the search update as non-urgent. The concurrent scheduler can interrupt it to process the urgent input update.

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

**Why this output:** React may pause, abandon, and restart a render. Anything observable from the outside belongs in an effect or ref, never in render.

### Real-World Cases

- **Search autocomplete:** Typing must remain responsive while results are computed.
- **Tab switching:** Navigating between heavy tabs should not freeze the input.
- **Data dashboards:** Charts and tables can render in the background while filters remain interactive.

### References

- React Blog — The Plan for React 18 — https://ar.react.dev/blog/2021/06/08/the-plan-for-react-18
- React Blog — React v18.0 — https://vi.legacy.reactjs.org/blog/2022/03/29/react-v18.html
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- React 19 concurrent mode and product tours — https://raw.githubusercontent.com/domidex01/tour-kit
- Keep render pure — never mutate, subscribe, or read external state during render — https://github.com/pproenca/dot-skills

---

## 2. Transitions

### Definitions

**Core Definition:** A transition is a state update that React treats as non-urgent, allowing it to be interrupted by more important updates.

**Technical Definition:** A transition marks an update as non-urgent, allowing React to interrupt it for more important work. Transitions let you defer non-urgent state updates, preventing them from blocking the UI thread while users are actively typing, clicking, or scrolling. Transitions are created with `startTransition` (standalone or from `useTransition`). State updates inside a transition are rendered in the background; if a more urgent update arrives, React abandons the background render and restarts it with the latest state.

**Beginner-Friendly Explanation:** A transition is a way of telling React: “This update is not urgent. If the user does something more important, drop this and handle that first.” It is how you keep typing smooth while a heavy filter or chart update happens in the background.

### Purposes

- To mark expensive state updates as non-urgent.
- To keep the UI responsive during background rendering.
- To prevent non-urgent updates from blocking user input.
- To avoid unwanted loading indicators for non-urgent updates.

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
4. Any state update after an `await` inside an async transition must be wrapped in another `startTransition` (React 19 supports async actions more directly).

**Constraints and Limitations:**

- You can only wrap updates if you have access to the `set` function of the state.
- State updates in `setTimeout` are not marked as transitions.
- Transitions reorder work; they do not reduce the cost of the work itself.

### Annotated Code Examples

```jsx
import { useState, useTransition } from 'react';

function TabContainer() {
  const [tab, setTab] = useState('about');
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <div>
      <TabButton onClick={() => selectTab('about')}>About</TabButton>
      <TabButton onClick={() => selectTab('posts')}>Posts</TabButton>
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

- React — useTransition — https://react.dev/reference/react/useTransition
- Steve Kinney — useTransition and startTransition — https://github.com/stevekinney/stevekinney.net
- React 19 concurrent mode and product tours — https://raw.githubusercontent.com/domidex01/tour-kit

---

## 3. `useTransition`

### Definitions

**Core Definition:** `useTransition` is a React Hook that returns a pending flag and a function to mark state updates as non-urgent transitions.

**Technical Definition:** `const [isPending, startTransition] = useTransition()` returns a tuple. `isPending` is a boolean indicating whether a transition is in progress. `startTransition` is a function that accepts an action (a synchronous function that calls state setters) and marks all state updates scheduled synchronously during that action as transitions. The action is called immediately. Transitions are non-blocking and do not display unwanted loading indicators.

**Beginner-Friendly Explanation:** `useTransition` gives you two things: a way to start a background update (`startTransition`) and a flag (`isPending`) that tells you whether that background update is still running. You use it when you want to keep the UI responsive during expensive state changes.

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
3. State updates after an `await` must be wrapped in another `startTransition` (unless using React 19 async actions).
4. `isPending` can be used to disable buttons or show subtle pending indicators.

**Constraints and Limitations:**

- Cannot be used to wrap updates whose `set` function you do not control.
- If you need to defer a value you do not control, use `useDeferredValue` instead.
- Transitions do not make work cheaper; they only reorder it.

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

- React — useTransition — https://react.dev/reference/react/useTransition
- Vercel — Use useTransition instead of manual loading states — https://github.com/vercel-labs/agent-skills
- Steve Kinney — useTransition and startTransition — https://github.com/stevekinney/stevekinney.net

---

## 4. `useDeferredValue`

### Definitions

**Core Definition:** `useDeferredValue` is a React Hook that returns a deferred version of a value, allowing expensive UI updates to lag behind urgent updates.

**Technical Definition:** `const deferredValue = useDeferredValue(value)` returns a version of `value` that “lags behind” during updates. On the initial render, the deferred value equals the provided value. During updates, React first re-renders with the old deferred value, then schedules a background re-render with the new value. The background re-render is interruptible. `useDeferredValue` is integrated with `Suspense`: if the background update suspends, the user continues to see the old deferred value rather than a fallback.

**Beginner-Friendly Explanation:** `useDeferredValue` lets you say: “Use the latest value for the urgent part of the UI, but use the old value for the expensive part until you have time to update it.” It is useful when you do not control the state update but want to defer the expensive rendering that depends on it.

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

**Why this output:** `deferredQuery` lags behind `query`. The expensive search uses `deferredQuery`, so it runs after urgent input updates have been processed.

### Real-World Cases

- **Charts and data visualisation:** Deferring expensive chart re-renders while controls remain responsive.
- **Search results:** Showing stale results while new results render in the background.
- **Large lists:** Deferring rendering of off-screen or expensive list sections.

### References

- React — useDeferredValue — https://18.react.dev/reference/react/useDeferredValue
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns
- Vercel — Use useDeferredValue for expensive computations — https://github.com/arc-reel/agent-skills

---

## 5. Suspense

### Definitions

**Core Definition:** `Suspense` is a React component that displays a fallback UI while its children are loading, and it serves as the boundary for streaming SSR and selective hydration.

**Technical Definition:** `<Suspense fallback={...}>` catches suspended rendering in its subtree. When used with `React.lazy`, it shows the fallback while a lazy component's code loads. When used with Suspense-enabled data sources, it shows the fallback while data is fetched. `Suspense` is integrated with streaming server rendering and selective hydration: each `Suspense` boundary becomes an independently streamable and hydratable chunk. If a visible tree suspends again, the fallback is shown unless the update was caused by `startTransition` or `useDeferredValue`.

**Beginner-Friendly Explanation:** `Suspense` is the loading boundary. You wrap a part of the UI in `Suspense` and give it a fallback (a spinner or skeleton). If that part is not ready — because its code or data is still loading — React shows the fallback. When the part is ready, React replaces the fallback with the real content. In server rendering, `Suspense` lets React send the page in pieces, so the user does not have to wait for the slowest part.

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

1. `Suspense` requires a `fallback` prop.
2. Children can be any React node; if they suspend, the fallback is rendered.
3. `Suspense` boundaries can be nested; the nearest boundary handles the suspension.
4. For SSR, `Suspense` boundaries define chunks that stream independently.
5. Only Suspense-enabled data sources (frameworks, `use`, `lazy`) can trigger `Suspense`; fetching inside `useEffect` does not.

**Constraints and Limitations:**

- State is not preserved for a tree that suspends before its first mount.
- If a visible tree suspends again, the fallback is shown unless the update was a transition or deferred value.
- React does not yet support Suspense-enabled data fetching without an opinionated framework.

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

**Expected Output:** The artist name renders immediately. The albums section shows “Loading albums…” until the data or code is ready, then renders the albums.

**Why this output:** `Albums` suspends while its data loads. The nearest `Suspense` boundary renders the fallback, while the rest of the page remains visible.

### Real-World Cases

- **Lazy-loaded routes:** Showing a page skeleton while the route chunk loads.
- **Slow data sections:** Streaming in comments or recommendations after the main content.
- **Modals and editors:** Deferring heavy component code until the user opens them.

### References

- React — Suspense — https://zh-hans.react.dev/reference/react/Suspense
- React — Suspense (React 18) — https://18.react.dev/reference/react/Suspense
- React Blog — React v18.0 — https://uk.react.dev/blog/2022/03/29/react-v18

---

## 6. Streaming SSR

### Definitions

**Core Definition:** Streaming SSR is a server-rendering technique that sends HTML to the browser in chunks as they become ready, rather than waiting for the entire page to render.

**Technical Definition:** Traditional SSR renders the full tree on the server and sends one HTML response. Streaming SSR uses `renderToPipeableStream` (Node) or `renderToReadableStream` (Web Streams) to send the document shell first, then stream in `Suspense` boundary content as it resolves. This dramatically improves Time to First Byte (TTFB) and allows the browser to start parsing and rendering while slower sections are still being prepared. Streaming SSR is built on concurrent rendering and is paired with selective hydration.

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
- Errors during streaming must be handled with `onError`.
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
    onError(err) {
      console.error(err);
      res.statusCode = 500;
      res.end('Server error');
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
        <UserProfile /> {/* Streams when ready */}
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <PostList /> {/* Streams when ready */}
      </Suspense>
    </div>
  );
}
```

**Expected Output:** The shell (header) is sent immediately. `UserProfile` and `PostList` stream in independently as their data resolves. The browser can render and hydrate the shell before the slow sections arrive.

**Why this output:** Each `Suspense` boundary is a streaming chunk. React sends the shell first, then streams each boundary's HTML when its content is ready.

### Real-World Cases

- **E-commerce product pages:** Streaming recommendations and reviews after the main product info.
- **Dashboards:** Sending the shell and streaming widgets as their data loads.
- **Content sites:** Streaming comments and related articles after the article body.

### References

- Steve Kinney — Streaming SSR with TypeScript — https://stevekinney.com/courses/react-typescript/streaming-ssr-typescript
- PatternsDev — Streaming SSR — https://github.com/PatternsDev/skills
- React — Server Rendering Overview — https://mintlify.wiki/facebook/react
- renderToPipeableStream — https://mintlify.wiki/facebook/react

---

## 7. Selective Hydration

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
- Framework support may be needed for the best integration.

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

**Expected Output:** `Header` and `Footer` hydrate as soon as their code is available. `Comments` hydrates later, without blocking the rest of the page. If the user clicks on `Comments` before it hydrates, React prioritises hydrating it.

**Why this output:** `hydrateRoot` enables selective hydration. Each `Suspense` boundary is an independently hydratable chunk. React prioritises the boundary the user interacts with.

### Real-World Cases

- **E-commerce:** Hydrating the “Add to Cart” button before the reviews section.
- **Dashboards:** Making filters interactive before heavy charts hydrate.
- **Content sites:** Hydrating navigation and search before comments.

### References

- PatternsDev — Selective Hydration — https://github.com/PatternsDev/skills
- Steve Kinney — Selective Hydration in Modern React Apps — https://stevekinney.com
- oakoss/agent-skills — Selective Hydration Reference — https://github.com/oakoss/agent-skills
- React — hydrateRoot — https://react.dev/reference/react-dom/client/hydrateRoot

---

## 8. Concurrent Features Best Practices

### Definitions

**Core Definition:** Concurrent features best practices are the guidelines for writing components that cooperate with React's interruptible, priority-based scheduler rather than fighting it.

**Technical Definition:** Best practices combine **render purity** (no side effects, mutations, subscriptions, or external state reads during render), **correct cleanup** (cancelling requests, timers, and subscriptions in effects), **appropriate use of transitions and deferred values** (only when the update is genuinely non-urgent), and **Suspense boundary placement** (wrapping slow sections without trapping critical interactions). In React 19, concurrent rendering is the default, so these practices are not optional enhancements — they are correctness requirements.

**Beginner-Friendly Explanation:** Concurrent React can pause and restart your components. That means your components must be predictable: they should not do anything with side effects while rendering, and they must clean up after themselves. Use `useTransition` and `useDeferredValue` only when an update can genuinely wait, and place `Suspense` boundaries around slow sections, not around critical buttons.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Keep render pure** | No mutations, subscriptions, analytics, or external state reads during render. |
| **Use effects for side effects** | Put logging, network calls, and subscriptions in `useEffect`. |
| **Clean up everything** | Cancel fetches, clear timers, remove listeners, unsubscribe. |
| **Use `startTransition` for non-urgent updates** | Keep urgent updates (typing, clicking) out of the transition. |
| **Use `useDeferredValue` when you do not control the setter** | Defer expensive rendering without changing state ownership. |
| **Do not read/write refs during render** | Refs are not stable across interrupted renders. |
| **Use `useId` for stable IDs** | Avoid counters or random IDs during render. |
| **Place `Suspense` around slow sections** | Do not trap critical interactive components in long-lived fallbacks. |
| **Combine with memoization** | `useTransition` and `useDeferredValue` reorder work; memoization reduces the work itself. |
| **Remember: concurrency reorders, it does not reduce** | Use `React.memo`, `useMemo`, and `useCallback` to make work cheaper. |

### Annotated Code Examples

**✅ Concurrent-Safe Component:**

```jsx
import { useId, useEffect, useState, useTransition } from 'react';

function UserCard({ user }) {
  const id = useId();
  const [isPending, startTransition] = useTransition();
  const [details, setDetails] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    fetch(`/api/users/${user.id}/details`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        startTransition(() => setDetails(data));
      })
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err);
      });
    return () => controller.abort();
  }, [user.id]);

  return (
    <div id={id}>
      <h2>{user.name}</h2>
      {isPending && <span>Loading details…</span>}
      {details && <UserDetails data={details} />}
    </div>
  );
}
```

**❌ Concurrent-Unsafe Patterns:**

```jsx
// ❌ Side effect during render — may double-fire
let counter = 0;
function BadCard({ user }) {
  const id = counter++;
  analytics.track('view', user.id);
  return <div id={`card-${id}`}>{user.name}</div>;
}

// ❌ Subscription during render — accumulates listeners
function BadSubscription() {
  window.addEventListener('resize', handler); // Never cleaned up
  return <div />;
}

// ❌ Ref read/write during render — unstable under interruption
function BadRef() {
  const ref = useRef(0);
  ref.current++; // Unpredictable under concurrent rendering
  return <div>{ref.current}</div>;
}
```

**Expected Output:** The safe component logs analytics once per commit, cancels its fetch on unmount, and uses stable IDs. The unsafe patterns produce double-fired analytics, leaked listeners, and unpredictable ref values.

**Why this output:** React may pause, abandon, and restart a render. Side effects in render are executed multiple times. Refs read or written during render may not reflect a consistent state across interrupted renders.

### Real-World Cases

- **All React 19 applications:** Concurrent rendering is the default, so these practices apply universally.
- **Analytics-heavy applications:** Double-fired analytics is a common concurrent-rendering bug.
- **Real-time dashboards:** Correct cleanup prevents subscription and request leaks under frequent re-renders.

### References

- pproenca/dot-skills — Keep render pure — https://github.com/pproenca/dot-skills
- React 19 concurrent mode and product tours — https://raw.githubusercontent.com/domidex01/tour-kit
- Vercel — useTransition and useDeferredValue best practices — https://github.com/vercel-labs/agent-skills
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## References

- React Blog — The Plan for React 18 — https://ar.react.dev/blog/2021/06/08/the-plan-for-react-18
- React Blog — React v18.0 — https://vi.legacy.reactjs.org/blog/2022/03/29/react-v18.html
- React Blog — React v19 — https://zh-hans.react.dev/blog/2024/12/05/react-19
- React — useTransition — https://react.dev/reference/react/useTransition
- React — useDeferredValue — https://18.react.dev/reference/react/useDeferredValue
- React — Suspense — https://zh-hans.react.dev/reference/react/Suspense
- React — hydrateRoot — https://react.dev/reference/react-dom/client/hydrateRoot
- React — Server Rendering Overview — https://mintlify.wiki/facebook/react
- React — renderToPipeableStream — https://mintlify.wiki/facebook/react
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- Steve Kinney — Streaming SSR with TypeScript — https://stevekinney.com/courses/react-typescript/streaming-ssr-typescript
- Steve Kinney — Selective Hydration in Modern React Apps — https://stevekinney.com
- PatternsDev — Selective Hydration — https://github.com/PatternsDev/skills
- PatternsDev — Streaming SSR — https://github.com/PatternsDev/skills
- oakoss/agent-skills — Selective Hydration Reference — https://github.com/oakoss/agent-skills
- pproenca/dot-skills — Keep render pure — https://github.com/pproenca/dot-skills
- Vercel — useTransition instead of manual loading states — https://github.com/vercel-labs/agent-skills
- Vercel — useDeferredValue for expensive computations — https://github.com/arc-reel/agent-skills
- React 19 concurrent mode and product tours — https://raw.githubusercontent.com/domidex01/tour-kit
- Steve Kinney — useTransition and startTransition — https://github.com/stevekinney/stevekinney.net
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns