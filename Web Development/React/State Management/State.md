# React State Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** State is a component's internal, mutable data that persists across renders and drives the component's behaviour and appearance.

**Technical Definition:** State in React is data managed within a component that can change over time, typically in response to user interactions or asynchronous events. It is declared using the `useState` Hook in function components (or `this.state` in class components) and updated via a setter function. When state changes, React re-renders the component and its children to reflect the new data. State is private to the component that owns it and is not accessible by other components unless explicitly passed down as props. State updates are asynchronous, batched, and must be performed immutably — the existing state value must never be mutated directly.

**Beginner-Friendly Explanation:** State is a component's memory. It lets a component remember things that change over time, such as what the user typed into a form, whether a modal is open, or how many items are in a shopping cart. When the memory changes, React updates what you see on screen. State is like a personal notebook that each component keeps for itself.

### Key Characteristics

- **Local and private:** State belongs to the component that declares it and cannot be read or modified by other components.
- **Mutable (via setters):** State values can change, but only through the setter function returned by `useState`.
- **Asynchronous:** State updates are queued and processed after the current event handler completes, not immediately.
- **Batched:** React groups multiple state updates into a single re-render for performance.
- **Immutable snapshots:** Each render receives a snapshot of state; the value never changes within a single render.
- **Trigger for re-renders:** Calling a state setter schedules a re-render of the component.
- **Not for derived data:** Values computable from props or existing state should not be stored in state.

### Prerequisites

- JavaScript fundamentals: variables, functions, objects, arrays, destructuring.
- Basic React component syntax (function components).
- JSX syntax and expressions.
- Understanding of props and one-way data flow.

### Related Programming Areas

- **Props** — read-only inputs passed from parent to child; unlike state, props are immutable and externally owned.
- **Hooks** — `useState`, `useReducer`, and `useContext` for managing state in function components.
- **State management libraries** — Redux, Zustand, Jotai for global state.
- **Rendering and reconciliation** — how React determines what to update in the DOM.
- **Immutability** — the principle of replacing rather than mutating state values.

### Core Concepts / Features

---

## 1. State

### Definitions

**Core Definition:** State is data that a component manages internally and that can change over time, triggering re-renders.

**Technical Definition:** State is a component's memory. It allows components to remember information and respond to user interactions by re-rendering with updated data. State is declared using `useState` in function components, which returns an array with the current state value and a setter function. State updates are asynchronous and batched, and the state value is a snapshot fixed for a given render.

**Beginner-Friendly Explanation:** State is like a component's personal notebook. It writes things down, changes them when needed, and reads them back. When the notebook changes, the component updates what it shows.

### Purposes

- To store data that changes over time (user input, API responses, UI toggles).
- To trigger re-renders when data changes so the UI stays in sync.
- To make components interactive and dynamic.
- To persist values between renders within a component's lifetime.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useState } from 'react';

const [state, setState] = useState(initialValue);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useState(initialValue)` | Hook that declares a state variable. |
| `state` | The current state value for this render. |
| `setState` | Function that updates the state and schedules a re-render. |
| `initialValue` | The value used on the first render only. |

**Syntax Rules:**

1. `useState` must be called at the top level of a component, never inside loops or conditions.
2. The convention is to name state variables `[something, setSomething]` using array destructuring.
3. The setter replaces the entire state value for primitives, but for objects, you must create a new object.

**Constraints and Limitations:**

- `useState` cannot be called conditionally.
- In Strict Mode, React calls the initializer function twice in development to detect impurities.
- State updates do not take effect immediately within the same render.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic State with Counter

```jsx
import { useState } from 'react';

