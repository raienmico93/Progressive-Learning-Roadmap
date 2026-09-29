# React Performance Hooks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React performance hooks—`useMemo` and `useCallback`—are built-in hooks that cache (memoise) values and function references across renders, preventing unnecessary recalculations and re-renders in React applications.

**Technical Definition:** `useMemo` and `useCallback` are React Hooks that leverage memoisation to preserve referential equality of computed values and function definitions across renders. `useMemo(fn, deps)` returns a memoised value, recomputing it only when one of its dependencies changes (compared with `Object.is`). `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)` and returns a memoised function reference. Both rely on the dependency array to determine when to invalidate the cache; omitting or mismanaging dependencies causes stale closures or unnecessary recomputation. These hooks are performance optimisations, not semantic guarantees: React may discard memoised values in future versions (as the React Compiler now does automatically). They are primarily useful when (1) passing stable references to memoised children (`React.memo`), (2) computing expensive values, or (3) serving as dependencies for other hooks.

**Beginner-Friendly Explanation:** Imagine you are cooking dinner and you keep chopping the same onions over and over, even though you already chopped them. `useMemo` is like putting the chopped onions in the fridge—if you need them again and nothing has changed, you grab the same bowl instead of chopping again. `useCallback` is the same idea, but for functions: instead of creating a brand-new function every render, you reuse the same one. But here is the catch: if you put everything in the fridge, you will run out of space and spend more time organising than cooking. That is why React developers say: do not memoise prematurely—only when you have measured a real performance problem.

### Key Characteristics

- **Memoisation, Not Caching Forever:** Memoised values are discarded when dependencies change or when React chooses to (e.g., under memory pressure).
- **Referential Equality is the Goal:** `useMemo` and `useCallback` preserve the same object or function reference across renders, which matters for `React.memo`, `useEffect`, and other dependency arrays.
- **`Object.is` Comparison:** Dependencies are compared with `Object.is`, not deep equality. `[1, 2] !== [1, 2]` because they are different references.
- **Not a Semantic Guarantee:** React's documentation states that `useMemo` is a performance optimisation, not a semantic guarantee. React may "forget" some memoised values.
- **The React Compiler Changes the Calculus:** In React 19 with the React Compiler enabled, `useMemo` and `useCallback` are often inserted automatically, reducing the need for manual memoisation.
- **Premature Optimisation is an Anti-Pattern:** Memoisation adds memory overhead and complexity; it should be applied only when profiling reveals a real bottleneck.
- **Dependency Arrays Are Contracts:** Missing dependencies cause stale closures; unstable dependencies cause the memo to never be reused. The `exhaustive-deps` ESLint rule enforces correctness.

### Prerequisites

- Solid understanding of React function components, JSX, and the render cycle.
- Familiarity with `useState`, `useEffect`, and `useRef`.
- Working knowledge of JavaScript closures, references, and `Object.is`.
- Understanding of `React.memo` and when it prevents re-renders.
- Awareness of the React DevTools Profiler for measuring render performance.

### Related Programming Areas

- **Memoisation:** Caching computed results and function references.
- **Referential Equality:** `Object.is`, `===`, and how React compares values.
- **Render Optimisation:** `React.memo`, `useMemo`, `useCallback`, and the React Compiler.
- **Closures:** How JavaScript captures variables and why stale closures occur.
- **Profiling:** Using React DevTools Profiler and browser performance tools.

### Core Concepts / Features

1. Memoisation Concepts
2. Referential Equality
3. `useMemo`
4. `useCallback`
5. Dependency Management
6. Avoiding Premature Optimisation

---

## Core Concept 1: Memoisation Concepts

### Definitions

**Core Definition:** Memoisation is an optimisation technique that stores the results of expensive function calls and returns the cached result when the same inputs occur again.

**Technical Definition:** Memoisation is a form of caching in which a function's output is stored keyed by its inputs, so subsequent calls with identical inputs return the cached output without recomputation. In React, memoisation appears at three levels: (1) **Component-level** — `React.memo` memoises a component's rendered output based on its props; (2) **Value-level** — `useMemo` memoises the result of a computation across renders; (3) **Function-level** — `useCallback` memoises a function reference across renders. React's memoisation is *per-component-instance* and *per-render*, and it relies on `Object.is` (reference equality) to determine whether dependencies have changed. Unlike classical memoisation, React's memoisation is not guaranteed to persist—React may discard memoised values to free memory.

**Beginner-Friendly Explanation:** Memoisation is like keeping a notebook of answers you have already worked out. When the same question comes up again, you look up the answer instead of redoing the calculation. In React, `useMemo` is the notebook for values, `useCallback` is the notebook for functions, and `React.memo` is the notebook for whole components. But the notebook only helps if the question is actually the same—if the inputs change, you have to work out the answer again.

### Purposes

