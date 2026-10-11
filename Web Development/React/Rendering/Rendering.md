# React Rendering Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Rendering is the process by which React calls your components to determine what should appear on the screen, and then updates the DOM to match.

**Technical Definition:** React rendering is a two-phase process. In the **render phase**, React calls your component functions to produce a new tree of React elements (a virtual representation of the UI), then compares this new tree with the previous one through a process called reconciliation. In the **commit phase**, React applies the minimal set of DOM mutations required to make the real DOM match the new tree. This two-phase pipeline ensures that updates are applied efficiently and predictably.

**Beginner-Friendly Explanation:** Rendering is React figuring out what your UI should look like and then updating the screen to match. When something changes — like a user clicking a button or data arriving from a server — React recalculates what should be shown and updates only the parts of the page that actually changed. Think of React as a skilled chef: you give it an order (your component code), it prepares the dish (renders the components), and then it serves it to the table (commits the changes to the DOM).

### Key Characteristics

- **Two-phase pipeline:** Rendering happens in two distinct phases — render (pure, interruptible) and commit (synchronous, blocking).
- **Pure rendering:** Components must be pure — same inputs, same output. No side effects during render.
- **Declarative:** You describe what the UI should look like, and React figures out how to update the DOM efficiently.
- **Reconciliation-driven:** React compares the new element tree with the previous one to compute the minimal set of changes.
- **Interruptible (React 18+):** Concurrent rendering allows React to pause, resume, and prioritise rendering work.

### Prerequisites

- JavaScript functions, objects, and closures.
- React function components and JSX.
- Understanding of props, state, and event handling.
- Familiarity with the DOM (Document Object Model).

### Related Programming Areas

- **Reconciliation** — the algorithm that determines what changed between renders.
- **React Fiber** — the internal engine that enables interruptible rendering.
- **Virtual DOM** — the in-memory representation of the UI.
- **Concurrent Rendering** — React 18+ features for prioritised, interruptible rendering.

---

## 1. Rendering

### Definitions

**Core Definition:** Rendering is the process by which React calls your components to determine what should appear on the screen.

**Technical Definition:** Rendering in React is a two-step process. First, React calls your component functions to produce React elements. Second, React compares the new elements with the previous ones and commits the necessary changes to the DOM. The render phase is pure and can be interrupted; the commit phase is synchronous and cannot be interrupted.

**Beginner-Friendly Explanation:** Rendering is React doing the work to figure out what your UI should look like and then updating the screen.

### Purposes

- To compute the UI based on current state and props.
- To determine what needs to change in the DOM.
- To ensure the UI stays consistent with application data.

### Syntax Rules and Structure

**The Three Steps of Rendering:**

| Step | Description |
|------|-------------|
| Trigger | A render is triggered by initial mount or a state update. |
| Render | React calls your components to compute the new element tree. |
| Commit | React applies the minimal DOM changes. |

**Syntax Rules:**

1. Rendering must be pure — no side effects during render.
2. React calls components recursively until the entire tree is rendered.
3. Rendering does not always produce DOM updates — React only commits when something changed.

### Annotated Code Examples

```jsx
function App() {
  const [count, setCount] = useState(0);

  console.log('Rendering App');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Expected Output:** "Rendering App" logs on initial render and on each state update.

**Why this output:** React calls `App` whenever a render is triggered. The `console.log` runs during the render phase, which is why it appears on every render.

### Real-World Cases

- **Initial page load:** React renders the entire component tree for the first time.
- **State updates:** Clicking a button triggers a re-render of the component and its children.
- **Data fetching:** When data arrives from an API, state updates trigger a re-render with the new data.

### References

- React — Render and Commit — https://18.react.dev/learn/render-and-commit

---

## 2. Initial Render

### Definitions

**Core Definition:** The initial render is the first time React renders a component tree, typically triggered by calling `root.render()`.

**Technical Definition:** The initial render occurs when the application starts. React calls the root component and recursively renders all child components to build the initial element tree. This tree is then used to create the DOM nodes and insert them into the page.

**Beginner-Friendly Explanation:** The initial render is the first time your app appears on screen.

### Purposes

- To build the initial UI from the component tree.
- To create the DOM nodes for the first time.
- To set up the application's starting state.

### Syntax Rules and Structure

```jsx
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**Syntax Rules:**

1. `root.render()` triggers the initial render.
2. React calls the root component and all descendants.
3. DOM nodes are created for each element in the tree.

### Annotated Code Examples

