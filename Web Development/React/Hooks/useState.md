# `useState` — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

`useState` is a React Hook that lets a functional component "remember" information between renders by declaring a state variable and a setter function to update it.

**Technical Definition**

`useState` accepts an initial state value (or a lazy initializer function) and returns an array pair: the current state snapshot for this render, and a setter function. React stores state externally in its Fiber tree, keyed by the Hook's call order. Calling the setter queues a re-render with the new value. State updates are asynchronous with respect to the running code—the state variable in the current render never changes; only the next render receives the updated value .

**Beginner-Friendly Explanation**

`useState` gives your component a memory. You tell it what to remember first, and it gives you back two things: the current value and a way to change it. When you change it, React re-renders your component with the new value.

### Key Characteristics

- **Snapshot Behavior**: The state variable is a constant within a single render; calling the setter does not change it in the currently executing code .
- **Lazy Initialization**: Passing a function to `useState` runs it only on the initial render, avoiding expensive recomputation .
- **Immutable Updates**: Objects and arrays in state must be replaced with new copies, never mutated directly .
- **Batched Updates**: React batches multiple state updates within the same event handler for performance .
- **Stable Setter Identity**: The setter function has a stable identity across renders, so it can be safely omitted from dependency arrays .

### Prerequisites

- JavaScript functions, arrays, and objects
- React functional components and JSX
- Basic understanding of rendering and re-rendering

### Related Programming Areas

- **`useReducer`**: An alternative for complex state logic involving multiple sub-values .
- **Props**: The other primary data source for components; unlike state, props are read-only and flow from parent to child.
- **Custom Hooks**: `useState` is the building block for many custom Hooks that encapsulate reusable stateful logic.

### Core Concepts / Features

- Initial State
- State Setters
- Functional State Updates
- Object State
- Array State
- Multiple State Variables
- State Initialization Patterns

---

## 1. Initial State

### Definitions

**Core Definition**

Initial state is the value a state variable holds on the first render, provided as the argument to `useState`.

**Technical Definition**

The argument passed to `useState` is used only during the initial render. React saves this value and ignores the argument on subsequent renders. If a function is passed, React treats it as a lazy initializer and calls it only once during initialization .

**Beginner-Friendly Explanation**

Initial state is the starting value—what the component remembers before the user does anything.

### Purposes

- To provide a starting value for a state variable.
- To avoid `undefined` errors on the first render.
- To set up the initial configuration of a component.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [state, setState] = useState(initialValue);
const [state, setState] = useState(() => computeExpensiveValue());
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `initialValue` | Direct value used on first render only. |
| `() => computeExpensiveValue()` | Lazy initializer function; called only once. |

**Syntax Rules**

1. **Direct value**: For primitives and simple literals, pass the value directly.
2. **Lazy initializer**: For expensive computations, pass a function. React calls it only during initialization .
3. **Function identity matters**: Pass `createInitialTodos`, not `createInitialTodos()` .

**Constraints and Limitations**

- The initial value is ignored after the first render.
- React may call initializer functions twice in development Strict Mode to verify purity .

### Annotated Complete Code Examples

**Example 1: Direct Initial State**