- To avoid recomputing expensive values on every render when inputs have not changed.
- To preserve referential equality of objects and functions across renders.
- To prevent unnecessary re-renders in memoised child components.
- To stabilise dependencies for `useEffect` and other hooks.
- To reduce CPU and memory pressure in large or complex applications.

### Syntax Rules and Structure

**Three Levels of React Memoisation:**

| Level | API | What It Memoises | Keyed By |
|---|---|---|---|
| Component | `React.memo` | Rendered output | Props (shallow) |
| Value | `useMemo` | Computed value | Dependencies (`Object.is`) |
| Function | `useCallback` | Function reference | Dependencies (`Object.is`) |

**General Syntax (`React.memo`):**
```jsx
const MemoisedChild = React.memo(function Child({ value, onClick }) {
  return <button onClick={onClick}>{value}</button>;
});
// Re-renders only when props change (shallow comparison)
```

**General Syntax (`useMemo`):**
```jsx
const expensiveValue = useMemo(() => computeExpensive(a, b), [a, b]);
```

**General Syntax (`useCallback`):**
```jsx
const stableHandler = useCallback(() => { doSomething(a); }, [a]);
```

**Syntax Rules:**
- `React.memo` performs a shallow comparison of props by default; custom comparison via the second argument is possible but discouraged (it is easy to get wrong).
- `useMemo` and `useCallback` take a function and a dependency array; the memo is recomputed when any dependency changes.
- The dependency array is compared with `Object.is`; objects, arrays, and functions are compared by reference.
- Memoisation is per-component-instance: each mounted component has its own memo cache.
- Memoised values are not shared across component instances.

**Constraints and Limitations:**
- Memoisation is not free: it costs memory and a comparison per render.
- `React.memo` only helps if the parent re-renders with the same props; if props change every render, it does nothing.
- Memoisation does not prevent the component from rendering the first time; it only prevents subsequent renders.
- React may discard memoised values in the future (the React Compiler may replace manual memoisation).
- Overusing memoisation can make the code harder to read and the app slower (due to memory pressure and comparison overhead).

### Annotated Code Example: Memoisation at Three Levels

```jsx
import { useState, useMemo, useCallback, memo } from 'react';

// 1. Component-level memoisation
const ExpensiveList = memo(function ExpensiveList({ items, onSelect }) {
  console.log('ExpensiveList rendered');
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id} onClick={() => onSelect(item)}>
          {item.name}
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

  // 2. Value-level memoisation
  const filteredItems = useMemo(() => {
    console.log('Filtering items');
    return items.filter((item) =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [items, query]);

  // 3. Function-level memoisation
  const handleSelect = useCallback((item) => {
    console.log('Selected:', item.name);
  }, []);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ExpensiveList items={filteredItems} onSelect={handleSelect} />
    </div>
  );
}
```

**Expected Output:** Typing in the input filters the list. Without `useMemo`, "Filtering items" logs on every keystroke (which is expected). Without `useCallback`, `handleSelect` is a new function on every render, causing `ExpensiveList` to re-render even when `filteredItems` and `handleSelect` have not changed. With both, `ExpensiveList` re-renders only when `filteredItems` changes.

**Why This Output Occurs:** `React.memo` prevents `ExpensiveList` from re-rendering when its props are referentially equal. `useMemo` keeps `filteredItems` stable between renders unless `items` or `query` change. `useCallback` keeps `handleSelect` stable forever (empty dependency array). Together, they ensure `ExpensiveList` re-renders only when the filtered list actually changes.

### Real-World Cases

- **Large data tables:** Memoising filtered/sorted rows to avoid recomputing on every keystroke.
- **Chart libraries:** Memoising computed data points for expensive chart renders.
- **Map components:** Memoising GeoJSON data and handler callbacks.
- **Virtualised lists:** Memoising row renderers and item components.

### References

- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- MDN Web Docs – Memoization: https://developer.mozilla.org/en-US/docs/Glossary/Memoization

---

## Core Concept 2: Referential Equality

### Definitions

**Core Definition:** Referential equality is the comparison of two values by their memory reference (not their contents), which is how React determines whether props, state, and dependencies have changed.

**Technical Definition:** JavaScript uses two kinds of equality: **value equality** (`===` for primitives, deep comparison for objects) and **referential equality** (`Object.is`, `===` for objects, which compares memory addresses). React uses `Object.is` to compare previous and next values in dependency arrays and to bail out of re-renders. For primitives (strings, numbers, booleans), `Object.is` behaves like `===` with two exceptions: `Object.is(NaN, NaN)` is `true`, and `Object.is(0, -0)` is `false`. For objects, arrays, and functions, `Object.is` returns `true` only if both operands reference the *same* object in memory. This means `{ a: 1 } !== { a: 1 }` and `[1, 2] !== [1, 2]`, even though their contents are identical.

