# `useDeferredValue` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useDeferredValue` is a React Hook that lets you defer updating a part of the UI, allowing urgent updates (like user input) to stay responsive while an expensive UI update lags behind.

**Technical Definition:** `useDeferredValue(value, initialValue?)` accepts a value and returns a deferred version of that value. On the initial render, the deferred value equals the provided value (or `initialValue` in Canary). During updates, React first re-renders with the old deferred value, then schedules a background re-render with the new value. The background re-render is interruptible: if a newer update arrives, React abandons the in-progress background render and restarts it with the latest value. Comparison uses `Object.is`.

**Beginner-Friendly Explanation:** `useDeferredValue` lets you say: "Use the latest value for the urgent part of the UI, but keep using the old value for the expensive part until React has time to update it." It is useful when you do not control the state setter — for example, when the value comes from props or an external store — and you want to defer the expensive rendering that depends on it.

### Key Characteristics

- **Value-centric deferral:** Defers a value, not a state update; React controls when to defer.
- **Adaptive, not timed:** Unlike `debounce` or `throttle`, there is no fixed delay. The deferred value updates immediately when React is idle, and lags behind when React is busy.
- **Interruptible background render:** The background re-render can be abandoned and restarted if the value changes again.
- **Stale-first rendering:** React first re-renders with the old deferred value, keeping the UI responsive, then updates in the background.
- **Suspense-integrated:** If the background update suspends, the user continues to see the old deferred value — not a fallback.
- **No automatic network cancellation:** `useDeferredValue` does not prevent extra network requests; it only defers rendering.
- **Pair with `useMemo`:** To be effective, the expensive computation or render must be memoized with the deferred value as a dependency.

### Prerequisites

- React function components, `useState`, and props.
- Understanding of reference equality and `Object.is`.
- Familiarity with `useMemo` and `React.memo`.
- Basic knowledge of concurrent rendering and Suspense.

### Related Programming Areas

- **`useTransition`** — the alternative when you control the state setter and want an `isPending` flag.
- **`useMemo`** — the companion hook for caching expensive computations based on the deferred value.
- **`Suspense`** — integrated with `useDeferredValue` for background updates.
- **Debouncing and Throttling** — alternative scheduling techniques with fixed delays.

---

## 1. `useDeferredValue`

### Definitions

**Core Definition:** `useDeferredValue` is a React Hook that lets you defer updating a part of the UI.

**Technical Definition:** `const deferredValue = useDeferredValue(value)` is called at the top level of a component. It returns a deferred version of `value`. On the initial render, the returned deferred value is the same as `value`. During updates, React first attempts a re-render with the old deferred value, then schedules a background re-render with the new value. The background re-render is interruptible. Comparison uses `Object.is`.

**Beginner-Friendly Explanation:** `useDeferredValue` gives you a "slow-motion" copy of a value. The real value updates immediately, but the deferred copy only catches up when React has spare time.

### Purposes

- To defer expensive UI updates caused by rapidly changing values.
- To keep high-priority updates (typing, clicking) responsive.
- To show stale content while fresh content renders in the background.
- To avoid Suspense fallbacks for background updates.
- To replace manual debouncing with adaptive, interruptible deferral.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useDeferredValue } from 'react';

const deferredValue = useDeferredValue(value);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `value` | The value you want to defer. Can be any type. |
| `initialValue` | (Canary only) Optional value to use during the initial render. |
| `deferredValue` | During the initial render, same as `value`. During updates, lags behind. |

**Syntax Rules:**

1. Call `useDeferredValue` at the top level of a component or custom Hook.
2. The value should be a primitive or an object created outside rendering.
3. The deferred value updates immediately when React is idle; it lags only when React is busy.
4. The background re-render is interruptible.
5. Inside a Transition, `useDeferredValue` always returns the new value and does not spawn a deferred render.

**Constraints and Limitations:**

- Creating a new object during render and passing it to `useDeferredValue` causes unnecessary background re-renders.
- `useDeferredValue` does not prevent extra network requests.
- It does not make the expensive work cheaper; it only reorders it.
- The expensive component must be memoized (`useMemo`, `React.memo`) or the deferral has no effect.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Deferred Search

```jsx
import { useState, useDeferredValue, useMemo } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const results = useMemo(
    () => expensiveSearch(deferredQuery),
    [deferredQuery]
  );

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ResultsList results={results} />
    </>
  );
}
```

