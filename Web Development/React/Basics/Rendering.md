# React Rendering — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

Rendering in React is the process by which React determines what the user interface should look like by calling components and producing React elements, then updating the DOM to match those elements.

**Technical Definition**

Rendering is React's process of invoking component functions (or `render()` methods) to produce a tree of React elements—lightweight JavaScript objects describing what should appear on screen. When state or props change, React re-renders the affected components, compares the new element tree with the previous one through a process called reconciliation, and computes the minimal set of DOM mutations required to update the actual browser DOM.

**Beginner-Friendly Explanation**

Think of rendering like a chef preparing a meal. You (the parent component) place an order (state change). The chef (React) goes into the kitchen and prepares the dish (calls your components to create a description of what should appear). Then a waiter takes the finished dish and places it on your table (React updates the DOM). The chef doesn't hand you ingredients—they hand you a finished plate. And if you order again later, the chef doesn't start from scratch; they just change what needs to be different.

### Key Characteristics

- **Pure Function Behavior**: Rendering must be a pure calculation—same inputs produce the same JSX output, and no external variables are modified during render .
- **Declarative**: You describe what the UI should look like for a given state; React figures out how to make the DOM match.
- **Recursive**: React renders the root component, then recursively renders any child components returned by that component, continuing until the entire tree is resolved .
- **Non-DOM-Immediate**: Rendering a component does **not** mean the DOM is immediately updated. The render phase and commit phase are distinct .

### Prerequisites

- React Components and JSX
- State and Props concepts
- Basic understanding of the DOM
- React 18+ (for `createRoot` API)

### Related Programming Areas

- **Reconciliation**: The diffing algorithm React uses to compare element trees.
- **Component Lifecycle**: The phases a component goes through from mount to unmount.
- **React Fiber**: React's internal reconciliation engine that enables concurrent rendering.
- **Virtual DOM**: The in-memory representation of the UI that React maintains.

### Core Concepts / Features

- Rendering React Elements
- Root Rendering
- Re-rendering
- Render Phase
- Commit Phase
- Component Lifecycle Concepts
- React Reconciliation

---

## 1. Rendering React Elements

### Definitions

**Core Definition**

Rendering React elements is the act of calling component functions to produce React elements—JavaScript objects that describe what should appear on screen.

**Technical Definition**

When React renders, it invokes component functions (or `render()` methods in class components). Each component returns React elements, which are plain JavaScript objects with a `type` (a string like `'div'` or a component function) and `props`. React uses these element objects to build an in-memory tree representation of the UI before any DOM operations occur .

**Beginner-Friendly Explanation**

Rendering elements is like a chef describing a dish before cooking. The description says "a plate with a burger, fries, and a pickle." It's not the actual food yet—just a precise description that the kitchen (React) uses to prepare the real meal (DOM nodes).

### Purposes

