# React Memoization: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React memoization is the practice of caching component render output, computed values, and function references across render cycles so that React can skip work when inputs have not changed.

**Technical Definition:** React memoization operates at three complementary levels. **Component-level memoization** (`React.memo`) caches a component's rendered output and skips re-rendering it when its props are shallowly equal to the previous props. **Value-level memoization** (`useMemo`) caches the result of a computation and recomputes it only when its declared dependencies change (compared with `Object.is`). **Function-level memoization** (`useCallback`) caches a function reference, returning the same reference across renders unless its dependencies change. All three rely on React's `Object.is` comparison and exist to preserve referential equality, avoid expensive recomputation, and prevent unnecessary re-render propagation. They are performance optimisations, not semantic guarantees: React may discard memoised values, and the React Compiler now automates much of this work. Memoization has real costs—memory allocation for the cache, comparison overhead per render, and increased code complexity—so it must be applied surgically rather than universally.

**Beginner-Friendly Explanation:** Imagine you are cooking dinner and you keep chopping the same onions over and over, even though you already chopped them. Memoization is like putting the chopped onions in the fridge—if nothing has changed, you grab the same bowl instead of chopping again. `React.memo` does this for whole components, `useMemo` for computed values, and `useCallback` for functions. But there is a catch: if you put everything in the fridge, you run out of space and spend more time organising than cooking. That is why React developers say: do not memoize prematurely—only when you have measured a real performance problem.

### Key Characteristics

- **Three Levels:** `React.memo` (component), `useMemo` (value), `useCallback` (function).
- **Shallow Comparison:** All three use `Object.is` for reference comparison, not deep equality.
- **Referential Equality is the Goal:** Preserving the same object/function reference is what makes memoization effective downstream.
- **Not a Semantic Guarantee:** React may discard memoised values under memory pressure; the React Compiler may replace manual memoization.
- **Cost-Benefit Dependent:** Each memoization adds memory and comparison overhead; it must pay for itself in saved work.
- **Chain Dependency:** `React.memo` only helps if the props passed to the memoised component are themselves stable (`useMemo`, `useCallback`, or primitives).
- **Composition Rule:** `useCallback(fn, deps)` is exactly `useMemo(() => fn, deps)`; they are interchangeable in principle.

### Prerequisites

- Solid understanding of React function components, JSX, and the render cycle.
- Familiarity with `useState`, `useEffect`, and `useRef`.
- Working knowledge of JavaScript closures, references, and `Object.is`.
- Understanding of how parent re-renders propagate to children.
- Awareness of the React DevTools Profiler for measuring render performance.

### Related Programming Areas

- **Rendering Performance:** Re-render triggers, component boundaries, and the Fiber architecture.
- **Referential Equality:** `Object.is`, `===`, and why new references defeat memoization.
- **State Architecture:** Where state lives and how updates propagate.
- **React Compiler:** Automatic memoization at build time.
- **Profiling:** React DevTools Profiler, flame graphs, and render counts.

### Core Concepts / Features

1. Pure Components with `React.memo`
2. Value Caching with `useMemo`
3. Stable Callbacks with `useCallback`
4. Cost-Benefit Tradeoffs

---

## Core Concept 1: Pure Components with `React.memo`

### Definitions

**Core Definition:** `React.memo` is a higher-order component that wraps a functional component and skips re-rendering it when its props are shallowly equal to the previous render's props.

**Technical Definition:** `React.memo(Component, arePropsEqual?)` returns a memoised version of `Component`. Before re-rendering, React compares each prop with the previous render using `Object.is`. If every prop is equal (or the custom `arePropsEqual` function returns `true`), React skips the re-render and reuses the last rendered output. If any prop differs, React re-renders the component. `React.memo` only affects re-renders triggered by the parent; it does not prevent re-renders caused by the component's own state, context, or hooks. A custom `arePropsEqual(prevProps, nextProps)` function can implement deep or selective comparison, but it is discouraged because it is easy to write incorrectly and can hide bugs.

**Beginner-Friendly Explanation:** Imagine a light switch that only flips when the room's brightness actually needs to change. `React.memo` is like a smart switch: it checks whether the information coming in (props) is the same as last time, and if it is, it does not bother re-rendering. But it only checks the surface—if you pass a new object with the same contents, it still thinks something changed. That is why you often need `useMemo` and `useCallback` alongside `React.memo` to keep props stable.

### Purposes