function Counter() {
  // Declare state variable 'count' with initial value 0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      {/* Calling setCount schedules a re-render */}
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**Expected Output:** A paragraph showing "Count: 0" and a button. Clicking the button updates the count.

**Why this output:** `useState(0)` initialises `count` to 0. Clicking calls `setCount(1)`, which schedules a re-render. On the next render, `count` is 1, and the paragraph updates.

#### Example 2: Toggle State

```jsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <div>
      <p>The switch is {isOn ? 'ON' : 'OFF'}</p>
      <button onClick={() => setIsOn(!isOn)}>
        Toggle
      </button>
    </div>
  );
}
```

**Expected Output:** The switch is OFF initially. Clicking the button toggles between ON and OFF.

**Why this output:** `isOn` starts as `false`. The ternary renders "OFF". Clicking sets `isOn` to `true`, re-rendering with "ON".

### Real-World Cases

- **Form inputs:** Storing the current value of text fields.
- **Modal visibility:** Tracking whether a dialog is open or closed.
- **Shopping carts:** Tracking items added to a cart.
- **API data:** Storing fetched data for display.

### References

- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory
- React — useState — https://react.dev/reference/react/useState

---

## 2. Component State

### Definitions

**Core Definition:** Component state is state that is declared and managed within a single component.

**Technical Definition:** Component state is local state owned by a specific component instance. It is declared with `useState` and is completely private to that component — parent and sibling components cannot access it directly. When state changes, React re-renders the component and its descendants. Component state is the default and most common form of state in React.

**Beginner-Friendly Explanation:** Component state is a component's own private data. No other component can see it or change it unless the component chooses to share it through props.

### Purposes

- To encapsulate data within a single component.
- To prevent other components from accidentally modifying internal data.
- To enable component-level interactivity without global state.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Component() {
  const [data, setData] = useState(initialValue);
  // data is accessible only within this component
}
```

**Syntax Rules:**

1. State is declared inside the component function.
2. Each instance of the component gets its own independent state.
3. State is not shared between component instances.

**Constraints and Limitations:**

- Component state cannot be accessed by other components.
- To share state, it must be lifted up to a common ancestor or stored in context/global state.

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

// Two independent instances
function App() {
  return (
    <div>
      <Counter />  {/* Has its own count */}
      <Counter />  {/* Has its own separate count */}
    </div>
  );
}
```

**Expected Output:** Two independent counters, each with its own count.

**Why this output:** Each `<Counter />` instance gets its own `useState` call and its own state variable. Updating one does not affect the other.

### Real-World Cases

- **Reusable components:** Each instance of a form input manages its own value.
- **Independent widgets:** Multiple charts, each with its own filter state.
- **Tab components:** Each tab group manages its own active tab.

### References

- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components

---

## 3. Local State

### Definitions

**Core Definition:** Local state is state that exists within a single component and is not shared outside it.

**Technical Definition:** Local state is synonymous with component state. It is the default scope of `useState`. Local state is appropriate when the data is only needed by one component and does not need to be synchronised with other parts of the application. Keeping state local is preferred because it minimises the surface area for bugs and improves performance by limiting re-renders to the smallest necessary subtree.

**Beginner-Friendly Explanation:** Local state is data that only one component cares about. It stays in that component and does not bother anyone else.

### Purposes

- To keep data scoped to the component that needs it.
- To minimise re-renders by keeping state local.
- To avoid unnecessary complexity from shared state.

### Syntax Rules and Structure

```jsx
function SearchBar() {
  const [query, setQuery] = useState('');
  // query is local to SearchBar
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

**Syntax Rules:**

1. Local state is declared with `useState` inside the component.
2. It should be used when no other component needs the data.
3. If multiple components need the data, lift it up.

### Annotated Code Examples

```jsx
function Accordion() {
  const [openIndex, setOpenIndex] = useState(null);

  return (
    <div>
      {[1, 2, 3].map((n) => (
        <div key={n}>
          <button onClick={() => setOpenIndex(openIndex === n ? null : n)}>
            Section {n}
          </button>
          {openIndex === n && <p>Content for section {n}</p>}
        </div>
      ))}
    </div>
  );
}
```

**Expected Output:** An accordion where clicking a section button opens its content.

**Why this output:** `openIndex` is local to `Accordion`. No other component needs to know which section is open.

### Real-World Cases

- **Accordions:** Tracking which section is open.
- **Tabs:** Tracking the active tab within a tab group.
- **Form inputs:** Storing the current value of a field.

### References

- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components

---

## 4. Global State

### Definitions

**Core Definition:** Global state is state that is shared across multiple components throughout an application.

**Technical Definition:** Global state refers to data that needs to be accessed by many components at different levels of the component tree. In React, global state is typically managed using the Context API (with `useContext`) or third-party libraries such as Redux, Zustand, or Jotai. Global state should be used sparingly — only for data that genuinely needs to be shared broadly, such as authentication status, theme, or locale.

**Beginner-Friendly Explanation:** Global state is data that many components need to know about. Instead of passing it down through many levels, it is made available everywhere.

### Purposes

- To share data across many components without prop drilling.
- To manage application-wide concerns (theme, auth, locale).
- To centralise complex state logic in one place.

### Syntax Rules and Structure

**Context API Example:**

```jsx
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  return useContext(ThemeContext);
}
```

**Syntax Rules:**

1. Create a context with `createContext()`.
2. Wrap the application or subtree with a Provider.
3. Consume with `useContext()` in any descendant.

### Annotated Code Examples

```jsx
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}

