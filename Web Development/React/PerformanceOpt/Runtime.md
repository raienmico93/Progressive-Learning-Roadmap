# React Runtime Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React runtime performance is the efficiency with which a React application executes JavaScript, manages memory, and responds to user interactions after it has loaded.

**Technical Definition:** Runtime performance in React concerns the main-thread work performed after the initial load: executing component functions, processing event handlers, running effects, managing memory, and performing garbage collection. It is measured primarily by **Interaction to Next Paint (INP)** — the latency from a user interaction to the next painted frame — and by memory stability over time. JavaScript is single-threaded, so any long-running task on the main thread blocks input handling, animation, and rendering. Runtime performance therefore depends on keeping individual tasks short (under 50ms), avoiding memory leaks that degrade performance over time, and offloading CPU-intensive work to Web Workers.

**Beginner-Friendly Explanation:** Runtime performance is how fast and smoothly your app runs after it has loaded. It is not about downloading files quickly — that is loading performance. Runtime performance is about what happens when a user clicks a button, types in a field, or scrolls a page. If the app freezes for a moment before responding, that is a runtime performance problem. It also covers memory: if your app slowly uses more and more memory because it forgets to clean up after itself, it will eventually slow down or crash.

### Key Characteristics

- **Single-threaded constraint:** JavaScript runs on one main thread; long tasks block everything else.
- **Long tasks are the enemy:** Any task exceeding 50ms blocks the main thread and degrades INP.
- **Memory leaks degrade over time:** Event listeners, timers, and subscriptions that are not cleaned up keep memory and CPU usage rising.
- **Cleanup is mandatory:** Every effect that starts something asynchronous must return a cleanup function.
- **Offloading is the escape hatch:** Web Workers move CPU-heavy work to a separate thread, keeping the UI responsive.
- **Concurrent features help:** `startTransition` and `useDeferredValue` mark non-urgent updates as interruptible.

### Prerequisites

- React function components, `useEffect`, and `useRef`.
- Understanding of the browser event loop and the main thread.
- Familiarity with `AbortController` and asynchronous JavaScript.
- Basic knowledge of Chrome DevTools Performance and Memory panels.

### Related Programming Areas

- **Loading Performance** — how quickly the app becomes interactive.
- **Rendering Performance** — how efficiently React reconciles and commits updates.
- **Core Web Vitals** — INP is the primary runtime metric.
- **Memory Management** — garbage collection, leaks, and heap profiling.

---

## 1. JavaScript Execution

### Definitions

**Core Definition:** JavaScript execution is the process by which the browser's engine parses and runs JavaScript code on the main thread.

**Technical Definition:** JavaScript execution in the browser is single-threaded and event-loop-driven. The engine executes tasks from a queue; each task runs to completion before the next task begins. Long-running tasks block the main thread, preventing the browser from processing user input, running animations, or painting frames. React's rendering, event handlers, and effects all execute as tasks on this main thread. Performance therefore depends on keeping individual tasks short and yielding control back to the browser between chunks of work.

**Beginner-Friendly Explanation:** JavaScript execution is your code actually running. Because the browser runs JavaScript on a single thread, if one piece of code takes a long time, everything else — clicks, typing, scrolling — has to wait.

### Purposes

- To understand why long-running code blocks the UI.
- To identify where execution time is spent (event handlers, effects, rendering).
- To apply techniques that keep individual tasks short.

### Syntax Rules and Structure

**Execution Time Budgets:**

| Task Type | Target | Rationale |
|-----------|--------|-----------|
| Event handler | < 50ms | Avoids long task; keeps INP < 200ms |
| Effect | < 50ms | Avoids blocking paint |
| Render (React) | < 16ms per frame | Maintains 60fps |
| Chunked work | Yield every ~5ms | Allows input processing |

**Syntax Rules:**

1. Measure execution time with `performance.mark` / `performance.measure` or the Performance panel.
2. Avoid synchronous layout reads in loops (layout thrashing).
3. Break up long loops with yielding (`scheduler.yield()` or `setTimeout`).
4. Move CPU-heavy work to Web Workers.

### Annotated Code Examples

