# `useTransition` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useTransition` is a React Hook that lets you render a part of the UI in the background by marking certain state updates as non-urgent "transitions."

**Technical Definition:** `const [isPending, startTransition] = useTransition()` returns a tuple containing a pending flag and a function that marks state updates as transitions. State updates scheduled inside `startTransition` are assigned to a lower-priority lane in React's concurrent scheduler, making them interruptible by urgent updates (such as user input). In React 18, transitions primarily targeted synchronous, CPU-bound state updates. In React 19, `startTransition` natively supports async actions, and `isPending` remains `true` for the entire duration of an asynchronous operation, including awaits. The `startTransition` function has a stable identity across renders and does not return a value.

**Beginner-Friendly Explanation:** `useTransition` is a way to tell React: "This update is not urgent. If the user does something more important — like typing or clicking — drop this and handle that first." You use it when a state update would take a long time to render (like filtering a huge list or switching to a heavy tab). By wrapping that update in `startTransition`, you keep the interface responsive. React works on the slow update in the background and only shows it when it is ready. The `isPending` flag tells you when that background work is still in progress.

### Key Characteristics

- **Two-item return:** Returns `[isPending, startTransition]` — a pending boolean and a marking function.
- **Lower-priority lane:** Transitions are assigned to a lower-priority lane, making them interruptible by urgent updates.
- **Double render:** `useTransition` causes two renders — one immediate render with `isPending = true`, and one deferred render with the completed state and `isPending = false`.
- **Stable `startTransition`:** The `startTransition` function identity is stable across renders and can be safely omitted from effect dependency arrays.
- **Async support (React 19+):** `isPending` stays `true` for the full duration of an async action, including awaits.
- **Hook-only:** `useTransition` is a Hook and must be called at the top level of a component or custom Hook. For use outside components, use the standalone `startTransition` import.

### Prerequisites

- React function components, `useState`, and event handling.
- Understanding of React's render → reconcile → commit pipeline.
- Familiarity with the concept of urgent vs. non-urgent work.
- Basic knowledge of concurrent rendering and React 19 async actions.

### Related Programming Areas

- **Concurrent Rendering** — the scheduling foundation that makes transitions possible.
- **`useDeferredValue`** — the alternative when you do not control the state setter.
- **`useActionState`** — the React 19 hook for form actions with built-in pending state.
- **Core Web Vitals** — Interaction to Next Paint (INP) is the primary metric that transitions improve.

---

## 1. `useTransition`

### Definitions

**Core Definition:** `useTransition` is a React Hook that lets you render a part of the UI in the background by marking state updates as transitions.

**Technical Definition:** `useTransition()` takes no parameters and returns an array with exactly two items: `isPending` (a boolean indicating whether a transition is pending) and `startTransition` (a function that accepts an action and marks all state updates scheduled synchronously during that action as transitions). The `startTransition` function calls its action immediately, marking state updates as transitions. Any state updates scheduled after an `await` inside the action must be wrapped in an additional `startTransition` (a known limitation that React plans to address).

**Beginner-Friendly Explanation:** `useTransition` gives you two tools: a way to start a background update (`startTransition`) and a flag that tells you whether that background update is still running (`isPending`). You call it at the top of a component and use those two tools to keep the UI responsive.

### Purposes

- To mark expensive state updates as non-urgent transitions.
- To keep the UI responsive during heavy rendering work.
- To track whether a transition is pending with `isPending`.
- To prevent non-urgent updates from blocking urgent user input.
- To avoid showing unwanted loading fallbacks for non-urgent updates.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useTransition } from 'react';

const [isPending, startTransition] = useTransition();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useTransition()` | Hook call at the top level of a component or custom Hook. Takes no parameters. |
| `isPending` | Boolean: `true` while a transition is in progress. |
| `startTransition(action)` | Function: marks state updates in `action` as non-urgent transitions. |

**Syntax Rules:**

1. Call `useTransition` at the top level of a component or custom Hook; it cannot be called conditionally or inside loops.
2. `useTransition` takes no parameters.
3. `startTransition` calls its action immediately and marks synchronous state updates within it as transitions.
4. State updates after an `await` inside an async action must be wrapped in another `startTransition`.
5. `startTransition` has a stable identity and can be omitted from effect dependency arrays.

**Constraints and Limitations:**

- You can only wrap an update in a transition if you have access to the state's `set` function. If the value comes from props or an external store, use `useDeferredValue` instead.
- State updates in a `setTimeout` are not marked as transitions.
- `useTransition` causes two renders (immediate `isPending = true`, then deferred completed state).
- The action passed to `startTransition` is called immediately; it is the state updates that are deferred, not the function invocation.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Tab Switching

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
      <button onClick={() => selectTab('about')}>About</button>
      <button onClick={() => selectTab('posts')}>Posts</button>
      {isPending && <p>Loading tab…</p>}
      {tab === 'about' ? <AboutTab /> : <PostsTab />}
    </div>
  );
}
```