**Beginner-Friendly Explanation:** Imagine two identical twins. They look the same, but they are different people. `===` on objects is like asking "Are you the same person?"—identical twins answer "No" even though they look identical. `useMemo` and `useCallback` are how you make the *same* person appear across renders instead of a new twin each time. If you create a new object or function on every render, React sees a new "person" and re-renders unnecessarily.

### Purposes

- To understand why objects, arrays, and functions created during render cause re-renders.
- To recognise when memoisation is needed to preserve referential equality.
- To diagnose unnecessary re-renders caused by new references.
- To correctly interpret `useEffect` and `useMemo` dependency behaviour.
- To reason about `React.memo`'s shallow comparison.

### Syntax Rules and Structure

**Referential Equality Examples:**
```javascript
// Primitives: value equality
1 === 1;                    // true
'hello' === 'hello';        // true
true === true;              // true
Object.is(NaN, NaN);        // true (=== would be false)
Object.is(0, -0);           // false (=== would be true)

// Objects: referential equality
const a = { x: 1 };
const b = { x: 1 };
a === b;                    // false (different references)
const c = a;
a === c;                    // true (same reference)

// Arrays
[1, 2] === [1, 2];          // false
const arr = [1, 2];
const arr2 = arr;
arr === arr2;               // true

// Functions
(() => {}) === (() => {});  // false
const fn = () => {};
const fn2 = fn;
fn === fn2;                 // true
```

**How React Uses Referential Equality:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // ❌ New object every render — child re-renders even if count unchanged
  const config = { theme: 'dark' };

  // ❌ New function every render — child re-renders
  const handleClick = () => console.log('clicked');

  // ✅ Stable object — child re-renders only when count changes
  const stableConfig = useMemo(() => ({ theme: 'dark' }), []);

  // ✅ Stable function — child re-renders only when its dependencies change
  const stableClick = useCallback(() => console.log('clicked'), []);

  return <Child config={stableConfig} onClick={stableClick} />;
}
```

**Component Breakdown:**
- `config = { theme: 'dark' }`: Creates a new object on every render; `React.memo` on `Child` sees a new prop and re-renders.
- `useMemo(() => ({ theme: 'dark' }), [])`: Creates the object once and reuses it.
- `handleClick = () => {}`: New function on every render.
- `useCallback(() => {}, [])`: Stable function reference.

**Syntax Rules:**
- Objects, arrays, and functions created during render are new references every time.
- `useMemo` and `useCallback` are the standard tools to stabilise references across renders.
- `React.memo` uses shallow comparison: for each prop, it compares with `Object.is`.
- When passing object or function props to memoised children, stabilise them or the memo will not help.
- Primitive props (strings, numbers, booleans) do not need stabilisation because `Object.is` compares by value.

**Constraints and Limitations:**
- Deep equality is not used by React; only shallow (`Object.is`) comparison.
- `useMemo` with an empty dependency array creates a stable reference but captures the initial closure; it will not see updated values.
- Custom `React.memo` comparison functions can cause bugs and performance issues if they are not carefully written.
- Referential equality is a JavaScript concept, not a React-specific one; it applies to all comparisons.

### Annotated Code Example: Diagnosing Re-renders with Referential Equality

```jsx
import { useState, useMemo, useCallback, memo, useRef } from 'react';

const Child = memo(function Child({ config, onClick }) {
  const renderCount = useRef(0);
  renderCount.current += 1;
  return (
    <div>
      <p>Child renders: {renderCount.current}</p>
      <p>Theme: {config.theme}</p>
      <button onClick={onClick}>Click</button>
    </div>
  );
});

export default function Parent() {
  const [count, setCount] = useState(0);

  // ❌ Unstable: new object every render
  const unstableConfig = { theme: 'dark' };

  // ✅ Stable: memoised object
  const stableConfig = useMemo(() => ({ theme: 'dark' }), []);

  // ❌ Unstable: new function every render
  const unstableClick = () => console.log('clicked');

  // ✅ Stable: memoised function
  const stableClick = useCallback(() => console.log('clicked'), []);

  return (
    <div>
      <p>Parent count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>

      <h3>Unstable Props</h3>
      <Child config={unstableConfig} onClick={unstableClick} />

      <h3>Stable Props</h3>
      <Child config={stableConfig} onClick={stableClick} />
    </div>
  );
}
```

**Expected Output:** Clicking "Increment" re-renders the parent. The "Unstable Props" child re-renders every time (its render count increases), because `unstableConfig` and `unstableClick` are new references. The "Stable Props" child does not re-render (its count stays at 1), because `stableConfig` and `stableClick` are stable across renders.

**Why This Output Occurs:** `React.memo` compares each prop with `Object.is`. For the unstable child, `config` and `onClick` are new references every render, so the comparison fails and the child re-renders. For the stable child, both props are the same references as the previous render, so `React.memo` skips the re-render.

### Real-World Cases

- **Prop-drilling optimisation:** Stabilising callbacks passed through multiple component layers.
- **Context values:** Memoising context provider values to prevent consumer re-renders.
- **Event handlers in lists:** Memoising row handlers to prevent every row from re-rendering.
- **Chart configuration:** Memoising config objects passed to chart components.
- **Third-party components:** Stabilising props passed to heavy third-party components (maps, editors).

### References

- MDN Web Docs – Equality comparisons and sameness: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness
- MDN Web Docs – `Object.is()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo

---

## Core Concept 3: `useMemo`

### Definitions

**Core Definition:** `useMemo` is a React Hook that caches the result of a computation between renders, recomputing it only when one of its dependencies changes.

**Technical Definition:** `useMemo(calculateValue, dependencies)` is a React Hook that memoises the value returned by `calculateValue`, re-executing it only when `dependencies` change (as determined by `Object.is`). It is called at the top level of a component and returns the memoised value. On the first render, `useMemo` calls the function and caches the result. On subsequent renders, if no dependency has changed, it returns the cached value without calling the function. If any dependency changes, it calls the function again and updates the cache. `useMemo` is a performance optimisation, not a semantic guarantee: React may discard memoised values in the future (the React Compiler now automates this).

**Beginner-Friendly Explanation:** `useMemo` is like a calculator with a memory button. You work out an answer once, and if you need it again and nothing has changed, you press the memory button instead of recalculating. It is useful when the calculation is expensive—like sorting a large list or running a complex transformation. If the calculation is trivial (like `a + b`), using `useMemo` is overkill.

### Purposes

- To avoid recomputing expensive values on every render.
- To preserve referential equality of objects and arrays passed as props to memoised children.
- To stabilise dependencies for `useEffect` and `useCallback`.
- To prevent infinite loops in `useEffect` caused by new object references.
- To skip expensive calculations when dependencies have not changed.

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
  () => items.filter((i) => i.name.includes(query)).sort(byName),
  [items, query]
);

// Memoise an object passed as a prop
const config = useMemo(() => ({ theme, locale }), [theme, locale]);

// Memoise an expensive derivation
const stats = useMemo(() => computeStatistics(data), [data]);

// Memoise a derived primitive (though this is rarely worth it)
const total = useMemo(() => items.reduce((sum, i) => sum + i.price, 0), [items]);
```

**Syntax Rules:**
- Call `useMemo` at the top level of a component or Hook; never inside loops, conditions, or nested functions.
- The calculation function must be pure; it must not mutate anything or cause side effects.
- Provide a dependency array; without it, `useMemo` recomputes on every render (pointless).
- The dependency array should include every reactive value used inside the calculation.
- For expensive computations, `useMemo` is worthwhile; for trivial ones, it adds overhead.
- `useMemo` is a performance optimisation, not a semantic guarantee; React may discard the cache.

**Constraints and Limitations:**
- `useMemo` adds memory overhead and a comparison per render; overuse degrades performance.
- The calculation function runs during render; it must not cause side effects.
- Memoisation is per-component-instance; it does not share across instances.
- If a dependency is an object or array created during render, the memo will never be reused (new reference every render).
- `useMemo` does not prevent the initial render or the recomputation when dependencies genuinely change.
- The React Compiler may make `useMemo` unnecessary in many cases.

### Annotated Code Example: Memoising an Expensive Computation

```jsx
import { useState, useMemo } from 'react';

// Simulated expensive computation
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

**Expected Output:** Changing the limit recomputes the primes ("Computing primes up to N" logs). Toggling dark mode does *not* recompute the primes, because `limit` has not changed; `useMemo` returns the cached value. Without `useMemo`, toggling dark mode would recompute the primes unnecessarily.

**Why This Output Occurs:** `useMemo(() => computePrimes(limit), [limit])` caches the result keyed by `limit`. When `darkMode` changes, the component re-renders, but `limit` is unchanged, so `useMemo` returns the cached primes without re-running the calculation.

### Real-World Cases

- **Data grids:** Memoising filtered, sorted, and paginated rows.
- **Charts:** Memoising computed data points and scales.
- **Search:** Memoising filtered search results.
- **Statistics:** Memoising aggregates and derived metrics.
- **Configuration objects:** Memoising config objects passed to third-party components.

### References

- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – Skipping Expensive Calculations: https://react.dev/reference/react/useMemo#skipping-expensive-recalculations
- React Official Documentation – Memoizing a Dependency of Another Hook: https://react.dev/reference/react/useMemo#memoizing-a-dependency-of-another-hook
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler

---

## Core Concept 4: `useCallback`

### Definitions

**Core Definition:** `useCallback` is a React Hook that returns a memoised version of a function, preserving its referential identity across renders unless its dependencies change.

**Technical Definition:** `useCallback(fn, dependencies)` is a React Hook that memoises the function `fn` and returns the same function reference across renders as long as `dependencies` remain unchanged (compared with `Object.is`). It is equivalent to `useMemo(() => fn, dependencies)`. The returned function has the same body but a stable reference, which matters when passing callbacks to memoised children (`React.memo`) or when using them in dependency arrays of other hooks (`useEffect`, `useMemo`, `useCallback`). `useCallback` is a performance optimisation, not a semantic guarantee.

