# `useEffect` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useEffect` is a React Hook that lets you synchronize a component with an external system.

**Technical Definition:** `useEffect(setup, dependencies?)` is a React Hook that accepts a setup function containing effect logic, and an optional dependency array. The setup function may return a cleanup function. React runs the setup function after the component is added to the DOM. After every re-render with changed dependencies, React first runs the cleanup function with the old values, then runs the setup function with the new values. After the component is removed from the DOM, React runs the cleanup function. React compares each dependency with its previous value using `Object.is` [7†L4-L24].

**Beginner-Friendly Explanation:** `useEffect` lets your component do things after it renders, like fetching data, setting up timers, or listening to events. You tell React what to do, and React runs your code at the right time. If your code needs to clean up after itself (like cancelling a timer), you return a cleanup function.

### Key Characteristics

- **Synchronisation tool:** Connects a component to systems outside React's control (browser APIs, network, third-party libraries) [7†L2-L3].
- **After-paint by default:** Runs after the browser paints the updated screen, unless triggered by an interaction [7†L48-L51].
- **Cleanup support:** Returns an optional cleanup function to undo setup logic [7†L11-L16].
- **Dependency-driven:** Re-runs only when reactive dependencies change, compared with `Object.is` [7†L17-L25].
- **Top-level only:** Must be called at the top level of a component or custom Hook [7†L30-L32].
- **Strict Mode aware:** Runs an extra development-only setup+cleanup cycle to detect impurities [7†L34-L38].

### Prerequisites

- JavaScript functions, closures, and the event loop.
- React function components, `useState`, and JSX.
- Understanding of rendering and re-renders.
- Familiarity with browser APIs (timers, event listeners, fetch).

### Related Programming Areas

- **`useLayoutEffect`** — fires synchronously before the browser repaints [8†L4-L6].
- **`useInsertionEffect`** — fires before layout effects, for CSS-in-JS libraries [9†L6-L7].
- **`useEffectEvent`** — React 19 API for extracting non-reactive logic from effects [5†L41-L46].
- **`use()`** — React 19 API for consuming promises during render, replacing many data-fetching effects [5†L13-L16].

---

## 1. `useEffect`

### Definitions

**Core Definition:** `useEffect` is a React Hook that lets you synchronize a component with an external system.

**Technical Definition:** `useEffect(setup, dependencies?)` returns `undefined`. The `setup` parameter is a function with the effect's logic, which may optionally return a cleanup function. The `dependencies` parameter is an optional array of reactive values referenced inside the setup code. Reactive values include props, state, and all variables and functions declared directly inside the component body. React compares each dependency with its previous value using `Object.is`. If the dependency array is omitted, the effect re-runs after every re-render. The list must have a constant number of items and be written inline [7†L4-L29].

**Beginner-Friendly Explanation:** `useEffect` is a way to run code after your component renders. You give it a function to run, and optionally a list of values to watch. When those values change, your function runs again.

### Purposes

- To synchronize a component with an external system (browser API, network, third-party library) [7†L2-L3].
- To perform side effects that are not related to rendering.
- To set up and tear down subscriptions, timers, and event listeners.
- To fetch data from an API after the component mounts.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useEffect } from 'react';