```jsx
function App() {
  return (
    <div>
      <h1>Hello, World!</h1>
      <p>This is the initial render.</p>
    </div>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**Expected Output:** The page displays "Hello, World!" and the paragraph.

**Why this output:** `root.render(<App />)` triggers the initial render. React calls `App`, which returns the JSX. React then creates the DOM nodes and inserts them into the page.

### Real-World Cases

- **Application startup:** The first render of the root component.
- **Server-side rendering:** The initial render happens on the server.
- **Testing:** The initial render is used to verify the component's output.

### References

- React — Render and Commit: Step 1 — https://18.react.dev/learn/render-and-commit

---

## 3. Re-rendering

### Definitions

**Core Definition:** Re-rendering is a subsequent render triggered by a state update or a parent component's re-render.

**Technical Definition:** Once a component has been initially rendered, you can trigger further renders by updating its state with the set function. Updating state automatically adds the component to the re-render queue. React then calls the component (and any children whose props changed) to compute the new element tree.

**Beginner-Friendly Explanation:** Re-rendering is React updating the UI after something changes, like a user clicking a button or data arriving.

### Purposes

- To reflect changes in state or props in the UI.
- To update the DOM with new data.
- To keep the UI in sync with application state.

### Syntax Rules and Structure

**Re-render Triggers:**

| Trigger | Description |
|---------|-------------|
| State change | `setState` or `useState` setter. |
| Prop change | Parent passes new props. |
| Parent render | Parent re-renders, causing child re-render. |
| Context change | Context value changes. |

**Syntax Rules:**

1. State updates queue a re-render.
2. React re-renders the component that owns the state and its descendants.
3. React only commits DOM changes if something actually changed.

### Annotated Code Examples

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  console.log('Counter rendered');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

**Expected Output:** "Counter rendered" logs on initial render and on each button click.

**Why this output:** Clicking the button calls `setCount`, which queues a re-render. React calls `Counter` again, logging the message.

### Real-World Cases

- **Interactive UI:** Buttons, forms, and toggles trigger re-renders.
- **Data updates:** API responses update state and trigger re-renders.
- **Animation:** State changes driven by animation loops trigger re-renders.

### References

- React — Render and Commit: Re-renders when state updates — https://18.react.dev/learn/render-and-commit#re-renders-when-state-updates

---

## 4. Render Phase

### Definitions

**Core Definition:** The render phase is the first phase of React's rendering process, where React calls your components and builds the new virtual tree.

**Technical Definition:** During the render phase, React calls your component functions to produce a new tree of React elements. It then compares this new tree with the previous tree (reconciliation). The render phase is pure — it produces no side effects and can be interrupted, paused, or restarted by React.

**Beginner-Friendly Explanation:** The render phase is where React figures out what the UI should look like. It calls your components and builds a description of the new UI.

### Purposes

- To compute the new element tree based on current state and props.
- To perform reconciliation — comparing the new tree with the previous one.
- To prepare the changes that will be applied in the commit phase.

### Syntax Rules and Structure

**Render Phase Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| Purity | No side effects — same inputs, same output. |
| Interruptibility | Can be paused, resumed, or restarted (React 18+). |
| Recursive | React calls components recursively until the tree is complete. |

**Syntax Rules:**

1. Components must be pure during render.
2. Do not mutate state or props during render.
3. Do not perform side effects (API calls, subscriptions) during render.

### Annotated Code Examples

```jsx
function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return <img src="https://i.imgur.com/ZF6s192.jpg" alt="Sculpture" />;
}
```

**Expected Output:** The DOM contains a `<section>`, `<h1>`, and three `<img>` elements.

**Why this output:** During the render phase, React calls `Gallery` and `Image` recursively. Each call produces React elements that describe the DOM structure. React then commits these to the DOM.

### Real-World Cases

- **Every render:** The render phase runs on every render, whether initial or re-render.
- **Concurrent rendering:** React can interrupt and restart the render phase to prioritise urgent updates.

### References

- React — Render and Commit: Step 2 — https://18.react.dev/learn/render-and-commit
- Steve Kinney — Understanding Reconciliation — https://github.com/stevekinney/stevekinney.net

---

## 5. Commit Phase

### Definitions

**Core Definition:** The commit phase is the second phase of React's rendering process, where React applies the calculated changes to the actual DOM.

**Technical Definition:** In the commit phase, React applies the minimal set of DOM mutations required to make the real DOM match the new element tree. This phase is synchronous and cannot be interrupted — React applies all changes before the browser repaints the screen. After the commit phase, the browser paints the updated screen.

**Beginner-Friendly Explanation:** The commit phase is where React actually updates the screen. It takes the changes calculated during the render phase and applies them to the real DOM.

### Purposes

- To apply the computed changes to the real DOM.
- To ensure the DOM matches the new element tree.
- To prepare the screen for the browser's paint cycle.

### Syntax Rules and Structure

**Commit Phase Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| Synchronous | Cannot be interrupted. |
| Minimal changes | React applies only the necessary DOM mutations. |
| Post-commit | After commit, the browser paints the updated screen. |

**Syntax Rules:**

1. The commit phase runs synchronously after the render phase.
2. React applies minimal DOM mutations.
3. After commit, the browser paints the screen.

### Annotated Code Examples

```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('Commit phase complete');
  });

  return <p>Count: {count}</p>;
}
```

**Expected Output:** "Commit phase complete" logs after each render.

**Why this output:** `useEffect` runs after the commit phase. The message appears after React has applied the DOM changes.

### Real-World Cases

- **DOM updates:** Every time React updates the DOM, the commit phase runs.
- **Focus management:** `useLayoutEffect` runs during the commit phase, before paint.
- **Third-party libraries:** DOM manipulation libraries run after commit.

### References

- React — Render and Commit: Step 3 — https://18.react.dev/learn/render-and-commit

---

## 6. Reconciliation

### Definitions

**Core Definition:** Reconciliation is React's process for figuring out what changed between renders and updating the DOM accordingly.

**Technical Definition:** Reconciliation is the algorithm React uses to compare the new element tree with the previous one and determine the minimal set of changes needed to update the DOM. React implements a heuristic O(n) algorithm based on two assumptions: (1) two elements of different types will produce different trees, and (2) the developer can hint at stable child elements with a `key` prop.

**Beginner-Friendly Explanation:** Reconciliation is React comparing the old UI description with the new one and figuring out what actually changed.

### Purposes

- To determine the minimal set of DOM changes.
- To preserve DOM nodes and component state where possible.
- To make updates efficient and predictable.

### Syntax Rules and Structure

**Reconciliation Rules:**

| Situation | Behaviour |
|-----------|-----------|
| Different element types | Tear down old tree, build new tree from scratch. |
| Same DOM element type | Keep DOM node, update changed attributes. |
| Same component type | Keep instance, update props, re-render. |
| Children with keys | Match children by key, preserve state. |
| Children without keys | Match by position, may lose state. |

**Syntax Rules:**

1. Different types cause full rebuild and state loss.
2. Same types preserve DOM node and state.
3. `key` prop helps React match children across renders.

### Annotated Code Examples

```jsx
// ❌ Different types: full rebuild
<div><Counter /></div>
<span><Counter /></span>
// Old Counter is destroyed, new one mounted