- To skip re-rendering expensive components when their props have not changed.
- To isolate heavy components from unrelated state changes in their parents.
- To reduce the total number of component renders in a tree.
- To preserve DOM state (focus, scroll, input values) by avoiding unnecessary re-renders.
- To make the re-render graph more predictable and reduce render churn.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const MemoisedComponent = React.memo(function Component(props) {
  // ...
});
```

**General Syntax with Custom Comparator:**
```jsx
const MemoisedComponent = React.memo(
  function Component(props) { /* ... */ },
  function arePropsEqual(prevProps, nextProps) {
    // Return true if props are equal (skip re-render)
    return prevProps.id === nextProps.id;
  }
);
```

**Component Breakdown:**
- `React.memo(Component)`: Wraps the component with shallow prop comparison.
- `arePropsEqual(prevProps, nextProps)`: Optional custom comparator; return `true` to skip re-render.
- The wrapped component must be defined outside the parent (or hoisted) so its reference is stable.

**Shallow Comparison Example:**
```jsx
const Card = React.memo(function Card({ title, count }) {
  console.log('Card rendered');
  return <div>{title}: {count}</div>;
});

function App() {
  const [title] = useState('Revenue');
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <Card title={title} count={count} />
    </div>
  );
}
```

**Component Breakdown:**
- `title` is a primitive string; `count` is a primitive number.
- `React.memo` compares `title` and `count` with `Object.is`.
- `Card` re-renders only when `count` changes.

**Syntax Rules:**
- Define the wrapped component outside the parent component, or hoist it to a stable reference.
- Pass only primitives or stable references as props to a memoised component.
- For object, array, or function props, stabilise them with `useMemo` or `useCallback`.
- Custom `arePropsEqual` must return `true` to skip re-render and `false` to re-render.
- `React.memo` does not prevent re-renders caused by the component's own state or context.
- `React.memo` does not prevent the initial render.

**Constraints and Limitations:**
- Shallow comparison means new object/array/function references defeat memoization.
- Custom comparators are easy to get wrong and can mask bugs (e.g., ignoring a prop that should trigger a re-render).
- `React.memo` adds a comparison cost per render; for trivial components, it can be a net loss.
- Memoization does not help if props change on every render (e.g., inline literals).
- The React Compiler may make `React.memo` unnecessary in many cases.

### Annotated Code Examples

**Example 1: Memoised Component with Stable Props**

```jsx
import { useState, memo, useMemo, useCallback } from 'react';

const ProductCard = memo(function ProductCard({ product, onSelect }) {
  console.log(`ProductCard rendered: ${product.name}`);
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onSelect(product.id)}>Select</button>
    </div>
  );
});

