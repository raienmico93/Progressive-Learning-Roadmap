# React Rendering Performance: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React rendering performance is the study and optimisation of how React converts state, props, and context changes into DOM updates—minimising unnecessary re-renders, controlling re-render propagation, and leveraging the Fiber architecture and the React Compiler to keep the UI responsive.

**Technical Definition:** React's rendering performance is governed by the interplay between the **render phase** (where React calls components to produce a new virtual DOM tree) and the **commit phase** (where React applies the minimal set of DOM mutations). A re-render is triggered whenever a component's state, its parent's render, a consumed context value, or (for `React.memo`-wrapped components) its props change. The **Fiber architecture** (introduced in React 16) allows the render phase to be interruptible and prioritised, enabling concurrent features. **Reconciliation** (the diffing algorithm) determines which DOM nodes need updating, using element type and `key` to match old and new trees. **Batching** groups multiple state updates into a single render pass (automatic in React 18+). The **React Compiler** (formerly React Forget) automates memoisation by analysing component code at build time and inserting `useMemo`/`useCallback` equivalents, reducing the manual burden of referential equality management.

**Beginner-Friendly Explanation:** Every time something changes in your app—a button click, a data fetch, a parent re-render—React has to figure out what part of the screen needs to update. If it re-renders too much, the app feels slow. Rendering performance is about making React do the smallest amount of work possible: only re-render what actually changed, avoid creating new objects and functions that trick React into thinking things changed, and let the React Compiler handle the tedious optimisation for you.

### Key Characteristics

- **Re-renders are Recursive by Default:** When a component re-renders, React re-renders all its descendants unless a memoisation boundary (`React.memo`) or a structural optimisation stops the propagation.
- **Render ≠ Commit:** React may call a component's function (render phase) without touching the DOM (commit phase) if the output is identical.
- **Referential Equality Drives Memoisation:** `React.memo`, `useMemo`, and `useCallback` all rely on `Object.is` comparison; new object/function references defeat them.
- **Fiber Enables Interruptibility:** The Fiber architecture breaks rendering into units of work that can be paused, prioritised, and resumed.
- **Keys Drive List Reconciliation:** Stable keys let React match elements across renders; unstable keys (indices, random values) cause remounting and lost state.
- **Automatic Batching:** React 18 batches all state updates (including in promises, timeouts, and native event handlers) into a single render.
- **React Compiler Automates Memoisation:** The compiler inserts memoisation automatically, making manual `useMemo`/`useCallback` increasingly unnecessary.

### Prerequisites

- Solid understanding of React function components, JSX, and the primary Hooks.
- Working knowledge of the render/commit lifecycle and the component tree.
- Familiarity with `useState`, `useEffect`, `useMemo`, and `useCallback`.
- Basic understanding of `React.memo` and referential equality.
- Awareness of React DevTools Profiler and the React Compiler.

### Related Programming Areas

- **Memoisation:** `React.memo`, `useMemo`, `useCallback`, and the React Compiler.
- **State Architecture:** Where state lives and how updates propagate.
- **Reconciliation:** Diffing, keys, and the Fiber tree.
- **Concurrent Rendering:** Transitions, Suspense, and interruptible rendering.
- **Profiling:** React DevTools Profiler, browser performance tools, and flame graphs.

### Core Concepts / Features

1. Re-render Triggers
2. Component Boundaries
3. Referential Equality
4. Reconciliation and the Fiber Architecture
5. Modern Compiler Optimisation (React Compiler)

---

## Core Concept 1: Re-render Triggers

### Definitions

**Core Definition:** Re-render triggers are the events that cause React to call a component's function again and reconcile its output—state changes, parent re-renders, context changes, and (for memoised components) prop changes.

**Technical Definition:** React schedules a re-render of a component when one of four conditions occurs: (1) **State change** — a `setState` call (from `useState` or `useReducer`) updates the component's state; (2) **Parent re-render** — the parent component's function is called, which by default calls all its children recursively; (3) **Context change** — a context provider's `value` changes, causing all consumers to re-render; (4) **Props change** — for a component wrapped in `React.memo`, a shallow comparison of props (`Object.is` per prop) detects a difference. Notably, a component's own state change re-renders that component and all its descendants, but **does not** re-render its parent or siblings. A parent re-render **does** re-render all descendants unless `React.memo` or a structural boundary stops it.

**Beginner-Friendly Explanation:** A re-render is React calling your component function again to see if the output changed. It happens when state changes, when a parent re-renders, when context changes, or when memoised props change. The important thing is that re-renders flow *down* the tree—a parent re-rendering re-renders all its children, but a child re-rendering does not re-render its parent. This is why where you put your state matters so much.

### Purposes

- To understand the precise conditions that cause a component to re-render.
- To diagnose unnecessary re-renders by tracing the trigger.
- To choose where state lives so that updates propagate minimally.
- To decide when `React.memo` is needed to block parent-driven re-renders.
- To avoid the misconception that "React re-renders everything."

