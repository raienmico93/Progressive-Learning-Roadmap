# External State Management: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** External state management refers to the practice of moving shared application state outside of the React component tree and into a dedicated store or state container, allowing any component to read from and write to that state without relying on prop drilling or Context providers.

**Technical Definition:** External state management libraries decouple state from the component hierarchy by maintaining a store that lives outside React. Components subscribe to slices of this store via hooks (e.g., `useSelector` in Redux, `useStore` in Zustand, `useAtom` in Jotai) and re-render only when their selected slice changes. This eliminates the prop-drilling and re-render issues associated with lifted state and Context. The dominant paradigms include: **Flux/Centralised** (Redux — a single global store with dispatched actions and pure reducers), **Hook-based/Minimal** (Zustand — a simple external store consumed via hooks with selector-based subscriptions), **Atomic/Bottom-up** (Jotai — independent state atoms composed together for fine-grained reactivity), **Reactive/Object-Oriented** (MobX — observable proxies that automatically track mutations and derivations), and **State Machine/Actor** (XState — explicit states and transitions for complex workflows). Each paradigm solves the problem of shared state from a different philosophical angle, trading off boilerplate, bundle size, learning curve, and debugging capability.

**Beginner-Friendly Explanation:** Imagine a public bulletin board in a town square. Anyone can pin a message to it, and anyone can read from it. Components don't need to go through their parent to communicate—they just read from and write to the bulletin board. That's an external state store: a shared place for data that lives outside the family (component tree). Redux is like a formal government office with strict procedures; Zustand is like a casual community whiteboard; Jotai is like a set of interconnected sticky notes; MobX is like a magic whiteboard that automatically updates when you erase something; and XState is like a flowchart that only allows certain actions at certain times.

### Key Characteristics

- **Tree Decoupling:** State lives outside React, so any component—sibling, deeply nested, or in a different branch—can subscribe to it directly.
- **Selector-Based Subscriptions:** Components subscribe only to the slices of state they need, preventing unnecessary re-renders when unrelated state changes.
- **Explicit Update Mechanisms:** State changes flow through defined channels (dispatched actions, setter functions, mutations), making updates traceable and debuggable.
- **Paradigm Diversity:** Different libraries model state differently—Redux uses actions and reducers, Zustand uses a single store with setter functions, Jotai uses atoms, MobX uses observable proxies, and XState uses state machines.
- **Trade-Off Spectrum:** Bundle size, boilerplate, learning curve, and debugging capability vary significantly across libraries. Redux is the heaviest but most structured; Zustand is the lightest but least opinionated; MobX is the most "magical" but hardest to debug; XState is the most rigorous but has the steepest learning curve.

### Prerequisites

- Solid understanding of React components, props, and the `useState` Hook.
- Familiarity with the component tree hierarchy and prop drilling.
- Working knowledge of `useContext` and the Context API.
- Basic understanding of JavaScript closures, functions as first-class values, and object references.
- Awareness of the trade-offs between different state management paradigms.

### Related Programming Areas

- **State Architecture:** Deciding where state should live and which tool to use.
- **Performance Optimisation:** Avoiding unnecessary re-renders through selective subscriptions.
- **Reactive Programming:** Managing data flow and change propagation.
- **Event-Driven Architecture:** Actions, events, and state transitions.
- **Debugging and Observability:** DevTools, time-travel debugging, and state inspection.

### Core Concepts / Features

1. Redux (Centralised Flux Architecture)
2. Redux Toolkit (Streamlined Redux)
3. Zustand (Lightweight Hooks-Based Store)
4. Jotai (Atomic State Management)
5. MobX (Reactive Observable State)
6. Other State-Management Approaches (XState, Signals)
7. Choosing an Appropriate Solution

---

## Core Concept 1: Redux (Centralised Flux Architecture)

### Definitions

**Core Definition:** Redux is a predictable state container for JavaScript applications that treats the entire application state as a single immutable object tree, updated exclusively through dispatched actions and pure reducer functions.

**Technical Definition:** Redux is built on three fundamental principles. First, **single source of truth**: the global state of the application is stored in an object tree within a single store, making it easy to create universal apps, debug, and implement features like undo/redo. Second, **state is read-only**: the only way to change the state is to emit an action, an object describing what happened. This ensures that neither views nor network callbacks ever write directly to the state, eliminating subtle race conditions and making actions loggable, serializable, and replayable. Third, **changes are made with pure functions**: to specify how the state tree is transformed by actions, you write pure reducers that take the previous state and an action, and return the next state. Reducers must return new state objects instead of mutating the previous state, which enables Redux to detect changes efficiently. The store is created with `createStore(reducer)`, and state is read with `store.getState()` and updated with `store.dispatch(action)`.

**Beginner-Friendly Explanation:** Redux is like a strict government office. There is one official record book (the store) that holds all the important information. You cannot just walk in and change the record yourself. Instead, you fill out a form (an action) that says what you want to change, and you hand it to the clerk (the reducer). The clerk checks the form, updates the record book accordingly, and hands you back a new copy of the record. This strict process makes it easy to track who changed what and when—but it also involves a lot of paperwork.

### Purposes

- To centralise application state in a single, predictable store.
- To enforce unidirectional data flow with explicit actions and pure reducers.
- To enable time-travel debugging by recording every action and state change.
- To support undo/redo functionality trivially because all state is in a single tree.
- To provide a strict, structured architecture for large applications with many contributors.

### Syntax Rules and Structure

**General Syntax (Classic Redux):**
```javascript
import { createStore } from 'redux';

// Reducer: a pure function (previousState, action) => newState
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { value: state.value + 1 };
    case 'DECREMENT':
      return { value: state.value - 1 };
    default:
      return state;
  }
}

// Create the store
const store = createStore(counterReducer);

// Dispatch an action
store.dispatch({ type: 'INCREMENT' });

// Read the state
console.log(store.getState()); // { value: 1 }
```

**Component Breakdown:**
- `counterReducer`: A pure function that takes the previous state and an action, and returns the next state.
- `createStore(counterReducer)`: Creates the Redux store with the reducer.
- `store.dispatch({ type: 'INCREMENT' })`: Dispatches an action to update the state.
- `store.getState()`: Returns the current state.

**Syntax Rules:**
- Reducers must be pure functions: no side effects, no API calls, no mutation of arguments.
- Reducers must return new state objects, not mutate the previous state.
- Actions must be plain objects with a `type` field (and optional payload).
- State is read-only: never modify `state` directly in a reducer.
- For larger applications, split the root reducer into smaller reducers with `combineReducers`.

**Constraints and Limitations:**
- Classic Redux requires significant boilerplate for actions, action types, and reducers.
- Redux is not inherently optimised for React; it requires `react-redux` for integration.
- The single global store can become a dumping ground if not carefully structured.
- Bundle size is significant (~6,200 kB for Redux alone, ~19 kB for the full setup with React-Redux and Redux Toolkit).