```jsx
function ExpensiveHandler() {
  async function handleClick() {
    performance.mark('start');
    // Simulate expensive synchronous work
    const result = heavyComputation();
    performance.mark('end');
    performance.measure('heavyComputation', 'start', 'end');
  }

  return <button onClick={handleClick}>Run</button>;
}
```

**Expected Output:** A performance measure named `"heavyComputation"` appears in DevTools, showing the execution duration.

**Why this output:** `performance.mark` records timestamps, and `performance.measure` computes the duration between them, making the execution time visible.

### Real-World Cases

- **Data processing:** Parsing large JSON payloads on the main thread.
- **Complex calculations:** Financial or scientific computations triggered by user input.
- **Layout thrashing:** Reading layout properties inside loops, forcing synchronous reflows.

### References

- Steve Kinney — Memory Management Deep Dive — https://stevekinney.com/courses/react-performance/memory-management-deep-dive
- React Academy — Reducing INP: Event Handler Optimisation — https://www.coddykit.com

---

## 2. Memory Usage

### Definitions

**Core Definition:** Memory usage is the amount of RAM an application consumes, including objects, closures, DOM nodes, and internal browser structures.

**Technical Definition:** JavaScript memory is managed automatically through **mark-and-sweep garbage collection**. Objects are allocated when created and freed when no longer reachable from a root (the global object, the current stack, or DOM references). In React, memory usage is shaped by component lifecycle (mount → update → unmount), closure capture (hooks capture variables from render scope), reference stability (new objects created on every render), and effect cleanup (manual cleanup required for side effects). Stable references (via `useCallback` and `useMemo`) reduce memory pressure by avoiding unnecessary allocations.

**Beginner-Friendly Explanation:** Memory usage is how much RAM your app takes up. JavaScript cleans up after itself automatically, but only if you stop referencing things you no longer need. If you keep references alive — through listeners, timers, or subscriptions — the memory cannot be reclaimed.

### Purposes

- To understand how garbage collection works in JavaScript.
- To identify patterns that increase memory pressure.
- To keep memory stable over long sessions.

### Syntax Rules and Structure

**Memory Lifecycle:**

| Phase | Description |
|-------|-------------|
| Allocation | Memory is allocated when objects are created |
| Usage | Memory is tracked by references |
| Deallocation | Garbage collection frees unreachable objects |

**Syntax Rules:**

1. Use stable references (`useCallback`, `useMemo`) to reduce allocations.
2. Extract values from props instead of capturing the entire props object in closures.
3. Avoid creating large objects inside render when they are only needed conditionally.
4. Profile with Chrome DevTools Memory panel and Performance Monitor.

### Annotated Code Examples

```jsx
// ✅ Stable references reduce memory pressure
function MemoryEfficient({ data }) {
  const stableCallback = useCallback(() => {
    // Uses only what it needs
  }, []);

  const stableStyle = useMemo(() => ({ color: 'red', margin: 10 }), []);

  return <div style={stableStyle} onClick={stableCallback} />;
}
```

**Expected Output:** The component allocates fewer objects per render, reducing garbage collection pressure.

**Why this output:** `useCallback` and `useMemo` keep the function and object references stable across renders, avoiding repeated allocations.

### Real-World Cases

- **Long-running dashboards:** Stable references keep memory flat over hours of use.
- **Lists with frequent updates:** Avoiding inline object creation reduces GC pauses.

### References

- Steve Kinney — Memory Management Deep Dive — https://stevekinney.com/courses/react-performance/memory-management-deep-dive
- CoreUI — How to fix memory leaks in React — https://coreui.io

---

## 3. Memory Leaks

### Definitions

**Core Definition:** A memory leak is memory that is allocated but never freed because something still holds a reference to it, causing memory usage to grow over time.

**Technical Definition:** A memory leak in React occurs when a component unmounts but something still holds a reference to its resources — an event listener, a timer, a subscription, or a closure. The garbage collector cannot free the memory because the reference is reachable from a root. Common leak patterns include event listeners not removed on unmount, timers not cleared, subscriptions not unsubscribed, and asynchronous callbacks that call `setState` on an unmounted component. The primary fix is returning a cleanup function from `useEffect`.

