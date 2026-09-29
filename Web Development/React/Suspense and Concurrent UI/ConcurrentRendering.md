# Concurrent Rendering Concepts: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrent Rendering is React's rendering architecture that allows multiple versions of the UI to be prepared simultaneously, enabling React to interrupt, pause, resume, or abandon renders to keep the application responsive.

**Technical Definition:** Concurrent Rendering is a set of capabilities introduced with React 18's new root API (`createRoot`) that allows React's rendering work to be broken into smaller units, prioritised, and interrupted. It is built on the Fiber reconciler architecture, which replaces the recursive, non-interruptible stack reconciler of React 15 and earlier. Under concurrent rendering, a render pass may be started, paused, resumed, restarted, or discarded entirely without committing to the DOM. This enables features such as `startTransition`, `useDeferredValue`, Suspense-enabled data fetching, streaming SSR, and selective hydration. Critically, Concurrent Rendering is a mechanism, not a feature: `useState`, `useEffect`, and other Hooks behave consistently across concurrent and legacy rendering modes.

**Beginner-Friendly Explanation:** Normally, when React needs to update the screen, it does all the work at once. If that work is heavy—like rendering a huge list of items—the browser freezes and the user can't type, click, or scroll. Concurrent Rendering changes this: React can start working on an update, pause if something more important happens (like the user typing), handle the urgent thing, and then come back to finish the original update. The UI stays responsive even during heavy renders.

### Key Characteristics

- **Interruptible Rendering:** React can pause a render, yield control to the browser, and resume later.
- **Prioritised Updates:** Urgent updates (typing, clicking) are processed before non-urgent updates (data fetching, list rendering).
- **Double Buffering:** React maintains two Fiber trees—the "current" tree (visible on screen) and the "work-in-progress" tree (being built offscreen).
- **Opt-In via Concurrent Features:** Apps using `createRoot` get concurrent rendering by default, but the concurrent *features* (`startTransition`, `useDeferredValue`) must be explicitly used.
- **No Tearing:** Concurrent rendering ensures that all components in a single render pass see the same version of state.
- **Compatible with SSR and Streaming:** Concurrent rendering works with `renderToPipeableStream` and `renderToReadableStream` for streaming server rendering and selective hydration.

### Prerequisites