### Annotated Code Example: Counter with Classic Redux

```javascript
import { createStore } from 'redux';

// Step 1: Define the reducer
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { value: state.value + 1 };
    case 'DECREMENT':
      return { value: state.value - 1 };
    default:
      return state;
  }
}

// Step 2: Create the store
const store = createStore(counterReducer);

// Step 3: Subscribe to state changes
store.subscribe(() => {
  console.log('Current state:', store.getState());
});

// Step 4: Dispatch actions
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'DECREMENT' });
```

**Expected Output:**
```
Current state: { value: 1 }
Current state: { value: 2 }
Current state: { value: 1 }
```

**Why This Output Occurs:** Each `store.dispatch` call sends an action to the reducer, which returns a new state object. The `store.subscribe` callback fires after each dispatch, logging the updated state. The reducer does not mutate the previous state—it returns a new object with the updated value. This immutability is what enables Redux's change detection and debugging capabilities.

### Real-World Cases

- **Large enterprise applications:** Redux's strict architecture and time-travel debugging are valuable for applications with many contributors and complex state transitions.
- **Undo/redo features:** Redux's action-based state changes make it trivial to implement undo/redo by replaying or reversing actions.
- **Collaborative applications:** The single state tree makes it easy to synchronize state across clients.
- **Financial dashboards:** The predictable state transitions and audit trail of actions are valuable for applications requiring strict data integrity.

### References

- Redux Official Documentation – Three Principles: https://redux.js.org/understanding/thinking-in-redux/three-principles
- Redux Official Documentation – Redux Essentials Tutorial: https://redux.js.org/tutorials/essentials/part-1-overview-concepts
- Redux Official Documentation – Glossary: https://redux.js.org/understanding/thinking-in-redux/glossary
- GitHub – Redux Three Principles: https://github.com/reduxjs/redux/blob/master/docs/understanding/thinking-in-redux/ThreePrinciples.md

---

## Core Concept 2: Redux Toolkit (Streamlined Redux)

### Definitions

**Core Definition:** Redux Toolkit (RTK) is the official, opinionated, batteries-included toolset for efficient Redux development, providing baked-in tools like Immer and configuration defaults to eliminate standard boilerplate code.

**Technical Definition:** Redux Toolkit is the standard way to write Redux logic today. It was created to address the three most common complaints about Redux: "configuring a store is too complicated," "I have to add a lot of packages to get Redux to do anything useful," and "Redux requires too much boilerplate code". RTK provides `configureStore()` which sets up a store with good defaults—automatically combining slice reducers, including thunk middleware, adding development-mode checks for accidental mutations and non-serializable values, and enabling the Redux DevTools Extension. It provides `createSlice()` which accepts a slice name, an initial state, and an object of reducer functions, and generates the slice reducer plus matching action creators and action types. Reducers can be written with "mutating" syntax thanks to the Immer library, which tracks attempts to mutate a drafted state value and produces a new immutable state behind the scenes. RTK also includes `createAsyncThunk()` for async logic, `createEntityAdapter()` for normalized data, `createListenerMiddleware()` for side effects, and `RTK Query` for data fetching and caching.

**Beginner-Friendly Explanation:** Redux Toolkit is like hiring a professional organiser for your government office. It sets up the filing system for you, provides pre-printed forms, and even fills in some of the fields automatically. You still follow the same rules—actions, reducers, single store—but the tedious paperwork is done for you. If classic Redux is filling out forms by hand, Redux Toolkit is using a smart form that fills itself in as you type.

### Purposes

- To eliminate the boilerplate of classic Redux by providing `configureStore` and `createSlice`.
- To enable "mutating" reducer syntax with Immer while still producing immutable state updates.
- To provide sensible defaults for middleware (thunk), DevTools, and serializability checks.
- To offer a batteries-included toolkit for async logic (`createAsyncThunk`), normalized data (`createEntityAdapter`), and data fetching (`RTK Query`).
- To serve as the standard, recommended way to write Redux logic in modern applications.

### Syntax Rules and Structure

**General Syntax (`createSlice`):**
```javascript
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },  // Immer: "mutating" syntax
    decrement: (state) => { state.value -= 1; },
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;

const store = configureStore({ reducer: { counter: counterSlice.reducer } });
```

**Component Breakdown:**
- `createSlice({ name, initialState, reducers })`: Creates a slice with auto-generated action creators and action types.
- `state.value += 1`: Immer allows "mutating" syntax; the draft state is tracked and a new immutable state is produced.
- `configureStore({ reducer: { counter: counterSlice.reducer } })`: Creates the store with default middleware and DevTools integration.

**General Syntax (`useSelector` and `useDispatch`):**
```jsx
import { Provider, useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch(increment())}>Count: {count}</button>
  );
}

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

**Component Breakdown:**
- `<Provider store={store}>`: Makes the Redux store available to all components.
- `useSelector((state) => state.counter.value)`: Selects the `value` slice from the store.
- `useDispatch()`: Returns the dispatch function for actions.

**Syntax Rules:**
- Use `createSlice` to define reducers and auto-generate actions; do not write action creators or action types by hand.
- Write reducers with "mutating" syntax inside `createSlice`; Immer handles immutability.
- Use `configureStore` instead of `createStore`; it includes thunk, DevTools, and serializability checks by default.
- Use `useSelector` to read state and `useDispatch` to dispatch actions in React components.
- For async logic, use `createAsyncThunk` or RTK Query.

**Constraints and Limitations:**
- RTK adds a dependency (~13 kB for the toolkit), though it eliminates many smaller packages.
- Immer's "mutating" syntax can be confusing for developers expecting strict immutability.
- RTK Query is opinionated about data fetching; teams with existing fetching solutions may not need it.
- Redux (and RTK) have the steepest learning curve among the major libraries.

### Annotated Code Example: Counter with Redux Toolkit

```javascript
import { createSlice, configureStore } from '@reduxjs/toolkit';

