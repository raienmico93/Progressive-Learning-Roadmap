# `useState` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useState` is a React Hook that lets you add a state variable to a function component, allowing the component to "remember" information between renders.

**Technical Definition:** `useState` is a React Hook that accepts an initial state value and returns an array with exactly two elements: the current state value and a setter function. The setter function updates the state and triggers a re-render of the component. State is local to the component instance and persists across renders. `useState` must be called at the top level of a component or a custom Hook — never inside loops, conditions, or nested functions.

**Beginner-Friendly Explanation:** `useState` is how a component remembers things. When you call `useState`, you give it a starting value, and it gives you back two things: the current value and a function to change it. When you change the value, React updates the screen to reflect the new data. Think of it like a component's personal notebook: you write something down, you can change it later, and whenever it changes, React updates what you see.

### Key Characteristics

- **Hook-based:** Only usable in function components and custom Hooks, not in class components.
- **Local scope:** State is private to the component instance; it is not shared across components.
- **Asynchronous:** State updates are queued and do not take effect immediately.
- **Batched:** React groups multiple state updates into a single re-render for performance.
- **Immutable:** State values must never be mutated directly; new values must be created.
- **Snapshot-based:** Each render receives a fixed snapshot of state that never changes within that render.

### Prerequisites

- JavaScript fundamentals: variables, functions, objects, arrays, destructuring, spread operator.
- Basic React function component syntax.
- JSX syntax and expressions.
- Understanding of rendering and re-renders.

### Related Programming Areas

- **`useReducer`** — an alternative for managing complex state objects with multiple sub-values.
- **`useEffect`** — for synchronising state with external systems.
- **`useContext`** — for sharing state across components without prop drilling.
- **State management libraries** — Redux, Zustand, Jotai for global state.

---

## 1. `useState`

### Definitions

**Core Definition:** `useState` is a React Hook that declares a state variable in a function component.

**Technical Definition:** `useState(initialState)` returns an array with two values: the current state, initially matching `initialState`, and a set function that updates the state and triggers a re-render. The convention is to name state variables using array destructuring: `[something, setSomething]`.

**Beginner-Friendly Explanation:** `useState` is a function you call to create a piece of state. You give it a starting value, and it gives you back the current value and a function to change it.

### Purposes

- To add local state to function components.
- To retain data between renders.
- To trigger re-renders when state changes.
- To make components interactive and dynamic.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useState } from 'react';

const [state, setState] = useState(initialState);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useState(initialState)` | Hook call that declares state. |
| `state` | The current state value for this render. |
| `setState` | Function to update the state and trigger re-render. |
| `initialState` | The value used on the first render only. |

**Syntax Rules:**

1. Call `useState` at the top level of a component or custom Hook.
2. Use array destructuring to name the state variable and setter.
3. The initial value is ignored after the first render.

**Constraints and Limitations:**

- Cannot be called inside loops or conditions.
- Cannot be called in class components.
- In Strict Mode, the initializer runs twice in development.

### Annotated Code Example

```jsx
import { useState } from 'react';