export default function ProductList() {
  const [selectedId, setSelectedId] = useState(null);
  const [filter, setFilter] = useState('');

  const products = useMemo(
    () => [
      { id: 1, name: 'Laptop', price: 999 },
      { id: 2, name: 'Phone', price: 699 },
      { id: 3, name: 'Tablet', price: 499 },
    ],
    []
  );

  const filtered = useMemo(
    () => products.filter(p => p.name.toLowerCase().includes(filter.toLowerCase())),
    [products, filter]
  );

  const handleSelect = useCallback((id) => setSelectedId(id), []);

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter products..."
      />
      <p>Selected: {selectedId ?? 'none'}</p>
      {filtered.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onSelect={handleSelect}
        />
      ))}
    </div>
  );
}
```

**Expected Output (console):**
```
ProductCard rendered: Laptop
ProductCard rendered: Phone
ProductCard rendered: Tablet
--- typing "la" ---
ProductCard rendered: Laptop
--- selecting a product ---
(no ProductCard renders)
```

**Why This Output Occurs:** `products` is memoised with `useMemo`, so its reference is stable. `handleSelect` is memoised with `useCallback`, so its reference is stable. `ProductCard` is wrapped in `React.memo`, so it re-renders only when its `product` or `onSelect` prop changes. When the filter changes, only the matching product's card re-renders. When `selectedId` changes, no cards re-render because neither prop changed for any card.

**Example 2: Custom Comparator (Discouraged but Useful)**

```jsx
const Chart = React.memo(
  function Chart({ data, options }) {
    console.log('Chart rendered');
    return <div>Chart with {data.length} points</div>;
  },
  (prevProps, nextProps) => {
    // Skip re-render if data length and options.animate are the same
    return (
      prevProps.data.length === nextProps.data.length &&
      prevProps.options.animate === nextProps.options.animate
    );
  }
);
```

**Expected Output:** `Chart` re-renders only when `data.length` or `options.animate` changes, even if other props (e.g., `options.color`) change. This is dangerous if `options.color` affects the visual output—the chart would not update.

**Why This Output Occurs:** The custom comparator ignores most of the props. It is used here to demonstrate the risk: if a prop that affects rendering is ignored, the component will not update when that prop changes. Custom comparators should be used only when the ignored props genuinely do not affect output.

### Real-World Cases

- **Data grids:** Rows are memoised so filtering/sorting only re-renders affected rows.
- **Charts:** Chart components are memoised so unrelated state changes do not re-render them.
- **Lists with actions:** Row components are memoised with stable callbacks for delete/edit.
- **Expensive layouts:** Layout components are memoised so theme or sidebar toggles do not re-render the main content.
- **Third-party integrations:** Memoised wrappers around maps, editors, and video players.

### References

- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – `memo` Caveats: https://react.dev/reference/react/memo#caveats
- React Official Documentation – `memo` Custom Comparison: https://react.dev/reference/react/memo#specifying-a-custom-comparison-function
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children

---

## Core Concept 2: Value Caching with `useMemo`

### Definitions

**Core Definition:** `useMemo` is a React Hook that caches the result of a computation between renders, recomputing it only when one of its dependencies changes.

**Technical Definition:** `useMemo(calculateValue, dependencies)` memoises the value returned by `calculateValue`, re-executing it only when `dependencies` change (compared with `Object.is`). On the first render, `useMemo` calls the function and caches the result. On subsequent renders, if no dependency has changed, it returns the cached value without calling the function. If any dependency changes, it calls the function again and updates the cache. `useMemo` is a performance optimisation, not a semantic guarantee: React may discard memoised values in the future (the React Compiler now automates this). It is most valuable when the computation is genuinely expensive (filtering, sorting, mathematical transformations) or when the result is passed as a prop to a memoised child or used as a dependency in another Hook.

**Beginner-Friendly Explanation:** `useMemo` is like a calculator with a memory button. You work out an answer once, and if you need it again and nothing has changed, you press the memory button instead of recalculating. It is useful when the calculation is expensive—like sorting a large list or running a complex transformation. If the calculation is trivial (like `a + b`), using `useMemo` is overkill.

### Purposes

- To avoid recomputing expensive values on every render.
- To preserve referential equality of objects and arrays passed as props to memoised children.
- To stabilise dependencies for `useEffect`, `useCallback`, and other `useMemo` calls.
- To prevent infinite loops caused by new object references in dependency arrays.
- To skip heavy mathematical, filtering, or data-transformation loops.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const memoisedValue = useMemo(() => computeExpensive(a, b), [a, b]);
```

**Component Breakdown:**
- `() => computeExpensive(a, b)`: The calculation function; called only when dependencies change.
- `[a, b]`: The dependency array; compared with `Object.is`.
- `memoisedValue`: The cached or freshly computed value.

**Common Patterns:**
```jsx
// Memoise a filtered/sorted list
const visibleItems = useMemo(
  () => items.filter(i => i.name.includes(query)).sort(byName),
  [items, query]
);

// Memoise an object passed as a prop
const config = useMemo(() => ({ theme, locale }), [theme, locale]);

// Memoise an expensive derivation
const stats = useMemo(() => computeStatistics(data), [data]);

// Memoise an array passed to a memoised child
const activeIds = useMemo(
  () => items.filter(i => i.active).map(i => i.id),
  [items]
);
```

**Syntax Rules:**
- Call `useMemo` at the top level of a component or Hook; never inside loops, conditions, or nested functions.
- The calculation function must be pure; it must not mutate anything or cause side effects.
- Provide a dependency array; without it, `useMemo` recomputes on every render (pointless).
- The dependency array should include every reactive value used inside the calculation.
- For expensive computations, `useMemo` is worthwhile; for trivial ones, it adds overhead.
- `useMemo` is a performance optimisation, not a semantic guarantee; React may discard the cache.
- Memoising a value that is not passed to a memoised child or used as a dependency is usually wasted work.

**Constraints and Limitations:**
- `useMemo` adds memory overhead and a comparison per render; overuse degrades performance.
- The calculation function runs during render; it must not cause side effects.
- Memoization is per-component-instance; it does not share across instances.
- If a dependency is an object or array created during render, the memo will never be reused (new reference every render).
- `useMemo` does not prevent the initial render or the recomputation when dependencies genuinely change.
- The React Compiler may make `useMemo` unnecessary in many cases.

### Annotated Code Examples