// ✅ Same type: state preserved
<div><Counter /></div>
<div><Counter /></div>
// Counter instance stays the same
```

**Expected Output:** The first example loses Counter state; the second preserves it.

**Why this output:** React compares the root elements. `<div>` and `<span>` are different types, so React tears down the old tree and builds a new one. `<div>` and `<div>` are the same type, so React keeps the DOM node and component instance.

### Real-World Cases

- **List rendering:** Keys help React match items correctly when the list changes.
- **Conditional rendering:** Changing element types causes full rebuilds.
- **Tab switching:** Same component type preserves state across tab changes.

### References

- React — Reconciliation — https://ru.react.js.org/docs/reconciliation.html
- Steve Kinney — Understanding Reconciliation in React 19 — https://raw.githubusercontent.com

---

## 7. Diffing Algorithm

### Definitions

**Core Definition:** The diffing algorithm is the heuristic O(n) algorithm React uses to compare two element trees and determine the minimal changes.

**Technical Definition:** React's diffing algorithm compares two trees element by element. When comparing two elements of different types, React tears down the old tree and builds a new one. When comparing two elements of the same type, React updates only the changed attributes and recurses on children. For lists, React uses the `key` prop to match children across renders.

**Beginner-Friendly Explanation:** The diffing algorithm is how React compares two versions of your UI and finds the differences.

### Purposes

- To find the minimal set of changes between two trees.
- To make updates efficient (O(n) instead of O(n³)).
- To preserve DOM nodes and state where possible.

### Syntax Rules and Structure

**Diffing Algorithm Rules:**

| Comparison | Behaviour |
|------------|-----------|
| Different types | Full rebuild. |
| Same DOM element | Update changed attributes. |
| Same component | Update props, re-render. |
| Children with keys | Match by key. |

**Syntax Rules:**

1. React uses a heuristic O(n) algorithm based on two assumptions.
2. Different types cause full rebuild and state loss.
3. Keys help React identify stable children.

### Annotated Code Examples

```jsx
// ❌ No keys: React matches by position
<ul>
  {items.map((item) => <li>{item.text}</li>)}
</ul>

// ✅ With keys: React matches by key
<ul>
  {items.map((item) => <li key={item.id}>{item.text}</li>)}
