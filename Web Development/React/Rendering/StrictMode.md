# React Strict Mode — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `StrictMode` is a React development tool that wraps a component tree and activates additional checks, warnings, and double-invocations to surface latent bugs before they reach production.

**Technical Definition:** `<StrictMode>` is a React component that enables development-only behaviours for its descendant tree: components re-render an extra time to find bugs caused by impure rendering; Effects re-run an extra time to find bugs caused by missing cleanup; ref callbacks re-run to find missing ref cleanup; and components are checked for deprecated API usage. `StrictMode` renders no visible UI, accepts no props, and has no effect on production builds. It cannot be opted out of within a wrapped tree, and it applies to all descendants regardless of depth.

**Beginner-Friendly Explanation:** Strict Mode is like a strict teacher who checks your work twice. While you are developing, it deliberately runs your components and effects an extra time to catch mistakes — like forgetting to clean up a timer or putting code in the wrong place. It also warns you if you are using old, unsafe patterns. The best part is that none of this affects your users: Strict Mode automatically switches off when you build for production. It exists so you can find bugs early, before your users do.

### Key Characteristics

- **Development-only:** All checks run exclusively in development; production builds are completely unaffected.
- **No visible UI:** `StrictMode` renders nothing; it is a pure behavioural wrapper.
- **Double-invocation of pure functions:** Component bodies, state initializers, `useMemo` factories, `useReducer` reducers, and `setState` updater functions are called twice in development.
- **Double-invocation of Effects:** `useEffect` setup and cleanup cycles run an extra time on mount to verify cleanup correctness.
- **Deprecation detection:** Warnings for unsafe lifecycles, legacy string refs, `findDOMNode`, and legacy context API.
- **Non-opt-outable within a tree:** Once a tree is wrapped, descendants cannot individually disable the checks.
- **Component-level scoping:** `StrictMode` can wrap the whole app or a specific subtree; only descendants are checked.

### Prerequisites

- React function components and Hooks (`useState`, `useEffect`, `useMemo`, `useReducer`).
- Understanding of pure functions and side effects.
- Familiarity with the render → commit lifecycle and effect cleanup.
- Basic knowledge of class component lifecycle methods (for unsafe lifecycle detection).

### Related Programming Areas

- **Concurrent Rendering** — Strict Mode simulates the double-invocation and interruption patterns that concurrent features rely on.
- **Effects and Cleanup** — the primary source of bugs detected by Strict Mode.
- **Pure Components** — Strict Mode enforces render purity.
- **Error Boundaries** — Strict Mode does not catch errors; boundaries handle runtime failures.

---

## 1. StrictMode

### Definitions

**Core Definition:** `<StrictMode>` is a React component that enables extra development-time checks for its descendants.

**Technical Definition:** `StrictMode` is imported from `react` and rendered as `<StrictMode>{children}</StrictMode>`. It activates four development-only behaviours: extra re-renders, extra Effect cycles, extra ref callback cycles, and deprecation checks. It has no props and renders no DOM. It can be applied at the application root or to any subtree. Nested `StrictMode` components have no additional effect beyond the outermost one.

**Beginner-Friendly Explanation:** `StrictMode` is the wrapper you put around your app (or part of it) to turn on React's strict checks.

### Purposes

- To detect impure rendering that could cause bugs under concurrent rendering.
- To detect missing Effect cleanup that could cause memory leaks and duplicate subscriptions.
- To detect missing ref cleanup that could cause stale DOM references.
- To warn about deprecated APIs that will be removed in future React versions.
- To prepare the application for concurrent rendering without waiting for production failures.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `<StrictMode>` | The wrapper component; accepts no props. |
| `children` | The component tree to be checked. |
| Root render | `createRoot(...).render(<StrictMode><App /></StrictMode>)` |

**Syntax Rules:**

1. `StrictMode` accepts no props.
2. It can wrap the entire application or any subtree.
3. Only descendants of `StrictMode` are checked; siblings outside it are not.
4. There is no way to opt out of Strict Mode inside a wrapped tree.
5. `StrictMode` renders no visible UI.