**Example 1: Memoising an Expensive Computation**

```jsx
import { useState, useMemo } from 'react';

function computePrimes(limit) {
  console.log('Computing primes up to', limit);
  const primes = [];
  for (let n = 2; n <= limit; n++) {
    let isPrime = true;
    for (let d = 2; d * d <= n; d++) {
      if (n % d === 0) { isPrime = false; break; }
    }
    if (isPrime) primes.push(n);
  }
  return primes;
}

export default function PrimeCalculator() {
  const [limit, setLimit] = useState(10000);
  const [darkMode, setDarkMode] = useState(false);

  // ✅ Memoised: recomputes only when `limit` changes, not when `darkMode` changes
  const primes = useMemo(() => computePrimes(limit), [limit]);

  return (
    <div style={{
      background: darkMode ? '#222' : '#fff',
      color: darkMode ? '#fff' : '#000',
      padding: 20,
    }}>
      <h1>Prime Calculator</h1>
      <input
        type="number"
        value={limit}
        onChange={(e) => setLimit(Number(e.target.value))}
      />
      <button onClick={() => setDarkMode((d) => !d)}>
        Toggle {darkMode ? 'Light' : 'Dark'} Mode
      </button>
      <p>Found {primes.length} primes up to {limit}.</p>
    </div>
  );
}
```

**Expected Output (console):**
```
Computing primes up to 10000
--- toggle dark mode ---
(no log — memoised value reused)
--- change limit to 20000 ---
Computing primes up to 20000
```

**Why This Output Occurs:** `useMemo(() => computePrimes(limit), [limit])` caches the result keyed by `limit`. When `darkMode` changes, the component re-renders, but `limit` is unchanged, so `useMemo` returns the cached primes without re-running the calculation.

**Example 2: Memoising a Prop for a Memoised Child**

```jsx
import { useState, useMemo, memo } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items, style }) {
  console.log('ExpensiveList rendered');
  return (
    <ul style={style}>
      {items.map((item) => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

export default function App() {
  const [query, setQuery] = useState('');
  const [theme, setTheme] = useState('light');

  const items = useMemo(
    () => [
      { id: 1, name: 'Apple' },
      { id: 2, name: 'Banana' },
      { id: 3, name: 'Cherry' },
    ],
    []
  );

  const filtered = useMemo(
    () => items.filter(i => i.name.toLowerCase().includes(query.toLowerCase())),
    [items, query]
  );

  const style = useMemo(
    () => ({ color: theme === 'dark' ? '#fff' : '#000' }),
    [theme]
  );

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <ExpensiveList items={filtered} style={style} />
    </div>
  );
}
```

**Expected Output (console):**
```
ExpensiveList rendered
--- typing ---
ExpensiveList rendered   (filtered changed)
--- toggling theme ---
ExpensiveList rendered   (style changed)
```

**Why This Output Occurs:** `filtered` and `style` are memoised, so they change only when their dependencies change. `ExpensiveList` re-renders only when `filtered` or `style` changes—not when unrelated state changes. Without `useMemo`, both props would be new references on every render, and `ExpensiveList` would re-render on every keystroke and toggle.

### Real-World Cases

- **Data grids:** Memoising filtered, sorted, and paginated rows.
- **Charts:** Memoising computed data points and configuration objects.
- **Search:** Memoising filtered search results.
- **Statistics:** Memoising aggregates and derived metrics.
- **Configuration objects:** Memoising config objects passed to third-party components.
- **Dependency stabilisation:** Memoising objects used as dependencies in `useEffect`.

### References

- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – Skipping Expensive Calculations: https://react.dev/reference/react/useMemo#skipping-expensive-recalculations
- React Official Documentation – Memoizing a Dependency of Another Hook: https://react.dev/reference/react/useMemo#memoizing-a-dependency-of-another-hook
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler

---

## Core Concept 3: Stable Callbacks with `useCallback`

### Definitions

**Core Definition:** `useCallback` is a React Hook that returns a memoised version of a function, preserving its referential identity across renders unless its dependencies change.

**Technical Definition:** `useCallback(fn, dependencies)` memoises the function `fn` and returns the same function reference across renders as long as `dependencies` remain unchanged (compared with `Object.is`). It is equivalent to `useMemo(() => fn, dependencies)`. The returned function has the same body but a stable reference, which matters when passing callbacks to memoised children (`React.memo`) or when using them in dependency arrays of other hooks (`useEffect`, `useMemo`, `useCallback`). `useCallback` is a performance optimisation, not a semantic guarantee. It is only beneficial when the callback is passed to a memoised component or used as a dependency; otherwise, it adds overhead without benefit.