function Navbar() {
  const { user, setUser } = useAuth();
  return (
    <nav>
      {user ? (
        <button onClick={() => setUser(null)}>Logout</button>
      ) : (
        <button onClick={() => setUser({ name: 'Alice' })}>Login</button>
      )}
    </nav>
  );
}
```

**Expected Output:** A navbar that shows "Login" when logged out and "Logout" when logged in.

**Why this output:** The `AuthProvider` makes `user` and `setUser` available to all descendants. `Navbar` consumes the auth state via `useAuth()`.

### Real-World Cases

- **Authentication:** Current user and permissions.
- **Theming:** Light/dark mode, brand colours.
- **Localisation:** Language and locale settings.
- **Shopping carts:** Cart contents shared across pages.

### References

- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- React — Scaling Up with Reducer and Context — https://react.dev/learn/scaling-up-with-reducer-and-context

---

## 5. Derived State

### Definitions

**Core Definition:** Derived state is data that can be computed from existing state or props rather than stored as separate state.

**Technical Definition:** Derived state is any value that can be calculated from existing data, whether from props or other state variables. Storing derived state redundantly leads to synchronization bugs, extra memory usage, and unnecessary re-renders. React recommends computing derived values during render instead of storing them in state. If computation is expensive, `useMemo` can be used to cache the result.

**Beginner-Friendly Explanation:** Derived state is information you can figure out from other information you already have. Instead of storing it separately, you just calculate it when you need it.

### Purposes

- To avoid redundant state and synchronization bugs.
- To reduce the number of state variables and re-renders.
- To keep a single source of truth for data.

### Syntax Rules and Structure

**Compute During Render:**

```jsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');

// Derived: computed during render
const fullName = `${firstName} ${lastName}`;
```

**Use `useMemo` for Expensive Computation:**

```jsx
const visibleTodos = useMemo(
  () => todos.filter((t) => t.completed),
  [todos]
);
```

**Syntax Rules:**

1. Compute derived values during render, not in state.
2. Use `useMemo` only for expensive computations.
3. Do not store derived values in `useState`.

**Constraints and Limitations:**

- `useMemo` is a performance optimisation, not a semantic guarantee.
- Overusing `useMemo` can add unnecessary complexity.

### Annotated Code Examples

**❌ Redundant State (Anti-Pattern):**

```jsx
function Cart() {
  const [items, setItems] = useState([]);
  const [total, setTotal] = useState(0); // Redundant!

  // Must manually keep total in sync — easy to forget
  function addItem(item) {
    setItems([...items, item]);
    setTotal(total + item.price);
  }
}
```

**✅ Derived State (Correct):**

```jsx
function Cart() {
  const [items, setItems] = useState([]);

  // Derived during render — always in sync
  const total = items.reduce((sum, item) => sum + item.price, 0);

  function addItem(item) {
    setItems([...items, item]);
  }
}
```

**Expected Output:** Both versions display the total. The derived version is always correct because it is computed from `items`.

**Why this output:** In the redundant version, forgetting to update `total` causes desynchronisation. In the derived version, `total` is recalculated from `items` on every render.

### Real-World Cases

- **Cart totals:** Summing item prices.
- **Filtered lists:** Filtering todos by completion status.
- **Formatted dates:** Converting timestamps to display strings.
- **Validation results:** Checking if a form is valid based on field values.

### References

- React Blog — You Probably Don't Need Derived State — https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure

---

## 6. State vs Props

### Definitions

**Core Definition:** State and props are the two primary ways data flows in React; state is internal and mutable, props are external and immutable.

**Technical Definition:** Props (short for "properties") are passed to a component from its parent (similar to function arguments), while state is managed within the component (similar to variables declared inside a function). Props are immutable from the child's perspective and flow one-way from parent to child. State is mutable (via setters) and is private to the component that owns it. Both are plain JavaScript objects, but they serve different roles in React's data flow.

**Beginner-Friendly Explanation:** Props are like ingredients given to a chef; state is like the notes the chef writes while cooking. Props come from outside and cannot be changed; state is managed inside and can change.

### Purposes

- **Props:** To pass data and configuration into a component from its parent.
- **Props:** To enable one-way data flow and make components predictable.
- **State:** To manage data that changes over time within a component.
- **State:** To trigger re-renders when data changes.

### Syntax Rules and Structure

| Aspect | Props | State |
|--------|-------|-------|
| Ownership | Parent component | The component itself |
| Mutability | Immutable | Mutable via setter |
| Access | Function parameter | `useState` return value |
| Triggers re-render | Yes (when parent re-renders) | Yes (when setter is called) |
| Purpose | Configuration and data input | Internal dynamic data |

**Syntax Rules:**

1. Props are received as function arguments; state is declared with `useState`.
2. Never mutate props; never mutate state directly.
3. Props flow down; state stays local unless lifted.

### Annotated Code Examples

```jsx
function UserCard({ name, role }) {        // Props
  const [isExpanded, setIsExpanded] = useState(false);  // State

  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        {isExpanded ? 'Collapse' : 'Expand'}
      </button>
      {isExpanded && <p>Additional details...</p>}
    </div>
  );
}

