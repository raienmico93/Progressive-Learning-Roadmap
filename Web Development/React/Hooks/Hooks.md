# Hook Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Hooks are JavaScript functions that let function components "hook into" React state and lifecycle features without writing class components.

**Technical Definition:** Hooks are a React 16.8 feature that allows function components to use state, side effects, context, refs, and other React features. They are functions whose names start with `use` and are subject to strict rules governing where they may be called. React identifies each Hook call by its position in the call order, matching it to a memory slot by position, not by name. This means Hooks must be called in the exact same order on every render. The core built-in Hooks include `useState`, `useEffect`, `useContext`, `useReducer`, `useRef`, `useMemo`, `useCallback`, and others. Custom Hooks allow developers to extract and reuse stateful logic across components.

**Beginner-Friendly Explanation:** Hooks are special functions that let your components "remember" things and respond to changes. Before Hooks, you had to write class components to use state. Now you can use simple functions. But Hooks have rules: you must call them at the top of your component, never inside conditions or loops, and only from React functions. These rules exist because React tracks Hooks by their order — like a numbered list — so the order must never change between renders.

### Key Characteristics

- **Function-based:** Only usable in function components and custom Hooks, not in class components.
- **Order-dependent:** React identifies each Hook by call order; the order must be identical on every render.
- **Top-level only:** Must be called at the top level of a component or custom Hook, before any early returns.
- **Composable:** Custom Hooks can call other Hooks, enabling logic reuse across components.
- **Stateful memory:** Hooks let function components retain data between renders.
- **Lint-enforced:** The `eslint-plugin-react-hooks` plugin catches violations at build time.

### Prerequisites

- JavaScript fundamentals: functions, closures, destructuring, arrays, objects.
- Basic React function component syntax.
- Understanding of rendering and re-renders.
- Familiarity with `useState` and `useEffect`.

### Related Programming Areas

- **State Management** — `useState`, `useReducer`, Context.
- **Side Effects** — `useEffect`, `useLayoutEffect`, `useInsertionEffect`.
- **Performance** — `useMemo`, `useCallback`, `React.memo`.
- **Refs** — `useRef`, `useImperativeHandle`.
- **Custom Hooks** — extracting reusable logic.

---

## 1. Hooks

### Definitions

**Core Definition:** Hooks are JavaScript functions that let function components use React state and lifecycle features.

**Technical Definition:** Hooks are functions whose names start with `use` that "hook into" React's internal state and lifecycle mechanisms. They were introduced in React 16.8 to allow function components to manage state, side effects, context, refs, and performance optimisations without writing class components. React maintains a per-component list of memory slots; each Hook call is mapped to a slot by its position in the call order.

**Beginner-Friendly Explanation:** Hooks are special functions that give your components superpowers. They let you store data, react to changes, and share logic — all without writing a class.

### Purposes

- To allow function components to use state and lifecycle features.
- To enable logic reuse across components without changing component hierarchy.
- To simplify component code by replacing class boilerplate.
- To make side effects and state management more predictable and testable.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useState, useEffect } from 'react';

