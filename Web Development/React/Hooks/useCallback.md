# `useCallback` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useCallback` is a React Hook that lets you cache a function definition between re-renders.

**Technical Definition:** `const cachedFn = useCallback(fn, dependencies)` returns a memoized version of the callback function `fn` that only changes if one of the dependencies has changed. On the initial render, `useCallback` returns the function you passed. On subsequent renders, React returns the already stored function if the dependencies have not changed (compared with `Object.is`), or returns the new function you passed during the current render. React does not call your function — it returns it to you so you can decide when and whether to call it. `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. It must be called at the top level of a component or custom Hook. In React 19, the React Compiler automatically memoizes values and functions, reducing the need for manual `useCallback` calls.

**Beginner-Friendly Explanation:** `useCallback` is a way to tell React: "Remember this function. Only give me a new one if these specific things change." This matters because in JavaScript, every time you create a function inside a component, it is a brand new function — even if the code inside is identical. When you pass that function to a child component that is wrapped in `React.memo`, the child sees a "new" prop every time and re-renders unnecessarily. `useCallback` keeps the function reference the same across renders, so the child can skip re-rendering.

### Key Characteristics

- **Function memoization:** Caches the function definition (reference), not the function's return value.
- **Dependency-driven:** Returns the same function reference until a dependency changes.
- **Referential stability:** Ensures stable function references for memoized children and Hook dependencies.
- **Performance optimization:** Not a semantic guarantee — React may discard the cache in certain situations (e.g., component suspends during initial mount).
- **Compiler-aware:** React 19's compiler can handle memoization automatically, reducing the need for manual `useCallback`.
- **Top-level only:** Must be called at the top level of a component or custom Hook.

### Prerequisites

- JavaScript functions, closures, and referential equality.
- React function components, JSX, and re-rendering.
- Understanding of `React.memo` and why child components re-render.
- Familiarity with `useMemo` and `useEffect` dependency arrays.

### Related Programming Areas

- **`useMemo`** — memoizes values, not functions.
- **`React.memo`** — memoizes component renders based on props.
- **`useEffect`** — dependency arrays follow the same `Object.is` comparison rules.
- **React Compiler** — automatic memoization in React 19+.

---

## 1. `useCallback`

### Definitions

**Core Definition:** `useCallback` is a React Hook that caches a function definition between re-renders.

**Technical Definition:** `useCallback(fn, dependencies)` returns a memoized version of `fn`. On the initial render, React returns `fn`. On subsequent renders, React returns the same `fn` if the dependencies have not changed since the last render; otherwise, it returns the new `fn` passed during the current render and stores it for future reuse. React will not call your function — it returns it to you. The dependency array must include all reactive values referenced inside `fn`. React compares each dependency with its previous value using `Object.is`.

**Beginner-Friendly Explanation:** `useCallback` is like a sticky note for your function. React writes the function on the note and keeps it. As long as the things the function depends on do not change, React keeps giving you the same function from the note instead of creating a new one.

### Purposes

- To maintain stable function references for memoized child components.
- To prevent unnecessary re-renders of components wrapped in `React.memo`.
- To stabilise function dependencies for `useEffect` and other Hooks.
- To avoid recreating event handlers on every render in performance-critical components.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useCallback } from 'react';

const cachedFn = useCallback(fn, dependencies);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `fn` | The function value to cache. |
| `dependencies` | Array of reactive values referenced inside `fn`. |
| `cachedFn` | The memoized function. |

**Syntax Rules:**

1. Call `useCallback` at the top level of a component or custom Hook.
2. The dependency array must have a constant number of items and be written inline like `[dep1, dep2, dep3]`.
3. React compares each dependency with its previous value using `Object.is`.
4. If the dependency array is omitted, a new function is created on every render (defeating the purpose).
5. If the dependency array is empty `[]`, the function never changes.

**Constraints and Limitations:**

- Cannot be called inside loops or conditions.
- React will throw away the cached function in development when you edit the component file, and in both development and production if your component suspends during initial mount.
- `useCallback` is only useful as an optimization when passing functions to components wrapped in `memo`, or when the function is used as a dependency in other Hooks.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Stable Callback for a Memoized Child

```jsx
import { useCallback, memo, useState } from 'react';

