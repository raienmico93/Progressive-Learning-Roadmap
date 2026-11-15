# React Hook Pitfalls — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Hook pitfalls are common mistakes, anti-patterns, and runtime errors that arise from misusing React Hooks — violations of the Rules of Hooks, mismanaged dependencies, improper effect cleanup, and unnecessary memoization.

**Technical Definition:** Hook pitfalls stem from three sources: (1) violations of the Rules of Hooks, which govern where and how Hooks may be called; (2) misalignment between React's render cycle and JavaScript's closure semantics, producing stale closures and race conditions; and (3) incorrect lifecycle management of side effects, leading to memory leaks and infinite loops. These issues are often silent in simple demos but surface as hard-to-trace bugs in production applications.

**Beginner-Friendly Explanation:** React Hooks have strict rules and subtle behaviors. If you break the rules, React crashes. If you get the dependencies wrong, your data becomes stale. If you forget to clean up, your app slows down and leaks memory. This cheat sheet covers all the ways Hooks can go wrong — and how to fix them.

### Key Characteristics

- **Rules-of-Hooks violations cause immediate crashes** — React throws "Rendered more hooks than during the previous render" or "Invalid hook call."
- **Stale closures are silent** — the component renders with outdated values without throwing any error.
- **Missing dependencies cause effects to skip re-runs** — data never updates when it should.
- **Over-memoization adds overhead** — `useMemo` and `useCallback` cost more than they save when misapplied.
- **Effect races produce wrong UI** — a slow request overwrites a fast one.
- **Missing cleanup leaks memory** — event listeners, timers, and subscriptions accumulate.

### Prerequisites

- A React application using Hooks (v16.8+).
- `eslint-plugin-react-hooks` installed and configured (essential for catching violations and missing dependencies).
- React DevTools browser extension (for inspecting hook state).
- Understanding of JavaScript closures and asynchronous operations.

### Related Programming Areas

- **Rules of Hooks** — the foundational constraints that prevent most violations.
- **Dependency arrays** — the mechanism that controls when effects, memos, and callbacks re-execute.
- **Suspense and concurrent rendering** — modern patterns that interact with effect timing.
- **Memory management** — cleanup functions and abort controllers.
- **Performance profiling** — React DevTools Profiler for identifying unnecessary renders.

### Core Concepts / Features

---

## 1. Hook Rules Violations

### Definitions

**Core Definition:** Hook rules violations are calls to React Hooks that break React's two mandatory rules: only call Hooks at the top level, and only call Hooks from React functions.

**Technical Definition:** React relies on the **order** in which Hooks are called to associate each Hook call with its corresponding state slot. If Hooks are called conditionally, in loops, after early returns, or inside non-React functions, the call order changes between renders, and React loses track of which state belongs to which Hook. This produces runtime errors such as "Rendered more hooks than during the previous render" or "Invalid hook call".

**Beginner-Friendly Explanation:** React expects your Hooks to run in the exact same order on every render. If you skip a Hook sometimes or call it extra times, React gets confused about which state goes where — and your app crashes.

### Purposes

- To understand why React enforces Hook call order.
- To recognize the patterns that violate the rules.
- To use ESLint to catch violations before runtime.

### Syntax Rules and Structure

**The Two Rules:**

1. **Only call Hooks at the top level.** Do not call Hooks inside conditions, loops, nested functions, `try`/`catch`/`finally` blocks, or after early returns.
2. **Only call Hooks from React functions.** Call Hooks from function components or custom Hooks — never from regular JavaScript functions, class components, or event handlers.

**Valid vs. Invalid Locations:**

| Location | Valid? |
|----------|--------|
| Top level of a function component | ✅ Yes |
| Top level of a custom Hook | ✅ Yes |
| Inside `if`, `for`, `while`, `try/catch` | ❌ No |
| After a conditional `return` | ❌ No |
| Inside event handlers or callbacks | ❌ No |
| Inside `useMemo`, `useEffect`, or `useReducer` callbacks | ❌ No |
| Inside class component methods | ❌ No |

**Constraints and Limitations:**

- The `use` hook (React 19) is **exempt** from the loop, conditional, and early-return checks — it may be called conditionally.
- Custom Hooks may call other Hooks (that is their purpose), but must themselves be called only from React functions.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Violating the Rules — Conditional Hook

```jsx
// ❌ VIOLATION: Hook called conditionally
function BadComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // Hook inside if
  }
  return <div>Content</div>;
}
```

**Expected behavior:** React throws an error on the second render if `isLoggedIn` changes, because the number of Hooks differs between renders.

**Correct pattern:**

```jsx
// ✅ CORRECT: Hook always called, condition applied to the value
function GoodComponent({ isLoggedIn }) {
  const [user, setUser] = useState(null); // Top level
  if (!isLoggedIn) return <div>Please log in</div>;
  return <div>Welcome, {user?.name}</div>;
}
```

#### Example 2: Violating the Rules — Hook After Early Return

```jsx
// ❌ VIOLATION: Hook called after a conditional return
function BadComponent({ data }) {
  if (!data) return <p>Loading...</p>;
  const [processed, setProcessed] = useState(null); // Hook after return
  return <p>{processed}</p>;
}
```

**Correct pattern:**

```jsx
// ✅ CORRECT: All Hooks before any early return
function GoodComponent({ data }) {
  const [processed, setProcessed] = useState(null); // Top level
  if (!data) return <p>Loading...</p>;
  return <p>{processed}</p>;
}
```

### Real-World Cases

- **Auth guards:** A common mistake is calling `useContext` only when `isAuthenticated` is true — this violates the rules. Instead, call the Hook unconditionally and branch on the value.
- **Feature flags:** Conditionally calling `useFeatureFlag` only when a feature is enabled causes crashes when the flag changes.
- **ESLint protection:** The `eslint-plugin-react-hooks` `rules-of-hooks` rule catches all these violations at development time.

### References

- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- React (18) — Rules of Hooks — https://18.react.dev/reference/rules/rules-of-hooks
- ESLint Plugin React Hooks — https://www.npmjs.com/package/eslint-plugin-react-hooks

---

## 2. Conditional Hooks

### Definitions

**Core Definition:** Conditional hooks are Hook calls placed inside `if`/`else` statements, ternary expressions, or short-circuit operators (`&&`, `||`), causing the Hook to be called on some renders but not others.

**Technical Definition:** Conditional Hook calls violate the first Rule of Hooks. Because React matches Hooks to state slots by call order, a Hook that is skipped on one render shifts the order of all subsequent Hooks, causing React to associate state values with the wrong Hooks. This results in runtime errors or silently corrupted state.

**Beginner-Friendly Explanation:** You cannot put a Hook inside an `if` statement. If the condition changes, the Hook might disappear, and React will lose track of which state belongs to which Hook.

### Purposes

- To recognize the most common form of Hook rules violation.
- To learn the correct pattern: call the Hook unconditionally, then use its value conditionally.
- To understand why the `use` hook (React 19) is exempt from this rule.

### Syntax Rules and Structure

**Violation:**

```jsx
if (condition) {
  useEffect(() => { /* ... */ }, []);
}
```

**Correct pattern:**

```jsx
useEffect(() => {
  if (condition) {
    /* ... */
  }
}, [condition]);
```