- Solid understanding of React components, state, and Hooks.
- Familiarity with `useState`, `useEffect`, and the render/commit lifecycle.
- Basic understanding of the browser's event loop and main thread.
- Knowledge of virtual DOM and reconciliation (React's diffing algorithm).
- Familiarity with Promises and asynchronous JavaScript.

### Related Programming Areas

- **Fiber Reconciler:** React's internal reconciliation engine.
- **Scheduler:** React's priority-based task scheduling system.
- **Server-Side Rendering (SSR):** Rendering React components to HTML on the server.
- **Streaming HTML:** Delivering HTML progressively as it's generated.
- **Selective Hydration:** Prioritising hydration of interactive parts of the page.
- **Web Performance:** Main thread responsiveness, frame rates, and Core Web Vitals.
- **Reactive Programming:** Managing data flow and change propagation.

### Core Concepts / Features

1. Time-Slicing & Rendering Priorities
2. Interruptible Rendering
3. The Fiber Architecture Evolution
4. Streaming Architecture Compatibility

---

## Core Concept 1: Time-Slicing & Rendering Priorities

### Definitions

**Core Definition:** Time-slicing is the technique of breaking a large rendering task into small chunks that are processed over multiple browser frames, while rendering priorities determine which updates React processes first.

**Technical Definition:** Time-slicing leverages the browser's `MessageChannel` API (or `setTimeout` as a fallback) to yield control back to the browser between chunks of work. React's Scheduler assigns each unit of work a priority based on the type of update: `ImmediatePriority` (synchronous, e.g., `flushSync`), `UserBlockingPriority` (e.g., clicks, keypresses), `NormalPriority` (e.g., data fetching, transitions), `LowPriority`, and `IdlePriority`. The Scheduler uses `shouldYield()` to determine whether the current time slice has expired (default 5ms), at which point it yields to the browser to allow painting and event handling. This mechanism prevents long tasks from blocking the main thread and keeps the UI responsive.

**Beginner-Friendly Explanation:** The browser can only do one thing at a time on the main thread. If React takes too long to render something, the browser can't respond to clicks or keypresses. Time-slicing is React's way of saying: "I'll work on this for a few milliseconds, then check if the browser needs to do something more important. If so, I'll pause and come back later." Priorities determine what "more important" means—a user typing is more important than loading a list of items.

### Purposes

- To prevent long rendering tasks from blocking the main thread.
- To keep the UI responsive during heavy renders.
- To prioritise user interactions over background updates.
- To enable smooth animations and transitions even during data loading.
- To reduce the time to interactive (TTI) for complex applications.
- To improve Core Web Vitals, particularly Interaction to Next Paint (INP).

### Syntax Rules and Structure

**General Syntax for Using Transitions (the primary API for lower-priority updates):**
```jsx
import { startTransition, useState } from 'react';

function SearchBox() {
  const [inputValue, setInputValue] = useState('');
  const [searchQuery, setSearchQuery] = useState('');

  function handleChange(e) {
    // Urgent update: keep the input responsive
    setInputValue(e.target.value);

    // Non-urgent update: mark as a transition
    startTransition(() => {
      setSearchQuery(e.target.value);
    });
  }

  return (
    <div>
      <input value={inputValue} onChange={handleChange} />
      <SearchResults query={searchQuery} />
    </div>
  );
}
```

**Component Breakdown:**
- `setInputValue(e.target.value)`: An urgent update that React processes immediately, keeping the input field responsive.
- `startTransition(() => { setSearchQuery(...) })`: Marks the update as non-urgent. React can interrupt or deprioritise it if the user continues typing.
- `useTransition`: An alternative Hook that provides an `isPending` flag for showing loading indicators.

**General Syntax for `useTransition`:**
```jsx
import { useTransition, useState } from 'react';

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
      <button onClick={() => selectTab('home')}>Home</button>
      <button onClick={() => selectTab('posts')}>Posts</button>
      {isPending && <p>Loading tab...</p>}
      <TabContent tab={tab} />
    </div>
  );
}
```

**Component Breakdown:**
- `useTransition()`: Returns `[isPending, startTransition]`.
- `isPending`: A boolean indicating whether the transition is still in progress.
- `startTransition(() => { setTab(nextTab) })`: Wraps the state update in a transition.
- React renders the current tab while preparing the next tab in the background.

**Syntax Rules:**
- Updates inside `startTransition` are interruptible and can be discarded if a newer transition starts.
- Transitions cannot be used to control text inputs (the input value must update urgently).
- `startTransition` does not delay the update; it deprioritises it.
- If a transition is interrupted, React discards the work and restarts with the latest state.
- `isPending` can be used to show a loading indicator while the transition is in progress.
- Multiple transitions can be batched together.

**Constraints and Limitations:**
- `startTransition` does not work with `flushSync`.
- Transitions do not work for controlling text inputs (React warns about this).
- If a transition is interrupted too many times, React may eventually commit the current state to avoid starvation.
- `useTransition` must be called inside a component or custom Hook.
- Time-slicing works best with the `MessageChannel` API; in environments without it, React falls back to `setTimeout`, which has a minimum 4ms delay.

### Annotated Code Examples

**Example 1: Search Input with Transition-Based Deferral**

```jsx
import React, { useState, useTransition } from 'react';

// A component that renders a large list of items
function SearchResults({ query }) {
  // Simulate a heavy computation
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`)
    .filter(item => item.toLowerCase().includes(query.toLowerCase()));

  return (
    <ul>
      {items.slice(0, 100).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}

function SearchApp() {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    // Urgent: update the input value immediately
    setInput(e.target.value);

    // Non-urgent: defer the search query update
    startTransition(() => {
      setQuery(e.target.value);
    });
  }

  return (
    <div>
      <input
        value={input}
        onChange={handleChange}
        placeholder="Type to search..."
      />
      {isPending && <p>Updating results...</p>}
      <SearchResults query={query} />
    </div>
  );
}

export default SearchApp;
```

**Expected Output:** As the user types, the input field updates instantly with every keystroke. The search results list updates a moment later, and "Updating results..." appears briefly while the transition is in progress. If the user types quickly, intermediate search results are skipped, and only the final query's results are rendered.

**Why This Output Occurs:** The `setInput` call is an urgent update, so React processes it immediately, keeping the input responsive. The `setQuery` call inside `startTransition` is marked as non-urgent, so React can interrupt it if the user types another character. When the user types quickly, React discards in-progress renders of `SearchResults` and restarts with the latest query. The `isPending` flag from `useTransition` indicates when the transition is in progress.

**Example 2: Tab Switching with `useTransition`**

```jsx
import React, { useState, useTransition } from 'react';

function SlowTab({ label }) {
  // Simulate a heavy render
  const start = performance.now();
  while (performance.now() - start < 100) {
    // Artificial delay to simulate a heavy component
  }

  return <div>Content for {label} (rendered slowly)</div>;
}

function TabApp() {
  const [tab, setTab] = useState('A');
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <div>
      <button onClick={() => selectTab('A')}>Tab A</button>
      <button onClick={() => selectTab('B')}>Tab B</button>
      <button onClick={() => selectTab('C')}>Tab C</button>

      {isPending && <p>Switching tab...</p>}

      <SlowTab label={tab} />
    </div>
  );
}