```jsx
function Counter() {
  // count starts at 0 on the first render
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Expected Output (initial)**

```
0
```

**Why This Output Occurs**

`useState(0)` stores `0` as the initial state. On the first render, `count` is `0`. React ignores the `0` argument on subsequent renders .

**Example 2: Lazy Initial State**

```jsx
function TodoList() {
  // createInitialTodos runs ONLY on the first render
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState('');

  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setTodos([...todos, { id: todos.length, text }])}>
        Add
      </button>
    </div>
  );
}
```

**Why This Works**

Passing `createInitialTodos` (the function itself) means React calls it only during initialization. If you passed `createInitialTodos()`, it would run on every render, wasting computation .

### Real-World Cases

**Case: Loading Settings from localStorage**

A component initializes state from `localStorage` using a lazy initializer to avoid parsing the stored JSON on every render .

### References

- React — Avoiding recreating the initial state - https://uk.react.dev/reference/react/useState
- Vercel Labs — Use Lazy State Initialization - https://github.com/vercel-labs/agent-skills/blob/react-best-practices/skills/react-best-practices//references/rules/rerender-lazy-state-init.md

---

## 2. State Setters

### Definitions

**Core Definition**

A state setter is the function returned by `useState` that updates the state variable and triggers a re-render.

**Technical Definition**

The setter function accepts either a new value or an updater function. When called, React queues a re-render and updates the stored state for the next render. The setter has a stable identity across renders. If the new value is identical to the current state (per `Object.is`), React skips the re-render .

**Beginner-Friendly Explanation**

The setter is how you change what your component remembers. Call it with a new value, and React will update the screen.

### Purposes

- To update state in response to user interactions.
- To trigger re-renders when data changes.
- To schedule state changes for the next render.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
setState(newValue);
setState(prevState => newValue);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `setState(newValue)` | Sets state to the given value. |
| `setState(prev => ...)` | Receives pending state and returns new state. |

**Syntax Rules**

1. **State is a snapshot**: Calling the setter does not change the variable in the current render .
2. **Stable identity**: The setter function is stable and can be safely omitted from dependency arrays .
3. **Batching**: React batches multiple setters within the same event handler.

**Constraints and Limitations**

- The setter has no return value.
- Calling the setter does not update the current render's state variable.

### Annotated Complete Code Examples

**Example 1: The Snapshot Behavior**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count); // Still 0!
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

**Expected Output (after one click)**

```
Button displays: 1
Console: 0
```

**Why This Output Occurs**

`count` is a constant for this render. `setCount` schedules a re-render with the new value, but the local `count` variable remains `0` in the running handler .

### Real-World Cases

**Case: Form Input**

An input's `onChange` calls `setName(e.target.value)` to update the name state, which re-renders the input with the new value.

### References

- React — useState API Reference (set function) - https://zh-hans.react.dev/reference/react/useState

---

## 3. Functional State Updates

### Definitions

**Core Definition**

A functional state update is passing a function to the setter that receives the pending state and returns the next state, ensuring updates are based on the latest queued value.

**Technical Definition**

When multiple updates are queued within a single event handler, passing a value to the setter uses the snapshot value from that render. Passing an updater function queues the function, and React calls them in order during the next render, each receiving the result of the previous update .

**Beginner-Friendly Explanation**

If you need to update state multiple times in a row, use a function instead of a value. That way, each update builds on the last one, instead of all using the same old value.

### Purposes

- To correctly queue multiple state updates in one handler.
- To update state based on its previous value.
- To avoid stale closures in async code.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
setState(prev => nextState);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `prev` | The pending state value. |
| `=> nextState` | Return the next state. |

**Syntax Rules**

1. **Use for sequential updates**: When multiple updates must accumulate.
2. **Use in timers/async**: Prevents stale closure issues.

### Annotated Complete Code Examples

**Example 1: Incorrect vs. Correct Sequential Updates**

```jsx
// ❌ All three use the same count (0)
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
// Result: count becomes 1

// ✅ Each receives the pending state
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
// Result: count becomes 3
```

**Explanation**

With direct values, all three calls compute `0 + 1 = 1`. With updater functions, React queues them and calls each with the latest pending state, producing `3` .

### Real-World Cases

**Case: Shopping Cart**

Adding multiple items rapidly uses `setCart(prev => [...prev, item])` to ensure each item is added to the latest cart state.

### References

- React — Updating state based on the previous state - https://zh-hans.react.dev/reference/react/useState

---

## 4. Object State

### Definitions

**Core Definition**

Object state is state whose value is a JavaScript object, requiring new object creation for updates rather than mutation.

**Technical Definition**

Objects in state are treated as immutable snapshots. To update a property, create a new object with the changed property, typically using the spread operator. React compares object references to determine if a re-render is needed .

**Beginner-Friendly Explanation**

State objects are like a sealed box. You can't change what's inside—you have to make a new box with the changes.

### Purposes

- To group related state fields into one object.
- To manage form data with multiple fields.
- To represent structured data (user, settings, configuration).

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
setState({ ...prev, field: newValue });
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `...prev` | Copy all existing properties. |
| `field: newValue` | Override the changed property. |

**Syntax Rules**

1. **Never mutate**: Do not write `state.field = value` .
2. **Spread for shallow copy**: `{ ...prev, field: value }`.
3. **Nested objects require multiple spreads**: `{ ...prev, nested: { ...prev.nested, key: value } }` .

### Annotated Complete Code Examples

**Example 1: Updating a Form Object**

```jsx
function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: '[email protected]'
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }

  return (
    <form>
      <input name="firstName" value={person.firstName} onChange={handleChange} />
      <input name="lastName" value={person.lastName} onChange={handleChange} />
      <input name="email" value={person.email} onChange={handleChange} />
    </form>
  );
}
```

**Explanation**

The spread operator copies all existing fields, and the computed property `[e.target.name]` overrides only the field that changed. This is the standard pattern for multi-field forms .

### Real-World Cases

**Case: User Settings**

A settings form stores theme, notifications, and language in one object, updating individual fields with spread.

### References

- React — Updating Objects in State - https://ar.react.dev/learn/updating-objects-in-state
- React — Immutability Lint Rule - https://de.react.dev/reference/eslint-plugin-react-hooks/lints/immutability

---

## 5. Array State

### Definitions

**Core Definition**

Array state is state whose value is a JavaScript array, requiring new array creation for updates rather than mutation.

**Technical Definition**

Arrays in state are immutable. Use non-mutating methods (`map`, `filter`, `concat`, spread) to create new arrays. Mutating methods (`push`, `pop`, `splice`, `sort`, `reverse`) must be avoided or used on a copy .

**Beginner-Friendly Explanation**

State arrays are like a numbered list on a whiteboard. To change an item, you don't erase and rewrite—you make a new list with the change.

### Purposes

- To manage collections of items (todos, messages, products).
- To add, remove, or update items in a list.
- To maintain ordered data.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// Add
setItems([...items, newItem]);
setItems(items => [...items, newItem]);

// Remove
setItems(items.filter(item => item.id !== id));

// Update
setItems(items.map(item => item.id === id ? { ...item, done: true } : item));
```

