# `useMemo` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useMemo` is a React Hook that lets you cache the result of a calculation between re-renders.

**Technical Definition:** `const cachedValue = useMemo(calculateValue, dependencies)` is a React Hook that accepts a `calculateValue` function and a `dependencies` array, and returns a memoized value. React calls `calculateValue` during the initial render. On subsequent renders, React returns the same cached value if none of the dependencies have changed since the last render; otherwise, it calls `calculateValue` again, stores the new result, and returns it. React compares each dependency with its previous value using `Object.is` comparison. `useMemo` is a performance optimization tool, not a semantic guarantee — your code should function correctly without it.

**Beginner-Friendly Explanation:** `useMemo` is like a smart calculator that remembers its last answer. If you ask the same question (same inputs), it gives you the stored answer instead of recalculating. This is useful when the calculation is slow or when you need to keep an object or array "the same" to prevent child components from re-rendering unnecessarily. Think of it as a sticky note that React checks before doing expensive work.

### Key Characteristics

- **Value memoization:** Caches the result of a calculation, not a function (use `useCallback` for functions).
- **Dependency-driven:** Recomputes only when a dependency changes, compared with `Object.is`.
- **Referential stability:** Returns the same object/array reference across renders when dependencies are unchanged.
- **Performance hint, not a guarantee:** React may discard the cache in certain situations (e.g., component suspends, development file edits).
- **Top-level only:** Must be called at the top level of a component or custom Hook.
- **Compiler-aware:** React 19's compiler can automatically memoize values, reducing the need for manual `useMemo` calls.

### Prerequisites

- JavaScript functions, closures, and the event loop.
- React function components, JSX, and re-rendering.
- Understanding of referential equality and `Object.is`.
- Familiarity with `useState` and `useEffect`.

### Related Programming Areas

- **`useCallback`** — memoizes function references, not values.
- **`React.memo`** — memoizes component renders based on props.
- **`useEffect`** — dependency arrays follow the same comparison rules.
- **React Compiler** — automatically handles memoization in React 19+.

---

## 1. `useMemo`

### Definitions

**Core Definition:** `useMemo` is a React Hook that caches the result of a calculation between re-renders.

**Technical Definition:** `useMemo(calculateValue, dependencies)` returns a memoized value. The `calculateValue` function should be pure, take no arguments, and return a value of any type. The `dependencies` array lists all reactive values referenced inside `calculateValue`. If the dependencies have not changed since the last render (compared with `Object.is`), React returns the cached value. Otherwise, it calls `calculateValue` again and caches the new result.

**Beginner-Friendly Explanation:** `useMemo` is a way to tell React: "Remember this calculated value. Only recalculate it if these specific things change."

### Purposes

- To skip expensive recalculations on every render.
- To maintain referential equality of objects and arrays passed as props to memoized children.
- To stabilise dependencies for other Hooks like `useEffect`.
- To prevent unnecessary re-renders in large component trees.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useMemo } from 'react';

const cachedValue = useMemo(calculateValue, dependencies);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `calculateValue` | Pure function with no arguments that returns the value to cache. |
| `dependencies` | Array of reactive values referenced inside `calculateValue`. |
| `cachedValue` | The memoized result. |

**Syntax Rules:**

1. Call `useMemo` at the top level of a component or custom Hook.
2. The dependency array must have a constant number of items and be written inline like `[dep1, dep2, dep3]`.
3. `calculateValue` must be pure and return a value.
4. React compares dependencies with `Object.is`.

**Constraints and Limitations:**

- In Strict Mode, React calls the calculation function twice in development to detect impurities.
- React may throw away the cache if the component suspends during initial mount.
- `useMemo` is not a semantic guarantee — do not rely on it for correctness.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Expensive Filtering and Sorting

```jsx
import { useMemo, useState } from 'react';

function ProductList({ products, query }) {
  const [theme, setTheme] = useState('light');

  // Step 1: Memoize the filtered and sorted list
  const visibleProducts = useMemo(() => {
    console.log('Filtering and sorting...');
    return products
      .filter((p) => p.name.toLowerCase().includes(query.toLowerCase()))
      .sort((a, b) => a.name.localeCompare(b.name));
  }, [products, query]);

  return (
    <div>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <ul>
        {visibleProducts.map((p) => (
          <li key={p.id}>{p.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** Toggling the theme does not re-run the filter/sort. The console only logs "Filtering and sorting..." when `products` or `query` changes.

**Why this output:** Without `useMemo`, the filter and sort operations would run on every render, including theme toggles that have nothing to do with the product data. `useMemo` caches the result until `products` or `query` changes.

#### Example 2: Stabilising Object Props for Memoized Children

```jsx
import { useMemo, memo, useState } from 'react';

