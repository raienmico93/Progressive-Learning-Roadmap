# React Fiber — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Fiber is a complete reimplementation of React's core reconciliation algorithm, introduced in React 16, that replaces the synchronous, recursive stack-based reconciler with an incremental, interruptible, priority-driven engine.

**Technical Definition:** Fiber is an internal data structure and scheduling architecture that represents each unit of work as a heap-allocated, linked Fiber node rather than a transient call-stack frame. This re-architecture enables React to pause, resume, reprioritise, and abandon rendering work dynamically based on runtime priorities. Fiber maintains two trees — the **current tree** (committed UI) and the **work-in-progress tree** (next UI being computed) — and processes them through a cooperative work loop that yields to the browser between units of work, preventing long tasks from blocking the main thread. The render phase is fully interruptible; the commit phase remains synchronous and atomic. Fiber's lane-based scheduler assigns every update a priority (Immediate, UserBlocking, Normal, Low, Idle), and the highest-priority lane is processed first. This architecture is the foundation of concurrent rendering features such as `startTransition`, `useDeferredValue`, and `Suspense`.

**Beginner-Friendly Explanation:** React Fiber is the engine inside React that makes it fast and responsive. Before Fiber, React would start updating the screen and could not stop until it was finished — if that took too long, the page would freeze. Fiber changed that: React now breaks its work into tiny pieces, and after each piece it checks whether the browser needs to do something more important (like respond to a click or a keypress). If so, React pauses, handles the urgent work, and comes back to where it left off. Think of it as a chef who can stop chopping vegetables to take a pot off the stove, then resume chopping. This pause-and-resume ability is what makes modern React feel smooth even when it is doing heavy work in the background.

### Key Characteristics

- **Incremental rendering:** Work is split into small units (Fiber nodes) that can be processed one at a time.
- **Interruptible render phase:** React can pause, resume, or abandon rendering work when higher-priority updates arrive.
- **Atomic commit phase:** DOM mutations are applied in a single uninterrupted step to ensure consistency.
- **Priority-driven scheduling:** Lane-based priorities determine which work is processed first.
- **Time slicing:** React yields to the browser between Fiber units, keeping the main thread available for input and paint.
- **Double buffering:** Two Fiber trees (current and work-in-progress) allow React to discard unfinished work without affecting the displayed UI.
- **Concurrent foundation:** Fiber is the architecture that enables `startTransition`, `useDeferredValue`, and `Suspense`.

### Prerequisites

- React function components, state, props, and JSX.
- Understanding of the render → reconcile → commit pipeline.
- Familiarity with the Virtual DOM and element trees.
- Basic knowledge of the browser's main thread and event loop.

### Related Programming Areas

- **Reconciliation** — the algorithm Fiber executes incrementally.
- **Concurrent Features** — `useTransition`, `useDeferredValue`, and `Suspense` built on Fiber.
- **Rendering Performance** — long tasks, yielding, and priority.
- **Scheduling** — lanes, time slicing, and priority levels.

---

## 1. React Fiber

### Definitions

**Core Definition:** React Fiber is the incremental, interruptible reconciliation engine that replaced React's stack reconciler in React 16.

**Technical Definition:** Fiber is a complete rewrite of React's core algorithm. It introduces time-slicing, priority scheduling, and interruptible rendering. Instead of a single recursive render pass, Fiber represents each unit of work as a heap-allocated, linked Fiber node, enabling React to pause, resume, reprioritise, or abandon work dynamically. React 19 builds on this model with improved priority handling and more consistent transitions.

**Beginner-Friendly Explanation:** Fiber is React's multitasking engine. It lets React stop working on a slow update to handle something urgent, then resume the slow update later.

### Purposes

- To enable incremental, interruptible rendering that keeps the UI responsive.
- To prioritise urgent updates (input, clicks) over non-urgent work.
- To prevent long tasks from blocking the main thread.
- To provide the architectural foundation for concurrent rendering features.
- To allow React to discard stale work and restart with the latest state.

### Syntax Rules and Structure

**Fiber Execution Model:**

| Concept | Description |
|---------|-------------|
| Unit of work | One Fiber node |
| Work loop | Processes Fiber nodes one at a time |
| Yield point | After each unit of work, React checks whether to yield |
| Two trees | Current tree (committed) and work-in-progress tree (being built) |
| Phases | Render (interruptible) → Commit (atomic) |

**Syntax Rules:**