export default TabApp;
```

**Expected Output:** Clicking a tab button shows "Switching tab..." briefly while the new tab's content is being prepared. The current tab's content remains visible until the new tab is ready, at which point it replaces the old content. The UI remains responsive to further clicks during the transition.

**Why This Output Occurs:** The `selectTab` function wraps the state update in `startTransition`, marking it as non-urgent. React keeps the current tab visible while preparing the new tab's render in the background. If the user clicks another tab before the first transition completes, React discards the in-progress render and starts a new transition for the latest tab. The `isPending` flag shows the loading indicator.

### Real-World Cases

- **Search autocomplete:** Keeping the input responsive while filtering a large dataset.
- **Tab navigation:** Switching between heavy tabs without blocking the UI.
- **Filtering large lists:** Applying filters without freezing the interface.
- **Route transitions:** Preparing the next page while keeping the current one interactive.
- **Theme switching:** Applying a new theme without blocking user interactions.
- **Data visualisation updates:** Re-rendering complex charts without locking the main thread.

---

## Core Concept 2: Interruptible Rendering

### Definitions

**Core Definition:** Interruptible rendering is the ability of React's concurrent renderer to pause, discard, or restart a render in progress when a higher-priority update occurs.

**Technical Definition:** In concurrent rendering, the render phase is divided into units of work represented by Fiber nodes. After completing each unit, React checks whether a higher-priority update has been scheduled (via the Scheduler's `shouldYield()` and priority comparison). If a higher-priority update exists, React pauses the current render, saves the work-in-progress Fiber tree, and processes the urgent update. Once the urgent update is committed, React resumes the lower-priority render. If the lower-priority render's state becomes stale (e.g., a new transition supersedes it), React discards the work-in-progress tree and starts over with the latest state. This is safe because the render phase is pure and has no side effects; only the commit phase mutates the DOM, and it is never interrupted.

**Beginner-Friendly Explanation:** Imagine you're writing a long document on your computer, and suddenly your boss asks you to do something urgent. You save your document, handle the urgent task, and then come back to your writing. React does the same thing: if it's in the middle of a big render and the user types a key, React saves its progress, handles the keystroke, and then resumes the big render. If the user keeps typing, React might decide the big render is no longer needed and start over with fresh data.

### Purposes

- To keep the UI responsive during heavy renders.
- To ensure that urgent updates (user input) are processed before non-urgent updates.
- To avoid wasting work on renders that will be discarded.
- To prevent visual inconsistencies (tearing) during concurrent updates.
- To enable smooth transitions between UI states.
- To allow React to prioritise the most relevant work.

### Syntax Rules and Structure

**General Syntax (internal mechanism—no direct API, but affects how you write components):**

The interruptibility of a render depends on the priority assigned to the update:

```jsx
import { startTransition, useState } from 'react';