**Constraints and Limitations:**

- Checks run only in development; production builds are unaffected.
- Strict Mode cannot catch runtime errors; use error boundaries for that.
- It cannot automatically detect side effects — it makes them more deterministic so you can spot them.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Enabling Strict Mode for the Entire App

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

const root = createRoot(document.getElementById('root'));

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**Expected Output:** All components inside `App` are checked in development. In production, `StrictMode` has no effect.

**Why this output:** Wrapping the root component activates development-only checks for the entire tree. The production build strips the checks.

#### Example 2: Enabling Strict Mode for a Subtree Only

```jsx
import { StrictMode } from 'react';

function App() {
  return (
    <>
      <Header /> {/* Not checked */}
      <StrictMode>
        <main>
          <Sidebar /> {/* Checked */}
          <Content /> {/* Checked */}
        </main>
      </StrictMode>
      <Footer /> {/* Not checked */}
    </>
  );
}
```

**Expected Output:** Only `Sidebar`, `Content`, and their descendants are checked. `Header` and `Footer` are unaffected.

**Why this output:** `StrictMode` applies checks only to its descendants. Components rendered as siblings outside the wrapper are not checked.

### Real-World Cases

- **New applications:** Wrapping the entire app from the start to catch bugs early.
- **Incremental adoption:** Wrapping individual features or routes while migrating a legacy codebase.
- **Third-party integration:** Isolating strict checks to application code while exempting problematic external libraries.

### References

- React — `<StrictMode>` — https://react.dev/reference/react/StrictMode
- React — `<StrictMode>` (React 18) — https://18.react.dev/reference/react/StrictMode
- React — Strict Mode (Legacy) — https://ru.react.js.org/docs/strict-mode.html

---

## 2. Double Rendering

### Definitions

**Core Definition:** Double rendering is Strict Mode’s deliberate second invocation of a component’s render function in development to expose impure rendering behaviour.

**Technical Definition:** In development, Strict Mode calls component function bodies, state initializer functions passed to `useState`, factories passed to `useMemo`, and reducers passed to `useReducer` twice. The first result is discarded; the second is used. This simulates the interruption and restart behaviour of concurrent rendering, where React may abandon and re-execute render work. Impure render logic — mutations, external reads, side effects — produces different results on the second call, revealing the impurity.

**Beginner-Friendly Explanation:** Strict Mode runs your component’s logic twice to see if it produces the same result both times. If it does not, your component is doing something it should not during rendering.

### Purposes

- To detect impure rendering functions that mutate state or read external mutable values.
- To surface side effects (analytics, subscriptions, DOM writes) placed inside render.
- To prepare components for concurrent rendering, where renders may be interrupted and restarted.
- To catch bugs that would otherwise appear only under specific production conditions.

### Syntax Rules and Structure

**Functions Called Twice in Development:**

| Function | Double-invoked? |
|----------|-----------------|
| Component function body | Yes |
| `useState` initializer function | Yes |
| `useMemo` factory function | Yes |
| `useReducer` reducer | Yes |
| `setState` updater function | Yes |
| Class `constructor`, `render`, `shouldComponentUpdate` | Yes (legacy) |

**Syntax Rules:**

1. The first invocation’s result is discarded; the second is used.
2. Functions must be pure: same inputs, same outputs, no side effects.
3. Mutations, external variable reads, and DOM operations inside these functions are bugs.
4. Double rendering occurs only in development; production calls each function once.

### Annotated Code Examples

**❌ Impure render (detected by Strict Mode):**

```jsx
let visitCount = 0;

function Profile({ user }) {
  visitCount++; // Mutates external state during render
  analytics.track('profile_view', user.id); // Side effect during render
  return <h1>{user.name}</h1>;
}
```

**Expected Output:** Strict Mode calls `Profile` twice. `visitCount` increments twice, and analytics fires twice, exposing the impurity.