**Exception (React 19):**

```jsx
if (condition) {
  const value = use(somePromiseOrContext); // ✅ Allowed with `use`
}
```

**Constraints and Limitations:**

- Only the `use` hook is exempt. All other Hooks (`useState`, `useEffect`, `useMemo`, etc.) must be called unconditionally.
- Even with `use`, the condition should be based on props or state, not on the promise's internal status.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Conditional useEffect (Violation) vs. Conditional Logic Inside Effect (Correct)

```jsx
// ❌ VIOLATION: useEffect called only when enabled
function Bad({ enabled }) {
  if (enabled) {
    useEffect(() => {
      console.log('Effect running');
    }, []);
  }
  return <div>Content</div>;
}

// ✅ CORRECT: useEffect always called; condition inside
function Good({ enabled }) {
  useEffect(() => {
    if (enabled) {
      console.log('Effect running');
    }
  }, [enabled]);
  return <div>Content</div>;
}
```

**Expected behavior:** `Bad` crashes when `enabled` toggles. `Good` logs only when `enabled` is true, without violating Hook order.

#### Example 2: Conditional useContext (Violation) vs. Unconditional Call

```jsx
// ❌ VIOLATION
function Bad({ showTheme }) {
  if (showTheme) {
    const theme = useContext(ThemeContext); // Hook inside condition
  }
  return <div>Content</div>;
}

// ✅ CORRECT
function Good({ showTheme }) {
  const theme = useContext(ThemeContext); // Always called
  return <div className={showTheme ? theme : ''}>Content</div>;
}
```

### Real-World Cases

- **Feature toggles:** Developers often try to call a Hook only when a feature flag is enabled — this breaks when the flag changes.
- **Auth context:** Reading user context only when logged in — must be done unconditionally, then branched on the value.
- **Lazy initialization:** Trying to call `useState` only for certain props — always call all Hooks first.

### References

- React — Rules of Hooks: Only call Hooks at the top level — https://react.dev/reference/rules/rules-of-hooks
- React (18) — Rules of Hooks — https://18.react.dev/reference/rules/rules-of-hooks

---

## 3. Hooks in Loops

### Definitions

**Core Definition:** Calling Hooks inside `for`, `while`, or `do-while` loops, or inside array iteration methods (`.map`, `.forEach`), causing a variable number of Hook calls per render.

**Technical Definition:** Loops that call Hooks produce a **dynamic number of Hook calls** — the count depends on the loop length. React cannot match a dynamic sequence of Hooks to its fixed internal state slots, producing the error "Rendered more hooks than during the previous render." The correct pattern is to call the Hook once and manage the collection via state, not by looping over Hooks.

**Beginner-Friendly Explanation:** You cannot call a Hook inside a loop because the number of Hooks would change depending on how many items are in the loop. React needs a fixed number of Hooks on every render.

### Purposes

- To understand why loops and Hooks are incompatible.
- To learn the correct pattern: manage collections in a single state variable.
- To recognize the `use` hook exemption for loops (React 19).

### Syntax Rules and Structure

**Violation:**

```jsx
for (let i = 0; i < items.length; i++) {
  const [value, setValue] = useState(null); // ❌ Hook in loop
}
```

**Correct pattern:**

```jsx
const [values, setValues] = useState(() => items.map(() => null)); // One Hook
```

**Constraints and Limitations:**

- The `use` hook is exempt and may be called inside loops (e.g., reading the same Context in each iteration).
- Even with `use`, calling it in a loop with different Promises is unusual and should be carefully justified.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Loop of useState (Violation) vs. Single Array State

```jsx
// ❌ VIOLATION: One useState per item in a loop
function BadList({ items }) {
  return (
    <ul>
      {items.map((item) => {
        const [selected, setSelected] = useState(false); // Hook in .map
        return <li key={item.id}>{item.name}</li>;
      })}
    </ul>
  );
}

// ✅ CORRECT: Single state for the entire collection
function GoodList({ items }) {
  const [selectedIds, setSelectedIds] = useState(new Set());

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

**Expected behavior:** `BadList` crashes when the number of items changes. `GoodList` manages all selection state in one Hook.

### Real-World Cases

- **Dynamic forms:** Rendering a variable number of form fields — use a single state object or array, not one `useState` per field.
- **List item selection:** Tracking selection for a list — use a `Set` or object in state, not per-item Hooks.
- **Accordion panels:** Managing open/closed state for dynamic panels — use one state object keyed by ID.

### References

- React — Rules of Hooks: Only call Hooks at the top level — https://react.dev/reference/rules/rules-of-hooks
- ESLint rules-of-hooks — https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/src/rules/RulesOfHooks.ts

---

## 4. Hooks in Nested Functions

### Definitions

**Core Definition:** Calling Hooks inside nested functions (functions defined inside a component), including event handlers, callbacks, `setTimeout` callbacks, and functions passed to `useMemo`/`useEffect`.

**Technical Definition:** Nested functions are not React components or custom Hooks. Hooks called inside them are not associated with the component's render cycle. React cannot track their state or order. This produces the "Invalid hook call" error. The correct pattern is to call Hooks at the top level of the component, and pass their values or setters into nested functions.

**Beginner-Friendly Explanation:** Hooks only work at the top level of a component — not inside functions you define inside the component. If you need a Hook's value inside a callback, call the Hook outside and use the value inside the callback.

### Purposes

- To distinguish between component-level Hook calls and nested-function misuse.
- To learn the correct pattern: call Hooks at the top, use their values in callbacks.
- To recognize the difference between a custom Hook (valid) and a nested function (invalid).

### Syntax Rules and Structure

**Violation:**

```jsx
function Component() {
  function handleClick() {
    const [count, setCount] = useState(0); // ❌ Invalid hook call
  }
  return <button onClick={handleClick}>Click</button>;
}
```

**Correct pattern:**

```jsx
function Component() {
  const [count, setCount] = useState(0); // ✅ Top level

  function handleClick() {
    setCount(count + 1); // ✅ Use the setter inside the callback
  }

  return <button onClick={handleClick}>Click</button>;
}
```

**Constraints and Limitations:**

- Custom Hooks are functions that start with `use` and are called at the top level — they are not "nested functions" in the violating sense.
- Hooks cannot be called inside `useMemo`, `useReducer`, or `useEffect` callbacks.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Hook Inside Event Handler (Violation)

```jsx
// ❌ VIOLATION: useState inside event handler
function Bad() {
  function handleClick() {
    const [clicked, setClicked] = useState(false); // Invalid hook call
    setClicked(true);
  }
  return <button onClick={handleClick}>Click</button>;
}

// ✅ CORRECT: useState at top level
function Good() {
  const [clicked, setClicked] = useState(false); // Top level

  function handleClick() {
    setClicked(true); // Use setter
  }

  return <button onClick={handleClick}>Click</button>;
}
```

**Expected behavior:** `Bad` throws "Invalid hook call" when clicked. `Good` works correctly.

#### Example 2: Hook Inside setTimeout (Violation)

```jsx
// ❌ VIOLATION: Hook inside setTimeout callback
useEffect(() => {
  setTimeout(() => {
    const [visible, setVisible] = useState(false); // Invalid
  }, 1000);
}, []);