function Component() {
  const [urgent, setUrgent] = useState('');
  const [nonUrgent, setNonUrgent] = useState('');

  // Urgent update — NOT interruptible
  function handleUrgent(value) {
    setUrgent(value);
  }

  // Non-urgent update — interruptible
  function handleNonUrgent(value) {
    startTransition(() => {
      setNonUrgent(value);
    });
  }
}
```

**Component Breakdown:**
- `setUrgent(value)`: An urgent update that React processes synchronously and cannot be interrupted.
- `startTransition(() => { setNonUrgent(value) })`: A non-urgent update that React can interrupt, pause, or discard.

**Render Phase vs Commit Phase:**

```jsx
function Example() {
  // RENDER PHASE (interruptible, must be pure)
  const [count, setCount] = useState(0);

  // ❌ Side effect during render — BAD
  // document.title = `Count: ${count}`;

  // ✅ Side effect in useEffect — GOOD
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  // COMMIT PHASE (not interruptible)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Component Breakdown:**
- The render phase (everything before the `return`) must be pure and free of side effects. React may call it multiple times, pause it, or discard it.
- The commit phase (after React determines the changes) is synchronous and never interrupted.
- Side effects must be placed in `useEffect` (which runs after commit) or in event handlers.

**Syntax Rules:**
- Render functions must be pure: no side effects, no mutations of external state, no calls to browser APIs.
- Do not rely on the render phase executing only once; React may call it multiple times.
- Do not perform side effects during render; use `useEffect` for post-commit effects.
- Use `startTransition` to mark updates as interruptible.
- Urgent updates (e.g., text input, hover, click) should never be wrapped in `startTransition`.
- The commit phase is synchronous and cannot be interrupted; keep commit-phase work minimal.

**Constraints and Limitations:**
- Only updates wrapped in `startTransition` (or `useDeferredValue`) are interruptible.
- Urgent updates (e.g., `setState` outside transitions) are processed synchronously and cannot be interrupted.
- The render phase must be pure; impure renders can cause bugs when React interrupts and restarts them.
- React may discard a work-in-progress tree entirely if it becomes stale.
- `flushSync` forces synchronous, non-interruptible updates.
- Interruptible rendering is only available with `createRoot` (React 18+), not with the legacy `ReactDOM.render`.

### Annotated Code Examples

**Example 1: Demonstrating Interruptibility with a Heavy List**

```jsx
import React, { useState, useTransition } from 'react';

function HeavyList({ count }) {
  // Simulate a very heavy render — 50ms per item
  const items = [];
  for (let i = 0; i < count; i++) {
    const start = performance.now();
    while (performance.now() - start < 50) {
      // Artificial delay
    }
    items.push(<li key={i}>Heavy Item {i + 1}</li>);
  }

  return <ul>{items}</ul>;
}

function InterruptibleDemo() {
  const [count, setCount] = useState(1);
  const [input, setInput] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleAddItem() {
    // Non-urgent: adding items is a transition
    startTransition(() => {
      setCount(c => c + 1);
    });
  }

  return (
    <div>
      <input
        value={input}
        onChange={e => setInput(e.target.value)}
        placeholder="Type here while list renders..."
      />
      <button onClick={handleAddItem}>Add Heavy Item</button>
      {isPending && <p>Rendering new item...</p>}
      <HeavyList count={count} />
    </div>
  );
}

export default InterruptibleDemo;
```

**Expected Output:** Clicking "Add Heavy Item" starts rendering a new item, which takes 50ms per item. While the list is rendering, the user can type in the input field without any lag. If the user types, React pauses the list render, handles the keystroke, and then resumes the list render. The "Rendering new item..." message appears while the transition is pending.

**Why This Output Occurs:** The `setCount` update is wrapped in `startTransition`, making it interruptible. While `HeavyList` is rendering the new item, React periodically checks if there are higher-priority updates. The user's keystrokes trigger urgent `setInput` updates, which React processes immediately, pausing the list render. Once the keystroke is handled, React resumes the list render. This demonstrates the core benefit of interruptible rendering: the UI stays responsive even during heavy renders.

**Example 2: Discarding Stale Renders**

```jsx
import React, { useState, useTransition } from 'react';

function SlowFilteredList({ query }) {
  // Simulate a slow filter operation
  const start = performance.now();
  const items = Array.from({ length: 5000 }, (_, i) => `Item ${i}`)
    .filter(item => {
      while (performance.now() - start < 100) {
        // Artificial delay
      }
      return item.includes(query);
    });

  return <ul>{items.slice(0, 20).map(item => <li key={item}>{item}</li>)}</ul>;
}

function DiscardDemo() {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setInput(e.target.value);
    startTransition(() => {
      setQuery(e.target.value);
    });
  }

  return (
    <div>
      <input value={input} onChange={handleChange} />
      {isPending && <p>Filtering...</p>}
      <SlowFilteredList query={query} />
    </div>
  );
}

export default DiscardDemo;
```

**Expected Output:** As the user types quickly, intermediate filter operations are discarded, and only the final query's results are displayed. The input field remains responsive, and "Filtering..." appears while a transition is in progress.

**Why This Output Occurs:** Each keystroke triggers a new transition. If a previous transition is still in progress, React discards its work-in-progress tree and starts a new render with the latest query. This prevents wasted work on renders that are no longer relevant. The `isPending` flag indicates that a transition is in progress.

### Real-World Cases

- **Rich text editors:** Keeping the editor responsive while rendering large documents.
- **Data grids:** Filtering and sorting thousands of rows without blocking the UI.
- **Code editors:** Highlighting syntax in large files while the user types.
- **Chat applications:** Rendering long message histories while new messages arrive.
- **Dashboards:** Updating multiple charts while the user interacts with controls.
- **Form validation:** Validating complex forms without blocking input.

---

## Core Concept 3: The Fiber Architecture Evolution

### Definitions

**Core Definition:** Fiber is React's reconciliation engine, introduced in React 16, that represents the component tree as a linked list of units of work, enabling interruptible, prioritised rendering.

**Technical Definition:** Fiber is a complete rewrite of React's core reconciliation algorithm. Each Fiber node represents a unit of work corresponding to a component instance, DOM element, or other React element. Fibers are organised as a linked list tree using `child`, `sibling`, and `return` pointers, allowing React to traverse the tree iteratively rather than recursively. This iterative traversal enables React to pause after completing a Fiber, check for higher-priority work, and resume later. React maintains two Fiber trees: the **current tree** (reflecting what's on screen) and the **work-in-progress tree** (being built offscreen). When the work-in-progress tree is complete, React commits it by swapping the two trees atomically, ensuring no tearing. The Fiber architecture also enables features like Suspense, error boundaries, Hooks, and concurrent rendering.

**Beginner-Friendly Explanation:** Before Fiber, React rendered components recursively—like following a trail of dominoes, one after another, without stopping. If the trail was long, the browser froze. Fiber changed this by turning the component tree into a linked list of small tasks. React can now process one task at a time, check if the browser needs to do something, and come back later. Think of it as the difference between reading a book chapter by chapter (Fiber) versus reading it in one sitting without breaks (old React).