// Step 1: Create the slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
    decrement(state) { state.value -= 1; },
    incrementByAmount(state, action) {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// Step 2: Configure the store
const store = configureStore({
  reducer: { counter: counterSlice.reducer },
});

// Step 3: Dispatch actions
store.dispatch(increment());
store.dispatch(incrementByAmount(5));

console.log(store.getState()); // { counter: { value: 6 } }
```

**Expected Output:**
```
{ counter: { value: 6 } }
```

**Why This Output Occurs:** `createSlice` generates action creators (`increment`, `incrementByAmount`) and a reducer. The reducer uses Immer's draft state, so `state.value += 1` and `state.value += action.payload` are written with "mutating" syntax but produce a new immutable state. `configureStore` sets up the store with the slice reducer. Dispatching `increment()` adds 1, and `incrementByAmount(5)` adds 5, resulting in a value of 6.

### Real-World Cases

- **Enterprise dashboards:** RTK's structure and DevTools integration are valuable for large, long-lived applications that need debuggability and predictable state transitions.
- **E-commerce platforms:** RTK Query simplifies data fetching and caching for product listings, cart state, and order management.
- **Collaborative tools:** RTK's single store and action-based updates make it easy to sync state across clients.
- **Financial applications:** RTK's serializability checks and DevTools help maintain data integrity.

### References

- Redux Toolkit – Overview: https://redux.js.org/redux-toolkit/overview
- Redux Toolkit – Quick Start: https://redux-toolkit.js.org/tutorials/quick-start
- Redux Toolkit – createSlice: https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit – configureStore: https://redux-toolkit.js.org/api/configureStore
- Redux Toolkit – Writing Reducers with Immer: https://redux-toolkit.js.org/usage/immer-reducers

---

## Core Concept 3: Zustand (Lightweight Hooks-Based Store)

### Definitions

**Core Definition:** Zustand is an ultra-lightweight, hooks-based global store that decouples shared state from the React tree context, enabling target selectors and fine-grained subscriptions with minimal boilerplate.

**Technical Definition:** Zustand (German for "state") is a small, fast, and scalable state-management solution built on the concept of a store as a hook. A store is created with `create()`, which returns a hook that can be called in any component to subscribe to state. The store is a single object containing both state values and actions (functions that update the state). Components use selector functions to subscribe to specific slices of the store: `const count = useCounterStore((s) => s.count)`. By default, Zustand uses strict equality (`===`) to determine whether the selected value has changed; for objects or arrays, `useShallow` provides shallow comparison to prevent unnecessary re-renders. Zustand requires no providers—the store is a hook that can be used anywhere in the component tree. It works across React, React Native, and vanilla JavaScript with the same API surface. Zustand also provides middleware for persistence, DevTools, Immer integration, and Redux-style middleware without changing the mental model.

**Beginner-Friendly Explanation:** Zustand is like a community whiteboard that anyone can walk up to and read or write on. You don't need to install a complex filing system (like Redux) or ask a parent to pass messages (like Context). You just create a whiteboard (the store) with a single function, and any component can look at the part of the whiteboard it cares about. If you only care about the "count" section, you don't get notified when someone writes in the "name" section. That's the power of selectors.

### Purposes

- To provide a minimal, hooks-based global store with almost no boilerplate.
- To enable fine-grained subscriptions through selectors, so components only re-render when their selected slice changes.
- To eliminate the need for providers, reducing the component tree complexity.
- To work seamlessly across React, React Native, and vanilla JavaScript.
- To offer opt-in middleware for persistence, DevTools, and Immer without changing the core mental model.

### Syntax Rules and Structure

**General Syntax (Creating a Store):**
```javascript
import { create } from 'zustand';

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));
```

**Component Breakdown:**
- `create((set) => ({ ... }))`: Creates the store with an initial state and actions.
- `set((state) => ({ count: state.count + 1 }))`: Updates the state immutably using a callback.
- The store is a hook (`useCounterStore`) that can be called in any component.

**General Syntax (Using the Store with Selectors):**
```jsx
function Counter() {
  const count = useCounterStore((state) => state.count);
  const increment = useCounterStore((state) => state.increment);

  return <button onClick={increment}>Count: {count}</button>;
}

function CountDisplay() {
  const count = useCounterStore((state) => state.count);
  return <p>Count: {count}</p>;
}
```

**Component Breakdown:**
- `useCounterStore((state) => state.count)`: Selects only the `count` slice; the component re-renders only when `count` changes.
- `useCounterStore((state) => state.increment)`: Selects the `increment` action.
- No provider is needed; the store is usable anywhere.

**General Syntax (useShallow for Object Selections):**
```jsx
import { useShallow } from 'zustand/react/shallow';

function UserProfile() {
  const { name, email } = useCounterStore(
    useShallow((state) => ({ name: state.name, email: state.email }))
  );
  // ...
}
```

**Component Breakdown:**
- `useShallow(selector)`: Wraps the selector so that the component only re-renders if the shallow comparison of the selected object changes.
- Without `useShallow`, a new object is created on every render, causing re-renders even when `name` and `email` are unchanged.

**Syntax Rules:**
- Use `create()` to define the store with state and actions.
- Use selectors to subscribe to specific slices: `useStore((s) => s.slice)`.
- For objects or arrays, use `useShallow` to prevent unnecessary re-renders.
- Use the two-argument `subscribe(selector, callback)` for imperative subscriptions outside React.
- Apply middleware (`persist`, `devtools`, `immer`) by wrapping the store creator.

**Constraints and Limitations:**
- Zustand provides less guidance for project structure than Redux; large applications may need conventions.
- Selector functions are re-created on every render unless memoised or defined outside the component.
- Over-globalising state (putting everything in one store) can lead to the same issues as a mega context.
- The two-argument `subscribe(selector, callback)` requires the `subscribeWithSelector` middleware.

### Annotated Code Example: Todo Store with Zustand

```jsx
import { create } from 'zustand';

// Create the store
const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text, done: false }],
  })),
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    ),
  })),
}));