// Usage
<UserCard name="Alice" role="Engineer" />
```

**Expected Output:** A user card showing the name and role, with an expand/collapse button.

**Why this output:** `name` and `role` are props passed from the parent. `isExpanded` is local state that toggles the details section.

### Real-World Cases

- **Form components:** Props pass initial values; state tracks current values.
- **List items:** Props pass item data; state tracks selection or expansion.
- **Layout components:** Props pass content; state tracks UI state like sidebar open/closed.

### References

- React — Component State (Legacy) — https://legacy.reactjs.org/docs/faq-state.html
- React — Props and State — https://mintlify.wiki/facebook/react/guides/props-and-state

---

## 7. State Initialization

### Definitions

**Core Definition:** State initialization is the process of providing an initial value when a state variable is first declared.

**Technical Definition:** The initial value passed to `useState` is used only during the first render. React ignores it on subsequent renders. If the initial value is expensive to compute, a function can be passed as an initializer; React will call it only once during initialisation and store its return value as the initial state. The initializer function should be pure, take no arguments, and return any type of value.

**Beginner-Friendly Explanation:** When you create a state variable, you give it a starting value. If computing that starting value is slow, you can give React a function instead, and React will only call it once.

### Purposes

- To set the starting value of a state variable.
- To avoid expensive computations on every render by using an initializer function.
- To make initial state explicit and readable.

### Syntax Rules and Structure

**Direct Initial Value:**

```jsx
const [count, setCount] = useState(0);
```

**Lazy Initializer Function:**

```jsx
const [todos, setTodos] = useState(() => createTodos());
```

**Syntax Rules:**

1. The initial value is used only on the first render.
2. Pass a function (not a function call) for expensive initialisation.
3. The initializer function must be pure and take no arguments.

**Constraints and Limitations:**

- In Strict Mode, React calls the initializer function twice in development to detect impurities.
- Passing a function call instead of a function reference causes the computation to run on every render.

### Annotated Code Examples

```jsx
// ❌ Expensive computation runs on every render
const [data, setData] = useState(expensiveComputation());

// ✅ Expensive computation runs only once
const [data, setData] = useState(() => expensiveComputation());

function expensiveComputation() {
  // Simulate expensive work
  return Array.from({ length: 1000 }, (_, i) => i);
}
```

**Expected Output:** The state is initialised with an array of 1000 numbers.

**Why this output:** The function form `() => expensiveComputation()` is passed as the initializer. React calls it once during initialisation and uses the result. The non-function form runs `expensiveComputation()` on every render before `useState` is even called.

### Real-World Cases

- **Reading from localStorage:** Initialising state from persisted data.
- **Computing derived initial state:** Parsing or transforming initial props.
- **Expensive data structures:** Building large arrays or objects once.

### References

- React — useState: Avoiding recreating the initial state — https://react.dev/reference/react/useState#avoiding-recreating-the-initial-state
- React — useState: Parameters — https://react.dev/reference/react/useState#parameters

---

## 8. State Updates

### Definitions

**Core Definition:** State updates are the process of changing a state variable's value using its setter function.

**Technical Definition:** Calling a state setter (e.g., `setCount`) tells React to re-render the component with the new value. State updates are asynchronous and batched: React waits until all code in the event handler has run before processing the updates. For updates that depend on the previous state, an updater function should be passed to the setter: `setCount(prev => prev + 1)`. This ensures the update is applied to the latest queued state rather than a stale snapshot.

**Beginner-Friendly Explanation:** To change state, you call its setter function. React then updates the component. If you want to change state based on its previous value, you pass a function to the setter.

### Purposes

- To change a state variable's value.
- To trigger a re-render with the new data.
- To queue multiple updates safely using updater functions.

### Syntax Rules and Structure

**Direct Update:**

```jsx
setCount(5);
```

**Updater Function:**

```jsx
setCount(prev => prev + 1);
```

**Syntax Rules:**

1. State updates are asynchronous; reading state immediately after setting gives the old value.
2. Use updater functions when the new state depends on the previous state.
3. Multiple updates in the same event handler are batched.

**Constraints and Limitations:**

- Do not mutate state directly; always use the setter.
- Reading state after calling the setter returns the old value for the current render.

### Annotated Code Examples

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    // ❌ Wrong: both use the same stale count (0)
    // setCount(count + 1);
    // setCount(count + 1);

    // ✅ Correct: each updater receives the latest queued value
    setCount(c => c + 1);
    setCount(c => c + 1);
  }

  return <button onClick={handleClick}>+2</button>;
}
```