</ul>
```

**Expected Output:** The version with keys preserves item state correctly when the list is reordered.

**Why this output:** Without keys, React matches children by position. When the list is reordered, React may match the wrong items and lose state. With keys, React matches items by their unique ID.

### Real-World Cases

- **Lists:** Keys ensure correct matching when items are added, removed, or reordered.
- **Dynamic content:** Conditional rendering with different types causes full rebuilds.

### References

- React — Reconciliation: The Diffing Algorithm — https://ru.react.js.org/docs/reconciliation.html

---

## 8. Virtual DOM

### Definitions

**Core Definition:** The virtual DOM (VDOM) is a programming concept where an ideal, or "virtual", representation of a UI is kept in memory and synced with the "real" DOM.

**Technical Definition:** The virtual DOM is a lightweight JavaScript representation of the DOM. React uses it to compute the minimal set of changes needed to update the real DOM. This process is called reconciliation. The virtual DOM enables React's declarative API — you describe what the UI should look like, and React figures out how to update the DOM efficiently.

**Beginner-Friendly Explanation:** The virtual DOM is a copy of your UI kept in memory. React compares it with the real DOM and updates only what changed.

### Purposes

- To enable declarative programming — you describe what, not how.
- To compute minimal DOM updates efficiently.
- To provide a consistent programming model across platforms.

### Syntax Rules and Structure

**Virtual DOM Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| In-memory | Kept in memory, not the real DOM. |
| Declarative | You describe the desired UI state. |
| Synced | Synced with the real DOM via reconciliation. |

**Syntax Rules:**

1. React elements form the virtual DOM tree.
2. Reconciliation compares the virtual DOM with the real DOM.
3. Only changed parts are updated in the real DOM.

### Annotated Code Examples

```jsx
function App() {
  return (
    <div>
      <h1>Hello</h1>
      <p>World</p>
    </div>
  );
}
```

**Expected Output:** The virtual DOM contains a `div` with `h1` and `p` children.

**Why this output:** React creates a virtual DOM tree from the JSX. This tree is compared with the real DOM, and only differences are applied.

### Real-World Cases

- **All React apps:** The virtual DOM is the foundation of React's rendering.
- **Cross-platform:** React Native uses the virtual DOM concept for native mobile components.
- **Server-side rendering:** The virtual DOM is used on the server to generate HTML.

### References

- React — Virtual DOM and Internals — https://ru.react.js.org/docs/faq-internals.html

---

## 9. React Elements

### Definitions

**Core Definition:** A React element is a lightweight description of a piece of the user interface.

**Technical Definition:** A React element is a plain object describing a component instance or DOM node and its desired properties. It contains only information about the component type, its properties (props), and any child elements. React elements are immutable — they are snapshots for a single render. Elements are not actual DOM nodes or component instances; they are descriptions.

**Beginner-Friendly Explanation:** A React element is a description of what you want to see on screen. It is not the actual DOM element — it is a blueprint.

### Purposes

- To describe the desired UI in a declarative way.
- To serve as the building blocks of the virtual DOM.
- To enable React to compute minimal DOM updates.

### Syntax Rules and Structure

**React Element Structure:**

```js
{
  type: 'div',           // HTML tag or component function
  props: {
    className: 'box',    // Props
    children: [...]      // Child elements
  },
  key: null              // Optional key
}
```

**Syntax Rules:**

1. React elements are produced by JSX or `React.createElement`.
2. Elements are immutable — do not modify them after creation.
3. Elements are plain objects, not DOM nodes.

### Annotated Code Examples

```jsx
// JSX produces a React element
const element = <h1 className="title">Hello</h1>;