useEffect(() => {
  // Setup logic
  return () => {
    // Cleanup logic
  };
}, [dependencies]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `setup` | Function containing effect logic; may return a cleanup function. |
| `dependencies` | Optional array of reactive values; React compares with `Object.is`. |
| Return value | `undefined`. |

**Syntax Rules:**

1. Call `useEffect` at the top level of a component or custom Hook [7†L30-L32].
2. The dependency array must have a constant number of items and be written inline [7†L22-L23].
3. If the dependency array is omitted, the effect runs after every render [7†L26-L27].
4. React runs cleanup with old values before running setup with new values [7†L13-L15].
5. Effects only run on the client, not during server rendering [8†L48-L49].

**Constraints and Limitations:**

- Cannot be called inside loops or conditions [7†L31-L32].
- If you are not synchronizing with an external system, you probably do not need an effect [7†L33-L34].
- Objects and functions defined inside the component can cause the effect to re-run more often than needed [7†L40-L44].

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Connection Effect

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

**Expected Output:** A heading welcoming the user to the room. The connection is re-established whenever `serverUrl` or `roomId` changes.

**Why this output:** The effect runs after the component mounts. It creates a connection and returns a cleanup that disconnects. When `serverUrl` or `roomId` changes, React runs the cleanup (disconnects the old connection) and then runs the setup (creates a new connection) [7†L8-L10].

#### Example 2: Window Resize Listener

```jsx
import { useState, useEffect } from 'react';

function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <p>Window: {size.width} x {size.height}</p>;
}
```

**Expected Output:** The window dimensions update as the browser is resized.

**Why this output:** The empty dependency array `[]` means the effect runs only once on mount. The cleanup removes the listener on unmount, preventing memory leaks [12†L13-L24].

### Real-World Cases

- **Chat applications:** Connecting to a chat server and disconnecting on room change.
- **Responsive components:** Listening to window resize events.
- **Data fetching:** Loading data from an API when a component mounts.
- **Timers:** Setting up intervals and clearing them on unmount.

### References

- React — useEffect — https://18.react.dev/reference/react/useEffect

---

## 2. Effect Function

### Definitions

**Core Definition:** The effect function (setup function) is the function passed to `useEffect` that contains the effect's logic.

**Technical Definition:** The `setup` function is the first argument to `useEffect`. It contains the code that synchronizes the component with an external system. It may optionally return a cleanup function. React runs the setup function after the component is added to the DOM, and after every re-render where dependencies have changed [7†L11-L15].

**Beginner-Friendly Explanation:** The effect function is the code you want React to run after rendering. It is where you put your side-effect logic.

### Purposes

- To contain the logic that synchronizes the component with an external system.
- To return a cleanup function that undoes the setup.
- To perform one-time or recurring side effects.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  // Effect logic here
  return () => {
    // Cleanup logic here
  };
}, [deps]);
```

**Syntax Rules:**

1. The effect function is called after the component renders [7†L11-L12].
2. It can return a cleanup function or nothing [7†L11-L12].
3. The cleanup function runs before the next effect and on unmount [7†L13-L16].
4. It must be synchronous; async functions cannot be passed directly.

### Annotated Code Examples

```jsx
useEffect(() => {
  // Setup: connect to a chat server
  const connection = createConnection(serverUrl, roomId);
  connection.connect();

  // Cleanup: disconnect
  return () => {
    connection.disconnect();
  };
}, [serverUrl, roomId]);
```

**Expected Output:** A connection is established and torn down as dependencies change.

**Why this output:** The effect function runs after render. It sets up the connection and returns a cleanup that disconnects. When dependencies change, React runs the cleanup first, then the setup with new values [7†L13-L15].

### Real-World Cases

- **Subscriptions:** Setting up and tearing down WebSocket connections.
- **Event listeners:** Adding and removing event listeners.
- **Timers:** Starting and clearing intervals.

### References

- React — useEffect: Parameters — https://18.react.dev/reference/react/useEffect#parameters

---

## 3. Cleanup Function

### Definitions

**Core Definition:** A cleanup function is an optional function returned from the effect function that undoes the setup logic before the next effect or on unmount.

**Technical Definition:** When the effect function returns a function, React treats it as a cleanup function. React runs the cleanup function before running the setup function again (with new dependencies) and before removing the component from the DOM. The cleanup function should stop or undo whatever the setup function started. In Strict Mode, React runs an extra setup+cleanup cycle in development to ensure the cleanup logic mirrors the setup logic [7†L11-L16][7†L34-L38].

**Beginner-Friendly Explanation:** The cleanup function is like an "undo" button for your effect. If your effect starts a timer, the cleanup clears it. If your effect connects to a server, the cleanup disconnects.

### Purposes

- To prevent memory leaks from timers, subscriptions, and event listeners.
- To cancel in-flight network requests.
- To reset DOM state before the next effect.
- To undo whatever the setup function started.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  // Setup
  return () => {
    // Cleanup
  };
}, [deps]);
```

**Syntax Rules:**

1. Return a function from the effect function to register cleanup [7†L11-L12].
2. React runs cleanup before the next effect and on unmount [7†L13-L16].
3. The cleanup function should mirror the setup logic.
4. In Strict Mode, React runs an extra setup+cleanup cycle in development [7†L34-L38].

**Constraints and Limitations:**

- If the cleanup function is missing, the setup logic may leak resources.
- The cleanup function must not throw errors.