// Component: Add todo
function AddTodo() {
  const addTodo = useTodoStore((state) => state.addTodo);
  const [text, setText] = React.useState('');

  function handleSubmit(e) {
    e.preventDefault();
    if (text.trim()) { addTodo(text.trim()); setText(''); }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

// Component: Todo list
function TodoList() {
  const todos = useTodoStore((state) => state.todos);
  const toggleTodo = useTodoStore((state) => state.toggleTodo);

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id} onClick={() => toggleTodo(todo.id)}
            style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

export default function App() {
  return (
    <div>
      <h1>Todo List</h1>
      <AddTodo />
      <TodoList />
    </div>
  );
}
```

**Expected Output:** A form with an input and "Add" button, and an empty todo list. Typing a task and clicking "Add" adds it to the list. Clicking a task toggles its done state (strikethrough).

**Why This Output Occurs:** The Zustand store holds `todos` state and actions (`addTodo`, `toggleTodo`). `AddTodo` selects only the `addTodo` action, so it does not re-render when `todos` changes. `TodoList` selects `todos` and `toggleTodo`, so it re-renders when todos change. When `addTodo` updates the store, `TodoList` re-renders with the new todo. No providers or prop drilling are needed.

### Real-World Cases

- **UI-heavy applications:** Zustand's minimal API and selector-based subscriptions are ideal for applications with many interactive components.
- **React Native apps:** Zustand works seamlessly across React and React Native with the same API.
- **Feature-scoped stores:** Zustand's small footprint makes it suitable for per-feature stores that are composed at the app level.
- **Prototyping:** Zustand's low boilerplate makes it ideal for rapid prototyping and small-to-medium applications.

### References

- Zustand Documentation: https://zustand.docs.pmnd.rs/
- Zustand – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- Zustand – useShallow: https://zustand.docs.pmnd.rs/hooks/use-shallow
- Zustand – Persisting Store Data: https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- GitHub – Zustand: https://github.com/pmndrs/zustand

---

## Core Concept 4: Jotai (Atomic State Management)

### Definitions

**Core Definition:** Jotai is an atomic-based state management library where the global application state is broken down into small, composable nodes of state (atoms) that trigger granular updates only in the components that read them.

**Technical Definition:** Jotai takes a bottom-up approach to React state management. State is represented as atoms—independent, composable units of state. An atom is created with `atom(initialValue)` and consumed with `useAtom(atom)`, which returns a tuple of the current value and a setter. Atoms can be derived from other atoms using the `atom((get) => ...)` pattern, creating a dependency graph where derived atoms automatically recompute when their dependencies change. The key advantage of the atomic model is granular updates: "A large monolithic store triggers re-renders in every subscriber when any single field changes. Atomic state splits each independent value into its own reactive unit, so components subscribe to exactly the atoms they read". Jotai provides utilities like `selectAtom` (a read-only derived atom with custom equality), `focusAtom` (to create an atom from a specific part of a large object), and `splitAtom` (to create an atom for each element in a list). Jotai's atoms are simpler than Redux slices and more granular than Zustand stores, making them ideal for UI state like filter values, modal open/close, or selected items in a list.

**Beginner-Friendly Explanation:** Jotai is like a set of sticky notes on a wall. Each note holds one piece of information—the current filter, the user's name, whether a modal is open. If you change the "filter" sticky note, only the components that read that note update. Other components that read different notes don't care and don't re-render. You can also create a "summary" sticky note that automatically updates when the notes it depends on change. This is the atomic model: small, independent pieces of state that only affect the components that read them.

### Purposes

- To provide fine-grained reactivity where only components that read a changed atom re-render.
- To enable a bottom-up, composable state model where atoms can derive from other atoms.
- To avoid the monolithic store problem where changing one field re-renders all subscribers.
- To simplify state management for UI state like filters, modals, and form inputs.
- To provide utilities for focusing on parts of large objects and splitting lists into atoms.

### Syntax Rules and Structure

**General Syntax (Primitive Atom):**
```javascript
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

**Component Breakdown:**
- `atom(0)`: Creates a primitive atom with an initial value of 0.
- `useAtom(countAtom)`: Returns `[value, setter]` and subscribes the component to the atom.
- Only components that read `countAtom` re-render when it changes.

**General Syntax (Derived Atom):**
```javascript
const countAtom = atom(0);
const doubledAtom = atom((get) => get(countAtom) * 2);

function DoubledDisplay() {
  const [doubled] = useAtom(doubledAtom);
  return <p>Doubled: {doubled}</p>;
}
```

**Component Breakdown:**
- `atom((get) => get(countAtom) * 2)`: Creates a derived atom that reads `countAtom` and returns its doubled value.
- `useAtom(doubledAtom)`: The component re-renders only when `doubledAtom` changes (i.e., when `countAtom` changes).

**General Syntax (selectAtom — Read-Only Derived):**
```javascript
import { selectAtom } from 'jotai/utils';

const userAtom = atom({ name: 'John', age: 30, email: 'john@example.com' });
const nameAtom = selectAtom(userAtom, (user) => user.name);
```

**Component Breakdown:**
- `selectAtom(userAtom, (user) => user.name)`: Creates a read-only atom that selects only the `name` property.
- Components reading `nameAtom` re-render only when `name` changes, not when `age` or `email` changes.

**Syntax Rules:**
- Use `atom(initialValue)` for primitive state; use `atom((get) => ...)` for derived state.
- Use `useAtom(atom)` to read and write; use `useAtomValue(atom)` for read-only and `useSetAtom(atom)` for write-only.
- Prefer derived atoms over `selectAtom`; the official docs call `selectAtom` an "escape hatch".
- Use `atomFamily` for parameterised atoms (e.g., one atom per item ID).
- Atoms can be created outside components; if created inside, use `useRef` or `useMemo` for stability.

**Constraints and Limitations:**
- Jotai's atomic model can be overkill for simple state that a single `useState` could handle.
- The number of atoms can grow large in complex applications, requiring organisation.
- `selectAtom` and `focusAtom` add complexity; prefer simple derived atoms where possible.
- Jotai does not provide the strict structure or DevTools of Redux.

### Annotated Code Example: Filter State with Jotai

```jsx
import { atom, useAtom, useAtomValue } from 'jotai';

// Primitive atoms
const filterAtom = atom('all');
const todosAtom = atom([
  { id: 1, text: 'Learn Jotai', done: false },
  { id: 2, text: 'Build app', done: true },
]);

// Derived atom: filtered todos
const filteredTodosAtom = atom((get) => {
  const filter = get(filterAtom);
  const todos = get(todosAtom);
  if (filter === 'done') return todos.filter(t => t.done);
  if (filter === 'active') return todos.filter(t => !t.done);
  return todos;
});

// Component: Filter controls
function FilterControls() {
  const [filter, setFilter] = useAtom(filterAtom);
  return (
    <div>
      {['all', 'active', 'done'].map(f => (
        <button key={f} onClick={() => setFilter(f)}
                style={{ fontWeight: filter === f ? 'bold' : 'normal' }}>
          {f}
        </button>
      ))}
    </div>
  );
}

// Component: Todo list (re-renders only when filtered list changes)
function TodoList() {
  const todos = useAtomValue(filteredTodosAtom);
  return (
    <ul>
      {todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
    </ul>
  );
}

export default function App() {
  return (
    <div>
      <h1>Jotai Todo</h1>
      <FilterControls />
      <TodoList />
    </div>
  );
}
```

**Expected Output:** Three filter buttons ("all", "active", "done") and a todo list. Clicking "active" shows only incomplete todos. Clicking "done" shows only completed todos. The `FilterControls` component re-renders only when the filter changes; `TodoList` re-renders only when the filtered list changes.

**Why This Output Occurs:** The `filterAtom` holds the current filter, and `todosAtom` holds the todos. `filteredTodosAtom` is a derived atom that reads both and returns the filtered list. `FilterControls` uses `useAtom(filterAtom)` and re-renders when the filter changes. `TodoList` uses `useAtomValue(filteredTodosAtom)` and re-renders only when the filtered result changes. Because the atoms are independent, changing the filter does not re-render components that only read unrelated atoms.

### Real-World Cases

- **Filterable lists:** `filterAtom`, `sortAtom`, and `searchAtom` as independent atoms; a derived atom combines them into the visible list.
- **Modals and dialogs:** `modalOpenAtom` for each modal; only the modal component re-renders when its atom changes.
- **Form state:** Atoms for each form field; derived atoms for validation state.
- **Selected items:** `selectedIdAtom` for a list; `selectedItemAtom` derived from `selectedIdAtom` and the items atom.
- **Multi-step wizards:** `stepAtom` and `formDataAtom` as independent atoms; a derived atom for the current step's data.

### References

- Jotai Official Documentation: https://jotai.org/
- Jotai – Large Objects (focusAtom, splitAtom): https://jotai.org/docs/recipes/large-objects
- Jotai – selectAtom: https://jotai.org/docs/utilities/select
- Jotai – Derived Atoms: https://jotai.org/docs/core/atom
- CoreUI – How to use Jotai in React: https://coreui.io/blog/how-to-use-jotai-in-react/

---

## Core Concept 5: MobX (Reactive Observable State)

### Definitions

**Core Definition:** MobX is a reactive, object-oriented state engine that uses transparently wrapped observable proxies to automatically track mutations and derivations, ensuring that any state that can be derived from the application state will be derived automatically.

**Technical Definition:** MobX is built on four core concepts: **observable state** (state that is tracked and can be observed), **computed values** (values that are derived from the state using pure functions and are cached automatically), **reactions** (side effects that happen automatically when the state changes), and **actions** (methods that modify the state). MobX wraps objects, arrays, and maps with JavaScript `Proxy` objects to track and notify on property changes without changing the vanilla JavaScript types. When an observable property is read during the execution of a tracked function (such as a computed value or a reaction), MobX automatically establishes a dependency, so any change to that property triggers a re-evaluation of the function. `computed` marks a getter that derives new facts from the state and caches its output; `action` marks a method that modifies the state; `reaction` (including `autorun`, `when`, and `reaction`) defines side effects that run automatically when the state changes. MobX reactions to state changes are fine-grained: only the computations and reactions that depend on the changed observables are re-evaluated.

**Beginner-Friendly Explanation:** MobX is like a magic whiteboard. When you write on it (change an observable), the whiteboard automatically updates any notes (computed values) that were calculated from what you wrote. Any notes (reactions) that were watching the whiteboard also update automatically. You don't have to manually tell the whiteboard, "Hey, I changed something—update the derived notes." It just happens. This is called "transparent reactive programming": the reactivity is invisible to the developer, who simply writes normal JavaScript.

### Purposes

- To provide automatic, fine-grained reactivity where derived values and reactions update only when their observable dependencies change.
- To eliminate the need for manual subscriptions and dependency tracking.
- To allow developers to write normal JavaScript objects and classes while gaining reactivity.
- To cache computed values automatically, recomputing only when dependencies change.
- To handle complex, object-oriented domain models with reactive state.

### Syntax Rules and Structure

**General Syntax (Observable State with makeObservable):**
```javascript
import { makeObservable, observable, action, computed } from 'mobx';

class TodoStore {
  todos = [];
  filter = 'all';

  constructor() {
    makeObservable(this, {
      todos: observable,
      filter: observable,
      addTodo: action,
      toggleTodo: action,
      filteredTodos: computed,
    });
  }

  addTodo(text) {
    this.todos.push({ id: Date.now(), text, done: false });
  }

  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) todo.done = !todo.done;
  }

  get filteredTodos() {
    if (this.filter === 'done') return this.todos.filter(t => t.done);
    if (this.filter === 'active') return this.todos.filter(t => !t.done);
    return this.todos;
  }
}
```

**Component Breakdown:**
- `observable`: Marks state fields as observable (tracked for changes).
- `action`: Marks methods that modify the state.
- `computed`: Marks getters that derive values from the state.
- `makeObservable(this, { ... })`: Connects the decorators to the class instance.

**General Syntax (React Integration with observer):**
```jsx
import { observer } from 'mobx-react-lite';

const TodoList = observer(({ store }) => {
  return (
    <ul>
      {store.filteredTodos.map(todo => (
        <li key={todo.id} onClick={() => store.toggleTodo(todo.id)}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
});
```

**Component Breakdown:**
- `observer(Component)`: Wraps the component so it re-renders when any observable it reads changes.
- `store.filteredTodos`: The component automatically subscribes to the `filteredTodos` computed value and its dependencies.

**Syntax Rules:**
- Use `makeObservable` (or `makeAutoObservable`) to mark fields as `observable`, methods as `action`, and getters as `computed`.
- Use `observer` from `mobx-react-lite` to wrap React components that read observable state.
- Mutate state only inside `action` methods; do not mutate observables directly from components.
- Use `computed` for derived values; they are cached and recomputed only when dependencies change.
- Use `reaction`, `autorun`, or `when` for side effects that should run when state changes.

**Constraints and Limitations:**
- MobX requires `Proxy` support (modern browsers); MobX 7 dropped the older ES5 fallback.
- MobX's "magic" can make it harder to debug state changes compared to Redux's explicit actions.
- MobX is less commonly used than Redux or Zustand in new React projects.
- Bundle size is significant (~4,718 kB for MobX alone).
- Class-based stores and decorators may feel outdated to developers using modern React function components.

### Annotated Code Example: Observable Counter with MobX

```jsx
import { makeAutoObservable } from 'mobx';
import { observer } from 'mobx-react-lite';

// Store: a plain class with makeAutoObservable
class CounterStore {
  count = 0;

  constructor() {
    // makeAutoObservable automatically marks fields as observable,
    // methods as actions, and getters as computed
    makeAutoObservable(this);
  }

  increment() { this.count += 1; }
  decrement() { this.count -= 1; }

  get doubled() { return this.count * 2; }
}

const store = new CounterStore();

// Component: observes the store
const Counter = observer(() => {
  return (
    <div>
      <p>Count: {store.count}</p>
      <p>Doubled: {store.doubled}</p>
      <button onClick={() => store.increment()}>+</button>
      <button onClick={() => store.decrement()}>-</button>
    </div>
  );
});

export default function App() {
  return <Counter />;
}
```

**Expected Output:** A counter displaying "Count: 0" and "Doubled: 0" with "+" and "-" buttons. Clicking "+" increments both the count and the doubled value. The `doubled` value is automatically recomputed when `count` changes.

**Why This Output Occurs:** `makeAutoObservable` marks `count` as observable, `increment` and `decrement` as actions, and `doubled` as computed. The `observer` wrapper makes the `Counter` component subscribe to `store.count` and `store.doubled`. When the user clicks "+", the `increment` action updates `count`, which triggers MobX to recompute `doubled` and re-render the component. The component does not need to manually subscribe or specify dependencies—MobX tracks them automatically.

### Real-World Cases

- **Complex domain models:** Applications with rich object-oriented domain models (e.g., financial systems, simulations).
- **Real-time dashboards:** MobX's fine-grained reactivity is efficient for dashboards with many independent data sources.
- **Form-heavy applications:** MobX's observable state and computed validation make form management concise.
- **Legacy React applications:** MobX was popular in the class-component era and remains in many existing codebases.

### References

- MobX Official Documentation: https://mobx.js.org/
- MobX – Creating Observable State: https://mobx.js.org/observable-state.html
- MobX – Understanding Reactivity: https://mobx.js.org/understanding-reactivity.html
- MobX – API Reference: https://mobx.js.org/api-reference.html
- mobx-react-lite – GitHub: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite

---

## Core Concept 6: Other State-Management Approaches (XState, Signals)

### Definitions

**Core Definition:** Other state-management approaches include alternative execution paradigms such as finite state machines (XState) and specialised signal abstractions, which model state and transitions differently from the store-and-reducer or atom paradigms.

**Technical Definition:** **XState** is a state management and orchestration solution for JavaScript and TypeScript apps, built around the idea of finite state machines (FSM) and statecharts. Instead of letting state change freely, XState allows you to define a set of possible states and describe how the app can move from one well-defined state to another. These transitions are explicit and triggered by events. When things get complicated, XState adds statecharts to organise complex state logic into a visual structure, and actors to allow multiple machines to run in parallel and communicate. **Signals** are a reactive primitive that simplifies state management by allowing values to be tracked and updated automatically. Preact Signals provides a performant state-management library with first-class React integration, including a Babel transform that automatically makes components using signals reactive. Signals are lightweight (starting at ~1.8KB) and are inspired by MobX and Solid. Other signal implementations include `@cerberus-design/signals` (an O(1) signal-based state management engine) and `@deepsignal/react` (a deep signal model for React).

**Beginner-Friendly Explanation:** XState is like a flowchart for your application. Instead of a single "state" variable that can be anything, you define the exact states your app can be in—"idle," "loading," "success," "error"—and the exact events that cause transitions between them. This makes it impossible to end up in an invalid state, like being both "loading" and "success" at the same time. Signals are like reactive variables that automatically notify any component using them when they change. They are smaller and simpler than most state-management libraries.

### Purposes

- **XState:** To model complex, stateful workflows with explicit states and transitions, preventing invalid states.
- **XState:** To visualise and test state logic with statecharts and the Stately Studio visual editor.
- **Signals:** To provide a lightweight, performant reactive primitive for state management.
- **Signals:** To enable fine-grained updates without the overhead of a full state-management library.
- **Both:** To offer alternatives to the store-and-reducer paradigm for teams whose domain models align better with these approaches.

### Syntax Rules and Structure

**XState General Syntax:**
```javascript
import { createMachine } from 'xstate';
import { useMachine } from '@xstate/react';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
});

function ToggleButton() {
  const [state, send] = useMachine(toggleMachine);
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.value === 'inactive' ? 'Off' : 'On'}
    </button>
  );
}
```

**Component Breakdown:**
- `createMachine({ ... })`: Defines the state machine with states and transitions.
- `useMachine(toggleMachine)`: Creates and starts the actor, returning `[state, send]`.
- `send({ type: 'TOGGLE' })`: Sends an event to the machine, triggering a transition.

**Signals General Syntax (Preact Signals with React):**
```jsx
import { signal } from '@preact/signals-react';
import { useSignals } from '@preact/signals-react/runtime';

const count = signal(0);

function Counter() {
  useSignals();
  return (
    <button onClick={() => count.value++}>
      Count: {count.value}
    </button>
  );
}
```

**Component Breakdown:**
- `signal(0)`: Creates a signal with an initial value of 0.
- `useSignals()`: Enables signal tracking in the component (required for the runtime API).
- `count.value`: Reading the signal subscribes the component to its changes; writing updates it.

**Syntax Rules (XState):**
- Define machines with `createMachine`, specifying `id`, `initial`, and `states`.
- Each state defines `on` handlers for events that trigger transitions.
- Use `useMachine` from `@xstate/react` to run a machine in a React component.
- Use `context` for extended state (data) and `actions` for side effects.
- Use `statecharts` for hierarchical, parallel, or history states.

**Syntax Rules (Signals):**
- Use `signal(initialValue)` to create a signal.
- Call `useSignals()` in components that read signals (for the runtime API).
- Read with `signal.value`; write with `signal.value = newValue`.
- Use `computed(() => ...)` for derived signals.
- Use `effect(() => ...)` for side effects.

**Constraints and Limitations:**
- **XState:** Has a steeper learning curve than store-based libraries; not necessary for simple state.
- **XState:** Adds a dependency and a conceptual model that may not fit all teams.
- **Signals:** React integration requires a Babel transform or `useSignals` hook; not as widely adopted as Zustand or Redux.
- **Signals:** The signals ecosystem is still maturing; fewer resources and community support than Redux.

### Annotated Code Example: Authentication Flow with XState

```jsx
import { createMachine, assign } from 'xstate';
import { useMachine } from '@xstate/react';

// Define the auth machine
const authMachine = createMachine({
  id: 'auth',
  initial: 'idle',
  context: { user: null, error: null },
  states: {
    idle: {
      on: { SUBMIT: 'loading' },
    },
    loading: {
      invoke: {
        src: 'authenticate',
        onDone: {
          target: 'success',
          actions: assign({ user: (_, event) => event.data }),
        },
        onError: {
          target: 'failure',
          actions: assign({ error: (_, event) => event.data }),
        },
      },
    },
    success: {
      type: 'final',
    },
    failure: {
      on: { RETRY: 'loading' },
    },
  },
}, {
  services: {
    authenticate: async (context, event) => {
      // Simulate API call
      if (event.email === 'user@example.com') {
        return { name: 'John', email: 'user@example.com' };
      }
      throw new Error('Invalid credentials');
    },
  },
});

function AuthForm() {
  const [state, send] = useMachine(authMachine);

  if (state.matches('success')) {
    return <p>Welcome, {state.context.user.name}!</p>;
  }

  return (
    <div>
      {state.matches('failure') && (
        <p>Error: {state.context.error.message}</p>
      )}
      <button onClick={() => send({ type: 'SUBMIT', email: 'user@example.com' })}>
        {state.matches('loading') ? 'Loading...' : 'Login'}
      </button>
    </div>
  );
}

export default function App() {
  return <AuthForm />;
}
```

**Expected Output:** A "Login" button. Clicking it shows "Loading..." briefly, then "Welcome, John!". If the email is invalid, an error message appears with a retry option.

**Why This Output Occurs:** The `authMachine` defines explicit states (`idle`, `loading`, `success`, `failure`) and transitions (`SUBMIT`, `RETRY`). The `invoke` config runs the `authenticate` service when entering `loading` and transitions to `success` or `failure` based on the result. The component reads `state.matches(...)` to render the appropriate UI. Invalid states (e.g., showing both "loading" and "success") are impossible because the machine only allows one state at a time.

### Real-World Cases (XState)

- **Authentication flows:** Explicit states for idle, loading, success, failure with retry logic.
- **Multi-step wizards:** Statecharts for step navigation, validation, and submission.
- **Media players:** States for playing, paused, buffering, stopped with precise transitions.
- **Checkout processes:** State machines for cart, shipping, payment, and confirmation steps.
- **Game logic:** Finite state machines for character states, game phases, and turn management.

### Real-World Cases (Signals)

- **High-frequency updates:** Signals are efficient for frequently changing values like mouse position, scroll position, or animation progress.
- **UI primitives:** Signals for toggle states, input values, and other simple reactive values.
- **Derived state:** Computed signals for values derived from other signals.
- **Performance-critical components:** Signals avoid the virtual DOM overhead of React state in some cases.

### References

- XState Official Documentation: https://stately.ai/docs/xstate
- XState – useMachine: https://stately.ai/docs/xstate-react
- XState – Introduction: https://mintlify.wiki/statelyai/xstate
- Preact Signals – GitHub: https://github.com/preactjs/signals
- Preact Signals – Documentation: https://preactjs.com/guide/v10/signals/
- @deepsignal/react – npm: https://www.npmjs.com/package/@deepsignal/react

---

## Core Concept 7: Choosing an Appropriate Solution

### Definitions

**Core Definition:** Choosing an appropriate state-management solution is the process of balancing product architectural needs against factors such as team workflow, bundle overhead, code complexity, data update frequencies, and long-term maintainability.

**Technical Definition:** There is no single "best" state-management solution. By 2026, the right tool depends on the problem, not popularity. The decision framework evaluates each library across several dimensions: **bundle size** (Zustand ~1.2 kB, Jotai ~12 kB, Redux Toolkit ~13 kB, MobX ~4,718 kB), **boilerplate** (Zustand and Jotai are minimal, Redux is high, MobX is low but class-based), **learning curve** (Zustand and Jotai are gentle, Redux and XState are steep), **debugging capability** (Redux has the best DevTools and time-travel debugging, Zustand and Jotai have basic DevTools, MobX has limited debugging), **performance** (Zustand, Jotai, and Signals are lightweight with fine-grained updates; Redux and MobX have larger bundle sizes and higher memory usage), and **team workflow** (Redux's strict structure suits large teams; Zustand's flexibility suits small teams and rapid prototyping). The key insight is that different state categories require different tools: server state goes in TanStack Query, form state in React Hook Form, URL state in `useSearchParams`, and client UI state in local `useState` or Zustand.

**Beginner-Friendly Explanation:** Choosing a state-management library is like choosing a vehicle. If you need to move a few boxes across town, a small car (Zustand) is perfect. If you are running a delivery company with many drivers and complex routes, a fleet of trucks with GPS tracking (Redux) might be necessary. If you need to navigate a maze, a map with explicit paths (XState) is ideal. And if you just need to carry a single box, you can use your hands (useState). There is no single "best" vehicle—only the right one for the job.

### Purposes

- To match the state-management tool to the application's size, complexity, and team workflow.
- To avoid over-engineering (using Redux for a small app) and under-engineering (using `useState` for a large app).
- To consider bundle size and performance implications, especially for performance-critical applications.
- To align the tool's paradigm with the domain model (e.g., state machines for workflows, atoms for independent UI state).
- To make a deliberate, informed decision rather than defaulting to the most popular option.

### Syntax Rules and Structure (Decision Framework)

**Step 1: Classify Your State**

```
├── Server state (API data) → TanStack Query / SWR
├── Form state → React Hook Form / Formik
├── URL state → useSearchParams / React Router
├── Local UI state → useState / useReducer
├── Shared client state → choose a library (Redux, Zustand, Jotai, MobX)
└── Complex workflows → XState
```

**Step 2: Evaluate Shared Client State Needs**

| Factor | Redux Toolkit | Zustand | Jotai | MobX | XState |
|---|---|---|---|---|---|
| **Bundle size** | ~13 kB | ~1.2 kB | ~12 kB | ~4.7 MB | ~15 kB |
| **Boilerplate** | High | Low | Low | Medium | High |
| **Learning curve** | Steep | Gentle | Gentle | Moderate | Steep |
| **Debugging** | Excellent (DevTools) | Basic (DevTools) | Basic (DevTools) | Limited | Good (visualiser) |
| **Performance** | Good (selector) | Excellent (selector) | Excellent (atomic) | Good (proxy) | Good |
| **Best for** | Large enterprise apps | Medium apps, rapid prototyping | UI state, fine-grained | OO domain models | Complex workflows |
| **Team size** | Large teams | Small-to-medium | Small-to-medium | Medium | Medium |

**Step 3: Consider Non-Negotiable Constraints**

- **Bundle size critical?** → Zustand (1.2 kB) or Jotai (12 kB).
- **Time-travel debugging required?** → Redux Toolkit.
- **Complex state machines?** → XState.
- **Object-oriented domain model?** → MobX.
- **Minimal boilerplate?** → Zustand or Jotai.
- **Fine-grained atomic updates?** → Jotai.
- **Large team, strict conventions?** → Redux Toolkit.
- **React Native support?** → Zustand, Redux, Jotai, MobX all work; Zustand is the lightest.

**Syntax Rules (Decision Heuristics):**
- **Start with the simplest tool that solves the problem.** For most shared state, Zustand is sufficient.
- **Use Redux Toolkit if you need structure, DevTools, and a large team can absorb the learning curve.**
- **Use Jotai if your state is naturally atomic (many independent values) and you need fine-grained updates.**
- **Use MobX if your domain model is object-oriented and your team prefers reactive programming.**
- **Use XState if your state has clear, discrete states and transitions (workflows, wizards, media players).**
- **Never use Redux for server state.** Use TanStack Query instead.
- **Never use a global store for form state.** Use React Hook Form.
- **Never use Context for high-frequency updates.** Use Zustand or Jotai.

**Constraints and Limitations:**
- No single library is optimal for all scenarios; most applications benefit from using multiple tools for different state categories.
- Team familiarity matters: adopting an unfamiliar library has a productivity cost.
- Bundle size benchmarks vary by measurement methodology; always verify with your own build.
- Library popularity is not a proxy for suitability; evaluate against your specific requirements.

### Annotated Code Example: Hybrid State Architecture

```jsx
// server-state.js — TanStack Query for API data
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useTodos() {
  return useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then(r => r.json()),
  });
}

export function useAddTodo() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (text) => fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify({ text }),
    }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),
  });
}
```

```jsx
// ui-state.js — Zustand for client UI state
import { create } from 'zustand';

export const useUIStore = create((set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
}));
```

```jsx
// TodoApp.jsx — composing tools
import { useTodos, useAddTodo } from './server-state';
import { useUIStore } from './ui-state';
import { useState } from 'react';

export default function TodoApp() {
  const { data: todos, isPending } = useTodos();
  const addTodo = useAddTodo();
  const sidebarOpen = useUIStore((s) => s.sidebarOpen);
  const toggleSidebar = useUIStore((s) => s.toggleSidebar);
  const [text, setText] = useState('');

  if (isPending) return <p>Loading...</p>;

  return (
    <div>
      <button onClick={toggleSidebar}>
        {sidebarOpen ? 'Close' : 'Open'} Sidebar
      </button>
      <form onSubmit={(e) => {
        e.preventDefault();
        if (text.trim()) { addTodo.mutate(text.trim()); setText(''); }
      }}>
        <input value={text} onChange={(e) => setText(e.target.value)} />
        <button type="submit">Add</button>
      </form>
      <ul>
        {todos?.map(todo => <li key={todo.id}>{todo.text}</li>)}
      </ul>
    </div>
  );
}
```

**Expected Output:** A todo list fetched from the server, an input to add new todos, and a sidebar toggle button. Adding a todo sends a POST request and refetches the list. The sidebar toggle is local UI state managed by Zustand.

**Why This Output Occurs:** Server state (`todos`) is managed by TanStack Query, which handles caching, loading states, and invalidation. Client UI state (`sidebarOpen`) is managed by Zustand, which provides a lightweight store for UI flags. Form state (`text`) is managed by `useState` because it is local and ephemeral. This hybrid architecture uses the right tool for each state category, avoiding the anti-pattern of putting everything in one store.

### Real-World Cases

- **Small apps / prototypes:** Zustand for shared state, `useState` for local state, TanStack Query for server data.
- **Medium apps:** Zustand for client state, React Hook Form for forms, TanStack Query for server state.
- **Large enterprise apps:** Redux Toolkit for complex client state, RTK Query for server state, React Hook Form for forms.
- **UI-heavy apps with fine-grained state:** Jotai for atomic UI state, TanStack Query for server data.
- **Complex workflows:** XState for state machines, Zustand for UI state.
- **Object-oriented domain models:** MobX for domain state, TanStack Query for server data.

### References

- Syncfusion – Top 5 React State Management Tools Developers Actually Use in 2026: https://www.syncfusion.com/blogs/post/react-state-management-libraries
- Syncfusion – Redux vs Zustand: Choosing the Right React State Manager: https://www.syncfusion.com/blogs/post/redux-vs-zustand
- Redux Official Documentation – Style Guide: https://redux.js.org/style-guide/
- Zustand – Comparison: https://zustand.docs.pmnd.rs/learn/getting-started/comparison
- Jotai – Comparison: https://jotai.org/docs/basics/comparison
- IEEE Xplore – Evaluation of State Management Libraries: https://ieeexplore.ieee.org/document/10293011
- Tuni University – State Management in React Applications: https://trepo.tuni.fi/

---

## Comparison and Decision Guidance

| Library | Paradigm | Bundle Size | Boilerplate | Learning Curve | Debugging | Best For |
|---|---|---|---|---|---|---|
| **Redux Toolkit** | Flux / Centralised | ~13 kB | High | Steep | Excellent (DevTools, time-travel) | Large enterprise apps, complex state, large teams |
| **Zustand** | Hook-based / Minimal | ~1.2 kB | Low | Gentle | Basic (DevTools) | Medium apps, rapid prototyping, React Native |
| **Jotai** | Atomic / Bottom-up | ~12 kB | Low | Gentle | Basic (DevTools) | UI state, fine-grained updates, atomic state |
| **MobX** | Reactive / OO | ~4.7 MB | Medium | Moderate | Limited | Object-oriented domain models, reactive programming |
| **XState** | State Machine / Actor | ~15 kB | High | Steep | Good (visualiser) | Complex workflows, authentication, wizards |
| **Signals** | Reactive Primitive | ~1.8 kB | Low | Gentle | Limited | High-frequency updates, UI primitives |

**Decision Tree:**

1. **Is the state server data?** → TanStack Query / SWR
2. **Is the state form data?** → React Hook Form
3. **Is the state URL-related?** → `useSearchParams`
4. **Is the state local to one component?** → `useState` / `useReducer`
5. **Is the state shared across components?** → Continue
6. **Is your team large and needs strict structure?** → Redux Toolkit
7. **Do you need minimal boilerplate and a small bundle?** → Zustand
8. **Do you have many independent, atomic pieces of state?** → Jotai
9. **Is your domain model object-oriented and reactive?** → MobX
10. **Do you have complex workflows with discrete states?** → XState
11. **Do you need high-frequency reactive updates?** → Signals

**The Anti-Goal:** One global store for everything. A global store that holds server data, form state, URL state, and derived values becomes a dumping ground with high coupling, low cohesion, and performance traps. Classify first, then choose the tool.

---

## References

- Redux Official Documentation – Three Principles: https://redux.js.org/understanding/thinking-in-redux/three-principles
- Redux Official Documentation – Redux Essentials Tutorial: https://redux.js.org/tutorials/essentials/part-1-overview-concepts
- Redux Toolkit – Overview: https://redux.js.org/redux-toolkit/overview
- Redux Toolkit – Quick Start: https://redux-toolkit.js.org/tutorials/quick-start
- Redux Toolkit – createSlice: https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit – configureStore: https://redux-toolkit.js.org/api/configureStore
- Zustand Documentation: https://zustand.docs.pmnd.rs/
- Zustand – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- Zustand – useShallow: https://zustand.docs.pmnd.rs/hooks/use-shallow
- Zustand – Persisting Store Data: https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- Jotai Official Documentation: https://jotai.org/
- Jotai – Large Objects (focusAtom, splitAtom): https://jotai.org/docs/recipes/large-objects
- Jotai – selectAtom: https://jotai.org/docs/utilities/select
- Jotai – Derived Atoms: https://jotai.org/docs/core/atom
- MobX Official Documentation: https://mobx.js.org/
- MobX – Creating Observable State: https://mobx.js.org/observable-state.html
- MobX – Understanding Reactivity: https://mobx.js.org/understanding-reactivity.html
- XState Official Documentation: https://stately.ai/docs/xstate
- XState – useMachine: https://stately.ai/docs/xstate-react
- Preact Signals – GitHub: https://github.com/preactjs/signals
- Preact Signals – Documentation: https://preactjs.com/guide/v10/signals/
- Syncfusion – Top 5 React State Management Tools Developers Actually Use in 2026: https://www.syncfusion.com/blogs/post/react-state-management-libraries
- Syncfusion – Redux vs Zustand: Choosing the Right React State Manager: https://www.syncfusion.com/blogs/post/redux-vs-zustand
- IEEE Xplore – Evaluation of State Management Libraries: https://ieeexplore.ieee.org/document/10293011
- DHTMLX – Using XState in React Gantt and Scheduler Apps: https://dhtmlx.com/blog/using-xstate-in-react-gantt-and-scheduler-apps-for-complex-state-scenarios/
- CoreUI – How to use Jotai in React: https://coreui.io/blog/how-to-use-jotai-in-react/
- GitHub – Zustand: https://github.com/pmndrs/zustand
- GitHub – Redux Three Principles: https://github.com/reduxjs/redux/blob/master/docs/understanding/thinking-in-redux/ThreePrinciples.md