### Purposes

- To enable interruptible, prioritised rendering.
- To represent the component tree as a linked list of units of work.
- To maintain two Fiber trees (current and work-in-progress) for double buffering.
- To support Hooks, Suspense, and error boundaries.
- To enable concurrent features like `startTransition` and `useDeferredValue`.
- To provide a foundation for streaming SSR and selective hydration.

### Syntax Rules and Structure

**Internal Fiber Node Structure (conceptual):**
```javascript
// Simplified representation of a Fiber node
const fiber = {
  // Identity
  tag: FunctionComponent, // Type of component
  type: MyComponent,       // The component function or class
  key: null,               // React key

  // Tree structure (linked list)
  return: parentFiber,     // Parent Fiber
  child: firstChildFiber,  // First child Fiber
  sibling: nextSiblingFiber, // Next sibling Fiber
  index: 0,                // Position among siblings

  // State and props
  pendingProps: {},        // Props for the current render
  memoizedProps: {},       // Props from the last committed render
  memoizedState: null,     // State from the last committed render

  // Effects
  updateQueue: null,       // Queue of state updates
  flags: 0,                // Side-effect flags (Placement, Update, Deletion)
  subtreeFlags: 0,         // Aggregated flags of the subtree

  // Double buffering
  alternate: otherFiber,   // Link to the other tree's Fiber

  // Output
  stateNode: domNode,      // The DOM node or component instance
};
```

**Component Breakdown:**
- `tag` / `type`: Identify the kind of component.
- `return` / `child` / `sibling`: Form the linked list structure that replaces recursion.
- `memoizedProps` / `memoizedState`: The last committed props and state.
- `pendingProps`: The props for the current render.
- `flags` / `subtreeFlags`: Bitmasks indicating what work needs to be done during commit.
- `alternate`: Points to the corresponding Fiber in the other tree (current ↔ work-in-progress).

**Two-Tree (Double Buffering) Model:**

```javascript
// Current tree (visible on screen)
//         App
//        /   \
//    Header  Main

// Work-in-progress tree (being built)
//         App
//        /   \
//    Header  Main (updated)

// When work-in-progress is complete:
// - React swaps the root's current pointer to the work-in-progress tree
// - The old current tree becomes the new work-in-progress tree for the next render
```

**Component Breakdown:**
- React never mutates the current tree during rendering.
- All changes are made to the work-in-progress tree.
- When the work-in-progress tree is complete, React commits it by atomically swapping the trees.
- This ensures that the user never sees a partially updated UI.

**Render Phase vs Commit Phase:**

```javascript
// RENDER PHASE (interruptible)
// - Build the work-in-progress Fiber tree
// - Call component functions
// - Compute the diff between current and work-in-progress
// - Set flags for changes (Placement, Update, Deletion)

// COMMIT PHASE (not interruptible)
// - Apply DOM mutations (insertions, updates, deletions)
// - Run layout Effects (useLayoutEffect)
// - Swap current and work-in-progress trees
// - Schedule passive Effects (useEffect)
```

**Syntax Rules:**
- The render phase must be pure; it may be called multiple times, paused, or discarded.
- The commit phase is synchronous and never interrupted.
- `useLayoutEffect` runs during commit, before the browser paints.
- `useEffect` runs after commit, asynchronously.
- The `alternate` pointer links the two trees; React reuses Fiber nodes across renders to minimise allocations.
- Work is processed in a loop (not recursively), checking `shouldYield()` after each Fiber.

**Constraints and Limitations:**
- Fiber is an internal implementation detail; its API is not public and may change.
- Understanding Fiber is not required for using React, but it explains why certain patterns (like pure renders) are important.
- The double-buffering model doubles memory usage for the Fiber tree, though Fibers are lightweight objects.
- Fiber nodes are reused across renders via the `alternate` pointer, reducing garbage collection pressure.
- The commit phase cannot be interrupted; long commits can still block the main thread.

### Annotated Code Examples

**Example 1: Demonstrating Pure Render Requirements**

```jsx
import React, { useState } from 'react';

// ❌ BAD: Impure render — mutates external state
let renderCount = 0;

function ImpureComponent() {
  renderCount++; // Side effect during render!
  return <p>Render count: {renderCount}</p>;
}

// ✅ GOOD: Pure render — no side effects
function PureComponent() {
  const [count, setCount] = useState(0);

  // Side effect is in an event handler, not during render
  function handleClick() {
    setCount(c => c + 1);
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export { ImpureComponent, PureComponent };
```

**Expected Output:** In StrictMode, `ImpureComponent` renders twice, causing `renderCount` to increment by 2 on each render. `PureComponent` behaves predictably because its render is pure.