**Why this output:** Render must be pure. Strict Mode’s double invocation makes the impure behaviour visible. In production, the bug might appear intermittently under concurrent rendering.

**✅ Pure render (safe under Strict Mode):**

```jsx
import { useId, useEffect } from 'react';

function Profile({ user }) {
  const id = useId(); // Stable ID, not a counter

  useEffect(() => {
    analytics.track('profile_view', user.id); // Side effect in effect
  }, [user.id]);

  return <h1>{user.name}</h1>;
}
```

**Expected Output:** `Profile` renders twice with the same output. Analytics fires once per commit, not once per render.

**Why this output:** The render function is pure: it returns the same JSX for the same props and performs no side effects. The effect runs after commit, outside the render phase.

### Real-World Cases

- **Analytics in render:** Double-fired page views are a common Strict Mode finding.
- **Counter IDs:** `let id = 0; id++` inside render produces unstable IDs.
- **External mutable reads:** Reading `window.innerWidth` during render can produce different values across the two calls.

### References

- React — `<StrictMode>`: Fixing bugs found by double rendering — https://react.dev/reference/react/StrictMode
- React — Keeping Components Pure — https://react.dev/learn/keeping-components-pure
- upGrad — React Strict Mode Explained — https://www.upgrad.com/blog/react-strict-mode/
- Stack Overflow — useState initializer called twice — https://stackoverflow.com

---

## 3. Double Effect Invocation

### Definitions

**Core Definition:** Double Effect invocation is Strict Mode’s deliberate extra run of the `useEffect` setup-and-cleanup cycle in development to verify that cleanup logic is correct.

**Technical Definition:** On mount in development, Strict Mode runs each Effect’s setup, then its cleanup, then its setup again. This simulates the unmount–remount cycle that concurrent rendering may perform. If the cleanup function is missing or incomplete, the second setup leaves behind duplicate subscriptions, timers, event listeners, or network requests. The pattern is often described as “mount → unmount → mount”. Effects are not double-invoked in production.

**Beginner-Friendly Explanation:** Strict Mode starts your Effect, stops it, and starts it again. If you forgot to clean up properly, the second start will create duplicates — like two timers instead of one — and you will see the problem immediately.

### Purposes

- To detect missing or incomplete Effect cleanup.
- To prevent memory leaks and duplicate subscriptions in production.
- To ensure Effects are resilient to mount–unmount–mount cycles.
- To prepare components for concurrent rendering and Fast Refresh remounts.

### Syntax Rules and Structure

**Effect Cycle in Development:**

| Step | What happens |
|------|--------------|
| 1 | Setup runs |
| 2 | Cleanup runs |
| 3 | Setup runs again |
| 4 | On real unmount, cleanup runs once more |

**Syntax Rules:**

1. Every Effect that starts something asynchronous must return a cleanup function.
2. Cleanup must undo the setup completely: clear timers, remove listeners, abort requests, close subscriptions.
3. Double invocation happens only in development; production runs the normal single cycle.
4. If an Effect has no cleanup, Strict Mode’s second setup exposes the resulting duplication.

### Annotated Code Examples

**❌ Missing cleanup (exposed by Strict Mode):**

```jsx
useEffect(() => {
  const id = setInterval(() => setTime(new Date()), 1000);
  // No cleanup returned!
}, []);
```

**Expected Output:** Strict Mode runs the setup, then runs a non-existent cleanup, then runs the setup again. Two intervals are now active, and the timer advances twice as fast.

**Why this output:** Without a cleanup, the first interval is never cleared. The second setup creates a duplicate. In production, the bug would appear as a memory leak or double-speed timer after remounts.

**✅ Correct cleanup:**

```jsx
useEffect(() => {
  const id = setInterval(() => setTime(new Date()), 1000);
  return () => clearInterval(id); // ✅ Cleanup mirrors setup
}, []);
```