**Beginner-Friendly Explanation:** A memory leak is like forgetting to throw something away. The object is no longer used, but something is still holding onto it, so the browser cannot free it. Over time, leaks make the app slower and can eventually crash the tab.

### Purposes

- To prevent memory growth and eventual crashes.
- To keep performance stable over long sessions.
- To understand which patterns create persistent references.

### Syntax Rules and Structure

**Common Leak Sources:**

| Source | Mechanism | Fix |
|--------|-----------|-----|
| Event listeners | Listener remains after unmount | Remove in cleanup |
| Timers | Interval/timeout keeps firing | Clear in cleanup |
| Subscriptions | WebSocket/EventEmitter stays open | Unsubscribe/close in cleanup |
| Fetch requests | Response updates unmounted state | `AbortController` in cleanup |
| Closures | Captures large objects | Extract only needed values |

**Syntax Rules:**

1. Every `useEffect` that starts an asynchronous operation must return a cleanup function.
2. Remove event listeners with the exact same function reference used to add them.
3. Clear timers with `clearInterval` / `clearTimeout`.
4. Abort fetches with `AbortController`.
5. Unsubscribe from observables and event emitters.

### Annotated Code Examples

```jsx
function SearchBox() {
  useEffect(() => {
    const handleResize = () => { /* ... */ };
    window.addEventListener('resize', handleResize);
    // ✅ Cleanup removes the listener on unmount
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return <input />;
}
```

**Expected Output:** The resize listener is removed when the component unmounts, preventing a leak.

**Why this output:** The cleanup function runs on unmount and removes the listener using the same `handleResize` reference.

### Real-World Cases

- **SPA route changes:** Components unmount and remount; missing cleanup accumulates listeners.
- **Chat applications:** WebSocket connections that are never closed.
- **Polling components:** Intervals that keep running after the component is gone.

### References

- CoreUI — How to fix memory leaks in React — https://coreui.io
- React Memory Leaks Guide — https://github.com/curiositech/windags-skills
- React/TypeScript Anti-Pattern: useEffect Cleanup — https://github.com/glennguilloux/llm-knowledge-base

---

### 3.1 Event Listener Leaks

**Definitions:**

- **Core Definition:** An event listener leak occurs when a listener added to `window`, `document`, or a DOM element is not removed when the component unmounts.
- **Technical Definition:** The listener callback holds a reference to the component's scope, preventing garbage collection. If the listener is added on every render without a dependency array, multiple listeners accumulate, each firing on every event.
- **Beginner-Friendly Explanation:** If you add a listener and never remove it, it keeps running even after the component is gone, and it keeps the component in memory.

**Purposes:** To prevent accumulating listeners and retained component memory.

**Syntax Rules:** Always remove the listener in the `useEffect` cleanup with the same function reference. Include a dependency array to avoid adding listeners on every render.

**Annotated Code Example:**

```jsx
useEffect(() => {
  const handleResize = () => { /* ... */ };
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, [handleResize]);
```

**Expected Output:** One listener is added and removed; no accumulation.

**Why this output:** The dependency array ensures the effect runs only when `handleResize` changes, and the cleanup removes the listener.

**Real-World Cases:** Resize handlers, scroll handlers, keyboard shortcuts.

**References:** CoreUI — https://coreui.io; React Memory Leaks Guide — https://github.com/curiositech/windags-skills

---

### 3.2 Timer Leaks

**Definitions:**

- **Core Definition:** A timer leak occurs when a `setInterval` or `setTimeout` is not cleared when the component unmounts.
- **Technical Definition:** The timer continues to fire, potentially calling `setState` on an unmounted component, which wastes CPU and may produce warnings. The interval ID must be stored and passed to `clearInterval`.
- **Beginner-Friendly Explanation:** A timer keeps ticking even after the component is gone. It wastes CPU and can cause errors if it tries to update state.

**Purposes:** To stop timers from running after unmount.

**Syntax Rules:** Store the timer ID and call `clearInterval` or `clearTimeout` in the cleanup.

**Annotated Code Example:**

```jsx
useEffect(() => {
  const intervalId = setInterval(() => setTime(new Date()), 1000);
  return () => clearInterval(intervalId);
}, []);
```