**Expected Output:** The input updates immediately. The results list updates after the deferred value catches up, keeping typing smooth.

**Why this output:** `deferredQuery` lags behind `query`. The expensive search uses `deferredQuery` and is memoized, so it only recomputes when the deferred value changes — not on every keystroke.

#### Example 2: Visual Staleness Indicator

```jsx
function SearchWithIndicator() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <SearchResults query={deferredQuery} />
      </div>
    </>
  );
}
```

**Expected Output:** While the deferred value is catching up, the results section is dimmed, indicating that the content is stale.

**Why this output:** `isStale` compares the current value with the deferred value. When they differ, the UI shows a visual indication that the results are updating.

### Real-World Cases

- **Search autocomplete:** Keeping typing responsive while results filter.
- **Data visualisation:** Deferring chart re-renders while controls remain interactive.
- **Large lists:** Deferring expensive list rendering triggered by filters.
- **Product tours:** Highlight positioning that lags behind rapid state changes.

### References

- React — useDeferredValue — https://18.react.dev/reference/react/useDeferredValue
- React — useDeferredValue (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-deferred-value
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns

---

## 2. Deferred Values

### Definitions

**Core Definition:** A deferred value is a version of a value that React is allowed to lag behind, updating only when higher-priority work is complete.

**Technical Definition:** A deferred value is the return value of `useDeferredValue`. It is not a separate state variable; it is a derived value that React controls. During the initial render, it equals the provided value. During updates, it holds the previous value while React renders the new value in a background, interruptible pass. When the background render completes, the deferred value catches up.

**Beginner-Friendly Explanation:** A deferred value is a "slow copy" of a value. It is always slightly behind the real value, but it is the copy that expensive UI components use so they do not block urgent updates.

### Purposes

- To decouple urgent value updates from expensive UI updates.
- To allow React to prioritise rendering work.
- To keep the UI responsive during heavy computations.
- To provide a stale-while-rendering experience.

### Syntax Rules and Structure

```jsx
const deferredValue = useDeferredValue(value);
```

**Syntax Rules:**

1. The deferred value is read-only; you do not set it directly.
2. It updates automatically when React has time.
3. It is compared with the original value using `Object.is`.
4. It can be of any type, but primitives and external objects are safest.

### Annotated Code Examples

```jsx
function FilteredList({ items, filter }) {
  const deferredFilter = useDeferredValue(filter);

  const filteredItems = useMemo(
    () => items.filter(item =>
      item.name.toLowerCase().includes(deferredFilter.toLowerCase())
    ),
    [items, deferredFilter]
  );

  return <List items={filteredItems} />;
}
```

**Expected Output:** The filter input updates immediately; the list filters using the deferred value, keeping the input responsive.

**Why this output:** `deferredFilter` is the deferred value. The expensive filter uses it, so it runs only when the deferred value changes — not on every keystroke.

### Real-World Cases

- **Search filters:** Deferring the filter value while the input stays urgent.
- **Chart data:** Deferring the data value while controls stay responsive.
- **List sorting:** Deferring the sort criteria while the selector updates immediately.

### References

- React — useDeferredValue: Returns — https://18.react.dev/reference/react/useDeferredValue
- React — useDeferredValue: Caveats — https://18.react.dev/reference/react/useDeferredValue

---

## 3. Concurrent Rendering

### Definitions

**Core Definition:** Concurrent rendering is React's interruptible rendering model that allows the framework to prepare multiple UI versions simultaneously and prioritise urgent updates.

**Technical Definition:** Concurrent rendering is the foundation that makes `useDeferredValue` possible. React can pause, resume, or abandon rendering work. When a value changes, React first re-renders with the old deferred value, then schedules a background re-render with the new value. This background re-render is interruptible: if the value changes again, React abandons the in-progress render and restarts with the latest value. The deferred value updates when the background render commits.

**Beginner-Friendly Explanation:** Concurrent rendering is React's multitasking ability. `useDeferredValue` uses this ability to put expensive rendering work in the background, where it can be paused and resumed without blocking the user.

### Purposes

- To enable interruptible background rendering.
- To prioritise urgent updates over non-urgent ones.
- To keep the UI responsive during heavy computations.
- To allow deferred values to catch up when React is idle.

### Syntax Rules and Structure

**Concurrent Rendering Behaviour:**

| Phase | What happens |
|-------|--------------|
| Urgent render | React re-renders with the old deferred value, keeping the UI responsive. |
| Background render | React schedules a re-render with the new deferred value. |
| Interruption | If the value changes again, React abandons the background render and restarts. |
| Commit | The deferred value updates when the background render commits. |

**Syntax Rules:**

1. `useDeferredValue` relies on concurrent rendering; it is not a timed debounce.
2. The background render is interruptible and adaptive.
3. Components must be pure to support interruption.
4. The deferred value does not update until the background render commits.

### Annotated Code Examples

```jsx
function Chart({ data }) {
  const deferredData = useDeferredValue(data);

  const chart = useMemo(() => buildChart(deferredData), [deferredData]);

  return <ChartCanvas data={chart} />;
}
```

**Expected Output:** When `data` changes rapidly, the chart renders using the deferred data, which catches up when React has spare cycles.

**Why this output:** Concurrent rendering allows React to defer the chart re-render while urgent updates (like data controls) are processed. The deferred data updates when the background render completes.

### Real-World Cases

- **Real-time dashboards:** Charts updating with deferred data while controls remain interactive.
- **Search results:** Results rendering in the background while typing continues.
- **Data tables:** Sorting and filtering deferred while column controls update immediately.

### References

- React — useDeferredValue: Caveats — https://18.react.dev/reference/react/useDeferredValue
- Concurrent Features (React 18+) — https://github.com/syngrisi/syngrisi/blob/7f44c5a24cbb451b75a7ff25a1b16e5acf95162f/.agents/skills/react-performance-optimization/references/concurrent-features.md
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns

---

## 4. Stale Values

### Definitions

**Core Definition:** A stale value is the previous version of a value that remains in the UI while React prepares the updated version in the background.

**Technical Definition:** When a value changes, `useDeferredValue` first returns the old value (the stale value) in a re-render, then schedules a background re-render with the new value. The stale value keeps the UI responsive and avoids blank screens or fallbacks. If the background update suspends, the user continues to see the stale value until the data loads. The staleness can be detected by comparing the current value with the deferred value.

**Beginner-Friendly Explanation:** A stale value is what the user sees while the new value is being prepared. It is the old version of the UI, kept visible so the user is not staring at a blank screen or a spinner.

### Purposes

- To keep the UI responsive during background rendering.
- To avoid blank screens or Suspense fallbacks for non-urgent updates.
- To provide a stale-while-rendering experience.
- To allow visual indicators of staleness.

### Syntax Rules and Structure

**Detecting Staleness:**

```jsx
const deferredValue = useDeferredValue(value);
const isStale = value !== deferredValue;
```

**Syntax Rules:**

1. The stale value is returned during the urgent render.
2. The deferred value catches up after the background render commits.
3. `isStale` can be used to dim or indicate that content is updating.
4. If the background update suspends, the stale value remains visible.

### Annotated Code Examples

```jsx
function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      <ResultsList query={deferredQuery} />
    </div>
  );
}
```

**Expected Output:** The results list uses the deferred query. When the deferred query lags behind, the list is dimmed, indicating that the content is stale.

**Why this output:** `isStale` compares the current query with the deferred query. When they differ, the UI shows that the results are updating. The stale results remain visible until the background render completes.

### Real-World Cases

- **Search interfaces:** Showing previous results while new results render.
- **Data visualisation:** Showing the previous chart while new data is processed.
- **Product tours:** Showing the previous highlight position while the new position is computed.

### References

- React — useDeferredValue: Showing stale content while fresh content is loading — https://18.react.dev/reference/react/useDeferredValue
- React — useDeferredValue: Caveats — https://18.react.dev/reference/react/useDeferredValue
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns

---

## 5. `useDeferredValue` Use Cases

### Definitions

**Core Definition:** Use cases for `useDeferredValue` are scenarios where a rapidly changing value drives an expensive UI update that should not block urgent interactions.

**Technical Definition:** `useDeferredValue` is appropriate when you do not control the state setter and need to defer the expensive rendering that depends on a value. The canonical use cases are: filtering or searching large lists, expensive visualisations (charts, graphs) reacting to input, and any derived state that causes noticeable render delays. It is also useful when optimizing a child component without modifying the parent.

**Beginner-Friendly Explanation:** Use `useDeferredValue` when you have a fast-changing value (like a search query) and a slow-rendering part of the UI (like a results list or chart). It is especially useful when the value comes from props or an external store that you cannot wrap in a transition.

### Purposes

- To keep input responsive during expensive filtering or searching.
- To defer expensive chart or graph re-renders.
- To optimize child components without changing the parent.
- To replace manual debouncing for derived renders.
- To defer expensive list rendering triggered by rapidly changing filters.

### Use Cases Table

| Use case | Why `useDeferredValue` |
|----------|------------------------|
| Filtering/searching large lists | The list rendering is expensive and should not block typing. |
| Expensive visualisations (charts, graphs) | Chart re-renders are costly; deferring keeps controls responsive. |
| Derived state causing render delays | Any expensive computation triggered by input can be deferred. |
| Optimizing a child component | The child creates its own deferred value without parent changes. |
| Replacing manual debounce for renders | Adaptive and interruptible, unlike fixed-delay debounce. |

### Annotated Code Examples

```jsx
function ProductSearch() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const filteredProducts = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(deferredQuery.toLowerCase())
    ),
    [deferredQuery]
  );

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ProductList products={filteredProducts} />
    </>
  );
}
```

**Expected Output:** The input remains responsive while the product list filters using the deferred query.

**Why this output:** The expensive filter uses `deferredQuery` and is memoized. The input uses `query`, so it updates immediately. The deferred value catches up when React has spare cycles.

### Real-World Cases

- **E-commerce search:** Filtering a product catalog while typing.
- **Analytics dashboards:** Deferring chart updates while date pickers remain interactive.
- **Admin panels:** Deferring table filtering while search inputs remain responsive.

### References

- Use useDeferredValue for Expensive Derived Renders — https://github.com/hbmartin/react-mentions-ts/blob/4354af18f1c0840b02ff8353eb50f8fbc633ac77/best-practices/rerender-use-deferred-value.md
- React — useDeferredValue: Deferring re-rendering for a part of the UI — https://mintlify.wiki/facebook/react/api/hooks/use-deferred-value
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns

---

## 6. `useDeferredValue` vs `useTransition`

### Definitions

**Core Definition:** `useDeferredValue` defers a value and lets React control when to update it, while `useTransition` defers a state update and lets you control when it is marked as non-urgent.

**Technical Definition:** The core difference is **what is deferred**: `useTransition` defers a state update function (you control when to defer); `useDeferredValue` defers a changing value (React controls when to defer). `useTransition` provides an `isPending` flag and is used when you control the state setter. `useDeferredValue` is used when the value comes from props or an external store you do not control. `useTransition` causes two renders; `useDeferredValue` also uses a double-render strategy but adapts to React's idle time.

**Beginner-Friendly Explanation:** Use `useTransition` when you have the `set` function and want a loading flag. Use `useDeferredValue` when you receive a value as a prop or from an external store and just want it to lag behind.

### Comparison Table

| Aspect | `useTransition` | `useDeferredValue` |
|--------|-----------------|-------------------|
| What is deferred | A state update function | A changing value |
| Who controls deferral | You (wrap the update) | React (wrap the value) |
| Loading flag | Yes (`isPending`) | No (detect staleness via `value !== deferredValue`) |
| Use when you control the setter | Yes | No (use when you do not) |
| Multiple related updates | Yes (wrap them together) | No (one value at a time) |
| Typical use case | Tab switching, form submission | Search filtering, chart updates |

### Annotated Code Examples

**`useTransition` (you control the setter):**

```jsx
const [isPending, startTransition] = useTransition();
startTransition(() => setFilter(value));
```

**`useDeferredValue` (you receive the value):**

```jsx
const deferredFilter = useDeferredValue(filterProp);
```

**Expected Output:** Both defer expensive rendering, but `useTransition` provides a pending flag and requires access to the setter.

**Why this output:** `useTransition` is appropriate when you own the state update. `useDeferredValue` is appropriate when the value is external and you only need a deferred version.

### Real-World Cases

- **Own forms:** `useTransition` for submissions with `isPending`.
- **Child components:** `useDeferredValue` for deferring props-driven renders.
- **External stores:** `useDeferredValue` when the value comes from a global store.

### References

- Concurrent Features (React 18+) — https://github.com/syngrisi/syngrisi/blob/7f44c5a24cbb451b75a7ff25a1b16e5acf95162f/.agents/skills/react-performance-optimization/references/concurrent-features.md
- React’s useTransition: The hook you’re probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide
- Choosing Between useTransition and useDeferredValue — https://www.educative.io

---

## 7. `useDeferredValue` Best Practices

### Definitions

**Core Definition:** `useDeferredValue` best practices are the guidelines for using the Hook effectively, avoiding common pitfalls, and ensuring that deferral actually improves performance.

**Technical Definition:** Best practices centre on four principles: **pair with `useMemo`** (or `React.memo`) so the expensive computation or render actually uses the deferred value; **pass primitives or external objects** to avoid unnecessary background re-renders; **do not use it to defer network requests** (it does not cancel them); and **use it when you do not control the state setter**, otherwise prefer `useTransition` for its `isPending` flag. The deferred value should drive the expensive part of the UI, not the urgent part.

**Beginner-Friendly Explanation:** Always wrap the expensive work in `useMemo` with the deferred value as a dependency. Do not pass newly created objects during render. Do not expect `useDeferredValue` to cancel fetch requests. Use it when you do not have access to the setter.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Pair with `useMemo`** | The expensive computation must be memoized with the deferred value as a dependency. |
| **Pass primitives or external objects** | Avoid creating new objects during render; they cause unnecessary background re-renders. |
| **Do not use for network requests** | `useDeferredValue` does not prevent or cancel extra network requests. |
| **Use when you do not control the setter** | If you control the setter, prefer `useTransition` for `isPending`. |
| **Defer the expensive part, not the urgent part** | The input should use the original value; the expensive UI should use the deferred value. |
| **Detect staleness** | Compare `value !== deferredValue` to show a visual indicator. |
| **Remember it is adaptive, not timed** | It updates immediately when React is idle; it is not a debounce. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);

  const results = useMemo(
    () => expensiveSearch(deferredQuery),
    [deferredQuery]
  );

  return <ResultsList results={results} />;
}
```

```jsx
// Parent uses the urgent value
<input value={query} onChange={(e) => setQuery(e.target.value)} />
<SearchResults query={query} />
```

**❌ Bad Practices:**

```jsx
// ❌ No useMemo: the expensive search runs on every render
const deferredQuery = useDeferredValue(query);
const results = expensiveSearch(deferredQuery); // Still recomputes every render