### Syntax Rules and Structure

**The Four Re-render Triggers:**

| Trigger | Affects | Propagates Down? |
|---|---|---|
| **State change** (`setState`) | The component itself + descendants | Yes |
| **Parent re-render** | All children (unless memoised) | Yes |
| **Context change** | All consumers of that context | Yes |
| **Props change** (memoised) | The memoised component | Yes |

**General Syntax (Parent-Triggered Re-render):**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <Child />          {/* Re-renders when Parent re-renders */}
      <MemoChild />      {/* Skipped if props are referentially equal */}
    </div>
  );
}

const Child = () => <p>Child</p>;

const MemoChild = React.memo(() => <p>Memo Child</p>);
```

**Component Breakdown:**
- `Parent` owns `count`; clicking the button triggers a state change.
- `Child` re-renders because the parent re-rendered (no memoisation).
- `MemoChild` is wrapped in `React.memo`; since it has no props, it never re-renders after the first render.

**General Syntax (State Change Does Not Re-render Parent):**
```jsx
function Parent() {
  const [parentCount, setParentCount] = useState(0);
  console.log('Parent render');

  return (
    <div>
      <button onClick={() => setParentCount(c => c + 1)}>Parent: {parentCount}</button>
      <Child />
    </div>
  );
}

function Child() {
  const [childCount, setChildCount] = useState(0);
  console.log('Child render');

  return (
    <button onClick={() => setChildCount(c => c + 1)}>Child: {childCount}</button>
  );
}
```

**Component Breakdown:**
- Clicking "Child" logs "Child render" only; the parent does not re-render.
- Clicking "Parent" logs "Parent render" and "Child render"; the child re-renders because the parent did.

**Syntax Rules:**
- A component's own state change re-renders that component and its descendants—never its parent or siblings.
- A parent re-render re-renders all descendants unless `React.memo` or a structural boundary stops it.
- A context value change re-renders all consumers of that context, regardless of where they are in the tree.
- `React.memo` performs a shallow prop comparison; if any prop is referentially different, the component re-renders.
- Moving state down to a leaf component reduces the number of components that re-render when that state changes.
- Lifting state up increases the number of components affected by a state change.

**Constraints and Limitations:**
- React does not skip a parent render just because the output is identical; the parent function is still called.
- State updates are asynchronous; reading state immediately after `setState` returns the old value.
- Context consumers re-render even if the specific part of the value they use has not changed (unless selectors are used).
- `React.memo` does not prevent the component from rendering the first time.

### Annotated Code Examples

**Example 1: Tracing Re-render Triggers**

```jsx
import { useState, memo } from 'react';

const ChildA = memo(function ChildA({ label }) {
  console.log('ChildA render');
  return <p>{label}</p>;
});

function ChildB({ label }) {
  console.log('ChildB render');
  return <p>{label}</p>;
}

export default function Parent() {
  const [count, setCount] = useState(0);
  console.log('Parent render');

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ChildA label="A" />
      <ChildB label="B" />
    </div>
  );
}
```

**Expected Output (console):**
```
Parent render
ChildA render
ChildB render
--- click ---
Parent render
ChildB render
```

**Why This Output Occurs:** On the first render, all three components render. Clicking the button changes `count`, causing `Parent` to re-render. `ChildA` is wrapped in `React.memo` and receives the same string prop `"A"`, so its props are referentially equal (`"A" === "A"`) and it is skipped. `ChildB` is not memoised, so it re-renders with the parent.

**Example 2: State Change Does Not Re-render Parent**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  console.log('Counter render');
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}

export default function App() {
  console.log('App render');
  return <Counter />;
}
```

**Expected Output (console):**
```
App render
Counter render
--- click ---
Counter render
```

**Why This Output Occurs:** Clicking the button changes `Counter`'s state, causing `Counter` to re-render. `App` does not re-render because its state and props did not change—only the child's state changed, and state updates do not propagate upward.

### Real-World Cases

- **Form inputs:** A controlled input's state change should not re-render the entire page.
- **Modals:** Opening/closing a modal should not re-render the page behind it.
- **Search boxes:** Typing in a search box re-renders the input and results, but ideally not the header or sidebar.
- **Tabs:** Switching tabs re-renders the active tab's content and the tab list, not the entire layout.
- **Data tables:** Sorting or filtering re-renders the table, not the surrounding dashboard.

### References

- React Official Documentation – Render and Commit: https://react.dev/learn/render-and-commit
- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – State as a Snapshot: https://react.dev/learn/state-as-a-snapshot
- React Official Documentation – Queueing a Series of State Updates: https://react.dev/learn/queueing-a-series-of-state-updates
- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context

---

## Core Concept 2: Component Boundaries

### Definitions

**Core Definition:** Component boundaries are deliberate structural and memoisation points that isolate volatile state changes to small components, protecting heavy ancestor or sibling components from unnecessary re-renders.