### Annotated Code Examples

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    setSeconds(prev => prev + 1);
  }, 1000);

  return () => {
    clearInterval(intervalId);
  };
}, []);
```

**Expected Output:** A timer that increments every second and stops when the component unmounts.

**Why this output:** The cleanup function clears the interval when the component unmounts or before the effect re-runs, preventing the interval from continuing after unmount [17†L10-L14].

### Real-World Cases

- **Timers:** Clearing `setInterval` and `setTimeout`.
- **Subscriptions:** Unsubscribing from WebSocket or observables.
- **Event listeners:** Removing `addEventListener` handlers.
- **Network requests:** Aborting fetch with `AbortController`.

### References

- React — Synchronizing with Effects: Cleanup — https://react.dev/learn/synchronizing-with-effects
- CoreUI — How to fix memory leaks in React — https://coreui.io

---

## 4. Dependency Array

### Definitions

**Core Definition:** The dependency array is an optional second argument to `useEffect` that lists all reactive values the effect reads, controlling when the effect re-runs.

**Technical Definition:** The dependency array must list every reactive value referenced inside the effect. Reactive values include props, state, and all variables and functions declared directly inside the component body. React compares each dependency with its previous value using `Object.is`. If any dependency changes, the effect re-runs. The list must have a constant number of items and be written inline like `[dep1, dep2, dep3]` [7†L17-L25].

**Beginner-Friendly Explanation:** The dependency array tells React which values your effect cares about. If any of those values change, React runs your effect again. If you leave it empty, the effect runs only once.

### Purposes

- To control when the effect re-runs.
- To avoid stale closures by including all referenced values.
- To optimise performance by running the effect only when necessary.

### Syntax Rules and Structure

**Dependency Array Behaviours:**

| Array | Behaviour |
|-------|-----------|
| Omitted | Runs after every render [7†L26-L27]. |
| Empty `[]` | Runs only on mount; cleanup on unmount. |
| `[dep1, dep2]` | Runs when any dependency changes [7†L23-L24]. |

**Syntax Rules:**

1. Include all reactive values referenced inside the effect [7†L17-L19].
2. The array must have a constant number of items [7†L22-L23].
3. Write the array inline, not as a variable [7†L22-L23].
4. React uses `Object.is` for comparison [7†L23-L25].

**Constraints and Limitations:**

- Missing dependencies cause stale closures [11†L6-L7].
- Objects and functions defined inside the component cause the effect to re-run more often than needed [7†L40-L44].
- Suppressing the dependency linter is dangerous [4†L17-L18].

### Annotated Code Examples

```jsx
// ✅ Correct: all dependencies included
useEffect(() => {
  fetchUser(userId);
}, [userId]);

// ❌ Wrong: missing dependency (stale closure)
useEffect(() => {
  fetchUser(userId);
}, []); // Missing 'userId'
```

**Expected Output:** The correct version re-fetches when `userId` changes.

**Why this output:** The dependency array `[userId]` tells React to re-run the effect whenever `userId` changes. Without it, the effect uses the stale `userId` value from the initial render [11†L15-L17].

### Real-World Cases

- **Data fetching:** Re-fetch when the query or URL changes.
- **Subscriptions:** Re-subscribe when the channel changes.
- **Animations:** Re-trigger when the animation target changes.

### References

- React — useEffect: Dependencies — https://18.react.dev/reference/react/useEffect#parameters
- React — exhaustive-deps — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 5. No Dependency Array

### Definitions

**Core Definition:** Omitting the dependency array causes the effect to run after every render of the component.

**Technical Definition:** When the `dependencies` argument is omitted entirely, React runs the effect after every render. This is the default behaviour when no second argument is passed. It is rarely what you want, because it causes the effect to run unnecessarily on every render, potentially causing performance issues or infinite loops if the effect updates state [7†L26-L27].

**Beginner-Friendly Explanation:** If you do not give `useEffect` a dependency array, it runs every single time the component renders. This is usually not what you want.

### Purposes

- (Rarely used; omitting the array is usually an anti-pattern.)
- To run an effect on every render when the effect genuinely needs the latest values every time.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  // Runs after every render
});
```

**Syntax Rules:**

1. Omitting the array means no dependency tracking [7†L26-L27].
2. The effect runs after every render, including the initial render.
3. This can cause infinite loops if the effect updates state.

### Annotated Code Examples

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}); // Runs after every render
```

**Expected Output:** The document title updates on every render.

**Why this output:** Without a dependency array, the effect runs after every render. The title is always in sync with the count, but the effect runs more often than necessary.

### Real-World Cases

- **Debugging:** Logging on every render.
- **Rare cases:** When the effect genuinely depends on every value in the component.

### References

- React — useEffect: Dependencies — https://18.react.dev/reference/react/useEffect#examples-dependencies

---

## 6. Empty Dependency Array

### Definitions

**Core Definition:** An empty dependency array `[]` causes the effect to run only once on mount and cleanup only on unmount.

**Technical Definition:** When the dependency array is empty, React runs the effect after the initial render only. The cleanup function runs when the component unmounts. This is useful for one-time setup logic, such as fetching initial data or setting up a subscription that does not depend on any changing values. However, if the effect references props or state, those values will be stale [7†L27-L28].

**Beginner-Friendly Explanation:** An empty array `[]` means "run this effect only once, when the component first appears." It is like `componentDidMount` in class components.

### Purposes

- To run setup logic only once on mount.
- To set up subscriptions, timers, or event listeners that do not depend on changing values.
- To fetch initial data.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  // Runs only on mount
  return () => {
    // Cleanup on unmount
  };
}, []);
```