function Component() {
  const [state, setState] = useState(initialValue);
  useEffect(() => { /* side effect */ }, [deps]);
  return <JSX />;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useState(initialValue)` | Declares a state variable. |
| `useEffect(callback, deps)` | Synchronises with an external system. |
| `useContext(Context)` | Reads context from the nearest provider. |
| `useRef(initialValue)` | Creates a mutable ref object. |
| `useMemo(fn, deps)` | Caches an expensive computation. |
| `useCallback(fn, deps)` | Caches a function reference. |

**Syntax Rules:**

1. Hook names must start with `use`.
2. Hooks must be called at the top level of a function component or custom Hook.
3. Hooks must be called in the same order on every render.
4. Hooks can only be called from React function components or custom Hooks.

**Constraints and Limitations:**

- Cannot be called inside class components.
- Cannot be called inside regular JavaScript functions.
- Cannot be called conditionally or in loops.
- In Strict Mode, React calls Hooks twice in development to detect impurities.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Hooks Usage

```jsx
import { useState, useEffect } from 'react';

function Counter() {
  // Hook 1: state
  const [count, setCount] = useState(0);

  // Hook 2: effect — runs after render when count changes
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Expected Output:** A counter that updates the document title to "Count: 0", "Count: 1", etc.

**Why this output:** `useState` declares state. `useEffect` synchronises the document title with the count. When `count` changes, the effect re-runs and updates the title.

#### Example 2: Custom Hook

```jsx
import { useState, useEffect } from 'react';

function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}

function App() {
  const width = useWindowWidth();
  return <p>Window width: {width}px</p>;
}
```

**Expected Output:** The window width updates as the browser is resized.

**Why this output:** `useWindowWidth` is a custom Hook that encapsulates the window width logic. `App` calls it and receives the current width. The Hook is reusable across components.

### Real-World Cases

- **Form state:** `useState` for input values.
- **Data fetching:** `useEffect` for API calls.
- **Theming:** `useContext` for theme values.
- **Performance:** `useMemo` and `useCallback` for optimisation.

### References

- React — Rules of Hooks — https://uk.react.dev/reference/rules/rules-of-hooks
- React — Hooks Overview — https://mintlify.wiki/facebook/react/guides/hooks

---

## 2. Hook Rules

### Definitions

**Core Definition:** Hook rules are the two mandatory constraints that govern where and how Hooks may be called in React.

**Technical Definition:** React enforces two rules for Hooks: (1) Only call Hooks at the top level of a function component or custom Hook — never inside loops, conditions, nested functions, or `try`/`catch`/`finally` blocks. (2) Only call Hooks from React function components or custom Hooks — never from regular JavaScript functions. These rules exist because React relies on the order in which Hooks are called to correctly preserve state between renders.

**Beginner-Friendly Explanation:** There are two big rules: (1) Only call Hooks at the top of your component, not inside ifs or loops. (2) Only call Hooks from React components or custom Hooks. If you break these rules, React gets confused about which state belongs to which Hook.

### Purposes

- To ensure React can correctly associate state with each Hook call.
- To prevent "Rendered fewer/more hooks than expected" errors.
- To make component behaviour predictable across renders.
- To enable the linter to detect violations automatically.

### Syntax Rules and Structure

**Rule 1: Only Call Hooks at the Top Level**

```jsx
// ✅ Correct
function Component() {
  const [count, setCount] = useState(0);
  useEffect(() => {}, []);
}

// ❌ Wrong: Hook inside a condition
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ❌ Wrong: Hook inside a loop
for (let i = 0; i < 10; i++) {
  const [x, setX] = useState(0);
}

// ❌ Wrong: Hook after early return
if (!data) return <Loading />;
const [processed, setProcessed] = useState(data);
```

**Rule 2: Only Call Hooks from React Functions**

```jsx
// ✅ Correct: function component
function Component() {
  const [count, setCount] = useState(0);
}

// ✅ Correct: custom Hook
function useCustomHook() {
  const [count, setCount] = useState(0);
}

// ❌ Wrong: regular JavaScript function
function regularFunction() {
  const [count, setCount] = useState(0);
}

// ❌ Wrong: class method
class Bad extends React.Component {
  render() {
    useEffect(() => {}); // Not supported
  }
}
```

**Syntax Rules:**

1. Hooks must be called unconditionally before any early return.
2. Hooks cannot be called inside event handlers or callbacks.
3. Hooks cannot be called at the module level.
4. The `use` hook is a special exception — it can be called conditionally and in loops.

**Constraints and Limitations:**

- Violating the rules causes runtime errors and state mismatches.
- The linter catches most violations at build time.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Common Violation and Fix

```jsx
// ❌ Wrong: conditional Hook
function Bad({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // Hook after condition
  }
  const [count, setCount] = useState(0); // Different order!
}

// ✅ Correct: Hooks always called, condition inside
function Good({ isLoggedIn }) {
  const [user, setUser] = useState(null); // Always called
  const [count, setCount] = useState(0);  // Always called

  useEffect(() => {
    if (isLoggedIn) {
      // Condition inside the Hook
    }
  }, [isLoggedIn]);
}
```

**Expected Output:** The correct version works consistently across renders.

**Why this output:** In the bad version, the number of Hooks called changes depending on `isLoggedIn`. React loses track of which state belongs to which Hook. In the correct version, the same Hooks are always called in the same order.

#### Example 2: The `use` Hook Exception

```jsx
// ✅ `use` can be called conditionally
if (shouldFetch) {
  const data = use(fetchPromise);
}

// ✅ `use` can be called in loops
for (const promise of promises) {
  results.push(use(promise));
}
```

**Expected Output:** The `use` Hook works conditionally, unlike other Hooks.

**Why this output:** The `use` API is specially designed to be called conditionally and in loops. Other Hooks do not have this exception.

### Real-World Cases

- **All React applications:** These rules apply universally to every component using Hooks.
- **Linting:** The ESLint plugin catches violations before they reach production.
- **Code reviews:** Reviewers check for Hook rule violations.

### References

- React — Rules of Hooks — https://uk.react.dev/reference/rules/rules-of-hooks
- React — eslint-plugin-react-hooks — https://ar.react.dev/reference/eslint-plugin-react-hooks

---

## 3. Hook Rules of Hooks

### Definitions

**Core Definition:** The "Rules of Hooks" is the official name for the two constraints that govern Hook usage in React.

**Technical Definition:** The Rules of Hooks are documented in the React reference and enforced by the `eslint-plugin-react-hooks` plugin. They state: (1) Only call Hooks at the top level — not inside loops, conditions, nested functions, or `try`/`catch`/`finally` blocks. (2) Only call Hooks from React function components or custom Hooks — not from regular JavaScript functions. These rules ensure that Hooks are called in the same order on every render, which is how React preserves state between multiple `useState` and `useEffect` calls.

**Beginner-Friendly Explanation:** The "Rules of Hooks" are the official rules you must follow. They are: keep Hooks at the top of your component, and only use them in React components or custom Hooks.

### Purposes

- To preserve the correct mapping between Hook calls and state.
- To prevent state corruption and runtime errors.
- To enable React's internal tracking mechanism.

### Syntax Rules and Structure

**The Two Rules:**

| Rule | Description |
|------|-------------|
| Top level only | Call Hooks at the top level, before any early returns. |
| React functions only | Call Hooks from function components or custom Hooks. |

**Syntax Rules:**

1. Never call Hooks inside conditions, loops, nested functions, or `try`/`catch` blocks.
2. Never call Hooks after a conditional `return`.
3. Never call Hooks from regular JavaScript functions.
4. Custom Hooks may call other Hooks.

### Annotated Code Examples

```jsx
function Form() {
  // 1. useState for name
  const [name, setName] = useState('Mary');

  // 2. useEffect to persist name
  useEffect(() => {
    localStorage.setItem('formData', name);
  }, [name]);

  // 3. useState for surname
  const [surname, setSurname] = useState('Poppins');

  // 4. useEffect to update title
  useEffect(() => {
    document.title = name + ' ' + surname;
  }, [name, surname]);
}
```

**Expected Output:** The form state is persisted and the title updates.

**Why this output:** React calls these four Hooks in the exact same order on every render, so it can correctly match each Hook call to its state slot.

### Real-World Cases

- **Every component using Hooks:** The Rules of Hooks apply universally.
- **Custom Hooks:** Custom Hooks must follow the same rules as built-in Hooks.

### References

- React — Rules of Hooks — https://uk.react.dev/reference/rules/rules-of-hooks
- React — eslint-plugin-react-hooks: rules-of-hooks — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/rules-of-hooks

---

## 4. Only Call Hooks at Top Level

### Definitions

**Core Definition:** Hooks must be called at the top level of a function component or custom Hook, before any early returns.

**Technical Definition:** React maintains a per-component list of memory slots. Each Hook call is mapped to a slot by its position in the call order. If Hooks are called conditionally or in loops, the number and order of Hooks called changes between renders, causing React to lose track of which state corresponds to which Hook call. This results in bugs like state mismatches and "Rendered fewer/more hooks than expected" errors.

**Beginner-Friendly Explanation:** Always call Hooks at the very top of your component, before any `if` statements or `return` statements. This way, React always knows which Hook is which.

### Purposes

- To ensure Hooks are called in the same order on every render.
- To preserve the mapping between Hook calls and state slots.
- To prevent runtime errors from inconsistent Hook counts.

### Syntax Rules and Structure

```jsx
// ✅ Correct
function Component() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  if (!count) return null; // Early return AFTER all Hooks
  return <div>{name}: {count}</div>;
}

// ❌ Wrong
function Bad() {
  if (!data) return <Loading />; // Early return BEFORE Hooks
  const [processed, setProcessed] = useState(data); // Hook after return
}
```

**Syntax Rules:**

1. All Hooks must be called before any early return.
2. Hooks cannot be called inside `if`/`else` blocks.
3. Hooks cannot be called inside loops.
4. Hooks cannot be called inside `try`/`catch`/`finally` blocks.

### Annotated Code Examples

```jsx
function Component({ isSpecial, shouldFetch, fetchPromise }) {
  // ✅ All Hooks at the top level
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  if (!isSpecial) {
    return null; // Early return AFTER all Hooks
  }

  if (shouldFetch) {
    // ✅ `use` can be conditional
    const data = use(fetchPromise);
    return <div>{data}</div>;
  }

  return <div>{name}: {count}</div>;
}
```

**Expected Output:** The component renders correctly regardless of props.

**Why this output:** All Hooks are called before any conditional return. The `use` hook is the only exception that can be called conditionally.

### Real-World Cases

- **Conditional rendering:** Early returns are common, but they must come after all Hooks.
- **Loading states:** Returning a spinner while loading must happen after Hooks.

### References

- React — Rules of Hooks: Only call Hooks at the top level — https://uk.react.dev/reference/rules/rules-of-hooks#only-call-hooks-at-the-top-level

---

## 5. Only Call Hooks from React Functions

### Definitions

**Core Definition:** Hooks can only be called from React function components or custom Hooks, never from regular JavaScript functions.

**Technical Definition:** React provides a mechanism for tracking Hook calls only within the rendering context of function components and custom Hooks. Calling a Hook from a regular JavaScript function (e.g., a utility function, event handler, or class method) is not supported because React cannot associate the Hook call with a component's state. Custom Hooks are the mechanism for extracting reusable logic that uses Hooks.

**Beginner-Friendly Explanation:** You can only use Hooks inside React components or custom Hooks. You cannot use them in plain JavaScript functions.

### Purposes

- To ensure Hooks are only called during React's rendering phase.
- To enable the extraction of reusable logic into custom Hooks.
- To maintain clear boundaries between React and non-React code.

### Syntax Rules and Structure

```jsx
// ✅ Correct: function component
function Component() {
  const [count, setCount] = useState(0);
}

// ✅ Correct: custom Hook
function useCustomHook() {
  const [count, setCount] = useState(0);
  return count;
}

// ❌ Wrong: regular function
function utilityFunction() {
  const [count, setCount] = useState(0); // Not supported
}

// ❌ Wrong: class component
class Bad extends React.Component {
  render() {
    useEffect(() => {}); // Not supported
  }
}
```

**Syntax Rules:**

1. Custom Hook names must start with `use`.
2. Hooks cannot be called from event handlers.
3. Hooks cannot be called from class components.

### Annotated Code Examples

```jsx
// ✅ Correct: custom Hook
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
function App() {
  const [name, setName] = useLocalStorage('name', '');
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

**Expected Output:** The input value is persisted to localStorage.

**Why this output:** `useLocalStorage` is a custom Hook that encapsulates the localStorage logic. It calls `useState` and `useEffect` internally, which is allowed because custom Hooks are React functions.

### Real-World Cases

- **Data fetching:** `useFetch`, `useQuery`.
- **Form handling:** `useForm`, `useInput`.
- **Local storage:** `useLocalStorage`.
- **Debouncing:** `useDebounce`.

### References

- React — Rules of Hooks: Only call Hooks from React functions — https://uk.react.dev/reference/rules/rules-of-hooks#only-call-hooks-from-react-functions
- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks

---

## 6. Custom Hooks

### Definitions

**Core Definition:** A custom Hook is a JavaScript function whose name starts with `use` that encapsulates reusable stateful logic.

**Technical Definition:** A custom Hook is a function that starts with `use` and may call other Hooks. It is the primary mechanism for sharing stateful logic between components without changing component hierarchy. Custom Hooks follow the same rules as built-in Hooks: they must be called at the top level of a component or another custom Hook. They can return any value: state, setters, or a combination. Custom Hooks are the modern replacement for Higher-Order Components (HOCs) and render props for logic reuse.

**Beginner-Friendly Explanation:** A custom Hook is a reusable function that contains Hook logic. You write it once and use it in many components. Its name must start with `use` so React knows it is a Hook.

### Purposes

- To extract and reuse stateful logic across components.
- To replace HOCs and render props for logic reuse.
- To keep components focused on rendering.
- To make logic testable in isolation.

### Syntax Rules and Structure

```jsx
function useCustomHook(initialValue) {
  const [value, setValue] = useState(initialValue);
  // Additional Hook logic
  return { value, setValue };
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useCustomHook` | Function name must start with `use`. |
| `initialValue` | Optional parameters. |
| Return value | Any value (state, setters, objects, arrays). |

**Syntax Rules:**

1. Custom Hook names must start with `use`.
2. Hooks must be called at the top level of the custom Hook.
3. Custom Hooks can call other Hooks, including other custom Hooks.
4. Custom Hooks can return any value.

**Constraints and Limitations:**

- Custom Hooks share stateful logic, not state itself. Each call to a custom Hook gets its own independent state.
- Custom Hooks do not share state between components; they share logic.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: useLocalStorage

```jsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

function App() {
  const [name, setName] = useLocalStorage('name', '');
  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
      placeholder="Enter name"
    />
  );
}
```

**Expected Output:** The input value is persisted to localStorage and restored on reload.

**Why this output:** `useLocalStorage` encapsulates the logic for reading from and writing to localStorage. Each component that calls it gets its own independent state.

#### Example 2: useDebounce

```jsx
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}

function Search() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) {
      // Fetch results with debouncedQuery
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Expected Output:** The search query is debounced by 300ms before fetching.

**Why this output:** `useDebounce` delays the update of `debouncedQuery` until the user stops typing for 300ms. The cleanup clears the timer if the value changes before the delay.

### Real-World Cases

- **Data fetching:** `useFetch`, `useQuery`.
- **Form handling:** `useForm`, `useInput`.
- **Local storage:** `useLocalStorage`.
- **Debouncing:** `useDebounce`.
- **Window dimensions:** `useWindowWidth`, `useWindowHeight`.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- React — Rules of Hooks: Custom Hooks may call other Hooks — https://uk.react.dev/reference/rules/rules-of-hooks

---

## 7. Hook Naming

### Definitions

**Core Definition:** Hook naming is the convention that custom Hooks must start with the word `use`.

**Technical Definition:** React relies on the `use` prefix to identify custom Hooks and enforce the Rules of Hooks. Without the `use` prefix, React cannot automatically check for violations of the Hooks rules because it does not know the function is a Hook. The naming convention is: camelCase with a `use` prefix (e.g., `useInvoiceData`, `useCustomers`, `useDebouncedValue`). Custom Hook file names should also use camelCase with the `use` prefix and a `.ts` or `.js` extension (e.g., `useInvoiceData.ts`).

**Beginner-Friendly Explanation:** Custom Hooks must always start with `use`. This tells React that the function is a Hook, so it can check for rule violations.

### Purposes

- To allow React and the linter to identify custom Hooks.
- To enforce the Rules of Hooks automatically.
- To follow React's built-in Hook naming pattern.
- To make code predictable and consistent.

### Syntax Rules and Structure

**Naming Conventions:**

| Element | Convention | Example |
|---------|-----------|---------|
| Custom Hook | camelCase with `use` prefix | `useInvoiceData` |
| Custom Hook file | camelCase with `use` prefix | `useInvoiceData.ts` |
| Bad | No `use` prefix | `invoiceDataHook.ts` |
| Bad | PascalCase | `UseInvoiceData.ts` |
| Bad | kebab-case | `use-invoice-data.ts` |

**Syntax Rules:**

1. Always start custom Hook names with `use`.
2. Describe what the Hook does, not how it works: `useDebouncedValue` (good), `useStuff` (bad).
3. Use camelCase after the `use` prefix.

### Annotated Code Examples

```jsx
// ✅ Good: descriptive, use prefix
function useDebouncedValue(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}

// ❌ Bad: missing use prefix
function debouncedValue(value, delay) {
  // React cannot identify this as a Hook
}

// ❌ Bad: vague name
function useStuff() {
  // What does this do?
}
```

**Expected Output:** The good Hook is identifiable by React and the linter.

**Why this output:** The `use` prefix signals to React that the function is a Hook. The descriptive name makes the Hook's purpose clear.

### Real-World Cases

- **All custom Hooks:** The `use` prefix is mandatory.
- **Linting:** The ESLint plugin uses the `use` prefix to identify Hooks.
- **Code reviews:** Naming consistency improves readability.

### References

- GitHub — Naming Conventions Guide — https://github.com/novalauliady/invoicegeneratorai
- Syncfusion — Implementing React Custom Hooks: A Complete Guide — https://www.syncfusion.com

---

## 8. Hook Order

### Definitions

**Core Definition:** Hook order is the sequence in which Hooks are called within a component, which must be identical on every render.

**Technical Definition:** React relies on the order in which Hooks are called to correctly preserve state between renders. Each time a component renders, React expects the exact same Hooks to be called in the exact same order. React identifies each Hook call by its position in the call order, matching it to a memory slot by position, not by name. If the order changes, React cannot correctly associate state with each Hook call, leading to state mismatches and "Rendered fewer/more hooks than expected" errors.

**Beginner-Friendly Explanation:** React tracks Hooks like a numbered list. The first Hook is slot 1, the second is slot 2, and so on. If the order changes between renders, React gets confused.

### Purposes

- To enable React to correctly preserve state between renders.
- To prevent state mismatches and runtime errors.
- To ensure predictable component behaviour.

### Syntax Rules and Structure

```jsx
function Component() {
  // Slot 1: useState
  const [count, setCount] = useState(0);

  // Slot 2: useEffect
  useEffect(() => {}, []);

  // Slot 3: useState
  const [name, setName] = useState('');

  // Slot 4: useEffect
  useEffect(() => {}, [name]);
}
```

**Syntax Rules:**

1. The same Hooks must be called in the same order on every render.
2. The number of Hooks called must not change between renders.
3. Conditional Hooks are forbidden because they change the order.

### Annotated Code Examples

```jsx
// ❌ Wrong: order changes
function Bad({ isLoggedIn }) {
  const [count, setCount] = useState(0); // Slot 1
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // Slot 2 (only when logged in!)
  }
  const [name, setName] = useState(''); // Slot 3 or 2 depending on condition
}

// ✅ Correct: consistent order
function Good({ isLoggedIn }) {
  const [count, setCount] = useState(0);  // Slot 1
  const [user, setUser] = useState(null);  // Slot 2 (always)
  const [name, setName] = useState('');    // Slot 3 (always)
}
```

**Expected Output:** The correct version works consistently; the bad version breaks when `isLoggedIn` changes.

**Why this output:** In the bad version, the number and order of Hooks changes depending on `isLoggedIn`. React's internal memory slots become misaligned, causing state mismatches.

### Real-World Cases

- **All components:** Hook order must be consistent.
- **Debugging:** "Rendered fewer/more hooks than expected" errors indicate order violations.

### References

- React — eslint-plugin-react-hooks: rules-of-hooks — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/rules-of-hooks
- Patterns.dev — Hooks Pattern — https://www.patterns.dev

---

## 9. Hook Dependencies

### Definitions

**Core Definition:** Hook dependencies are the values listed in the dependency array of `useEffect`, `useMemo`, and `useCallback` that determine when the Hook should re-run or recompute.

**Technical Definition:** React Hooks like `useEffect`, `useMemo`, and `useCallback` accept a dependency array as an optional second argument. The array lists all reactive values (props, state, and variables declared inside the component) referenced inside the Hook. React compares each dependency with its previous value using `Object.is`. If any dependency changes, the Hook re-runs. If the array is omitted, the Hook runs after every render. If the array is empty, the Hook runs only on mount and cleanup on unmount. The dependency array tells React which values the effect uses, so React knows when to re-synchronise.

**Beginner-Friendly Explanation:** The dependency array tells React when to run your effect. If you list a value in the array, the effect re-runs whenever that value changes. If you leave the array empty, it runs only once.

### Purposes

- To control when effects re-run and memos recompute.
- To avoid stale closures by including all referenced values.
- To optimise performance by running effects only when necessary.
- To synchronise the component with external systems.

### Syntax Rules and Structure

**Dependency Array Behaviours:**

| Array | Behaviour |
|-------|-----------|
| No array | Runs after every render. |
| Empty `[]` | Runs only on mount; cleanup on unmount. |
| `[dep1, dep2]` | Runs when any dependency changes. |

**Syntax Rules:**

1. The array must have a constant number of items and be written inline.
2. Include all reactive values referenced inside the Hook.
3. React compares dependencies with `Object.is`.
4. Omit the array to run on every render.

**Constraints and Limitations:**

- Missing dependencies cause stale closures.
- Including functions/objects that change every render can cause infinite loops.
- The linter (`exhaustive-deps`) catches missing dependencies.

### Annotated Code Examples

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // Re-runs when either changes
}
```

**Expected Output:** The connection is re-established when `serverUrl` or `roomId` changes.

**Why this output:** The dependency array `[serverUrl, roomId]` tells React to re-run the effect when either value changes. The cleanup disconnects the old connection before the new one is created.

### Real-World Cases

- **Data fetching:** Re-fetch when the query or URL changes.
- **Subscriptions:** Re-subscribe when the channel changes.
- **Animations:** Re-trigger when the animation target changes.

### References

- React — useEffect — https://zh-hant.react.dev/reference/react/useEffect
- React — exhaustive-deps — https://pt-br.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 10. Hook Linting

### Definitions

**Core Definition:** Hook linting is the use of ESLint rules to catch violations of the Rules of Hooks and missing dependencies.

**Technical Definition:** The `eslint-plugin-react-hooks` plugin provides two core rules: `rules-of-hooks` (validates that components and Hooks follow the Rules of Hooks) and `exhaustive-deps` (validates that dependency arrays contain all necessary dependencies). The plugin also includes rules flagged by React Compiler, such as immutability, purity, and refs. These lints run at build time and catch errors before they reach production.

**Beginner-Friendly Explanation:** Linting is like a spell-checker for your Hooks. It catches mistakes like calling a Hook inside an `if` statement or forgetting a dependency.

### Purposes

- To catch Hook rule violations at build time.
- To prevent stale closures by enforcing complete dependency arrays.
- To improve code quality and prevent bugs.
- To enforce React's rules for correctness and performance.

### Syntax Rules and Structure

**Installation:**

```bash
npm install eslint-plugin-react-hooks --save-dev
```

**Configuration (`.eslintrc.json`):**

```json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