// Equivalent createElement call
const element2 = React.createElement('h1', { className: 'title' }, 'Hello');
```

**Expected Output:** Both produce the same React element object.

**Why this output:** JSX is syntactic sugar for `React.createElement`. Both produce a plain object describing the `<h1>` element.

### Real-World Cases

- **Every render:** React elements are created during the render phase.
- **Virtual DOM:** Elements form the virtual DOM tree.

### References

- React — React Components, Elements, and Instances — https://id.legacy.reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html
- React — isValidElement — https://18.react.dev/reference/react/isValidElement

---

## 10. React Fiber

### Definitions

**Core Definition:** React Fiber is React's internal reconciliation engine that enables interruptible, prioritised rendering.

**Technical Definition:** React Fiber is an internal engine change geared to make React faster and smarter. The Fiber reconciler, which became the default reconciler for React 16 and above, is a complete rewrite of React's reconciliation algorithm. Because Fiber is asynchronous, React can pause, resume, and restart rendering work on components as new updates come in. It can reuse previously completed work and even abort it if not needed. It can also split work into chunks and prioritise tasks based on importance.

**Beginner-Friendly Explanation:** React Fiber is the engine inside React that makes rendering interruptible. It lets React pause work on less important updates to handle more urgent ones.

### Purposes

- To enable interruptible rendering.
- To prioritise updates based on importance.
- To split rendering work into small units.
- To improve perceived performance and responsiveness.

### Syntax Rules and Structure

**Fiber Capabilities:**

| Capability | Description |
|------------|-------------|
| Pause/Resume | Pause and resume rendering work. |
| Reuse | Reuse completed work or abort it. |
| Prioritise | Split work and prioritise tasks. |

**Syntax Rules:**

1. Fiber is the default reconciler for React 16+.
2. Fiber is asynchronous and interruptible.
3. Fiber enables concurrent rendering features.

### Annotated Code Examples

```jsx
// Fiber enables interruptible rendering
import { useTransition } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (value) => {
    setQuery(value); // Urgent: update input
    startTransition(() => {
      // Non-urgent: expensive search can be interrupted
      setResults(expensiveSearch(value));
    });
  };

  return (
    <input value={query} onChange={(e) => handleSearch(e.target.value)} />
  );
}
```

**Expected Output:** The input stays responsive while the search runs in the background.

**Why this output:** `useTransition` uses Fiber's interruptible rendering to prioritise the input update over the expensive search. The search can be interrupted if the user keeps typing.

### Real-World Cases

- **Search inputs:** Keeping input responsive while filtering large datasets.
- **Data visualisation:** Prioritising user interactions over chart rendering.
- **Navigation:** Prioritising route changes over background data loading.

### References

- LogRocket — A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Steve Kinney — Virtual DOM & Fiber Architecture — https://stevekinney.com

---

## 11. Fiber Tree

### Definitions

**Core Definition:** A Fiber tree is the internal data structure React uses to represent the component tree during rendering.

**Technical Definition:** React maintains two Fiber trees: the **current tree** (representing what is currently on screen) and the **work-in-progress tree** (representing what is being built during the current render). Each Fiber node corresponds to a component instance and contains information about the component's type, props, state, and child/sibling relationships. Fiber nodes are heap-allocated linked objects, allowing React to traverse the tree incrementally and interrupt work.

**Beginner-Friendly Explanation:** The Fiber tree is React's internal map of your component tree. React uses it to track what is on screen and what needs to be updated.

### Purposes

- To represent the component tree in a way that supports interruptible rendering.
- To track the current and work-in-progress states.
- To enable incremental traversal and prioritisation.

### Syntax Rules and Structure

**Fiber Tree Characteristics:**

| Characteristic | Description |
|----------------|-------------|
| Two trees | Current tree and work-in-progress tree. |
| Linked nodes | Each Fiber is a heap-allocated linked object. |
| Unit of work | Each Fiber represents one unit of rendering work. |

**Syntax Rules:**

1. React alternates between the current and work-in-progress trees.
2. Each Fiber node has `child`, `sibling`, and `return` pointers.
3. Fiber nodes store component type, props, state, and effect flags.

### Annotated Code Examples

```jsx
// Fiber tree structure (conceptual)
function App() {
  return (
    <div>
      <Header />
      <Content />
    </div>
  );
}
```

**Expected Output:** The Fiber tree has nodes for `App`, `div`, `Header`, and `Content`.

**Why this output:** React creates a Fiber node for each component and DOM element. The tree structure mirrors the component hierarchy, with linked pointers for traversal.

### Real-World Cases

- **All React apps:** The Fiber tree is the internal representation of the component tree.
- **Concurrent rendering:** Fiber tree enables interruptible rendering.

### References

- Educative — Understanding the Fiber Architecture — https://www.educative.io
- LogRocket — A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/

---

## 12. Work Loop

### Definitions

**Core Definition:** The work loop is the mechanism React uses to process units of work in the Fiber tree incrementally.

**Technical Definition:** Starting in React 16, Fiber replaced a single recursive render pass with a cooperative, interruptible work loop. React processes the work-in-progress tree one Fiber at a time. After each unit of work, React checks if there is time remaining in the current frame. If not, it yields control back to the browser, allowing the browser to handle higher-priority tasks like user input. When time is available, React resumes the work loop.

**Beginner-Friendly Explanation:** The work loop is React's way of processing rendering work in small chunks, so it can pause and respond to user input.

### Purposes

- To process rendering work incrementally.
- To allow interruption for higher-priority tasks.
- To keep the UI responsive during expensive renders.

### Syntax Rules and Structure

**Work Loop Pseudocode:**

```js
function workLoop() {
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

**Syntax Rules:**

1. React processes one Fiber at a time.
2. `shouldYield()` checks if time is remaining in the frame.
3. If time runs out, React yields control to the browser.

### Annotated Code Examples

```jsx
// Conceptual: the work loop allows interruption
function workLoop() {
  while (workInProgress && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
  if (workInProgress) {
    // Schedule continuation
    scheduleCallback(workLoop);
  }
}
```

**Expected Output:** Rendering work is split into chunks and processed over multiple frames.

**Why this output:** The work loop processes as many units as time allows, then yields. When the browser is free, React resumes the work.

### Real-World Cases

- **Large component trees:** The work loop prevents long blocking renders.
- **Concurrent features:** `useTransition` and `useDeferredValue` rely on the work loop.

### References

- Educative — Understanding the Fiber Architecture — https://www.educative.io
- LogRocket — A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/

---

## 13. Rendering Priority

### Definitions

**Core Definition:** Rendering priority is the classification of updates based on their urgency, determining which updates React processes first.

**Technical Definition:** React's concurrent scheduler introduces time-slicing and priority-based scheduling. The scheduler uses a priority hierarchy: ImmediatePriority (user input, focus events), UserBlockingPriority (click handlers, form inputs), NormalPriority (data fetching), LowPriority (analytics, logging), and IdlePriority (background cleanup). The scheduler can interrupt lower-priority work when higher-priority work arrives, ensuring the app stays responsive to user interactions.

**Beginner-Friendly Explanation:** Not all updates are equally urgent. A user typing should be handled immediately; updating a chart can wait. React prioritises urgent updates over less urgent ones.

### Purposes

- To keep the UI responsive to user interactions.
- To prioritise urgent updates over background work.
- To enable smooth animations and transitions.

### Syntax Rules and Structure

**Priority Levels:**

| Priority | Description | Examples |
|----------|-------------|----------|
| Immediate | User input, focus events | Typing, clicking |
| User Blocking | Click handlers, form inputs | Button clicks |
| Normal | Data fetching, network requests | API responses |
| Low | Analytics, logging | Tracking events |
| Idle | Background cleanup tasks | Cache cleanup |

**Syntax Rules:**

1. React automatically assigns priorities to updates.
2. `useTransition` and `useDeferredValue` let you mark updates as non-urgent.
3. Higher-priority updates can interrupt lower-priority work.

### Annotated Code Examples

```jsx
import { useTransition, useState } from 'react';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (value) => {
    setQuery(value); // Urgent: update input immediately
    startTransition(() => {
      // Non-urgent: expensive search can be interrupted
      setResults(expensiveSearch(value));
    });
  };

  return (
    <div>
      <input value={query} onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <div>Searching...</div>}
      <ResultsList results={results} />
    </div>
  );
}
```

**Expected Output:** The input stays responsive while the search runs in the background.

**Why this output:** `setQuery` is an urgent update that React processes immediately. The search inside `startTransition` is marked as non-urgent and can be interrupted if the user keeps typing.

### Real-World Cases

- **Search inputs:** Keeping typing responsive while filtering results.
- **Animations:** Ensuring smooth animations by prioritising them over data updates.
- **Form submissions:** Prioritising user input over background validation.

### References

- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 14. Rendering Scheduling

### Definitions

**Core Definition:** Rendering scheduling is React's mechanism for deciding when and in what order to process rendering work.

**Technical Definition:** React's scheduler uses time-slicing and priority-based scheduling to process rendering work. It breaks rendering into small units and processes them during browser idle time. The scheduler can interrupt lower-priority work when higher-priority work arrives. React 18 introduced concurrent features that fundamentally change how React processes updates.

**Beginner-Friendly Explanation:** Scheduling is how React decides what work to do first and when to pause to let the browser respond to the user.

### Purposes

- To keep the UI responsive during expensive renders.
- To prioritise urgent updates.
- To make efficient use of browser idle time.

### Syntax Rules and Structure

**Scheduling Mechanisms:**

| Mechanism | Description |
|-----------|-------------|
| Time-slicing | Split work into small chunks. |
| Priority-based | Process high-priority work first. |
| Interruptible | Pause work for higher-priority tasks. |

**Syntax Rules:**

1. React schedules work based on priority.
2. `useTransition` marks updates as non-urgent.
3. `useDeferredValue` defers expensive computations.

### Annotated Code Examples

```jsx
import { useDeferredValue, useMemo } from 'react';

function ExpensiveChart({ data }) {
  const deferredData = useDeferredValue(data);
  const chartElements = useMemo(() => {
    return deferredData.map((value, index) => (
      <ChartBar key={index} value={value} />
    ));
  }, [deferredData]);

  return <div>{chartElements}</div>;
}
```

**Expected Output:** The chart updates with a slight delay, keeping the UI responsive.

**Why this output:** `useDeferredValue` creates a deferred version of `data` that lags behind during rapid updates. The expensive computation uses the deferred value, allowing React to prioritise more urgent updates.

### Real-World Cases

- **Data visualisation:** Deferring chart updates during rapid data changes.
- **Search results:** Keeping input responsive while search runs.
- **Large lists:** Deferring rendering of off-screen items.

### References

- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling

---

## 15. Rendering Batching

### Definitions

**Core Definition:** Rendering batching is React's practice of grouping multiple state updates into a single re-render for performance.

**Technical Definition:** React waits until all code in the event handler has run before processing state updates. This lets you update multiple state variables without triggering too many re-renders. This behaviour, also known as batching, makes your React app run much faster. It also avoids dealing with confusing "half-finished" renders where only some of the variables have been updated.

**Beginner-Friendly Explanation:** Batching is React waiting until you finish telling it what to change, then doing all the updates at once instead of one by one.

### Purposes

- To reduce the number of re-renders.
- To improve performance.
- To avoid inconsistent UI states.

### Syntax Rules and Structure

**Batching Behaviour:**

| Scenario | Behaviour |
|----------|-----------|
| Multiple state updates in one event handler | Batched into one re-render. |
| Updates from multiple components | Batched into one re-render. |
| Updates across separate events | Not batched — each event is separate. |

**Syntax Rules:**

1. React batches updates within event handlers.
2. React does not batch across separate intentional events.
3. Use functional updates to update the same state multiple times.

### Annotated Code Examples

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // Queued
    setCount(count + 1); // Queued
    setCount(count + 1); // Queued
    // React batches these into ONE re-render
    // But all three use the same stale count (0)
  }

  return <button onClick={handleClick}>+3 (but only +1)</button>;
}
```

**Expected Output:** The counter increments by 1, not 3, because of stale state.

**Why this output:** React batches the three `setCount` calls, but each uses the same `count` value (0) from the current render. To fix, use `setCount(c => c + 1)`.

### Real-World Cases

- **Form submissions:** Updating multiple fields before a single re-render.
- **Complex UI state:** Toggling multiple flags together.
- **Data fetching:** Setting loading, data, and error states together.

### References

- React — Queueing a Series of State Updates — https://hi.react.dev/learn/queueing-a-series-of-state-updates

---

## 16. Automatic Batching

### Definitions

**Core Definition:** Automatic batching is React 18's feature that batches all state updates by default, including those outside React event handlers.

**Technical Definition:** Starting in React 18 with `createRoot`, all updates are automatically batched, no matter where they originate from. This includes updates inside promises, `setTimeout`, and native event handlers. Previously, React only batched updates inside React event handlers. Automatic batching reduces unnecessary renders and improves application performance.

**Beginner-Friendly Explanation:** Automatic batching means React groups all your state updates together, even the ones that happen after an API call or inside a timer.

### Purposes

- To reduce unnecessary renders.
- To improve performance across all update sources.
- To provide consistent batching behaviour.

### Syntax Rules and Structure

**Automatic Batching Scope:**

| Source | React 17 | React 18+ |
|--------|----------|-----------|
| React event handlers | Batched | Batched |
| Promises | Not batched | Batched |
| setTimeout | Not batched | Batched |
| Native event handlers | Not batched | Batched |

**Syntax Rules:**

1. Use `createRoot` to enable automatic batching.
2. Use `flushSync` to opt out of batching (rarely recommended).
3. Automatic batching is enabled by default in React 18+.

### Annotated Code Examples

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetch('/api').then(() => {
      setCount(c => c + 1);  // Batched in React 18
      setFlag(f => !f);       // Batched in React 18
    });
  }

  return <button onClick={handleClick}>Update</button>;
}
```