// Step 1: Define a memoized child component
const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  console.log('ShippingForm rendered');
  return <button onClick={() => onSubmit({})}>Submit</button>;
});

// Step 2: Parent component with useCallback
function ProductPage({ productId, referrer }) {
  const [count, setCount] = useState(0);

  // Step 3: Memoize the callback with dependencies
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**Expected Output:** Clicking the count button does not re-render `ShippingForm`.

**Why this output:** Without `useCallback`, `handleSubmit` would be a new function on every render. `React.memo` would detect a changed prop and re-render `ShippingForm`. With `useCallback`, the function reference stays stable until `productId` or `referrer` changes, so `ShippingForm` skips re-rendering.

#### Example 2: Stabilising a Function Dependency for `useEffect`

```jsx
import { useState, useCallback, useEffect } from 'react';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ✅ Memoize the function so it doesn't change on every render
  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId,
    };
  }, [roomId]);

  // ✅ Effect only re-runs when createOptions changes (i.e., roomId changes)
  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]);

  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <p>Room: {roomId}</p>
    </>
  );
}
```

**Expected Output:** The connection is re-established only when `roomId` changes, not when the user types in the message input.

**Why this output:** Without `useCallback`, `createOptions` would be a new function on every render. The `useEffect` dependency array `[createOptions]` would see a "changed" dependency on every render, causing the connection to be torn down and recreated on every keystroke. `useCallback` keeps `createOptions` stable until `roomId` changes.

### Real-World Cases

- **Design systems:** Components that accept callback props and are wrapped in `React.memo`.
- **Form handling:** Passing stable `onSubmit` or `onChange` handlers to memoized form components.
- **Real-time applications:** Stable event handlers for WebSocket or game room components.
- **Lists:** Stable callbacks for each item in a virtualised or memoized list.

### References

- React — useCallback — https://18.react.dev/reference/react/useCallback
- React — useCallback (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-callback

---

## 2. Function Memoization

### Definitions

**Core Definition:** Function memoization is the technique of caching a function definition so that the same function reference is returned across renders.

**Technical Definition:** In JavaScript, every time a function expression is evaluated inside a component, a new function object is created with a new reference. React compares props and Hook dependencies by reference (`Object.is`). If a function prop changes reference on every render, any component wrapped in `React.memo` will re-render, and any `useEffect` with that function in its dependency array will re-run. `useCallback` memoizes the function definition, returning the same reference until a dependency changes.

**Beginner-Friendly Explanation:** When you write a function inside a component, JavaScript creates a brand new function every time the component renders — even if the code is identical. `useCallback` stops that from happening by keeping the same function around until something it depends on changes.

### Purposes

- To prevent unnecessary re-renders of memoized child components.
- To stabilise function dependencies for `useEffect` and other Hooks.
- To avoid recreating event handlers on every render.

### Syntax Rules and Structure

```jsx
const cachedFn = useCallback(fn, dependencies);
```

**Syntax Rules:**

1. `useCallback` returns the same function reference until a dependency changes.
2. The function is not called by `useCallback` — it is returned for you to call.
3. `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

### Annotated Code Examples

```jsx
// ❌ Without useCallback: new function every render
function Parent() {
  const handleClick = () => {
    console.log('Clicked');
  };
  return <MemoizedChild onClick={handleClick} />;
}

// ✅ With useCallback: same function reference until deps change
function Parent() {
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []); // No dependencies — never changes
  return <MemoizedChild onClick={handleClick} />;
}
```

**Expected Output:** The `MemoizedChild` only re-renders in the second version when other props change, not because of `handleClick`.

**Why this output:** In the first version, `handleClick` is a new function on every render, so `MemoizedChild` sees a changed prop and re-renders. In the second version, `handleClick` is the same function until the (empty) dependency array changes.

### Real-World Cases

- **Event handlers in memoized lists:** Each list item receives a stable callback.
- **Form components:** Stable `onSubmit` and `onChange` handlers passed to memoized inputs.
- **Animation callbacks:** Stable callbacks for animation libraries.

### References

- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/usememo-usecallback-in-react-19

---

## 3. Dependency Array

### Definitions

**Core Definition:** The dependency array is the second argument to `useCallback` that lists all reactive values the function reads.

**Technical Definition:** The `dependencies` array must include every reactive value referenced inside the callback function. Reactive values include props, state, and all variables and functions declared directly inside the component body. The array must have a constant number of items and be written inline like `[dep1, dep2, dep3]`. React compares each dependency with its previous value using `Object.is`. If any dependency changes, `useCallback` returns the new function passed during the current render. If the array is omitted, a new function is created on every render. If the array is empty `[]`, the function never changes.

**Beginner-Friendly Explanation:** The dependency array tells `useCallback` which values to watch. If any of those values change, `useCallback` gives you a new function. If none change, it gives you the same function as before.

### Purposes

- To control when the memoized function is recreated.
- To prevent stale closures by including all referenced reactive values.
- To optimise performance by keeping the function stable when dependencies are unchanged.

### Syntax Rules and Structure

**Dependency Array Behaviours:**

| Array | Behaviour |
|-------|-----------|
| Omitted | New function on every render (defeats the purpose) |
| Empty `[]` | Function never changes |
| `[dep1, dep2]` | Function updates when dependencies change |

**Syntax Rules:**

1. Include all reactive values referenced inside the callback.
2. The array must have a constant number of items and be written inline.
3. React uses `Object.is` for comparison.

**Constraints and Limitations:**

- Missing dependencies cause stale closures.
- Including values that change every render causes the function to be recreated on every render.
- The `exhaustive-deps` ESLint rule catches missing dependencies.

### Annotated Code Examples

```jsx
// ✅ Correct: all dependencies included
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', {
    referrer,
    orderDetails,
  });
}, [productId, referrer]); // Function only changes when these change