**Recommended Rules:**

| Rule | Description |
|------|-------------|
| `rules-of-hooks` | Validates that components and Hooks follow the Rules of Hooks. |
| `exhaustive-deps` | Validates that dependency arrays contain all necessary dependencies. |

**Syntax Rules:**

1. `rules-of-hooks` should be set to `"error"`.
2. `exhaustive-deps` should be set to `"warn"` or `"error"`.
3. The plugin is included by default in Create React App.

### Annotated Code Examples

```jsx
// ❌ Missing dependency: exhaustive-deps warns
useEffect(() => {
  console.log(count);
}, []); // Missing 'count'

// ✅ All dependencies included
useEffect(() => {
  console.log(count);
}, [count]);

// ❌ Hook in condition: rules-of-hooks errors
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ✅ Hook at top level
const [user, setUser] = useState(null);
```

**Expected Output:** The linter flags the violations; the correct versions pass.

**Why this output:** The `rules-of-hooks` rule catches conditional Hook calls. The `exhaustive-deps` rule catches missing dependencies.

### Real-World Cases

- **All React projects:** Linting is a standard part of the development workflow.
- **CI/CD pipelines:** Linting catches errors before deployment.
- **Code reviews:** Linting enforces consistency across the team.

### References

- React — eslint-plugin-react-hooks — https://ar.react.dev/reference/eslint-plugin-react-hooks
- npm — eslint-plugin-react-hooks — https://www.npmjs.com/package/eslint-plugin-react-hooks