function Counter() {
  // Declare state 'count' with initial value 0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**Expected Output:** A paragraph showing "Count: 0" and a button. Clicking increments the count.

**Why this output:** `useState(0)` initialises `count` to 0. Clicking calls `setCount(1)`, which schedules a re-render. On the next render, `count` is 1.

### Real-World Cases

- **Counters:** Tracking numeric values.
- **Toggles:** Tracking boolean states (open/closed, on/off).
- **Form inputs:** Storing the current value of text fields.
- **API data:** Storing fetched data for display.

### References

- React — useState — https://react.dev/reference/react/useState
- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory

---

## 2. State Initialization

### Definitions

**Core Definition:** State initialization is the process of providing an initial value when a state variable is first declared.

**Technical Definition:** The initial value passed to `useState` is used only during the first render. React ignores it on subsequent renders. If the initial value is expensive to compute, a function can be passed as an initializer; React will call it only once during initialisation and store its return value as the initial state.

**Beginner-Friendly Explanation:** When you create a state variable, you give it a starting value. This value is only used the first time the component renders.

### Purposes

- To set the starting value of a state variable.
- To avoid expensive computations on every render.
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

### Annotated Code Example

```jsx
// ❌ Expensive computation runs on every render
const [data, setData] = useState(expensiveComputation());

// ✅ Expensive computation runs only once
const [data, setData] = useState(() => expensiveComputation());

function expensiveComputation() {
  return Array.from({ length: 1000 }, (_, i) => i);
}
```

**Expected Output:** The state is initialised with an array of 1000 numbers.

**Why this output:** The function form `() => expensiveComputation()` is passed as the initializer. React calls it once during initialisation and uses the result. The non-function form runs `expensiveComputation()` on every render.

### Real-World Cases

- **Reading from localStorage:** Initialising state from persisted data.
- **Computing derived initial state:** Parsing or transforming initial props.
- **Expensive data structures:** Building large arrays or objects once.

### References

- React — useState: Avoiding recreating the initial state — https://react.dev/reference/react/useState#avoiding-recreating-the-initial-state
- Vercel — Use Lazy State Initialization — https://github.com/mdresch/adpa/blob/main/skills/vercel-react-best-practices/rules/rerender-lazy-state-init.md

---

## 3. Lazy Initialization

### Definitions

**Core Definition:** Lazy initialization is the practice of passing a function to `useState` so that the initial value is computed only when needed, not on every render.

**Technical Definition:** Without the function form, the initializer expression runs on every render even though the value is only used once. By passing a function, React calls it only during the initial render and stores its return value as the initial state.

**Beginner-Friendly Explanation:** If computing the starting value is slow, you can give React a function instead of a value. React will call that function only once, saving time on every subsequent render.

### Purposes

- To avoid wasted computation on every render.
- To improve performance for expensive initial values.
- To read from external sources (localStorage, DOM) only once.

### Syntax Rules and Structure

```jsx
const [state, setState] = useState(() => computeExpensiveValue());
```

**Syntax Rules:**

1. Pass a function reference, not a function call.
2. The initializer function takes no arguments.
3. The function should be pure.

### Annotated Code Example

```jsx
function FilteredList({ items }) {
  // ❌ buildSearchIndex() runs on EVERY render
  // const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items));

  // ✅ buildSearchIndex() runs ONLY on initial render
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items));
  const [query, setQuery] = useState('');

  return <SearchResults index={searchIndex} query={query} />;
}
```

**Expected Output:** The search index is built once and reused across renders.

**Why this output:** The lazy initializer function is called only during the first render. Subsequent renders skip the computation entirely.

### Real-World Cases

- **localStorage/sessionStorage:** Reading persisted state once.
- **Building indexes:** Creating search indexes or maps.
- **Reading from the DOM:** Measuring elements once on mount.
- **Heavy transformations:** Parsing JSON or complex calculations.

### References

- Vercel — Use Lazy State Initialization — https://github.com/mdresch/adpa/blob/main/skills/vercel-react-best-practices/rules/rerender-lazy-state-init.md
- Egghead — Use a lazy initializer with useState — https://egghead.io/lessons/react-use-a-lazy-initializer-with-usestate

---

## 4. State Updater Function

### Definitions

**Core Definition:** The state updater function is the second element returned by `useState`, used to update the state and trigger a re-render.

**Technical Definition:** `setSomething(nextState)` accepts either a new state value or a function that computes the next state from the previous one. When a function is passed, it is treated as an updater function and is placed in a queue. During the next render, React applies all queued updaters to the previous state to compute the new state.

**Beginner-Friendly Explanation:** The setter function is how you change state. You can give it the new value directly, or you can give it a function that receives the old value and returns the new one.

### Purposes

- To update a state variable's value.
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

### Annotated Code Example

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

**Why this output:** The first setter receives the current count (0) and queues an update to 1. The second setter receives the queued value (1) and queues an update to 2.

### Real-World Cases

- **Counters:** Incrementing values safely.
- **Todo lists:** Adding items based on previous state.
- **Shopping carts:** Updating quantities.
- **Form state:** Toggling checkboxes.

### References

- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React — useState: set functions — https://react.dev/reference/react/useState#setstate

---

## 5. Functional Updates

### Definitions

**Core Definition:** Functional updates are state updates where you pass a function to the setter instead of a direct value, ensuring the update is based on the latest state.

**Technical Definition:** When updating state based on the current state value, the functional update form `setState(prev => newState)` should be used instead of directly referencing the state variable. This prevents stale closures, eliminates unnecessary dependencies, and creates stable callback references.

**Beginner-Friendly Explanation:** If you need to change state based on its current value, use a function instead of the value itself. This ensures you always work with the latest state.

### Purposes

- To prevent stale closures in asynchronous code.
- To safely update state multiple times in one event handler.
- To eliminate unnecessary dependencies in `useCallback` and `useEffect`.
- To create stable callback references.

### Syntax Rules and Structure

```jsx
setState(prevState => nextState);
```

**Syntax Rules:**

1. The updater function receives the pending state as its only argument.
2. It must be pure and return the next state.
3. Use functional updates whenever the new state depends on the previous state.

### Annotated Code Example

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

**Expected Output:** Clicking appends "Hello" to the messages list and updates the count.

**Why this output:** The functional update `prev => [...prev, text]` receives the latest queued state, so no messages are lost even if multiple messages are sent quickly.

### Real-World Cases

- **Chat applications:** Appending messages safely.
- **Debounced search:** Reading the latest query in a debounced callback.
- **Timers:** Incrementing counters inside `setInterval`.
- **WebSocket handlers:** Updating state when messages arrive.

### References

- Vercel — Use Functional setState Updates — https://github.com/sickn33/agentic-awesome-skills/blob/main/skills/react-best-practices/rules/rerender-functional-setstate.md
- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates

---

## 6. State Destructuring

### Definitions

**Core Definition:** State destructuring is the use of array destructuring to extract the state value and setter from the array returned by `useState`.

**Technical Definition:** `useState` returns an array with exactly two elements. Array destructuring assigns these to named variables: `const [state, setState] = useState(initialState)`. The convention is to name the setter with a `set` prefix followed by the state name.

**Beginner-Friendly Explanation:** `useState` gives you back an array of two things. Destructuring lets you give them names, like `[count, setCount]`.

### Purposes

- To give meaningful names to the state value and setter.
- To follow React's naming convention.
- To make code readable and consistent.

### Syntax Rules and Structure

```jsx
const [something, setSomething] = useState(initialValue);
```

**Syntax Rules:**

1. The first element is the current state.
2. The second element is the setter function.
3. The naming convention is `[thing, setThing]`.

### Annotated Code Example

```jsx
function Profile() {
  const [name, setName] = useState('Alice');
  const [age, setAge] = useState(30);
  const [isAdmin, setIsAdmin] = useState(false);

  return (
    <div>
      <p>{name}, {age}</p>
      <button onClick={() => setIsAdmin(!isAdmin)}>
        {isAdmin ? 'Admin' : 'User'}
      </button>
    </div>
  );
}
```

**Expected Output:** A profile showing name and age, with a toggle for admin status.

**Why this output:** Each `useState` call returns an array that is destructured into a named value and setter.

### Real-World Cases

- **All function components:** Destructuring is the standard way to use `useState`.
- **Multiple state variables:** Each state variable gets its own `useState` call.

### References

- React — useState — https://react.dev/reference/react/useState

---

## 7. Multiple State Variables

### Definitions

**Core Definition:** Multiple state variables are separate `useState` calls used to manage independent pieces of state within a single component.

**Technical Definition:** React allows multiple `useState` calls in the same component. Each call creates an independent state variable with its own setter. This is preferred over storing everything in a single object because it keeps state updates simple and avoids unnecessary object spreading.

**Beginner-Friendly Explanation:** You can have as many state variables as you need. Each one is separate and independent.

### Purposes

- To manage multiple independent pieces of state.
- To keep state updates simple and focused.
- To avoid unnecessary object spreading.
- To improve readability and maintainability.

### Syntax Rules and Structure

```jsx
const [name, setName] = useState('');
const [age, setAge] = useState(0);
const [isAdmin, setIsAdmin] = useState(false);
```

**Syntax Rules:**

1. Each `useState` call creates a separate state variable.
2. State variables are independent of each other.
3. Use multiple state variables when the values do not change together.

### Annotated Code Example

```jsx
function UserForm() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    console.log({ firstName, lastName, email });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={firstName} onChange={(e) => setFirstName(e.target.value)} placeholder="First" />
      <input value={lastName} onChange={(e) => setLastName(e.target.value)} placeholder="Last" />
      <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** A form with three independent inputs.

**Why this output:** Each field has its own state variable and setter, so updating one field does not affect the others.

### Real-World Cases

- **Forms:** Separate state for each field.
- **UI state:** Separate booleans for different UI toggles.
- **Pagination:** Separate state for page and page size.

### References

- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure

---

## 8. State with Objects

### Definitions

**Core Definition:** State with objects involves storing an object as a state value, requiring immutable updates when changing any property.

**Technical Definition:** Unlike class component `setState`, `useState` does not automatically merge update objects. You must create a new object by spreading the existing state and overriding the changed properties. React compares state by reference, so mutating an object does not trigger a re-render.

**Beginner-Friendly Explanation:** If your state is an object, you cannot change just one property. You have to copy the whole object, change the property you want, and give the new object to React.

### Purposes

- To store related data together in a single state variable.
- To manage complex data structures like user profiles or settings.

### Syntax Rules and Structure

```jsx
const [user, setUser] = useState({ name: 'Alice', age: 30 });

// Update one property
setUser({ ...user, name: 'Bob' });
```

**Syntax Rules:**

1. Always spread the existing object when updating.
2. Never mutate nested objects directly; copy each level that changes.
3. Shallow copying only copies the top level.

### Annotated Code Example

```jsx
function UserProfile() {
  const [user, setUser] = useState({
    name: 'Alice',
    address: { city: 'London', country: 'UK' },
  });

  function updateCity(newCity) {
    // ✅ Copy each level that changes
    setUser({
      ...user,
      address: {
        ...user.address,
        city: newCity,
      },
    });
  }

  return (
    <div>
      <p>{user.name} lives in {user.address.city}</p>
      <button onClick={() => updateCity('Paris')}>Move to Paris</button>
    </div>
  );
}
```

**Expected Output:** The city updates to Paris when the button is clicked.

**Why this output:** A new top-level object and a new nested `address` object are created, so React detects the change and re-renders.

### Real-World Cases

- **User profiles:** Storing name, email, address.
- **Settings:** Storing multiple configuration options.
- **Form state:** Storing form data as an object.

### References

- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state

---

## 9. State with Arrays

### Definitions

**Core Definition:** State with arrays involves storing an array as a state value, requiring immutable updates when adding, removing, or modifying items.

**Technical Definition:** Arrays in React state must be treated as immutable. Methods like `push()`, `pop()`, `splice()`, and `sort()` mutate the original array and must be avoided. Instead, create a new array using the spread operator or methods like `filter()`, `map()`, and `concat()`.

**Beginner-Friendly Explanation:** If your state is an array, you cannot use `push` to add items. You have to create a new array with the changes you want.

### Purposes

- To store lists of data (todos, messages, items).
- To manage dynamic collections that change over time.

### Syntax Rules and Structure

```jsx
const [items, setItems] = useState([]);

// Add
setItems([...items, newItem]);

// Remove
setItems(items.filter(i => i.id !== id));

// Update
setItems(items.map(i => i.id === id ? { ...i, done: true } : i));
```

**Syntax Rules:**

1. Never mutate the array directly.
2. Use spread to create a new array.
3. Use `filter`, `map`, and `concat` for transformations.

### Annotated Code Example

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  function addTodo() {
    setTodos([...todos, { id: Date.now(), text: input, done: false }]);
    setInput('');
  }

  function toggleTodo(id) {
    setTodos(todos.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    ));
  }

  function removeTodo(id) {
    setTodos(todos.filter(t => t.id !== id));
  }

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(t => (
          <li key={t.id}>
            <span style={{ textDecoration: t.done ? 'line-through' : 'none' }}
                  onClick={() => toggleTodo(t.id)}>
              {t.text}
            </span>
            <button onClick={() => removeTodo(t.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** A todo list where items can be added, toggled, and removed.

**Why this output:** Each operation creates a new array, so React detects the change and re-renders.

### Real-World Cases

- **Todo lists:** Adding, toggling, and removing items.
- **Chat messages:** Appending new messages.
- **Shopping carts:** Adding and removing products.
- **Data tables:** Sorting and filtering rows.

### References

- React — Updating Arrays in State — https://react.dev/learn/updating-arrays-in-state

---

## 10. State with Primitives

### Definitions

**Core Definition:** State with primitives involves storing simple values like numbers, strings, or booleans as state.

**Technical Definition:** Primitives (numbers, strings, booleans, `null`, `undefined`) are immutable by nature. Updating primitive state simply means passing the new value to the setter. No spreading or copying is required.

**Beginner-Friendly Explanation:** For simple values like numbers and strings, you just give the new value to the setter.

### Purposes

- To track simple values like counts, names, and toggles.
- To avoid unnecessary complexity for simple data.

### Syntax Rules and Structure

```jsx
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [isOn, setIsOn] = useState(false);
```

**Syntax Rules:**

1. Primitive state updates are direct.
2. No spreading or copying is needed.
3. Use functional updates when the new value depends on the previous value.

### Annotated Code Example

```jsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}
```

**Expected Output:** A button that toggles between ON and OFF.

**Why this output:** `isOn` is a boolean primitive. The setter is called with the new boolean value.

### Real-World Cases

- **Counters:** Incrementing numeric values.
- **Toggles:** Boolean on/off states.
- **Text inputs:** String values.
- **Loading flags:** Boolean loading states.

### References

- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory

---

## 11. State Batching

### Definitions

**Core Definition:** State batching is React's practice of grouping multiple state updates into a single re-render for performance.

**Technical Definition:** When multiple state setters are called within the same event handler, React batches them and performs a single re-render after all the code has run. Since React 18, automatic batching is enabled by default, meaning updates are batched even inside promises, `setTimeout`, and native event handlers.

**Beginner-Friendly Explanation:** If you change several pieces of state at once, React waits and updates everything together in one go.

### Purposes

- To improve performance by reducing re-renders.
- To avoid inconsistent UI states during multiple updates.
- To ensure state updates from a single event are processed together.

### Syntax Rules and Structure

```jsx
function handleClick() {
  setCount(c => c + 1);   // Queued
  setFlag(f => !f);       // Queued
  // React batches these into ONE re-render
}
```

**Syntax Rules:**

1. React batches updates within event handlers by default.
2. React 18 batches all updates by default, including async code.
3. To opt out, use `flushSync` (rarely recommended).

### Annotated Code Example

```jsx
function BatchingExample() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount(c => c + 1);
    setFlag(f => !f);
    // React batches these into ONE re-render
  }

  return (
    <button onClick={handleClick}>
      Count: {count}, Flag: {String(flag)}
    </button>
  );
}
```

**Expected Output:** Clicking updates both `count` and `flag` in a single re-render.

**Why this output:** Both setters are called in the same event handler. React queues both updates and processes them together after the handler finishes.

### Real-World Cases

- **Form submissions:** Updating multiple fields before a single re-render.
- **Complex UI state:** Toggling multiple flags together.
- **Data fetching:** Setting data, loading, and error states together.

### References

- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React Blog — Automatic batching in React 18 — https://reactjs.org/blog/2022/03/29/react-v18.html

---

## 12. State Update Ordering

### Definitions

**Core Definition:** State update ordering refers to the sequence in which queued state updates are processed by React.

**Technical Definition:** React processes state updates in the order they were queued. When multiple updater functions are queued for the same state variable, they are applied in order, with each receiving the result of the previous update. React may batch updates, but it does not change their order.

**Beginner-Friendly Explanation:** If you update the same state multiple times, React processes them in the order you called them.

### Purposes

- To understand how multiple updates to the same state are processed.
- To ensure correct state transitions when multiple updates are queued.

### Syntax Rules and Structure

```jsx
setCount(c => c + 1); // Applied first
setCount(c => c + 1); // Applied second
// Final: count increased by 2
```

**Syntax Rules:**

1. Updates are processed in the order they were queued.
2. Each updater receives the result of the previous update.
3. Batching does not change the order of updates.

### Annotated Code Example

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(c => c + 1);  // Queue 1: 0 → 1
    setCount(c => c + 1);  // Queue 2: 1 → 2
    setCount(c => c + 1);  // Queue 3: 2 → 3
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

**Expected Output:** Clicking increments the count by 3.

**Why this output:** React processes the three queued updaters in order, each receiving the result of the previous one. The final state is 3.

### Real-World Cases

- **Multi-step updates:** Incrementing by multiple values in one event.
- **Complex state transitions:** Applying multiple transformations in sequence.

### References

- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates

---

## 13. State Update Pitfalls

### Definitions

**Core Definition:** State update pitfalls are common mistakes developers make when updating React state, leading to bugs or unexpected behaviour.

**Technical Definition:** Common pitfalls include direct mutation of state, stale closures in async code, relying on state immediately after setting it, storing redundant or derived state, and violating the Rules of Hooks. Understanding these pitfalls is essential for writing correct React code.

**Beginner-Friendly Explanation:** These are the traps that catch developers when working with state.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Direct mutation | No re-render; state appears unchanged | Create a new object/array |
| Stale closure | Async code reads old state | Use functional updates |
| Reading state after setter | Gets old value | Use `useEffect` to react to changes |
| Redundant state | Synchronisation bugs | Derive during render |
| Conditional Hook | Breaks Hook order | Always call Hooks at the top level |
| State in render | Infinite loop | Use `useEffect` for side effects |

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

**Expected Output:** The correct versions behave predictably; the incorrect versions cause bugs.

**Why this output:** Direct mutation does not trigger re-renders. Stale closures read old state. Redundant state desynchronises.

### Real-World Cases

- **Debugging:** Identifying why state is not updating.
- **Code reviews:** Catching common state mistakes.
- **Performance:** Avoiding unnecessary re-renders from redundant state.

### References

- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- Refine — 5 Most Common useState Mistakes — https://refine.dev/blog/common-usestate-mistakes

---

## 14. `useState` Best Practices

### Definitions

**Core Definition:** `useState` best practices are guidelines for using the Hook effectively, safely, and maintainably.

**Technical Definition:** Best practices encompass using functional updates when state depends on previous state, avoiding redundant and derived state, using lazy initialization for expensive initial values, grouping related state logically, and keeping state as local as possible. These practices improve code quality, prevent bugs, and align with React's recommendations.

**Beginner-Friendly Explanation:** Best practices are the "do's and don'ts" that experienced React developers follow to keep their code clean and bug-free.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| Use functional updates | When new state depends on previous state. |
| Avoid redundant state | Derive values during render instead of storing them. |
| Use lazy initialization | For expensive initial values. |
| Keep state local | Lift state only when necessary. |
| Group related state | Use objects for values that change together. |
| Avoid contradictory state | Do not store values that can be derived from each other. |
| Use multiple state variables | For independent values. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Functional update for dependent state
setCount(c => c + 1);

// Lazy initialization for expensive values
const [index, setIndex] = useState(() => buildIndex(items));

// Derived state computed during render
const total = items.reduce((sum, item) => sum + item.price, 0);

// Multiple independent state variables
const [name, setName] = useState('');
const [age, setAge] = useState(0);
```