// ❌ Wrong: missing dependency (stale closure)
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', {
    referrer,
    orderDetails,
  });
}, []); // Uses stale productId and referrer
```

**Expected Output:** The correct version uses the latest `productId` and `referrer`; the incorrect version uses the initial values.

**Why this output:** The dependency array `[productId, referrer]` tells React to recreate the function when either value changes. Without them, the function captures the initial values and never updates.

### Real-World Cases

- **Form submissions:** Recreating the submit handler when the product ID or user changes.
- **Event handlers with props:** Including props that the handler reads.
- **Effects with callbacks:** Including callbacks that the effect depends on.

### References

- React — useCallback: Parameters — https://18.react.dev/reference/react/useCallback#parameters
- React — exhaustive-deps — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 4. Referential Equality

### Definitions

**Core Definition:** Referential equality means two values are the same object in memory, compared with `===` or `Object.is`.

**Technical Definition:** In JavaScript, functions are objects. Every time a function expression is evaluated, a new function object is created with a new reference. React uses referential equality in several places: `React.memo` does a shallow comparison of props; `useEffect`, `useMemo`, and `useCallback` compare dependencies with `Object.is`. When you pass a function created inside a component to a memoized child, the child sees a "new" prop on every render because the function reference changes. `useCallback` solves this by returning the same function reference across renders when dependencies are unchanged.

**Beginner-Friendly Explanation:** In JavaScript, two functions that look identical are not the same — they are different objects. `useCallback` makes them the same object so React knows nothing has changed.

### Purposes

- To prevent unnecessary re-renders of memoized child components.
- To stabilise dependencies for other Hooks.
- To maintain function identity across renders.

### Syntax Rules and Structure

```jsx
// ❌ New function every render — child re-renders
<MemoizedChild onClick={() => console.log('clicked')} />