**Expected Output:** Both state updates are batched into a single re-render.

**Why this output:** In React 18, updates inside promises are automatically batched. Both state updates are processed together after the promise resolves.

### Real-World Cases

- **Data fetching:** Setting data, loading, and error states together after a fetch.
- **Async operations:** Batching updates in `setTimeout` and promises.
- **Native event handlers:** Batching updates from non-React event listeners.

### References

- React Blog — React v18.0: Automatic Batching — https://vi.legacy.reactjs.org/blog/2022/03/29/react-v18.html
- Educative — Concurrent Rendering and Automatic Batching — https://www.educative.io

---

## 17. Rendering Triggers

### Definitions

**Core Definition:** Rendering triggers are the events that cause React to render or re-render a component.

**Technical Definition:** There are four events that cause a React component to render: state change, prop change, parent render, and context change. State change occurs when a component's state is updated with its set function. Prop change occurs when a parent passes new props. Parent render occurs when a parent re-renders, causing all children to re-render. Context change occurs when a context value changes.

**Beginner-Friendly Explanation:** These are the things that cause React to update the screen.

### Purposes

- To understand what causes re-renders.
- To optimise performance by controlling re-renders.
- To debug unnecessary renders.

### Syntax Rules and Structure

**Render Triggers:**

| Trigger | Description |
|---------|-------------|
| State change | `setState` or `useState` setter |
| Prop change | Parent passes new props |
| Parent render | Parent re-renders |
| Context change | Context value changes |