**Expected Output:** Strict Mode runs setup → cleanup → setup. Only one interval remains active. Production behaves identically.

**Why this output:** The cleanup clears the interval from the first setup before the second setup creates a new one. No duplication occurs.

**✅ Fetch with AbortController:**

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/data`, { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => {
      if (err.name !== 'AbortError') console.error(err);
    });

  return () => controller.abort(); // ✅ Cancels the in-flight request
}, []);
```

**Expected Output:** Strict Mode aborts the first request before the second begins. Only one request completes.

**Why this output:** `AbortController` cancels the first fetch during the simulated unmount, preventing duplicate network requests and stale state updates.

### Real-World Cases

- **Timers and intervals:** Duplicate intervals cause accelerated clocks and resource waste.
- **Event listeners:** Duplicate listeners cause handlers to fire twice per event.
- **WebSocket subscriptions:** Duplicate connections consume server resources and duplicate messages.
- **API requests:** Duplicate requests waste bandwidth and can cause race conditions.

### References

- React — `<StrictMode>`: Fixing bugs found by re-running Effects — https://react.dev/reference/react/StrictMode
- Stack Overflow — StrictMode calls effects twice for cleanup verification — https://stackoverflow.com
- GitHub — Fix double API calls in React StrictMode — https://github.com

---

## 4. Deprecation Warnings

### Definitions

**Core Definition:** Deprecation warnings are Strict Mode’s development-time alerts about APIs that are unsafe, obsolete, or scheduled for removal in future React versions.

**Technical Definition:** Strict Mode checks for usage of deprecated APIs and logs warnings when they are detected. The checked APIs include: unsafe legacy lifecycle methods (`componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate`), legacy string refs, `findDOMNode`, and the legacy context API. These warnings help developers migrate to modern APIs before the deprecated APIs are removed.

**Beginner-Friendly Explanation:** Strict Mode tells you when you are using old React features that will eventually stop working. It is like a warning light on a car dashboard: fix the issue before it becomes a breakdown.

### Purposes

- To alert developers to unsafe legacy lifecycle methods that break under concurrent rendering.
- To encourage migration from legacy string refs to object refs or callback refs.
- To discourage `findDOMNode`, which breaks component abstraction and Fragments.
- To surface the legacy context API before it is removed.
- To ease upgrades to future React versions.

### Syntax Rules and Structure

**Detected Deprecations:**

| Deprecated API | Warning |
|----------------|---------|
| `componentWillMount` | Unsafe lifecycle warning |
| `componentWillReceiveProps` | Unsafe lifecycle warning |
| `componentWillUpdate` | Unsafe lifecycle warning |
| String refs (`ref="myRef"`) | Legacy string ref warning |
| `findDOMNode` | Deprecated `findDOMNode` warning |
| Legacy context API | Legacy context warning |

**Syntax Rules:**

1. Strict Mode compiles a list of all class components using unsafe lifecycles and logs a warning with component information.
2. String ref warnings appear when `ref="string"` syntax is used.
3. `findDOMNode` warnings appear when the method is called.
4. Warnings are development-only; production builds do not emit them.

### Annotated Code Examples

**❌ Unsafe lifecycle (warned by Strict Mode):**

```jsx
class LegacyComponent extends React.Component {
  componentWillMount() { // ⚠️ Unsafe lifecycle
    fetchData();
  }

  render() {
    return <div>{this.props.data}</div>;
  }
}
```

**Expected Output:** Strict Mode logs a warning listing `LegacyComponent` as using an unsafe lifecycle.

**Why this output:** `componentWillMount` is unsafe under concurrent rendering and has been deprecated. Strict Mode detects and warns.

**✅ Migrated lifecycle:**

```jsx
class ModernComponent extends React.Component {
  componentDidMount() { // ✅ Safe lifecycle
    fetchData();
  }

  render() {
    return <div>{this.props.data}</div>;
  }
}
```

**Expected Output:** No unsafe lifecycle warning.