**Beginner-Friendly Explanation:** `useCallback` is like saving a phone number in your contacts. Every time you call the number, you dial the same saved entry—not a fresh copy of the number. If the person changes their number (a dependency changes), you update the contact. Without `useCallback`, every render creates a new "phone number" (function), and your memoised children think they have been given a different contact—so they re-render.

### Purposes

- To preserve referential equality of event handlers passed to memoised children.
- To stabilise callbacks used as dependencies in `useEffect` or `useMemo`.
- To prevent infinite loops caused by new function references in dependency arrays.
- To avoid re-subscribing to event listeners or re-running effects unnecessarily.
- To improve performance in components with many child components.

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
```

**Syntax Rules:**
- Call `useCallback` at the top level of a component or Hook; never inside loops or conditions.
- The function must be pure with respect to React state; mutations should be done via setters or effects.
- Provide a dependency array that includes all reactive values used inside the function.
- An empty dependency array creates a stable function that captures the initial render's values—this causes stale closures if the function reads state.
- `useCallback` is unnecessary for functions passed to non-memoised children (they re-render regardless).
- Use the functional form of state setters (`setX((prev) => ...)`) to avoid depending on `x`.

**Constraints and Limitations:**
- `useCallback` does not prevent re-renders by itself; it only helps when the child is memoised with `React.memo`.
- If the callback depends on frequently changing values, it is recreated frequently, negating the benefit.
- The callback body runs at call time, not at memoisation time; only the reference is memoised.
- Overusing `useCallback` adds complexity without measurable benefit.
- The React Compiler may insert `useCallback` automatically.

### Annotated Code Example: Stable Callback for a Memoised Child

```jsx
import { useState, useCallback, memo } from 'react';

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

**Expected Output:** Typing in the filter box re-renders the parent and the visible rows, but the `handleDelete` reference stays stable. When deleting an item, only the affected rows re-render. Without `useCallback`, `handleDelete` would be a new function on every render, causing every row to re-render on every keystroke.

**Why This Output Occurs:** `useCallback((id) => ..., [])` returns the same function reference forever. `React.memo` on `MemoisedRow` compares its props with `Object.is`. Since `onDelete` is stable and `item` is stable (from the `items` array), each row re-renders only when its own `item` reference changes (i.e., when the list is filtered or an item is deleted).

### Real-World Cases

- **Lists with per-row actions:** Delete, edit, toggle handlers passed to memoised rows.
- **Event listeners in effects:** Stable listeners for resize, scroll, or online/offline events.
- **Context callbacks:** Stable dispatch functions or action creators in context values.
- **Form field handlers:** Stable change handlers passed to memoised input components.
- **Third-party integrations:** Stable callbacks passed to chart, map, or editor libraries.

### References

- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – `useCallback` vs `useMemo`: https://react.dev/reference/react/useCallback#usecallback-vs-usememo
- React Official Documentation – Preventing an Effect from Re-connecting: https://react.dev/reference/react/useCallback#preventing-an-effect-from-re-connecting
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler

---

## Core Concept 5: Dependency Management

### Definitions

**Core Definition:** Dependency management is the practice of correctly declaring the reactive values that a Hook depends on in its dependency array, ensuring the Hook re-runs when—and only when—it should.

**Technical Definition:** Every React Hook that accepts a dependency array (`useMemo`, `useCallback`, `useEffect`, `useLayoutEffect`, `useImperativeHandle`) uses that array to determine when to invalidate its memoisation or re-run its effect. React compares each dependency with `Object.is`; if any dependency has changed since the previous render, the Hook is invalidated. The dependency array is a *contract*: it must include every reactive value (props, state, context, or values derived from them) that is read inside the Hook. Omitting a dependency causes a stale closure; including an unstable dependency causes the Hook to re-run on every render. The `eslint-plugin-react-hooks` `exhaustive-deps` rule statically verifies that the array is correct.

**Beginner-Friendly Explanation:** Dependency management is like telling React: "Only redo this work if these specific things have changed." If you forget to list something the Hook uses, React will keep using the old value—a bug called a stale closure. If you list something that changes on every render, React will redo the work every render—defeating the purpose of memoisation. The ESLint rule `exhaustive-deps` is your safety net: it tells you when you have missed or misused a dependency.

### Purposes

- To ensure Hooks re-run exactly when their inputs change, no more and no less.
- To prevent stale closures where the Hook reads outdated props or state.
- To prevent infinite loops caused by unstable dependencies.
- To make dependencies explicit and auditable.
- To satisfy the `exhaustive-deps` ESLint rule and avoid subtle bugs.

### Syntax Rules and Structure