// ✅ CORRECT: Hook at top level, value used inside timeout
const [visible, setVisible] = useState(false);

useEffect(() => {
  const timer = setTimeout(() => {
    setVisible(false); // Use setter
  }, 1000);
  return () => clearTimeout(timer);
}, []);
```

### Real-World Cases

- **Data fetching:** Calling `useState` inside a `fetch().then()` callback — always call state Hooks at the component top level.
- **Event handlers:** Trying to use `useContext` inside an `onClick` handler — read context at the top level and close over the value.
- **Async callbacks:** Hooks inside `async` functions that are not custom Hooks — always invalid.

### References

- React — Rules of Hooks: Do not call Hooks in event handlers — https://react.dev/reference/rules/rules-of-hooks
- React — Invalid Hook Call Warning — https://react.dev/warnings/invalid-hook-call-warning

---

## 5. Stale Closures

### Definitions

**Core Definition:** A stale closure occurs when a callback or effect captures a variable's value at the time it was created, and later uses that outdated value even after the variable has changed.

**Technical Definition:** JavaScript closures capture variables by reference at the time of function creation. In React, each render creates a new scope with new state values. An effect or callback created during an earlier render closes over that render's state — not the latest state. If the effect is not re-created (because its dependency array omits the changing value), the closure continues to use the stale value.

**Beginner-Friendly Explanation:** A stale closure is a function that "remembers" an old value and never sees updates. For example, a `setInterval` created once keeps logging `0` even after the counter has increased, because it captured the initial `count` value and never re-created.

### Purposes

- To recognize when an effect or callback is using outdated values.
- To learn the three fixes: correct dependencies, functional updates, and refs.
- To understand why the `exhaustive-deps` lint rule exists.

### Syntax Rules and Structure

**Classic stale closure:**

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count); // Always logs initial count
  }, 1000);
  return () => clearInterval(id);
}, []); // ❌ Missing `count` dependency
```

**Fix 1: Add the dependency:**

```jsx
useEffect(() => {
  const id = setInterval(() => console.log(count), 1000);
  return () => clearInterval(id);
}, [count]); // ✅ Re-creates interval when count changes
```

**Fix 2: Functional update (when only updating state):**

```jsx
setCount((c) => c + 1); // ✅ Always receives latest state from React
```

**Fix 3: Ref for latest value without re-creating:**

```jsx
const countRef = useRef(count);
useEffect(() => { countRef.current = count; });

useEffect(() => {
  const id = setInterval(() => console.log(countRef.current), 1000);
  return () => clearInterval(id);
}, []); // ✅ Stable effect, always reads latest ref value
```

**Constraints and Limitations:**

- Fix 2 is only applicable when you only need the previous value to compute the next state.
- Fix 3 is for callbacks that must remain stable but need the latest value (e.g., event handlers passed to third-party libraries).
- The `exhaustive-deps` ESLint rule catches most stale closures automatically — enable it.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Stale Closure in setInterval

```jsx
import { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log('Count is:', count); // ❌ Always 0
    }, 1000);
    return () => clearInterval(id);
  }, []); // Missing count

  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Expected output:** The console logs `Count is: 0` every second, regardless of the button clicks.

**Why:** The interval callback captured `count = 0` from the initial render. Adding `[count]` to the dependency array re-creates the interval whenever `count` changes, capturing the fresh value.

#### Example 2: Fix with Functional Update

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount((c) => c + 1); // ✅ Never reads stale count
  }, 1000);
  return () => clearInterval(id);
}, []); // Empty array is now fine
```

**Why:** The functional form of `setCount` receives the latest state directly from React, bypassing the closure entirely.

### Real-World Cases

- **Timer components:** A countdown timer that always shows the initial time because the interval captured a stale value.
- **Event handlers in third-party libraries:** A chart library's click handler captured a stale filter value.
- **Debounced search:** A debounced function that always searches for the initial query.

### References

- CoreUI — How to fix stale closures in React hooks — https://coreui.io/answers/how-to-fix-stale-closures-in-react-hooks/
- React — exhaustive-deps — https://zh-hans.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 6. Missing Dependencies

### Definitions

**Core Definition:** Missing dependencies occur when a reactive value (state, prop, or variable) used inside `useEffect`, `useMemo`, or `useCallback` is omitted from the dependency array, causing the Hook to use an outdated value.

**Technical Definition:** React's `exhaustive-deps` ESLint rule validates that dependency arrays contain every reactive value referenced inside the Hook. Omitting a dependency means React will not re-run the effect or recalculate the memo/callback when that value changes, producing a stale closure.

**Beginner-Friendly Explanation:** If your effect uses a variable, that variable must be in the dependency array. If you leave it out, the effect will never see updates to that variable.

### Purposes

- To understand why every reactive value must be listed.
- To learn to fix missing dependencies without causing infinite loops.
- To use the `exhaustive-deps` ESLint rule effectively.

### Syntax Rules and Structure

**Violation:**

```jsx
useEffect(() => {
  console.log(count); // Uses count
}, []); // ❌ Missing `count`
```

**Correct:**

```jsx
useEffect(() => {
  console.log(count);
}, [count]); // ✅
```

**Common scenarios and fixes:**

| Scenario | Problem | Fix |
|----------|---------|-----|
| Effect reads a prop | Prop missing from deps | Add the prop |
| Effect reads a state variable | State missing from deps | Add the state |
| Effect calls a function defined in component | Function is a new reference every render | Move function inside effect, or wrap in `useCallback` |
| Effect uses an object/array | Object is a new reference every render | Destructure primitives, or memoize with `useMemo` |
| Effect should run only once on mount | Linter complains about missing deps | Include deps (recommended) or use a ref guard |

**Constraints and Limitations:**

- Disabling `react-hooks/exhaustive-deps` without strong justification hides real bugs.
- Adding a function dependency can cause infinite loops if the function is not memoized.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Missing Prop Dependency

```jsx
// ❌ Missing `userId`
useEffect(() => {
  fetchUser(userId).then(setUser);
}, []); // Never re-fetches when userId changes

// ✅ Correct
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]); // Re-fetches when userId changes
```

**Expected output:** With the fix, changing `userId` triggers a new fetch. Without it, the component always shows the first user's data.

#### Example 2: Function Dependency Causing Infinite Loop

```jsx
// ❌ New function every render → effect runs every render → infinite loop
const logItems = () => console.log(items);
useEffect(() => {
  logItems();
}, [logItems]);

// ✅ Fix 1: Move function inside effect
useEffect(() => {
  const logItems = () => console.log(items);
  logItems();
}, [items]);

// ✅ Fix 2: Wrap in useCallback
const logItems = useCallback(() => console.log(items), [items]);
useEffect(() => { logItems(); }, [logItems]);
```

**Why:** The original `logItems` is re-created on every render. React sees a new function reference in the dependency array and re-runs the effect — which triggers another render, creating another function, and so on.

### Real-World Cases

- **Data fetching:** Forgetting to include a filter or pagination parameter in the dependency array, so the list never updates.
- **Subscriptions:** Missing a channel ID dependency, so the component subscribes to the wrong channel.
- **Event listeners:** Missing a handler dependency, so the listener always calls the old handler.