**Why this output:** `componentDidMount` is the supported lifecycle for side effects after mount.

### Real-World Cases

- **Legacy class components:** Migrating lifecycle methods to `componentDidMount` / `componentDidUpdate`.
- **String refs:** Replacing `ref="input"` with `this.inputRef = React.createRef()` or callback refs.
- **`findDOMNode`:** Replacing with direct refs on DOM nodes.

### References

- React — Strict Mode (Legacy): Identifying unsafe lifecycles — https://ru.react.js.org/docs/strict-mode.html
- React — Strict Mode (Legacy): Legacy string ref warning — https://ru.react.js.org/docs/strict-mode.html
- React — Strict Mode (Legacy): Deprecated findDOMNode warning — https://ru.react.js.org/docs/strict-mode.html

---

## 5. Identifying Unsafe Lifecycles

### Definitions

**Core Definition:** Unsafe lifecycle detection is Strict Mode’s ability to flag class components that use legacy lifecycle methods incompatible with concurrent rendering.

**Technical Definition:** Certain legacy lifecycle methods — `componentWillMount`, `componentWillReceiveProps`, and `componentWillUpdate` — are unsafe in async React applications because they may be called multiple times or interleaved with other work. When Strict Mode is enabled, React compiles a list of all class components using these lifecycles and logs a warning with information about the offending components.

**Beginner-Friendly Explanation:** Strict Mode tells you which class components are using old lifecycle methods that can break when React does background rendering.

### Purposes

- To identify class components that will break under concurrent rendering.
- To guide migration to safe lifecycles (`componentDidMount`, `componentDidUpdate`, `getDerivedStateFromProps`).
- To audit third-party libraries that may use unsafe lifecycles internally.
- To prepare the codebase for future React versions.

### Syntax Rules and Structure

**Unsafe vs. Safe Lifecycles:**

| Unsafe (warned) | Safe alternative |
|-----------------|------------------|
| `componentWillMount` | `componentDidMount` / `UNSAFE_componentWillMount` |
| `componentWillReceiveProps` | `getDerivedStateFromProps` / `componentDidUpdate` |
| `componentWillUpdate` | `getSnapshotBeforeUpdate` / `componentDidUpdate` |

**Syntax Rules:**

1. Strict Mode lists all class components using unsafe lifecycles.
2. The warning includes component names and locations.
3. Prefixing with `UNSAFE_` acknowledges the risk but does not fix the underlying issue.
4. Migration to safe lifecycles is the long-term fix.

### Annotated Code Examples

```jsx
class UnsafeComponent extends React.Component {
  componentWillReceiveProps(nextProps) { // ⚠️ Unsafe
    if (nextProps.userId !== this.props.userId) {
      this.fetchUser(nextProps.userId);
    }
  }

  render() {
    return <div>{this.props.user.name}</div>;
  }
}
```

**Expected Output:** Strict Mode logs a warning identifying `UnsafeComponent` as using an unsafe lifecycle.

**Why this output:** `componentWillReceiveProps` is unsafe under concurrent rendering and is flagged by Strict Mode.

### Real-World Cases

- **Third-party class components:** Auditing libraries that have not migrated to safe lifecycles.
- **Legacy codebases:** Identifying class components that need lifecycle migration.
- **React upgrades:** Preparing for versions where unsafe lifecycles are removed.

### References

- React — Strict Mode (Legacy): Identifying unsafe lifecycles — https://ru.react.js.org/docs/strict-mode.html
- React Blog — Update on Async Rendering — https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html

---

## 6. Identifying Side Effects

### Definitions

**Core Definition:** Side effect identification is Strict Mode’s technique of using deterministic double-invocation to reveal unintended side effects in render and Effect code.

**Technical Definition:** Strict Mode cannot automatically detect side effects, but it makes them more deterministic by calling certain functions twice in development. Functions that should be pure — component bodies, state initializers, `useMemo` factories, `useReducer` reducers, and `setState` updaters — are called twice. If a side effect (mutation, external read, subscription, DOM write) is present, the second call produces different observable behaviour, exposing the bug. For Effects, the setup–cleanup–setup cycle exposes missing cleanup.