**Expected Output:** Clicking a tab updates the active tab in the background. The UI remains responsive, and "Loading tab…" appears while the transition is pending.

**Why this output:** `startTransition` marks `setTab` as non-urgent. React renders the new tab in the background and commits when ready. `isPending` is `true` during the background render and `false` after the commit.

#### Example 2: Search with Async Action (React 19)

```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(value) {
    setQuery(value); // Urgent: update input immediately
    startTransition(async () => {
      const data = await fetchResults(value); // Async action
      setResults(data); // Included in the transition in React 19
    });
  }

  return (
    <div>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </div>
  );
}
```

**Expected Output:** The input updates immediately. The results update after the fetch completes, and `isPending` stays `true` for the entire fetch duration.

**Why this output:** In React 19, `isPending` remains `true` throughout the async action. The input update is urgent, while the fetch and result update are non-urgent transitions.

### Real-World Cases

- **Tab containers:** Switching between heavy tabs without blocking navigation.
- **Search and filtering:** Keeping input responsive while results update.
- **Route transitions:** Navigating between screens while keeping the current screen interactive.
- **Product tours:** Step navigation without input delay on slow devices.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- React — useTransition (official) — https://react.dev/reference/react/useTransition.md
- Steve Kinney — useTransition and startTransition — https://raw.githubusercontent.com/stevekinney/stevekinney.net

---

## 2. Transitions

### Definitions

**Core Definition:** A transition is a state update that React treats as non-urgent, allowing it to be interrupted by more important updates.

**Technical Definition:** A transition marks an update as non-urgent, allowing React to interrupt it for more important work. Transitions are created by passing state updates to `startTransition`. The updates are assigned to a lower-priority lane in React's scheduler. If an urgent update arrives (e.g., a keystroke), React abandons the in-progress background render and restarts it with the latest state. Transitions are distinct from urgent updates, which are processed immediately.

**Beginner-Friendly Explanation:** A transition is a "background update." It is a way of telling React: "This can wait." If the user does something more important, React pauses the transition and handles the urgent work first. When the urgent work is done, React resumes or restarts the transition.

### Purposes

- To mark expensive state updates as non-urgent.
- To keep the UI responsive during background rendering.
- To allow urgent updates to interrupt and preempt non-urgent work.
- To avoid blocking the main thread with heavy renders.

### Syntax Rules and Structure

**Creating a Transition:**

```jsx
startTransition(() => {
  setExpensiveState(newValue);
});
```

**Syntax Rules:**

1. Only state updates inside `startTransition` are marked as transitions.
2. Transitions are interruptible; urgent updates preempt them.
3. The `isPending` flag indicates whether a transition is in progress.
4. Transitions reorder work; they do not make the work itself cheaper.

### Annotated Code Examples

```jsx
function FilterableList({ items }) {
  const [query, setQuery] = useState('');
  const [filtered, setFiltered] = useState(items);
  const [isPending, startTransition] = useTransition();

  function handleChange(value) {
    setQuery(value); // Urgent
    startTransition(() => {
      setFiltered(expensiveFilter(items, value)); // Non-urgent transition
    });
  }

  return (
    <>
      <input value={query} onChange={(e) => handleChange(e.target.value)} />
      {isPending && <p>Filtering…</p>}
      <List items={filtered} />
    </>
  );
}
```

**Expected Output:** The input remains responsive while the expensive filter runs in a background transition.

**Why this output:** `setQuery` is urgent. The filter and `setFiltered` are inside `startTransition`, so they are interruptible. If the user types again, React abandons the in-progress filter and restarts with the new query.

### Real-World Cases

- **Large list filtering:** Filtering thousands of items without blocking typing.
- **Tab switching:** Rendering heavy tab content in the background.
- **Data visualisation:** Updating charts without freezing controls.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- Concurrent Features — React — https://mintlify.wiki/facebook/react/advanced/concurrent-features
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 3. Pending State

