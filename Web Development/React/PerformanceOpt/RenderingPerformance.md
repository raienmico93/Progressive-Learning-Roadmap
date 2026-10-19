# React Rendering Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React rendering performance is the efficiency with which React executes component functions, reconciles the resulting element tree, and commits changes to the DOM — and the practice of reducing unnecessary work in that pipeline.

**Technical Definition:** React's rendering model is built on comparing values to decide when components need to re-render. When a component renders, React executes the component function again, generates a new JSX output, compares it with the previous output (reconciliation), and updates only the changed DOM nodes. The critical distinction is that **re-render ≠ DOM update**: a re-render means the component function is re-executed, and every calculation inside it runs again. Rendering performance therefore concerns the cost of component execution, reconciliation, and commit — not just DOM mutations. React's default behaviour is to re-render a component and all its children whenever state changes, unless manual memoization or architectural patterns prevent it.

**Beginner-Friendly Explanation:** React re-renders a component by running its function again. Even if the screen does not change, all the code inside that function runs — sorting, filtering, building JSX, and so on. If a component re-renders unnecessarily, that work is wasted. Rendering performance is about stopping that waste: only re-running the components whose output actually needs to change, and only doing the expensive calculations when their inputs really change.

### Key Characteristics

- **Re-render is function execution:** Every re-render re-runs the component body, including all calculations and JSX construction.
- **Blast radius:** A state change at a high level re-renders the component and all its children, even those that do not use the changed state.
- **Reference equality matters:** React compares props and dependencies with `Object.is`; new object, array, or function references on every render defeat memoization.
- **State placement is the primary lever:** Moving state closer to its consumers prevents render storms more effectively than sprinkling memoization everywhere.
- **Memoization is surgical:** `React.memo`, `useMemo`, and `useCallback` are targeted tools, not blanket solutions.

### Prerequisites

- React function components, state, props, and JSX.
- Understanding of the render → reconcile → commit pipeline.
- Familiarity with reference equality and `Object.is`.
- Basic knowledge of list rendering and keys.

### Related Programming Areas

- **Reconciliation** — the algorithm that compares element trees.
- **Memoization** — `React.memo`, `useMemo`, `useCallback`.
- **Virtualization** — rendering only visible list items.
- **State management** — colocation, lifting, and context.

---

## 1. Unnecessary Re-renders

### Definitions

**Core Definition:** An unnecessary re-render is a component execution that produces the same output as the previous render, wasting CPU time without any visible change.

**Technical Definition:** An unnecessary re-render occurs when React re-executes a component function even though the component's output would be identical — because its props and state have not meaningfully changed. The most common causes are state lifted too high (causing the entire subtree to re-render), unstable prop references (new objects or functions on every render), and context value changes that re-render all consumers. These re-renders drive up CPU usage, increase garbage collection overhead, and degrade the user experience.

**Beginner-Friendly Explanation:** An unnecessary re-render is React doing work for nothing. The component runs again, but the screen looks exactly the same. A few of these are harmless; thousands of them make the app feel slow.

### Purposes

- (Unnecessary re-renders are a problem to eliminate, not a feature.)
- To understand where wasted rendering work comes from.
- To prioritise architectural fixes over premature memoization.

### Syntax Rules and Structure

**Common Sources of Unnecessary Re-renders:**

| Source | Mechanism |
|--------|-----------|
| State lifted too high | Every state change re-renders the entire subtree |
| Unstable object props | New object reference on every render defeats `React.memo` |
| Inline functions | New function reference on every render |
| Context value churn | New context value object re-renders all consumers |
| Redundant state | Storing derived values triggers extra renders |
| Controlled inputs | Every keystroke re-renders the component and its children |

**Syntax Rules:**

1. Re-rendering is not inherently bad; unnecessary re-rendering is the target.
2. Measure with the React DevTools Profiler before optimising.
3. Prefer architectural fixes (state colocation, component splitting) before memoization.

### Annotated Code Examples

```jsx
// ❌ State lifted too high: HeavyComponent re-renders on every count change
function Page() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <HeavyComponent /> {/* Does not use count, but re-renders anyway */}
    </>
  );
}
```

**Expected Output:** `HeavyComponent` re-executes on every button click, even though its output does not change.

**Why this output:** React's default behaviour is to re-render all children when a parent re-renders. `HeavyComponent` is caught in the blast radius.

### Real-World Cases

- **Dashboards:** A counter in the header re-rendering the entire chart grid.
- **Forms:** Every keystroke re-rendering a sibling data table.
- **Lists:** The parent re-rendering all list items when only one item changes.

### References

- React Performance and Preventing Unnecessary Re-renders — https://aspnetzero.com/blog/react-performance-and-preventing-unnecessary-renders
- Remediating Superfluous Re-Rendering in React Applications (ICSE 2026) — https://dl.acm.org/doi/10.1145/3744916.3773235
- RT Camp — Performance Optimization for React-based projects — https://rtcamp.com/handbook/react-best-practices/performance-optimization/

---

## 2. Re-render Causes

### Definitions

**Core Definition:** Re-render causes are the specific events that trigger React to execute a component function again.

**Technical Definition:** A component re-renders when: (1) its own state changes via a state setter; (2) the props passed to it change; (3) its parent component re-renders (by default, all children re-render); or (4) a context value it consumes changes. React does not perform deep analysis to determine whether a child's output would actually differ; it relies on a deterministic model where parent renders propagate to children.

**Beginner-Friendly Explanation:** A component re-renders when its own data changes, when its parent re-renders, or when it receives new props. The parent re-render trigger is the one that most often causes unnecessary work.

### Purposes

- To understand the triggers so they can be controlled.
- To identify which trigger is causing a performance problem.
- To choose the right optimisation for the specific trigger.

### Syntax Rules and Structure

**Re-render Triggers:**

| Trigger | Description |
|---------|-------------|
| State change | A setter from `useState` or `useReducer` is called. |
| Prop change | The parent passes different prop values. |
| Parent render | The parent re-renders, causing children to re-render by default. |
| Context change | A consumed context value changes. |

**Syntax Rules:**