**Technical Definition:** A component boundary is a point in the component tree where re-render propagation is intentionally stopped or isolated. This is achieved through: (1) **State colocation** — moving state down to the smallest component that needs it, so updates do not bubble up; (2) **`React.memo`** — wrapping components that should not re-render when their parent re-renders with the same props; (3) **`children` as props** — passing children from a parent that does not re-render, so the children's identity is preserved and the heavy component does not re-render; (4) **Component splitting** — extracting the volatile part of a component into a separate, small component so that only it re-renders. The key insight is that a parent re-rendering with the same `children` prop does not re-render those children if the parent's render output preserves the same element identity (which is not automatic in React, but is achieved by hoisting the element or using `children` as a prop).

**Beginner-Friendly Explanation:** Imagine a large, expensive component at the top of your page—a chart, a map, a heavy table. If a small piece of state near it changes (like a toggle for a sidebar), you do not want the whole chart to re-render. Component boundaries are the walls you build to keep the volatile state isolated. You can either move the state down into a tiny component, wrap the heavy component in `React.memo`, or pass the heavy component as `children` so its element identity does not change.

### Purposes

- To isolate volatile state changes to the smallest possible components.
- To protect heavy components (charts, maps, tables) from re-rendering due to unrelated state changes.
- To reduce the total number of component renders per state update.
- To improve perceived performance by keeping the UI responsive.
- To make the re-render graph explicit and predictable.

### Syntax Rules and Structure

**Pattern 1: State Colocation**
```jsx
// ❌ Bad: state in the parent re-renders the heavy child
function Page() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setIsOpen(o => !o)}>Toggle</button>
      {isOpen && <p>Panel</p>}
      <HeavyChart />  {/* Re-renders on every toggle */}
    </div>
  );
}

// ✅ Good: state colocated in a small component
function Page() {
  return (
    <div>
      <TogglePanel />   {/* Only this re-renders on toggle */}
      <HeavyChart />    {/* Never re-renders */}
    </div>
  );
}

function TogglePanel() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setIsOpen(o => !o)}>Toggle</button>
      {isOpen && <p>Panel</p>}
    </div>
  );
}
```

**Component Breakdown:**
- Bad version: `isOpen` lives in `Page`; toggling re-renders `Page` and all its children, including `HeavyChart`.
- Good version: `isOpen` lives in `TogglePanel`; toggling re-renders only `TogglePanel`. `HeavyChart` is in a different branch and is never re-rendered.

**Pattern 2: `children` as Props**
```jsx
// ❌ Bad: ScrollTracker re-renders HeavyChart on every scroll
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const onScroll = () => setScrollY(window.scrollY);
    window.addEventListener('scroll', onScroll);
    return () => window.removeEventListener('scroll', onScroll);
  }, []);

  return (
    <div>
      <p>Scroll: {scrollY}</p>
      <HeavyChart />
    </div>
  );
}

// ✅ Good: children preserve element identity
function ScrollTracker({ children }) {
  const [scrollY, setScrollY] = useState(0);
  useEffect(() => {
    const onScroll = () => setScrollY(window.scrollY);
    window.addEventListener('scroll', onScroll);
    return () => window.removeEventListener('scroll', onScroll);
  }, []);

  return (
    <div>
      <p>Scroll: {scrollY}</p>
      {children}
    </div>
  );
}

function Page() {
  return (
    <ScrollTracker>
      <HeavyChart />   {/* Element identity preserved; no re-render */}
    </ScrollTracker>
  );
}
```

**Component Breakdown:**
- Bad version: `HeavyChart` is rendered inside `ScrollTracker`, so it re-renders on every scroll.
- Good version: `HeavyChart` is passed as `children` from `Page`. `Page` does not re-render on scroll, so the `children` element is the same reference, and React skips re-rendering it.

**Pattern 3: `React.memo` Boundary**
```jsx
const HeavyChart = React.memo(function HeavyChart({ data }) {
  return <div>{/* expensive chart */}</div>;
});

function Dashboard() {
  const [isOpen, setIsOpen] = useState(false);
  const data = useMemo(() => computeData(), []);

  return (
    <div>
      <button onClick={() => setIsOpen(o => !o)}>Toggle</button>
      <HeavyChart data={data} />   {/* Skipped when isOpen changes */}
    </div>
  );
}
```

**Component Breakdown:**
- `HeavyChart` is memoised; it re-renders only when `data` changes.
- `data` is memoised with `useMemo`, so its reference is stable across toggles.
- Toggling `isOpen` re-renders `Dashboard`, but `HeavyChart` is skipped because `data` is referentially equal.

**Syntax Rules:**
- Move state down to the smallest component that needs it (state colocation).
- Pass heavy components as `children` when a parent must re-render but the heavy component should not.
- Wrap heavy components in `React.memo` and stabilise their props.
- Memoise props (objects, arrays, functions) with `useMemo`/`useCallback` before passing them to memoised children.
- Split components so that volatile state lives in small, focused components.
- Avoid passing inline object/array/function literals to memoised children.