**Expected Output:** Clicking the button increments the count by 2.

**Why this output:** The first setter receives the current count (0) and queues an update to 1. The second setter receives the queued value (1) and queues an update to 2. React processes the queue and re-renders with 2.

### Real-World Cases

- **Counters:** Incrementing or decrementing values.
- **Todo lists:** Adding or removing items.
- **Forms:** Updating field values.
- **Toggles:** Flipping boolean values.

### References

- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React — State as a Snapshot — https://react.dev/learn/state-as-a-snapshot

---

## 9. State Immutability

### Definitions

**Core Definition:** State immutability is the principle that state values must never be modified directly; instead, new values must be created.

**Technical Definition:** Although objects in React state are technically mutable, they should be treated as immutable — like numbers, booleans, and strings. When updating an object or array in state, a new object or array must be created (typically using the spread operator) and passed to the setter. Mutating state directly does not trigger a re-render because React compares references, not contents. Treating state as immutable enables React's rendering optimisations and prevents a whole class of bugs.

**Beginner-Friendly Explanation:** You cannot change state directly. Instead, you make a copy with the changes you want and give that copy to React. This is like editing a document by making a new version instead of scribbling on the original.

### Purposes

- To ensure React detects changes and re-renders correctly.
- To enable safe comparison of previous and next state.
- To prevent bugs caused by shared references.
- To enable React's performance optimisations.

### Syntax Rules and Structure

**Updating Objects:**

```jsx
setUser({ ...user, name: 'New Name' });
```

**Updating Arrays:**

```jsx
setItems([...items, newItem]);           // Add
setItems(items.filter(i => i.id !== id)); // Remove
setItems(items.map(i => i.id === id ? { ...i, done: true } : i)); // Update
```

**Syntax Rules:**

1. Never assign to `state.property` directly.
2. Use the spread operator to create shallow copies.
3. For nested objects, copy each level that changes.

**Constraints and Limitations:**

- Shallow copying only copies the top level; nested objects still share references.
- Mutating state does not trigger re-renders and leads to stale UI.

### Annotated Code Examples

**❌ Mutation (Anti-Pattern):**

```jsx
function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div
      onPointerMove={(e) => {
        position.x = e.clientX;  // ❌ Mutation — no re-render!
        position.y = e.clientY;
      }}
    >
      <div style={{ transform: `translate(${position.x}px, ${position.y}px)` }} />
    </div>
  );
}
```

**✅ Immutable Update (Correct):**

```jsx
function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div
      onPointerMove={(e) => {
        setPosition({ x: e.clientX, y: e.clientY });  // ✅ New object
      }}
    >
      <div style={{ transform: `translate(${position.x}px, ${position.y}px)` }} />
    </div>
  );
}
```

**Expected Output:** The dot follows the pointer only in the correct version.

**Why this output:** In the mutation version, React does not know the object changed because the reference is the same. In the immutable version, a new object is passed to `setPosition`, triggering a re-render.

### Real-World Cases

- **Todo lists:** Adding, removing, and updating items.
- **Forms:** Updating nested form fields.
- **Game state:** Moving pieces on a board.
- **Data grids:** Editing rows and columns.

### References

- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — Updating Arrays in State — https://react.dev/learn/updating-arrays-in-state

---

## 10. State Batching

### Definitions

**Core Definition:** State batching is React's practice of grouping multiple state updates into a single re-render for performance.

**Technical Definition:** When multiple state setters are called within the same event handler, React batches them and performs a single re-render after all the code has run. Starting from React 18, automatic batching is enabled by default, meaning updates are batched even inside promises, `setTimeout`, and native event handlers. Batching prevents unnecessary re-renders and avoids "half-finished" renders where only some state variables have been updated.