---

## 11. `eslint-plugin-react-hooks`

### Definitions

**Core Definition:** `eslint-plugin-react-hooks` is the official ESLint plugin that enforces the Rules of Hooks and validates dependency arrays.

**Technical Definition:** The plugin provides ESLint rules that enforce the Rules of React. It covers fundamental React patterns (`exhaustive-deps` and `rules-of-hooks`) and issues flagged by React Compiler. React Compiler diagnostics are automatically surfaced by the plugin, even if the app has not adopted the compiler. The plugin is part of the Hooks API for React and is included by default in Create React App.

**Beginner-Friendly Explanation:** This is the official tool that checks your Hooks code for mistakes. It tells you if you broke a rule or forgot a dependency.

### Purposes

- To enforce the Rules of Hooks at build time.
- To catch missing dependencies in `useEffect`, `useMemo`, and `useCallback`.
- To surface React Compiler diagnostics.
- To improve code correctness and performance.

### Syntax Rules and Structure

**Installation:**

```bash
npm install eslint-plugin-react-hooks --save-dev
```

**Configuration:**

```json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

**Recommended Rules:**

| Rule | Description |
|------|-------------|
| `exhaustive-deps` | Validates that dependency arrays contain all necessary dependencies. |
| `rules-of-hooks` | Validates that components and hooks follow the Rules of Hooks. |
| `component-hook-factories` | Validates higher order functions defining nested components or hooks. |
| `immutability` | Validates against mutating props, state, and other immutable values. |
| `purity` | Validates that components/hooks are pure. |
| `refs` | Validates correct usage of refs. |
| `set-state-in-effect` | Validates against calling `setState` synchronously in an effect. |
| `set-state-in-render` | Validates against setting state during render. |

**Syntax Rules:**

1. The plugin must be added to the `plugins` array in ESLint config.
2. Rules are configured under `rules`.
3. The recommended preset includes the core rules.

### Annotated Code Examples

```jsx
// ❌ rules-of-hooks error: Hook in condition
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ❌ exhaustive-deps warning: missing dependency
useEffect(() => {
  fetchUser(userId);
}, []); // Missing 'userId'