**Constraints and Limitations:**
- `React.memo` only skips re-renders if all props are shallowly equal; unstable props defeat it.
- Passing `children` as a prop preserves element identity only if the parent that renders `children` does not re-render.
- Over-splitting components can make the tree harder to understand and increase the number of components.
- `React.memo` adds a comparison cost per render; for trivial components, it can be a net loss.
- Context providers that change often can still re-render all consumers, regardless of boundaries.

### Annotated Code Example: Protecting a Heavy Component

```jsx
import { useState, memo, useMemo } from 'react';

const HeavyTable = memo(function HeavyTable({ rows }) {
  console.log('HeavyTable render');
  return (
    <table>
      <tbody>
        {rows.map(r => <tr key={r.id}><td>{r.name}</td></tr>)}
      </tbody>
    </table>
  );
});

export default function Dashboard() {
  const [filter, setFilter] = useState('');
  const rows = useMemo(
    () => Array.from({ length: 1000 }, (_, i) => ({ id: i, name: `Row ${i}` })),
    []
  );

  console.log('Dashboard render');

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      <HeavyTable rows={rows} />
    </div>
  );
}
```

**Expected Output (console):**
```
Dashboard render
HeavyTable render
--- typing ---
Dashboard render
Dashboard render
Dashboard render
```

**Why This Output Occurs:** `rows` is memoised with `useMemo` and has a stable reference. `HeavyTable` is wrapped in `React.memo`, so its shallow prop comparison finds `rows` unchanged and skips its render. Only `Dashboard` re-renders on each keystroke.

### Real-World Cases

- **Dashboards:** A filter input re-renders the filter component, not the heavy data grid.
- **Maps:** A sidebar toggle re-renders the sidebar, not the map.
- **Editors:** Typing in a rich text editor re-renders the editor, not the preview pane (or vice versa).
- **Layouts:** A theme toggle re-renders a theme-aware button, not the entire page.
- **Modals:** Opening a modal re-renders the modal, not the background page.

### References

- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React Official Documentation – Preserving and Resetting State: https://react.dev/learn/preserving-and-resetting-state
- React Official Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure
- Kent C. Dodds – One React Mistake That's Keeping Your App Slow: https://kentcdodds.com/blog/optimize-react-re-renders

---

## Core Concept 3: Referential Equality

### Definitions

**Core Definition:** Referential equality is the comparison of two values by their memory reference (`Object.is`), which determines whether React considers a prop, state value, or dependency to have changed; new object/array/function references across renders defeat memoisation.

**Technical Definition:** JavaScript objects, arrays, and functions are compared by reference, not by content. `{ a: 1 } === { a: 1 }` is `false` because they are different objects in memory. React uses `Object.is` to compare previous and next values in `React.memo`, `useMemo`, `useCallback`, and dependency arrays. This means that creating a new object, array, or function during render—even if its content is identical—causes React to treat it as a change, triggering re-renders or recomputations. Referential equality is the root cause of most "unnecessary re-render" bugs: a parent passes `style={{ color: 'red' }}` to a memoised child, and the child re-renders every time because the style object is new every render.

**Beginner-Friendly Explanation:** Imagine you have two identical-looking boxes. Even though the contents are the same, they are different boxes. React compares boxes by identity, not by what is inside. If you create a new box every time, React thinks something changed—even if it did not. The fix is to either reuse the same box (`useMemo`, `useCallback`) or compare by content (custom `React.memo` comparator, though this is discouraged).

### Purposes

- To understand why memoised components still re-render.
- To identify which props, state values, or dependencies are defeating memoisation.
- To apply `useMemo` and `useCallback` to stabilise references.
- To avoid creating new objects, arrays, and functions during render when they are passed to memoised children or used as dependencies.
- To reason about `React.memo`'s shallow comparison.

### Syntax Rules and Structure

**Referential Equality Examples:**
```javascript
// Primitives: value equality
"a" === "a";                    // true
1 === 1;                        // true
true === true;                  // true

// Objects: referential equality
{ a: 1 } === { a: 1 };          // false
const obj = { a: 1 };
obj === obj;                    // true

// Arrays: referential equality
[1, 2] === [1, 2];              // false

// Functions: referential equality
(() => {}) === (() => {});      // false
```

**Common Referential Equality Bugs:**
```jsx
// ❌ New style object every render
<MemoChild style={{ color: 'red' }} />

// ✅ Stable style object
const style = useMemo(() => ({ color: 'red' }), []);
<MemoChild style={style} />

// ❌ New callback every render
<MemoChild onClick={() => doSomething(id)} />

// ✅ Stable callback
const handleClick = useCallback(() => doSomething(id), [id]);
<MemoChild onClick={handleClick} />

// ❌ New array every render
<MemoList items={items.filter(i => i.active)} />

// ✅ Memoised array
const activeItems = useMemo(() => items.filter(i => i.active), [items]);
<MemoList items={activeItems} />
```