**Syntax Rules:**

1. The empty array means no dependencies [7†L27-L28].
2. The effect runs once after the initial render.
3. The cleanup runs on unmount.
4. Do not reference props or state inside the effect without including them in dependencies.

### Annotated Code Examples

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, []);
```

**Expected Output:** The connection is established on mount and disconnected on unmount.

**Why this output:** The empty dependency array ensures the effect runs only once. The cleanup runs on unmount, disconnecting the connection [7†L9-L10].

### Real-World Cases

- **Initial data fetching:** Loading data when the component first mounts.
- **Subscriptions:** Setting up a one-time subscription.
- **Third-party library initialisation:** Initialising a chart or map on mount.

### References

- React — useEffect: Empty dependencies — https://18.react.dev/reference/react/useEffect#examples-dependencies

---

## 7. Effect Execution Timing

### Definitions

**Core Definition:** Effect execution timing refers to when React runs the effect function relative to the browser's paint cycle.

**Technical Definition:** `useEffect` runs after the render is committed to the screen. If the effect was not caused by an interaction (like a click), React generally lets the browser paint the updated screen first before running the effect. If the effect is doing something visual and the delay is noticeable, `useLayoutEffect` should be used instead [7†L48-L51]. The execution order is: `useInsertionEffect` → `useLayoutEffect` → `useEffect` [13†L11-L16].

**Beginner-Friendly Explanation:** `useEffect` runs after the screen has been updated. If you need to do something before the user sees the screen, use `useLayoutEffect` instead.

### Purposes

- To understand when effects run relative to painting.
- To choose the right effect Hook for the task.
- To avoid visual flicker by using `useLayoutEffect`.

### Syntax Rules and Structure

**Execution Order:**

| Hook | Timing |
|------|--------|
| `useInsertionEffect` | Before DOM mutations [13†L11-L12]. |
| `useLayoutEffect` | After DOM mutations, before paint [13†L13-L14]. |
| `useEffect` | After paint [13†L15-L16]. |

**Syntax Rules:**

1. `useEffect` runs after the browser paints [7†L48-L51].
2. `useLayoutEffect` runs before paint [8†L4-L6].
3. `useInsertionEffect` runs before layout effects [9†L6-L7].

### Annotated Code Examples

```jsx
// useEffect: runs after paint — good for most side effects
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);

// useLayoutEffect: runs before paint — good for DOM measurements
useLayoutEffect(() => {
  const { height } = ref.current.getBoundingClientRect();
  setTooltipHeight(height);
}, []);
```

**Expected Output:** The title updates after paint; the tooltip height is measured before paint.

**Why this output:** `useEffect` runs after the browser paints, so the user may briefly see the old title. `useLayoutEffect` runs before paint, so the tooltip is positioned correctly without flicker [8†L9-L11].

### Real-World Cases

- **Tooltips:** Measuring and positioning before paint with `useLayoutEffect`.
- **Document title:** Updating after paint with `useEffect`.
- **CSS-in-JS:** Injecting styles before layout with `useInsertionEffect`.

### References

- React — useEffect: Caveats — https://18.react.dev/reference/react/useEffect#caveats
- Stack Overflow — Order of hooks in React 18 — https://stackoverflow.com/questions/71797967/what-is-the-order-of-hooks-in-react-18/71814373

---

## 8. Effect Cleanup Timing

### Definitions

**Core Definition:** Effect cleanup timing refers to when React runs the cleanup function returned from an effect.

**Technical Definition:** React runs the cleanup function in three situations: (1) before running the setup function again with new dependencies, (2) when the component unmounts, and (3) in Strict Mode, an extra setup+cleanup cycle runs in development [7†L13-L16][7†L34-L38]. The cleanup function receives no arguments and should undo the setup logic.

**Beginner-Friendly Explanation:** React runs your cleanup function before it runs your effect again, and when the component is removed from the screen.

### Purposes

- To ensure resources are released before the next effect.
- To prevent memory leaks on unmount.
- To reset state before re-running the effect.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  // Setup
  return () => {
    // Cleanup: runs before next effect and on unmount
  };
}, [deps]);
```

**Syntax Rules:**

1. Cleanup runs before the next effect with old values [7†L13-L15].
2. Cleanup runs on unmount [7†L16].
3. In Strict Mode, an extra setup+cleanup cycle runs in development [7†L34-L38].

### Annotated Code Examples

```jsx
useEffect(() => {
  console.log('Setup');
  return () => console.log('Cleanup');
}, [count]);
```

**Expected Output (on count change):**
```
Setup (initial)
Cleanup
Setup (count changed)
```