**Syntax Rules:**

1. State changes queue a re-render of the component.
2. Parent re-renders cause child re-renders by default.
3. `React.memo` can prevent re-renders from prop changes.

### Annotated Code Examples

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <Child />  {/* Re-renders when Parent re-renders */}
    </div>
  );
}

function Child() {
  console.log('Child rendered');
  return <p>Child</p>;
}
```

**Expected Output:** "Child rendered" logs on every Parent re-render.

**Why this output:** When `Parent` re-renders due to the state change, React re-renders all its children by default, including `Child`.

### Real-World Cases

- **Debugging:** Identifying why components re-render.
- **Performance:** Using `React.memo` to prevent unnecessary child re-renders.
- **State management:** Understanding when state changes trigger re-renders.

### References

- Cory House — Four Reasons React Renders — https://coryhouse.dev

---

## 18. Rendering Optimization

### Definitions

**Core Definition:** Rendering optimization is the practice of reducing unnecessary renders and improving the performance of React rendering.

**Technical Definition:** Rendering optimization techniques include memoization (`React.memo`, `useMemo`, `useCallback`), virtualization, code splitting, and concurrent features (`useTransition`, `useDeferredValue`). Memoization prevents unnecessary re-renders and recalculations. Virtualization renders only visible items in large lists. Code splitting loads components on demand. Concurrent features prioritise urgent updates and defer non-urgent work.

**Beginner-Friendly Explanation:** Rendering optimization is making your app faster by doing less unnecessary work.

### Purposes

- To reduce the number of re-renders.
- To improve perceived performance.
- To keep large UIs responsive.

### Syntax Rules and Structure

**Optimization Techniques:**

| Technique | Description |
|-----------|-------------|
| `React.memo` | Skip re-render if props unchanged |
| `useMemo` | Cache expensive computations |
| `useCallback` | Cache function references |
| Virtualization | Render only visible items |
| Code splitting | Load components on demand |
| `useTransition` | Mark updates as non-urgent |
| `useDeferredValue` | Defer expensive computations |

**Syntax Rules:**

1. Profile first — use React DevTools Profiler to identify bottlenecks.
2. Apply memoization only when there is a measurable problem.
3. Use concurrent features to prioritise urgent updates.

### Annotated Code Examples

```jsx
import { memo, useMemo, useCallback } from 'react';