// ✅ Stable function with useCallback
const handleClick = useCallback(() => console.log('clicked'), []);
<MemoizedChild onClick={handleClick} />
```

**Syntax Rules:**

1. Functions created inside the component body are new on every render.
2. `React.memo` uses shallow comparison — new references fail equality checks.
3. `useCallback` returns the same reference when dependencies are unchanged.

### Annotated Code Examples

```jsx
const ExpensiveChild = memo(function ExpensiveChild({ onAction }) {
  console.log('Child rendered');
  return <button onClick={onAction}>Act</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  // ✅ Stable function reference
  const handleAction = useCallback(() => {
    console.log('Action triggered');
  }, []);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveChild onAction={handleAction} />
    </div>
  );
}
```

**Expected Output:** Clicking the count button does not re-render `ExpensiveChild`.

**Why this output:** Without `useCallback`, `handleAction` would be a new function on every render. `React.memo` would see a changed `onAction` prop and re-render the child. With `useCallback`, the reference stays stable, so the child skips re-rendering.

### Real-World Cases

- **Memoized lists:** Stable callbacks for each list item.
- **Context values:** Stabilising function values in context providers.
- **Effect dependencies:** Preventing effects from running on every render.

### References

- React Basics — How to Use React useCallback Effectively — https://www.telerik.com/blogs/react-basics-how-use-react-usecallback-effectively
- Oracle — Developing Open Storefront Framework Applications — https://docs.oracle.com

---

## 5. When to Use `useCallback`

### Definitions

**Core Definition:** `useCallback` should be used when you have a concrete performance problem that memoizing a function solves.

**Technical Definition:** `useCallback` is appropriate in three scenarios: (1) passing callbacks to child components wrapped in `React.memo`, (2) using functions as dependencies in `useEffect`, `useMemo`, or other Hooks, and (3) event handlers in components that re-render frequently (e.g., real-time updates, animations). The benefit depends on the stability of the dependencies. If dependencies change frequently, the cache is invalidated often and the benefit is reduced.

**Beginner-Friendly Explanation:** Use `useCallback` when you are passing a function to a memoized child component, or when a function is used in another Hook's dependency array. Also use it for event handlers in components that update very frequently.

### Purposes

- To prevent unnecessary re-renders of memoized child components.
- To stabilise function dependencies for other Hooks.
- To avoid recreating event handlers in high-frequency components.

### Syntax Rules and Structure

**When to use:**

| Scenario | Example |
|----------|---------|
| Callbacks passed to memoized children | `onClick`, `onSubmit` props for `React.memo` components |
| Functions used in Hook dependencies | Functions in `useEffect` dependency arrays |
| Event handlers in high-frequency components | Real-time updates, game rooms, animations |

**Syntax Rules:**

1. Profile first — use React DevTools Profiler to confirm a performance problem.
2. Use `useCallback` only when the function is passed to a memoized child or used as a Hook dependency.
3. Prefer functional state updates to keep dependency arrays minimal.

### Annotated Code Examples

```jsx
// ✅ Good: callback passed to memoized child
const MemoizedChild = React.memo(ChildComponent);

function Parent() {
  const handleClick = useCallback(() => {
    // handler logic
  }, [dependencies]);

  return <MemoizedChild onClick={handleClick} />;
}

// ✅ Good: function used in useEffect dependency
const fetchData = useCallback(async () => {
  const data = await api.get('/data');
  setData(data);
}, [api]);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

**Expected Output:** The memoized child only re-renders when other props change; the effect only re-runs when `fetchData` changes.

**Why this output:** Both scenarios involve referential stability requirements that `useCallback` directly addresses.

### Real-World Cases

- **Design systems:** Stable callbacks for memoized primitives.
- **Form libraries:** Stable `onSubmit` handlers for memoized form components.
- **Game rooms:** Stable event handlers for high-frequency updates.

### References

- React — useCallback Optimization Guide — https://raw.githubusercontent.com/Shuffle-and-Sync/reimagined-guacamole/refs/heads/main/docs/performance/USECALLBACK_OPTIMIZATION_GUIDE.md
- CoreUI — How to Use useCallback in React — https://coreui.io/answers/how-to-use-usecallback-in-react/

---

## 6. When Not to Use `useCallback`

### Definitions

**Core Definition:** `useCallback` should not be used for simple event handlers, functions not passed as props, or as a precautionary optimisation.

**Technical Definition:** `useCallback` has overhead: React stores the previous function and dependencies in memory and compares them on every render. For simple event handlers that are not passed to memoized children, the comparison overhead exceeds any benefit. `useCallback` should not be used for functions that are not passed as props, handlers in components that rarely re-render, or every function in every component. React 19's compiler and better rendering improvements may reduce the necessity of `useCallback` in some cases.

**Beginner-Friendly Explanation:** Don't use `useCallback` for every function. If the function is not passed to a memoized child and is not a dependency of another Hook, you probably don't need it.