1. Fiber is an internal implementation detail; developers do not interact with Fiber nodes directly.
2. Rendering work is split into Fiber units and processed incrementally.
3. The render phase can be paused, resumed, or abandoned.
4. The commit phase is synchronous and atomic.
5. Components must be pure during render because work may be repeated or discarded.

**Constraints and Limitations:**

- Fiber does not make individual work cheaper; it reorders and interrupts work.
- Impure renders (side effects, mutations) can execute multiple times under Fiber's interruptible model.
- Large monolithic components reduce scheduling flexibility because there are fewer yield points.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Pure Component for Fiber Safety

```jsx
// ✅ Pure render: safe under Fiber's interruptible model
function UserCard({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// ❌ Impure render: side effects may double-fire
let counter = 0;
function BadCard({ user }) {
  counter++; // Side effect during render
  analytics.track('view', user.id); // Called multiple times
  return <div>{user.name}</div>;
}
```

**Expected Output:** The pure component renders consistently regardless of how many times Fiber executes it. The impure component may fire analytics multiple times.

**Why this output:** Fiber may pause, abandon, and restart render work. Side effects during render are executed each time, making them unreliable.

### Real-World Cases

- **Search autocomplete:** Typing remains responsive while results render in the background.
- **Dashboard widgets:** Heavy charts can be interrupted by user interactions.
- **Tab navigation:** Switching tabs does not block clicks or input.

### References

- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Virtual DOM & Fiber Architecture — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture
- A Closer Look at React Fiber — https://www.alibabacloud.com/blog/598138

---

## 2. Fiber Nodes

### Definitions

**Core Definition:** A Fiber node is a heap-allocated JavaScript object that represents one unit of work in React's reconciliation process.

**Technical Definition:** Each Fiber node corresponds to a React element (a component or DOM element) and carries everything needed to evaluate that unit: `type` (component or tag), `key`, `pendingProps` (props to be applied), `memoizedProps` (props from last render), `memoizedState` (state from last render), `pendingWorkPriority` (lane priority), and effect markers. Crucially, Fiber nodes are linked via `child`, `sibling`, and `return` pointers, forming a linked-list tree that React can traverse incrementally without relying on the call stack.

**Beginner-Friendly Explanation:** A Fiber node is a little package of information about one part of your UI. React creates one for every component and every DOM element. Each package knows what it is, what its children are, and what work needs to be done on it.

### Purposes

- To represent one unit of rendering work that can be processed independently.
- To carry the component's type, props, state, and priority.
- To link to child, sibling, and parent Fiber nodes for incremental traversal.
- To enable React to pause and resume work at Fiber boundaries.

### Syntax Rules and Structure

**Key Fiber Node Properties:**

| Property | Description |
|----------|-------------|
| `type` | Component type or DOM tag |
| `key` | Reconciliation key |
| `pendingProps` | Props to be applied |
| `memoizedProps` | Props from the last render |
| `memoizedState` | State from the last render |
| `child` | First child Fiber |
| `sibling` | Next sibling Fiber |
| `return` | Parent Fiber |
| `pendingWorkPriority` | Lane priority |

**Syntax Rules:**

1. Fiber nodes are created for every React element during the render phase.
2. The `child`, `sibling`, and `return` pointers form a traversable tree.
3. `pendingProps` and `memoizedProps` enable React to detect prop changes.
4. `memoizedState` preserves state across renders.
5. `pendingWorkPriority` determines scheduling order.

### Annotated Code Examples

```jsx
// Conceptual Fiber node structure (not developer-authored)
const fiber = {
  type: 'div',
  key: null,
  pendingProps: { className: 'card' },
  memoizedProps: { className: 'card' },
  memoizedState: null,
  child: childFiber,
  sibling: siblingFiber,
  return: parentFiber,
  pendingWorkPriority: NormalPriority,
};
```

**Expected Output:** React uses this structure internally to evaluate one unit of work at a time.

**Why this output:** Each Fiber node is self-contained, allowing React to process it, yield, and later resume from the same point.

### Real-World Cases

- **Component trees:** Every component and DOM element in a React app has a corresponding Fiber node.
- **Priority scheduling:** `pendingWorkPriority` determines which Fiber is processed first.
- **State preservation:** `memoizedState` keeps state across renders.

### References

- A Closer Look at React Fiber — https://www.alibabacloud.com/blog/598138
- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture

---

## 3. Fiber Tree

### Definitions