### References

- React — exhaustive-deps — https://zh-hans.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React — Removing Effect Dependencies — https://react.dev/learn/removing-effect-dependencies

---

## 7. Overuse of `useEffect`

### Definitions

**Core Definition:** Overuse of `useEffect` is the anti-pattern of using effects for logic that should be computed during render, derived from props/state, or handled in event handlers.

**Technical Definition:** `useEffect` is designed for **synchronizing with external systems** — subscriptions, timers, DOM mutations, network requests. It is not a general-purpose "do something after render" tool. Using effects to derive state from props, transform data for rendering, or handle user events adds unnecessary render cycles, introduces race conditions, and makes data flow harder to trace.

**Beginner-Friendly Explanation:** Not everything belongs in `useEffect`. If you can compute a value directly from props or state, do it during render — no effect needed. If a user action triggers logic, put it in the event handler, not an effect.

### Purposes

- To distinguish between genuine side effects and derived computations.
- To eliminate unnecessary effects that add complexity and render cycles.
- To replace effect-based state synchronization with derived values.

### Syntax Rules and Structure

**Anti-patterns to avoid:**

| Anti-pattern | Problem | Better approach |
|--------------|---------|-----------------|
| Syncing props to state with `useEffect` | Extra render, stale state | Derive during render or use a `key` to reset |
| Deriving filtered/sorted lists in `useEffect` | Extra render, duplicated state | Compute with `useMemo` or directly during render |
| Handling user events in `useEffect` | Effect runs on every relevant change, not on the event | Handle in the event handler |
| Resetting state on prop change | Effect-based reset causes double render | Use `key` on the component |

**Constraints and Limitations:**

- Some effects are unavoidable: subscriptions, timers, manual DOM manipulation, and network requests that must happen on mount.
- React 19's `use` hook and Server Components reduce the need for client-side effect-based fetching.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Deriving State with useEffect (Anti-Pattern)

```jsx
// ❌ Anti-pattern: derived state in useEffect
function Bad({ items, searchTerm }) {
  const [filtered, setFiltered] = useState([]);

  useEffect(() => {
    setFiltered(items.filter((i) => i.name.includes(searchTerm)));
  }, [items, searchTerm]);

  return <List data={filtered} />;
}

// ✅ Correct: derive during render
function Good({ items, searchTerm }) {
  const filtered = items.filter((i) => i.name.includes(searchTerm));
  return <List data={filtered} />;
}

// ✅ Better: memoize if expensive
function Better({ items, searchTerm }) {
  const filtered = useMemo(
    () => items.filter((i) => i.name.includes(searchTerm)),
    [items, searchTerm]
  );
  return <List data={filtered} />;
}
```

**Why:** The `useEffect` version causes an extra render (first render with empty `filtered`, then a re-render after the effect sets it). The direct computation renders the correct data immediately.

#### Example 2: Resetting State on Prop Change (Anti-Pattern)

```jsx
// ❌ Effect-based reset
function Bad({ userId }) {
  const [comment, setComment] = useState('');
  useEffect(() => { setComment(''); }, [userId]); // Double render
  return <textarea value={comment} onChange={(e) => setComment(e.target.value)} />;
}

// ✅ Key-based reset
function Good({ userId }) {
  return <CommentBox key={userId} />;
}

function CommentBox() {
  const [comment, setComment] = useState(''); // Fresh state on key change
  return <textarea value={comment} onChange={(e) => setComment(e.target.value)} />;
}
```

### Real-World Cases

- **Search filtering:** Filtering a list based on a search term — compute during render or with `useMemo`, not in an effect.
- **Form reset:** Resetting a form when the user changes — use `key` instead of an effect.
- **Derived totals:** Computing a cart total — derive directly from items, no effect needed.

### References

- rtCamp — React Hooks Best Practices: useEffect isn't for everything — https://rtcamp.com/handbook/react-best-practices/hooks/
- TechRepublic — React JS Best Practices: Anti-pattern of wrapping everything in useEffect — https://www.techrepublic.com/forums/discussions/react-js-best-practices-pitfalls-performance-tweaks-for-2025/

---

## 8. Overuse of `useMemo`

### Definitions

**Core Definition:** Overuse of `useMemo` is the practice of wrapping cheap computations in memoization "just to be safe," adding overhead without measurable performance benefit.

**Technical Definition:** `useMemo` caches the result of a computation between renders. It has its own cost: React must compare dependencies and store the cached value. For cheap calculations (simple arithmetic, small array filters), this overhead exceeds the cost of recomputation. Over-memoization also creates dependency chains, increases code complexity, and can mask underlying performance issues.

**Beginner-Friendly Explanation:** `useMemo` is for expensive calculations. If your calculation is fast, memoizing it actually makes your app slower and your code harder to read.

### Purposes

- To identify when `useMemo` genuinely helps versus when it adds overhead.
- To apply memoization only when profiling shows a real benefit.
- To avoid the complexity of unnecessary dependency chains.

### Syntax Rules and Structure

**When `useMemo` helps:**

| Scenario | Why |
|----------|-----|
| Expensive computation (large array sort/filter, complex math) | Avoids recomputation on unrelated renders |
| Referential stability for a memoized child's prop | Prevents the child from re-rendering |
| Stabilizing an object/array used as a dependency in another Hook | Prevents infinite loops |

**When `useMemo` does not help (and can hurt):**

| Scenario | Why |
|----------|-----|
| Cheap calculations (simple arithmetic, small arrays) | Overhead exceeds savings |
| Primitive values (`string`, `number`, `boolean`) | No referential identity issue |
| No measured performance problem | Premature optimization |

**Constraints and Limitations:**

- React Compiler (React 19+) can automatically memoize many patterns, reducing the need for manual `useMemo`.
- Every `useMemo` adds a dependency array that must be maintained correctly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Unnecessary useMemo (Anti-Pattern)

```jsx
// ❌ Unnecessary: simple arithmetic
function Bad({ price, quantity }) {
  const total = useMemo(() => price * quantity, [price, quantity]);
  return <p>Total: {total}</p>;
}

// ✅ Correct: no memo needed for cheap computation
function Good({ price, quantity }) {
  const total = price * quantity;
  return <p>Total: {total}</p>;
}
```

**Why:** `price * quantity` is a primitive multiplication — the overhead of `useMemo` (dependency comparison, cache storage) exceeds the cost of recomputing it.

#### Example 2: Justified useMemo (Expensive Computation)

```jsx
// ✅ Justified: expensive filtering and sorting of thousands of items
function Good({ items, searchTerm, sortOrder }) {
  const processed = useMemo(() => {
    return items
      .filter((i) => i.name.toLowerCase().includes(searchTerm.toLowerCase()))
      .sort((a, b) => (sortOrder === 'asc' ? a.price - b.price : b.price - a.price));
  }, [items, searchTerm, sortOrder]);

  return <List data={processed} />;
}
```

**Why:** Filtering and sorting a large array on every render (even when unrelated state changes) is expensive. `useMemo` caches the result until the relevant inputs change.

### Real-World Cases