**Correct Dependency Arrays:**
```jsx
// ✅ Correct: all reactive values listed
const filtered = useMemo(
  () => items.filter((i) => i.name.includes(query)),
  [items, query]
);

// ✅ Correct: no reactive values used → empty array
const config = useMemo(() => ({ theme: 'dark' }), []);

// ✅ Correct: all values used in the callback listed
const handleSearch = useCallback(
  (q) => search(q, category, limit),
  [category, limit]
);
```

**Incorrect Dependency Arrays:**
```jsx
// ❌ Missing dependency: `query` is used but not listed
const filtered = useMemo(
  () => items.filter((i) => i.name.includes(query)),
  [items] // Stale: `query` will always be the initial value
);

// ❌ Unstable dependency: new object every render
const config = useMemo(() => ({ theme }), [{ theme }]);
// `{ theme }` is a new object every render → memo never reuses

// ❌ Missing dependency in useCallback
const handleSearch = useCallback(
  (q) => search(q, category), // `category` used but not listed
  []
);
```

**Fixing Common Dependency Problems:**

**Problem 1: Object/array dependency created during render**
```jsx
// ❌ New object every render
const options = { sort: 'name', filter };
const sorted = useMemo(() => sort(items, options), [items, options]);

// ✅ List the primitive properties instead
const sorted = useMemo(
  () => sort(items, { sort: 'name', filter }),
  [items, filter]
);

// ✅ Or memoise the object
const options = useMemo(() => ({ sort: 'name', filter }), [filter]);
const sorted = useMemo(() => sort(items, options), [items, options]);
```

**Problem 2: Function dependency recreated every render**
```jsx
// ❌ `transform` is recreated every render
function Component({ items }) {
  const transform = (item) => item.name.toUpperCase();
  const names = useMemo(() => items.map(transform), [items, transform]);
}

// ✅ Move the function outside the component
function transform(item) { return item.name.toUpperCase(); }
function Component({ items }) {
  const names = useMemo(() => items.map(transform), [items]);
}

// ✅ Or wrap in useCallback
const transform = useCallback((item) => item.name.toUpperCase(), []);
```

**Problem 3: Functional state updates avoid dependencies**
```jsx
// ❌ Depends on `count`
const increment = useCallback(() => setCount(count + 1), [count]);

// ✅ Functional update avoids the dependency
const increment = useCallback(() => setCount((c) => c + 1), []);
```

**Syntax Rules:**
- Include every reactive value read inside the Hook in the dependency array.
- Use `eslint-plugin-react-hooks`'s `exhaustive-deps` rule to catch errors.
- Do not lie to the linter by omitting dependencies; restructure the code instead.
- Use functional state updates (`setX((prev) => ...)`) to avoid depending on state.
- Move functions and objects outside the component when they do not depend on props/state.
- Memoise objects and functions that must be dependencies of other Hooks.
- When in doubt, include the dependency; the Hook will be re-run less often than you fear.

**Constraints and Limitations:**
- Dependency arrays are compared with `Object.is`; deep equality is not used.
- The `exhaustive-deps` rule is a lint rule, not a type check; it can be disabled but doing so is discouraged.
- Adding a missing dependency can cause an infinite loop if the dependency is unstable; fix the instability instead.
- The React Compiler may handle dependency management automatically in the future.
- Some patterns (e.g., `useEffect` with `[]` to run once) rely on an empty array; ensure the Effect truly has no reactive dependencies.

### Annotated Code Example: Fixing a Stale Closure

```jsx
import { useState, useEffect, useCallback, useMemo } from 'react';

export default function SearchBox() {
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');
  const [results, setResults] = useState([]);

  // ❌ Before: `category` missing → stale closure
  // const performSearch = useCallback((q) => {
  //   fetchResults(q, category).then(setResults);
  // }, []); // `category` is stale

  // ✅ After: `category` included → correct behaviour
  const performSearch = useCallback((q) => {
    fetchResults(q, category).then(setResults);
  }, [category]);

  // ❌ Before: new options object every render → effect runs every render
  // const options = { query, category };
  // useEffect(() => { performSearch(query); }, [options, performSearch]);

  // ✅ After: primitives listed directly → effect runs only when they change
  useEffect(() => {
    performSearch(query);
  }, [query, performSearch]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <select value={category} onChange={(e) => setCategory(e.target.value)}>
        <option value="all">All</option>
        <option value="books">Books</option>
        <option value="movies">Movies</option>
      </select>
      <ul>{results.map((r) => <li key={r.id}>{r.title}</li>)}</ul>
    </div>
  );
}
```

**Expected Output:** Typing in the input triggers a search. Changing the category triggers a new search with the updated category. The effect runs only when `query` or `performSearch` changes; since `performSearch` changes only when `category` changes, the effect runs when either the query or the category changes—exactly as intended.

**Why This Output Occurs:** `performSearch` includes `category` in its dependency array, so it is recreated when `category` changes. The `useEffect` depends on `performSearch` and `query`, so it re-runs when either changes. This avoids both the stale closure (missing `category`) and the infinite loop (unstable `options` object).