// ❌ New object during render: causes unnecessary background re-renders
const deferredObj = useDeferredValue({ data: query }); // New object each render

// ❌ Deferring a network request: does not cancel it
const deferredId = useDeferredValue(id);
useEffect(() => { fetch(`/api/${deferredId}`); }, [deferredId]);
```

**Expected Output:** The good practices produce responsive input and deferred expensive rendering. The bad practices recompute on every render or create unnecessary background work.

**Why this output:** Without `useMemo`, the expensive computation runs on every render regardless of the deferred value. New objects trigger unnecessary background renders. `useDeferredValue` does not manage network requests.

### Real-World Cases

- **Search interfaces:** Memoized filtering with the deferred query.
- **Chart components:** Memoized chart building with deferred data.
- **Child optimization:** Child receives the original value and creates its own deferred version.

### References

- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns
- Use useDeferredValue for Expensive Derived Renders — https://github.com/hbmartin/react-mentions-ts/blob/4354af18f1c0840b02ff8353eb50f8fbc633ac77/best-practices/rerender-use-deferred-value.md
- React — useDeferredValue: Caveats — https://18.react.dev/reference/react/useDeferredValue
- React’s useTransition: The hook you’re probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide

---

## 8. `useDeferredValue` Pitfalls

### Definitions

**Core Definition:** `useDeferredValue` pitfalls are common mistakes that make deferral ineffective, introduce bugs, or cause unintended behaviour.

**Technical Definition:** Common pitfalls include: failing to memoize the expensive computation (so it runs on every render), creating new objects during render and passing them to `useDeferredValue` (causing unnecessary background re-renders), expecting `useDeferredValue` to cancel network requests (it does not), deferring the urgent part of the UI instead of the expensive part, and using it inside a Transition where it has no effect. Integration with Suspense can also cause stale values to persist until data loads.

**Beginner-Friendly Explanation:** These are the traps that make `useDeferredValue` either useless or harmful. The biggest one is forgetting `useMemo` — without it, the expensive work still runs on every render.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **No `useMemo`** | Expensive computation runs on every render, defeating deferral. | Wrap in `useMemo` with `deferredValue` as dependency. |
| **New object during render** | Causes unnecessary background re-renders. | Pass primitives or objects created outside rendering. |
| **Deferring network requests** | `useDeferredValue` does not cancel or prevent requests. | Use a query library or `AbortController`. |
| **Deferring the urgent part** | The input becomes laggy if it uses the deferred value. | Use the original value for urgent UI; deferred for expensive UI. |
| **Inside a Transition** | `useDeferredValue` has no effect; it returns the new value immediately. | Do not wrap transition updates in `useDeferredValue`. |
| **Stale value stuck with Suspense** | Deferred value can get stuck if combined with `use()` and `Suspense`. | Be aware of React issue #35932; test thoroughly. |

### Annotated Code Examples

**Pitfall 1: No `useMemo`**

```jsx
// ❌ Deferral has no effect: search runs on every render
const deferredQuery = useDeferredValue(query);
const results = expensiveSearch(deferredQuery);