const ExpensiveChild = memo(function ExpensiveChild({ data, onAction }) {
  return <div>{data.join(', ')}</div>;
});

function Parent({ items }) {
  const [count, setCount] = useState(0);

  const sortedItems = useMemo(() => {
    return [...items].sort();
  }, [items]);

  const handleAction = useCallback(() => {
    console.log('Action');
  }, []);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveChild data={sortedItems} onAction={handleAction} />
    </div>
  );
}
```

**Expected Output:** `ExpensiveChild` only re-renders when `sortedItems` or `handleAction` changes.

**Why this output:** `React.memo` prevents re-renders when props are referentially equal. `useMemo` keeps `sortedItems` stable until `items` changes. `useCallback` keeps `handleAction` stable.

### Real-World Cases

- **Large lists:** Virtualization renders only visible items.
- **Data visualisation:** Deferring expensive chart updates.
- **Design systems:** Memoizing components to prevent re-render cascades.

### References

- Amazon Developer — Boost Application Performance with Concurrent Rendering — https://www.developer.amazon.com
- Sourcetrail — React Performance Optimization: Techniques That Actually Work — https://www.sourcetrail.com
- RT Camp — Performance Optimization for React-based projects — https://rtcamp.com

---

## References

- React — Render and Commit — https://18.react.dev/learn/render-and-commit
- React — Reconciliation — https://ru.react.js.org/docs/reconciliation.html
- React — Virtual DOM and Internals — https://ru.react.js.org/docs/faq-internals.html
- React — React Components, Elements, and Instances — https://id.legacy.reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html
- React — isValidElement — https://18.react.dev/reference/react/isValidElement
- React — Queueing a Series of State Updates — https://hi.react.dev/learn/queueing-a-series-of-state-updates
- React Blog — React v18.0: Automatic Batching — https://vi.legacy.reactjs.org/blog/2022/03/29/react-v18.html
- LogRocket — A deep dive into React Fiber — https://blog.logrocket.com/deep-dive-react-fiber/
- Steve Kinney — Understanding Reconciliation in React 19 — https://github.com/stevekinney/stevekinney.net
- Steve Kinney — Concurrent React and Scheduling — https://stevekinney.com/courses/react-performance/concurrent-react-scheduling
- Steve Kinney — Virtual DOM & Fiber Architecture — https://stevekinney.com
- Educative — Understanding the Fiber Architecture — https://www.educative.io
- Educative — Concurrent Rendering and Automatic Batching — https://www.educative.io
- Cory House — Four Reasons React Renders — https://coryhouse.dev
- Amazon Developer — Boost Application Performance with Concurrent Rendering — https://www.developer.amazon.com
- Sourcetrail — React Performance Optimization: Techniques That Actually Work — https://www.sourcetrail.com
- RT Camp — Performance Optimization for React-based projects — https://rtcamp.com
- GitHub — Understanding Reconciliation in React 19 — https://raw.githubusercontent.com
- GitHub — React Core Architecture — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Alibaba Cloud — A Closer Look at React Fiber — https://www.alibabacloud.com
- InfoQ — React Fiber: A Closer Look at the New Engine of React — https://www.infoq.com