**Beginner-Friendly Explanation:** `useCallback` is like saving a phone number in your contacts. Every time you call the number, you dial the same saved entry—not a fresh copy of the number. If the person changes their number (a dependency changes), you update the contact. Without `useCallback`, every render creates a new "phone number" (function), and your memoised children think they have been given a different contact—so they re-render.

### Purposes

- To preserve referential equality of event handlers passed to memoised children.
- To stabilise callbacks used as dependencies in `useEffect` or `useMemo`.
- To prevent infinite loops caused by new function references in dependency arrays.
- To avoid re-subscribing to event listeners or re-running effects unnecessarily.
- To retain the memoization chain across a component tree (parent → child → grandchild).

### Syntax Rules and Structure

**General Syntax:**
```jsx
const memoisedCallback = useCallback((arg) => { doSomething(arg, dep); }, [dep]);
```

**Component Breakdown:**
- `(arg) => { ... }`: The function to memoise.
- `[dep]`: The dependency array; the function is recreated only when `dep` changes.
- `memoisedCallback`: The stable function reference.

**Common Patterns:**
```jsx
// Stable click handler passed to a memoised child
const handleClick = useCallback((id) => {
  setSelectedId(id);
}, []);

// Handler that depends on a value
const handleSearch = useCallback((query) => {
  setResults(search(query, category));
}, [category]);

// Handler used in a useEffect dependency
const handleResize = useCallback(() => {
  setWidth(window.innerWidth);
}, []);

useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, [handleResize]);

// Functional state update avoids depending on state
const increment = useCallback(() => setCount(c => c + 1), []);
```

**Syntax Rules:**
- Call `useCallback` at the top level of a component or Hook; never inside loops or conditions.
- The function must be pure with respect to React state; mutations should be done via setters or effects.
- Provide a dependency array that includes all reactive values used inside the function.
- An empty dependency array creates a stable function that captures the initial render's values—this causes stale closures if the function reads state.
- `useCallback` is unnecessary for functions passed to non-memoised children (they re-render regardless).
- Use the functional form of state setters (`setX((prev) => ...)`) to avoid depending on `x`.
- `useCallback` is equivalent to `useMemo(() => fn, deps)`; choose whichever reads more clearly.

**Constraints and Limitations:**
- `useCallback` does not prevent re-renders by itself; it only helps when the child is memoised with `React.memo`.
- If the callback depends on frequently changing values, it is recreated frequently, negating the benefit.
- The callback body runs at call time, not at memoization time; only the reference is memoised.
- Overusing `useCallback` adds complexity without measurable benefit.
- The React Compiler may insert `useCallback` automatically.

### Annotated Code Examples

**Example 1: Stable Callback for a Memoised Child**

```jsx
import { useState, useCallback, memo, useRef } from 'react';

const MemoisedRow = memo(function MemoisedRow({ item, onDelete }) {
  const renderCount = useRef(0);
  renderCount.current += 1;
  console.log(`Row ${item.id} rendered (count: ${renderCount.current})`);

  return (
    <li>
      {item.name}
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </li>
  );
});

export default function ItemList() {
  const [items, setItems] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' },
  ]);
  const [filter, setFilter] = useState('');

  // ✅ Stable callback — same reference across renders
  const handleDelete = useCallback((id) => {
    setItems((prev) => prev.filter((item) => item.id !== id));
  }, []);

  const visibleItems = items.filter((item) =>
    item.name.toLowerCase().includes(filter.toLowerCase())
  );

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter items..."
      />
      <ul>
        {visibleItems.map((item) => (
          <MemoisedRow key={item.id} item={item} onDelete={handleDelete} />
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output (console):**
```
Row 1 rendered (count: 1)
Row 2 rendered (count: 1)
Row 3 rendered (count: 1)
--- typing "a" ---
Row 1 rendered (count: 2)
Row 2 rendered (count: 2)
Row 3 rendered (count: 2)
--- typing "ap" ---
Row 1 rendered (count: 3)
--- deleting row 1 ---
Row 2 rendered (count: 3)
Row 3 rendered (count: 3)
```

**Why This Output Occurs:** `handleDelete` is memoised with `useCallback` and an empty dependency array, so its reference is stable. `MemoisedRow` is wrapped in `React.memo`, so it re-renders only when its `item` or `onDelete` prop changes. Typing in the filter changes `visibleItems`, so the visible rows re-render (because their `item` prop is the same reference but the list is re-mapped—React re-renders children when the parent re-renders and the element is recreated). Deleting a row changes the `items` array, so the remaining rows re-render. In all cases, `onDelete` remains stable, so it does not contribute to re-renders.

**Example 2: Stable Callback as an Effect Dependency**

```jsx
import { useState, useCallback, useEffect } from 'react';