### Purposes

- To avoid the memoization tax (memory + comparison overhead).
- To keep code simple and readable.
- To prevent over-engineering.

### Common Over-Memoization Patterns

| Pattern | Why It's Bad |
|---------|-------------|
| Simple event handlers not passed as props | `useCallback(() => console.log('clicked'), [])` — overhead outweighs benefit |
| Handlers in rarely re-rendered components | Static pages don't need `useCallback` |
| Every function in every component | `useCallback` has overhead — only use when it provides measurable benefit |
| Functions not used in dependency arrays | If the function is only called locally, memoization adds no value |

**Syntax Rules:**

1. If the function is not passed to a memoized child or used as a Hook dependency, skip `useCallback`.
2. If you haven't measured a performance problem, skip `useCallback`.
3. Prefer functional state updates to avoid needing state in dependencies.

### Annotated Code Examples

```jsx
// ❌ Unnecessary: simple event handler not passed to memoized child
function Button() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  return <button onClick={handleClick}>Click</button>;
}

// ✅ Just let it be a regular function
function Button() {
  const handleClick = () => {
    console.log('clicked');
  };
  return <button onClick={handleClick}>Click</button>;
}
```

**Expected Output:** Both versions render the same output, but the second is simpler and faster.

**Why this output:** The `<button>` is a DOM element, not a memoized component. The function reference does not matter for DOM elements — React updates the listener anyway. The `useCallback` overhead is wasted.

### Real-World Cases

- **Static pages:** Components that rarely re-render.
- **Simple buttons:** Handlers that only call `setState` or log.
- **Utility functions:** Functions that are not passed as props.

### References

- React — Avoiding Over‑Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- React 19 — useCallback not needed now — https://github.com/react/react/issues/31913

---

## 7. `useCallback` Pitfalls

### Definitions

**Core Definition:** `useCallback` pitfalls are common mistakes that make memoization ineffective, harmful, or incorrect.

**Technical Definition:** Common pitfalls include: using `useCallback` for side effects, missing dependencies (stale closures), unnecessary memoization (over-memoization), using `useCallback` with refs (timing mismatch), relying on `useCallback` for correctness, and violating the Rules of Hooks (conditional calls). Each of these undermines the purpose of `useCallback` and can lead to bugs or performance degradation.

**Beginner-Friendly Explanation:** These are the traps that make `useCallback` either useless or harmful.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Missing dependencies** | Stale closure — function uses old values | Include all reactive values. |
| **Over-memoization** | Memoization overhead exceeds benefit | Only memoize when passing to memoized children or using as Hook deps. |
| **useCallback with refs** | `ref.current` is null during render; timing mismatch | Access refs in event handlers or effects. |
| **No dependency array** | New function every render (defeats the purpose) | Always provide a dependency array. |
| **Conditional useCallback** | Breaks Rules of Hooks | Always call at the top level. |
| **Relying on useCallback for correctness** | React may discard the cache | Use `useState`/`useRef` for values that must persist. |

### Annotated Code Examples

**Pitfall 1: Missing Dependency**

```jsx
// ❌ Stale closure: productId is captured from initial render
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', { orderDetails });
}, []); // Missing 'productId'

// ✅ Correct
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', { orderDetails });
}, [productId]);
```

**Pitfall 2: useCallback with Refs**

```jsx
// ❌ ref.current is null during render
const handleClick = useCallback(() => {
  inputRef.current.focus(); // May be null!
}, []);

// ✅ Access refs in event handlers
function handleClick() {
  inputRef.current?.focus();
}
```

**Pitfall 3: No Dependency Array**

```jsx
// ❌ New function every render
const handleClick = useCallback(() => {
  console.log(count);
}); // No dependency array!

// ✅ Correct
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);
```

**Expected Output:** The correct versions behave predictably; the incorrect versions cause bugs.

**Why this output:** Missing dependencies cause stale closures. Refs accessed during render are null. No dependency array defeats the purpose of `useCallback`.

### Real-World Cases

- **Debugging:** Identifying why memoized functions are stale.
- **Code reviews:** Catching over-memoization and missing dependencies.
- **Performance:** Avoiding the memoization tax on cheap functions.

### References