- **Large data tables:** Sorting/filtering thousands of rows — justified.
- **Simple counters:** Computing a display string from a number — unnecessary.
- **Dependency stabilization:** Creating an options object for a `useEffect` dependency — sometimes necessary, but prefer destructuring to primitives.

### References

- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- Steve Kinney — Avoiding Over-Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- DebugBear — Improve React Performance With useMemo And useCallback — https://www.debugbear.com/blog/react-usememo-usecallback

---

## 9. Overuse of `useCallback`

### Definitions

**Core Definition:** Overuse of `useCallback` is the practice of wrapping every function in memoization, even when the function is not passed to a memoized child or used as a dependency.

**Technical Definition:** `useCallback` returns a memoized version of a callback function. It is only beneficial when the function's referential identity matters — specifically when it is passed to a `React.memo` child or used in another Hook's dependency array. Wrapping functions that are passed to regular DOM elements or non-memoized children provides no benefit and adds memory and comparison overhead.

**Beginner-Friendly Explanation:** `useCallback` only helps when a memoized component or a Hook cares whether the function is the same as before. If you're just passing a click handler to a regular `<button>`, wrapping it in `useCallback` does nothing useful.

### Purposes

- To identify the narrow cases where `useCallback` is beneficial.
- To avoid the "defensive memoization" anti-pattern.
- To understand the relationship between `useCallback`, `React.memo`, and dependency arrays.

### Syntax Rules and Structure

**When `useCallback` helps:**

| Scenario | Why |
|----------|-----|
| Passing a function to a `React.memo` child | Prevents child re-render when the parent re-renders for unrelated reasons |
| Using a function as a dependency in `useEffect` | Prevents the effect from re-running on every render |
| Passing a stable function to a third-party library that relies on identity | Prevents unnecessary resubscriptions |

**When `useCallback` does not help:**

| Scenario | Why |
|----------|-----|
| Function passed to a regular DOM element (`<button onClick={...}>`) | No child component to memoize — no benefit |
| Function passed to a non-memoized child | The child re-renders anyway |
| Function with dependencies that change every render | Memoization is wasted — the function is recreated anyway |

**Constraints and Limitations:**

- `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.
- Every `useCallback` adds a dependency array to maintain.
- React Compiler may eliminate the need for manual `useCallback` in many cases.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Unnecessary useCallback (Anti-Pattern)

```jsx
// ❌ Unnecessary: handler passed to a regular button
function Bad() {
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return <button onClick={handleClick}>Click</button>;
}

// ✅ Correct: no memo needed
function Good() {
  const handleClick = () => {
    console.log('Clicked');
  };

  return <button onClick={handleClick}>Click</button>;
}
```

**Why:** The `<button>` is a DOM element, not a memoized React component. It re-renders regardless of whether `handleClick` is stable. The `useCallback` adds overhead with no benefit.

#### Example 2: Justified useCallback (Memoized Child)

```jsx
// ✅ Justified: handler passed to a React.memo child
const MemoizedChild = React.memo(function Child({ onAction }) {
  return <button onClick={onAction}>Act</button>;
});

function Good({ id }) {
  const handleAction = useCallback(() => {
    console.log('Action for', id);
  }, [id]);

  return <MemoizedChild onAction={handleAction} />;
}
```

**Why:** `MemoizedChild` only re-renders when its props change. Without `useCallback`, `handleAction` would be a new function on every parent render, causing `MemoizedChild` to re-render unnecessarily.

### Real-World Cases

- **Memoized lists:** Passing stable callbacks to items wrapped in `React.memo`.
- **Effect dependencies:** A callback used inside `useEffect` that would otherwise cause the effect to re-run every render.
- **Custom Hooks:** Returning a stable function from a custom Hook that consumers pass to memoized children.

### References

- rtCamp — React Hooks Best Practices: Avoid defensive memoization — https://rtcamp.com/handbook/react-best-practices/hooks/
- FreeCodeCamp — How to Avoid Overusing useCallback and useMemo in React — https://www.freecodecamp.org/news/how-to-avoid-overusing-usecallback-and-usememo-in-react/
- PodRocket — The useless useCallback: React performance myths unpacked — https://podrocket.logrocket.com/usecallback

---

## 10. Effect Races

### Definitions

**Core Definition:** An effect race condition occurs when multiple asynchronous operations triggered by an effect complete in an unpredictable order, and a slower, older request overwrites the result of a faster, newer one.

**Technical Definition:** When a component re-renders with new dependencies, React runs the cleanup of the previous effect and then the new effect. If the previous effect initiated an async request and the new effect starts another, the responses can arrive out of order. Without cancellation or a staleness guard, the older response's `setState` overwrites the newer data, producing a UI that does not match the current state.

**Beginner-Friendly Explanation:** If you search for "a", then "ab", then "abc", three requests fire. If "a" is slow and returns last, it overwrites the "abc" results. The user sees the wrong data.

### Purposes

- To recognize the conditions that produce effect races (rapid dependency changes, variable network latency).
- To learn the two primary fixes: `AbortController` and a cancellation flag.
- To ensure the UI always reflects the latest request.

### Syntax Rules and Structure

**Race-prone pattern:**

```jsx
useEffect(() => {
  fetch(`/api/search?q=${query}`)
    .then((r) => r.json())
    .then((data) => setResults(data)); // No staleness check
}, [query]);
```

**Fix 1: Cancellation flag (compatible everywhere):**

```jsx
useEffect(() => {
  let cancelled = false;

  fetch(`/api/search?q=${query}`)
    .then((r) => r.json())
    .then((data) => {
      if (!cancelled) setResults(data);
    });

  return () => { cancelled = true; };
}, [query]);
```

**Fix 2: AbortController (true cancellation):**

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setResults)
    .catch((err) => {
      if (err.name !== 'AbortError') setError(err.message);
    });

  return () => controller.abort();
}, [query]);
```

**Constraints and Limitations:**

- `AbortController` actually cancels the network request; the cancellation flag only ignores the result (the request still consumes bandwidth).
- `AbortError` must be handled separately — it is expected and should not be shown to the user.
- When the request is aborted, `setLoading(false)` in `.then` is skipped — ensure loading state is handled correctly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Race Condition Demonstrated

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    fetch(`/api/search?q=${query}`)
      .then((r) => r.json())
      .then(setResults); // ❌ Race: old response can overwrite new
  }, [query]);

  return <ul>{results.map((r) => <li key={r.id}>{r.name}</li>)}</ul>;
}
```

**Expected behavior:** Rapidly changing `query` can display results from an earlier query if its response arrives later.

**Fixed version:**

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then((r) => r.json())
      .then(setResults)
      .catch((err) => {
        if (err.name !== 'AbortError') console.error(err);
      });

    return () => controller.abort();
  }, [query]);

  return <ul>{results.map((r) => <li key={r.id}>{r.name}</li>)}</ul>;
}
```

**Why:** On `query` change, the cleanup aborts the previous request. The aborted request's `.then` never runs (or throws `AbortError`), so stale data cannot overwrite fresh data.

### Real-World Cases

- **Search-as-you-type:** Rapidly changing queries produce out-of-order responses.
- **Tab switching:** Switching tabs fetches different data; a slow response from the previous tab can overwrite the new tab's data.
- **Pagination:** Clicking through pages quickly — a slow response from page 1 can overwrite page 3.