**Beginner-Friendly Explanation:** If you change several pieces of state at once, React waits and updates everything together in one go, instead of re-rendering after each change.

### Purposes

- To improve performance by reducing the number of re-renders.
- To avoid inconsistent UI states during multiple updates.
- To ensure state updates from a single event are processed together.

### Syntax Rules and Structure

**Batched Updates:**

```jsx
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React batches these into one re-render
}
```

**Syntax Rules:**

1. React batches updates within event handlers by default.
2. React 18 batches all updates by default, including async code.
3. To opt out of batching, use `flushSync` (rarely recommended).

**Constraints and Limitations:**

- Batching means state reads immediately after setters return stale values.
- React does not batch across separate intentional events (e.g., two separate clicks).

### Annotated Code Examples

```jsx
function BatchingExample() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount(c => c + 1);   // Queued
    setFlag(f => !f);       // Queued
    // React batches these into ONE re-render
    console.log(count);     // Still old value (0)
    console.log(flag);      // Still old value (false)
  }

  return (
    <button onClick={handleClick}>
      Count: {count}, Flag: {String(flag)}
    </button>
  );
}
```

**Expected Output:** Clicking the button updates both `count` and `flag` in a single re-render.

**Why this output:** Both setters are called in the same event handler. React queues both updates and processes them together after the handler finishes. The `console.log` statements show old values because state is a snapshot for the current render.

### Real-World Cases

- **Form submissions:** Updating multiple fields before a single re-render.
- **Complex UI state:** Toggling multiple boolean flags together.
- **Data fetching:** Setting data, loading, and error states together.

### References

- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React Blog — Automatic batching in React 18 — https://reactjs.org/blog/2022/03/29/react-v18.html

---

## 11. State Updates Are Asynchronous

### Definitions

**Core Definition:** State updates are asynchronous, meaning the new state value is not available immediately after calling the setter.

**Technical Definition:** Calling a state setter does not change the state variable in the current render. Instead, it schedules a re-render. The state variable inside the current event handler continues to hold the value from the render in which it was defined. This is often confused with JavaScript's asynchronous behaviour, but the real reason is that state values are snapshots fixed for a given render. The setter queues an update for the next render, and the current render's state remains unchanged.

**Beginner-Friendly Explanation:** When you call a state setter, React does not change the state right away. The state variable you have in your code still shows the old value. The new value appears after React re-renders the component.

### Purposes

- To allow React to batch updates and optimise rendering.
- To ensure each render has a consistent snapshot of state.
- To prevent unexpected side effects during rendering.

### Syntax Rules and Structure

```jsx
function handleClick() {
  setCount(1);
  console.log(count); // Still 0 — not 1!
}
```

**Syntax Rules:**

1. The state variable always reflects the value from the current render.
2. To read the latest queued value, use an updater function: `setCount(c => c + 1)`.
3. To perform side effects after a state update, use `useEffect`.

**Constraints and Limitations:**

- Do not rely on state values immediately after setting them.
- Async code (e.g., `setTimeout`, promises) captures the state value from the render in which the closure was created.

### Annotated Code Examples

```jsx
function DelayedCounter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);        // Queues update to 1
    setTimeout(() => {
      console.log(count);       // Logs 0, not 1!
      // Because the closure captured the count from this render
    }, 1000);
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

**Expected Output:** The button shows "Count: 1" after the click, but the `setTimeout` logs `0`.

**Why this output:** The `setTimeout` callback closes over the `count` variable from the render in which `handleClick` was created. That render's `count` was 0. The state update triggers a new render with `count = 1`, but the old closure still sees 0.

### Real-World Cases

- **Async data fetching:** Using fetched data to update state.
- **Debounced inputs:** Reading state in a debounced callback.
- **Timers:** Reading state inside `setTimeout` or `setInterval`.

### References

- React — State as a Snapshot — https://react.dev/learn/state-as-a-snapshot
- React — useState: Caveats — https://react.dev/reference/react/useState#caveats

---

## 12. State Updates Are Merged

### Definitions

**Core Definition:** State updates are merged, meaning that when updating a state object, only the specified properties are changed, and the rest are preserved.

**Technical Definition:** In class components, `setState` shallowly merges the provided object into the existing state. In function components, `useState`'s setter does **not** automatically merge — it replaces the entire state value. To merge, you must manually spread the existing state into the new object: `setUser({ ...user, name: 'New' })`. This is a critical difference between class and function component state updates.

**Beginner-Friendly Explanation:** In class components, when you update one field of an object, the other fields stay the same. In function components, you have to copy the other fields yourself using the spread operator.

### Purposes

- **Class components:** To update a single field of a state object without specifying all fields.
- **Function components:** To explicitly create a new state object with the desired changes.

### Syntax Rules and Structure

**Class Component (Shallow Merge):**

```jsx
this.setState({ name: 'New' }); // Other state fields preserved
```

**Function Component (Replace — Must Spread):**

```jsx
setUser({ ...user, name: 'New' }); // Must spread to preserve other fields
```

**Syntax Rules:**

1. Class `setState` shallowly merges.
2. Function `useState` setter replaces.
3. Always spread the existing state when updating an object in function components.

### Annotated Code Examples

**❌ Function Component Without Spread (Replaces):**

```jsx
const [user, setUser] = useState({ name: 'Alice', age: 30 });