- React — useCallback: Caveats — https://18.react.dev/reference/react/useCallback#caveats
- The useless useCallback — https://podrocket.logrocket.com

---

## 8. `useCallback` Best Practices

### Definitions

**Core Definition:** `useCallback` best practices are guidelines for using the Hook effectively, safely, and maintainably.

**Technical Definition:** Best practices include: using `useCallback` only when passing functions to memoized children or using them as Hook dependencies, including all dependencies, preferring functional state updates to keep dependencies minimal, profiling before memoizing, not relying on `useCallback` for correctness, and leveraging React Compiler when available. In React 19, the compiler automatically memoizes values and functions, reducing the need for manual `useCallback` calls.

**Beginner-Friendly Explanation:** These are the "do's and don'ts" that experienced React developers follow to keep their memoization clean and effective.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Only use with memoized children or Hook deps** | Otherwise the overhead outweighs the benefit. |
| **Include all dependencies** | Prevent stale closures. |
| **Prefer functional state updates** | `setCount(c => c + 1)` avoids needing `count` in deps. |
| **Profile first** | Measure before memoizing. |
| **Don't rely on useCallback for correctness** | It is a performance hint, not a guarantee. |
| **Use React Compiler when available** | Automatic memoization in React 19+. |
| **Keep it simple** | If memoization adds complexity without clear benefit, remove it. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Pass stable callback to memoized child
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', { orderDetails });
}, [productId]);

// Use functional state update to avoid dependencies
const handleAddTodo = useCallback((text) => {
  setTodos(prevTodos => [...prevTodos, { id: nextId++, text }]);
}, []); // No dependencies needed!
```

**❌ Bad Practices:**

```jsx
// ❌ Memoizing a simple handler not passed to a memoized child
const handleClick = useCallback(() => console.log('clicked'), []);

// ❌ Missing dependency
const handleSubmit = useCallback(() => {
  post('/product/' + productId + '/buy');
}, []); // Missing 'productId'
```

**Expected Output:** The good practices produce predictable, performant code.

**Why this output:** Using `useCallback` only when needed avoids unnecessary overhead. Functional updates keep dependency arrays minimal.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Code reviews:** Best practices are enforced through ESLint and review checklists.
- **React 19 migration:** Moving from manual to compiler-driven memoization.

### References

- React — useCallback — https://18.react.dev/reference/react/useCallback
- React — useCallback (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-callback
- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/usememo-usecallback-in-react-19
- Steve Kinney — Avoiding Over‑Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- CoreUI — How to Use useCallback in React — https://coreui.io/answers/how-to-use-usecallback-in-react/

---

## References

- React — useCallback — https://18.react.dev/reference/react/useCallback
- React — useCallback (Mintlify) — https://mintlify.wiki/facebook/react/api/hooks/use-callback
- React — useCallback: Caveats — https://18.react.dev/reference/react/useCallback#caveats
- React — exhaustive-deps — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React 19 — useCallback not needed now — https://github.com/react/react/issues/31913
- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/usememo-usecallback-in-react-19
- Steve Kinney — Avoiding Over‑Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- Telerik — React Basics: How to Use React useCallback Effectively — https://www.telerik.com/blogs/react-basics-how-use-react-usecallback-effectively
- DebugBear — Improve React Performance With useMemo And useCallback — https://www.debugbear.com/blog/react-usememo-usecallback
- CoreUI — How to Use useCallback in React — https://coreui.io/answers/how-to-use-usecallback-in-react/
- Shuffle-and-Sync — useCallback Optimization Guide — https://raw.githubusercontent.com/Shuffle-and-Sync/reimagined-guacamole/refs/heads/main/docs/performance/USECALLBACK_OPTIMIZATION_GUIDE.md
- aiskillstore — Performance Optimization Patterns — https://github.com/aiskillstore/marketplace/blob/main/skills/diet103/frontend-dev-guidelines/resources/performance.md
- LogRocket — The useless useCallback: React performance myths unpacked — https://podrocket.logrocket.com
- Oracle — Developing Open Storefront Framework Applications — https://docs.oracle.com
- Feature-Sliced Design — A Guide to React's useCallback Hook — https://feature-sliced.design
- hygraph — React useCallback() - A complete guide — https://hygraph.com