1. State changes are local to the component that owns the state.
2. Parent renders propagate to all children unless blocked by `React.memo` or architectural patterns.
3. Context changes propagate to all consumers of that context.

### Annotated Code Examples

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <Child /> {/* Re-renders because Parent re-renders */}
    </div>
  );
}

function Child() {
  console.log('Child rendered');
  return <p>Child</p>;
}
```

**Expected Output:** `"Child rendered"` logs on every `Parent` re-render.

**Why this output:** The parent's state change triggers a parent re-render, which by default re-renders all children.

### Real-World Cases

- **Debugging:** Identifying which trigger causes a re-render storm.
- **Architecture:** Placing state to avoid parent-render propagation.
- **Context design:** Splitting context to avoid consumer-wide re-renders.

### References

- Reducing Re-renders in React Applications (Webkul) — https://webkul.com/blog/reducing-re-renders-in-react-applications/
- React Performance and Preventing Unnecessary Re-renders — https://aspnetzero.com/blog/react-performance-and-preventing-unnecessary-renders
- Open Storefront Framework Developer's Guide — https://docs.oracle.com/en/industries/retail/retail-digital-commerce/latest/dosfa/avoid-unnecessary-rerendering.htm

---

## 3. Re-render Optimization

### Definitions

**Core Definition:** Re-render optimization is the set of techniques used to prevent or reduce unnecessary component executions.

**Technical Definition:** Re-render optimization combines **architectural techniques** (state colocation, component splitting, children as props) and **memoization techniques** (`React.memo`, `useMemo`, `useCallback`) to reduce the number of component functions executed during an update. The architectural techniques address the root cause (state placed too high, components too large); memoization techniques address the symptoms (unstable references, expensive recomputation). The React Compiler (React 19+) automates memoization at build time, reducing the need for manual `useMemo` and `useCallback`.

**Beginner-Friendly Explanation:** Re-render optimization means stopping React from running components that do not need to run. You do this first by putting state in the right place and splitting components, and second by telling React when a component can safely skip re-rendering.

### Purposes

- To reduce wasted CPU work and improve responsiveness.
- To prevent render storms from high-level state changes.
- To stabilise prop references so memoization can work.
- To keep large component trees fast as they grow.

### Syntax Rules and Structure

**Optimization Toolbox:**

| Technique | Type | When to use |
|-----------|------|-------------|
| State colocation | Architectural | State is used by a small subtree |
| Component splitting | Architectural | A large component re-renders often |
| Children as props | Architectural | A parent's state changes frequently but children are static |
| `React.memo` | Memoization | A component re-renders with the same props |
| `useMemo` | Memoization | An expensive calculation repeats |
| `useCallback` | Memoization | A function prop breaks a memoized child |

**Syntax Rules:**

1. Measure first; optimise only proven bottlenecks.
2. Prefer architectural fixes over memoization.
3. Use memoization only when there is a concrete re-render problem.

### Annotated Code Examples

```jsx
// ✅ State colocated: HeavyComponent is outside the state-owning component
function App() {
  return (
    <>
      <Counter /> {/* State lives here */}
      <HeavyComponent /> {/* Never re-renders from counter changes */}
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Expected Output:** Clicking the counter does not re-render `HeavyComponent`.

**Why this output:** State is colocated with its consumer (`Counter`). `HeavyComponent` is a sibling, not a child, so it is outside the re-render blast radius.

### Real-World Cases

- **Large dashboards:** Colocating filter state to avoid re-rendering charts.
- **Forms:** Splitting form sections so keystrokes only re-render the active field.
- **Lists:** Memoizing list items so only the changed item re-renders.

### References

- RT Camp — Performance Optimization for React-based projects — https://rtcamp.com/handbook/react-best-practices/performance-optimization/
- React Performance and Preventing Unnecessary Re-renders — https://aspnetzero.com/blog/react-performance-and-preventing-unnecessary-renders
- React Compiler Introduction — https://zh-hans.react.dev/learn/react-compiler/introduction

---

## 4. `React.memo`

### Definitions

**Core Definition:** `React.memo` is a higher-order component that prevents a function component from re-rendering when its props have not changed.

**Technical Definition:** `React.memo(Component)` returns a memoized version of the component. React renders the component only if its props have changed, performing a shallow comparison of props by default. If the props are shallowly equal to the previous render, React skips rendering and reuses the previous output. `React.memo` is effective only when the component renders the same output for the same props and when prop references are stable.

**Beginner-Friendly Explanation:** `React.memo` tells React: “Only re-render this component if its props actually changed.” If the parent re-renders but the props are the same, React skips the child.

### Purposes

- To skip re-renders of pure components when props are unchanged.
- To isolate expensive components from parent re-render waves.
- To optimize frequently re-rendered list items and charts.

### Syntax Rules and Structure

**General Syntax:**

```jsx
const MemoizedComponent = memo(function Component(props) {
  return <JSX />;
});
```

**Syntax Rules:**

1. `React.memo` performs a shallow comparison of props.
2. It is effective only when prop references are stable.
3. Do not wrap every component; unnecessary use adds overhead.
4. `React.memo` does not prevent re-renders caused by context changes.

**Constraints and Limitations:**

- New object or function props on every render defeat the memoization.
- The comparison is shallow; nested object changes are not detected.
- Memoization is a performance hint, not a guarantee.

### Annotated Code Examples

```jsx
import { memo, useState } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log('ExpensiveList rendered');
  return <ul>{items.map(i => <li key={i}>{i}</li>)}</ul>;
});

function App() {
  const [count, setCount] = useState(0);
  const items = ['apple', 'banana', 'cherry']; // Stable reference

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveList items={items} />
    </div>
  );
}
```

**Expected Output:** `"ExpensiveList rendered"` logs once; clicking the button does not re-render it.

**Why this output:** `items` is a stable reference (defined outside the render cycle in this example). `React.memo` compares the prop reference and skips the re-render.

### Real-World Cases

- **Large lists:** Memoizing list item components.
- **Charts and visualisations:** Preventing re-renders when only unrelated state changes.
- **Layout components:** Isolating static layout sections from dynamic parents.

### References

- CoreUI — How to prevent unnecessary re-renders in React — https://coreui.io/answers/how-to-prevent-unnecessary-re-renders-in-react/
- Open Storefront Framework Developer's Guide — https://docs.oracle.com/en/industries/retail/retail-digital-commerce/latest/dosfa/avoid-unnecessary-rerendering.htm
- Webkul — Reducing Re-renders in React Applications — https://webkul.com/blog/reducing-re-renders-in-react-applications/

---

## 5. `useMemo`

### Definitions

**Core Definition:** `useMemo` is a React Hook that caches the result of an expensive calculation between renders.

**Technical Definition:** `useMemo(calculateValue, dependencies)` returns a memoized value. React calls `calculateValue` on the initial render and re-runs it only when a dependency changes (compared with `Object.is`). `useMemo` is used for expensive computations — filtering, sorting, and building derived data structures. It should not be used for simple object literals or arrays, where the overhead outweighs the benefit.

**Beginner-Friendly Explanation:** `useMemo` remembers the result of a slow calculation. If the inputs have not changed, React reuses the remembered result instead of recalculating.

### Purposes

- To avoid repeating expensive calculations on every render.
- To stabilise object and array references passed to memoized children.
- To keep derived data computations efficient.

### Syntax Rules and Structure

```jsx
const cachedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**Syntax Rules:**

1. Use only for genuinely expensive computations.
2. Include all reactive dependencies.
3. Do not use for simple primitives or trivial object literals.
4. `useMemo` caches a value, not a function (use `useCallback` for functions).

**Constraints and Limitations:**

- Every `useMemo` call has comparison overhead.
- Overuse adds complexity without benefit.
- React may discard the cache in certain situations.

### Annotated Code Examples

```jsx
function ProductList({ products, filterText }) {
  const filtered = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(filterText.toLowerCase())
    ),
    [products, filterText]
  );

  return <ul>{filtered.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

**Expected Output:** The filter runs only when `products` or `filterText` changes.

**Why this output:** `useMemo` caches the filtered array and recomputes it only when a dependency changes.

### Real-World Cases

- **Data grids:** Sorting and filtering large datasets.
- **Search:** Filtering results by query.
- **Derived data:** Computing aggregates or transformed structures.

### References

- CoreUI — How to prevent unnecessary re-renders in React — https://coreui.io/answers/how-to-prevent-unnecessary-re-renders-in-react/
- RT Camp — Performance Optimization — https://rtcamp.com/handbook/react-best-practices/performance-optimization/
- Steve Kinney — Identity Stability Props — https://stevekinney.com/courses/react-performance/identity-stability-props

---

## 6. `useCallback`

### Definitions

**Core Definition:** `useCallback` is a React Hook that caches a function reference between renders.

**Technical Definition:** `useCallback(fn, dependencies)` returns the same function reference between renders as long as its dependencies have not changed. It is used when passing functions to memoized children, because a new function reference on every render would break `React.memo`'s shallow comparison. `useCallback` should only be used when a function is passed to a memoized child or used as a dependency in another Hook.

**Beginner-Friendly Explanation:** `useCallback` keeps the same function around between renders. This matters when a memoized child receives the function as a prop — if the function reference changes, the child re-renders anyway.

### Purposes

- To stabilise function references passed to memoized children.
- To prevent memoized components from re-rendering unnecessarily.
- To stabilise dependencies in `useEffect` and other Hooks.

### Syntax Rules and Structure

```jsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

**Syntax Rules:**

1. Use only when passing functions to memoized children.
2. Include all reactive dependencies.
3. Avoid “defensive wrapping” of every function.
4. Use functional state updates to keep dependencies minimal.

**Constraints and Limitations:**

- Inline arrow functions in JSX still create new references and break memoization.
- Overuse adds overhead without benefit.

### Annotated Code Examples

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // ✅ Stable function reference
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <MemoizedChild onClick={handleClick} />
    </>
  );
}

const MemoizedChild = memo(function MemoizedChild({ onClick }) {
  return <button onClick={onClick}>Click me</button>;
});
```

**Expected Output:** Clicking the count button does not re-render `MemoizedChild`.

**Why this output:** `useCallback` keeps `handleClick` stable, so `React.memo` sees an unchanged `onClick` prop and skips the child re-render.

### Real-World Cases

- **Design system components:** Passing stable callbacks to memoized primitives.
- **Lists:** Stable handlers for each list item.
- **Effects:** Stable callbacks used as effect dependencies.

### References

- CoreUI — How to prevent unnecessary re-renders — https://coreui.io/answers/how-to-prevent-unnecessary-re-renders-in-react/
- RT Camp — Performance Optimization — https://rtcamp.com/handbook/react-best-practices/performance-optimization/
- React Compiler Introduction — https://zh-hans.react.dev/learn/react-compiler/introduction

---

## 7. Component Composition

### Definitions

**Core Definition:** Component composition is the practice of building complex components by nesting smaller components, and it is a primary architectural tool for controlling re-render scope.

**Technical Definition:** Composition allows you to place state in a component that is a **sibling** or **parent** of the expensive components, rather than an ancestor, so that state changes do not propagate through the expensive subtree. It also enables the **children as props** pattern, where expensive JSX is created outside the state-owning component and passed in as `children`, preserving its identity across renders.

**Beginner-Friendly Explanation:** Composition means arranging components so that state lives in a small component, and the expensive components are outside its re-render blast radius. You build the UI by putting small pieces together, not by making one giant component.

### Purposes

- To isolate state changes to the smallest possible subtree.
- To prevent re-render waves from reaching expensive components.
- To make components easier to test and reason about.

### Syntax Rules and Structure

**Composition Patterns for Performance:**

| Pattern | Mechanism |
|---------|-----------|
| Sibling state | State component and expensive component are siblings under a common parent |
| Children as props | Expensive JSX is created outside the state-owning component and passed as `children` |
| Extracted children | Complex child components are extracted into separate components |

**Syntax Rules:**

1. Place state in the component that consumes it, not at the top of the tree.
2. Pass expensive JSX as `children` so it is not re-created when the parent's state changes.
3. Extract child components to create new reconciliation boundaries.

### Annotated Code Examples

```jsx
// ✅ Children as props: expensive children keep their identity
function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {children}
    </div>
  );
}

function App() {
  return (
    <Counter>
      <ExpensiveComponent /> {/* Created by App, not Counter */}
    </Counter>
  );
}
```

**Expected Output:** `ExpensiveComponent` does not re-render when the counter increments.

**Why this output:** The `children` prop is created in `App`'s render. When `Counter` re-renders due to its own state change, the `children` reference remains the same, so React skips reconciling that subtree.

### Real-World Cases

- **Layout components:** Passing static sidebars and headers as children to a dynamic layout.
- **Modals:** Passing modal content as children to a modal controller.
- **Forms:** Passing expensive preview components as children to a form state provider.

### References

- Children Pattern to Prevent Parent Re-Renders — https://github.com/pproenca/dot-skills
- State Colocation with Consumers — https://github.com/pproenca/dot-skills
- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state

---

## 8. Component Splitting

### Definitions

**Core Definition:** Component splitting is the practice of breaking a large component into smaller components to create distinct reconciliation boundaries.

**Technical Definition:** When a component is large, any state change within it re-renders the entire component, including all JSX and child components. Splitting the component into smaller components isolates state to the component that owns it, so a state change in one part does not re-render the others. This is described as “physically blocking the spread of the re-render wave by breaking components down into logical” units.

**Beginner-Friendly Explanation:** If one big component holds all the state and renders everything, changing anything re-renders everything. Splitting it into smaller components means only the relevant part re-renders.

### Purposes

- To limit the re-render blast radius of a state change.
- To isolate expensive subtrees from frequently changing state.
- To improve maintainability and testability.

### Syntax Rules and Structure

**Splitting Strategy:**

| Before | After |
|--------|-------|
| One component with state and expensive children | State-owning component + memoized/expensive sibling components |
| All state at the root | State colocated in the component that uses it |
| Large JSX block with inline handlers | Extracted child components with stable props |

**Syntax Rules:**

1. Identify which parts of the UI are expensive and which state changes frequently.
2. Move state down to the component that actually uses it.
3. Extract the expensive parts into separate components so they are outside the state-owning component.

### Annotated Code Examples

```jsx
// ❌ One large component: everything re-renders on every state change
function Dashboard() {
  const [filter, setFilter] = useState('');
  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      <ExpensiveChart />
      <ExpensiveTable />
    </div>
  );
}

// ✅ Split: filter state only re-renders the input
function Dashboard() {
  return (
    <div>
      <FilterInput /> {/* State lives here */}
      <ExpensiveChart />
      <ExpensiveTable />
    </div>
  );
}
```

**Expected Output:** In the split version, typing in the filter does not re-render the chart or table.

**Why this output:** The filter state is isolated in `FilterInput`. `ExpensiveChart` and `ExpensiveTable` are siblings, not children, so they are outside the re-render blast radius.

### Real-World Cases

- **Dashboard widgets:** Each widget manages its own state.
- **Forms:** Each section manages its own fields.
- **Data tables:** Filters and pagination split from the table body.

### References

- React Performance and Preventing Unnecessary Re-renders — https://aspnetzero.com/blog/react-performance-and-preventing-unnecessary-renders
- RT Camp — Performance Optimization — https://rtcamp.com/handbook/react-best-practices/performance-optimization/

---

## 9. State Colocation

### Definitions

**Core Definition:** State colocation is the practice of keeping each piece of state as close as possible to the components that actually use it.

**Technical Definition:** State lifted too high forces intermediate components to forward props they do not use, and every parent in the chain re-renders when the state changes, even though only the leaf component reads it. Moving state to the component that consumes it eliminates the forwarding chain and confines re-renders. The rule is: keep state as low as it can live, and lift it only when another component genuinely needs to read or change it.

**Beginner-Friendly Explanation:** Put state where it is used. If only a small part of the UI needs a piece of state, that state should live in that small part, not at the top of the app.

### Purposes

- To eliminate prop drilling.
- To confine re-renders to the components that actually depend on the state.
- To simplify data flow and improve maintainability.

### Syntax Rules and Structure

**Colocation Decision:**

| State usage | Placement |
|-------------|-----------|
| Only one component uses it | Inside that component |
| A small subtree uses it | In the subtree's root |
| Multiple distant components use it | Lift to their nearest common ancestor or use Context |

**Syntax Rules:**

1. Start with state in the component that uses it.
2. Lift state only when a sibling or cousin also needs it.
3. Use Context only when prop drilling becomes unwieldy, not as a default.

### Annotated Code Examples

```jsx
// ❌ State too high: every keystroke re-renders the whole page
function ProductPage() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <div>
      <Header />
      <SearchBar query={searchQuery} onQueryChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
      <Footer />
    </div>
  );
}

// ✅ State colocated: only the search subtree re-renders
function ProductPage() {
  return (
    <div>
      <Header />
      <ProductSearch /> {/* State lives here */}
      <Footer />
    </div>
  );
}

function ProductSearch() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <>
      <SearchBar query={searchQuery} onQueryChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
    </>
  );
}
```

**Expected Output:** In the colocated version, typing in the search bar does not re-render `Header` or `Footer`.

**Why this output:** The `searchQuery` state is confined to `ProductSearch`. `Header` and `Footer` are outside its subtree, so they are not re-rendered.

### Real-World Cases

- **Search bars:** Keeping query state inside the search component.
- **Modals:** Keeping open/closed state inside the modal trigger.
- **Tabs:** Keeping active tab state inside the tab group.

### References

- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state
- State Colocation with Consumers — https://github.com/pproenca/dot-skills
- React Performance and Preventing Unnecessary Re-renders — https://aspnetzero.com/blog/react-performance-and-preventing-unnecessary-renders

---

## 10. Children as Props

### Definitions

**Core Definition:** The children-as-props pattern passes expensive JSX as the `children` prop so it is created outside the state-owning component and preserves its identity across renders.

**Technical Definition:** When a parent component owns state that changes frequently, all JSX declared inside that parent re-renders on every state change. Moving expensive children above the state-owning component and passing them as `children` props preserves their identity across renders, skipping reconciliation entirely. The `children` prop is created by the outer component and passed down as a prop; React sees the same element reference and skips re-rendering that subtree.

**Beginner-Friendly Explanation:** If a component has state that changes often, do not put expensive children inside it. Put the expensive children in the parent and pass them down as `children`. That way, when the component's state changes, the children are not re-created.

### Purposes

- To prevent expensive children from re-rendering when the parent's state changes.
- To preserve the identity of static or expensive subtrees.
- To combine state management with isolated rendering.

### Syntax Rules and Structure

```jsx
function Stateful({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      {children}
    </div>
  );
}

<Stateful>
  <ExpensiveComponent /> {/* Created outside Stateful */}
</Stateful>
```

**Syntax Rules:**

1. Create the expensive JSX in the outer component, not inside the state-owning component.
2. Pass it as `children` to the state-owning component.
3. The state-owning component renders `{children}` in its output.

### Annotated Code Examples

```jsx
function ExpensiveTree() {
  console.log('ExpensiveTree rendered');
  return <div>{/* heavy content */}</div>;
}

function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {children}
    </div>
  );
}

function App() {
  return (
    <Counter>
      <ExpensiveTree />
    </Counter>
  );
}
```

**Expected Output:** `"ExpensiveTree rendered"` logs once. Clicking the counter does not re-render it.

**Why this output:** `ExpensiveTree` is created in `App`'s render and passed as `children`. When `Counter` re-renders, the `children` prop reference is unchanged, so React skips reconciling that subtree.

### Real-World Cases

- **Modal controllers:** Passing modal body content as children to a stateful modal controller.
- **Layout components:** Passing static sidebars and headers as children to a dynamic layout.
- **Providers:** Passing the application tree as children to context providers.

### References

- Children Pattern to Prevent Parent Re-Renders — https://github.com/pproenca/dot-skills
- State Colocation with Consumers — https://github.com/pproenca/dot-skills

---

## 11. Render Optimization

### Definitions

**Core Definition:** Render optimization is the practice of minimizing the work React performs during the render phase — component execution, reconciliation, and commit.

**Technical Definition:** Render optimization combines architectural techniques (colocation, splitting, children as props) with reference stability (stable objects, arrays, and functions) and memoization (`React.memo`, `useMemo`, `useCallback`). The goal is to reduce the number of component functions executed and the amount of reconciliation work performed, not to manipulate the DOM directly.

**Beginner-Friendly Explanation:** Render optimization means making React's rendering work smaller and more predictable. It is about doing less unnecessary work, not about hacking the DOM.

### Purposes

- To reduce the total time spent in the render phase.
- To keep updates fast and predictable.
- To maintain responsiveness as the component tree grows.

### Syntax Rules and Structure

**Render Optimization Checklist:**

| Check | Action |
|-------|--------|
| Is state in the right place? | Colocate it with its consumers |
| Is the component too large? | Split it into smaller components |
| Are expensive children re-rendering? | Pass them as `children` |
| Are prop references stable? | Move static values outside; use `useMemo`/`useCallback` |
| Is a pure component re-rendering often? | Wrap in `React.memo` |
| Are list keys stable? | Use unique IDs, not indices |

**Syntax Rules:**

1. Architecture first; memoization second.
2. Measure with the Profiler before and after changes.
3. Avoid premature optimization; it adds complexity without benefit.

### Annotated Code Examples

```jsx
// ✅ Stable props + memoized child + colocated state
const MemoizedRow = memo(function Row({ row, onToggle }) {
  return <tr><td>{row.name}</td><td><input type="checkbox" onChange={() => onToggle(row.id)} /></td></tr>;
});

function Table({ rows }) {
  const [selected, setSelected] = useState(new Set());

  const handleToggle = useCallback((id) => {
    setSelected(prev => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  }, []);

  return (
    <table>
      <tbody>
        {rows.map(row => (
          <MemoizedRow key={row.id} row={row} onToggle={handleToggle} />
        ))}
      </tbody>
    </table>
  );
}
```

**Expected Output:** Toggling one row does not re-render the other rows.

**Why this output:** `handleToggle` is stable via `useCallback`, and `MemoizedRow` skips re-rendering when its `row` prop is unchanged and `onToggle` reference is stable.

### Real-World Cases

- **Data tables:** Row-level memoization and stable callbacks.
- **Forms:** Stable field handlers and isolated field components.
- **Charts:** Memoized chart components with stable data references.

### References

- RT Camp — Performance Optimization — https://rtcamp.com/handbook/react-best-practices/performance-optimization/
- Steve Kinney — Identity Stability Props — https://stevekinney.com/courses/react-performance/identity-stability-props
- CoreUI — How to prevent unnecessary re-renders — https://coreui.io/answers/how-to-prevent-unnecessary-re-renders-in-react/

---

## 12. Reconciliation Optimization

### Definitions

**Core Definition:** Reconciliation optimization is the practice of structuring element trees and keys so React's diffing algorithm performs minimal DOM work.

**Technical Definition:** React's reconciler compares the new element tree with the previous one. When element types differ, React tears down the old tree and builds a new one. When types are the same, React updates only changed attributes and recurses on children. For lists, React uses `key` props to match children across renders. Stable, unique keys let React match elements and perform minimal DOM moves; index keys cause full DOM teardown and rebuild when items are reordered.

**Beginner-Friendly Explanation:** Reconciliation optimization is about helping React compare the old and new UI efficiently. The most important thing is using stable keys in lists so React knows which items are the same.

### Purposes

- To minimize DOM mutations during updates.
- To preserve component state across renders.
- To avoid unnecessary teardown and rebuild of list items.

### Syntax Rules and Structure

**Reconciliation Rules:**

| Situation | Behaviour |
|-----------|-----------|
| Different element types | Full teardown and rebuild |
| Same element type | Update changed attributes, recurse on children |
| Same component type | Update props, re-render |
| Stable key | Match by key, minimal DOM moves |
| Index key | Match by position, full rebuild on reorder |

**Syntax Rules:**

1. Use stable, unique IDs from data as keys.
2. Never use array indices for dynamic lists.
3. Never use `Math.random()` as keys.
4. Keep element types consistent across renders where possible.

### Annotated Code Examples

```jsx
// ❌ Index keys: reorder causes full rebuild
<ul>
  {sorted.map((item, index) => (
    <li key={index}>{item.name}</li>
  ))}
</ul>

// ✅ Stable ID keys: reorder causes minimal DOM moves
<ul>
  {sorted.map((item) => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
```

**Expected Output:** The stable-key version preserves component state and performs minimal DOM updates when the list is reordered.

**Why this output:** Index keys change when items are reordered, causing React to treat every item as changed. Stable IDs let React match items across renders.

### Real-World Cases

- **Sortable tables:** Stable keys prevent full rebuilds when sorting.
- **Chat feeds:** Stable keys preserve message state when new messages arrive.
- **Notifications:** Stable keys prevent animation glitches on reorder.

### References

- Use Stable Keys for List Rendering Performance — https://github.com/pproenca/dot-skills
- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- Reconciliation (React legacy docs) — https://ru.react.js.org/docs/reconciliation.html

---

## 13. Key Optimization

### Definitions

**Core Definition:** Key optimization is the practice of choosing the right `key` prop for list items so React's reconciler can efficiently match elements across renders.

**Technical Definition:** The `key` prop tells React which child elements are the same across renders. React uses keys to match children in the original tree with children in the subsequent tree. Stable, unique, and predictable keys enable minimal DOM moves and preserve component state; unstable keys (indices, random values) cause full teardown and rebuild.

**Beginner-Friendly Explanation:** Keys are how React identifies list items. If the key is stable and unique, React knows exactly which item is which and only moves the DOM nodes that changed.

### Purposes

- To minimize DOM manipulations during list updates.
- To preserve component state across reorders.
- To avoid rendering bugs caused by mismatched keys.

### Syntax Rules and Structure

**Key Selection Guide:**

| Scenario | Recommended key |
|----------|-----------------|
| Data has a unique ID | `item.id` |
| Data has a composite unique value | `` `${item.category}-${item.name}` `` |
| Data has no unique value | Generate a stable ID when the data is created |
| Never | Array index, `Math.random()` |

**Syntax Rules:**

1. Keys must be stable across renders.
2. Keys must be unique among siblings.
3. Keys must be predictable (not random).
4. Use IDs from the data, not derived values that change.

### Annotated Code Examples

```jsx
// ✅ Stable composite key
<ul>
  {items.map(item => (
    <li key={`${item.category}-${item.name}`}>{item.name}</li>
  ))}
</ul>
```

**Expected Output:** List items are matched correctly across renders, preserving state and minimizing DOM updates.

**Why this output:** The composite key is stable and unique, so React can match items even if the list is filtered or reordered.

### Real-World Cases

- **Filtered lists:** Stable keys preserve item state when the filter changes.
- **Dynamic forms:** Stable keys prevent input state from being assigned to the wrong row.
- **Animations:** Stable keys prevent animation glitches.

### References

- CoreUI — How to use the key prop in React lists — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- Use Stable Keys for List Rendering Performance — https://github.com/pproenca/dot-skills
- Reconciliation (React legacy docs) — https://ru.react.js.org/docs/reconciliation.html

---

## 14. List Virtualization

### Definitions

**Core Definition:** List virtualization is the technique of rendering only the list items currently visible in the viewport (plus a small buffer), rather than rendering the entire list.

**Technical Definition:** Virtualization (also called windowing) calculates which items fit in the visible area based on scroll position and renders only those items, keeping the DOM node count constant regardless of dataset size. This enables 60fps scrolling and low memory usage even with 100K+ items. The trade-off is that virtualized content is not in the DOM until scrolled, which can affect SEO and browser find-in-page.

**Beginner-Friendly Explanation:** Virtualization means only rendering the list items you can actually see. Instead of creating 10,000 DOM nodes, you create the 10 that fit on screen. As you scroll, React swaps out the visible items. The list feels smooth because the browser only manages a handful of nodes.

### Purposes

- To render massive lists at 60fps.
- To keep memory usage low regardless of dataset size.
- To avoid the performance cost of creating thousands of DOM nodes.
- To enable smooth infinite scrolling and large data tables.

### Syntax Rules and Structure

**Virtualization Concepts:**

| Concept | Description |
|---------|-------------|
| Window | The set of items currently rendered |
| Overscan | Extra items rendered above and below the viewport |
| Item size | Fixed or variable height of each item |
| Scroll container | The element that scrolls the virtualized list |

**Syntax Rules:**

1. Use virtualization only for large lists (100+ items).
2. Provide accurate item sizes or a measurement function for variable heights.
3. Tune overscan based on scroll speed and device performance.
4. Use stable keys for virtualized items.

### Annotated Code Examples

```jsx
import { FixedSizeList as List } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style}>Row {index}</div>
);

function VirtualizedList() {
  return (
    <List
      height={500}       // Viewport height
      itemCount={10000}  // Total items
      itemSize={35}      // Row height
      width={300}
    >
      {Row}
    </List>
  );
}
```

**Expected Output:** A scrollable list of 10,000 rows that renders only the visible rows, staying smooth at 60fps.

**Why this output:** `react-window` calculates which rows fit in the 500px viewport and renders only those (plus a small buffer), keeping the DOM node count constant.

### Real-World Cases

- **Chat history:** Rendering only visible messages.
- **Data tables:** Rendering only visible rows.
- **Infinite feeds:** Loading and rendering items as the user scrolls.

### References

- Handling Large Lists (Virtualization) — https://github.com/MarcoNasi/AI-Skills
- Virtualization Specialist Skill — https://github.com/curiositech/windags-skills

---

## 15. Windowing

### Definitions

**Core Definition:** Windowing is the mechanism behind virtualization: maintaining a moving "window" of rendered items that corresponds to the visible viewport.

**Technical Definition:** Windowing calculates the start and end indices of the items that should be rendered based on the scroll position, viewport height, and item sizes. Items outside the window are unmounted. As the user scrolls, the window moves and React mounts/unmounts items accordingly.

**Beginner-Friendly Explanation:** Windowing is the "sliding window" of items that are currently rendered. As you scroll, the window slides, and React swaps out the items that left the window for the ones that entered it.

### Purposes

- To keep the rendered item count proportional to the viewport, not the dataset.
- To enable smooth scrolling with massive datasets.
- To reduce memory usage from DOM nodes.

### Syntax Rules and Structure

**Windowing Parameters:**

| Parameter | Effect |
|-----------|--------|
| Viewport size | How many items fit on screen |
| Item size | How much space each item takes |
| Overscan | Extra buffer items for smooth scrolling |
| Scroll offset | Which items are currently visible |

**Syntax Rules:**

1. The window must be recalculated on scroll.
2. Overscan should be tuned based on device and scroll speed.
3. Item sizes must be accurate for correct window calculation.

### Annotated Code Examples

```jsx
// Conceptual windowing calculation
function getVisibleRange(scrollTop, viewportHeight, itemSize, totalItems) {
  const start = Math.floor(scrollTop / itemSize);
  const visibleCount = Math.ceil(viewportHeight / itemSize);
  const end = Math.min(start + visibleCount, totalItems);
  return { start, end };
}
```

**Expected Output:** The function returns the range of items that should be rendered for the current scroll position.

**Why this output:** The start index is determined by dividing the scroll offset by the item size. The end index is the start plus the number of items that fit in the viewport.

### Real-World Cases

- **Virtualized tables:** Calculating visible rows from scroll position.
- **Chat logs:** Rendering only the messages in view.
- **Image galleries:** Rendering only visible thumbnails.

### References

- Handling Large Lists (Virtualization) — https://github.com/MarcoNasi/AI-Skills
- Virtualization Specialist Skill — https://github.com/curiositech/windags-skills

---

## 16. React Window

### Definitions

**Core Definition:** `react-window` is a lightweight React library for virtualizing large lists and grids.

**Technical Definition:** `react-window` provides components like `FixedSizeList`, `VariableSizeList`, and `FixedSizeGrid` that render only the visible items in a scrollable container. It is designed to be small and fast, making it suitable for most virtualization needs.

**Beginner-Friendly Explanation:** `react-window` is a small, focused library that makes virtualized lists easy. You give it the total item count, the item height, and a row component, and it handles the windowing.

### Purposes

- To virtualize large lists with minimal configuration.
- To keep bundle size small.
- To support fixed-size and variable-size items.

### Syntax Rules and Structure

```jsx
import { FixedSizeList as List } from 'react-window';

<List
  height={500}
  itemCount={10000}
  itemSize={35}
  width={300}
>
  {Row}
</List>
```

**Syntax Rules:**

1. Provide `height`, `itemCount`, `itemSize`, and `width`.
2. The child is a component that receives `{ index, style }`.
3. Apply the provided `style` to the row element for correct positioning.

### Annotated Code Examples

```jsx
import { FixedSizeList as List } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style} className="row">
    Row {index}
  </div>
);

function App() {
  return (
    <List height={400} itemCount={5000} itemSize={40} width={300}>
      {Row}
    </List>
  );
}
```

**Expected Output:** A smooth, scrollable list of 5,000 rows that only renders the visible rows.

**Why this output:** `react-window` calculates the visible range and passes the correct `style` (including `top` positioning) to each rendered row.

### Real-World Cases

- **Chat applications:** Virtualized message lists.
- **Admin panels:** Large data tables.
- **Dropdowns:** Long option lists.

### References

- Handling Large Lists (Virtualization) — https://github.com/MarcoNasi/AI-Skills
- Virtualization Specialist Skill — https://github.com/curiositech/windags-skills

---

## 17. React Virtualized

### Definitions

**Core Definition:** `react-virtualized` is a feature-rich React library for virtualizing lists, tables, and grids, offering advanced capabilities beyond basic windowing.

**Technical Definition:** `react-virtualized` provides a comprehensive set of virtualization components, including `List`, `Table`, `Grid`, `Masonry`, and `Collection`. It supports variable sizes, infinite loading, and complex layouts. It is larger and more feature-rich than `react-window`.

**Beginner-Friendly Explanation:** `react-virtualized` is the older, more feature-packed sibling of `react-window`. It can handle more complex layouts, but it is heavier.

### Purposes

- To virtualize complex layouts (tables, grids, masonry).
- To support advanced features like infinite loading and dynamic sizes.
- To provide a comprehensive virtualization toolkit.

### Syntax Rules and Structure

```jsx
import { List } from 'react-virtualized';

<List
  width={300}
  height={500}
  rowCount={1000}
  rowHeight={35}
  rowRenderer={({ index, key, style }) => (
    <div key={key} style={style}>Row {index}</div>
  )}
/>
```

**Syntax Rules:**

1. Provide `width`, `height`, `rowCount`, `rowHeight`, and `rowRenderer`.
2. `rowRenderer` receives `{ index, key, style }` and must apply the `style` and `key`.

### Annotated Code Examples

```jsx
import { List } from 'react-virtualized';

function VirtualizedTable({ rows }) {
  return (
    <List
      width={600}
      height={400}
      rowCount={rows.length}
      rowHeight={30}
      rowRenderer={({ index, key, style }) => (
        <div key={key} style={style}>{rows[index].name}</div>
      )}
    />
  );
}
```

**Expected Output:** A virtualized table that renders only visible rows.

**Why this output:** `react-virtualized` calculates the visible range and calls `rowRenderer` only for those rows.

### Real-World Cases

- **Complex data tables:** Multi-column tables with fixed headers.
- **Masonry layouts:** Virtualized Pinterest-style grids.
- **Infinite loading:** Virtualized lists with server-side pagination.

### References

- Handling Large Lists (Virtualization) — https://github.com/MarcoNasi/AI-Skills
- Virtualization Specialist Skill — https://github.com/curiositech/windags-skills

---

## 18. TanStack Virtual

### Definitions

**Core Definition:** TanStack Virtual is a headless, framework-agnostic virtualization library that provides the core windowing logic without prescribing markup or styling.

**Technical Definition:** TanStack Virtual is a headless UI utility for virtualizing long lists of elements. It provides hooks like `useVirtualizer` that calculate the virtual items, total size, and measurement functions. The consumer renders the markup and applies the positioning. It supports fixed, variable, and dynamic item sizes, as well as 2D virtualization for grids.

**Beginner-Friendly Explanation:** TanStack Virtual gives you the virtualization logic but lets you build the UI however you want. It is framework-agnostic and highly flexible.

### Purposes

- To virtualize lists with complete control over markup and styling.
- To support complex scenarios (variable heights, dynamic measurement, 2D grids).
- To work across frameworks (React, Vue, Solid, etc.).

### Syntax Rules and Structure

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

const virtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 35,
});
```

**Syntax Rules:**

1. Provide `count`, `getScrollElement`, and `estimateSize`.
2. The virtualizer returns `getVirtualItems()` and `getTotalSize()`.
3. Render only the virtual items and apply the calculated `transform` or `top`.

### Annotated Code Examples

```jsx
import { useRef } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 35,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map(virtualItem => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              transform: `translateY(${virtualItem.start}px)`,
              height: virtualItem.size,
            }}
          >
            {items[virtualItem.index]}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Expected Output:** A headless virtualized list that renders only visible items with absolute positioning.