### Definitions

**Core Definition:** Pending state is the boolean flag returned by `useTransition` that indicates whether a transition is currently in progress.

**Technical Definition:** The `isPending` flag is `true` while a transition is being rendered in the background and `false` once the transition has committed. In React 18, `isPending` would become `false` at the first `await` inside an async action. In React 19, `isPending` remains `true` for the entire duration of the asynchronous operation, making it suitable for managing busy indicators across async flows. The flag can be used to disable buttons, show subtle loading indicators, or prevent duplicate submissions.

**Beginner-Friendly Explanation:** `isPending` tells you whether the background update is still running. It is `true` while React is working on the transition and `false` when it is done. You use it to show a spinner, disable a button, or display a "loading" message.

### Purposes

- To show a loading indicator while a transition is in progress.
- To disable buttons or prevent duplicate submissions during a transition.
- To provide subtle feedback without blocking the UI.
- To track the completion of async actions in React 19.

### Syntax Rules and Structure

```jsx
const [isPending, startTransition] = useTransition();
```

**Syntax Rules:**

1. `isPending` is `true` during the transition render and `false` after commit.
2. In React 19, `isPending` stays `true` for the entire async action duration.
3. Use `isPending` to conditionally render loading UI or disable controls.
4. Do not use `isPending` for urgent updates; it only reflects transitions.

**Constraints and Limitations:**

- `isPending` causes an extra render (the immediate `isPending = true` render).
- In React 18, async transitions may set `isPending` to `false` prematurely at the first await.
- `isPending` does not tell you the progress or percentage of the transition.

### Annotated Code Examples

```jsx
function SubmitButton({ submitAction }) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await submitAction();
        });
      }}
    >
      {isPending ? 'Submitting…' : 'Submit'}
    </button>
  );
}
```

**Expected Output:** The button is disabled and shows "Submitting…" during the async transition, then returns to "Submit" when done.

**Why this output:** `isPending` is `true` throughout the async action in React 19. The `disabled` prop and button text react to the pending state.

### Real-World Cases

- **Form submissions:** Disabling the submit button during async submission.
- **Search results:** Showing "Updating results…" while filtering.
- **Tab navigation:** Showing a pending indicator while the next tab renders.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- Async React: Building Non-Blocking UIs with useTransition and useActionState — https://www.rubrik.com/blog/architecture/26/2/async-react-building-non-blocking-uis-with-usetransition-and-useactionstate
- Use useTransition Over Manual Loading States — https://github.com/zebbern/claude-code-guide

---

## 4. `startTransition`

### Definitions

**Core Definition:** `startTransition` is the function returned by `useTransition` (or imported standalone from React) that marks state updates as non-urgent transitions.

**Technical Definition:** `startTransition(action)` calls the `action` function immediately with no parameters and marks all state updates scheduled synchronously during that call as transitions. The action is called immediately; it is the state updates that are deferred to a lower-priority lane. `startTransition` does not return anything. A standalone `startTransition` import is available for use outside components (e.g., in data libraries).

**Beginner-Friendly Explanation:** `startTransition` is the tool you use to wrap state updates that you want to defer. You pass it a function, and any state updates inside that function become background updates.

### Purposes

- To mark state updates as non-urgent transitions.
- To wrap multiple related state updates in a single transition.
- To use transitions outside components via the standalone import.
- To enable interruption of expensive updates by urgent work.

### Syntax Rules and Structure

**Hook Version:**

```jsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setExpensiveState(newValue);
});
```

**Standalone Version:**

```jsx
import { startTransition } from 'react';

startTransition(() => {
  setGlobalState(newValue);
});
```

**Syntax Rules:**

1. `startTransition` calls its action immediately.
2. Synchronous state updates inside the action are marked as transitions.
3. State updates after `await` must be wrapped in another `startTransition`.
4. The standalone `startTransition` can be used outside components.
5. `startTransition` has a stable identity.

**Constraints and Limitations:**

- You must have access to the state setter to wrap an update in a transition.
- State updates in `setTimeout` are not marked as transitions.
- Async state updates after `await` require an additional `startTransition` wrapper.

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
      setResults(expensiveFilter(value)); // Non-urgent
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

**Expected Output:** The input updates immediately; the results update in the background with a pending indicator.

**Why this output:** `setQuery` is outside `startTransition`, so it is urgent. `setResults` is inside `startTransition`, so it is deferred and interruptible.