**Expected Output:** The interval stops when the component unmounts.

**Why this output:** The cleanup calls `clearInterval` with the stored ID, stopping the timer.

**Real-World Cases:** Clocks, polling, countdown timers.

**References:** CoreUI — https://coreui.io; React Memory Leaks Guide — https://github.com/curiositech/windags-skills

---

### 3.3 Subscription Leaks

**Definitions:**

- **Core Definition:** A subscription leak occurs when a WebSocket, EventEmitter, or observable subscription is not closed or unsubscribed when the component unmounts.
- **Technical Definition:** The subscription continues to receive messages and invoke callbacks that reference the unmounted component, preventing garbage collection and potentially causing state updates on an unmounted component.
- **Beginner-Friendly Explanation:** If you subscribe to something and never unsubscribe, it keeps sending data to a component that no longer exists.

**Purposes:** To close connections and unsubscribe from data streams on unmount.

**Syntax Rules:** Return a cleanup that calls `.close()` (WebSocket), `.unsubscribe()` (observables), or `.off()` (EventEmitter).

**Annotated Code Example:**

```jsx
useEffect(() => {
  const ws = new WebSocket('wss://example.com/prices');
  ws.onmessage = (event) => setPrice(JSON.parse(event.data).price);
  return () => ws.close(); // ✅ Close on unmount
}, []);
```

**Expected Output:** The WebSocket is closed when the component unmounts.

**Why this output:** The cleanup calls `ws.close()`, terminating the connection.

**Real-World Cases:** Live price feeds, chat rooms, real-time notifications.

**References:** React/TypeScript Anti-Pattern: useEffect Cleanup — https://github.com/glennguilloux/llm-knowledge-base; CoreUI — https://coreui.io

---

## 4. Long Tasks

### Definitions

**Core Definition:** A long task is any JavaScript task that occupies the main thread for more than 50 milliseconds.

**Technical Definition:** Long tasks block the event loop, delaying the browser's ability to process input, run animations, and paint frames. The browser's Long Tasks API reports tasks exceeding 50ms. In React, long tasks commonly arise from expensive event handlers, large synchronous state updates, and heavy rendering work. Long tasks directly inflate INP.

**Beginner-Friendly Explanation:** A long task is a piece of code that hogs the browser for too long. While it runs, nothing else can happen — clicks are ignored, the page does not scroll, and animations freeze.

### Purposes

- To identify and eliminate main-thread blocking.
- To keep INP under 200ms.
- To understand why the UI freezes during certain operations.

### Syntax Rules and Structure

**Long Task Thresholds:**

| Duration | Impact |
|----------|--------|
| < 50ms | Acceptable |
| 50–100ms | Noticeable delay; INP risk |
| > 100ms | Severe blocking; poor INP |
| > 1s | UI freeze |

**Syntax Rules:**

1. Break long tasks into chunks under 50ms.
2. Yield to the browser between chunks (`scheduler.yield()` or `setTimeout`).
3. Move CPU-bound work to Web Workers.
4. Avoid synchronous layout reads in loops.

### Annotated Code Examples

```jsx
async function processInChunks(items) {
  const CHUNK_SIZE = 100;
  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);
    processChunk(chunk);
    // ✅ Yield to the browser between chunks
    await new Promise(r => setTimeout(r, 0));
  }
}
```

**Expected Output:** The work is processed in chunks, allowing the browser to handle input between chunks.

**Why this output:** The `await setTimeout(0)` yields control back to the event loop, breaking the long task into shorter ones.

### Real-World Cases

- **Large list rendering:** Processing thousands of items synchronously.
- **Data transformation:** Parsing and mapping large datasets.
- **Complex calculations:** Cryptographic or scientific computations.

### References

- React Academy — Reducing INP: Event Handler Optimisation — https://www.coddykit.com
- Neptune Infotech — INP Optimization for React — https://neptuneinfotech.in

---

## 5. Task Splitting

### Definitions

**Core Definition:** Task splitting is the practice of breaking a long-running task into smaller chunks that yield to the browser between them.