**Why This Output Occurs:** React's concurrent renderer may call a component's render function multiple times (e.g., in StrictMode, or when a render is interrupted and restarted). If the render function has side effects like `renderCount++`, those side effects accumulate incorrectly. Pure renders ensure that the same input always produces the same output, regardless of how many times React calls the function.

**Example 2: `useLayoutEffect` vs `useEffect` Timing**

```jsx
import React, { useState, useEffect, useLayoutEffect } from 'react';

function TimingDemo() {
  const [value, setValue] = useState('initial');

  // useLayoutEffect runs synchronously after DOM mutations, before paint
  useLayoutEffect(() => {
    console.log('useLayoutEffect: DOM is updated, before paint');
  }, [value]);

  // useEffect runs asynchronously after paint
  useEffect(() => {
    console.log('useEffect: after paint');
  }, [value]);

  console.log('Render: component function called');

  return (
    <div>
      <p>{value}</p>
      <button onClick={() => setValue('updated')}>Update</button>
    </div>
  );
}

export default TimingDemo;
```

**Expected Output (console order on initial mount):**
```
Render: component function called
useLayoutEffect: DOM is updated, before paint
useEffect: after paint
```

**Expected Output (console order on update):**
```
Render: component function called
useLayoutEffect: DOM is updated, before paint
useEffect: after paint
```

**Why This Output Occurs:** The render phase (component function) runs first, building the work-in-progress Fiber tree. Then the commit phase applies DOM mutations and runs `useLayoutEffect` synchronously. After the browser paints, `useEffect` runs asynchronously. This ordering is guaranteed by the Fiber architecture and is crucial for avoiding visual flickers when reading layout information.

### Real-World Cases

- **Animation libraries:** Using `useLayoutEffect` to measure DOM elements before painting.
- **Tooltips and popovers:** Positioning elements based on their rendered size.
- **Data visualisation:** Reading SVG dimensions after render.
- **Smooth transitions:** Ensuring DOM updates are applied atomically.
- **Debugging performance:** Understanding why renders happen multiple times in StrictMode.
- **Third-party integrations:** Knowing when to access the DOM (after commit).

---

## Core Concept 4: Streaming Architecture Compatibility

### Definitions

**Core Definition:** Streaming architecture compatibility refers to the integration of concurrent rendering with Server-Side Rendering (SSR) and streaming HTML, enabling progressive content delivery and selective hydration.

**Technical Definition:** React 18 introduced `renderToPipeableStream` (Node.js) and `renderToReadableStream` (Web Streams) for streaming SSR. These APIs render React components to HTML on the server and stream the HTML to the client in chunks as it becomes available. Suspense boundaries on the server emit placeholder HTML and inline `<script>` tags that, when executed on the client, replace the placeholder with the actual content once it's ready. Concurrent rendering on the client enables **selective hydration**: React prioritises hydrating the parts of the page the user is interacting with, rather than hydrating the entire page before responding to input. This integration allows users to see content faster, interact with parts of the page while others are still loading, and reduces Time to First Byte (TTFB) and First Contentful Paint (FCP).

**Beginner-Friendly Explanation:** Traditionally, when you use server-side rendering, the server has to generate the entire HTML page before sending anything to the browser. If one part of the page is slow, the whole page waits. Streaming changes this: the server sends the page in pieces as soon as each piece is ready. The browser can start showing content immediately, and when the slow parts are ready, they're streamed in and swapped into place. Concurrent rendering makes this possible on the client by allowing React to hydrate (make interactive) the parts the user is interacting with first.

### Purposes

- To reduce Time to First Byte (TTFB) by sending HTML as it's generated.
- To improve First Contentful Paint (FCP) by showing content before all data is ready.
- To enable selective hydration, prioritising interactive parts of the page.
- To allow users to interact with parts of the page while others are still loading.
- To synchronise server-rendered HTML with client-side concurrent rendering.
- To support React Server Components (RSC) with streaming.

### Syntax Rules and Structure

**Server-Side Streaming with `renderToPipeableStream` (Node.js):**
```jsx
import { renderToPipeableStream } from 'react-dom/server';
import express from 'express';
import App from './App';

const app = express();

app.get('/', (req, res) => {
  let didError = false;

  const { pipe, abort } = renderToPipeableStream(
    <App />,
    {
      bootstrapScripts: ['/main.js'],
      onShellReady() {
        // The shell (everything outside Suspense) is ready
        res.statusCode = didError ? 500 : 200;
        res.setHeader('Content-Type', 'text/html');
        pipe(res); // Stream the HTML to the client
      },
      onShellError(error) {
        // The shell itself failed to render
        res.statusCode = 500;
        res.send('<h1>Something went wrong</h1>');
      },
      onError(error) {
        didError = true;
        console.error(error);
      },
    }
  );

  // Abort after 10 seconds if not finished
  setTimeout(abort, 10000);
});

app.listen(3000);
```