### References

- Tencent Cloud — React 中用 useEffect 请求数据，为什么总会出现“竞态”和内存泄漏警告 — https://cloud.tencent.cn/developer/article/2744647
- Tencent Cloud — React 中的竞态条件与内存泄漏 — https://cloud.tencent.com.cn/developer/article/2744647
- DEV Community — The Phantom Re-Render: How an Unhandled Async Race Condition Tanked Dashboard Performance — https://dev.to/

---

## 11. Effect Cleanup Issues

### Definitions

**Core Definition:** Effect cleanup issues are failures to properly clean up side effects (timers, listeners, subscriptions, requests) when an effect re-runs or the component unmounts, leading to memory leaks and state updates on unmounted components.

**Technical Definition:** Every effect that establishes a resource must return a cleanup function that releases it. If the cleanup is missing, resources accumulate on every re-run or remain after unmount. Common leaks include: `setInterval` without `clearInterval`, `addEventListener` without `removeEventListener`, WebSocket connections without `close()`, and fetch requests without cancellation.

**Beginner-Friendly Explanation:** If you start something in an effect — a timer, a listener, a connection — you must stop it when the effect is cleaned up. Otherwise, it keeps running forever, consuming memory and causing errors.

### Purposes

- To recognize the resource types that require cleanup.
- To learn the correct cleanup pattern for each resource.
- To prevent memory leaks and "setState on unmounted component" warnings.

### Syntax Rules and Structure

**Cleanup patterns:**

| Resource | Setup | Cleanup |
|----------|-------|---------|
| `setInterval` | `const id = setInterval(...)` | `clearInterval(id)` |
| `setTimeout` | `const id = setTimeout(...)` | `clearTimeout(id)` |
| Event listener | `window.addEventListener(...)` | `window.removeEventListener(...)` (same function reference) |
| WebSocket | `new WebSocket(url)` | `ws.close()` |
| Fetch | `fetch(url)` | `controller.abort()` |
| Subscription | `source.subscribe(...)` | `subscription.unsubscribe()` |

**Syntax template:**

```jsx
useEffect(() => {
  const resource = acquireResource();

  return () => {
    releaseResource(resource);
  };
}, [dependencies]);
```

**Constraints and Limitations:**

- `removeEventListener` requires the **same function reference** used in `addEventListener` — an inline arrow function will not be removed.
- `setInterval` without `clearInterval` is one of the most common leak sources.
- Cleanup runs on dependency change **and** on unmount.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Missing Event Listener Cleanup (Memory Leak)

```jsx
// ❌ LEAK: Listener added but never removed
useEffect(() => {
  window.addEventListener('resize', handleResize);
}, []); // No cleanup

// ✅ CORRECT: Cleanup removes the listener
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

**Why:** Without cleanup, every re-mount adds another listener. Even after unmount, the listener keeps the component's closure alive, preventing garbage collection.

#### Example 2: Missing Interval Cleanup

```jsx
// ❌ LEAK: Interval keeps running after unmount
useEffect(() => {
  setInterval(() => setCount((c) => c + 1), 1000);
}, []);

// ✅ CORRECT: Cleanup clears the interval
useEffect(() => {
  const id = setInterval(() => setCount((c) => c + 1), 1000);
  return () => clearInterval(id);
}, []);
```

**Why:** An uncleared interval continues firing after the component is gone, calling `setCount` on an unmounted component and leaking memory.

### Real-World Cases

- **Resize listeners:** A responsive layout that adds a `resize` listener without removing it — multiple listeners accumulate on every navigation.
- **WebSocket connections:** A chat component that opens a WebSocket but never closes it — connections stack up.
- **Timers in modals:** A countdown timer that keeps running after the modal closes.

### References

- CoreUI — How to fix memory leaks in React — https://coreui.io/blog/how-to-fix-memory-leaks-in-react/
- Tencent Cloud — Codex写React组件为什么页面切几次就越来越卡？用Effect清理避免内存泄漏 — https://grapecity.csdn.net/
- Epic React — Effect Cleanup — https://www.epicreact.dev/effect-cleanup

---

## 12. Infinite Loops

### Definitions

**Core Definition:** An infinite loop occurs when a `useEffect` continuously triggers state updates that re-run the effect, or when a dependency array contains a value that is recreated on every render.

**Technical Definition:** Two primary causes: (1) **Missing dependency array** — the effect runs after every render, and if it calls `setState`, it triggers another render, creating a loop. (2) **Unstable dependencies** — objects, arrays, or functions created during render are new references on every render. React's `Object.is` comparison sees them as changed, re-running the effect, which may set state, triggering another render, and so on.

**Beginner-Friendly Explanation:** An infinite loop happens when an effect updates state, which causes a re-render, which runs the effect again, which updates state again... forever. It also happens when you put a function or object in the dependency array — React sees it as "new" every time and re-runs the effect.

### Purposes

- To identify the two root causes of infinite loops.
- To learn the fixes: proper dependency arrays, `useMemo` for objects, and `useCallback` for functions.
- To avoid the trap of "fixing" the loop by disabling the linter.

### Syntax Rules and Structure

**Cause 1: Missing dependency array:**

```jsx
useEffect(() => {
  setCount(count + 1); // Runs after every render → loop
}); // No dependency array
```

**Fix:** Add a dependency array or remove the state update from the effect.

**Cause 2: Unstable object/function in deps:**

```jsx
const options = { page: 1 }; // New object every render
useEffect(() => {
  fetchData(options);
}, [options]); // New reference every render → loop
```

**Fixes:**

- Destructure to primitives: `[options.page]`
- Memoize the object: `useMemo(() => ({ page: 1 }), [])`
- Move the object inside the effect if it does not depend on render values

**Cause 3: Function dependency:**

```jsx
const logItems = () => console.log(items); // New function every render
useEffect(() => { logItems(); }, [logItems]); // Loop
```

**Fix:** Move `logItems` inside the effect, or wrap it in `useCallback`.

**Constraints and Limitations:**

- Removing the dependency array to "fix" a missing dependency is an anti-pattern — it creates an infinite loop.
- Disabling `exhaustive-deps` without restructuring hides the real problem.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Missing Dependency Array Causes Loop

```jsx
function Bad() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setCount(count + 1); // ❌ Runs every render → infinite loop
  }); // No dependency array

  return <p>{count}</p>;
}
```

**Expected behavior:** The component renders infinitely, freezing the browser. The console may show "Maximum update depth exceeded."

**Fix:**

```jsx
useEffect(() => {
  setCount((c) => c + 1); // Still a loop if effect runs every render!
}, []); // Runs once — no loop
```

Or, better, do not update state in an effect at all if it can be derived during render.

#### Example 2: Object Dependency Loop

```jsx
function Bad({ filters }) {
  useEffect(() => {
    fetchData(filters); // filters is a new object every render
  }, [filters]); // ❌ New reference → infinite loop
}

// ✅ Fix 1: Destructure to primitives
function Good({ filters }) {
  useEffect(() => {
    fetchData({ status: filters.status, type: filters.type });
  }, [filters.status, filters.type]);
}