**Technical Definition:** Task splitting (also called yielding or time-slicing) divides a long task into shorter segments separated by `await` points that return control to the event loop. This allows the browser to process pending input, run animations, and paint frames between chunks. React 19's concurrent scheduler automatically slices rendering work into ~5ms chunks and yields between them, preventing long renders from blocking input. Developers can also use `scheduler.yield()` or `setTimeout` for manual task splitting.

**Beginner-Friendly Explanation:** Task splitting is breaking a big job into smaller pieces and letting the browser breathe between them. It is like taking breaks while doing a long chore so you can answer the door if someone knocks.

### Purposes

- To prevent long tasks from blocking the main thread.
- To keep input responsive during heavy processing.
- To improve INP and animation smoothness.
- To complement React's concurrent rendering.

### Syntax Rules and Structure

**Task Splitting Techniques:**

| Technique | Mechanism |
|-----------|-----------|
| `scheduler.yield()` | Yields to the event loop, prioritising pending input |
| `setTimeout(0)` | Yields to the event loop |
| `requestIdleCallback` | Runs work during idle time |
| `startTransition` | Marks React state updates as interruptible |
| `useDeferredValue` | Defers expensive re-renders |

**Syntax Rules:**

1. Yield every few milliseconds or after a fixed number of items.
2. Use `scheduler.yield()` for the best input responsiveness.
3. Use `startTransition` for React state updates that are not urgent.
4. Combine manual yielding with React's concurrent features.

### Annotated Code Examples

```jsx
async function processWithYield(items) {
  for (let i = 0; i < items.length; i++) {
    processItem(items[i]);
    // ✅ Yield every 100 items
    if (i % 100 === 0) {
      await scheduler.yield();
    }
  }
}
```

**Expected Output:** The browser can process input between chunks of work.

**Why this output:** `scheduler.yield()` returns control to the event loop, allowing pending interactions to run before the next chunk.

### Real-World Cases

- **Search filtering:** Filtering large datasets without blocking the input.
- **Data visualisation:** Processing data points before rendering a chart.
- **Bulk updates:** Applying many state updates without freezing.

### References

- React 19 concurrent mode — https://github.com/domidex01/tour-kit
- React Academy — Reducing INP — https://www.coddykit.com
- Neptune Infotech — INP Optimization for React — https://neptuneinfotech.in

---

## 6. Web Workers

### Definitions

**Core Definition:** A Web Worker is a JavaScript script that runs in a background thread, separate from the main thread, enabling parallel execution without blocking the UI.

**Technical Definition:** Web Workers run JavaScript in a separate thread with its own global context. Workers cannot access the DOM, `window`, or `localStorage`; they communicate with the main thread via `postMessage` and `onmessage`. They are ideal for CPU-intensive tasks — data parsing, image processing, complex calculations — because they keep the main thread free for rendering and input. Workers have a separate heap (typically 10–50MB baseline) and communicate with ~0.1–0.3ms message latency.

**Beginner-Friendly Explanation:** A Web Worker is a helper thread that does heavy work in the background while the main thread stays free to respond to the user. The worker cannot touch the page directly, so it sends results back to the main thread.

### Purposes

- To move CPU-intensive work off the main thread.
- To keep the UI responsive during heavy processing.
- To achieve true parallelism on multi-core CPUs.
- To prevent long tasks from degrading INP.

### Syntax Rules and Structure

**Creating a Worker:**

```js
const worker = new Worker(new URL('./heavy-worker.js', import.meta.url));
worker.postMessage(data);
worker.onmessage = (e) => setResult(e.data);
```

**Worker Capabilities:**

| Can do | Cannot do |
|--------|-----------|
| Run CPU-intensive calculations | Access the DOM |
| Make network requests | Access `window` or globals |
| Access IndexedDB | Use `localStorage` |
| Run WebAssembly | Use React components or hooks |

**Syntax Rules:**

1. Create the worker with `new Worker(new URL('./worker.js', import.meta.url))`.
2. Send data with `worker.postMessage(data)`.
3. Receive results with `worker.onmessage`.
4. Terminate workers when no longer needed to avoid memory leaks.
5. Handle errors with `worker.onerror`.

### Annotated Code Examples