### Real-World Cases

- **Data fetching:** Fetching data when query parameters change.
- **Event listeners:** Attaching and detaching listeners when the handler changes.
- **Animations:** Restarting animations when their configuration changes.
- **Subscriptions:** Re-subscribing to WebSocket topics when the topic changes.
- **Analytics:** Logging events when the route or user changes.

### References

- React Official Documentation – Removing Effect Dependencies: https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – `useMemo` Dependencies: https://react.dev/reference/react/useMemo#parameters
- React Official Documentation – `useCallback` Dependencies: https://react.dev/reference/react/useCallback#parameters
- React Official Documentation – `exhaustive-deps` ESLint Rule: https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- GitHub – `eslint-plugin-react-hooks`: https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks

---

## Core Concept 6: Avoiding Premature Optimisation

### Definitions

**Core Definition:** Avoiding premature optimisation is the practice of not applying performance optimisations (such as `useMemo` and `useCallback`) until profiling has identified a real, measurable performance problem.

**Technical Definition:** Premature optimisation refers to adding performance optimisations before measuring whether they are needed. In React, this manifests as wrapping every value in `useMemo` and every function in `useCallback` "just in case." This is counterproductive because (1) memoisation costs memory and a comparison per render; (2) it adds cognitive overhead and code complexity; (3) it can hide bugs and make dependencies harder to reason about; and (4) React's reconciliation is often fast enough without it. The React documentation states: "You should only rely on `useMemo` as a performance optimization. If your code doesn't work without it, find the underlying problem and fix it first. Then you may add `useMemo` to improve performance." Donald Knuth's famous adage applies: "Premature optimization is the root of all evil."

**Beginner-Friendly Explanation:** Premature optimisation is like putting a turbocharger on a bicycle before you have learned to ride it. You are adding complexity and cost for a speed improvement you may not even need. Instead, ride the bike first, see if it is too slow, and then decide whether a turbocharger (or a different bike) is the right fix. In React, write the simplest code first, profile it, and only add `useMemo` or `useCallback` when the profiler shows a real problem.

### Purposes

- To avoid adding unnecessary complexity to the codebase.
- To prevent memory and CPU overhead from excessive memoisation.
- To keep the code readable and maintainable.
- To focus optimisation efforts on actual bottlenecks identified by profiling.
- To allow the React Compiler to handle memoisation automatically where appropriate.

### Syntax Rules and Structure

**When to Memoise (Checklist):**

| Signal | Action |
|---|---|
| The value is used as a dependency of `useEffect` / `useMemo` / `useCallback` | Memoise it |
| The value is passed as a prop to a `React.memo`-wrapped component | Memoise it |
| The calculation is genuinely expensive (profiler shows > 1ms) | Memoise it |
| The value is a context provider value | Memoise it |
| The calculation is trivial (`a + b`, `arr.length`) | Do **not** memoise |
| The value is passed to a non-memoised child | Do **not** memoise |
| The value is a primitive | Do **not** memoise |
| You are "not sure if it helps" | Do **not** memoise; profile first |

**Profiling Workflow:**
```
1. Build the feature with the simplest code (no memoisation).
2. Open React DevTools Profiler.
3. Record a session while interacting with the feature.
4. Identify components with high render times or frequent re-renders.
5. Apply targeted memoisation (React.memo, useMemo, useCallback) to those components.
6. Re-profile to confirm the improvement.
7. Remove any memoisation that did not measurably help.
```

**Syntax Rules:**
- Write the simplest code first; optimise only after measuring.
- Use React DevTools Profiler to identify actual bottlenecks.
- Do not memoise primitives (`const total = a + b` does not need `useMemo`).
- Do not memoise values passed to non-memoised children.
- Do not memoise trivial calculations; the comparison overhead may exceed the calculation cost.
- Remove memoisation that does not measurably improve performance.
- Consider enabling the React Compiler to automate memoisation.

**Constraints and Limitations:**
- Profiling in development mode includes `StrictMode` double-renders, which can skew measurements; profile in production mode for accurate results.
- Memoisation benefits are more visible in large, complex components than in small ones.
- The React Compiler (React 19+) can automatically memoise, reducing the need for manual `useMemo` and `useCallback`.
- Premature optimisation can mask design problems (e.g., a component doing too much).
- Memoisation does not fix architectural problems like prop drilling or over-rendering due to context.

### Annotated Code Example: Before and After Profiling

```jsx
// ❌ Premature optimisation: everything memoised
function OverMemoised({ items, filter, onSelect }) {
  const filtered = useMemo(
    () => items.filter((i) => i.name.includes(filter)),
    [items, filter]
  );

  const count = useMemo(() => filtered.length, [filtered]); // Trivial
  const title = useMemo(() => `Found ${count} items`, [count]); // Trivial
  const handleSelect = useCallback((item) => onSelect(item), [onSelect]);

  return (
    <div>
      <h2>{title}</h2>
      <ul>
        {filtered.map((item) => (
          <li key={item.id} onClick={() => handleSelect(item)}>
            {item.name}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```jsx