// ✅ Fix 2: Memoize the object
function Good2({ filters }) {
  const stableFilters = useMemo(
    () => ({ status: filters.status, type: filters.type }),
    [filters.status, filters.type]
  );
  useEffect(() => {
    fetchData(stableFilters);
  }, [stableFilters]);
}
```

**Why:** The original `filters` object is recreated on every parent render. React's `Object.is` comparison sees a different reference and re-runs the effect — which may set state, triggering another render.

### Real-World Cases

- **Data fetching with options objects:** Passing a new options object to a fetch effect — always destructure or memoize.
- **Event listeners with inline handlers:** Passing `() => handleSomething()` to a dependency array — always use a stable reference.
- **Syncing props to state:** An effect that sets state from props creates a loop when the state update triggers a re-render that passes new props.

### References

- React — exhaustive-deps: Adding a function dependency causes infinite loops — https://zh-hans.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- Stack Overflow — React useEffect dependency array causes infinite re-render — https://stackoverflow.com/questions/58872158/react-useeffect-dependency-array-causes-infinite-re-render

---

## 13. Memory Leaks

### Definitions

**Core Definition:** Memory leaks in React occur when resources (event listeners, timers, subscriptions, DOM references) remain active after the component that created them has unmounted, preventing garbage collection.

**Technical Definition:** JavaScript's garbage collector cannot free memory that is still referenced by an active listener, timer, or subscription. If an effect establishes a resource without a cleanup function, the resource's callback retains a closure over the component's state and props. Even after the component is removed from the DOM, the closure keeps those values in memory, and the callback continues executing — often producing "Can't perform a React state update on an unmounted component" warnings.

**Beginner-Friendly Explanation:** A memory leak is when something your component started (a timer, a listener) keeps running after the component is gone. It holds onto memory and can cause errors because it is still trying to update a component that no longer exists.

### Purposes

- To understand the resource types that cause leaks.
- To apply consistent cleanup patterns.
- To use tools (React DevTools, browser memory profilers) to detect leaks.

### Syntax Rules and Structure

**Common leak sources and fixes:**

| Leak source | Detection | Fix |
|-------------|-----------|-----|
| `addEventListener` without `removeEventListener` | Multiple listeners firing after navigation | Return cleanup that removes the same function reference |
| `setInterval` without `clearInterval` | Timer keeps running after unmount | Return cleanup that clears the interval |
| WebSocket without `close()` | Multiple open connections | Return cleanup that closes the socket |
| Fetch without abort | "setState on unmounted component" warnings | Use `AbortController` or a `cancelled` flag |
| Subscription without `unsubscribe` | Callbacks firing after unmount | Return cleanup that unsubscribes |

**Constraints and Limitations:**

- `removeEventListener` must receive the exact same function reference used in `addEventListener` — an inline arrow function will not be removed.
- React 18+ no longer warns about state updates on unmounted components, but the underlying leak still exists.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Event Listener Leak

```jsx
// ❌ LEAK: Listener added but never removed
function Bad() {
  useEffect(() => {
    const handleScroll = () => console.log('Scrolling');
    window.addEventListener('scroll', handleScroll);
    // No cleanup
  }, []);

  return <div>Scroll the page</div>;
}