**Core Definition:** The Fiber tree is the linked-list tree of Fiber nodes that mirrors the component tree and is used for incremental reconciliation.

**Technical Definition:** React maintains two Fiber trees: the **current tree**, which represents the committed UI displayed to the user, and the **work-in-progress tree**, which is the next UI state being computed. React mutates only the work-in-progress tree. When rendering completes, React swaps the root pointers of the two trees, making the work-in-progress tree the new current tree. If a higher-priority update arrives during rendering, React can discard the unfinished work-in-progress tree without affecting the displayed UI.

**Beginner-Friendly Explanation:** React keeps two copies of its plan: one for what you see now, and one for what it is building next. It only changes the second copy. When it is done, it swaps them. If something urgent comes up, it can throw away the second copy and start over without breaking what you see.

### Purposes

- To enable double buffering so React can discard unfinished work safely.
- To provide a traversable structure for incremental rendering.
- To separate committed UI from in-progress work.
- To support atomic commits via pointer swapping.

### Syntax Rules and Structure

**Two-Tree Model:**

| Tree | Description |
|------|-------------|
| Current tree | Committed UI displayed to the user |
| Work-in-progress tree | Next UI state being computed |

**Syntax Rules:**

1. React mutates only the work-in-progress tree during render.
2. The current tree remains unchanged until commit.
3. At commit, React swaps the root pointers of the two trees.
4. If rendering is interrupted, the work-in-progress tree can be discarded without affecting the UI.

### Annotated Code Examples

```jsx
// Conceptual: React swaps current and work-in-progress roots at commit
let currentRoot = committedFiberRoot;
let workInProgressRoot = buildNewTree(currentRoot);

// On commit:
const previousRoot = currentRoot;
currentRoot = workInProgressRoot;
workInProgressRoot = null;
```

**Expected Output:** The UI updates atomically when the work-in-progress tree becomes the current tree.

**Why this output:** Double buffering ensures that unfinished or stale work never affects the displayed UI.

### Real-World Cases

- **Interruptible rendering:** Work-in-progress tree is discarded when higher-priority work arrives.
- **Atomic updates:** The commit phase swaps trees in one step.
- **Concurrent rendering:** Two trees enable React to prepare multiple UI versions simultaneously.

### References

- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/

---

## 4. Work Loop

### Definitions

**Core Definition:** The work loop is the cooperative scheduling mechanism that processes Fiber nodes one at a time, yielding to the browser between units of work.

**Technical Definition:** Starting in React 16, Fiber replaced a single recursive render pass with a cooperative, interruptible work loop. The work loop processes the work-in-progress tree one Fiber at a time. After each unit of work, React evaluates whether it still has time in the current frame or should yield to the browser. If urgent work arrives, React can pause, abandon, or restart the render. This mechanism underpins time slicing and concurrent rendering.

**Beginner-Friendly Explanation:** The work loop is React's "one step at a time" process. It does a small piece of work, checks if the browser needs a turn, does another piece, and so on.

### Purposes

- To process rendering work incrementally.
- To yield to the browser between Fiber units, keeping the main thread responsive.
- To allow interruption and resumption of rendering work.
- To enable time slicing and priority-based scheduling.

### Syntax Rules and Structure

**Work Loop Phases:**

| Phase | Description |
|-------|-------------|
| `beginWork` | Process a Fiber node |
| `completeUnitOfWork` | Finalise the Fiber's work |
| Yield check | After each unit, React decides whether to yield |
| Resume | React continues from the next Fiber when the browser is free |

**Syntax Rules:**

1. The work loop processes one Fiber node at a time.
2. After each unit, React checks whether to yield to the browser.
3. If interrupted, React resumes from the same point later.
4. The work loop is part of the render phase and is fully interruptible.

### Annotated Code Examples

```jsx
// Conceptual work loop (simplified)
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1; // Yield if no time left
  }
  if (nextUnitOfWork) {
    requestIdleCallback(workLoop); // Resume later
  }
}
```

**Expected Output:** Work is processed in chunks, yielding to the browser between chunks.

**Why this output:** The work loop processes as many Fiber units as time allows, then yields and schedules a continuation.

### Real-World Cases

- **Large component trees:** The work loop prevents long blocking renders.
- **Concurrent features:** `useTransition` and `useDeferredValue` rely on the work loop's interruptibility.
- **Smooth animations:** Yielding keeps the main thread available for animation frames.

### References

- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- A Closer Look at React Fiber — https://www.alibabacloud.com/blog/598138