### Real-World Cases

- **Search filtering:** Marking the filter update as non-urgent while the query update stays urgent.
- **Tab containers:** Wrapping tab state updates in `startTransition`.
- **Global state updates:** Using the standalone `startTransition` in data libraries.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- Concurrent Features — React — https://mintlify.wiki/facebook/react/advanced/concurrent-features
- Steve Kinney — useTransition and startTransition — https://raw.githubusercontent.com/stevekinney/stevekinney.net

---

## 5. Concurrent Rendering

### Definitions

**Core Definition:** Concurrent rendering is React's interruptible rendering model that allows the framework to prepare multiple UI versions simultaneously and prioritise urgent updates.

**Technical Definition:** Concurrent rendering is a behind-the-scenes mechanism that enables React to prepare multiple versions of the UI at the same time. Rendering is interruptible: React may start rendering an update, pause in the middle, abandon it, or continue later. Transitions are a developer-facing entry point to this model. In React 18, concurrent rendering was opt-in for transitions; in React 19, it is the default scheduling behaviour for all components. The scheduler slices rendering work into ~5ms chunks and yields to the browser between them.

**Beginner-Friendly Explanation:** Concurrent rendering is React's ability to multitask. Instead of freezing while it finishes a slow render, React can pause and let the browser respond to the user, then resume. Transitions are how you tell React which work is allowed to be paused and resumed.

### Purposes

- To keep the UI responsive during expensive renders.
- To prioritise urgent updates over non-urgent ones.
- To prevent long tasks from blocking the main thread.
- To enable features like transitions, deferred values, and Suspense.

### Syntax Rules and Structure

**How Transitions Use Concurrent Rendering:**

| Concept | Description |
|---------|-------------|
| Lane | A priority bucket for scheduling updates |
| Transition lane | Lower-priority lane for transitions |
| Urgent lane | Higher-priority lane for user input |
| Interruption | Urgent updates preempt transition lanes |

**Syntax Rules:**

1. Concurrent rendering is the default in React 19.
2. Transitions use lower-priority lanes; urgent updates use higher-priority lanes.
3. Rendering is interruptible; commits are atomic.
4. Components must be pure to support interruption.

### Annotated Code Examples

```jsx
function ProductSearch() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(value) {
    setQuery(value); // Urgent lane
    startTransition(() => {
      setResults(searchProducts(value)); // Transition lane
    });
  }

  return (
    <div>
      <input value={query} onChange={(e) => handleChange(e.target.value)} />
      {isPending && <p>Searching…</p>}
      <Results results={results} />
    </div>
  );
}
```

**Expected Output:** Typing remains smooth even while the search runs, because the input update is in an urgent lane and the search is in a lower-priority transition lane.

**Why this output:** Concurrent rendering assigns the input update to a higher-priority lane and the search update to a transition lane. When the user types, React can interrupt the search to process the input.

### Real-World Cases

- **Search autocomplete:** Keeping typing responsive during expensive filtering.
- **Data dashboards:** Rendering charts in the background while controls remain interactive.
- **Product tours:** Step navigation without input delay on slow devices.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- Concurrent Features — React — https://mintlify.wiki/facebook/react/advanced/concurrent-features
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 6. Non-Urgent Updates

### Definitions

**Core Definition:** Non-urgent updates are state updates that can be deferred and interrupted by more important work, typically marked as transitions.

**Technical Definition:** Non-urgent updates are state updates assigned to a lower-priority lane by `startTransition`. They do not block user input and can be interrupted if a higher-priority update arrives. Examples include filtering a large list, switching to a heavy tab, or rendering a complex chart. Non-urgent updates are the counterpart to urgent updates, which are processed immediately.

**Beginner-Friendly Explanation:** Non-urgent updates are updates that can wait. They are the "background work" of your UI. If the user does something more important, React pauses the background work and handles the urgent thing first.

### Purposes

- To keep the UI responsive by deferring expensive updates.
- To allow urgent work to preempt non-urgent work.
- To avoid blocking the main thread with heavy renders.
- To improve Interaction to Next Paint (INP).

### Syntax Rules and Structure

```jsx
startTransition(() => {
  setNonUrgentState(newValue); // Non-urgent
});
```

**Syntax Rules:**

1. Wrap non-urgent updates in `startTransition`.
2. Keep urgent updates outside the transition.
3. Use `isPending` to indicate background work.
4. Non-urgent updates are interruptible.