**Component Breakdown:**
- `renderToPipeableStream(<App />, options)`: Renders the app to a stream.
- `bootstrapScripts`: The client-side JavaScript bundles to load.
- `onShellReady`: Called when the initial HTML shell (outside Suspense) is ready. The stream is piped to the response.
- `onShellError`: Called if the shell itself fails.
- `onError`: Called for any error during rendering.
- `pipe(res)`: Streams the HTML to the client.
- `abort()`: Aborts the stream if it takes too long.

**Client-Side Hydration with `hydrateRoot`:**
```jsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

// Hydrate the server-rendered HTML
hydrateRoot(document, <App />);
```

**Component Breakdown:**
- `hydrateRoot(document, <App />)`: Attaches React to the server-rendered HTML, making it interactive.
- With concurrent rendering, hydration is selective: React prioritises hydrating the parts the user interacts with.

**Suspense Boundaries for Streaming:**
```jsx
import { Suspense } from 'react';
import Comments from './Comments';
import Recommendations from './Recommendations';

function ProductPage({ productId }) {
  return (
    <div>
      {/* This content is part of the shell and streams immediately */}
      <h1>Product Details</h1>
      <ProductInfo productId={productId} />

      {/* This content streams in when ready */}
      <Suspense fallback={<p>Loading comments...</p>}>
        <Comments productId={productId} />
      </Suspense>

      {/* This content streams in independently */}
      <Suspense fallback={<p>Loading recommendations...</p>}>
        <Recommendations productId={productId} />
      </Suspense>
    </div>
  );
}
```

**Component Breakdown:**
- The `<h1>` and `<ProductInfo>` are outside Suspense, so they're part of the shell and stream immediately.
- `<Comments>` and `<Recommendations>` are inside Suspense boundaries. On the server, React emits the fallback HTML and a placeholder. When the data is ready, React streams the actual content and inline scripts that replace the fallback.
- On the client, React hydrates the shell first, then hydrates each Suspense boundary as its content arrives.

**Syntax Rules:**
- Use `renderToPipeableStream` (Node.js) or `renderToReadableStream` (Web Streams) for streaming SSR. Avoid `renderToString`, which is synchronous and non-streaming.
- Place Suspense boundaries around parts of the UI that can load independently.
- The shell (content outside Suspense) streams first; Suspense fallbacks stream next; actual content streams as it becomes ready.
- On the client, use `hydrateRoot` (not `createRoot`) to hydrate server-rendered HTML.
- Selective hydration is automatic with concurrent rendering; React prioritises hydration based on user interactions.
- Use `bootstrapScripts` to load the client bundle; the scripts are deferred until the shell is ready.

**Constraints and Limitations:**
- Streaming SSR requires a server environment that supports streams (Node.js 16+, Deno, Bun, or edge runtimes).
- `renderToString` is still available but does not support streaming or Suspense.
- Streaming SSR does not work with `ReactDOM.render`; it requires `hydrateRoot`.
- Selective hydration requires the client bundle to be loaded; until then, the page is not interactive.
- Errors during streaming are handled by the nearest Error Boundary on the server; if none exists, the stream may be aborted.
- Some third-party libraries may not be compatible with streaming SSR.

### Annotated Code Examples

**Example 1: Streaming SSR with Suspense (Node.js/Express)**

```jsx
// server.js
import express from 'express';
import { renderToPipeableStream } from 'react-dom/server';
import React, { Suspense } from 'react';

// Simulated async data fetch
function fetchData(id) {
  return new Promise(resolve => {
    setTimeout(() => resolve({ id, content: `Data for ${id}` }), 1000);
  });
}

// A component that suspends on the server
async function SlowComponent({ id }) {
  const data = await fetchData(id);
  return <p>{data.content}</p>;
}

function App() {
  return (
    <html>
      <head>
        <title>Streaming SSR Demo</title>
      </head>
      <body>
        <h1>Streaming SSR</h1>
        <p>This content streams immediately.</p>

        <Suspense fallback={<p>Loading slow content...</p>}>
          <SlowComponent id="A" />
        </Suspense>

        <Suspense fallback={<p>Loading another slow section...</p>}>
          <SlowComponent id="B" />
        </Suspense>

        <script src="/client.js" defer></script>
      </body>
    </html>
  );
}

const server = express();

server.get('/', (req, res) => {
  const { pipe, abort } = renderToPipeableStream(<App />, {
    onShellReady() {
      res.setHeader('Content-Type', 'text/html');
      pipe(res);
    },
    onShellError(error) {
      res.statusCode = 500;
      res.send('<h1>Something went wrong</h1>');
    },
    onError(error) {
      console.error(error);
    },
  });

  setTimeout(abort, 10000);
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

**Expected Output:** The browser receives the shell HTML (the `<h1>`, the immediate `<p>`, and the Suspense fallbacks) almost immediately. The "Loading slow content..." and "Loading another slow section..." messages appear. After 1 second, the actual content for each Suspense boundary streams in and replaces the fallbacks.

**Why This Output Occurs:** The `renderToPipeableStream` API renders the shell (everything outside Suspense) first and pipes it to the response. Suspense boundaries emit their fallbacks. When the `SlowComponent` Promises resolve on the server, React streams the actual HTML along with inline scripts that replace the fallback content on the client. This allows the browser to show content progressively rather than waiting for the entire page.

**Example 2: Selective Hydration with Concurrent Rendering**

```jsx
// client.js
import { hydrateRoot } from 'react-dom/client';
import React, { Suspense } from 'react';