- To create a lightweight, in-memory description of the desired UI.
- To allow React to compute what needs to change before touching the real DOM.
- To enable declarative UI development where developers describe "what" not "how."

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function Component() {
  return (
    <div>
      <h1>Hello</h1>
    </div>
  );
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function Component()` | The component function React calls during render. |
| `return (...)` | Returns JSX, which React converts into React elements. |
| `<div>...</div>` | JSX syntax describing the UI structure. |

**Syntax Rules**

1. Render must be **pure**: Same inputs → same output. Do not modify variables outside the component during render .
2. Components can return other components, enabling recursive rendering.
3. In Strict Mode, React calls each component twice in development to detect impurity .

**Constraints and Limitations**

- Do not perform side effects (API calls, DOM mutations) during render.
- Do not modify state or props during render.

### Annotated Complete Code Examples

**Example 1: Basic Element Rendering**

```jsx
function Greeting() {
  // This function IS the render. React calls it during rendering.
  return <h1>Hello, React!</h1>;
}

export default function App() {
  return (
    <div>
      <Greeting />
    </div>
  );
}
```

**Expected Output**

```
Hello, React!
```

**Why This Output Occurs**

React calls `App()`, which returns a `<div>` element containing a `<Greeting />` element. React then calls `Greeting()`, which returns an `<h1>` element. React now has a tree: `div > h1 > "Hello, React!"`. This element tree is the "rendered output" before any DOM updates.

**Example 2: Recursive Rendering**

```jsx
function Item({ name }) {
  return <li>{name}</li>;
}

function List() {
  const items = ['Apple', 'Banana', 'Cherry'];
  return (
    <ul>
      {items.map((item, index) => (
        <Item key={index} name={item} />
      ))}
    </ul>
  );
}

export default function App() {
  return <List />;
}
```

**Expected Output**

```
• Apple
• Banana
• Cherry
```

**Why This Output Occurs**

React calls `App()` → returns `<List />`. React calls `List()` → returns `<ul>` with three `<Item />` elements. React then calls `Item()` three times, once for each name. The rendering is recursive: React follows the component tree until all leaves are resolved .

### Real-World Cases

**Case: Rendering a Dashboard**

A `Dashboard` component renders multiple child components (`StatsPanel`, `ActivityFeed`, `UserList`). Each child recursively renders its own children. React resolves the entire tree in memory before any DOM updates occur.

### References

- React — Render and Commit - https://react.dev/learn/render-and-commit

---

## 2. Root Rendering

### Definitions

**Core Definition**

Root rendering is the initial rendering that occurs when the application starts, triggered by calling `createRoot()` and `root.render()` with the root component.

**Technical Definition**

In React 18+, `ReactDOM.createRoot(domNode)` creates a React root bound to a DOM element. Calling `root.render(<App />)` triggers the initial render pass, which renders the root component and its entire subtree. This is the entry point for client-side React applications.

**Beginner-Friendly Explanation**

Root rendering is like opening a restaurant for the first time. You unlock the doors (`createRoot`), turn on the lights, and the kitchen starts preparing the first orders (`root.render`). The root component is the head chef who coordinates everything.

### Purposes

- To initialize the React rendering context bound to a specific DOM element.
- To trigger the first render pass that produces the initial UI.
- To enable React's concurrent features and automatic batching (React 18+).

### Syntax Rules and Structure

**Complete General Syntax (React 18+)**

```jsx
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createRoot(domNode)` | Creates a React root bound to the DOM node. |
| `root.render(element)` | Triggers the initial render of the element tree. |

**Syntax Rules**

1. Use `react-dom/client` (not `react-dom`) for the new API.
2. The DOM node passed to `createRoot()` must already exist in the document.
3. `ReactDOM.render()` is **deprecated** in React 18+ and should not be used .

**Constraints and Limitations**

- Only one React root can be created per DOM element.
- The old `ReactDOM.render()` API is legacy and does not support concurrent features.

### Annotated Complete Code Examples

**Example 1: Initial Root Render**

```jsx
// main.jsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

// Find the DOM element in index.html
const container = document.getElementById('root');

// Create the React root
const root = createRoot(container);

// Trigger the initial render
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

**Expected Behavior**

The application renders into the `#root` div. In Strict Mode, React calls components twice in development to detect impurities.

**Why This Occurs**

`createRoot()` establishes the React root context. `root.render()` queues the initial render. React calls `App()` and recursively renders all children, computes the necessary DOM operations, and commits them to the DOM.

**Example 2: Legacy vs. New API**

```jsx
// ❌ DEPRECATED: Do not use in React 18+
import ReactDOM from 'react-dom';
ReactDOM.render(<App />, document.getElementById('root'));

// ✅ CORRECT: React 18+
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**Explanation**

The old API showed a console warning in React 18 and does not enable concurrent features. The new `createRoot` API supports automatic batching and concurrent rendering .

### Real-World Cases

**Case: Server-Side Rendering Hydration**

In SSR, the server renders HTML, and the client "hydrates" it. The client uses `hydrateRoot(container, <App />)` instead of `createRoot().render()` .

### References

- React — createRoot API Reference - https://react.dev/reference/react-dom/client/createRoot
- React — How to Upgrade to React 18 - https://react.dev/blog/2022/03/08/react-18-upgrade-guide

---

## 3. Re-rendering

### Definitions

**Core Definition**

Re-rendering is the process by which React calls a component function again to produce a new element tree in response to a state change.

**Technical Definition**

Once a component has been initially rendered, updating its state (or the state of an ancestor) triggers a re-render. React calls the component function again, receives a new JSX snapshot, and reconciles it with the previous snapshot to determine the minimal DOM updates required.

**Beginner-Friendly Explanation**

Re-rendering is like a chef revising a dish after a customer changes their order. The chef doesn't start from scratch—they just change what's different. If you ordered a burger and then said "no pickles," the chef remakes the burger without pickles, not the whole meal.

### Purposes

- To reflect state changes in the UI.
- To update the DOM with minimal operations.
- To keep the user interface in sync with application data.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [count, setCount] = useState(0);

// Clicking this button triggers a re-render
<button onClick={() => setCount(count + 1)}>
  Count: {count}
</button>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `useState(0)` | Declares state. Returns `[value, setter]`. |
| `setCount(newValue)` | Updates state, queuing a re-render. |

**Syntax Rules**

1. **State updates trigger re-renders**. React queues a re-render when a setter is called .
2. **Re-renders are recursive**: When a component re-renders, React also re-renders its children by default (unless memoized).
3. **State is a snapshot**: Within a single render, state values are constant. Calling `setCount(count + 1)` three times in one handler still only increments by 1 per render .

**Constraints and Limitations**

- Props changes alone do not directly cause re-renders; they are a consequence of a parent re-rendering .
- Re-rendering does not always mean the DOM changes—React only commits differences.

### Annotated Complete Code Examples

**Example 1: Re-render on State Change**

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  console.log('Counter rendered'); // Runs on every render

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

export default function App() {
  console.log('App rendered');
  return <Counter />;
}
```

**Expected Output (clicking button twice)**

```
App rendered
Counter rendered
App rendered        ← App re-renders? No! Only Counter re-renders.
Counter rendered    ← But wait, App is the parent...
```

**Why This Output Occurs**

Actually, `App` does **not** re-render because its state didn't change. Only `Counter` re-renders when `setCount` is called. This is a key insight: state changes only re-render the component that owns the state and its descendants .

**Example 2: The State Snapshot Behavior**

```jsx
import { useState } from 'react';

function Form() {
  const [count, setCount] = useState(0);

  function handleClick() {
    // All three calls use the SAME count value (0) from this render
    setCount(count + 1); // setCount(0 + 1) → queues 1
    setCount(count + 1); // setCount(0 + 1) → queues 1 again
    setCount(count + 1); // setCount(0 + 1) → queues 1 again
    // Result: count becomes 1, not 3
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

**Expected Output (clicking once)**

```
Count: 1
```

**Why This Output Occurs**

State behaves as a snapshot. In a single render, `count` is `0`. All three `setCount(count + 1)` calls compute `0 + 1 = 1`. React processes these as three calls to set the same value (1), resulting in a single re-render with `count = 1` .

### Real-World Cases

**Case: Form Input**

Typing in an input triggers `setState` on every keystroke, causing the input component to re-render and display the new value. React only updates the DOM attribute that changed.

### References

- React — Render and Commit (Re-renders when state updates) - https://react.dev/learn/render-and-commit
- React — State as a Snapshot - https://react.dev/learn/state-as-a-snapshot

---

## 4. Render Phase

### Definitions

**Core Definition**

The render phase is the first part of React's update process, during which React calls component functions to compute the new element tree without performing any DOM operations.

**Technical Definition**

The render phase is when React executes component functions (and `render()` methods), computes the resulting React elements, and performs reconciliation (diffing) between the old and new element trees. It is **pure** and can be interrupted, paused, or restarted in concurrent mode .

**Beginner-Friendly Explanation**

The render phase is the chef planning and preparing the dish in the kitchen. The chef looks at the order (state), figures out what needs to change from the previous order, and prepares the updated dish. Nothing has been sent to the customer yet—the dish is still in the kitchen.

### Purposes

- To compute the new element tree based on current state and props.
- To perform reconciliation (diffing) to determine what changed.
- To prepare a list of changes that need to be applied in the commit phase.

### Syntax Rules and Structure

**Complete General Syntax (Conceptual)**

```
Trigger → Render Phase → Commit Phase
            ├── Call components
            ├── Produce elements
            └── Reconcile (diff)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Call components | React invokes component functions. |
| Produce elements | Components return JSX/elements. |
| Reconcile | React compares old and new trees. |

**Syntax Rules**

1. **Must be pure**: No side effects, no DOM mutations, no state updates during render .
2. **Can be interrupted**: In concurrent mode, React may pause or restart the render phase.
3. **Strict Mode double-invokes**: In development, React calls components twice to detect impurity .

**Constraints and Limitations**

- Do not call `setState` during render (would cause infinite loops).
- Do not perform API calls or DOM operations during render.

### Annotated Complete Code Examples

**Example 1: Pure Render Phase**

```jsx
function UserProfile({ name, age }) {
  // This is the render phase: computing what to show
  const greeting = `Hello, ${name}`; // Pure calculation
  const ageLabel = `Age: ${age}`;    // Pure calculation

  return (
    <div>
      <h1>{greeting}</h1>
      <p>{ageLabel}</p>
    </div>
  );
}
```

**Explanation**

The render phase computes `greeting` and `ageLabel` based on props. These are pure calculations—no side effects. React then creates elements describing the UI. No DOM has been touched yet.

**Example 2: Impure Render (Anti-pattern)**

```jsx
function BadComponent() {
  // ❌ IMPURE: Mutating external variable during render
  window.someGlobal = 'changed';

  // ❌ IMPURE: Random output for same input
  return <div>{Math.random()}</div>;
}
```

**Why This Is Wrong**

Mutating external state during render violates purity. `Math.random()` produces different output for the same input, making the component unpredictable. In Strict Mode, React will call this component twice, exposing the impurity .

### Real-World Cases

**Case: Concurrent Rendering**

In React 18's concurrent mode, the render phase can be interrupted. If a high-priority update (like user input) arrives while a low-priority render (like a large list) is in progress, React can pause the list render, handle the input, and resume later.

### References

- React — Render and Commit (Step 2: React renders your components) - https://react.dev/learn/render-and-commit
- Epic React — Intro to Optimize Rendering - https://www.epicreact.dev/workshops/react-performance/intro-to-optimize-rendering

---

## 5. Commit Phase

### Definitions

**Core Definition**

The commit phase is the second part of React's update process, during which React applies the computed changes to the actual DOM.

**Technical Definition**

After the render phase computes the differences between the old and new element trees, the commit phase applies those differences to the DOM. For initial renders, React uses `appendChild()` to insert all created nodes. For updates, React applies the minimal set of mutations required to make the DOM match the new element tree. The commit phase is **synchronous** and cannot be interrupted .

**Beginner-Friendly Explanation**

The commit phase is the waiter bringing the finished dish from the kitchen to your table. The chef (render phase) has prepared it; now it's actually placed in front of you (the DOM is updated). Once the waiter sets it down, you can interact with it.

### Purposes

- To apply the minimal DOM mutations needed to match the new element tree.
- To update the browser's visual representation of the UI.
- To trigger lifecycle methods and effects after DOM changes.

### Syntax Rules and Structure

**Complete General Syntax (Conceptual)**

```
Render Phase (compute changes)
    ↓
Commit Phase
    ├── Apply DOM mutations
    ├── Run useLayoutEffect
    └── Browser paints
    ↓
    ├── Run useEffect
    └── Run componentDidMount/DidUpdate
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Apply DOM mutations | React updates, inserts, or removes DOM nodes. |
| Browser paints | The browser visually updates the screen. |
| Effects | `useEffect`, `componentDidMount`, etc. run after paint. |

**Syntax Rules**

1. **Synchronous**: The commit phase runs to completion without interruption.
2. **Minimal mutations**: React only changes what's different—it doesn't rebuild the entire DOM .
3. **Effects after commit**: `useEffect` runs after the browser paints; `useLayoutEffect` runs before paint .

**Constraints and Limitations**

- The commit phase cannot be interrupted; long commits can cause jank.
- DOM mutations happen here, so this is where actual performance costs occur.

### Annotated Complete Code Examples

**Example 1: Minimal DOM Mutation**

```jsx
function Clock() {
  const [time, setTime] = useState(new Date().toLocaleTimeString());

  useEffect(() => {
    const timer = setInterval(() => {
      setTime(new Date().toLocaleTimeString());
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  return (
    <div>
      <h1>Current time: {time}</h1>
      <input type="text" placeholder="Type here..." />
    </div>
  );
}
```

**Expected Behavior**

Every second, the time updates. The `<input>` element and any text typed into it remain unchanged.

**Why This Occurs**

During the commit phase, React compares the old and new element trees. It sees that only the text inside `<h1>` changed. The `<input>` element is in the same position and has the same props, so React does not touch it . This is why typing in the input doesn't get interrupted by the clock updates.

**Example 2: Commit Phase and Effects**

```jsx
function Logger() {
  console.log('Render phase'); // Runs during render

  useEffect(() => {
    console.log('Commit phase (effect)'); // Runs after DOM update
  });

  useLayoutEffect(() => {
    console.log('Before paint (layout effect)'); // Runs before browser paint
  });

  return <div>Check the console</div>;
}
```

**Expected Output (on mount)**

```
Render phase
Before paint (layout effect)
Commit phase (effect)
```

**Why This Output Occurs**

The render phase runs first (component function called). Then the commit phase applies DOM mutations. `useLayoutEffect` runs synchronously after DOM mutations but before the browser paints. The browser paints. Then `useEffect` runs asynchronously after paint .

### Real-World Cases

**Case: Animations and Layout Measurements**

`useLayoutEffect` is used when you need to read layout information (like element dimensions) and synchronously apply changes before the browser paints, preventing visual flicker.

### References

- React — Render and Commit (Step 3: React commits changes to the DOM) - https://react.dev/learn/render-and-commit
- React — useLayoutEffect vs useEffect - https://react.dev/reference/react/useLayoutEffect

---

## 6. Component Lifecycle Concepts

### Definitions

**Core Definition**

Component lifecycle is the series of phases a React component goes through from its creation (mounting) to its removal (unmounting), including updates in between.

**Technical Definition**

Every React component has a lifecycle with three main phases: **Mounting** (component is created and inserted into the DOM), **Updating** (component re-renders due to state or props changes), and **Unmounting** (component is removed from the DOM). In class components, specific methods are called at each phase. In function components, Hooks like `useEffect` provide equivalent functionality.

**Beginner-Friendly Explanation**

A component's lifecycle is like a person's life stages. A component is born (mounted), lives and changes (updates), and eventually dies (unmounts). At each stage, you can run code—like setting up a timer when the component is born, updating it when things change, and cleaning up when it dies.

### Purposes

- To run side effects at the right time (data fetching, subscriptions, timers).
- To clean up resources when a component is removed.
- To optimize performance by controlling when updates occur.

### Syntax Rules and Structure

**Complete General Syntax (Function Components with Hooks)**

```jsx
import { useState, useEffect } from 'react';

function Component() {
  const [data, setData] = useState(null);

  // Mounting + Updating: runs after every render (or when deps change)
  useEffect(() => {
    // Side effect (e.g., fetch data)
    fetchData().then(setData);

    // Unmounting: cleanup function
    return () => {
      // Cleanup (e.g., cancel fetch, clear timer)
    };
  }, []); // Empty array = run once on mount, cleanup on unmount

  return <div>{data}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `useEffect(callback, deps)` | Runs after render. |
| `return () => {}` | Cleanup function runs on unmount or before next effect. |
| `[]` | Dependency array: empty = mount/unmount only. |

**Complete General Syntax (Class Components — Legacy)**

```jsx
class Component extends React.Component {
  constructor(props) { super(props); this.state = {}; }
  componentDidMount() { /* Mounting */ }
  shouldComponentUpdate(nextProps, nextState) { return true; }
  componentDidUpdate(prevProps, prevState) { /* Updating */ }
  componentWillUnmount() { /* Unmounting */ }
  render() { return <div />; }
}
```

**Syntax Rules**

1. **Function components**: Use `useEffect` for mount, update, and unmount behaviors .
2. **Class components**: Use lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`).
3. **Empty dependency array** `[]`: Effect runs only on mount; cleanup runs on unmount.
4. **No dependency array**: Effect runs after every render.

**Constraints and Limitations**

- `componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate` are **deprecated** and should not be used in new code .
- Hooks must be called at the top level—not inside conditions or loops.

### Annotated Complete Code Examples

**Example 1: Timer with Cleanup (Function Component)**

```jsx
import { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // MOUNTING: Set up the timer
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    // UNMOUNTING: Clean up the timer
    return () => clearInterval(interval);
  }, []); // Empty deps: run once on mount

  return <h1>Seconds: {seconds}</h1>;
}
```

**Expected Behavior**

The timer increments every second. When the component is removed from the DOM, the interval is cleared, preventing memory leaks.

**Why This Occurs**

`useEffect` with `[]` runs once after the initial render (mounting). The cleanup function runs when the component unmounts .

**Example 2: Class Component Lifecycle (Legacy)**

```jsx
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { seconds: 0 };
  }

  componentDidMount() {
    // MOUNTING
    this.interval = setInterval(() => {
      this.setState(s => ({ seconds: s.seconds + 1 }));
    }, 1000);
  }

  componentWillUnmount() {
    // UNMOUNTING
    clearInterval(this.interval);
  }

  render() {
    return <h1>Seconds: {this.state.seconds}</h1>;
  }
}
```

**Explanation**

`componentDidMount` runs after the component is inserted into the DOM. `componentWillUnmount` runs before the component is removed. This is the class-component equivalent of the function component's `useEffect` .

### Real-World Cases

**Case: Data Fetching on Mount**

A component fetches user data when it mounts using `useEffect(() => { fetch(...).then(...) }, [])`. The data is displayed once loaded.

**Case: Subscription Management**

A chat component subscribes to a WebSocket on mount and unsubscribes on unmount, ensuring no leaked connections.

### References

- React — Synchronizing with Effects - https://react.dev/learn/synchronizing-with-effects
- React — Component Lifecycle (Legacy) - https://legacy.reactjs.org/docs/react-component.html
- React — useEffect API Reference - https://react.dev/reference/react/useEffect

---

## 7. React Reconciliation

### Definitions

**Core Definition**

Reconciliation is React's algorithm for comparing two element trees (the previous render and the new render) to determine the minimal set of DOM operations needed to update the UI.

**Technical Definition**

Reconciliation is React's diffing algorithm that compares the previous element tree with the new element tree. React uses a heuristic O(n) algorithm based on two assumptions: (1) two elements of different types produce different trees, and (2) developers can use `key` props to hint which children are stable across renders .

**Beginner-Friendly Explanation**

Reconciliation is like editing a document instead of rewriting it. If you change one word, you don't reprint the whole page—you just erase and rewrite that word. React compares the old UI description with the new one and only changes what's different.

### Purposes

- To minimize expensive DOM operations by only updating what changed.
- To maintain component state across re-renders when appropriate.
- To provide predictable, performant updates for complex UIs.

### Syntax Rules and Structure

**Complete General Syntax (Keys in Lists)**

```jsx
{items.map(item => (
  <ListItem key={item.id} data={item} />
))}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `key={item.id}` | Unique identifier for each list item. |
| `item.id` | Stable, unique value (not array index if order changes). |

**Syntax Rules**

1. **Different types → rebuild**: If the root element type changes (e.g., `<div>` to `<span>`), React tears down the old tree and builds new .
2. **Same type → update attributes**: React keeps the same DOM node and only updates changed attributes.
3. **Keys for lists**: Each list item should have a stable, unique `key` prop to help React match items across renders .
4. **Avoid array indices as keys**: If list order can change, using index can cause state bugs and unnecessary re-renders .

**Constraints and Limitations**

- Reconciliation is a heuristic—it doesn't find the theoretical minimum operations.
- Component state is destroyed if the component's position or type changes.

### Annotated Complete Code Examples

**Example 1: Same Type vs. Different Type**

```jsx
function App({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <UserDashboard /> : <LoginPage />}
    </div>
  );
}
```

**Explanation**

When `isLoggedIn` toggles, React sees `<UserDashboard />` replaced by `<LoginPage />`. Different component types → React destroys the old tree (including all state) and builds the new tree from scratch .

**Example 2: Keys in Lists**

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

**Expected Behavior**

If a todo is added at the beginning, React uses the `key` props to identify which `<li>` elements correspond to which todos. It only inserts the new item, rather than re-rendering every item.

**Why This Occurs**

Without keys, React matches children by position. Adding an item at the beginning would make React think every item changed. With keys, React matches `key="1"` to the same todo regardless of position .

### Real-World Cases

**Case: Reordering a Draggable List**

When users reorder items, stable keys ensure React correctly moves DOM nodes instead of destroying and recreating them, preserving input state and animations.

### References

- React — Reconciliation - https://legacy.reactjs.org/docs/reconciliation.html
- Steve Kinney — Understanding Reconciliation in React 19 - https://stevekinney.com/courses/react-performance/understanding-reconciliation-react-19

---

## References

- React Official Documentation — Render and Commit - https://react.dev/learn/render-and-commit
- React Official Documentation — Reconciliation - https://legacy.reactjs.org/docs/reconciliation.html
- React Official Documentation — State as a Snapshot - https://react.dev/learn/state-as-a-snapshot
- React Official Documentation — How to Upgrade to React 18 - https://react.dev/blog/2022/03/08/react-18-upgrade-guide
- React Official Documentation — Synchronizing with Effects - https://react.dev/learn/synchronizing-with-effects
- React Official Documentation — useEffect API Reference - https://react.dev/reference/react/useEffect
- Steve Kinney — Understanding Reconciliation in React 19 - https://stevekinney.com/courses/react-performance/understanding-reconciliation-react-19
- Epic React by Kent C. Dodds — Intro to Optimize Rendering - https://www.epicreact.dev/workshops/react-performance/intro-to-optimize-rendering
- Software Mind — Optimizing React Re-renders for UI Performance - https://softwaremind.com/blog/by-exploring-how-react-re-renders-teams-can-deliver-better-performing-user-interfaces/
- GitHub — React Lifecycle Events - https://raw.githubusercontent.com/MagicMart/react-handbook/master/content/react-lifecycle-events/index.md