---

## 5. Rendering Phases

### Definitions

**Core Definition:** Fiber divides rendering into two distinct phases: the render phase (interruptible, pure computation) and the commit phase (synchronous, atomic DOM mutation).

**Technical Definition:** The render phase traverses the Fiber tree, computes the next UI state, and builds the work-in-progress tree. It is pure and interruptible. The commit phase applies the computed changes to the DOM in a single, uninterrupted step. This two-phase pipeline controls update scheduling, effect timing, and DOM consistency in React 19's concurrent runtime. Fiber makes the first stage fully interruptible and the second strictly atomic.

**Beginner-Friendly Explanation:** React does its thinking in two steps. First it figures out what needs to change (this can be paused). Then it makes the changes all at once (this cannot be paused).

### Purposes

- To separate pure computation (render) from side effects (commit).
- To allow the render phase to be interrupted without affecting the UI.
- To ensure DOM mutations are applied consistently and atomically.
- To control when effects run relative to paint.

### Syntax Rules and Structure

**Phase Comparison:**

| Phase | Interruptible? | Side effects? |
|-------|----------------|---------------|
| Render | Yes | No (must be pure) |
| Commit | No (atomic) | Yes (DOM mutations, effects) |

**Syntax Rules:**

1. The render phase is interruptible and must be pure.
2. The commit phase is synchronous and atomic.
3. All side effects belong in the commit phase or later.
4. Interacting with the DOM during render violates Fiber's guarantees.

### Annotated Code Examples

```jsx
function App() {
  // Render phase: pure computation
  const [count, setCount] = useState(0);

  // ✅ Side effect in event handler (after commit)
  function handleClick() {
    setCount(c => c + 1);
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

**Expected Output:** The render phase computes the new `count` value; the commit phase updates the DOM.

**Why this output:** The render phase is pure; side effects like state updates are triggered by events after commit.

### Real-World Cases

- **Concurrent rendering:** The render phase can be interrupted; the commit phase cannot.
- **Effects:** `useEffect` and `useLayoutEffect` run after the commit phase.
- **Animations:** DOM mutations are applied atomically during commit.

### References

- Render and Commit Phases — https://www.educative.io/courses/learn-react/render-and-commit-phases
- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture

---

## 6. Render Phase

### Definitions

**Core Definition:** The render phase is the interruptible part of Fiber's execution where React computes the next UI state and builds the work-in-progress Fiber tree.

**Technical Definition:** During the render phase, React traverses the Fiber tree, evaluates each Fiber node, and determines which changes may need to be committed. It builds or updates the work-in-progress tree and produces an effect list of nodes to update, add, or delete. The render phase is pure, can be paused, resumed, or abandoned, and must not produce side effects.

**Beginner-Friendly Explanation:** The render phase is React's planning stage. It figures out what the next screen should look like, without actually changing anything yet.

### Purposes

- To compute the next UI state.
- To build the work-in-progress Fiber tree.
- To identify changes (effects) to apply during commit.
- To allow interruption and resumption without affecting the UI.

### Syntax Rules and Structure

**Render Phase Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| Purity | No side effects, no DOM mutations |
| Interruptibility | Can be paused, resumed, or abandoned |
| Output | Work-in-progress tree and effect list |

**Syntax Rules:**

1. Components must be pure during render.
2. No side effects (analytics, subscriptions, DOM reads/writes) in render.
3. The render phase can be restarted if interrupted.

### Annotated Code Examples

```jsx
// ✅ Pure render phase
function PureComponent({ data }) {
  const sorted = [...data].sort(); // Computation only
  return <ul>{sorted.map(item => <li key={item}>{item}</li>)}</ul>;
}