**Why this output:** React runs the cleanup with the old count before running the setup with the new count [7†L13-L15].

### Real-World Cases

- **Subscriptions:** Disconnecting before reconnecting.
- **Timers:** Clearing the old interval before starting a new one.
- **Event listeners:** Removing the old listener before adding a new one.

### References

- React — useEffect: Parameters — https://18.react.dev/reference/react/useEffect#parameters

---

## 9. Effect Ordering

### Definitions

**Core Definition:** Effect ordering refers to the sequence in which effects run across components and within a single component.

**Technical Definition:** Within a single component, effects run in the order they are declared. Across components, effects run from child components to parent components. When a component re-renders, React first runs cleanup functions from the previous effect, then runs the new effects. The overall order of effect hooks is: `useInsertionEffect` → `useLayoutEffect` → `useEffect` [13†L11-L16].

**Beginner-Friendly Explanation:** Effects run in a specific order: child components before parents, and within a component, in the order they are written.

### Purposes

- To understand why effects run in a particular order.
- To debug ordering issues.
- To coordinate multiple effects.

### Syntax Rules and Structure

**Ordering Rules:**

| Context | Order |
|---------|-------|
| Within a component | Declaration order. |
| Across components | Child → parent. |
| Cleanup vs setup | Cleanup first, then setup. |
| Hook types | `useInsertionEffect` → `useLayoutEffect` → `useEffect` [13†L11-L16]. |

**Syntax Rules:**

1. Effects run in declaration order within a component.
2. Child effects run before parent effects.
3. Cleanup runs before the next setup.

### Annotated Code Examples

```jsx
function Parent() {
  useEffect(() => { console.log('Parent effect'); }, []);
  return <Child />;
}

function Child() {
  useEffect(() => { console.log('Child effect'); }, []);
  return <p>Child</p>;
}
```

**Expected Output:**
```
Child effect
Parent effect
```

**Why this output:** Child effects run before parent effects. This is because React mounts children before parents [13†L11-L16].

### Real-World Cases

- **Data fetching:** Coordinating multiple fetches across components.
- **Subscriptions:** Managing cleanup order.
- **Debugging:** Understanding why effects run in a particular sequence.

### References

- Stack Overflow — Order of hooks in React 18 — https://stackoverflow.com/questions/71797967/what-is-the-order-of-hooks-in-react-18/71814373

---

## 10. Effect Batching

### Definitions

**Core Definition:** Effect batching is React's practice of grouping multiple state updates triggered by effects into a single re-render.

**Technical Definition:** When an effect triggers state updates, React batches them into a single re-render for performance. In React 18 and later, automatic batching is enabled by default, meaning updates are batched even inside promises, `setTimeout`, and native event handlers. This prevents unnecessary re-renders and avoids "half-finished" renders.

**Beginner-Friendly Explanation:** If your effect changes several pieces of state, React updates them all at once in a single re-render.

### Purposes

- To improve performance by reducing re-renders.
- To avoid inconsistent UI states during multiple updates.
- To ensure state updates from an effect are processed together.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React batches these into one re-render
}, [dep]);
```

**Syntax Rules:**

1. React batches updates within effects by default.
2. React 18 batches all updates, including async code.
3. To opt out, use `flushSync` (rarely recommended).

### Annotated Code Examples

```jsx
useEffect(() => {
  setLoading(false);
  setData(fetchedData);
  setError(null);
  // All three updates batched into one re-render
}, [fetchedData]);
```

**Expected Output:** Loading, data, and error states update together in a single re-render.

**Why this output:** React batches the three state updates into a single re-render, avoiding intermediate states where loading is false but data is not yet set.

### Real-World Cases

- **Data fetching:** Setting loading, data, and error states together.
- **Form submissions:** Updating multiple fields before a single re-render.
- **Complex UI state:** Toggling multiple flags together.

### References

- React Blog — Automatic batching in React 18 — https://reactjs.org/blog/2022/03/29/react-v18.html

---

## 11. Effect Pitfalls

### Definitions

**Core Definition:** Effect pitfalls are common mistakes developers make when using `useEffect`, leading to bugs, performance issues, or memory leaks.

**Technical Definition:** Common pitfalls include: missing dependencies (stale closures), infinite loops from object/function dependencies, forgetting cleanup, using async functions directly as effects, conditional effect calls, and using effects for derived state or data fetching when simpler alternatives exist [4†L4-L18][11†L6-L7][21†L11-L19].

**Beginner-Friendly Explanation:** These are the traps that catch developers when working with `useEffect`.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Missing dependency | Stale closure [11†L6-L7] | Include all reactive values. |
| Object/function dependency | Infinite loop [21†L22-L25] | Move outside or use `useCallback`. |
| Missing cleanup | Memory leak [4†L12-L14] | Return a cleanup function. |
| Async function as effect | Cannot return cleanup | Define async function inside. |
| Conditional effect | Breaks Hook order | Extract to a component. |
| Effect for derived state | Unnecessary render cycle [4†L31-L33] | Calculate during render. |
| Effect for data fetching | Race conditions, no cache [5†L13-L16] | Use `use()` or a query library. |

### Annotated Code Examples

**Pitfall 1: Missing Dependency**

```jsx
// ❌ Stale closure
useEffect(() => {
  console.log(count);
}, []); // Missing 'count'