const ExpensiveChild = memo(function ExpensiveChild({ config }) {
  console.log('ExpensiveChild rendered');
  return <div>{config.theme}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);

  // ✅ Stable reference: same object until dependencies change
  const config = useMemo(() => {
    return { theme: 'dark', timeout: 5000 };
  }, []);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveChild config={config} />
    </div>
  );
}
```

**Expected Output:** Clicking the button does not re-render `ExpensiveChild`.

**Why this output:** Without `useMemo`, `config` would be a new object on every render, causing `React.memo` to detect a prop change and re-render the child. With `useMemo`, the reference stays stable, so `ExpensiveChild` skips re-rendering.

### Real-World Cases

- **Data grids:** Filtering, sorting, and paginating large datasets.
- **Charts:** Computing derived data points for visualisation.
- **Search:** Debounced search filtering on large lists.
- **Form validation:** Computing validation results from form state.

### References

- React — useMemo — https://ar.react.dev/reference/react/useMemo
- React — useMemo (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-memo

---

## 2. Memoization

### Definitions

**Core Definition:** Memoization is a technique that caches the result of a computation and reuses it when the inputs haven't changed.

**Technical Definition:** Memoization is an optimisation technique where a function's return value is cached based on its arguments. When the same arguments are passed again, the cached result is returned instead of recomputing. In React, `useMemo` applies this concept to the render cycle: it caches the result of `calculateValue` and only re-runs it when the dependencies change. Memoization trades memory (storing the cache) for computation time (avoiding redundant work).

**Beginner-Friendly Explanation:** Memoization is like writing down the answer to a maths problem so you don't have to solve it again.

### Purposes

- To avoid redundant expensive computations.
- To improve application performance.
- To maintain referential stability for downstream consumers.

### Syntax Rules and Structure

```jsx
const result = useMemo(() => expensiveOperation(a, b), [a, b]);
```

**Syntax Rules:**

1. Memoization only works when inputs are referentially stable.
2. The cache is invalidated when any dependency changes.
3. React may discard the cache in certain situations.

### Annotated Code Examples

```jsx
function DataAnalyzer({ dataset }) {
  // ✅ Good: expensive statistical calculation
  const statistics = useMemo(() => {
    const sorted = [...dataset].sort((a, b) => a - b);
    const mean = dataset.reduce((sum, n) => sum + n, 0) / dataset.length;
    const median = sorted[Math.floor(sorted.length / 2)];
    const standardDeviation = Math.sqrt(
      dataset.reduce((sum, n) => sum + Math.pow(n - mean, 2), 0) / dataset.length
    );
    return { mean, median, standardDeviation };
  }, [dataset]);

  return <div>{/* Render statistics */}</div>;
}
```

**Expected Output:** The statistics are computed only when `dataset` changes.

**Why this output:** The statistical calculation is computationally expensive. `useMemo` caches the result and only recalculates when `dataset` changes.

### Real-World Cases

- **Analytics dashboards:** Computing aggregates from raw data.
- **Large lists:** Filtering and sorting thousands of items.
- **Fuzzy search:** Running expensive string matching algorithms.

### References

- React — useMemo — https://ar.react.dev/reference/react/useMemo
- DebugBear — Improve React Performance With useMemo And useCallback — https://www.debugbear.com/blog/react-usememo-usecallback

---

## 3. Dependency Array

### Definitions

**Core Definition:** The dependency array is the second argument to `useMemo` that lists all reactive values the calculation reads.

**Technical Definition:** The `dependencies` array must include every reactive value referenced inside `calculateValue`. Reactive values include props, state, and all variables and functions declared directly inside the component body. The array must have a constant number of items and be written inline like `[dep1, dep2, dep3]`. React compares each dependency with its previous value using `Object.is`. If any dependency changes, `useMemo` recomputes and caches a new value. If the array is omitted, `useMemo` recalculates on every render (defeating the purpose). If the array is empty `[]`, the value is calculated only once.

**Beginner-Friendly Explanation:** The dependency array tells `useMemo` which values to watch. If any of those values change, `useMemo` recalculates. If none change, it returns the cached value.

### Purposes

- To control when the memoized calculation re-runs.
- To prevent stale values by including all referenced reactive values.
- To optimise performance by running the calculation only when necessary.

### Syntax Rules and Structure

**Dependency Array Behaviours:**

| Array | Behaviour |
|-------|-----------|
| Omitted | Recalculates on every render. |
| Empty `[]` | Calculates only once on mount. |
| `[dep1, dep2]` | Recalculates when any dependency changes. |

**Syntax Rules:**

1. Include all reactive values referenced inside `calculateValue`.
2. The array must have a constant number of items and be written inline.
3. React uses `Object.is` for comparison.

**Constraints and Limitations:**

- Missing dependencies cause stale values.
- Including objects/functions that change every render causes recomputation on every render.
- The `exhaustive-deps` ESLint rule catches missing dependencies.

### Annotated Code Examples

```jsx
// ✅ Correct: all dependencies included
const sorted = useMemo(() => {
  return [...items].sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
}, [items, sortBy]);