**Component Breakdown:**
- Each "bad" example creates a new reference on every render, defeating `React.memo`.
- Each "good" example uses `useMemo`/`useCallback` to stabilise the reference.
- The dependency array controls when the memoised value is recreated.

**Custom `React.memo` Comparator (Discouraged):**
```jsx
const MemoChild = React.memo(
  function Child({ config }) { /* ... */ },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.config.theme === nextProps.config.theme;
  }
);
```

**Component Breakdown:**
- The second argument is a custom comparator.
- Returning `true` skips the re-render.
- This is discouraged because it is easy to get wrong and can hide bugs.

**Syntax Rules:**
- Memoise objects, arrays, and functions passed to memoised children with `useMemo`/`useCallback`.
- Memoise values used as dependencies of `useEffect`, `useMemo`, and `useCallback`.
- Use `useCallback` for event handlers passed to memoised children.
- Use `useMemo` for computed values (filtered lists, config objects, style objects) passed to memoised children.
- Move constant objects and functions outside the component when they do not depend on props/state.
- Avoid custom `React.memo` comparators; prefer stabilising references instead.

**Constraints and Limitations:**
- `useMemo` and `useCallback` are performance optimisations, not semantic guarantees; React may discard memoised values.
- Memoising values that are not passed to memoised children or used as dependencies is wasted work.
- `useMemo` with an unstable dependency is useless; the dependency must also be stable.
- Deeply nested objects require deep memoisation or a state management library.
- The React Compiler automates much of this, reducing the need for manual memoisation.

### Annotated Code Example: Fixing a Referential Equality Bug

```jsx
import { useState, memo, useMemo, useCallback } from 'react';

const MemoList = memo(function MemoList({ items, onSelect }) {
  console.log('MemoList render');
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <button onClick={() => onSelect(item)}>{item.name}</button>
        </li>
      ))}
    </ul>
  );
});

export default function App() {
  const [query, setQuery] = useState('');
  const [items] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' },
  ]);

  // ✅ Memoised filtered list
  const filtered = useMemo(
    () => items.filter(i => i.name.toLowerCase().includes(query.toLowerCase())),
    [items, query]
  );

  // ✅ Stable callback
  const handleSelect = useCallback((item) => {
    console.log('Selected', item.name);
  }, []);

  console.log('App render');

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <MemoList items={filtered} onSelect={handleSelect} />
    </div>
  );
}
```

**Expected Output (console):**
```
App render
MemoList render
--- typing "ap" ---
App render
MemoList render   (because `filtered` changed)
--- typing "app" ---
App render
App render
App render
--- deleting "p" ---
App render
MemoList render   (because `filtered` changed)
```

**Why This Output Occurs:** `filtered` changes only when the filtered result actually changes (e.g., when "ap" becomes "app", the filter still matches Apple, so `filtered` is a new array—wait, `useMemo` recreates it when `query` changes, so `filtered` is new). The key insight: `useMemo` recreates `filtered` whenever `query` changes, so `MemoList` re-renders whenever the query changes. To truly skip renders, you would need to memoise based on the *content* of the filtered list, which is not what `useMemo` does. However, `handleSelect` is stable, so it does not contribute to re-renders. The main benefit is that `MemoList` re-renders only when `filtered` changes, not on every unrelated state change.

### Real-World Cases

- **Lists with row actions:** Stable delete/edit callbacks for memoised rows.
- **Config objects:** Memoised chart/map configuration to prevent re-renders.
- **Filtered data:** Memoised filtered/sorted arrays passed to memoised tables.
- **Event handlers:** Stable handlers passed to deeply nested components.
- **Style objects:** Memoised inline styles for memoised components.

### References

- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- MDN Web Docs – `Object.is()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- Kent C. Dodds – When to useMemo and useCallback: https://kentcdodds.com/blog/usememo-and-usecallback

---

## Core Concept 4: Reconciliation and the Fiber Architecture

### Definitions

**Core Definition:** Reconciliation is React's algorithm for comparing the previous virtual DOM tree with the new one and determining the minimal set of DOM mutations; the Fiber architecture is React's internal representation of the component tree that makes this process interruptible, prioritised, and resumable.

**Technical Definition:** The **Fiber** architecture (React 16+) represents each component instance as a "fiber node" containing its type, props, state, and pointers to its parent, child, and sibling. The fiber tree is a linked list that React traverses during the render phase, producing a new "work-in-progress" tree. **Reconciliation** (also called "diffing") compares the old and new trees: if the element type is different, React unmounts the old subtree and mounts a new one; if the type is the same, React updates the existing DOM node and recurses into children. **Keys** are used to match children in lists: a stable key lets React move, reuse, or remove elements without remounting. **Batching** groups multiple state updates into a single render pass (automatic in React 18+). The Fiber architecture's key advantage is **interruptibility**: the render phase can be paused, prioritised (transitions, Suspense), and resumed, enabling concurrent rendering.

**Beginner-Friendly Explanation:** Imagine React as a librarian reorganising a bookshelf. Reconciliation is the librarian comparing the old shelf to the new one and only moving the books that changed. Keys are the labels on the books—without them, the librarian has to guess which book is which and might rebuild the whole shelf. The Fiber architecture is the librarian's ability to pause, answer a phone call, and resume exactly where they left off. This is what makes React's concurrent features possible.

### Purposes

- To understand how React decides which DOM nodes to update.
- To use keys correctly so React can match elements across renders.
- To understand why unstable keys cause remounting and lost state.
- To understand how Fiber enables interruptible, prioritised rendering.
- To reason about batching and the number of render passes.

### Syntax Rules and Structure

**Diffing Algorithm (Simplified):**
```
1. If the element type is different (e.g., <div> → <span>):
   - Unmount the old subtree.
   - Mount a new subtree.