// ✅ Correct
useEffect(() => {
  console.log(count);
}, [count]);
```

**Pitfall 2: Infinite Loop**

```jsx
// ❌ Infinite loop: new function every render
const logItems = () => console.log(items);
useEffect(() => {
  logItems();
}, [logItems]);

// ✅ Correct: move inside or use useCallback
useEffect(() => {
  console.log(items);
}, [items]);
```

**Pitfall 3: Missing Cleanup**

```jsx
// ❌ Memory leak
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
}, []); // No cleanup!

// ✅ Correct
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  return () => clearInterval(timer);
}, []);
```

**Expected Output:** The correct versions behave predictably; the incorrect versions cause bugs.

**Why this output:** Missing dependencies cause stale closures. Object/function dependencies cause infinite loops. Missing cleanup causes memory leaks.

### Real-World Cases

- **Debugging:** Identifying why state is not updating.
- **Code reviews:** Catching common effect mistakes.
- **Performance:** Avoiding unnecessary re-renders and memory leaks.

### References

- React — Removing Effect Dependencies — https://18.react.dev/learn/removing-effect-dependencies
- CoreUI — How to fix memory leaks in React — https://coreui.io
- Feature-Sliced Design — The Ultimate Guide to Mastering React's useEffect — https://feature-sliced.design

---

## 12. Effect Alternatives

### Definitions

**Core Definition:** Effect alternatives are patterns and APIs that replace `useEffect` for common use cases where an effect is not the right tool.

**Technical Definition:** React provides several alternatives to `useEffect` for common scenarios: derived state should be calculated during render, expensive computations should use `useMemo`, resetting state on prop change should use `key`, notifying parents of state changes should call the callback in the event handler, and data fetching should use `use()` with Suspense or a query library [4†L31-L35][5†L13-L16].

**Beginner-Friendly Explanation:** Sometimes you do not need `useEffect` at all. React has simpler ways to handle common patterns.

### Purposes

- To avoid unnecessary effects.
- To simplify code by using the right tool for the job.
- To improve performance by avoiding extra render cycles.

### Alternatives Table

| Use Case | Instead of `useEffect` | Use |
|----------|----------------------|-----|
| Derived state | Effect to set state | Calculate during render [4†L31-L33] |
| Expensive computation | Effect to compute | `useMemo` [4†L33] |
| Reset state on prop change | Effect to reset | `key` prop [4†L33] |
| Notify parent of state change | Effect to call callback | Call in event handler [4†L34] |
| Data fetching | Effect + fetch | `use(promise)` + Suspense [5†L13-L16] |
| External store subscription | Effect + setState | `useSyncExternalStore` |

### Annotated Code Examples

```jsx
// ❌ Unnecessary effect for derived state
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// ✅ Calculate during render
const fullName = `${firstName} ${lastName}`;
```

**Expected Output:** The correct version computes `fullName` directly, avoiding an extra render cycle.

**Why this output:** Derived state should be calculated during render, not stored in state via an effect [4†L31-L33].

### Real-World Cases

- **Form validation:** Calculate validity during render.
- **Filtered lists:** Use `useMemo` instead of an effect.
- **Data fetching:** Use `use()` or TanStack Query instead of `useEffect` + `fetch`.

### References

- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Removing Effect Dependencies — https://18.react.dev/learn/removing-effect-dependencies

---

## 13. `useLayoutEffect`

### Definitions

**Core Definition:** `useLayoutEffect` is a version of `useEffect` that fires synchronously after all DOM mutations but before the browser repaints the screen.

**Technical Definition:** `useLayoutEffect(setup, dependencies?)` has the same signature as `useEffect` but fires synchronously after React has performed all DOM mutations and before the browser paints. It blocks the browser from repainting until the effect completes. This is useful for measuring layout or synchronously re-rendering to prevent visual flicker. However, it can hurt performance because it blocks painting. Prefer `useEffect` when possible [8†L2-L6][8†L49-L52].

**Beginner-Friendly Explanation:** `useLayoutEffect` is like `useEffect` but runs before the screen updates. Use it when you need to measure something or prevent flicker.

### Purposes

- To measure layout before the browser paints.
- To synchronously re-render to prevent visual flicker.
- To read from or write to the DOM before paint [8†L36-L38].

### Syntax Rules and Structure

```jsx
import { useLayoutEffect } from 'react';