// ✅ All correct
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

**Expected Output:** The linter flags the violations with clear error messages.

**Why this output:** The `rules-of-hooks` rule detects the conditional Hook call. The `exhaustive-deps` rule detects the missing `userId` dependency.

### Real-World Cases

- **All React projects:** The plugin is a standard development dependency.
- **CI/CD:** Linting runs as part of the build pipeline.
- **Team consistency:** The plugin enforces the same rules across all developers.

### References

- React — eslint-plugin-react-hooks — https://ar.react.dev/reference/eslint-plugin-react-hooks
- npm — eslint-plugin-react-hooks — https://www.npmjs.com/package/eslint-plugin-react-hooks

---

## 12. Hook Execution Order

### Definitions

**Core Definition:** Hook execution order is the sequence in which Hooks run during a component's render and effect phases.

**Technical Definition:** During rendering, React calls Hooks in the order they appear in the component body. `useState` and `useMemo` run during the render phase. `useEffect` runs after the render is committed to the DOM. Effects run in the order they are declared, from child components to parent components. When a component re-renders, React first runs cleanup functions from the previous effect, then runs the new effect. Understanding this order is essential for debugging and for writing correct effects.

**Beginner-Friendly Explanation:** Hooks run in a specific order. State Hooks run during render, and effect Hooks run after the screen updates. Effects run from the inside out — child components first, then parents.