setUser({ name: 'Bob' }); // ❌ age is lost!
// Result: { name: 'Bob' } — age is undefined
```

**✅ Function Component With Spread (Merges):**

```jsx
const [user, setUser] = useState({ name: 'Alice', age: 30 });

setUser({ ...user, name: 'Bob' }); // ✅ age preserved
// Result: { name: 'Bob', age: 30 }
```

**Expected Output:** The correct version preserves the `age` field.

**Why this output:** `useState`'s setter replaces the entire state value. Without spreading, the new object only contains `name`, and `age` is lost.

### Real-World Cases

- **User profiles:** Updating a single field without losing others.
- **Form state:** Updating one field while preserving the rest.
- **Settings:** Changing one option without resetting others.

### References

- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — useState: set functions — https://react.dev/reference/react/useState#setstate

---

## 13. State Updates Are Shallow

### Definitions

**Core Definition:** State updates are shallow, meaning that only the top level of the state object is copied or compared.

**Technical Definition:** When you spread an object to create a new state value, only the top-level properties are copied. Nested objects still share references with the original. Similarly, React's comparison of previous and next state (in `React.memo`, `PureComponent`, or `useMemo`) is shallow — it checks top-level property references, not deep equality. This means that updating a nested property requires copying every level of the object that changes.

**Beginner-Friendly Explanation:** When you copy an object, only the top layer is copied. Nested objects are still shared. So if you change something deep inside, you have to copy all the layers above it too.

### Purposes

- To understand why nested state updates require careful copying.
- To avoid bugs caused by shared references in nested objects.
- To use `useMemo` and `React.memo` effectively.

### Syntax Rules and Structure

**Shallow Copy:**

```jsx
const newUser = { ...user }; // Top-level properties copied
// newUser.address === user.address (same reference!)
```

**Deep Update (Copy Each Level):**

```jsx
setUser({
  ...user,
  address: {
    ...user.address,
    city: 'New City',
  },
});
```

**Syntax Rules:**

1. Spread copies only the top level.
2. Nested objects must be copied at each level that changes.
3. Shallow comparison means nested changes are not detected by `React.memo`.

### Annotated Code Examples

```jsx
const [user, setUser] = useState({
  name: 'Alice',
  address: {
    city: 'London',
    country: 'UK',
  },
});

// ❌ Wrong: mutates nested object
user.address.city = 'Paris';
setUser(user); // Same reference — no re-render!