useLayoutEffect(() => {
  // Measure layout
  return () => {
    // Cleanup
  };
}, [deps]);
```

**Syntax Rules:**

1. Same signature as `useEffect` [8†L6-L7].
2. Fires synchronously after DOM mutations, before paint [8†L4-L6].
3. Blocks the browser from repainting [8†L49-L50].
4. Effects only run on the client [8†L48-L49].

**Constraints and Limitations:**

- Can hurt performance by blocking painting [8†L2-L3].
- Prefer `useEffect` when possible [8†L2-L3].
- State updates inside `useLayoutEffect` block painting [8†L49-L51].

### Annotated Code Examples

```jsx
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);

  return <div ref={ref}>Tooltip content</div>;
}
```

**Expected Output:** The tooltip height is measured before the browser paints, preventing flicker.

**Why this output:** `useLayoutEffect` runs before paint, so the tooltip is positioned correctly before the user sees it [8†L9-L11].

### Real-World Cases

- **Tooltips:** Measuring and positioning before paint.
- **Animations:** Preventing flicker by applying initial styles before paint.
- **DOM measurements:** Reading element dimensions before the browser paints.

### References

- React — useLayoutEffect — https://18.react.dev/reference/react/useLayoutEffect
- Steve Kinney — Performance Characteristics of useLayoutEffect — https://stevekinney.com

---

## 14. `useInsertionEffect`

### Definitions

**Core Definition:** `useInsertionEffect` is a Hook for CSS-in-JS library authors that inserts elements into the DOM before any layout effects fire.

**Technical Definition:** `useInsertionEffect(setup, dependencies?)` fires before any layout effects. It is designed for CSS-in-JS libraries to inject `<style>` tags before code that reads layout runs. Unlike other effects, it fires cleanup and setup one component at a time, interleaving them. State cannot be updated from inside `useInsertionEffect`, and refs are not yet attached when it runs. It may run before or after the DOM has been updated [9†L4-L37].

**Beginner-Friendly Explanation:** `useInsertionEffect` is a special Hook for libraries that inject CSS. Most developers will never need it.

### Purposes

- To inject styles into the DOM before layout effects [9†L6-L7].
- To insert `<style>` tags before code that reads layout [9†L13-L14].

### Syntax Rules and Structure

```jsx
import { useInsertionEffect } from 'react';

useInsertionEffect(() => {
  // Inject <style> tags here
});
```

**Syntax Rules:**

1. Same signature as `useEffect` [9†L7-L8].
2. Fires before layout effects [9†L6-L7].
3. State cannot be updated from inside [9†L31].
4. Refs are not attached yet [9†L32].
5. May run before or after DOM updates [9†L33-L34].

**Constraints and Limitations:**

- For CSS-in-JS library authors only [9†L2-L3].
- Most developers should use `useEffect` or `useLayoutEffect` instead [9†L3-L5].

### Annotated Code Examples

```jsx
import { useInsertionEffect } from 'react';

function useCSS(rule) {
  useInsertionEffect(() => {
    // Inject <style> tags here
  });
  return rule;
}
```

**Expected Output:** Styles are injected before layout effects fire.

**Why this output:** `useInsertionEffect` runs before layout effects, so the styles are available before any code that reads layout runs [9†L8-L11].

### Real-World Cases

- **CSS-in-JS libraries:** Styled-components, Emotion.
- **Style injection:** Inserting dynamic styles before layout.

### References

- React — useInsertionEffect — https://uk.react.dev/reference/react/useInsertionEffect

---

## 15. `useEffect` Best Practices

### Definitions

**Core Definition:** `useEffect` best practices are guidelines for using the Hook effectively, safely, and maintainably.

**Technical Definition:** Best practices encompass using effects only for synchronising with external systems, including all dependencies, returning cleanup functions, avoiding unnecessary effects, using `useEffectEvent` for non-reactive logic, and preferring `use()` or query libraries for data fetching. React 19 reduces the need for `useEffect` by providing `use()` and `useEffectEvent` [5†L4-L8][5†L12-L16].

**Beginner-Friendly Explanation:** These are the "do's and don'ts" that experienced React developers follow to keep their effects clean and bug-free.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Synchronise, don't fetch** | Use effects to synchronise with external systems, not for data fetching [7†L33-L34]. |
| **Include all dependencies** | Prevent stale closures by listing every reactive value [11†L6-L7]. |
| **Return cleanup** | Always clean up timers, subscriptions, and listeners [4†L12-L14]. |
| **Avoid unnecessary effects** | Use `useMemo`, `key`, or event handlers instead [4†L31-L35]. |
| **Use `useEffectEvent`** | Extract non-reactive logic from effects [5†L41-L46]. |
| **Prefer `use()`** | Use React 19's `use()` for data fetching [5†L13-L16]. |
| **Keep effects focused** | One effect, one reason [3†L9]. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Cleanup with dependencies
useEffect(() => {
  const controller = new AbortController();
  fetchData(url, { signal: controller.signal });
  return () => controller.abort();
}, [url]);

// Derived state calculated during render
const total = items.reduce((sum, i) => sum + i.price, 0);
```