### Purposes

- To understand when effects run relative to rendering.
- To debug effect ordering issues.
- To write correct cleanup logic.
- To coordinate multiple effects within a component.

### Syntax Rules and Structure

**Execution Order:**

| Phase | What Runs |
|-------|-----------|
| Render | `useState`, `useMemo`, `useCallback` |
| Commit | React updates the DOM |
| Effects | `useEffect` (child → parent) |
| Cleanup | Previous effect cleanups run before new effects |

**Syntax Rules:**

1. All Hooks in a component run in declaration order.
2. Effects run after the render is committed.
3. Child effects run before parent effects.
4. Cleanup functions run before the next effect.

### Annotated Code Examples

```jsx
function Parent() {
  console.log('Parent render');
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('Parent effect');
    return () => console.log('Parent cleanup');
  }, [count]);

  return <Child />;
}

function Child() {
  console.log('Child render');

  useEffect(() => {
    console.log('Child effect');
    return () => console.log('Child cleanup');
  }, []);

  return <p>Child</p>;
}
```

**Expected Output (initial render):**
```
Parent render
Child render
Child effect
Parent effect
```

**Expected Output (on count change):**
```
Parent render
Child render
Parent cleanup
Parent effect
```

**Why this output:** Render functions run parent-to-child. Effects run child-to-parent. Cleanup runs before the next effect.