// ❌ Wrong: missing dependency
const sorted = useMemo(() => {
  return [...items].sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
}, [items]); // Missing 'sortBy'
```

**Expected Output:** The correct version re-sorts when `sortBy` changes; the incorrect version uses the stale `sortBy`.

**Why this output:** The dependency array `[items, sortBy]` tells React to recompute when either changes. Without `sortBy`, the memoized value uses the initial `sortBy` value even after it changes.

### Real-World Cases

- **Sorted lists:** Re-sorting when the sort field changes.
- **Filtered data:** Re-filtering when the search query or filter criteria change.
- **Derived objects:** Recomputing configuration objects when their inputs change.

### References

- React — useMemo: Parameters — https://ar.react.dev/reference/react/useMemo#parameters
- React — exhaustive-deps — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 4. Expensive Computations

### Definitions

**Core Definition:** Expensive computations are operations that consume significant CPU time or memory, making them candidates for memoization.

**Technical Definition:** An expensive computation is any operation whose cost is high enough that running it on every render would noticeably degrade performance. Examples include sorting and filtering large arrays (thousands of items), complex mathematical or statistical calculations, deep object transformations, and expensive library calls (chart layout, syntax highlighting, fuzzy search). The cost of memoization (memory for the cache and comparison overhead) must be weighed against the cost of recomputation.

**Beginner-Friendly Explanation:** An expensive computation is one that takes a long time to run. `useMemo` is useful for these because it avoids running them unnecessarily.

### Purposes

- To identify when `useMemo` provides real performance value.
- To avoid wasting CPU cycles on redundant calculations.
- To keep the UI responsive with large datasets.

### Syntax Rules and Structure

**When `useMemo` helps:**

| Scenario | Example |
|----------|---------|
| Large array operations | Sorting/filtering 10,000+ items. |
| Complex calculations | Statistical analysis, matrix math. |
| Deep transformations | Normalising API data, building indexes. |
| Expensive library work | Chart layout, syntax highlighting. |

**Syntax Rules:**

1. Measure before memoizing — use React DevTools Profiler or `console.time`.
2. Memoize only when the computation is genuinely expensive.
3. The memoization overhead must be less than the computation cost.

### Annotated Code Examples

```jsx
function SalesDashboard({ transactions }) {
  // ✅ Expensive: aggregating thousands of transactions
  const monthlyTotals = useMemo(() => {
    const totals = {};
    for (const tx of transactions) {
      const month = tx.date.slice(0, 7);
      totals[month] = (totals[month] || 0) + tx.amount;
    }
    return Object.entries(totals).sort();
  }, [transactions]);

  return <Chart data={monthlyTotals} />;
}
```

**Expected Output:** The aggregation runs only when `transactions` changes.

**Why this output:** Iterating over thousands of transactions and building a summary object is expensive. `useMemo` caches the result and avoids re-aggregating on unrelated re-renders.

### Real-World Cases

- **Financial dashboards:** Aggregating transaction data.
- **E-commerce:** Filtering and sorting product catalogs.
- **Analytics:** Computing statistics from large datasets.
- **Search engines:** Fuzzy matching against large corpora.

### References

- Feature-Sliced Design — When to useMemo: A React Performance Guide — https://feature-sliced.design/vi/blog/react-usememo-optimization
- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/usememo-usecallback-in-react-19

---

## 5. Referential Equality

### Definitions

**Core Definition:** Referential equality means two values are the same object in memory, compared with `===` or `Object.is`.

**Technical Definition:** In JavaScript, objects and arrays are compared by reference, not by their contents. `{} !== {}` even if both objects have identical properties. React uses referential equality in several places: `React.memo` does a shallow comparison of props; `useEffect`, `useMemo`, and `useCallback` compare dependencies with `Object.is`. When you create a new object or array inside a component on every render, React sees it as "changed" even if the contents are identical. `useMemo` solves this by returning the same reference across renders when dependencies are unchanged.

**Beginner-Friendly Explanation:** Referential equality means two things are literally the same object. Even if two objects look identical, they are different if they were created separately. `useMemo` keeps the same object so React knows nothing has changed.

### Purposes

- To prevent unnecessary re-renders of memoized child components.
- To stabilise dependencies for other Hooks.
- To maintain identity for objects passed through context or props.

### Syntax Rules and Structure

**The Referential Equality Problem:**

```jsx
// ❌ New object every render — child re-renders even if contents are same
<MemoizedChild config={{ theme: 'dark' }} />