export default function WindowSizeTracker() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  // ✅ Stable callback — effect re-subscribes only once
  const handleResize = useCallback(() => {
    setSize({ width: window.innerWidth, height: window.innerHeight });
  }, []);

  useEffect(() => {
    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, [handleResize]);

  return <p>Window: {size.width} × {size.height}</p>;
}
```

**Expected Output:** The component displays the current window size and updates on resize. The effect runs once on mount (because `handleResize` is stable), and the listener is attached and removed once. Without `useCallback`, `handleResize` would be a new function on every render, causing the effect to re-run on every render (removing and re-adding the listener repeatedly).

**Why This Output Occurs:** `useCallback(() => { ... }, [])` returns the same function reference across all renders. The `useEffect` dependency `[handleResize]` therefore never changes, so the effect runs only on mount and cleanup. This is the canonical pattern for event listeners in React.

### Real-World Cases

- **Lists with per-row actions:** Delete, edit, toggle handlers passed to memoised rows.
- **Event listeners in effects:** Stable listeners for resize, scroll, or online/offline events.
- **Context callbacks:** Stable dispatch functions or action creators in context values.
- **Form field handlers:** Stable change handlers passed to memoised input components.
- **Third-party integrations:** Stable callbacks passed to chart, map, or editor libraries.
- **Memoization chains:** Ensuring a parent's `useCallback` is not invalidated by a child's unstable callback.

### References

- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – `useCallback` vs `useMemo`: https://react.dev/reference/react/useCallback#usecallback-vs-usememo
- React Official Documentation – Preventing an Effect from Re-connecting: https://react.dev/reference/react/useCallback#preventing-an-effect-from-re-connecting
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler

---

## Core Concept 4: Cost-Benefit Tradeoffs

### Definitions

**Core Definition:** Cost-benefit tradeoffs in memoization are the analysis of whether the performance benefit of caching (avoided computation, avoided re-renders) outweighs the costs (memory allocation, comparison overhead, code complexity).

**Technical Definition:** Every memoization has explicit and implicit costs. **Memory cost:** `React.memo`, `useMemo`, and `useCallback` all allocate memory for cached values and dependency arrays. **Comparison cost:** React compares dependencies or props with `Object.is` on every render; for memoised components, this is per-prop. **Complexity cost:** Memoization adds code that must be read, maintained, and debugged; it can obscure the data flow. **Correctness risk:** Incorrect dependency arrays cause stale closures; incorrect custom comparators cause missed updates. The benefit is measured in avoided work: a heavy computation that takes 50ms and runs on every render is a clear win; a trivial calculation (`a + b`) that takes 0.001ms is a clear loss. The decision must be informed by profiling, not intuition. The React Compiler changes the calculus by automating memoization at build time, eliminating the complexity cost for compliant code.

**Beginner-Friendly Explanation:** Memoization is like putting things in a fridge. If you cook a big meal and will eat it again tomorrow, the fridge saves you time. If you cook a tiny snack and eat it immediately, the fridge wastes space and time. In React, memoization is worth it when the saved work is significant and the same inputs recur. It is not worth it when the work is trivial or the inputs change every time. The best way to know is to measure—open React DevTools Profiler and see where the time actually goes.

### Purposes

- To identify when memoization genuinely improves performance.
- To avoid the anti-pattern of memoizing everything "just in case."
- To reduce memory usage and comparison overhead in large applications.
- To keep code readable and maintainable.
- To focus optimisation on measured bottlenecks rather than guesses.
- To prepare for the React Compiler, which handles memoization automatically.

### Syntax Rules and Structure

**When to Memoise (Checklist):**

| Signal | Action |
|---|---|
| Value/function is passed to a `React.memo` child | Memoise it (`useMemo`/`useCallback`) |
| Value/function is a dependency of `useEffect`/`useMemo`/`useCallback` | Memoise it |
| Calculation is genuinely expensive (profiler shows > 1ms) | Memoise with `useMemo` |
| Value is a context provider value | Memoise it |
| Calculation is trivial (`a + b`, `arr.length`) | Do **not** memoise |
| Value is passed to a non-memoised child | Do **not** memoise |
| Value is a primitive | Do **not** memoise |
| Callback is passed to a non-memoised child | Do **not** memoise with `useCallback` |
| You are "not sure if it helps" | Do **not** memoise; profile first |

**Anti-Pattern: Memoising Everything**
```jsx
// ❌ Over-memoised: trivial calculations and unused memoisation
function OverMemoised({ items, filter, onSelect }) {
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(filter)),
    [items, filter]
  );

  const count = useMemo(() => filtered.length, [filtered]);        // Trivial
  const title = useMemo(() => `Found ${count} items`, [count]);    // Trivial
  const handleSelect = useCallback((item) => onSelect(item), [onSelect]); // May not help

  return (
    <div>
      <h2>{title}</h2>
      <ul>
        {filtered.map(item => (
          <li key={item.id} onClick={() => handleSelect(item)}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Component Breakdown:**
- `filtered` is genuinely expensive for large lists; worth memoising.
- `count` and `title` are trivial; the comparison overhead may exceed the computation cost.
- `handleSelect` is only useful if the `<li>` is memoised; otherwise, it adds overhead.

**Optimised Version (After Profiling):**
```jsx
// ✅ Targeted memoisation: only the expensive filter and the memoised row
function Optimised({ items, filter, onSelect }) {
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(filter)),
    [items, filter]
  );

  const count = filtered.length;
  const title = `Found ${count} items`;
  const handleSelect = useCallback((item) => onSelect(item), [onSelect]);

  return (
    <div>
      <h2>{title}</h2>
      <ul>
        {filtered.map(item => (
          <MemoisedRow key={item.id} item={item} onSelect={handleSelect} />
        ))}
      </ul>
    </div>
  );
}
```

**Component Breakdown:**
- `filtered` is still memoised (genuinely expensive).
- `count` and `title` are computed directly (trivial).
- `handleSelect` is memoised because it is passed to `MemoisedRow`.

**Syntax Rules:**
- Profile before memoising; use React DevTools Profiler to identify real bottlenecks.
- Memoise values passed to `React.memo` children, used as dependencies, or genuinely expensive.
- Do not memoise primitives, trivial calculations, or values passed to non-memoised children.
- Do not memoise callbacks passed to non-memoised children.
- Remove memoization that does not measurably improve performance.
- Prefer colocation and component boundaries over memoization when possible.
- Consider the React Compiler to automate memoization where it applies.

**Constraints and Limitations:**
- Profiling in development mode includes StrictMode double-renders, which can skew measurements; profile in production mode for accurate results.
- Memoization benefits are more visible in large, complex components than in small ones.
- The React Compiler (React 19+) can automatically memoise, reducing the need for manual `useMemo` and `useCallback`.
- Premature optimisation can mask design problems (e.g., a component doing too much).
- Memoization does not fix architectural problems like prop drilling or over-rendering due to context.

### Annotated Code Example: Measuring the Impact

```jsx
import { useState, useMemo, useCallback, memo, Profiler } from 'react';

// Without memoization
function ExpensiveListNoMemo({ items, filter }) {
  const filtered = items.filter(i => i.name.includes(filter));
  return (
    <ul>
      {filtered.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

// With memoization
const ExpensiveListWithMemo = memo(function ExpensiveListWithMemo({ items, filter }) {
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(filter)),
    [items, filter]
  );
  return (
    <ul>
      {filtered.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

const ITEMS = Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  name: `Item ${i}`,
}));

export default function App() {
  const [filter, setFilter] = useState('');
  const [theme, setTheme] = useState('light');

  function onRender(id, phase, actualDuration) {
    console.log(`${id} (${phase}): ${actualDuration.toFixed(2)}ms`);
  }

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>

      <Profiler id="NoMemo" onRender={onRender}>
        <ExpensiveListNoMemo items={ITEMS} filter={filter} />
      </Profiler>

      <Profiler id="WithMemo" onRender={onRender}>
        <ExpensiveListWithMemo items={ITEMS} filter={filter} />
      </Profiler>
    </div>
  );
}
```

**Expected Output (console):**
```
NoMemo (mount): 45.20ms
WithMemo (mount): 45.80ms
--- toggling theme (no filter change) ---
NoMemo (update): 42.10ms    ← re-renders unnecessarily
(no WithMemo update)        ← skipped by React.memo + useMemo
```

**Why This Output Occurs:** The `Profiler` component logs the time spent rendering each subtree. `ExpensiveListNoMemo` re-renders on every theme toggle because it is not memoised. `ExpensiveListWithMemo` is wrapped in `React.memo` and uses `useMemo` for the filtered list, so it is skipped entirely when the theme changes but the filter does not. The profiler shows the difference: the memoised version saves ~42ms per theme toggle.

### Real-World Cases

- **Large data tables:** Profiling shows that filtering and sorting on every render is the bottleneck; memoise those.
- **Chart components:** Profiling shows that recomputing chart data is expensive; memoise it.
- **Lists with many items:** Profiling shows that all rows re-render on every parent render; add `React.memo` and stabilise callbacks.
- **Context providers:** Profiling shows that all consumers re-render on every provider render; memoise the context value.
- **Small components:** Profiling shows no measurable benefit from memoization; remove it.

### References

- React Official Documentation – `useMemo` (Performance Optimisation): https://react.dev/reference/react/useMemo
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – Profiler: https://react.dev/reference/react/Profiler
- React Official Documentation – `memo` Caveats: https://react.dev/reference/react/memo#caveats
- Kent C. Dodds – When to useMemo and useCallback: https://kentcdodds.com/blog/usememo-and-usecallback
- Kent C. Dodds – One React Mistake That's Keeping Your App Slow: https://kentcdodds.com/blog/optimize-react-re-renders
- Donald Knuth – Structured Programming with go to Statements: https://dl.acm.org/doi/10.1145/356635.356640

---

## Comparison and Decision Guidance

| Technique | What It Memoises | When to Use | When NOT to Use |
|---|---|---|---|
| **`React.memo`** | Component render output | Component re-renders often with same props; expensive to render | Component's props change every render; trivial component |
| **`useMemo`** | Computed value | Expensive calculation; dependency of another Hook; context value | Trivial calculation; primitive; passed to non-memoised child |
| **`useCallback`** | Function reference | Callback passed to memoised child; dependency of `useEffect` | Callback passed to non-memoised child; trivial event handler |
| **Custom `React.memo` comparator** | Component render output with custom equality | Rare; when shallow comparison is insufficient | Almost always; prefer stabilising props |
| **React Compiler** | All of the above | Any compliant codebase | Non-compliant code (violates Rules of React) |

**Decision Guidance:**
- **Write the simplest code first** without memoization.
- **Profile with React DevTools** to find real bottlenecks.
- **Memoise values that are dependencies** of other Hooks (to prevent infinite loops).
- **Memoise callbacks passed to memoised children** (`React.memo`).
- **Memoise genuinely expensive calculations** (>1ms in the profiler).
- **Do not memoise primitives, trivial calculations, or props of non-memoised children.**
- **Use functional state updates** (`setX((prev) => ...)`) to avoid dependencies.
- **Move functions and constants outside the component** when they do not depend on props/state.
- **Enable the React Compiler** if available; it automates much of this.
- **Satisfy the `exhaustive-deps` ESLint rule**; do not disable it.
- **Remove memoization that does not measurably help.**

---

## References

- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – `memo` Caveats: https://react.dev/reference/react/memo#caveats
- React Official Documentation – `memo` Custom Comparison: https://react.dev/reference/react/memo#specifying-a-custom-comparison-function
- React Official Documentation – Skipping Expensive Calculations: https://react.dev/reference/react/useMemo#skipping-expensive-recalculations
- React Official Documentation – Memoizing a Dependency of Another Hook: https://react.dev/reference/react/useMemo#memoizing-a-dependency-of-another-hook
- React Official Documentation – `useCallback` vs `useMemo`: https://react.dev/reference/react/useCallback#usecallback-vs-usememo
- React Official Documentation – Preventing an Effect from Re-connecting: https://react.dev/reference/react/useCallback#preventing-an-effect-from-re-connecting
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – React Compiler Introduction: https://react.dev/learn/react-compiler/introduction
- React Official Documentation – React Compiler Installation: https://react.dev/learn/react-compiler/installation
- React Official Documentation – Rules of React: https://react.dev/reference/rules
- React Official Documentation – Profiler: https://react.dev/reference/react/Profiler
- MDN Web Docs – `Object.is()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- Kent C. Dodds – When to useMemo and useCallback: https://kentcdodds.com/blog/usememo-and-usecallback
- Kent C. Dodds – One React Mistake That's Keeping Your App Slow: https://kentcdodds.com/blog/optimize-react-re-renders
- Donald Knuth – Structured Programming with go to Statements: https://dl.acm.org/doi/10.1145/356635.356640