**Beginner-Friendly Explanation:** Strict Mode cannot see your side effects directly, but it runs your code twice so that the side effects become obvious. If something happens twice when it should happen once, you know there is a bug.

### Purposes

- To reveal side effects hidden inside render functions.
- To expose mutations of external variables or module-level state.
- To detect missing cleanup in Effects.
- To catch impure updater functions in `useState` and `useReducer`.

### Syntax Rules and Structure

**Side Effects and Their Locations:**

| Side effect | Correct location | Strict Mode exposure |
|-------------|------------------|----------------------|
| Mutating external variable | Event handler or Effect | Double mutation |
| Analytics/logging | Effect | Double firing |
| DOM writes | Effect or event handler | Double writes |
| Subscriptions | Effect with cleanup | Duplicate subscriptions |
| Timers | Effect with cleanup | Duplicate timers |
| Network requests | Effect with AbortController | Duplicate requests |

**Syntax Rules:**

1. Render must be pure; move side effects to event handlers or Effects.
2. Effects must return cleanup that mirrors setup.
3. Updater functions must be pure and return the next state without side effects.
4. Strict Mode does not catch every side effect, but it makes many of them observable.

### Annotated Code Examples

**❌ Side effect in `useMemo` factory:**

```jsx
const memoizedValue = useMemo(() => {
  analytics.track('computed'); // ❌ Side effect in a pure function
  return expensiveCompute(data);
}, [data]);
```

**Expected Output:** Strict Mode calls the factory twice, firing analytics twice.

**Why this output:** `useMemo` factories must be pure. Strict Mode’s double invocation reveals the side effect.

**✅ Side effect moved to Effect:**

```jsx
const memoizedValue = useMemo(() => expensiveCompute(data), [data]);

useEffect(() => {
  analytics.track('computed'); // ✅ Side effect in Effect
}, [memoizedValue]);
```

**Expected Output:** Analytics fires once per commit, not once per render.

**Why this output:** The computation is pure; the side effect is in an Effect, where it belongs.

### Real-World Cases

- **Analytics in `useState` initializers:** Double-fired initialization events.
- **DOM reads in `useMemo`:** Layout thrashing and inconsistent measurements.
- **`setState` updaters with side effects:** Duplicate mutations of external state.

### References

- React — `<StrictMode>`: Fixing bugs found by double rendering — https://react.dev/reference/react/StrictMode
- upGrad — React Strict Mode: Detecting unexpected side effects — https://www.upgrad.com/blog/react-strict-mode/
- GitHub — setState updaters must be pure — https://github.com

---

## 7. Strict Mode in Development

### Definitions

**Core Definition:** Strict Mode in development is the active mode where all checks, double-invocations, and warnings operate.

**Technical Definition:** In development builds, Strict Mode activates its full suite of checks: double rendering of pure functions, double Effect cycles, double ref callback cycles, and deprecation warnings. These checks add overhead but provide early bug detection. The behaviour simulates concurrent rendering’s interruption and remount patterns.

**Beginner-Friendly Explanation:** During development, Strict Mode is on and checking everything. This is when you see the extra renders, the double-fired effects, and the warnings.

### Purposes

- To catch bugs early, before code reaches production.
- To simulate concurrent rendering’s interruption and remount patterns.
- To enforce render purity and Effect cleanup discipline.
- To surface deprecated API usage during development.

### Syntax Rules and Structure

**Development Behaviours:**

| Behaviour | Development |
|-----------|-------------|
| Component re-render | Extra render |
| Effect cycle | Extra setup + cleanup |
| Ref callback | Extra invocation |
| Deprecation checks | Active |
| Production impact | None |

**Syntax Rules:**

1. All Strict Mode checks run exclusively in development.
2. The checks add development-time overhead but no production cost.
3. Development behaviour simulates concurrent rendering scenarios.
4. Production builds strip all Strict Mode logic.