```jsx
const worker = new Worker(new URL('./heavy-worker.js', import.meta.url));

function processData(data) {
  return new Promise((resolve) => {
    worker.postMessage(data);
    worker.onmessage = (e) => resolve(e.data);
  });
}

async function handleClick() {
  const result = await processData(largeDataset);
  setResult(result); // Main thread stayed responsive
}
```

**Expected Output:** The heavy computation runs in the worker; the UI remains responsive while it completes.

**Why this output:** `postMessage` sends the data to the worker thread, which computes without blocking the main thread. `onmessage` resolves the Promise with the result.

### Real-World Cases

- **Image processing:** Applying filters or resizing in the background.
- **Data parsing:** Parsing large CSV or JSON files.
- **Cryptography:** Hashing or encryption operations.
- **Complex calculations:** Financial modelling or simulations.

### References

- Steve Kinney — Web Workers with React — https://stevekinney.com/courses/react-performance/web-workers-with-react
- Scaling React Apps with Parallelism — https://gitnation.com
- @okaygift/react-use-worker — https://www.npmjs.com/package/@okaygift/react-use-worker

---

## 7. Offloading Work

### Definitions

**Core Definition:** Offloading work is the practice of moving CPU-intensive tasks from the main thread to a Web Worker or other background context.

**Technical Definition:** Offloading moves work that does not need DOM access — data transformation, computation, parsing — to a background thread. The main thread remains free for rendering, input handling, and animation. Offloading is the most effective remedy for long tasks that cannot be split effectively, because it removes the work from the main thread entirely rather than merely interleaving it.

**Beginner-Friendly Explanation:** Offloading is giving heavy work to a helper thread so the main thread can stay responsive. It is like hiring someone to do the heavy lifting while you answer the phone.

### Purposes

- To eliminate long tasks caused by CPU-bound work.
- To keep INP low during heavy processing.
- To leverage multi-core CPUs.
- To prevent UI freezes during data-intensive operations.

### Syntax Rules and Structure

**Offloading Decision:**

| Work type | Offload? |
|-----------|----------|
| CPU-intensive calculation | Yes |
| Large data parsing | Yes |
| Image/video processing | Yes |
| DOM manipulation | No |
| React state updates | No (must run on main thread) |

**Syntax Rules:**

1. Offload only work that does not need DOM access.
2. Use `postMessage` for data transfer; consider transferable objects for large buffers.
3. Terminate workers when no longer needed.
4. Handle errors from the worker gracefully.

### Annotated Code Examples

```jsx
// Main thread
const worker = new Worker(new URL('./processor.js', import.meta.url));

function offloadData(data) {
  return new Promise((resolve, reject) => {
    worker.postMessage(data);
    worker.onmessage = (e) => resolve(e.data);
    worker.onerror = reject;
  });
}
```

```js
// processor.js (worker)
self.onmessage = (e) => {
  const result = heavyTransform(e.data);
  self.postMessage(result);
};
```

**Expected Output:** The heavy transformation runs in the worker; the main thread receives the result without blocking.

**Why this output:** The worker performs the computation in a separate thread and sends the result back via `postMessage`.

### Real-World Cases

- **Data dashboards:** Transforming large datasets before charting.
- **Search indexes:** Building an index in a worker while the UI remains interactive.
- **File processing:** Parsing uploaded files in the background.

### References

- Steve Kinney — Web Workers with React — https://stevekinney.com/courses/react-performance/web-workers-with-react
- React Performance Optimizer — https://github.com/curiositech/windags-skills

---

## 8. Runtime Performance Best Practices

### Definitions

**Core Definition:** Runtime performance best practices are the guidelines for keeping a React application responsive, memory-stable, and free of main-thread blocking after the initial load.

**Technical Definition:** Best practices combine **memory hygiene** (cleanup of listeners, timers, subscriptions, and fetches), **task management** (avoiding long tasks, splitting work, yielding), **offloading** (Web Workers for CPU-bound work), **concurrent features** (`startTransition`, `useDeferredValue`), and **continuous profiling** (Chrome DevTools Performance and Memory panels, React Profiler). The overarching principle is to measure first, fix the largest source of blocking or leakage, and verify the improvement.