// ✅ Memoized: search runs only when deferredQuery changes
const results = useMemo(
  () => expensiveSearch(deferredQuery),
  [deferredQuery]
);
```

**Pitfall 2: New Object During Render**

```jsx
// ❌ New object every render: unnecessary background re-renders
const deferredData = useDeferredValue({ data: query });

// ✅ Primitive or external object
const deferredQuery = useDeferredValue(query);
```

**Pitfall 3: Deferring the Urgent Part**

```jsx
// ❌ Input uses deferred value: typing feels laggy
<input value={deferredQuery} onChange={...} />

// ✅ Input uses original value; expensive UI uses deferred value
<input value={query} onChange={...} />
<ResultsList query={deferredQuery} />
```

**Expected Output:** The correct versions produce responsive input and effective deferral. The incorrect versions recompute on every render or make the input laggy.

**Why this output:** Without `useMemo`, deferral does not reduce computation. New objects trigger unnecessary renders. Deferring the urgent part reverses the priority.

### Real-World Cases

- **Search interfaces:** Forgetting `useMemo` is the most common `useDeferredValue` mistake.
- **Chart components:** New object props cause unnecessary chart re-renders.
- **Suspense integration:** Stale values can persist if the background update suspends.

### References

- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns
- React — useDeferredValue: Caveats — https://18.react.dev/reference/react/useDeferredValue
- GitHub Issue #35932 — useDeferredValue gets stuck with stale value — https://github.com/facebook/react/issues/35932
- React’s useTransition: The hook you’re probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide

---

## References

- React — useDeferredValue — https://18.react.dev/reference/react/useDeferredValue
- React — useDeferredValue (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-deferred-value
- React — useDeferredValue (React 19) — https://19.reactjs.ac.cn
- React — useTransition — https://react.dev/reference/react/useTransition
- React — Suspense — https://react.dev/reference/react/Suspense
- Steve Kinney — useDeferredValue Patterns — https://stevekinney.com/courses/react-performance/usedeferredvalue-patterns
- Use useDeferredValue for Expensive Derived Renders — https://github.com/hbmartin/react-mentions-ts/blob/4354af18f1c0840b02ff8353eb50f8fbc633ac77/best-practices/rerender-use-deferred-value.md
- Concurrent Features (React 18+) — https://github.com/syngrisi/syngrisi/blob/7f44c5a24cbb451b75a7ff25a1b16e5acf95162f/.agents/skills/react-performance-optimization/references/concurrent-features.md
- React’s useTransition: The hook you’re probably using wrong — https://www.nutrient.io/blog/react-usetransition-guide
- Choosing Between useTransition and useDeferredValue — https://www.educative.io
- GitHub Issue #35932 — useDeferredValue gets stuck with stale value — https://github.com/facebook/react/issues/35932
- GitHub Issue #34376 — useDeferredValue initialValue skipped — https://github.com/facebook/react/issues/34376
- GitHub Issue #38132 — Infinite loop with useDeferredValue + Suspense — https://github.com/facebook/react/issues/38132
- React 19 concurrent mode and product tours — https://raw.githubusercontent.com/domidex01/tour-kit