**Syntax Rules**

1. **Use `concat` or spread to add**: `[...items, newItem]` .
2. **Use `filter` to remove**: Creates a new array without the removed item.
3. **Use `map` to update**: Returns a new array with the changed item .
4. **Copy before `sort`/`reverse`**: `[...items].sort()` .

### Annotated Complete Code Examples

**Example 1: Todo List Operations**

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', done: true },
    { id: 2, text: 'Build app', done: false }
  ]);

  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text, done: false }]);
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };

  const removeTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input type="checkbox" checked={todo.done} onChange={() => toggleTodo(todo.id)} />
          {todo.text}
          <button onClick={() => removeTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

**Explanation**

Each operation creates a new array. `addTodo` spreads the old array and appends. `toggleTodo` maps and returns a new object for the changed item. `removeTodo` filters out the item .

### Real-World Cases

**Case: Chat Messages**

Messages are stored in an array, appended with spread, and filtered for deletion.

### References

- React — Updating Arrays in State - https://zh-hans.react.dev/learn/updating-arrays-in-state
- React — Immutability Lint Rule (Array push) - https://de.react.dev/reference/eslint-plugin-react-hooks/lints/immutability

---

## 6. Multiple State Variables

### Definitions

**Core Definition**

Multiple state variables means calling `useState` multiple times in a single component, each creating an independent piece of state.

**Technical Definition**

React identifies each `useState` call by its order of execution during render. The first call gets the first state slot, the second gets the second, and so on. Each state variable is independent and updates trigger re-renders of the component .

**Beginner-Friendly Explanation**

You can give your component multiple memories. One for the count, one for the name, one for the color. They don't interfere with each other.

### Purposes

- To manage multiple independent values.
- To separate unrelated state for clarity.
- To avoid a single complex state object.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [isActive, setIsActive] = useState(false);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| First `useState` | State slot 1 |
| Second `useState` | State slot 2 |
| Third `useState` | State slot 3 |

**Syntax Rules**

1. **Order matters**: Hooks must be called in the same order every render .
2. **Independent updates**: Changing one state does not affect others.

### Annotated Complete Code Examples

**Example 1: Independent State Variables**

```jsx
function Profile() {
  const [name, setName] = useState('John');
  const [age, setAge] = useState(30);
  const [isActive, setIsActive] = useState(true);

  return (
    <div>
      <p>{name}, {age}</p>
      <button onClick={() => setAge(age + 1)}>Birthday</button>
      <button onClick={() => setIsActive(!isActive)}>
        {isActive ? 'Deactivate' : 'Activate'}
      </button>
    </div>
  );
}
```

**Explanation**

Three independent state variables. Clicking "Birthday" only updates `age`; clicking "Deactivate" only updates `isActive`. They do not interfere .

### Real-World Cases

**Case: Search Component**

A search component uses separate state for `query`, `results`, `isLoading`, and `error`, keeping each concern isolated.

### References

- LabEx — Can I use multiple states? - https://labex.io/questions/can-i-use-multiple-states-270832
- Epic React — All useState Hooks Are Overwriting the Same State - https://www.epicreact.dev/tutorials/build-react-hooks/all-usestate-hooks-are-overwriting-the-same-state

---

## 7. State Initialization Patterns

### Definitions

**Core Definition**

State initialization patterns are techniques for providing the initial value of state, ranging from direct values to lazy functions and component key resets.

**Technical Definition**

React supports several patterns: passing a direct value, passing a lazy initializer function, and resetting state by changing a component's `key` prop. Each serves different use cases and performance characteristics .

**Beginner-Friendly Explanation**

There are different ways to tell a component what to remember first. For simple things, just give the value. For expensive things, give a function. For resetting everything, change the key.

### Purposes

- To initialize state with static values.
- To avoid expensive computations on every render.
- To reset component state cleanly.

### Syntax Rules and Structure

**Complete General Syntax (Direct)**

```jsx
const [count, setCount] = useState(0);
```

**Complete General Syntax (Lazy)**

```jsx
const [data, setData] = useState(() => expensiveComputation());
```

**Complete General Syntax (Key Reset)**

```jsx
<Form key={version} />
```

**Syntax Rules**

1. **Direct for cheap values**: Numbers, strings, booleans, simple objects .
2. **Lazy for expensive values**: `localStorage` reads, large arrays, transformations .
3. **Key for reset**: Changing `key` remounts the component with fresh state .

**Constraints and Limitations**

- Lazy initializers should be pure.
- Key resets destroy all state in the component and its children.

### Annotated Complete Code Examples

**Example 1: Lazy Initialization from localStorage**

```jsx
function SettingsForm() {
  // JSON.parse runs only on the first render
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings');
    return stored ? JSON.parse(stored) : {};
  });

  return <form>{/* settings fields */}</form>;
}
```

**Explanation**

Without the lazy initializer, `JSON.parse` would run on every render even though the result is only used once. The function form ensures it runs only during initialization .

**Example 2: Resetting State with Key**

```jsx
function App() {
  const [version, setVersion] = useState(0);

  return (
    <div>
      <button onClick={() => setVersion(version + 1)}>Reset</button>
      <Form key={version} />
    </div>
  );
}
```

**Explanation**

When `version` changes, the `key` on `<Form />` changes. React unmounts the old `Form` and mounts a new one with fresh state, effectively resetting the form .

### Real-World Cases

**Case: Multi-Step Form Reset**

After successful submission, incrementing a `version` state and passing it as a `key` to the form resets all fields.

### References

- React — Avoiding recreating the initial state - https://uk.react.dev/reference/react/useState
- React — Resetting state with a key - https://18.react.dev/reference/react/useState
- Vercel Labs — Use Lazy State Initialization - https://github.com/vercel-labs/agent-skills/blob/react-best-practices/skills/react-best-practices//references/rules/rerender-lazy-state-init.md

---

## References

- React — useState API Reference - https://uk.react.dev/reference/react/useState
- React — useState API Reference (React 18) - https://18.react.dev/reference/react/useState
- React — useState API Reference (Chinese) - https://zh-hans.react.dev/reference/react/useState
- React — Updating Objects in State - https://ar.react.dev/learn/updating-objects-in-state
- React — Updating Arrays in State - https://zh-hans.react.dev/learn/updating-arrays-in-state
- React — Immutability Lint Rule - https://de.react.dev/reference/eslint-plugin-react-hooks/lints/immutability
- React — Built-in React Hooks - https://18.react.dev/reference/react/hooks
- React — Hooks API Reference (useReducer) - https://ru.react.js.org/docs/hooks-reference.html
- Vercel Labs — Use Lazy State Initialization - https://github.com/vercel-labs/agent-skills/blob/react-best-practices/skills/react-best-practices//references/rules/rerender-lazy-state-init.md
- LabEx — Can I use multiple states? - https://labex.io/questions/can-i-use-multiple-states-270832
- Epic React — All useState Hooks Are Overwriting the Same State - https://www.epicreact.dev/tutorials/build-react-hooks/all-usestate-hooks-are-overwriting-the-same-state
- GitHub — useState initial value and lazy initialization - https://raw.githubusercontent.com/rohan-paul/Awesome-JavaScript-Interviews/33d091e5666c4ef41574d75e69d5b0d398a25d0e/React/Hooks/useState-replace-componentWillReceiveProps-getDerivedStateFromProps.md