// ❌ Impure render phase: side effect
function ImpureComponent({ data }) {
  analytics.track('render'); // Called multiple times under Fiber
  return <div>{data.length}</div>;
}
```

**Expected Output:** The pure component produces the same output every time. The impure component fires analytics multiple times.

**Why this output:** The render phase may execute multiple times for the same component under Fiber's interruptible model.

### Real-World Cases

- **Sorting and filtering:** Computation belongs in the render phase (memoized if expensive).
- **Logging and analytics:** Belong in effects, not render.
- **DOM measurement:** Belongs in `useLayoutEffect` or effects, not render.

### References

- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture

---

## 7. Commit Phase

### Definitions

**Core Definition:** The commit phase is the synchronous, atomic part of Fiber's execution where React applies the computed changes to the DOM.

**Technical Definition:** In the commit phase, React swaps the root pointers of the current and work-in-progress trees and applies all DOM mutations in a single, uninterrupted step. This ensures consistency and accuracy. Side effects, layout effects, and passive effects are scheduled according to their timing. The commit phase cannot be paused or interrupted.

**Beginner-Friendly Explanation:** The commit phase is when React actually changes the screen. It does all the changes at once, without stopping.

### Purposes

- To apply computed changes to the DOM atomically.
- To ensure the displayed UI is consistent.
- To swap the current and work-in-progress trees.
- To schedule effects (layout and passive) after DOM mutations.

### Syntax Rules and Structure

**Commit Phase Operations:**

| Operation | Timing |
|-----------|--------|
| DOM mutations | Synchronous, atomic |
| Layout effects (`useLayoutEffect`) | After DOM mutations, before paint |
| Passive effects (`useEffect`) | After paint |

**Syntax Rules:**

1. The commit phase is synchronous and cannot be interrupted.
2. DOM mutations are applied in a single step.
3. All side effects belong after commit.
4. Interacting with the DOM during render violates Fiber's guarantees.

### Annotated Code Examples

```jsx
function Component() {
  const [visible, setVisible] = useState(false);

  // ✅ Side effect after commit
  useEffect(() => {
    if (visible) {
      document.title = 'Visible';
    }
  }, [visible]);

  return <div>{visible ? 'Shown' : 'Hidden'}</div>;
}
```

**Expected Output:** The DOM is updated atomically during commit, then the effect runs after paint.

**Why this output:** The commit phase applies the DOM change; the effect runs after the commit phase completes.

### Real-World Cases

- **Animations:** DOM changes are applied atomically before paint.
- **Focus management:** `useLayoutEffect` runs after DOM mutations, before paint.
- **Third-party libraries:** DOM manipulation libraries run after commit.

### References

- Render and Commit Phases — https://www.educative.io/courses/learn-react/render-and-commit-phases
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/

---

## 8. Fiber Reconciliation

### Definitions

**Core Definition:** Fiber reconciliation is the incremental process of comparing the current and work-in-progress Fiber trees to determine the minimal set of changes needed.

**Technical Definition:** Fiber reconciliation traverses the Fiber tree using the `child`, `sibling`, and `return` pointers, evaluating each Fiber node and comparing the new element tree with the previous one. React uses heuristics (type identity and keys) to match nodes efficiently. The output is an effect list of nodes to update, add, or delete. Unlike the stack reconciler, Fiber reconciliation can be paused and resumed between Fiber nodes.

**Beginner-Friendly Explanation:** Fiber reconciliation is React's comparison step, but done in small pieces. It compares the old and new UI descriptions and notes what needs to change, one Fiber at a time.

### Purposes

- To determine the minimal set of DOM changes.
- To produce the effect list for the commit phase.
- To preserve state and DOM nodes where identity is unchanged.
- To enable incremental, interruptible comparison.

### Syntax Rules and Structure

**Reconciliation Heuristics:**

| Situation | Behaviour |
|-----------|-----------|
| Different types | Full teardown and rebuild |
| Same type | Update attributes and recurse |
| Keys | Match children by key |
| No keys | Match by position |

**Syntax Rules:**

1. Fiber reconciliation traverses the tree incrementally.
2. The output is an effect list of changes.
3. Type and key heuristics determine matching.
4. Reconciliation can be interrupted between Fiber nodes.

### Annotated Code Examples

```jsx
// Keyed list: Fiber reconciliation matches by key
{todos.map(todo => (
  <TodoItem key={todo.id} todo={todo} />
))}
```

**Expected Output:** Fiber reconciliation matches existing `TodoItem` fibers by key and moves DOM nodes instead of recreating them.

**Why this output:** Keys provide stable identity for Fiber's incremental matching.

### Real-World Cases

- **List updates:** Keyed reconciliation preserves state and minimises DOM operations.
- **Conditional rendering:** Type changes trigger full rebuilds.
- **Large trees:** Incremental reconciliation keeps the main thread responsive.

### References

- A Closer Look at React Fiber — https://www.alibabacloud.com/blog/598138
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/

---

## 9. Fiber Scheduling

### Definitions

**Core Definition:** Fiber scheduling is the priority-driven mechanism that determines which rendering work React processes first.

**Technical Definition:** Fiber's scheduler assigns every update a priority and processes the highest-priority lane first. The scheduler priority levels are Immediate (sync, must happen now), UserBlocking (user interactions), Normal, Low, and Idle. React 18 introduced the **lane model** — a 31-bit priority bitmask — which replaced the earlier scheduler priority abstraction. Lanes allow an update to occupy multiple lanes and enable fine-grained prioritisation of concurrent work.

**Beginner-Friendly Explanation:** Fiber scheduling is React's priority system. It decides what to work on first: urgent things like typing and clicking get priority, while background updates wait their turn.

### Purposes

- To process urgent updates before non-urgent ones.
- To enable concurrent rendering features.
- To keep the UI responsive under heavy load.
- To provide predictable scheduling behaviour.

### Syntax Rules and Structure

**Scheduler Priority Levels:**

| Level | Meaning |
|-------|---------|
| Immediate | Must happen now (synchronous) |
| UserBlocking | User interactions (click, input) |
| Normal | Default priority |
| Low | Non-critical work |
| Idle | Background work |

**Syntax Rules:**

1. Every update enters the scheduler with a priority.
2. The highest-priority lane is processed first.
3. Lanes are a 31-bit bitmask (React 18+).
4. High-priority tasks can interrupt low-priority tasks.

### Annotated Code Examples

```jsx
// Conceptual: priority scheduling (internal to React)
const ImmediatePriority = 1;   // Sync
const UserBlockingPriority = 2; // Input, click
const NormalPriority = 3;       // Default
const LowPriority = 4;
const IdlePriority = 5;
```

**Expected Output:** React processes higher-priority work first.

**Why this output:** The scheduler walks the highest-priority bit set first, enabling urgent work to preempt background work.

### Real-World Cases

- **Typing:** Input updates are UserBlocking priority and processed immediately.
- **Transitions:** `startTransition` marks updates as lower priority.
- **Data fetching:** Normal or Low priority, deferred until urgent work is done.

### References

- React Rendering Architecture: Fiber, Lanes, Streaming SSR — https://raw.githubusercontent.com
- Scheduling and Lanes — https://github.com/johnxie/awesome-code-docs
- React Fiber 架构详解 — https://cloud.tencent.com.cn

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

- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- Concurrent Rendering and Automatic Batching — https://www.educative.io

---

## 11. Priority Levels

### Definitions

**Core Definition:** Priority levels are the discrete scheduling tiers that determine the order in which React processes updates.

**Technical Definition:** Fiber's scheduler uses priority levels — Immediate, UserBlocking, Normal, Low, and Idle — to determine processing order. React 18 replaced these with the lane model, a 31-bit bitmask where each update gets a lane bit and the scheduler walks the highest-priority bit set first. Lanes allow an update to occupy multiple lanes and enable fine-grained concurrent prioritisation.

**Beginner-Friendly Explanation:** Priority levels are labels that tell React what is urgent and what can wait. Typing is urgent; updating a chart is not.

### Purposes

- To ensure urgent updates are processed first.
- To enable concurrent features like transitions.
- To prevent background work from blocking user input.
- To provide predictable scheduling behaviour.

### Syntax Rules and Structure

**Priority Level Mapping:**

| Priority | Use case |
|----------|----------|
| Immediate | Synchronous, must happen now |
| UserBlocking | Clicks, input, focus |
| Normal | Data fetching, default updates |
| Low | Analytics, logging |
| Idle | Background cleanup |

**Syntax Rules:**

1. Every update receives a priority level.
2. The highest-priority lane is processed first.
3. High-priority work can interrupt low-priority work.
4. Lanes replaced scheduler priorities in React 18.

### Annotated Code Examples

```jsx
// Conceptual: priority assignment
const update = {
  lane: UserBlockingLane, // Input updates get high priority
  action: () => setQuery(value),
};
```

**Expected Output:** The update is scheduled in the UserBlocking lane and processed before lower-priority work.

**Why this output:** The lane model assigns priority bits and processes the highest bit first.

### Real-World Cases

- **Input fields:** UserBlocking priority for keystrokes.
- **Transitions:** Lower priority for non-urgent state updates.
- **Background sync:** Idle priority for analytics and logging.

### References

- Scheduling and Lanes — https://github.com/johnxie/awesome-code-docs
- React Fiber 架构详解 — https://cloud.tencent.com.cn

---

## 12. Interruptible Rendering

### Definitions

**Core Definition:** Interruptible rendering is Fiber's ability to pause, resume, or abandon rendering work when higher-priority updates arrive.

**Technical Definition:** Because Fiber represents work as heap-allocated Fiber nodes rather than call-stack frames, React can pause rendering at Fiber boundaries, switch to a higher-priority subtree, and later resume or discard the interrupted work. This is the core capability that distinguishes Fiber from the stack reconciler. Concurrent features such as `startTransition`, `useDeferredValue`, and `Suspense` use interruptible rendering to keep the UI responsive.

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

- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/

---

## 13. Concurrent Rendering

### Definitions

**Core Definition:** Concurrent rendering is React's ability to prepare multiple versions of the UI simultaneously and prioritise updates, built on Fiber's interruptible architecture.

**Technical Definition:** Concurrent rendering is not parallel processing (JavaScript remains single-threaded). Instead, it splits rendering work into small units (Fiber nodes) and uses the scheduler to execute them during browser idle time, while allowing high-priority tasks to preempt lower-priority ones. Fiber's architecture is the foundation: the work loop, time slicing, and lane-based priorities together enable concurrent behaviour. Concurrent features such as `startTransition`, `useDeferredValue`, and `Suspense` expose this capability to developers.

**Beginner-Friendly Explanation:** Concurrent rendering is React's ability to work on several things without finishing one before starting another. It does not do them at the same time — it just switches between them intelligently.

### Purposes

- To keep the UI responsive during heavy rendering.
- To prioritise urgent updates over non-urgent ones.
- To enable smooth transitions and deferred updates.
- To improve perceived performance and Interaction to Next Paint (INP).

### Syntax Rules and Structure

**Concurrent Rendering Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| Single-threaded | Not parallel execution |
| Incremental | Work split into Fiber units |
| Interruptible | Urgent work preempts background work |
| Priority-driven | Lane-based scheduling |

**Syntax Rules:**

1. Concurrent rendering is enabled by Fiber's work loop and scheduler.
2. `startTransition` and `useDeferredValue` mark updates as interruptible.
3. High-priority tasks preempt low-priority tasks.
4. The commit phase remains atomic.

### Annotated Code Examples

```jsx
function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setQuery(e.target.value); // Urgent
    startTransition(() => {
      setResults(expensiveSearch(e.target.value)); // Interruptible
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  );
}
```

**Expected Output:** Typing remains responsive while results render in an interruptible transition.

**Why this output:** `startTransition` marks the search update as lower priority; Fiber can pause it to process the urgent input update.

### Real-World Cases

- **Search autocomplete:** Input stays responsive during expensive filtering.
- **Tab navigation:** Switching tabs does not block clicks.
- **Data dashboards:** Charts render in the background while controls remain interactive.

### References

- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Concurrent Rendering and Automatic Batching — https://www.educative.io
- React 18 并发渲染内幕 — https://cloud.tencent.com.cn

---

## 14. Fiber vs Stack Reconciler

### Definitions

**Core Definition:** The stack reconciler was React's original synchronous, recursive reconciliation engine; Fiber replaced it with an incremental, interruptible architecture.

**Technical Definition:** The stack reconciler (React 15 and earlier) processed the component tree synchronously in a single recursive pass. It used the JavaScript call stack to traverse the tree, which meant rendering could not be paused or interrupted. Deeply nested components or expensive subtrees would block the main thread until rendering completed. Fiber (React 16+) reimagines reconciliation as an incremental, interruptible process using heap-allocated Fiber nodes and a cooperative work loop.

**Beginner-Friendly Explanation:** The old stack reconciler was like reading a book without stopping until you finish the chapter. Fiber is like reading a few pages, taking a break, and coming back later.

### Purposes

- To understand why Fiber was introduced.
- To recognise the limitations of the stack reconciler.
- To appreciate the performance benefits of Fiber's interruptible architecture.

### Comparison Table

| Aspect | Stack Reconciler | Fiber Reconciler |
|--------|------------------|------------------|
| Traversal | Recursive call stack | Linked-list Fiber nodes |
| Interruptible | No | Yes |
| Time slicing | No | Yes |
| Priority scheduling | No | Yes |
| Large tree handling | Blocks main thread | Yields to browser |
| Concurrent features | Not supported | Supported |

**Syntax Rules:**

1. Stack reconciler was synchronous and could not be interrupted.
2. Fiber is incremental and interruptible.
3. Fiber enables priority scheduling and time slicing.
4. Stack reconciler could cause stack overflows on deep trees.

### Annotated Code Examples

```jsx
// Stack reconciler: synchronous recursion (React 15)
function reconcileStack(element) {
  // Cannot yield; blocks until complete
  for (const child of element.children) {
    reconcileStack(child);
  }
}