### Real-World Cases

- **Debugging:** Understanding why effects run in a particular order.
- **Data fetching:** Coordinating multiple fetches across components.
- **Subscriptions:** Managing cleanup order.

### References

- React — Lifecycle of Reactive Effects — https://react.dev/learn/lifecycle-of-reactive-effects
- Egghead — Understand the React Hook Flow — https://egghead.io

---

## 13. Hook State

### Definitions

**Core Definition:** Hook state is data that a Hook retains between renders, most commonly managed by `useState` and `useReducer`.

**Technical Definition:** Hook state is the data stored in React's internal memory slots associated with each Hook call. `useState` provides a state variable and a setter. `useReducer` provides state and a dispatch function. State is local to the component instance and persists across renders. React identifies each state slot by the Hook's call order. When state changes, React re-renders the component. State is a snapshot for each render — the value never changes within a single render.

**Beginner-Friendly Explanation:** Hook state is the data your component remembers. `useState` gives you a variable and a way to change it. React keeps track of it between renders.

### Purposes

- To retain data between renders.
- To trigger re-renders when data changes.
- To manage component-specific memory.

### Syntax Rules and Structure

```jsx
const [state, setState] = useState(initialValue);
```

**Syntax Rules:**

1. State is local to the component instance.
2. State updates are asynchronous.
3. State is a snapshot for each render.
4. Never mutate state directly.

### Annotated Code Examples

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

**Expected Output:** A counter that increments by 1 on each click.

**Why this output:** `useState(0)` creates a state slot. Clicking calls `setCount(1)`, which schedules a re-render. On the next render, `count` is 1.

### Real-World Cases

- **Form inputs:** Storing field values.
- **Toggles:** Boolean on/off states.
- **Counters:** Numeric values.
- **API data:** Storing fetched data.

### References

- React — State: A Component's Memory — https://zh-hant.react.dev/learn/state-a-components-memory
- React — useState — https://react.dev/reference/react/useState

---

## 14. Hook Closure

### Definitions

**Core Definition:** Hook closure refers to the fact that Hooks capture the state and props from the render in which they were created.

**Technical Definition:** Each render creates new closures for event handlers and effects. These closures capture the state and props values from that specific render. If an asynchronous operation references a state variable, it sees the value from the render in which the closure was created, not the latest value. This is known as a "stale closure." The solution is to use functional updates or to include the state in dependency arrays so the closure is recreated when the state changes. When a value referenced inside a Hook isn't included in the dependency array, React won't re-run the effect or recalculate the value when that dependency changes — this causes stale closures where the Hook uses outdated values.

**Beginner-Friendly Explanation:** Hooks remember the values from when they were created. If you read state inside a `setTimeout` or promise, you get the old value, not the new one.

### Purposes

- To understand why async code sometimes reads stale state.
- To fix stale closure bugs with functional updates.
- To correctly manage dependencies in `useEffect` and `useCallback`.