### Annotated Code Examples

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    console.log('Effect setup');
    const id = setInterval(() => setSeconds(s => s + 1), 1000);
    return () => {
      console.log('Effect cleanup');
      clearInterval(id);
    };
  }, []);

  return <p>{seconds}</p>;
}
```

**Expected Output (development with Strict Mode):**
```
Effect setup
Effect cleanup
Effect setup
```

**Why this output:** Strict Mode runs setup, cleanup, and setup again on mount. The logs show the complete cycle, verifying that cleanup works.

### Real-World Cases

- **Development debugging:** Using the double logs to verify cleanup correctness.
- **Testing:** Ensuring tests account for Strict Mode’s double invocation.
- **Onboarding:** Teaching new developers about render purity through visible double renders.

### References

- React — `<StrictMode>` — https://react.dev/reference/react/StrictMode
- React — `<StrictMode>` (React 18) — https://18.react.dev/reference/react/StrictMode

---

## 8. Strict Mode in Production

### Definitions

**Core Definition:** Strict Mode in production is the inactive state where none of the development checks run and `StrictMode` has no effect on application behaviour or performance.

**Technical Definition:** Production builds of React strip all Strict Mode logic. Components render once, Effects run a single setup-and-cleanup cycle, ref callbacks run once, and no deprecation warnings are emitted. `StrictMode` renders no UI and adds no runtime overhead. Keeping `StrictMode` in the codebase for production is safe and recommended; it is not a build flag and does not slow down the application.

**Beginner-Friendly Explanation:** Strict Mode automatically turns itself off in production. Your users never experience the double renders or extra checks. Keeping the wrapper in your code costs nothing.

### Purposes

- To confirm that Strict Mode does not affect production performance.
- To decide whether to keep `StrictMode` in the codebase (the answer is yes).
- To understand that production behaviour is the “real” behaviour.
- To avoid unnecessary conditional removal of `StrictMode`.

### Syntax Rules and Structure

**Production Behaviour:**

| Behaviour | Production |
|-----------|------------|
| Component render | Once |
| Effect cycle | Single setup + cleanup |
| Ref callback | Once |
| Deprecation warnings | None |
| Performance overhead | None |

**Syntax Rules:**

1. Strict Mode checks are development-only and do not impact production builds.
2. `StrictMode` renders no visible UI and adds no runtime cost.
3. Do not conditionally remove `StrictMode` based on `NODE_ENV`; leave it in place.
4. Production builds behave as if `StrictMode` were not present.

### Annotated Code Examples

```jsx
// ✅ Keep StrictMode in the codebase for all environments
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**Expected Output:** Development shows double renders and extra Effect cycles; production renders normally with no overhead.

**Why this output:** The production build strips Strict Mode logic. The wrapper remains in source but has no runtime effect.

### Real-World Cases

- **All React applications:** Keeping `StrictMode` in production source is standard practice.
- **Performance-sensitive apps:** Confirming no production overhead from `StrictMode`.
- **CI/CD:** Ensuring production builds do not accidentally disable `StrictMode`.

### References

- React — `<StrictMode>`: Caveats — https://react.dev/reference/react/StrictMode
- React — Strict Mode (Legacy): Development-only note — https://ru.react.js.org/docs/strict-mode.html
- Mintlify — StrictMode: Development-only — https://mintlify.wiki

---

## 9. Strict Mode Best Practices

### Definitions

**Core Definition:** Strict Mode best practices are the guidelines for using `StrictMode` effectively — enabling it broadly, writing pure components, cleaning up Effects, and migrating away from deprecated APIs.

**Technical Definition:** Best practices centre on: wrapping the entire application (or the largest practical subtree) in `StrictMode`; keeping render functions pure; returning complete cleanup from every Effect; memoizing Promises and avoiding side effects in `useMemo` and `useState` initializers; migrating unsafe lifecycles and legacy APIs; and treating Strict Mode warnings as bugs to fix, not noise to suppress. Strict Mode is a development aid, not a production concern.