**❌ Bad Practices:**

```jsx
// Stale closure
setCount(count + 1);

// Expensive computation on every render
const [index, setIndex] = useState(buildIndex(items));

// Redundant state
const [total, setTotal] = useState(0);

// Contradictory state
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
```

**Expected Output:** The good practices produce predictable, performant code.

**Why this output:** Functional updates prevent stale closures. Lazy initialization avoids wasted computation. Derived state stays in sync. Multiple state variables keep updates simple.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Code reviews:** Best practices are enforced through ESLint and review checklists.
- **Performance optimisation:** Avoiding redundant state and using lazy initialization improves performance.

### References

- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- LogRocket — React Hooks cheat sheet: Best practices — https://blog.logrocket.com/react-hooks-cheat-sheet-simple-examples
- Vercel — React Best Practices — https://github.com/sickn33/agentic-awesome-skills

---

## References

- React — useState — https://react.dev/reference/react/useState
- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory
- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — Updating Arrays in State — https://react.dev/learn/updating-arrays-in-state
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React Blog — Automatic batching in React 18 — https://reactjs.org/blog/2022/03/29/react-v18.html
- Vercel — Use Lazy State Initialization — https://github.com/mdresch/adpa/blob/main/skills/vercel-react-best-practices/rules/rerender-lazy-state-init.md
- Vercel — Use Functional setState Updates — https://github.com/sickn33/agentic-awesome-skills/blob/main/skills/react-best-practices/rules/rerender-functional-setstate.md
- Refine — 5 Most Common useState Mistakes — https://refine.dev/blog/common-usestate-mistakes
- LogRocket — React Hooks cheat sheet: Best practices — https://blog.logrocket.com/react-hooks-cheat-sheet-simple-examples
- Egghead — Use a lazy initializer with useState — https://egghead.io/lessons/react-use-a-lazy-initializer-with-usestate