### Syntax Rules and Structure

**Stale Closure:**

```jsx
function Component() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setTimeout(() => {
      console.log(count); // Old value!
    }, 1000);
  }
}
```

**Fix with Functional Update:**

```jsx
setCount(prev => prev + 1);
```

**Fix with Dependency Array:**

```jsx
useEffect(() => {
  console.log(count);
}, [count]); // Re-runs when count changes
```

**Syntax Rules:**

1. Closures capture state from the render in which they are created.
2. Use functional updates for state that depends on previous state.
3. Include state in `useEffect` dependencies if the effect reads it.
4. Use `useRef` for mutable values that should not trigger re-renders.

### Annotated Code Examples

```jsx
function Chat() {
  const [messages, setMessages] = useState([]);

  function sendMessage(text) {
    // ✅ Functional update: always works on the latest messages
    setMessages(prev => [...prev, text]);
  }

  return (
    <button onClick={() => sendMessage('Hello')}>
      Send ({messages.length})
    </button>
  );
}
```

**Expected Output:** Clicking the button appends "Hello" to the messages list and updates the count.

**Why this output:** The functional update `prev => [...prev, text]` receives the latest queued state, so even if multiple messages are sent quickly, none are lost. Using `setMessages([...messages, text])` would use the stale `messages` value from the render.

### Real-World Cases

- **Chat applications:** Appending messages in async callbacks.
- **Debounced search:** Reading the latest query in a debounced callback.
- **Timers:** Incrementing counters inside `setInterval`.
- **WebSocket handlers:** Updating state when messages arrive.

### References

- CoreUI — How to fix stale closures in React hooks — https://coreui.io
- React — exhaustive-deps — https://pt-br.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 15. Hook Memory

### Definitions

**Core Definition:** Hook memory is React's internal mechanism for storing state, effects, and other Hook data between renders.

**Technical Definition:** React maintains a per-component list of memory slots. Each Hook call is mapped to a slot by its position in the call order. When a component renders, React walks through the Hooks in order, reading from or writing to the corresponding memory slots. This is why the order of Hooks must be consistent — React identifies each Hook by its position, not by name. The memory slots store state values, effect dependencies, memoised values, and ref objects.

**Beginner-Friendly Explanation:** React keeps a list of memory slots for each component. The first Hook uses slot 1, the second uses slot 2, and so on. When the component re-renders, React reads from the same slots in the same order.

### Purposes

- To retain state and effect data between renders.
- To enable React to associate Hook calls with their data.
- To explain why Hook order is critical.

### Syntax Rules and Structure

**Memory Slot Mapping:**

```jsx
function Component() {
  const [count, setCount] = useState(0);  // Slot 1: state
  useEffect(() => {}, []);                 // Slot 2: effect
  const [name, setName] = useState('');    // Slot 3: state
  useMemo(() => compute(), []);            // Slot 4: memo
}
```

**Syntax Rules:**

1. Each Hook call consumes one memory slot.
2. Slots are identified by call order, not by name.
3. Changing the order of Hooks corrupts the slot mapping.
4. Conditional Hooks change the number of slots, causing corruption.

### Annotated Code Examples

```jsx
function Counter() {
  // Slot 1: stores count
  const [count, setCount] = useState(0);

  // Slot 2: stores effect state and dependencies
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

**Expected Output:** The counter increments and the title updates.

**Why this output:** React maps `useState` to slot 1 and `useEffect` to slot 2. On every render, it reads from these slots in the same order, so the state and effect are correctly preserved.

### Real-World Cases

- **All components using Hooks:** Memory slot mapping is the foundation of Hooks.
- **Debugging:** "Rendered fewer/more hooks than expected" errors indicate slot corruption.

### References

- React — Rules of Hooks: Explanation — https://vi.legacy.reactjs.org/docs/hooks-rules.html
- Patterns.dev — Hooks Pattern — https://www.patterns.dev

---

## References

- React — Rules of Hooks — https://uk.react.dev/reference/rules/rules-of-hooks
- React — eslint-plugin-react-hooks — https://ar.react.dev/reference/eslint-plugin-react-hooks
- React — eslint-plugin-react-hooks: rules-of-hooks — https://de.react.dev/reference/eslint-plugin-react-hooks/lints/rules-of-hooks
- React — eslint-plugin-react-hooks: exhaustive-deps — https://pt-br.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React — useEffect — https://zh-hant.react.dev/reference/react/useEffect
- React — State: A Component's Memory — https://zh-hant.react.dev/learn/state-a-components-memory
- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- React — Lifecycle of Reactive Effects — https://react.dev/learn/lifecycle-of-reactive-effects
- React — Rules of Hooks (Legacy) — https://vi.legacy.reactjs.org/docs/hooks-rules.html
- CoreUI — How to fix stale closures in React hooks — https://coreui.io
- npm — eslint-plugin-react-hooks — https://www.npmjs.com/package/eslint-plugin-react-hooks
- GitHub — Naming Conventions Guide — https://github.com/novalauliady/invoicegeneratorai
- Patterns.dev — Hooks Pattern — https://www.patterns.dev
- Egghead — Understand the React Hook Flow — https://egghead.io
- Syncfusion — Implementing React Custom Hooks: A Complete Guide — https://www.syncfusion.com