**Beginner-Friendly Explanation:** Turn Strict Mode on for your whole app. Fix the warnings it gives you instead of ignoring them. Keep your render functions pure and always clean up after your Effects. These habits make your app more reliable and ready for React’s future.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Wrap the entire app** | Enable `StrictMode` at the root for maximum coverage. |
| **Keep render pure** | No mutations, external reads, or side effects in render. |
| **Return complete cleanup** | Every Effect that starts something must clean it up. |
| **Abort fetches** | Use `AbortController` in Effect cleanup. |
| **Clear timers and listeners** | Mirror every `setInterval`/`addEventListener` with cleanup. |
| **Pure state initializers** | No side effects in `useState` initializer functions. |
| **Pure `useMemo` factories** | Move side effects out of memoization. |
| **Migrate unsafe lifecycles** | Replace `componentWillMount` etc. with safe alternatives. |
| **Replace legacy APIs** | Migrate string refs and `findDOMNode` to modern refs. |
| **Fix warnings, don’t suppress** | Treat Strict Mode warnings as actionable bugs. |
| **Do not remove StrictMode for production** | It has no production cost; keep it permanently. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
import { StrictMode, useState, useEffect } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(setResults)
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err);
      });

    return () => controller.abort(); // ✅ Cleanup aborts stale request
  }, [query]);

  return (
    <input value={query} onChange={e => setQuery(e.target.value)} />
  );
}

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**❌ Bad Practices:**

```jsx
// ❌ No cleanup: Strict Mode exposes duplicate requests
useEffect(() => {
  fetch('/api/data').then(res => res.json()).then(setData);
}, []);

// ❌ Side effect in render: double analytics events
function BadComponent() {
  analytics.track('render'); // Fires twice in development
  return <div />;
}

// ❌ Removing StrictMode to silence warnings
// (StrictMode is not the problem; the code is)
```

**Expected Output:** The good practices produce predictable, leak-free behaviour. The bad practices cause duplicate requests, double analytics, and hidden bugs.

**Why this output:** Strict Mode does not create bugs; it reveals them. Fixing the underlying code makes the warnings disappear and improves production reliability.

### Real-World Cases

- **New projects:** Enabling `StrictMode` from day one avoids retrofitting pain.
- **Legacy migrations:** Wrapping subtrees incrementally while fixing warnings.
- **Third-party libraries:** Wrapping only application code if a library is incompatible.

### References

- React — `<StrictMode>` — https://react.dev/reference/react/StrictMode
- upGrad — React Strict Mode Explained: Best Practices — https://www.upgrad.com/blog/react-strict-mode/
- React — Keeping Components Pure — https://react.dev/learn/keeping-components-pure

---

## References

- React — `<StrictMode>` — https://react.dev/reference/react/StrictMode
- React — `<StrictMode>` (Markdown) — https://react.dev/reference/react/StrictMode.md
- React — `<StrictMode>` (React 18) — https://18.react.dev/reference/react/StrictMode
- React — Strict Mode (Legacy) — https://ru.react.js.org/docs/strict-mode.html
- React — Strict Mode (Legacy, raw) — https://raw.githubusercontent.com/reactjs/react.dev/fae6d6b2ebd6d4e15c2b21ea30308bbdae266540/content/docs/strict-mode.md
- React Blog — Update on Async Rendering — https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html
- React — Keeping Components Pure — https://react.dev/learn/keeping-components-pure
- upGrad — React Strict Mode Explained: How to Apply It in Your Projects — https://www.upgrad.com/blog/react-strict-mode/
- Mintlify — StrictMode — https://mintlify.wiki
- Stack Overflow — StrictMode double render and effect behavior — https://stackoverflow.com
- GitHub — setState updaters must be pure — https://github.com
- GitHub — Fix double API calls in React StrictMode — https://github.com