// ✅ Stable reference with useMemo
const config = useMemo(() => ({ theme: 'dark' }), []);
<MemoizedChild config={config} />
```

**Syntax Rules:**

1. Objects and arrays created inside the component body are new on every render.
2. `React.memo` uses shallow comparison — new references fail equality checks.
3. `useMemo` returns the same reference when dependencies are unchanged.

### Annotated Code Examples

```jsx
const ExpensiveChild = memo(function ExpensiveChild({ options }) {
  console.log('Child rendered');
  return <div>{options.sortBy}</div>;
});

function Parent({ items }) {
  const [count, setCount] = useState(0);

  // ✅ Stable reference: only changes when items changes
  const options = useMemo(() => ({
    sortBy: 'name',
    filterBy: 'all',
  }), [items]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveChild options={options} />
    </div>
  );
}
```

**Expected Output:** Clicking the button does not re-render `ExpensiveChild`.

**Why this output:** Without `useMemo`, `options` would be a new object on every render. `React.memo` would see a changed prop and re-render the child. With `useMemo`, the reference stays stable until `items` changes.

### Real-World Cases

- **Memoized lists:** Preventing re-renders when parent state changes.
- **Context values:** Stabilising context values to prevent consumer re-renders.
- **Effect dependencies:** Preventing effects from running on every render.

### References

- Steve Kinney — Identity Stability Props — https://stevekinney.com/courses/react-performance/identity-stability-props
- Oracle — Developing Open Storefront Framework Applications — https://docs.oracle.com

---

## 6. When to Use `useMemo`

### Definitions

**Core Definition:** `useMemo` should be used when you have a concrete performance problem that memoization solves.

**Technical Definition:** `useMemo` is appropriate in two scenarios: (1) skipping expensive recalculations of derived data (filtering, sorting, aggregating, parsing), and (2) keeping a stable reference (object/array) so memoized children can skip re-rendering. The payoff depends on the cost of the calculation and the stability of dependencies. If dependencies change frequently, the cache is invalidated often and the benefit is reduced.

**Beginner-Friendly Explanation:** Use `useMemo` when you have a slow calculation that doesn't need to run every time, or when you need to keep an object "the same" to prevent child components from re-rendering.

### Purposes

- To skip expensive recalculations of derived data.
- To maintain stable references for memoized children.
- To stabilise dependencies for other Hooks.

### Syntax Rules and Structure

**When to use:**

| Scenario | Example |
|----------|---------|
| Expensive calculations | Sorting/filtering large arrays. |
| Complex transformations | Data normalisation, building indexes. |
| Stable props for memoized children | Object/array props passed to `React.memo` components. |
| Stable dependencies for effects | Object/array dependencies for `useEffect`. |

**Syntax Rules:**

1. Profile first — use React DevTools to confirm a performance problem.
2. Use `useMemo` only when the computation is expensive or reference stability is required.
3. Measure the impact before and after.

### Annotated Code Examples

```jsx
// ✅ Good: expensive computation
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}, [items]);