**Why this output:** `useVirtualizer` calculates which items are visible and provides their positions. The consumer renders only those items.

### Real-World Cases

- **Custom design systems:** Virtualization that matches a specific visual style.
- **2D grids:** Virtualizing both rows and columns.
- **Chat with variable heights:** Dynamic measurement of message heights.

### References

- Handling Large Lists (Virtualization) — https://github.com/MarcoNasi/AI-Skills
- Virtualization Specialist Skill — https://github.com/curiositech/windags-skills
- TanStack Virtual documentation — https://tanstack.com/virtual

---

## 19. Rendering Best Practices

### Definitions

**Core Definition:** Rendering best practices are guidelines for structuring React components and state to minimize unnecessary rendering work while maintaining correctness and maintainability.

**Technical Definition:** Rendering best practices combine architectural principles (state colocation, component splitting, children as props), reference stability (stable objects, arrays, and functions), targeted memoization (`React.memo`, `useMemo`, `useCallback`), stable keys, and virtualization for large lists. The overarching principle is to measure first, fix architecture before adding memoization, and avoid premature optimization.

**Beginner-Friendly Explanation:** These are the habits that keep your app fast. Put state where it is used. Split big components. Use stable references. Memoize only when needed. Use stable keys in lists. Virtualize long lists. And always measure before you optimize.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Colocate state** | Keep state as close as possible to its consumers |
| **Split components** | Break large components into smaller reconciliation boundaries |
| **Use children as props** | Pass expensive JSX from outside the state-owning component |
| **Stabilise references** | Move static values outside; use `useMemo`/`useCallback` only when needed |
| **Memoize selectively** | `React.memo` for pure, frequently re-rendered components |
| **Use stable keys** | Never use array indices for dynamic lists |
| **Virtualize large lists** | Render only visible items for 100+ item lists |
| **Measure first** | Use React DevTools Profiler before optimizing |
| **Avoid premature optimization** | Do not memoize everything; it adds complexity without benefit |