function App() {
  return (
    <html>
      <body>
        <h1>Selective Hydration</h1>
        <Suspense fallback={<p>Loading comments...</p>}>
          <Comments />
        </Suspense>
        <Suspense fallback={<p>Loading sidebar...</p>}>
          <Sidebar />
        </Suspense>
      </body>
    </html>
  );
}

function Comments() {
  // Simulate a component that takes time to hydrate
  return <div>Comments section (interactive)</div>;
}

function Sidebar() {
  return <div>Sidebar (interactive)</div>;
}

// Hydrate the server-rendered HTML
hydrateRoot(document, <App />);
```

**Expected Output:** The server-rendered HTML appears immediately. As the client bundle loads, React hydrates the shell first. If the user clicks on the "Comments" section before the "Sidebar" is hydrated, React prioritises hydrating the Comments section, making it interactive sooner.

**Why This Output Occurs:** With concurrent rendering and `hydrateRoot`, React performs selective hydration. It listens for user interactions (clicks, keypresses) on parts of the page that haven't been hydrated yet. When an interaction occurs, React prioritises hydrating that part of the tree, allowing the user to interact with it sooner. This is possible because Suspense boundaries create hydration boundaries that React can prioritise independently.

### Real-World Cases

- **E-commerce product pages:** Streaming the product shell immediately while reviews and recommendations load.
- **News websites:** Streaming the article content while comments and related articles load.
- **Dashboards:** Streaming the dashboard shell while individual widgets load.
- **Social media feeds:** Streaming the feed layout while posts load progressively.
- **Next.js App Router:** Uses streaming SSR with Suspense by default for Server Components.
- **Remix:** Uses streaming SSR for deferred data loading.

---

## References

- React Official Documentation – Concurrent Rendering: https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – `renderToPipeableStream`: https://react.dev/reference/react-dom/server/renderToPipeableStream
- React Official Documentation – `renderToReadableStream`: https://react.dev/reference/react-dom/server/renderToReadableStream
- React Official Documentation – `hydrateRoot`: https://react.dev/reference/react-dom/client/hydrateRoot
- React Official Documentation – Suspense: https://react.dev/reference/react/Suspense
- React Working Group – Concurrent React: https://github.com/reactwg/react-18/discussions/64
- React Working Group – New Suspense SSR Architecture: https://github.com/reactwg/react-18/discussions/37
- React Working Group – `useDeferredValue` and `startTransition`: https://github.com/reactwg/react-18/discussions/129
- React Working Group – Selective Hydration: https://github.com/reactwg/react-18/discussions/130
- React Fiber Architecture (Andrew Clark): https://github.com/acdlite/react-fiber-architecture
- React 18 Release Notes: https://react.dev/blog/2022/03/29/react-v18
- React 19 Release Notes: https://react.dev/blog/2024/12/05/react-19
- MDN Web Docs – MessageChannel: https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel
- MDN Web Docs – Streams API: https://developer.mozilla.org/en-US/docs/Web/API/Streams_API
- MDN Web Docs – Scheduler API: https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API
- web.dev – Interaction to Next Paint (INP): https://web.dev/articles/inp
- web.dev – Time to First Byte (TTFB): https://web.dev/articles/ttfb
- web.dev – First Contentful Paint (FCP): https://web.dev/articles/fcp
- Next.js – Streaming with Suspense: https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- Epic React – How React Suspense Works Under the Hood: https://www.epicreact.dev/how-react-suspense-works-under-the-hood-throwing-promises-and-declarative-loading-states
- Steve Kinney – The Fiber Architecture: https://stevekinney.com/courses/react-performance/the-fiber-architecture
- Steve Kinney – Concurrent Rendering: https://stevekinney.com/courses/react-performance/concurrent-rendering
- Syncfusion – React 19 Concurrent Rendering: https://www.syncfusion.com/blogs/post/react-19-concurrent-rendering
- LogRocket – A Guide to React 18's Concurrent Rendering: https://blog.logrocket.com/react-18-concurrent-rendering/
- Smashing Magazine – A Deep Dive into React Fiber: https://www.smashingmagazine.com/2020/07/deep-dive-react-fiber/