// ✅ Good: stable reference for memoized child
const config = useMemo(() => ({ theme, timeout: 5000 }), [theme]);
<MemoizedChild config={config} />
```

**Expected Output:** The computation only runs when dependencies change; the child only re-renders when `theme` changes.

**Why this output:** Both scenarios involve either expensive work or referential stability requirements that `useMemo` directly addresses.

### Real-World Cases

- **Large data tables:** Sorting and filtering thousands of rows.
- **Design systems:** Stable configuration objects for memoized components.
- **Dashboards:** Aggregating data for charts.

### References

- Feature-Sliced Design — When to useMemo: A React Performance Guide — https://feature-sliced.design/vi/blog/react-usememo-optimization
- CoreUI — How to Use useMemo in React — https://coreui.io/answers/how-to-use-usememo-in-react/

---

## 7. When Not to Use `useMemo`

### Definitions

**Core Definition:** `useMemo` should not be used for simple computations, primitive values, or as a precautionary optimisation.

**Technical Definition:** Memoization is not free — it requires memory to store the cached value and comparison logic to check dependencies on every render. For simple computations (string concatenation, arithmetic, boolean checks), the overhead of memoization exceeds the cost of recomputation. `useMemo` should not be used for side effects (use `useEffect` instead), for values that are primitives (strings, numbers, booleans), or as a precaution without measuring a performance problem.

**Beginner-Friendly Explanation:** Don't use `useMemo` for everything. If the calculation is fast, memoizing it actually makes things slower because React has to store and compare the value.

### Purposes

- To avoid the memoization tax (memory + comparison overhead).
- To keep code simple and readable.
- To prevent over-engineering.

### Common Over-Memoization Patterns

| Pattern | Why It's Bad |
|---------|-------------|
| Memoizing primitives | `useMemo(() => count % 2 === 0)` — cheap to compute |
| Memoizing string concatenation | Fast operation; overhead outweighs benefit |
| Memoizing simple objects | `useMemo(() => ({ a: 1 }), [])` — not worth the overhead |
| Memoizing functions | Use `useCallback` instead |
| Precautionary memoization | Without measuring a problem |

**Syntax Rules:**

1. If the computation is simple (under ~1ms), skip `useMemo`.
2. If the value is a primitive, skip `useMemo`.
3. If you haven't measured a performance problem, skip `useMemo`.

### Annotated Code Examples

```jsx
// ❌ Unnecessary: string concatenation is fast
function UserProfile({ user }) {
  const displayName = useMemo(() => {
    return `${user.firstName} ${user.lastName}`;
  }, [user.firstName, user.lastName]);
  return <h1>{displayName}</h1>;
}

// ✅ Just let it render
function UserProfile({ user }) {
  const displayName = `${user.firstName} ${user.lastName}`;
  return <h1>{displayName}</h1>;
}
```

**Expected Output:** Both versions render the same output, but the second is simpler and faster.

**Why this output:** String concatenation is extremely fast. The overhead of storing the cache and comparing dependencies outweighs any benefit.

### Real-World Cases

- **Simple derived values:** Full names, formatted numbers, boolean flags.
- **Small lists:** Filtering fewer than 100 items.
- **Static values:** Configuration objects that never change.

### References

- Steve Kinney — Avoiding Over‑Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- FreeCodeCamp — How to Avoid Overusing useCallback and useMemo in React — https://www.freecodecamp.org

---

## 8. `useMemo` Pitfalls

### Definitions

**Core Definition:** `useMemo` pitfalls are common mistakes that make memoization ineffective, harmful, or incorrect.

**Technical Definition:** Common pitfalls include: using `useMemo` for side effects, missing dependencies (stale values), unnecessary memoization (over-memoization), memoizing functions instead of values, relying on `useMemo` for semantic correctness, and violating the Rules of Hooks (conditional calls). Each of these undermines the purpose of `useMemo` and can lead to bugs or performance degradation.

**Beginner-Friendly Explanation:** These are the traps that make `useMemo` either useless or harmful.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Side effects in useMemo** | `useMemo` is for computing values, not running side effects. | Use `useEffect` or event handlers. |
| **No return value** | `useMemo` without return yields `undefined`. | Ensure `calculateValue` returns a value. |
| **Missing dependencies** | Stale memoized value. | Include all reactive values. |
| **Over-memoization** | Memoization overhead exceeds benefit. | Only memoize expensive computations. |
| **Memoizing functions** | `useMemo` returns the function, not its result. | Use `useCallback`. |
| **Relying on useMemo for correctness** | React may discard the cache. | Use `useState`/`useRef` for values that must persist. |
| **Conditional useMemo** | Breaks Rules of Hooks. | Always call at the top level. |

### Annotated Code Examples

**Pitfall 1: Side Effect in `useMemo`**

```jsx
// ❌ Wrong: side effect in useMemo
useMemo(() => {
  analytics.track('UserViewed', { userId: user.id });
}, [user.id]); // No return value!

// ✅ Correct: side effect in useEffect
useEffect(() => {
  analytics.track('UserViewed', { userId: user.id });
}, [user.id]);
```

**Pitfall 2: Missing Dependency**

```jsx
// ❌ Wrong: stale sortBy
const sorted = useMemo(() => {
  return [...items].sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
}, [items]); // Missing 'sortBy'