**❌ Bad Practices:**

```jsx
// Missing cleanup
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
}, []); // Leak!

// Effect for derived state
const [total, setTotal] = useState(0);
useEffect(() => {
  setTotal(items.reduce((sum, i) => sum + i.price, 0));
}, [items]); // Extra render cycle
```

**Expected Output:** The good practices produce predictable, performant code.

**Why this output:** Cleanup prevents memory leaks. Derived state calculated during render avoids extra render cycles.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Code reviews:** Best practices are enforced through ESLint and review checklists.
- **Performance optimisation:** Avoiding unnecessary effects improves performance.

### References

- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Removing Effect Dependencies — https://18.react.dev/learn/removing-effect-dependencies
- React 19 — useEffectEvent — https://react.dev

---

## 16. Common `useEffect` Patterns

### Data Fetching

**Pattern:**

```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [url]);
```

**Explanation:** Fetch data when `url` changes. The cleanup aborts the request if the component unmounts or the URL changes, preventing race conditions and memory leaks. React 19's `use()` Hook and query libraries like TanStack Query are preferred alternatives for complex data fetching [5†L13-L16].

### Subscriptions

**Pattern:**

```jsx
useEffect(() => {
  const subscription = source.subscribe(handleChange);
  return () => subscription.unsubscribe();
}, [source]);
```

**Explanation:** Subscribe to an external data source on mount and unsubscribe on unmount. The cleanup prevents memory leaks and stale updates.

### Event Listeners

**Pattern:**

```jsx
useEffect(() => {
  const handleResize = () => setSize({ width: innerWidth, height: innerHeight });
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

**Explanation:** Add the listener on mount and remove it on unmount. The empty dependency array ensures the listener is added only once [12†L13-L24].

### Timers

**Pattern:**

```jsx
useEffect(() => {
  const intervalId = setInterval(() => setSeconds(s => s + 1), 1000);
  return () => clearInterval(intervalId);
}, []);
```

**Explanation:** Start the timer on mount and clear it on unmount. The cleanup prevents the timer from continuing after the component is removed [17†L10-L14].

### DOM Manipulation

**Pattern:**

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

**Explanation:** Update the document title after render. Use `useLayoutEffect` if the DOM change must happen before paint to avoid flicker [18†L7-L8].

### Animation

**Pattern:**

```jsx
useEffect(() => {
  const animation = animate(element, { opacity: 1 });
  return () => animation.cancel();
}, [element]);
```

**Explanation:** Start the animation on mount and cancel it on unmount. The cleanup prevents the animation from continuing after the element is removed.

### Syncing with External Systems

**Pattern:**

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);
```

**Explanation:** Connect to an external system on mount and reconnect when dependencies change. The cleanup disconnects before reconnecting [7†L8-L10].

---

## References

- React — useEffect — https://18.react.dev/reference/react/useEffect
- React — useLayoutEffect — https://18.react.dev/reference/react/useLayoutEffect
- React — useInsertionEffect — https://uk.react.dev/reference/react/useInsertionEffect
- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Removing Effect Dependencies — https://18.react.dev/learn/removing-effect-dependencies
- React — exhaustive-deps — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React Blog — Automatic batching in React 18 — https://reactjs.org/blog/2022/03/29/react-v18.html
- Stack Overflow — Order of hooks in React 18 — https://stackoverflow.com/questions/71797967/what-is-the-order-of-hooks-in-react-18/71814373
- Feature-Sliced Design — Architectural Patterns for Frontend Side Effects — https://feature-sliced.design
- Feature-Sliced Design — The Ultimate Guide to Mastering React's useEffect — https://feature-sliced.design
- CoreUI — How to fix memory leaks in React — https://coreui.io
- Steve Kinney — Performance Characteristics of useLayoutEffect — https://stevekinney.com
- fusengine/agents — Effect Patterns Template (React 19 + TypeScript) — https://github.com/fusengine/agents
- React 19 — useEffectEvent — https://react.dev