**Beginner-Friendly Explanation:** Runtime performance is a set of habits: clean up everything you start, do not block the main thread for too long, give heavy work to a worker, use React's non-urgent update tools, and profile regularly. Do these consistently and your app stays fast.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Clean up every effect** | Return a cleanup from every `useEffect` that starts something async |
| **Remove listeners with matching references** | Use the same function for `addEventListener` and `removeEventListener` |
| **Clear timers** | `clearInterval` / `clearTimeout` in cleanup |
| **Abort fetches** | Use `AbortController` in cleanup |
| **Unsubscribe** | Close WebSockets, unsubscribe from observables |
| **Break long tasks** | Yield every few milliseconds or every N items |
| **Use `scheduler.yield()`** | Best input responsiveness during yielding |
| **Use `startTransition`** | Mark non-urgent React updates as interruptible |
| **Use `useDeferredValue`** | Defer expensive re-renders |
| **Offload CPU work** | Use Web Workers for data parsing, computation, image processing |
| **Terminate unused workers** | Avoid memory leaks from idle workers |
| **Profile with DevTools** | Memory panel (heap snapshots), Performance panel (long tasks) |
| **Track INP in RUM** | Measure real user responsiveness |

### Annotated Code Examples

**✅ Good Practices (Combined):**

```jsx
function LiveSearch() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        startTransition(() => setResults(data));
      })
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err);
      });

    return () => controller.abort(); // ✅ Cleanup
  }, [query]);

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ No cleanup: fetch continues after unmount
useEffect(() => {
  fetch('/api/data').then(res => res.json()).then(setData);
}, []);

// ❌ No cleanup: interval runs forever
useEffect(() => {
  setInterval(() => setTime(new Date()), 1000);
}, []);

// ❌ Long synchronous task blocks input
function handleClick() {
  const result = processMillionsOfItems(data); // Blocks main thread
  setResult(result);
}
```

**Expected Output:** The good practices produce a responsive, memory-stable component. The bad practices cause leaks, frozen UI, and stale state updates.

**Why this output:** Cleanup prevents leaks. `startTransition` keeps the input responsive while results update. The bad practices block the main thread and never free resources.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Long-running dashboards:** Memory hygiene is essential for stability over hours.
- **Data-intensive tools:** Offloading and task splitting keep the UI responsive.

### References

- React Optimise Best Practices (43 rules across 8 categories) — https://raw.githubusercontent.com/pproenca/dot-skills
- Steve Kinney — Memory Management Deep Dive — https://stevekinney.com/courses/react-performance/memory-management-deep-dive
- CoreUI — How to fix memory leaks in React — https://coreui.io
- React Academy — Reducing INP — https://www.coddykit.com
- Neptune Infotech — INP Optimization for React — https://neptuneinfotech.in
- React 19 concurrent mode — https://github.com/domidex01/tour-kit

---

## References

- Steve Kinney — Memory Management Deep Dive — https://stevekinney.com/courses/react-performance/memory-management-deep-dive
- Steve Kinney — Web Workers with React — https://stevekinney.com/courses/react-performance/web-workers-with-react
- CoreUI — How to fix memory leaks in React — https://coreui.io
- React Memory Leaks Guide (curiositech/windags-skills) — https://github.com/curiositech/windags-skills
- React/TypeScript Anti-Pattern: useEffect Cleanup — https://github.com/glennguilloux/llm-knowledge-base
- React Academy — Reducing INP: Event Handler Optimisation — https://www.coddykit.com
- Neptune Infotech — INP Optimization for React — https://neptuneinfotech.in
- React 19 concurrent mode and product tours — https://github.com/domidex01/tour-kit
- Mark expensive state updates as low-priority — https://raw.githubusercontent.com/pproenca/dot-skills
- React Optimise Best Practices (43 rules) — https://raw.githubusercontent.com/pproenca/dot-skills
- React runbooks — Memory leaks — https://github.com/aviator-co/runbooks-library
- Scaling React Apps with Parallelism — https://gitnation.com
- @okaygift/react-use-worker — https://www.npmjs.com/package/@okaygift/react-use-worker
- LabEx — Common cleanup scenarios — https://labex.io