2. If the element type is the same:
   - Update the existing DOM node (props, attributes).
   - Recurse into children.
3. For lists of children:
   - Use `key` to match old and new elements.
   - Without keys, match by index (fragile).
```

**Keys: Stable vs. Unstable:**
```jsx
// ❌ Bad: index as key — remounts on reorder/insert/delete
{items.map((item, index) => <Row key={index} item={item} />)}

// ❌ Bad: random key — remounts every render
{items.map(item => <Row key={Math.random()} item={item} />)}

// ✅ Good: stable, unique key from data
{items.map(item => <Row key={item.id} item={item} />)}
```

**Component Breakdown:**
- Index keys: when items are reordered, React matches by position, not identity, causing state loss and unnecessary remounts.
- Random keys: every render produces new keys, so React remounts every item.
- Stable IDs: React matches the same item across renders, preserving state and DOM nodes.

**Demonstrating Key Problems:**
```jsx
function List({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          <input placeholder={item.name} />
        </li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- If the user types into an input and then the list is reordered, index keys cause the typed value to stay with the *position*, not the *item*—a classic bug.
- Using `item.id` as the key preserves the input's state with the item.

**Automatic Batching (React 18+):**
```jsx
function App() {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);

  function handleClick() {
    setA(a + 1);
    setB(b + 1);
    // React batches both updates into a single render
  }

  return <button onClick={handleClick}>{a} {b}</button>;
}
```

**Component Breakdown:**
- Both `setA` and `setB` are batched into one render, even though they are called sequentially.
- In React 18, batching also applies to updates inside promises, `setTimeout`, and native event handlers.

**Syntax Rules:**
- Always use stable, unique keys from data; never use indices or random values.
- Keys must be unique among siblings, not globally.
- Keys should be stable across renders (do not regenerate them).
- Do not use keys to force remounting unless you intend to reset state (e.g., `<Component key={userId} />`).
- Understand that React batches state updates automatically in React 18+.
- Use `flushSync` only when you need to force a synchronous render (rare).

**Constraints and Limitations:**
- React cannot know which item is which without keys; it guesses by position.
- Keys are not passed to components as props; they are internal to React.
- The diffing algorithm is heuristic, not optimal; it runs in O(n) but may produce suboptimal results in edge cases.
- Fiber's interruptibility is a React 18+ feature; React 17 and earlier render synchronously.
- Batching can be surprising if you rely on reading state immediately after `setState`.

### Annotated Code Example: Key-Induced State Loss

```jsx
import { useState } from 'react';

export default function ReorderableList() {
  const [items, setItems] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' },
  ]);

  function reverse() {
    setItems(prev => [...prev].reverse());
  }

  return (
    <div>
      <button onClick={reverse}>Reverse</button>
      <ul>
        {items.map((item, index) => (
          <li key={index}>
            <input placeholder={item.name} />
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** Typing into the input for "Apple", then clicking "Reverse", causes the typed value to appear on "Cherry" (the new first item). The input state does not follow the item—it stays with the position.

**Why This Output Occurs:** React uses the key (index) to match elements. After reversing, position 0 has a new item ("Cherry"), but React sees key `0` and reuses the same input, preserving its state. The typed value stays with position 0, not with "Apple". Using `key={item.id}` would preserve the input state with the item.

**Fixed Version:**
```jsx
{items.map(item => (
  <li key={item.id}>
    <input placeholder={item.name} />
  </li>
))}
```

**Why the Fix Works:** React matches by `item.id`, so when the list reverses, the input for "Apple" moves with "Apple" and preserves its state.

### Real-World Cases

- **Todo lists:** Stable keys preserve checkbox state across reorders.
- **Sortable tables:** Stable keys preserve row state and DOM nodes when sorting.
- **Chat messages:** Stable message IDs preserve scroll position and input state.
- **Drag-and-drop lists:** Stable keys are essential for correct reordering.
- **Infinite scroll:** Stable keys prevent remounting when new pages are appended.

### References

- React Official Documentation – Reconciliation: https://legacy.reactjs.org/docs/reconciliation.html
- React Official Documentation – Rendering Lists: https://react.dev/learn/rendering-lists
- React Official Documentation – Preserving and Resetting State: https://react.dev/learn/preserving-and-resetting-state
- React Official Documentation – React Fiber Architecture: https://github.com/acdlite/react-fiber-architecture
- React Official Documentation – Automatic Batching: https://react.dev/blog/2022/03/29/react-v18#automatically-batch-updates
- Andrew Clark – React Fiber Architecture: https://github.com/acdlite/react-fiber-architecture

---

## Core Concept 5: Modern Compiler Optimisation (React Compiler)

### Definitions

**Core Definition:** The React Compiler (formerly React Forget) is a build-time tool that automatically memoises components and hooks by analysing their code, inserting `useMemo`/`useCallback`-equivalent optimisations without requiring manual intervention.

**Technical Definition:** The React Compiler is a Babel plugin (and increasingly a Rust-based compiler) that runs at build time and analyses React components and hooks. It applies memoisation automatically: each component's output is cached based on its props and state, and each value or function inside the component is cached based on the values it reads. The compiler relies on the **Rules of React** (components must be pure, props and state are immutable, hooks follow the Rules of Hooks) and the **Rules of Hooks**. When code violates these rules, the compiler skips optimisation for that component. The compiler is designed to be incremental: it can be adopted in one file at a time, and it coexists with existing `useMemo`/`useCallback` (which are preserved and often made redundant). In React 19, the compiler is stable and can be enabled via `babel-plugin-react-compiler`.

**Beginner-Friendly Explanation:** The React Compiler is like a smart assistant that looks at your components and automatically adds the memoisation you would have written by hand. If you have a component that filters a list, the compiler caches the filtered list. If you pass a callback to a child, the compiler caches the callback. You stop worrying about `useMemo` and `useCallback`—the compiler handles it. The catch: your code must follow the Rules of React (pure components, no mutation) or the compiler skips it.

### Purposes

- To automate memoisation and eliminate manual `useMemo`/`useCallback`.
- To reduce the cognitive burden of referential equality management.
- To improve rendering performance by default, without code changes.
- To make performance optimisation the default rather than an afterthought.
- To prepare codebases for React 19 and beyond.

### Syntax Rules and Structure

**Installation (Vite):**
```bash
npm install -D babel-plugin-react-compiler@latest
```

```javascript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: [['babel-plugin-react-compiler', {}]],
      },
    }),
  ],
});
```

**Installation (Next.js):**
```javascript
// next.config.js
module.exports = {
  experimental: {
    reactCompiler: true,
  },
};
```

**Before the Compiler (Manual Memoisation):**
```jsx
function ProductList({ products, query }) {
  const filtered = useMemo(
    () => products.filter(p => p.name.includes(query)),
    [products, query]
  );

  const handleSelect = useCallback((product) => {
    console.log(product);
  }, []);

  return (
    <ul>
      {filtered.map(p => (
        <ProductRow key={p.id} product={p} onSelect={handleSelect} />
      ))}
    </ul>
  );
}
```

**After the Compiler (Automatic Memoisation):**
```jsx
function ProductList({ products, query }) {
  const filtered = products.filter(p => p.name.includes(query));

  const handleSelect = (product) => {
    console.log(product);
  };

  return (
    <ul>
      {filtered.map(p => (
        <ProductRow key={p.id} product={p} onSelect={handleSelect} />
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- The second version has no `useMemo` or `useCallback`.
- The compiler inserts memoisation automatically: `filtered` is cached based on `products` and `query`; `handleSelect` is cached.
- The code is simpler and equally performant.

**What the Compiler Optimises:**
- **Component output:** The component re-renders only when its props or state change.
- **Expensive computations:** Values like `filtered` are memoised.
- **Function identities:** Callbacks passed to children are memoised.
- **JSX elements:** Elements created inside the component are memoised.

**Syntax Rules:**
- Enable the compiler via the Babel plugin or framework integration.
- Follow the Rules of React: components must be pure, props and state must be immutable, and Hooks must follow the Rules of Hooks.
- Do not mutate props, state, or values derived from them; the compiler assumes immutability.
- You can keep existing `useMemo`/`useCallback`; the compiler coexists with them.
- Adopt incrementally: the compiler optimises compliant components and skips non-compliant ones.
- Use the `eslint-plugin-react-compiler` to catch violations early.

**Constraints and Limitations:**
- The compiler requires code that follows the Rules of React; violating code is skipped (not broken).
- The compiler does not optimise components that read mutable external state (e.g., global variables).
- The compiler may not eliminate all re-renders; `React.memo`-like behaviour is applied to component output, but some patterns still re-render.
- The compiler is a build-time tool; it does not help if the bottleneck is network or DOM, not rendering.
- Existing `useMemo`/`useCallback` may become redundant, but removing them is optional.

### Annotated Code Example: Compiler-Optimised Component

```jsx
// Before compiler (manual optimisation)
function SearchPage({ items }) {
  const [query, setQuery] = useState('');
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(query)),
    [items, query]
  );
  const handleSelect = useCallback((item) => console.log(item), []);

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <List items={filtered} onSelect={handleSelect} />
    </div>
  );
}

// After compiler (same code, no manual memoisation)
function SearchPage({ items }) {
  const [query, setQuery] = useState('');
  const filtered = items.filter(i => i.name.includes(query));
  const handleSelect = (item) => console.log(item);

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <List items={filtered} onSelect={handleSelect} />
    </div>
  );
}
```

**Expected Output:** Both versions behave identically. The compiler inserts memoisation for `filtered` and `handleSelect`, so `List` re-renders only when `filtered` or `handleSelect` changes. The second version is simpler and equally performant.

**Why This Output Occurs:** The compiler analyses the component, identifies that `filtered` depends on `items` and `query`, and caches it. It identifies that `handleSelect` has no dependencies and caches it. The memoisation is inserted automatically, so the developer does not need to write `useMemo`/`useCallback`.

### Real-World Cases

- **Large codebases:** Adopting the compiler incrementally to reduce memoisation boilerplate.
- **New projects:** Writing simpler code without manual `useMemo`/`useCallback`.
- **Design systems:** Components benefit from automatic memoisation without manual tuning.
- **Performance-critical apps:** The compiler reduces the risk of missed memoisation.
- **React 19 migration:** The compiler is the recommended path for new performance work.

### References

- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – React Compiler Introduction: https://react.dev/learn/react-compiler/introduction
- React Official Documentation – React Compiler Installation: https://react.dev/learn/react-compiler/installation
- React Official Documentation – Rules of React: https://react.dev/reference/rules
- React Official Documentation – eslint-plugin-react-compiler: https://www.npmjs.com/package/eslint-plugin-react-compiler
- babel-plugin-react-compiler: https://www.npmjs.com/package/babel-plugin-react-compiler

---

## Comparison and Decision Guidance

| Technique | When to Use | Cost | Benefit |
|---|---|---|---|
| **State colocation** | Volatile state affects a small subtree | Low (code organisation) | High (fewer re-renders) |
| **`children` as props** | Parent must re-render, but a heavy child should not | Low | High |
| **`React.memo`** | Component re-renders with the same props | Low (comparison per render) | High for expensive components |
| **`useMemo`** | Expensive computation or stable prop | Low (memory + comparison) | High when passed to memoised children |
| **`useCallback`** | Callback passed to memoised child or used as dependency | Low | High for memoised children |
| **Stable keys** | Lists that reorder, insert, or delete | None | Prevents remounting and state loss |
| **React Compiler** | Any codebase following Rules of React | Build-time only | Automatic memoisation |

**Decision Guidance:**
- **Start with state colocation** — move volatile state down to the smallest component.
- **Use `children` as props** when a parent must re-render but a heavy child should not.
- **Use `React.memo`** for expensive components that re-render with the same props.
- **Stabilise props** with `useMemo`/`useCallback` before passing them to memoised children.
- **Always use stable keys** from data; never use indices or random values.
- **Adopt the React Compiler** for automatic memoisation; keep manual memoisation only where the compiler cannot help.
- **Profile before and after** any optimisation; measure the impact.

---

## References

- React Official Documentation – Render and Commit: https://react.dev/learn/render-and-commit
- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – Rendering Lists: https://react.dev/learn/rendering-lists
- React Official Documentation – Preserving and Resetting State: https://react.dev/learn/preserving-and-resetting-state
- React Official Documentation – Reconciliation (Legacy): https://legacy.reactjs.org/docs/reconciliation.html
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – React Compiler Introduction: https://react.dev/learn/react-compiler/introduction
- React Official Documentation – React Compiler Installation: https://react.dev/learn/react-compiler/installation
- React Official Documentation – Rules of React: https://react.dev/reference/rules
- React Official Documentation – Automatic Batching (React 18): https://react.dev/blog/2022/03/29/react-v18#automatically-batch-updates
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React Official Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure
- Andrew Clark – React Fiber Architecture: https://github.com/acdlite/react-fiber-architecture
- MDN Web Docs – `Object.is()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- Kent C. Dodds – When to useMemo and useCallback: https://kentcdodds.com/blog/usememo-and-usecallback
- Kent C. Dodds – One React Mistake That's Keeping Your App Slow: https://kentcdodds.com/blog/optimize-react-re-renders
- babel-plugin-react-compiler: https://www.npmjs.com/package/babel-plugin-react-compiler
- eslint-plugin-react-compiler: https://www.npmjs.com/package/eslint-plugin-react-compiler