// ✅ Correct: copy each level
setUser({
  ...user,
  address: {
    ...user.address,
    city: 'Paris',
  },
});
```

**Expected Output:** The correct version updates the city and triggers a re-render.

**Why this output:** The mutation version changes the nested object but passes the same top-level reference to `setUser`. React compares the old and new state with `Object.is` and sees no change. The correct version creates a new top-level object and a new nested object, so React detects the change.

### Real-World Cases

- **User profiles:** Updating nested address or preferences.
- **Data grids:** Updating a cell within a row.
- **Nested forms:** Updating a field within a section.
- **Immutable data libraries:** Immer simplifies deep updates.

### References

- React — Updating Objects in State: Copying nested objects — https://react.dev/learn/updating-objects-in-state#copying-nested-objects
- React — Updating Objects in State: What's a mutation? — https://react.dev/learn/updating-objects-in-state#whats-a-mutation

---

## 14. State Updates and Closures

### Definitions

**Core Definition:** Closures in React state updates refer to the fact that event handlers capture the state value from the render in which they were created.

**Technical Definition:** Each render creates new closures for event handlers and effects. These closures capture the state values from that specific render. If an asynchronous operation (e.g., `setTimeout`, promise, `useEffect`) references a state variable, it sees the value from the render in which the closure was created, not the latest value. This is known as a "stale closure." The solution is to use functional updates (`setCount(c => c + 1)`) or to include the state in dependency arrays so the closure is recreated when the state changes.

**Beginner-Friendly Explanation:** Event handlers remember the state value from when they were created. If you read state inside a `setTimeout` or promise, you get the old value, not the new one.

### Purposes

- To understand why async code sometimes reads stale state.
- To fix stale closure bugs with functional updates.
- To correctly manage dependencies in `useEffect` and `useCallback`.

### Syntax Rules and Structure

**Stale Closure:**

```jsx
function handleClick() {
  setTimeout(() => {
    console.log(count); // Old value!
  }, 1000);
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

**Constraints and Limitations:**

- Functional updates cannot be used when you need to read other state variables.
- Dependency arrays can cause infinite loops if not managed carefully.

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

- React — State as a Snapshot — https://react.dev/learn/state-as-a-snapshot
- React — useEffect: Removing unnecessary dependencies — https://react.dev/reference/react/useEffect#removing-unnecessary-dependencies

---

## 15. State Update Pitfalls

### Definitions

**Core Definition:** State update pitfalls are common mistakes developers make when updating React state, leading to bugs, stale UI, or performance issues.

**Technical Definition:** State update pitfalls arise from React's asynchronous, batched, and immutable state model. Common pitfalls include: mutating state directly (no re-render), relying on state immediately after setting it (stale value), storing redundant or derived state (synchronisation bugs), using stale closures in async code (lost updates), and violating the Rules of Hooks (conditional `useState`). Understanding these pitfalls is essential for writing correct React code.

**Beginner-Friendly Explanation:** These are the traps that catch developers when working with state. Knowing them helps you avoid bugs.

### Purposes

- To identify and avoid common state-related bugs.
- To write correct, predictable state updates.
- To improve performance by avoiding redundant state.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Direct mutation | No re-render; state appears unchanged | Create a new object/array |
| Stale closure | Async code reads old state | Use functional updates or dependencies |
| Redundant state | Synchronisation bugs, extra renders | Derive during render |
| Conditional `useState` | Breaks Hook order; runtime errors | Always call Hooks at the top level |
| State in render | Infinite loop | Use `useEffect` for side effects |
| Reading state after setter | Gets old value | Use `useEffect` to react to changes |

### Annotated Code Examples

**Pitfall 1: Direct Mutation**

```jsx
// ❌ Wrong
items.push(newItem);
setItems(items); // Same reference — no re-render

// ✅ Correct
setItems([...items, newItem]);
```

**Pitfall 2: Stale Closure**

```jsx
// ❌ Wrong
setTimeout(() => {
  setCount(count + 1); // Uses stale count
}, 1000);

// ✅ Correct
setTimeout(() => {
  setCount(c => c + 1); // Always latest
}, 1000);
```

**Pitfall 3: Redundant State**

```jsx
// ❌ Wrong
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);

// ✅ Correct
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);
```

**Pitfall 4: Conditional Hook**

```jsx
// ❌ Wrong
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ✅ Correct
const [user, setUser] = useState(null);
if (isLoggedIn) {
  // use user
}
```

**Expected Output:** The correct versions behave predictably; the incorrect versions cause bugs.

**Why this output:** Direct mutation does not trigger re-renders. Stale closures read old state. Redundant state desynchronises. Conditional Hooks break React's internal state tracking.

### Real-World Cases

- **Debugging:** Identifying why state is not updating.
- **Code reviews:** Catching common state mistakes.
- **Performance:** Avoiding unnecessary re-renders from redundant state.

### References

- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — State as a Snapshot — https://react.dev/learn/state-as-a-snapshot
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect

---

## References

- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory
- React — useState — https://react.dev/reference/react/useState
- React — Managing State — https://mintlify.wiki/facebook/react/guides/managing-state
- React — State as a Snapshot — https://react.dev/learn/state-as-a-snapshot
- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — Updating Arrays in State — https://react.dev/learn/updating-arrays-in-state
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- React — Scaling Up with Reducer and Context — https://react.dev/learn/scaling-up-with-reducer-and-context
- React Blog — You Probably Don't Need Derived State — https://legacy.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html
- React Blog — Automatic batching in React 18 — https://reactjs.org/blog/2022/03/29/react-v18.html
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Props and State — https://mintlify.wiki/facebook/react/guides/props-and-state