// ✅ Correct
const sorted = useMemo(() => {
  return [...items].sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
}, [items, sortBy]);
```

**Expected Output:** The correct versions behave predictably; the incorrect versions cause bugs.

**Why this output:** Side effects in `useMemo` violate React's purity rules. Missing dependencies cause stale values.

### Real-World Cases

- **Debugging:** Identifying why memoized values are stale.
- **Code reviews:** Catching over-memoization and side effects.
- **Performance:** Avoiding the memoization tax on cheap operations.

### References

- React — use-memo lint rule — https://zh-hant.react.dev/reference/eslint-plugin-react-hooks/lints/use-memo
- Steve Kinney — Avoiding Over‑Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization

---

## 9. `useMemo` Best Practices

### Definitions

**Core Definition:** `useMemo` best practices are guidelines for using the Hook effectively, safely, and maintainably.

**Technical Definition:** Best practices include: profiling before memoizing, memoizing only expensive computations, including all dependencies, not relying on `useMemo` for correctness, using `useCallback` for functions, preferring compiler-driven memoization in React 19+, and keeping memoization logic simple and readable. These practices ensure that `useMemo` is used as a surgical optimisation tool rather than a blanket solution.

**Beginner-Friendly Explanation:** These are the "do's and don'ts" that experienced React developers follow to keep their memoization clean and effective.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Profile first** | Measure before memoizing. |
| **Memoize expensive computations** | Filtering, sorting, complex calculations. |
| **Include all dependencies** | Prevent stale values. |
| **Don't rely on useMemo for correctness** | It is a performance hint, not a guarantee. |
| **Use useCallback for functions** | `useMemo` memoizes values, not functions. |
| **Use React Compiler when available** | Automatic memoization in React 19+. |
| **Keep it simple** | If memoization adds complexity without clear benefit, remove it. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Memoize expensive computation with all dependencies
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
}, [items, sortBy]);

// Use useCallback for functions
const handleClick = useCallback(() => {
  setCount(c => c + 1);
}, []);
```

**❌ Bad Practices:**

```jsx
// ❌ Memoizing a primitive
const isEven = useMemo(() => count % 2 === 0, [count]);

// ❌ Missing dependency
const sorted = useMemo(() => sortBy(items, sortBy), [items]);

// ❌ Side effect in useMemo
useMemo(() => { logAnalytics(); }, [data]);
```

**Expected Output:** The good practices produce predictable, performant code.

**Why this output:** Memoizing primitives adds overhead without benefit. Missing dependencies cause stale values. Side effects in `useMemo` violate purity rules.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Code reviews:** Best practices are enforced through ESLint and review checklists.
- **React 19 migration:** Moving from manual to compiler-driven memoization.

### References

- Feature-Sliced Design — When to useMemo: A React Performance Guide — https://feature-sliced.design/vi/blog/react-usememo-optimization
- CoreUI — How to Use useMemo in React — https://coreui.io/answers/how-to-use-usememo-in-react/
- React — useMemo — https://ar.react.dev/reference/react/useMemo

---

## References

- React — useMemo — https://ar.react.dev/reference/react/useMemo
- React — useMemo (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-memo
- React — use-memo lint rule — https://zh-hant.react.dev/reference/eslint-plugin-react-hooks/lints/use-memo
- React — exhaustive-deps — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- CoreUI — How to Use useMemo in React — https://coreui.io/answers/how-to-use-usememo-in-react/
- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/usememo-usecallback-in-react-19
- Steve Kinney — Avoiding Over‑Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- Steve Kinney — Identity Stability Props — https://stevekinney.com/courses/react-performance/identity-stability-props
- DebugBear — Improve React Performance With useMemo And useCallback — https://www.debugbear.com/blog/react-usememo-usecallback
- Feature-Sliced Design — When to useMemo: A React Performance Guide — https://feature-sliced.design/vi/blog/react-usememo-optimization
- FreeCodeCamp — How to Avoid Overusing useCallback and useMemo in React — https://www.freecodecamp.org
- Total TypeScript — Specify or Infer useMemo's Return Type — https://www.totaltypescript.com
- Oracle — Developing Open Storefront Framework Applications — https://docs.oracle.com
- GitHub — dev48v/usememo-vs-usecallback — https://github.com/dev48v/usememo-vs-usecallback
- LogRocket — The useless useCallback: React performance myths unpacked — https://podrocket.logrocket.com