// Fiber: incremental work loop (React 16+)
function workLoop() {
  while (workInProgress && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
  if (workInProgress) scheduleContinuation(workLoop);
}
```

**Expected Output:** The stack reconciler blocks until the entire tree is processed; Fiber yields between units.

**Why this output:** The stack reconciler relies on the call stack, which cannot be paused. Fiber uses a work loop that can yield to the browser.

### Real-World Cases

- **Large applications:** Fiber prevents main-thread blocking that the stack reconciler caused.
- **Concurrent features:** Only possible with Fiber.
- **Deep component trees:** Fiber handles them incrementally; the stack reconciler could overflow.

### References

- 深入理解React Fiber架构及其性能优化 — https://developer.aliyun.com/article/1623479
- React Fiber: A Closer Look at the New Engine of React — https://www.infoq.com
- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture

---

## 15. Fiber Best Practices

### Definitions

**Core Definition:** Fiber best practices are the guidelines for writing React components that cooperate with Fiber's interruptible, priority-driven scheduling model.

**Technical Definition:** Best practices centre on render purity (no side effects during render), stable component structure (avoiding unnecessarily deep trees that reduce yield flexibility), stable props and keys (to minimise wasted reconciliation work), appropriate use of concurrent features (`startTransition`, `useDeferredValue`), and keeping expensive work out of the render phase. Because Fiber may repeat or discard render work, any observable side effect during render is a correctness bug, not just a performance issue.

**Beginner-Friendly Explanation:** Write components that are safe to run multiple times and easy for React to pause. Do not cause side effects while rendering. Keep your component tree from getting too deep. Use transitions for non-urgent updates.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Keep render pure** | No side effects, mutations, or external reads during render. |
| **Avoid overly deep trees** | Large monolithic components reduce yield flexibility. |
| **Stabilise props and keys** | Reduce wasted reconciliation work. |
| **Use concurrent features** | `startTransition` and `useDeferredValue` for non-urgent work. |
| **Put side effects in effects** | DOM work, analytics, and subscriptions belong after commit. |
| **Use stable keys** | Prevent unnecessary Fiber subtree revisits. |
| **Measure with Profiler** | Verify that scheduling works as intended. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(value) {
    setQuery(value); // Urgent
    startTransition(() => {
      setResults(expensiveSearch(value)); // Interruptible
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
// ❌ Side effect during render: may double-fire under Fiber
function BadComponent() {
  analytics.track('render'); // Called multiple times
  return <div />;
}

// ❌ Monolithic component: few yield points
function HugeComponent() {
  // Thousands of lines of JSX and logic
}
```

**Expected Output:** The good practices keep the UI responsive and avoid double-fired side effects. The bad practices cause unreliable behaviour and reduce scheduling flexibility.

**Why this output:** Fiber may pause and restart renders; side effects in render are executed multiple times. Large components have fewer Fiber boundaries, reducing yield opportunities.

### Real-World Cases

- **All React 19 applications:** Fiber is the default scheduler, so these practices apply universally.
- **Analytics-heavy apps:** Double-fired analytics is a common Fiber-related bug.
- **Performance-sensitive UIs:** Stable keys and concurrent features keep interactions smooth.

### References

- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Virtual DOM & Fiber Architecture — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture

---

## References

- A Closer Look at React Fiber — https://www.alibabacloud.com/blog/598138
- Understanding the Fiber Architecture — https://www.educative.io/courses/learn-react/understanding-the-fiber-architecture
- A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Virtual DOM & Fiber Architecture — https://stevekinney.com/courses/react-performance/virtual-dom-fiber-architecture
- 深入理解React Fiber架构及其性能优化 — https://developer.aliyun.com/article/1623479
- Render and Commit Phases — https://www.educative.io/courses/learn-react/render-and-commit-phases
- Scheduling and Lanes — https://github.com/johnxie/awesome-code-docs
- React Rendering Architecture: Fiber, Lanes, Streaming SSR — https://raw.githubusercontent.com
- React Fiber: A Closer Look at the New Engine of React — https://www.infoq.com
- React Fiber 架构详解 — https://cloud.tencent.com.cn
- Concurrent Rendering and Automatic Batching — https://www.educative.io