// ✅ CORRECT: Cleanup removes the listener
function Good() {
  useEffect(() => {
    const handleScroll = () => console.log('Scrolling');
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return <div>Scroll the page</div>;
}
```

**Expected behavior:** Navigating away from `Bad` leaves the scroll listener active, leaking memory and logging to the console forever. `Good` removes the listener on unmount.

#### Example 2: WebSocket Leak

```jsx
// ❌ LEAK: WebSocket never closed
useEffect(() => {
  const ws = new WebSocket('wss://api.example.com');
  ws.onmessage = (e) => setMessages((prev) => [...prev, e.data]);
}, []); // No cleanup

// ✅ CORRECT: Close on unmount
useEffect(() => {
  const ws = new WebSocket('wss://api.example.com');
  ws.onmessage = (e) => setMessages((prev) => [...prev, e.data]);
  return () => ws.close();
}, []);
```

### Real-World Cases

- **Single-page applications:** Navigating between routes leaves listeners and timers from previous pages active, gradually degrading performance.
- **Chat applications:** Opening and closing chat windows without closing WebSocket connections.
- **Dashboard widgets:** Each widget adds a resize listener; reordering widgets accumulates listeners.

### References

- CoreUI — How to fix memory leaks in React — https://coreui.io/blog/how-to-fix-memory-leaks-in-react/
- GitHub — Memory Leak Detection & Resolution Skill — https://github.com/
- Tencent Cloud — React 中用 useEffect 请求数据，为什么总会出现“竞态”和内存泄漏警告 — https://cloud.tencent.cn/developer/article/2744647

---

## 14. Hook Debugging

### Definitions

**Core Definition:** Hook debugging is the process of diagnosing Hook-related issues — stale closures, missing dependencies, infinite loops, and unexpected re-renders — using React DevTools, console logging, and custom debugging Hooks.

**Technical Definition:** Debugging Hooks requires visibility into three dimensions: (1) **hook state** — what values are stored in `useState`/`useReducer`; (2) **execution order** — which effects run and when; and (3) **dependency changes** — which values caused a re-run. React DevTools provides state inspection; `useDebugValue` labels custom Hooks; `usePrevious` and change-detection Hooks reveal dependency changes; and render counters detect infinite loops.

**Beginner-Friendly Explanation:** Debugging Hooks means figuring out why your component is re-rendering too much, why an effect isn't running, or why a value is stuck. Tools like React DevTools and simple logging Hooks make these problems visible.

### Purposes

- To inspect Hook state and dependency changes in real time.
- To detect infinite loops early.
- To debug custom Hooks with meaningful labels.
- To track which dependencies caused an effect to re-run.

### Syntax Rules and Structure

**Key debugging tools:**

| Tool | Purpose |
|------|---------|
| React DevTools | Inspect Hook state in the component tree |
| `console.log` in effects | See when effects run and with what values |
| `usePrevious` | Compare current and previous prop/state values |
| `useRenderCount` | Track how many times a component renders |
| `useWhyDidYouUpdate` | Log which props changed between renders |
| `useDebugValue` | Label custom Hooks in DevTools |
| `useInfiniteLoopDetector` | Detect runaway render loops |

**Constraints and Limitations:**

- `console.log` in render bodies logs on every render — use effects or conditional logging for cleaner output.
- `useDebugValue` only works for custom Hooks, not built-in Hooks.
- React DevTools Profiler adds overhead — use in development only.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: useRenderCount and useWhyDidYouUpdate

```jsx
import { useRef, useEffect } from 'react';

function useRenderCount(componentName) {
  const count = useRef(0);
  count.current++;
  console.log(`[${componentName}] Render #${count.current}`);
  return count.current;
}

function useWhyDidYouUpdate(name, props) {
  const previous = useRef();

  useEffect(() => {
    if (previous.current) {
      const changed = {};
      Object.keys({ ...previous.current, ...props }).forEach((key) => {
        if (previous.current[key] !== props[key]) {
          changed[key] = { from: previous.current[key], to: props[key] };
        }
      });
      if (Object.keys(changed).length > 0) {
        console.log(`[why-did-you-update] ${name}`, changed);
      }
    }
    previous.current = props;
  });
}

function UserCard({ user, theme }) {
  useRenderCount('UserCard');
  useWhyDidYouUpdate('UserCard', { user, theme });
  return <div>{user.name}</div>;
}
```

**Expected output:** The console shows render counts and, whenever props change, a log of which props changed and their old/new values.

#### Example 2: Infinite Loop Detector

```jsx
function useInfiniteLoopDetector(name, maxRenders = 50) {
  const count = useRef(0);
  const timer = useRef(null);

  count.current++;
  clearTimeout(timer.current);
  timer.current = setTimeout(() => { count.current = 0; }, 1000);

  if (count.current > maxRenders) {
    console.error(`⚠️ INFINITE LOOP DETECTED in ${name}!`, {
      renders: count.current,
      window: '1 second',
    });
  }
}
```

**Expected output:** If the component renders more than 50 times within one second, a console error identifies the component and the render count.

### Real-World Cases

- **Stale closure diagnosis:** Using `usePrevious` to confirm that a value changed but the effect did not re-run.
- **Performance investigation:** `useRenderCount` reveals components rendering far more than expected.
- **Custom Hook debugging:** `useDebugValue` labels Hook state in React DevTools, making it readable.

### References

- CoreUI — How to debug React hooks — https://coreui.io/answers/how-to-debug-react-hooks/
- CoreUI — How to debug React with console.log — https://coreui.io/answers/how-to-debug-react-with-console-log/
- Telerik — The React useDebugValue Hook — https://www.telerik.com/blogs/usebugvalue-hook

---

## 15. Hook Best Practices

### Definitions

**Core Definition:** Hook best practices are guidelines for writing Hooks that are correct, performant, testable, and maintainable — avoiding the pitfalls covered in this cheat sheet.

**Technical Definition:** Best practices encompass obeying the Rules of Hooks, using dependency arrays correctly, cleaning up all side effects, applying memoization only with evidence, preferring derived values over effect-based state, and composing custom Hooks with single responsibilities.

**Beginner-Friendly Explanation:** Best practices are the habits that keep your Hooks from becoming bugs. Follow the rules, clean up after yourself, do not over-optimize, and keep each Hook focused.

### Purposes

- To establish a consistent, safe approach to Hooks across a codebase.
- To prevent the most common pitfalls before they occur.
- To make Hook code easier to review, test, and debug.

### Syntax Rules and Structure

**Best Practices Checklist:**

| Practice | Description |
|----------|-------------|
| **Follow the Rules of Hooks** | Top level only, React functions only |
| **Enable `exhaustive-deps`** | Never disable it without strong justification |
| **Clean up every effect** | Return a cleanup function for timers, listeners, subscriptions, and requests |
| **Use `AbortController` for fetches** | Cancel requests on dependency change or unmount |
| **Prefer derived values over effects** | Compute from props/state during render; do not sync with `useEffect` |
| **Memoize only with evidence** | Profile first; do not wrap cheap computations in `useMemo` |
| **Do not wrap every function in `useCallback`** | Only when passing to a `React.memo` child or using as a dependency |
| **Split overloaded effects** | One effect should handle one responsibility |
| **Use functional updates for state** | `setCount(c => c + 1)` avoids stale closures |
| **Use refs for latest values in stable callbacks** | When an effect must not re-run but needs fresh data |
| **Test custom Hooks in isolation** | Use `renderHook` and `act` from `@testing-library/react` |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Well-Behaved Effect (Cleanup + Dependencies + Abort)

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const controller = new AbortController();

    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
      controller.abort(); // If a fetch were involved
    };
  }, []); // ✅ No reactive values from component scope

  return width;
}
```

**Why this follows best practices:** The effect has a complete cleanup (listener removal + abort), and the dependency array is correct (empty because the effect uses no reactive values from the component).

#### Example 2: Anti-Pattern — Disabling exhaustive-deps

```jsx
// ❌ Disabling the linter hides a stale closure bug
useEffect(() => {
  fetchUser(userId).then(setUser);
}, []); // eslint-disable-line react-hooks/exhaustive-deps
```

**Why this is dangerous:** The effect never re-runs when `userId` changes. The linter was correct — the dependency is missing. The correct fix is to add `[userId]`, not to disable the warning.

### Real-World Cases

- **Code reviews:** Best practices provide a checklist for reviewing Hook code.
- **Team onboarding:** New developers learn the correct patterns from the start.
- **Production stability:** Consistent cleanup and dependency management prevent the majority of Hook-related production bugs.

### References

- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- rtCamp — React Hooks Best Practices — https://rtcamp.com/handbook/react-best-practices/hooks/
- React — Removing Effect Dependencies — https://react.dev/learn/removing-effect-dependencies
- CoreUI — How to fix memory leaks in React — https://coreui.io/blog/how-to-fix-memory-leaks-in-react/
- CoreUI — How to fix stale closures in React hooks — https://coreui.io/answers/how-to-fix-stale-closures-in-react-hooks/

---

## References

- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- React (18) — Rules of Hooks — https://18.react.dev/reference/rules/rules-of-hooks
- React — exhaustive-deps — https://zh-hans.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React — Removing Effect Dependencies — https://react.dev/learn/removing-effect-dependencies
- CoreUI — How to fix stale closures in React hooks — https://coreui.io/answers/how-to-fix-stale-closures-in-react-hooks/
- CoreUI — How to debug React hooks — https://coreui.io/answers/how-to-debug-react-hooks/
- CoreUI — How to fix memory leaks in React — https://coreui.io/blog/how-to-fix-memory-leaks-in-react/
- rtCamp — React Hooks Best Practices — https://rtcamp.com/handbook/react-best-practices/hooks/
- TechRepublic — React JS: Best Practices, Pitfalls & Performance Tweaks for 2025 — https://www.techrepublic.com/forums/discussions/react-js-best-practices-pitfalls-performance-tweaks-for-2025/
- Tencent Cloud — React 中用 useEffect 请求数据，为什么总会出现“竞态”和内存泄漏警告 — https://cloud.tencent.cn/developer/article/2744647
- Tencent Cloud — React 中的竞态条件与内存泄漏 — https://cloud.tencent.com.cn/developer/article/2744647
- Steve Kinney — useMemo and useCallback in React 19 — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- Steve Kinney — Avoiding Over-Memoization — https://stevekinney.com/courses/react-performance/avoiding-over-memoization
- DebugBear — Improve React Performance With useMemo And useCallback — https://www.debugbear.com/blog/react-usememo-usecallback
- FreeCodeCamp — How to Avoid Overusing useCallback and useMemo in React — https://www.freecodecamp.org/news/how-to-avoid-overusing-usecallback-and-usememo-in-react/
- PodRocket — The useless useCallback: React performance myths unpacked — https://podrocket.logrocket.com/usecallback
- Telerik — The React useDebugValue Hook — https://www.telerik.com/blogs/usebugvalue-hook
- Epic React — Effect Cleanup — https://www.epicreact.dev/effect-cleanup
- GitHub — Memory Leak Detection & Resolution Skill — https://github.com/
- Stack Overflow — React useEffect dependency array causes infinite re-render — https://stackoverflow.com/questions/58872158/react-useeffect-dependency-array-causes-infinite-re-render