### Annotated Code Examples

```jsx
// ✅ Best-practice component: colocated state, memoized child, stable callback, stable key
const Row = memo(function Row({ item, onSelect }) {
  return (
    <tr>
      <td>{item.name}</td>
      <td><input type="checkbox" onChange={() => onSelect(item.id)} /></td>
    </tr>
  );
});

function Table({ items }) {
  const [selected, setSelected] = useState(new Set());

  const handleSelect = useCallback((id) => {
    setSelected(prev => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  }, []);

  return (
    <table>
      <tbody>
        {items.map(item => (
          <Row key={item.id} item={item} onSelect={handleSelect} />
        ))}
      </tbody>
    </table>
  );
}
```

**Expected Output:** Selecting one row does not re-render the other rows, and the selection state is preserved.

**Why this output:** The state is colocated in `Table`. `handleSelect` is stable via `useCallback`. `Row` is memoized, so unchanged rows skip re-rendering. Keys are stable IDs.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Large-scale dashboards:** Colocation and virtualization are essential.
- **Design systems:** Stable references and memoization keep primitives fast.

### References

- RT Camp — Performance Optimization — https://rtcamp.com/handbook/react-best-practices/performance-optimization/
- React Performance and Preventing Unnecessary Re-renders — https://aspnetzero.com/blog/react-performance-and-preventing-unnecessary-renders
- Steve Kinney — Identity Stability Props — https://stevekinney.com/courses/react-performance/identity-stability-props
- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state
- CoreUI — How to prevent unnecessary re-renders — https://coreui.io/answers/how-to-prevent-unnecessary-re-renders-in-react/
- CoreUI — How to use the key prop — https://coreui.io/answers/how-to-use-the-key-prop-in-react-lists/
- Handling Large Lists (Virtualization) — https://github.com/MarcoNasi/AI-Skills
- Virtualization Specialist Skill — https://github.com/curiositech/windags-skills
- Use Stable Keys for List Rendering Performance — https://github.com/pproenca/dot-skills
- Children Pattern to Prevent Parent Re-Renders — https://github.com/pproenca/dot-skills
- State Colocation with Consumers — https://github.com/pproenca/dot-skills
- React Compiler Introduction — https://zh-hans.react.dev/learn/react-compiler/introduction
- Open Storefront Framework Developer's Guide — https://docs.oracle.com/en/industries/retail/retail-digital-commerce/latest/dosfa/avoid-unnecessary-rerendering.htm
- Webkul — Reducing Re-renders in React Applications — https://webkul.com/blog/reducing-re-renders-in-react-applications/
- Reconciliation (React legacy docs) — https://ru.react.js.org/docs/reconciliation.html