### Annotated Code Examples

```jsx
function FilterPanel({ items }) {
  const [filter, setFilter] = useState('');
  const [filtered, setFiltered] = useState(items);
  const [isPending, startTransition] = useTransition();

  function handleFilterChange(value) {
    setFilter(value); // Urgent: update the input
    startTransition(() => {
      setFiltered(applyFilter(items, value)); // Non-urgent: expensive
    });
  }

  return (
    <>
      <input value={filter} onChange={(e) => handleFilterChange(e.target.value)} />
      {isPending && <p>Applying filter…</p>}
      <List items={filtered} />
    </>
  );
}
```

**Expected Output:** The input updates instantly; the filtered list updates in the background with a pending indicator.

**Why this output:** The input update is urgent and processed immediately. The filter computation and list update are non-urgent and deferred, so typing remains smooth.

### Real-World Cases

- **Search results:** Deferring the results update while keeping the input urgent.
- **Tab content:** Deferring heavy tab rendering while tab selection is urgent.
- **Chart updates:** Deferring chart re-renders while controls remain interactive.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- Concurrent Features — React — https://mintlify.wiki/facebook/react/advanced/concurrent-features
- Async React: Building Non-Blocking UIs — https://www.rubrik.com/blog/architecture/26/2/async-react-building-non-blocking-uis-with-usetransition-and-useactionstate

---

## 7. Urgent vs Non-Urgent Updates

### Definitions

**Core Definition:** Urgent updates are state updates that must be processed immediately to keep the UI responsive; non-urgent updates are those that can be deferred.

**Technical Definition:** React distinguishes between urgent and non-urgent updates through its lane-based scheduler. Urgent updates (typing, clicking, focus) are processed in higher-priority lanes and cannot be interrupted. Non-urgent updates (filtering, charting, tab switching) are wrapped in transitions and processed in lower-priority lanes. Urgent updates preempt non-urgent ones: if a user types while a transition is rendering, React abandons the transition and restarts it later.

**Beginner-Friendly Explanation:** Urgent updates are things the user is actively doing — typing, clicking, dragging. They need to feel instant. Non-urgent updates are things that can happen in the background — updating a chart, filtering a list. React handles urgent things first and does non-urgent things when it has time.

### Purposes

- To explicitly separate work that must be immediate from work that can wait.
- To prevent background work from blocking user input.
- To improve perceived responsiveness and INP.
- To let React prioritise correctly without manual debouncing in many cases.

### Syntax Rules and Structure

**Urgent vs Non-Urgent Classification:**

| Update type | Classification | Where to place it |
|-------------|---------------|-------------------|
| Input value | Urgent | Outside `startTransition` |
| Toggle state | Urgent | Outside `startTransition` |
| Filtered results | Non-urgent | Inside `startTransition` |
| Chart data | Non-urgent | Inside `startTransition` |
| Tab content | Non-urgent | Inside `startTransition` |

**Syntax Rules:**

1. Keep input state updates outside `startTransition`.
2. Wrap derived, filtered, or expensive state updates inside `startTransition`.
3. Never put urgent state updates inside a transition.
4. Use `isPending` to indicate non-urgent work.

### Annotated Code Examples

```jsx
function SearchWithUrgency() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(value) {
    setQuery(value); // URGENT: input must update immediately
    startTransition(() => {
      setResults(expensiveSearch(value)); // NON-URGENT: can be deferred
    });
  }

  return (
    <>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  );
}
```

**Expected Output:** The input updates immediately (urgent); the results update in the background (non-urgent) with a spinner.

**Why this output:** `setQuery` is outside `startTransition`, so it is processed in an urgent lane. `setResults` is inside `startTransition`, so it is deferred. The user can keep typing without waiting for the search.

### Real-World Cases

- **Search interfaces:** Urgent query updates, non-urgent results updates.
- **Tab containers:** Urgent tab selection, non-urgent content rendering.
- **Dashboards:** Urgent filter changes, non-urgent chart re-renders.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- React's useTransition: The hook you're probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide
- Steve Kinney — useTransition and startTransition — https://raw.githubusercontent.com/stevekinney/stevekinney.net

---

## 8. `useTransition` Best Practices

### Definitions

**Core Definition:** `useTransition` best practices are the guidelines for using transitions effectively, avoiding common pitfalls, and choosing between `useTransition` and `useDeferredValue`.