// ✅ After profiling: only the expensive filter is memoised
function Optimised({ items, filter, onSelect }) {
  // Genuinely expensive: filtering a large list
  const filtered = useMemo(
    () => items.filter((i) => i.name.includes(filter)),
    [items, filter]
  );

  // Trivial calculations: no memoisation needed
  const count = filtered.length;
  const title = `Found ${count} items`;

  // Only memoise if the parent is memoised or if it stabilises a dependency
  const handleSelect = useCallback((item) => onSelect(item), [onSelect]);

  return (
    <div>
      <h2>{title}</h2>
      <ul>
        {filtered.map((item) => (
          <li key={item.id} onClick={() => handleSelect(item)}>
            {item.name}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** Both versions render the same UI. The over-memoised version adds three unnecessary `useMemo` calls (`count`, `title`) and one `useCallback` that may not help. The optimised version keeps only the memoisation that addresses a real cost (filtering) and the callback stabilisation that supports a memoised child.

**Why This Output Occurs:** `count` and `title` are trivial calculations (`filtered.length` and string interpolation). Memoising them adds comparison overhead without measurable benefit. `filtered` is genuinely expensive for large lists and is used as a dependency elsewhere, so it is worth memoising. `handleSelect` is memoised only because it is passed to a (potentially) memoised child. This targeted approach follows the profiling-first rule.

### Real-World Cases

- **Large data tables:** Profiling reveals that filtering and sorting on every render is the bottleneck; memoise those.
- **Chart components:** Profiling shows that recomputing chart data is expensive; memoise it.
- **Lists with many items:** Profiling shows that all rows re-render on every parent render; add `React.memo` and stabilise callbacks.
- **Context providers:** Profiling shows that all consumers re-render on every provider render; memoise the context value.
- **Small components:** Profiling shows no measurable benefit from memoisation; remove it.

### References

- React Official Documentation – `useMemo` (Performance Optimisation): https://react.dev/reference/react/useMemo
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – Profiler: https://react.dev/reference/react/Profiler
- Donald Knuth – Structured Programming with go to Statements: https://dl.acm.org/doi/10.1145/356635.356640
- Kent C. Dodds – When to useMemo and useCallback: https://kentcdodds.com/blog/usememo-and-usecallback

---

## Comparison and Decision Guidance

| Hook/API | What It Memoises | When to Use | When NOT to Use |
|---|---|---|---|
| **`React.memo`** | Component render output | Component re-renders often with same props; expensive to render | Component's props change every render; trivial component |
| **`useMemo`** | Computed value | Expensive calculation; dependency of another Hook; context value | Trivial calculation; primitive; passed to non-memoised child |
| **`useCallback`** | Function reference | Callback passed to memoised child; dependency of `useEffect` | Callback passed to non-memoised child; trivial event handler |
| **Dependency array** | When to invalidate | Always; include all reactive values | Never omit; never lie to the linter |
| **Profiling** | Where the bottleneck is | Before optimising | Never skip; premature optimisation is an anti-pattern |

**Decision Guidance:**
- **Write the simplest code first** without memoisation.
- **Profile with React DevTools** to find real bottlenecks.
- **Memoise values that are dependencies** of other Hooks (to prevent infinite loops).
- **Memoise callbacks passed to memoised children** (`React.memo`).
- **Memoise genuinely expensive calculations** (>1ms in the profiler).
- **Do not memoise primitives, trivial calculations, or props of non-memoised children.**
- **Use functional state updates** (`setX((prev) => ...)`) to avoid dependencies.
- **Move functions and constants outside the component** when they do not depend on props/state.
- **Enable the React Compiler** if available; it automates much of this.
- **Satisfy the `exhaustive-deps` ESLint rule**; do not disable it.
- **Remove memoisation that does not measurably help.**

---

## References

- React Official Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Official Documentation – `memo`: https://react.dev/reference/react/memo
- React Official Documentation – Removing Effect Dependencies: https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – `exhaustive-deps` ESLint Rule: https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React Official Documentation – React Compiler: https://react.dev/learn/react-compiler
- React Official Documentation – Profiler: https://react.dev/reference/react/Profiler
- MDN Web Docs – `Object.is()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- MDN Web Docs – Equality comparisons and sameness: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness
- MDN Web Docs – Memoization: https://developer.mozilla.org/en-US/docs/Glossary/Memoization
- GitHub – `eslint-plugin-react-hooks`: https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks
- Kent C. Dodds – When to useMemo and useCallback: https://kentcdodds.com/blog/usememo-and-usecallback
- Donald Knuth – Structured Programming with go to Statements: https://dl.acm.org/doi/10.1145/356635.356640