**Technical Definition:** Best practices centre on five principles: use transitions for genuinely expensive CPU-bound updates, not network requests; keep urgent input state outside the transition; replace manual loading state with `isPending`; wrap state updates after `await` in another `startTransition`; and choose `useDeferredValue` when you do not control the state setter. The React team and community consistently emphasise that transitions reorder work — they do not reduce the cost of the work itself — and that blind use of transitions without profiling can add unnecessary double renders.

**Beginner-Friendly Explanation:** Use `useTransition` when you have a slow, non-urgent state update and you control the setter. Do not wrap network requests in a transition and expect it to cancel them — it does not. Keep typing and clicking urgent. Use `isPending` instead of manually managing a loading state. And if the value comes from props or an external store, use `useDeferredValue` instead.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use for CPU-bound work, not network requests** | Transitions optimise rendering priority; they do not abort or manage fetch requests. |
| **Keep input state outside the transition** | The input value must update immediately; only defer the derived/expensive state. |
| **Replace manual loading state with `isPending`** | `isPending` is built-in, resets automatically, and handles errors and interruptions. |
| **Wrap state updates after `await` in another `startTransition`** | This is a known limitation; without it, post-await updates are not marked as transitions. |
| **Choose `useDeferredValue` when you don't control the setter** | If the value comes from props or an external store, `useDeferredValue` is the right tool. |
| **Do not put urgent updates inside a transition** | Urgent updates (typing, clicking) must remain outside `startTransition`. |
| **Use transitions for tab navigation** | Tab content rendering is a classic use case for non-urgent transitions. |
| **Profile before and after** | Transitions reorder work; verify they actually improve responsiveness for your use case. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(value) {
    setQuery(value); // Urgent — outside the transition
    startTransition(async () => {
      const data = await fetchResults(value);
      // State update after await is inside the transition (React 19)
      setResults(data);
    });
  }

  return (
    <>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ Wrapping the input update in the transition
startTransition(() => {
  setQuery(value); // Input becomes laggy!
});

// ❌ Using manual loading state instead of isPending
const [isLoading, setIsLoading] = useState(false);
setIsLoading(true);
await fetchResults();
setIsLoading(false);

// ❌ Wrapping a network request without managing abort
startTransition(async () => {
  await fetch('/api/data'); // Transition does not cancel this
});
```

**Expected Output:** The good practices keep the input responsive, use `isPending` for loading feedback, and correctly handle async state updates. The bad practices cause laggy input, redundant loading state, and unmanaged requests.

**Why this output:** Keeping the input outside the transition preserves urgent responsiveness. `isPending` replaces manual state and resets automatically. Transitions do not abort network requests; they only affect rendering priority.

### Real-World Cases

- **Search autocomplete:** Urgent input, non-urgent results, `isPending` for loading.
- **Tab navigation:** Urgent tab selection, non-urgent content rendering.
- **Form submissions:** Async actions with `isPending` for button states.
- **Data dashboards:** Urgent controls, non-urgent chart updates.

### References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- Use useTransition Over Manual Loading States — https://github.com/zebbern/claude-code-guide
- React's useTransition: The hook you're probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide
- Async React: Building Non-Blocking UIs with useTransition and useActionState — https://www.rubrik.com/blog/architecture/26/2/async-react-building-non-blocking-uis-with-usetransition-and-useactionstate

---

## References

- React — useTransition — https://ar.react.dev/reference/react/useTransition
- React — useTransition (official) — https://react.dev/reference/react/useTransition.md
- React — startTransition (standalone) — https://react.dev/reference/react/startTransition
- React — useDeferredValue — https://react.dev/reference/react/useDeferredValue
- Concurrent Features — React — https://mintlify.wiki/facebook/react/advanced/concurrent-features
- Steve Kinney — useTransition and startTransition — https://raw.githubusercontent.com/stevekinney/stevekinney.net
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- Async React: Building Non-Blocking UIs with useTransition and useActionState — https://www.rubrik.com/blog/architecture/26/2/async-react-building-non-blocking-uis-with-usetransition-and-useactionstate
- Use useTransition Over Manual Loading States — https://github.com/zebbern/claude-code-guide
- React's useTransition: The hook you're probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide
- React 19 concurrent mode and product tours — https://raw.githubusercontent.com/domidex01/tour-kit
- agent-skills — React Best Practices: useTransition — https://github.com/tech-leads-club/agent-skills