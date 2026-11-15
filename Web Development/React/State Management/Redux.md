# Redux — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Redux is a predictable state container for JavaScript applications that centralizes application state in a single store, enforces that state can only change through dispatched actions processed by pure reducers, and enables powerful debugging through time-travel and state inspection.

**Technical Definition:** Redux is an open-source JavaScript library for managing application state. It operates on three fundamental principles: **single source of truth** (the global state of your application is stored in an object tree within a single store), **state is read-only** (the only way to change the state is to emit an action, an object describing what happened), and **changes are made with pure functions** (to specify how the state tree is transformed by actions, you write pure reducers) . The store holds the state tree, `dispatch` sends actions to reducers, `subscribe` registers listeners for state changes, and `getState` returns the current state. **Redux Toolkit (RTK)** is the official, opinionated, batteries-included toolset and the standard way to write Redux logic today, addressing the three most common complaints about legacy Redux ("configuring a store is too complicated," "I have to add a lot of packages," and "Redux requires too much boilerplate") . RTK includes `configureStore`, `createSlice`, `createAsyncThunk`, `createEntityAdapter`, `createSelector`, and **RTK Query** for data fetching and caching .

**Beginner-Friendly Explanation:** Redux is a tool that keeps all of your app's data in one place, like a single filing cabinet. You cannot open the cabinet and rewrite a file directly. Instead, you write a note (an action) saying what happened, and a clerk (the reducer) reads the note, looks at the current file, and produces a new, updated file. Because every change goes through the same process, you can rewind and replay everything, which makes debugging much easier. The modern way to use Redux is with Redux Toolkit, which handles all the repetitive setup for you.

### Key Characteristics

- **Single source of truth:** All application state lives in one store as an object tree .
- **State is read-only:** The only way to change state is to dispatch an action .
- **Pure reducers:** State transitions are handled by pure functions that return new state without mutating the previous state .
- **Redux Toolkit is the standard:** `configureStore` and `createSlice` are the recommended way to write Redux today .
- **Middleware ecosystem:** Thunk, Saga, and Observable handle async side effects with different paradigms .
- **RTK Query:** Built-in data fetching and caching layer that eliminates manual loading state .
- **DevTools:** Time-travel debugging, action inspection, and state tree visualization .
- **Immer integration:** `createSlice` uses Immer so reducers can be written with "mutating" syntax that is safely converted to immutable updates .

### Prerequisites

- JavaScript fundamentals: objects, arrays, functions, and immutability concepts.
- React function components and Hooks (`useSelector`, `useDispatch`).
- Asynchronous JavaScript: Promises, `async`/`await`, and side effects.
- Basic understanding of state management categories (local, server, global).

### Related Programming Areas

- **State Management** — Context API, Zustand, Jotai, TanStack Query.
- **Middleware** — Redux Thunk, Redux Saga, Redux Observable.
- **Data Fetching** — RTK Query, TanStack Query, SWR.
- **DevTools** — Redux DevTools Extension for time-travel debugging.

---

## 1. Redux Principles

### Definitions

**Core Definition:** Redux's three principles — single source of truth, read-only state, and pure-function changes — are the architectural rules that make Redux predictable and debuggable.

**Technical Definition:** The three principles are: (1) **Single source of truth** — the global state of your application is stored in an object tree within a single store. This makes it easy to create universal apps, as the state from your server can be serialized and hydrated into the client with no extra coding effort. A single state tree also makes it easier to debug or inspect an application and enables persisting state in development . (2) **State is read-only** — the only way to change the state is to emit an action, an object describing what happened. This ensures that neither views nor network callbacks ever write directly to the state; they express an intent to transform it. Because all changes are centralized and happen one by one in a strict order, there are no subtle race conditions. Actions are plain objects and can be logged, serialized, stored, and replayed . (3) **Changes are made with pure functions** — to specify how the state tree is transformed by actions, you write pure reducers. Reducers take the previous state and an action and return the next state. You return new state objects instead of mutating the previous state. Reducers are just functions, so you can control the order in which they are called, pass additional data, or make reusable reducers .

**Beginner-Friendly Explanation:** Three rules make Redux reliable. First, all your data lives in one place — no scattered copies. Second, you cannot change the data directly; you must describe what happened. Third, the function that updates the data must be pure: same inputs, same output, no side effects.

### Purposes

- To make application state predictable and debuggable.
- To enable time-travel debugging and state replay.
- To centralize state transitions in a single, auditable location.
- To support universal (server-rendered) apps through serializable state.
- To make undo/redo and state persistence trivial to implement.

### Syntax Rules and Structure

| Principle | Rule | Consequence |
|-----------|------|-------------|
| Single source of truth | One store, one state tree | Serializable, inspectable, hydratable |
| State is read-only | Only actions can change state | No race conditions, fully auditable |
| Pure reducers | Reducers return new state, never mutate | Predictable, testable, replayable |

**Syntax Rules:**

1. All application state must live in the single store; do not duplicate state elsewhere.
2. Never mutate state directly; always dispatch an action.
3. Reducers must be pure: no side effects, no external mutations, no randomness.
4. Reducers must return new objects/arrays; never modify the previous state.
5. Split reducers as the app grows, but keep them pure and composable.

**Constraints and Limitations:**

- The single store can become a bottleneck if too much unrelated state is crammed into it.
- Pure reducers cannot perform async operations; side effects belong in middleware.
- Deeply nested state requires careful immutable updates.

### Annotated Code Examples

```js
// Single source of truth: one store, one state tree
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: {
    todos: todosReducer,
    visibilityFilter: visibilityFilterReducer,
  },
});

console.log(store.getState());
// { todos: [...], visibilityFilter: 'SHOW_ALL' }

// State is read-only: dispatch an action to change it
store.dispatch({ type: 'todos/todoAdded', payload: 'Learn Redux' });

// Changes are made with pure functions
function todosReducer(state = [], action) {
  switch (action.type) {
    case 'todos/todoAdded':
      return [...state, { text: action.payload, completed: false }];
    default:
      return state;
  }
}
```

**Expected Output:** The state tree is a single object; the only way to add a todo is to dispatch an action; the reducer returns a new array instead of mutating the old one.

**Why this output:** The three principles are enforced by construction. The store holds everything; `dispatch` is the only mutation path; the reducer is pure and returns new state.

### Real-World Cases

- **Undo/redo:** Because every change is an action, you can record and replay them.
- **Time-travel debugging:** Redux DevTools can step through every state transition.
- **Server hydration:** The serializable state tree can be sent from server to client.
- **Testing:** Pure reducers are trivial to unit test.

### References

- Redux — Three Principles — https://redux.js.org/understanding/thinking-in-redux/three-principles
- Redux — Three Principles (2026) — https://redux.js.org/understanding/thinking-in-redux/three-principles
- Redux — Three Principles (GitBook) — https://redux.gitbook.io

---

## 2. Actions

### Definitions

**Core Definition:** An action is a plain JavaScript object that describes what happened in the application, and it is the only way to send data to the Redux store.

**Technical Definition:** Actions are plain objects with a `type` field (a string constant) and optionally a `payload` field containing additional data. They are dispatched to the store via `store.dispatch(action)`. Actions are the only source of information for the store; they describe an event that occurred, not how the state should change. In Redux Toolkit, `createSlice` generates action creators and action types automatically from the reducer names.

**Beginner-Friendly Explanation:** An action is a note that says "this happened." It has a type (like `'todos/todoAdded'`) and sometimes extra data (the payload). You dispatch the note to the store, and the reducers decide what to do with it.

### Purposes

- To describe state changes as plain, serializable events.
- To enable logging, replaying, and time-travel debugging.
- To decouple event producers from state transformers.
- To provide a single, auditable path for all state changes.

### Syntax Rules and Structure

```js
// Manual action
const action = { type: 'todos/todoAdded', payload: 'Learn Redux' };
store.dispatch(action);
```

**Syntax Rules:**

1. Every action must have a `type` field (a string).
2. The `payload` field is conventional and can contain any serializable data.
3. Actions must be plain objects (no class instances, functions, or Promises).
4. Action types should be descriptive and namespaced (e.g., `'todos/todoAdded'`).
5. In Redux Toolkit, use `createSlice` to generate action creators automatically.

**Constraints and Limitations:**

- Actions must be serializable; non-serializable values break DevTools and persistence.
- Action types must be unique across the application.
- Do not mutate actions inside reducers (reducers must be pure).

### Annotated Code Examples

```js
// Action creator (manual)
function todoAdded(text) {
  return { type: 'todos/todoAdded', payload: text };
}

store.dispatch(todoAdded('Learn Redux'));

// Redux Toolkit generates action creators
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    todoAdded(state, action) {
      state.push({ text: action.payload, completed: false });
    },
  },
});

todosSlice.actions.todoAdded('Learn Redux'); // { type: 'todos/todoAdded', payload: 'Learn Redux' }
```

**Expected Output:** Dispatching `todoAdded('Learn Redux')` sends an action with type `'todos/todoAdded'` and payload `'Learn Redux'` to the store.

**Why this output:** The action is a plain object. Redux Toolkit's `createSlice` automatically creates the action creator and type from the reducer name.

### Real-World Cases

- **Form submissions:** Dispatching actions with form data as payload.
- **User interactions:** Button clicks, toggles, and selections.
- **Async lifecycle:** `pending`, `fulfilled`, and `rejected` actions from `createAsyncThunk`.
- **WebSocket events:** Dispatching actions when messages arrive.

### References

- Redux — Actions — https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow
- Redux Toolkit — createSlice — https://redux-toolkit.js.org/api/createSlice
- Redux — Three Principles (actions) — https://redux.js.org/understanding/thinking-in-redux/three-principles

---

## 3. Action Creators

### Definitions

**Core Definition:** An action creator is a function that returns an action object, encapsulating the creation of actions and reducing duplication.

**Technical Definition:** Action creators are functions that create and return action objects. They can be as simple as `() => ({ type: 'INCREMENT' })` or take arguments and include them in the payload. In Redux Toolkit, `createSlice` automatically generates action creators for every reducer defined in the `reducers` object. These generated action creators are available on `slice.actions` and have the correct `type` string.

**Beginner-Friendly Explanation:** An action creator is a helper function that builds an action for you. Instead of writing `{ type: 'todos/todoAdded', payload: text }` every time, you call `todoAdded(text)`.

### Purposes

- To avoid repeating action object shapes.
- To centralize action type strings.
- To make dispatching actions more readable and less error-prone.
- To enable generated action creators in Redux Toolkit.

### Syntax Rules and Structure

```js
// Manual action creator
const increment = () => ({ type: 'counter/increment' });
const addBy = (amount) => ({ type: 'counter/addBy', payload: amount });

// Generated action creators (Redux Toolkit)
const { increment, addBy } = counterSlice.actions;
```

**Syntax Rules:**

1. Action creators are pure functions that return plain objects.
2. They should not have side effects.
3. In Redux Toolkit, use `createSlice` to generate them automatically.
4. Generated action creators have `type` and `match` properties.

**Constraints and Limitations:**

- Manual action creators can drift from action types if not centralized.
- Redux Toolkit's generated action creators are the recommended approach.

### Annotated Code Examples

```js
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
    addBy(state, action) { state.value += action.payload; },
  },
});

// Generated action creators
console.log(counterSlice.actions.increment()); // { type: 'counter/increment' }
console.log(counterSlice.actions.addBy(5));     // { type: 'counter/addBy', payload: 5 }
```

**Expected Output:** `increment()` returns `{ type: 'counter/increment' }` and `addBy(5)` returns `{ type: 'counter/addBy', payload: 5 }`.

**Why this output:** `createSlice` generates action creators from the reducer names, ensuring the action type matches the reducer.

### Real-World Cases

- **Redux Toolkit slices:** Every slice exposes generated action creators.
- **Typed dispatch:** Action creators provide type safety with TypeScript.
- **Testing:** Easier to assert on action shapes.

### References

- Redux Toolkit — createSlice: actions — https://redux-toolkit.js.org/api/createSlice#return-value
- Redux — Actions: Action Creators — https://redux.js.org/usage/reducing-boilerplate

---

## 4. Reducers

### Definitions

**Core Definition:** A reducer is a pure function that takes the current state and an action, and returns the next state.

**Technical Definition:** Reducers specify how the application's state changes in response to actions sent to the store. They are pure functions: given the same state and action, they must always return the same result, with no side effects, no mutation of the previous state, and no external dependencies. Redux Toolkit's `createSlice` uses Immer, allowing reducers to be written with "mutating" syntax that is safely converted to immutable updates . Reducers must return new state objects instead of mutating the previous state .

**Beginner-Friendly Explanation:** A reducer is the function that decides how the state changes. You give it the old state and an action, and it returns a new state. It must be pure: no side effects, no mutating the old state.

### Purposes

- To centralize state transition logic.
- To make state changes predictable and testable.
- To enable time-travel debugging through pure state transitions.
- To compose multiple reducers into a root reducer.

### Syntax Rules and Structure

```js
// Manual reducer
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'counter/increment':
      return { ...state, value: state.value + 1 };
    default:
      return state;
  }
}

// Redux Toolkit reducer (Immer-powered "mutating" syntax)
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
  },
});
```

**Syntax Rules:**

1. Reducers must be pure functions with no side effects.
2. Return new state objects; never mutate the previous state.
3. Always return the current state for unrecognized actions.
4. Split reducers as the app grows; combine them with `combineReducers` or `configureStore`.
5. In Redux Toolkit, use `createSlice` for Immer-powered reducers.

**Constraints and Limitations:**

- Reducers must not perform async operations, generate random values, or access external state.
- Deeply nested updates require careful immutable logic (Immer simplifies this).
- Mutating state breaks re-renders and DevTools time-travel .

### Annotated Code Examples

```js
// ✅ Pure reducer: returns new state
function todosReducer(state = [], action) {
  switch (action.type) {
    case 'todos/todoAdded':
      return [...state, { text: action.payload, completed: false }];
    case 'todos/todoToggled':
      return state.map((todo, index) =>
        index === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    default:
      return state;
  }
}

// ✅ Redux Toolkit reducer: Immer makes "mutation" safe
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    todoAdded(state, action) {
      state.push({ text: action.payload, completed: false });
    },
  },
});
```

**Expected Output:** The manual reducer returns new arrays; the Redux Toolkit reducer uses Immer to produce immutable updates from "mutating" code.

**Why this output:** Pure reducers return new state without mutating the old state. Redux Toolkit's Immer integration lets you write simpler code while preserving immutability.

### Real-World Cases

- **Entity lists:** Adding, updating, and removing items.
- **Filters and sorting:** Updating view state.
- **Form state:** Managing multi-step form data.
- **Normalized data:** Managing entities by ID.

### References

- Redux — Reducers — https://redux.js.org/tutorials/fundamentals/part-3-state-actions-reducers
- Redux Toolkit — createSlice — https://redux-toolkit.js.org/api/createSlice
- Redux Style Guide — Reducers Must Not Have Side Effects — https://redux.js.org/style-guide#reducers-must-not-have-side-effects
- Redux — Three Principles (pure functions) — https://redux.js.org/understanding/thinking-in-redux/three-principles

---

## 5. Store

### Definitions

**Core Definition:** The store is the object that holds the complete state tree of the application, provides methods to read and update it, and notifies listeners of changes.

**Technical Definition:** The Redux store brings together the state, actions, and reducers. It exposes `getState()` (returns the current state tree), `dispatch(action)` (triggers a state change), `subscribe(listener)` (registers a callback for state changes), and `replaceReducer(nextReducer)` (for hot reloading and code splitting). In Redux Toolkit, the store is created with `configureStore`, which automatically combines slice reducers, adds the thunk middleware, enables development-mode immutability and serializability checks, and enables the Redux DevTools Extension .

**Beginner-Friendly Explanation:** The store is the single object that holds all your app's data. You read from it with `getState`, change it with `dispatch`, and listen for changes with `subscribe`.

### Purposes

- To hold the single source of truth for application state.
- To provide a controlled API for reading and updating state.
- To notify subscribers when state changes.
- To serve as the integration point for middleware and DevTools.

### Syntax Rules and Structure

**Redux Toolkit (recommended):**

```js
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: {
    todos: todosReducer,
    counter: counterReducer,
  },
});
```

**Legacy `createStore` (not recommended):**

```js
import { createStore } from 'redux';
const store = createStore(rootReducer);
```

**Syntax Rules:**

1. Use `configureStore` from Redux Toolkit; `createStore` is deprecated .
2. `configureStore` accepts an object with a `reducer` field (a single reducer or an object of slice reducers).
3. The store provides `getState`, `dispatch`, and `subscribe`.
4. Use the React-Redux `Provider` to make the store available to components.

**Constraints and Limitations:**

- Only one store per application is the Redux convention.
- `createStore` is legacy; use `configureStore` for modern Redux.
- The store should not be recreated on every render; create it once at module level.

### Annotated Code Examples

```js
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './todosSlice';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: {
    todos: todosReducer,
    counter: counterReducer,
  },
});

// Reading state
console.log(store.getState()); // { todos: [...], counter: { value: 0 } }

// Dispatching actions
store.dispatch({ type: 'counter/increment' });

// Subscribing to changes
const unsubscribe = store.subscribe(() =>
  console.log('State changed:', store.getState())
);
```

**Expected Output:** `configureStore` creates a store with combined reducers, thunk middleware, and DevTools support. `getState` returns the state tree; `dispatch` updates it; `subscribe` notifies listeners.

**Why this output:** `configureStore` wraps the slice reducers into a root reducer and applies the default middleware. The store exposes the standard Redux API.

### Real-World Cases

- **Application root:** One store provided to the entire React tree.
- **SSR:** A new store per request to avoid cross-request leaks.
- **Testing:** A fresh store per test to isolate state.

### References

- Redux Toolkit — configureStore — https://redux-toolkit.js.org/api/configureStore
- Redux — Store — https://redux.js.org/api/store
- Redux Style Guide — Use Redux Toolkit — https://redux.js.org/style-guide#use-redux-toolkit-for-writing-redux-logic
- agents-inc/skills — Redux Toolkit MUST use configureStore — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-redux-toolkit/SKILL.md

---

## 6. `createStore`

### Definitions

**Core Definition:** `createStore` is the legacy Redux function for creating a store, now superseded by Redux Toolkit's `configureStore`.

**Technical Definition:** `createStore(reducer, preloadedState?, enhancer?)` creates a Redux store holding the state tree. It is the original Redux API and remains functional, but the Redux team recommends `configureStore` for all Redux applications because `createStore` requires manual middleware and DevTools setup, does not include development-mode checks, and leads to boilerplate . Redux Toolkit's `configureStore` is a drop-in replacement that wraps `createStore` with good defaults.

**Beginner-Friendly Explanation:** `createStore` is the old way to create a Redux store. It still works, but you should use `configureStore` instead because it sets up everything for you.

### Purposes

- To create a Redux store in legacy codebases.
- To understand the historical Redux API.
- To serve as the underlying primitive that `configureStore` wraps.

### Syntax Rules and Structure

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

const store = createStore(rootReducer, applyMiddleware(thunk));
```

**Syntax Rules:**

1. `createStore` is imported from `redux`.
2. Middleware must be applied manually with `applyMiddleware`.
3. DevTools must be configured manually with `composeWithDevTools`.
4. The Redux team strongly recommends `configureStore` instead .

**Constraints and Limitations:**

- No built-in development-mode checks for mutations or non-serializable values.
- Requires manual middleware and DevTools setup.
- Considered a legacy anti-pattern in modern Redux .

### Annotated Code Examples

```js
// ❌ Legacy: manual setup with createStore
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';

const store = createStore(
  rootReducer,
  compose(
    applyMiddleware(thunk),
    window.__REDUX_DEVTOOLS_EXTENSION__?.()
  )
);

// ✅ Modern: configureStore handles everything
import { configureStore } from '@reduxjs/toolkit';
const store = configureStore({ reducer: rootReducer });
```

**Expected Output:** Both create a store, but `configureStore` includes thunk, DevTools, and development checks automatically.

**Why this output:** `configureStore` is a wrapper around `createStore` that applies the recommended defaults, eliminating manual setup.

### Real-World Cases

- **Legacy codebases:** Maintaining applications that predate Redux Toolkit.
- **Migration:** Understanding what `configureStore` replaces.
- **Education:** Learning how Redux works underneath the toolkit.

### References

- Redux — createStore — https://redux.js.org/api/createstore
- Redux Toolkit — configureStore — https://redux-toolkit.js.org/api/configureStore
- Redux Style Guide — Use Redux Toolkit — https://redux.js.org/style-guide#use-redux-toolkit-for-writing-redux-logic

---

## 7. `configureStore`

### Definitions

**Core Definition:** `configureStore` is Redux Toolkit's store factory that sets up a Redux store with the recommended defaults, including middleware, DevTools, and development-mode checks.

**Technical Definition:** `configureStore({ reducer, middleware?, devTools?, preloadedState?, enhancer? })` creates a store with sensible defaults: it automatically combines slice reducers, includes the thunk middleware, adds development-mode checks for accidental mutations and non-serializable values, and enables the Redux DevTools Extension . It accepts an object with a `reducer` field (a single reducer or an object of slice reducers). Custom middleware can be added with `middleware` (a callback or an array), and `devTools` controls DevTools configuration.

**Beginner-Friendly Explanation:** `configureStore` is the modern way to create a Redux store. It sets up everything you need — thunk, DevTools, and safety checks — with almost no configuration.

### Purposes

- To create a store with recommended defaults in one line.
- To automatically combine slice reducers.
- To include thunk middleware for async logic.
- To enable development-mode immutability and serializability checks.
- To enable Redux DevTools automatically.
- To support custom middleware and enhancers when needed.

### Syntax Rules and Structure

```js
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: {
    todos: todosReducer,
    counter: counterReducer,
  },
});
```

**Syntax Rules:**

1. `reducer` can be a single function or an object of slice reducers.
2. The default middleware includes thunk and development checks.
3. Use the `middleware` option to add custom middleware.
4. `devTools` defaults to `true` in development.
5. `configureStore` replaces `createStore` + `combineReducers` + `applyMiddleware` .

**Constraints and Limitations:**

- The default middleware includes serializability checks that may warn for non-serializable values (e.g., `Date`, `File`).
- Custom middleware ordering matters; use the callback form for precise control.

### Annotated Code Examples

```js
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './todosSlice';
import counterReducer from './counterSlice';
import logger from 'redux-logger';

export const store = configureStore({
  reducer: {
    todos: todosReducer,
    counter: counterReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger),
  devTools: process.env.NODE_ENV !== 'production',
});
```

**Expected Output:** A store with combined reducers, thunk, immutability checks, serializability checks, and the logger middleware appended.

**Why this output:** `configureStore` provides the defaults; the `middleware` callback adds custom middleware; `devTools` controls the extension.

### Real-World Cases

- **All Redux Toolkit applications:** The standard store creation method.
- **SSR:** Create a new store per request with `preloadedState`.
- **Testing:** Create a fresh store per test.

### References

- Redux Toolkit — configureStore — https://redux-toolkit.js.org/api/configureStore
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview
- agents-inc/skills — MUST use configureStore — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-redux-toolkit/SKILL.md

---

## 8. Dispatch

### Definitions

**Core Definition:** `dispatch` is the store method used to send an action to the store, triggering the reducers to compute the next state.

**Technical Definition:** `store.dispatch(action)` is the only way to trigger a state change. It sends the action to the store's reducer, which computes the new state. The store then notifies all subscribers. In React, the `useDispatch` hook returns a reference to the store's dispatch function. In Redux Toolkit, `createAsyncThunk` generates thunks that dispatch `pending`, `fulfilled`, and `rejected` actions.

**Beginner-Friendly Explanation:** `dispatch` is how you send an action to the store. You call `dispatch(action)`, the reducer runs, and the state updates.

### Purposes

- To trigger state changes by sending actions.
- To send async thunks that dispatch multiple actions.
- To provide a single, controlled mutation path.
- To integrate with React via `useDispatch`.

### Syntax Rules and Structure

```js
// Plain action
store.dispatch({ type: 'counter/increment' });

// Generated action creator
store.dispatch(counterSlice.actions.increment());

// Async thunk
store.dispatch(fetchUsers());
```

**Syntax Rules:**

1. `dispatch` accepts an action object or a thunk function (with thunk middleware).
2. In React, use `useDispatch()` to get the dispatch function.
3. Thunks can dispatch multiple actions and perform async work.
4. `dispatch` returns the dispatched action (or a promise for thunks).

**Constraints and Limitations:**

- Without thunk middleware, `dispatch` only accepts plain action objects.
- Dispatching non-serializable actions triggers development warnings.

### Annotated Code Examples

```js
import { useDispatch } from 'react-redux';
import { todoAdded } from './todosSlice';

function AddTodo() {
  const dispatch = useDispatch();
  const [text, setText] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    dispatch(todoAdded(text)); // Dispatch an action
    setText('');
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

**Expected Output:** Submitting the form dispatches `todos/todoAdded`, the reducer adds the todo, and the UI updates.

**Why this output:** `useDispatch` returns the store's dispatch function. `todoAdded(text)` creates the action object, which `dispatch` sends to the reducer.

### Real-World Cases

- **Event handlers:** Dispatching actions on clicks, submissions, and changes.
- **Async thunks:** Dispatching thunks for API calls.
- **Effects:** Dispatching from `useEffect` after data loads.

### References

- Redux — Store: dispatch — https://redux.js.org/api/store#dispatchaction
- React Redux — useDispatch — https://react-redux.js.org/api/hooks#usedispatch
- Redux Toolkit — createAsyncThunk — https://redux-toolkit.js.org/api/createAsyncThunk

---

## 9. Subscribe

### Definitions

**Core Definition:** `subscribe` is the store method that registers a listener function called whenever an action is dispatched and the state changes.

**Technical Definition:** `store.subscribe(listener)` adds a change listener to the store. The listener is called after each `dispatch` (and after the state has been updated). It receives no arguments; the listener typically calls `store.getState()` to read the current state. `subscribe` returns an unsubscribe function. In React, `subscribe` is used internally by `useSelector`; developers rarely call it directly in React components.

**Beginner-Friendly Explanation:** `subscribe` lets you listen for state changes. You give it a function, and it calls that function every time the state changes.

### Purposes

- To react to state changes outside React components.
- To persist state to storage after changes.
- To log state transitions for debugging.
- To bridge Redux state into non-React code.

### Syntax Rules and Structure

```js
const unsubscribe = store.subscribe(() => {
  console.log('State changed:', store.getState());
});

// Later: stop listening
unsubscribe();
```

**Syntax Rules:**

1. `subscribe` accepts a listener function with no arguments.
2. It returns an unsubscribe function.
3. The listener runs after each dispatch.
4. In React, prefer `useSelector` over manual `subscribe`.

**Constraints and Limitations:**

- `subscribe` does not provide the previous state; the listener must read `getState()`.
- Subscribing in React components without cleanup causes memory leaks.

### Annotated Code Examples

```js
// Persist state to localStorage on every change
const unsubscribe = store.subscribe(() => {
  const state = store.getState();
  localStorage.setItem('appState', JSON.stringify(state));
});

// Later, during cleanup
unsubscribe();
```

**Expected Output:** Every state change is persisted to `localStorage`. Calling `unsubscribe()` stops the persistence.

**Why this output:** `subscribe` registers the listener, which runs after each dispatch. The returned function removes the listener.

### Real-World Cases

- **Persistence:** Saving state to `localStorage` after changes.
- **Logging:** Recording state transitions.
- **Non-React integration:** Bridging Redux into legacy code.

### References

- Redux — Store: subscribe — https://redux.js.org/api/store#subscribelistener
- React Redux — useSelector — https://react-redux.js.org/api/hooks#useselector

---

## 10. Selectors

### Definitions

**Core Definition:** A selector is a function that reads a specific piece of data from the Redux store state.

**Technical Definition:** Selectors take the Redux store state as an argument and return a derived value. They can be simple (returning a property) or memoized (using `createSelector` from Reselect) to compute derived data efficiently. In React, `useSelector(selector)` subscribes the component to the store and re-renders when the selected value changes. Selectors centralize state-reading logic and make it reusable and testable.

**Beginner-Friendly Explanation:** A selector is a function that gets a specific piece of data from the store. You give it the whole state, and it returns just the part you need.

### Purposes

- To encapsulate state-reading logic.
- To compute derived data efficiently with memoization.
- To avoid duplicating state access across components.
- To make state shape refactoring easier (components read from selectors, not raw state).

### Syntax Rules and Structure

```js
// Simple selector
const selectTodos = (state) => state.todos;

// Memoized selector
import { createSelector } from '@reduxjs/toolkit';
const selectCompletedTodos = createSelector(
  [(state) => state.todos],
  (todos) => todos.filter((todo) => todo.completed)
);

// In React
const todos = useSelector(selectTodos);
```

**Syntax Rules:**

1. Selectors take the entire state as their first argument.
2. Use `createSelector` for derived data that requires computation.
3. `useSelector` subscribes to the selected value.
4. Keep selectors pure and colocated with their slice.

**Constraints and Limitations:**

- Selectors that return new object/array references on every call cause re-renders; memoize them.
- Do not store selector results back into Redux state .

### Annotated Code Examples

```js
import { createSelector } from '@reduxjs/toolkit';

const selectTodos = (state) => state.todos;
const selectFilter = (state) => state.visibilityFilter;

const selectVisibleTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    switch (filter) {
      case 'SHOW_COMPLETED':
        return todos.filter((t) => t.completed);
      case 'SHOW_ACTIVE':
        return todos.filter((t) => !t.completed);
      default:
        return todos;
    }
  }
);

function TodoList() {
  const visibleTodos = useSelector(selectVisibleTodos);
  return <ul>{visibleTodos.map((t) => <li key={t.id}>{t.text}</li>)}</ul>;
}
```

**Expected Output:** `selectVisibleTodos` memoizes the filtered list, so the component only re-renders when the filtered result actually changes.

**Why this output:** `createSelector` caches the computed result and returns the same reference if the inputs are unchanged, preventing unnecessary re-renders.

### Real-World Cases

- **Filtered lists:** Selecting todos by status.
- **Aggregations:** Computing totals and counts.
- **Derived data:** Joining normalized entities.
- **Performance:** Preventing re-renders with memoized selectors.

### References

- Redux Toolkit — createSelector — https://redux-toolkit.js.org/api/createSelector
- React Redux — useSelector — https://react-redux.js.org/api/hooks#useselector
- Redux Style Guide — Do Not Store Selector Results — https://redux.js.org/style-guide

---

## 11. Middleware

### Definitions

**Core Definition:** Middleware is a function that sits between `dispatch` and the reducer, allowing you to intercept actions, perform side effects, and dispatch additional actions.

**Technical Definition:** Middleware in Redux is a higher-order function that wraps `dispatch`. It provides a third-party extension point between dispatching an action and the moment it reaches the reducer. Middleware is composed via `applyMiddleware` (or automatically included in `configureStore`). Common middleware includes Redux Thunk (function actions), Redux Saga (generator-based side effects), and Redux Observable (RxJS-based side effects).

**Beginner-Friendly Explanation:** Middleware is a layer that sits between your dispatch and the reducer. It can intercept actions, do async work, and dispatch more actions.

### Purposes

- To handle asynchronous side effects (API calls, timers).
- To log actions and state transitions.
- To handle action routing and transformation.
- To implement cross-cutting concerns (analytics, error reporting).

### Syntax Rules and Structure

**Middleware signature:**

```js
const middleware = (store) => (next) => (action) => {
  // Do something before
  const result = next(action);
  // Do something after
  return result;
};
```

**Applying middleware (Redux Toolkit):**

```js
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(myMiddleware),
});
```

**Syntax Rules:**

1. Middleware must call `next(action)` to pass the action along.
2. Middleware can dispatch additional actions.
3. In Redux Toolkit, use the `middleware` callback to add custom middleware.
4. The default middleware includes thunk and development checks.

**Constraints and Limitations:**

- Middleware ordering matters; custom middleware should typically run after the default checks.
- Middleware should not directly mutate state.

### Annotated Code Examples

```js
// Custom logger middleware
const logger = (store) => (next) => (action) => {
  console.log('dispatching:', action);
  const result = next(action);
  console.log('next state:', store.getState());
  return result;
};

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger),
});
```

**Expected Output:** Every dispatched action is logged before and after reaching the reducer.

**Why this output:** The middleware wraps `dispatch`, logs the action, passes it to the next middleware/reducer, and logs the resulting state.

### Real-World Cases

- **Async API calls:** Thunk, Saga, or Observable middleware.
- **Logging:** Development-time action logging.
- **Error tracking:** Catching and reporting errors from actions.
- **Analytics:** Dispatching analytics events based on actions.

### References

- Redux — Middleware — https://redux.js.org/understanding/history-and-design/middleware
- Redux Toolkit — Middleware — https://redux-toolkit.js.org/api/getDefaultMiddleware
- Redux — applyMiddleware — https://redux.js.org/api/applymiddleware

---

## 12. Redux Thunk

### Definitions

**Core Definition:** Redux Thunk is a middleware that allows action creators to return a function (a thunk) instead of an action, enabling delayed and conditional dispatching.

**Technical Definition:** Redux Thunk middleware allows you to write action creators that return a function instead of an action. The thunk is called with `(dispatch, getState)` and can dispatch multiple actions, perform async work, and access the current state. It is the simplest and most common middleware for async side effects and is included by default in Redux Toolkit's `configureStore` .

**Beginner-Friendly Explanation:** Redux Thunk lets you write action creators that return a function. That function can wait for an API call and then dispatch real actions when it is done.

### Purposes

- To handle asynchronous API calls.
- To dispatch multiple actions conditionally.
- To access the current state inside async logic.
- To sequence dependent actions.

### Syntax Rules and Structure

```js
// Thunk action creator
const fetchUser = (id) => async (dispatch, getState) => {
  dispatch({ type: 'user/loading' });
  try {
    const user = await api.getUser(id);
    dispatch({ type: 'user/loaded', payload: user });
  } catch (error) {
    dispatch({ type: 'user/error', payload: error.message });
  }
};

// Dispatch the thunk
store.dispatch(fetchUser(1));
```

**Syntax Rules:**

1. Thunk action creators return a function (not an action object).
2. The function receives `(dispatch, getState)`.
3. Dispatch regular actions for loading, success, and error states.
4. Thunk middleware is included by default in `configureStore`.

**Constraints and Limitations:**

- Thunks are simple; complex async flows (race conditions, cancellation, debouncing) require manual coordination.
- Testing thunks requires mocking `dispatch` and `getState`.

### Annotated Code Examples

```js
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await api.getUsers();
      return response.data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);
```

**Expected Output:** `fetchUsers` dispatches `users/fetchUsers/pending`, `fulfilled`, or `rejected` based on the API result.

**Why this output:** `createAsyncThunk` generates the thunk and the three lifecycle actions, handling loading and error states automatically.

### Real-World Cases

- **API fetching:** Loading user data, lists, and details.
- **Form submission:** Sending data and handling success/error.
- **Sequential requests:** Fetching data that depends on previous results.

### References

- Redux Thunk — GitHub — https://github.com/reduxjs/redux-thunk
- Redux Toolkit — createAsyncThunk — https://redux-toolkit.js.org/api/createAsyncThunk
- SitePoint — Async Operations in React Redux Applications — https://www.sitepoint.com/async-operations-react-redux-applications/

---

## 13. Redux Saga

### Definitions

**Core Definition:** Redux Saga is a middleware that uses ES6 generators to handle complex asynchronous side effects, making them easy to test and reason about.

**Technical Definition:** Redux Saga is a library that aims to make application side effects (async data fetching, accessing browser cache, and other impure operations) easier to manage, more efficient to execute, simple to test, and better at handling failures. It uses ES6 generator functions called "sagas" that yield plain JavaScript objects describing the side effects (e.g., `call`, `put`, `take`, `fork`, `race`). The saga middleware runs these generators, executing the effects and resuming the saga with the results. Sagas are excellent for complex flows with race conditions, cancellation, and parallel execution .

**Beginner-Friendly Explanation:** Redux Saga is a more powerful way to handle async work. Instead of writing async functions, you write generator functions that yield instructions like "call this API" or "dispatch this action." Saga executes them and gives you the result.

### Purposes

- To manage complex async flows with cancellation and race conditions.
- To test side effects declaratively (generators yield plain objects).
- To handle long-running background tasks (WebSockets, polling).
- To coordinate multiple sagas running in parallel.

### Syntax Rules and Structure

```js
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects';

function* fetchUser(action) {
  try {
    const user = yield call(api.getUser, action.payload);
    yield put({ type: 'user/loaded', payload: user });
  } catch (error) {
    yield put({ type: 'user/error', payload: error.message });
  }
}

function* userSaga() {
  yield takeLatest('user/fetchRequested', fetchUser);
}

// Root saga
function* rootSaga() {
  yield all([userSaga()]);
}

// Setup
const sagaMiddleware = createSagaMiddleware();
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({ thunk: false }).concat(sagaMiddleware),
});
sagaMiddleware.run(rootSaga);
```

**Syntax Rules:**

1. Sagas are generator functions that yield effects.
2. Use `takeEvery` for every action, `takeLatest` for the latest only.
3. Use `call` for invoking functions, `put` for dispatching actions.
4. Use `race` for race conditions and `fork` for parallel tasks.
5. Run the root saga with `sagaMiddleware.run(rootSaga)`.

**Constraints and Limitations:**

- Generators have a learning curve.
- Sagas add bundle size and complexity.
- More boilerplate than thunks for simple cases.

### Annotated Code Examples

```js
import { call, put, takeLatest } from 'redux-saga/effects';

function* fetchUserSaga(action) {
  try {
    yield put({ type: 'user/loading' });
    const user = yield call(api.fetchUser, action.payload);
    yield put({ type: 'user/loaded', payload: user });
  } catch (error) {
    yield put({ type: 'user/error', payload: error.message });
  }
}

function* watchFetchUser() {
  yield takeLatest('user/fetchRequested', fetchUserSaga);
}
```

**Expected Output:** When `user/fetchRequested` is dispatched, the saga calls the API, dispatches loading, and then dispatches loaded or error. `takeLatest` cancels the previous request if a new one arrives.

**Why this output:** The saga yields effects (`call`, `put`) that the middleware executes. `takeLatest` ensures only the latest request is processed.

### Real-World Cases

- **Complex API flows:** Dependent requests, cancellation, and retries.
- **WebSocket management:** Listening for messages and dispatching actions.
- **Background sync:** Periodic polling and data synchronization.
- **Race conditions:** `race` to handle competing async operations.

### References

- Redux Saga — Official Documentation — https://redux-saga.js.org/
- SitePoint — Async Operations in React Redux Applications — https://www.sitepoint.com/async-operations-react-redux-applications/
- Stack Overflow — Saga vs Thunk vs Observable — https://stackoverflow.com

---

## 14. Redux Observable

### Definitions

**Core Definition:** Redux Observable is a middleware that uses RxJS Observables to handle asynchronous side effects as streams of actions.

**Technical Definition:** Redux Observable is a middleware for Redux that is inspired by redux-thunk. It allows developers to dispatch a function that returns an Observable, Promise, or iterable of action(s). The middleware then subscribes to the Observable and dispatches the emitted actions. Epics (the Observable functions) receive a stream of actions and return a stream of actions. It is built on RxJS and provides excellent scalability and control over async operations in sophisticated app architectures .

**Beginner-Friendly Explanation:** Redux Observable treats actions as a stream and lets you write functions (epics) that listen to that stream and emit new actions. It is the most powerful option, but it requires knowing RxJS.

### Purposes

- To manage complex async flows with RxJS operators.
- To handle high-frequency events (search, scroll) with debouncing and throttling.
- To compose and transform streams of actions.
- To provide fine-grained control over side effects.

### Syntax Rules and Structure

```js
import { ofType } from 'redux-observable';
import { mergeMap, map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';
import { ajax } from 'rxjs/ajax';

const fetchUserEpic = (action$) =>
  action$.pipe(
    ofType('user/fetchRequested'),
    mergeMap((action) =>
      ajax.getJSON(`/api/users/${action.payload}`).pipe(
        map((user) => ({ type: 'user/loaded', payload: user })),
        catchError((error) => of({ type: 'user/error', payload: error.message }))
      )
    )
  );
```

**Syntax Rules:**

1. Epics are functions that take a stream of actions and return a stream of actions.
2. Use `ofType` to filter actions.
3. Use RxJS operators (`mergeMap`, `switchMap`, `debounceTime`, etc.) for async logic.
4. Register epics with `combineEpics` and `createEpicMiddleware`.

**Constraints and Limitations:**

- Requires knowledge of RxJS operators and reactive programming.
- Steeper learning curve than thunks.
- Overkill for simple async flows.

### Annotated Code Examples

```js
import { ofType } from 'redux-observable';
import { switchMap, map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';
import { ajax } from 'rxjs/ajax';

const searchEpic = (action$) =>
  action$.pipe(
    ofType('search/queryChanged'),
    switchMap((action) =>
      ajax.getJSON(`/api/search?q=${action.payload}`).pipe(
        map((results) => ({ type: 'search/resultsLoaded', payload: results })),
        catchError((error) => of({ type: 'search/error', payload: error.message }))
      )
    )
  );
```

**Expected Output:** When `search/queryChanged` is dispatched, the epic cancels the previous request (via `switchMap`) and fetches new results for the latest query.

**Why this output:** `switchMap` unsubscribes from the previous inner Observable, cancelling stale requests and ensuring only the latest result is dispatched.

### Real-World Cases

- **Search autocomplete:** Debouncing and cancelling stale requests.
- **High-frequency events:** Scroll, resize, and mouse move streams.
- **Complex async composition:** Coordinating multiple streams.
- **Real-time data:** WebSocket message streams.

### References

- Redux Observable — Official Documentation — https://redux-observable.js.org/
- SitePoint — Async Operations in React Redux Applications — https://www.sitepoint.com/async-operations-react-redux-applications/
- Stack Overflow — Saga vs Thunk vs Observable — https://stackoverflow.com

---

## 15. Redux Toolkit

### Definitions

**Core Definition:** Redux Toolkit (RTK) is the official, opinionated, batteries-included toolset for efficient Redux development, and the standard way to write Redux logic today.

**Technical Definition:** Redux Toolkit is the official, opinionated, batteries-included toolset for efficient Redux development. It is intended to be the standard way to write Redux logic, and the Redux team strongly recommends it. It includes `configureStore` (store setup with good defaults), `createSlice` (reducer and action creator generation with Immer), `createAsyncThunk` (async action generation), `createEntityAdapter` (normalized state management), `createSelector` (memoized selectors from Reselect), and RTK Query (data fetching and caching) . It addresses the three most common complaints about legacy Redux: complicated store configuration, too many packages, and too much boilerplate .

**Beginner-Friendly Explanation:** Redux Toolkit is the modern, official way to use Redux. It gives you functions that handle the repetitive parts — creating the store, writing reducers, handling async — so you can write less code and avoid common mistakes.

### Purposes

- To eliminate Redux boilerplate.
- To provide good defaults for store setup (thunk, DevTools, checks).
- To simplify reducer and action creation with Immer.
- To handle async logic with `createAsyncThunk`.
- To manage normalized data with `createEntityAdapter`.
- To provide data fetching and caching with RTK Query.
- To enforce best practices and catch common mistakes.

### Syntax Rules and Structure

**Core APIs:**

| API | Purpose |
|-----|---------|
| `configureStore` | Store setup with defaults |
| `createSlice` | Reducers + action creators + action types |
| `createAsyncThunk` | Async lifecycle actions |
| `createEntityAdapter` | Normalized entity state |
| `createSelector` | Memoized selectors |
| `RTK Query` | Data fetching and caching |

**Syntax Rules:**

1. Use `configureStore` for store setup; never `createStore` .
2. Use `createSlice` for reducers; never switch statements or manual action creators .
3. Use `createAsyncThunk` for async logic; never hand-write pending/fulfilled/rejected actions.
4. Use `createEntityAdapter` for normalized collections.
5. Use `createSelector` for derived data.
6. Use RTK Query for server state; do not duplicate it in slices.

**Constraints and Limitations:**

- RTK Query requires `middleware` setup; do not omit it .
- Typed hooks (`useAppSelector`, `useAppDispatch`) are recommended for TypeScript projects.

### Annotated Code Examples

```js
import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchTodos = createAsyncThunk('todos/fetchTodos', async () => {
  const response = await api.getTodos();
  return response.data;
});

// Slice
const todosSlice = createSlice({
  name: 'todos',
  initialState: { items: [], status: 'idle', error: null },
  reducers: {
    todoAdded(state, action) {
      state.items.push(action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

// Store
export const store = configureStore({
  reducer: {
    todos: todosSlice.reducer,
  },
});
```

**Expected Output:** A complete Redux setup with async data fetching, slice reducers, and store configuration, with minimal boilerplate.

**Why this output:** `createAsyncThunk` generates the lifecycle actions, `createSlice` generates the reducer and action creators, and `configureStore` combines them with defaults.

### Real-World Cases

- **All new Redux applications:** Redux Toolkit is the standard.
- **Migrating legacy Redux:** Incrementally replace `createStore` with `configureStore` and `createSlice` .
- **Normalized data:** `createEntityAdapter` for entities keyed by ID.

### References

- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview
- Redux Toolkit — Quick Start — https://redux-toolkit.js.org/tutorials/quick-start
- agents-inc/skills — Redux Toolkit Best Practices — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-redux-toolkit/SKILL.md
- Redux — Migrating to Modern Redux — https://redux.js.org/usage/migrating-to-modern-redux

---

### 15.1 `createSlice`

**Definitions:**

- **Core Definition:** `createSlice` is a Redux Toolkit function that accepts a slice name, an initial state, and an object of reducer functions, and generates the slice reducer, action creators, and action types.
- **Technical Definition:** `createSlice({ name, initialState, reducers, extraReducers? })` returns a slice object with `name`, `reducer`, `actions`, `caseReducers`, and `getInitialState`. The `name` is used as the prefix for action types. Reducers can be written with "mutating" syntax thanks to Immer. `extraReducers` handles actions defined outside the slice (e.g., from `createAsyncThunk`) .
- **Beginner-Friendly Explanation:** `createSlice` creates a "slice" of your Redux state. You give it a name, initial state, and reducer functions, and it generates everything else for you.

**Purposes:**

- To eliminate handwritten action creators and action types.
- To write reducers with simpler "mutating" syntax via Immer.
- To group related state and reducers in one place.
- To handle external actions with `extraReducers`.

**Syntax Rules and Structure:**

```js
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
    decrement(state) { state.value -= 1; },
    addBy(state, action) { state.value += action.payload; },
  },
});

export const { increment, decrement, addBy } = counterSlice.actions;
export default counterSlice.reducer;
```

**Syntax Rules:**

1. `name` is used as the action type prefix.
2. `initialState` defines the starting state.
3. `reducers` is an object of reducer functions; each generates an action creator.
4. Immer allows "mutating" syntax safely.
5. `extraReducers` handles actions from other sources.

**Constraints and Limitations:**

- `createSlice` assumes Immer; do not manually mutate state outside the slice.
- `extraReducers` in Redux Toolkit 2.0 requires the builder callback pattern .

**Annotated Code Examples:**

```js
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    todoAdded(state, action) {
      state.push({ text: action.payload, completed: false });
    },
    todoToggled(state, action) {
      const todo = state.find((t) => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },
  },
});

todosSlice.actions.todoAdded('Learn Redux'); // { type: 'todos/todoAdded', payload: 'Learn Redux' }
```

**Real-World Cases:**

- **Feature slices:** One slice per feature (todos, counter, auth).
- **Entity management:** Combined with `createEntityAdapter`.
- **Async state:** `extraReducers` for `createAsyncThunk` actions.

**References:**

- Redux Toolkit — createSlice — https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview

---

### 15.2 `createAsyncThunk`

**Definitions:**

- **Core Definition:** `createAsyncThunk` is a Redux Toolkit function that generates a thunk for async operations and dispatches `pending`, `fulfilled`, and `rejected` actions.
- **Technical Definition:** `createAsyncThunk(typePrefix, payloadCreator, options?)` accepts an action type string and a function that returns a promise, and generates a thunk that dispatches `pending`, `fulfilled`, and `rejected` actions based on that promise . The `payloadCreator` receives `(arg, thunkAPI)` and can return data or reject with a value. The generated thunk also has `.pending`, `.fulfilled`, and `.rejected` properties for use in `extraReducers`.
- **Beginner-Friendly Explanation:** `createAsyncThunk` handles the three states of an API call — loading, success, and failure — automatically. You give it the API call function, and it generates the actions.

**Purposes:**

- To standardize async action lifecycle (pending, fulfilled, rejected).
- To avoid hand-writing loading, success, and error actions.
- To handle errors and cancellations.
- To integrate with `extraReducers` for state updates.

**Syntax Rules and Structure:**

```js
export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await api.getUser(userId);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);
```

**Syntax Rules:**

1. `typePrefix` is used as the base action type.
2. `payloadCreator` performs the async work.
3. `thunkAPI` provides `dispatch`, `getState`, `rejectWithValue`, and `signal`.
4. The generated thunk dispatches `user/fetchUser/pending`, `/fulfilled`, or `/rejected`.

**Constraints and Limitations:**

- Cannot be used for non-serializable payloads without configuration.
- Requires `extraReducers` to handle the lifecycle actions.

**Annotated Code Examples:**

```js
const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, status: 'idle', error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload;
      });
  },
});
```

**Real-World Cases:**

- **API data fetching:** Users, posts, products.
- **Form submission:** Sending data with loading and error states.
- **Authentication:** Login and logout flows.

**References:**

- Redux Toolkit — createAsyncThunk — https://redux-toolkit.js.org/api/createAsyncThunk
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview

---

### 15.3 `createEntityAdapter`

**Definitions:**

- **Core Definition:** `createEntityAdapter` is a Redux Toolkit function that generates reusable reducers and selectors for managing normalized data in the store.
- **Technical Definition:** `createEntityAdapter({ selectId?, sortComparer? })` returns an adapter with methods for managing normalized state: `addOne`, `addMany`, `setAll`, `updateOne`, `removeOne`, and generated selectors (`selectAll`, `selectById`, `selectIds`, `selectTotal`). The normalized state shape is `{ ids: [], entities: {} }` .
- **Beginner-Friendly Explanation:** `createEntityAdapter` helps you store collections of items (like users or todos) efficiently. It gives you ready-made functions for adding, updating, and selecting items.

**Purposes:**

- To manage normalized collections efficiently.
- To avoid writing repetitive add/update/remove reducers.
- To provide memoized selectors for entities.
- To support sorting and ID customization.

**Syntax Rules and Structure:**

```js
const usersAdapter = createEntityAdapter({
  selectId: (user) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name),
});

const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState(),
  reducers: {
    userAdded: usersAdapter.addOne,
    userUpdated: usersAdapter.updateOne,
    userRemoved: usersAdapter.removeOne,
  },
});

export const {
  selectAll: selectAllUsers,
  selectById: selectUserById,
} = usersAdapter.getSelectors((state) => state.users);
```

**Syntax Rules:**

1. `selectId` customizes the ID field (default `id`).
2. `sortComparer` sorts the `ids` array.
3. `getInitialState()` provides the initial normalized shape.
4. Adapter methods can be used directly as reducers.
5. `getSelectors(state => state.slice)` generates memoized selectors.

**Constraints and Limitations:**

- Deep nesting in entity state complicates updates .
- The normalized shape (`ids` + `entities`) is opinionated.

**Annotated Code Examples:**

```js
const booksAdapter = createEntityAdapter();
const booksSlice = createSlice({
  name: 'books',
  initialState: booksAdapter.getInitialState(),
  reducers: {
    bookAdded: booksAdapter.addOne,
    booksReceived: booksAdapter.setAll,
  },
});

const selectAllBooks = booksAdapter.getSelectors(
  (state) => state.books
).selectAll;
```

**Real-World Cases:**

- **User lists:** Normalized users by ID.
- **Product catalogs:** Products with sorting.
- **Todo lists:** Todos with add/update/remove.

**References:**

- Redux Toolkit — createEntityAdapter — https://redux-toolkit.js.org/api/createEntityAdapter
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview

---

### 15.4 `createSelector`

**Definitions:**

- **Core Definition:** `createSelector` is a Redux Toolkit re-export of Reselect's memoized selector creator, used to compute derived data efficiently.
- **Technical Definition:** `createSelector(inputSelectors, resultFunc)` creates a memoized selector that only recomputes the result when the input selectors' results change. It caches the last inputs and outputs, returning the cached result if the inputs are unchanged. This prevents unnecessary re-renders and expensive recomputations .
- **Beginner-Friendly Explanation:** `createSelector` remembers the last result of a computation. If the inputs are the same, it returns the cached result instead of recomputing.

**Purposes:**

- To memoize derived data computations.
- To prevent unnecessary re-renders.
- To compose complex selectors from simpler ones.
- To improve performance for expensive filtering or aggregation.

**Syntax Rules and Structure:**

```js
import { createSelector } from '@reduxjs/toolkit';

const selectTodos = (state) => state.todos;
const selectFilter = (state) => state.visibilityFilter;

const selectVisibleTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    switch (filter) {
      case 'SHOW_COMPLETED': return todos.filter((t) => t.completed);
      case 'SHOW_ACTIVE': return todos.filter((t) => !t.completed);
      default: return todos;
    }
  }
);
```

**Syntax Rules:**

1. Input selectors are passed as the first argument.
2. The result function receives the input selector results.
3. The selector caches the last result and inputs.
4. Only recomputes when inputs change (by reference).

**Constraints and Limitations:**

- Input selectors must return stable references to benefit from memoization.
- Do not store selector results back into Redux state .

**Annotated Code Examples:**

```js
const selectTodos = (state) => state.todos;
const selectCompletedCount = createSelector(
  [selectTodos],
  (todos) => todos.filter((t) => t.completed).length
);
```

**Real-World Cases:**

- **Filtered lists:** Filtering todos by status.
- **Aggregations:** Counting completed items.
- **Derived entities:** Joining normalized data.

**References:**

- Redux Toolkit — createSelector — https://redux-toolkit.js.org/api/createSelector
- Reselect — GitHub — https://github.com/reduxjs/reselect

---

### 15.5 RTK Query

**Definitions:**

- **Core Definition:** RTK Query is a powerful data fetching and caching layer built into Redux Toolkit that eliminates the need to hand-write loading state, caching, and data fetching logic.
- **Technical Definition:** RTK Query is a data fetching and caching layer built for Redux. You define API endpoints, and it generates the reducers, thunks, and React hooks for fetching, caching, and invalidating data. It uses a **cache tag** system to automate re-fetching for query endpoints whose data is affected by mutation endpoints . Queries provide tags (`providesTags`), mutations invalidate tags (`invalidatesTags`), and RTK Query automatically refetches affected queries. It supports automatic caching, deduplication, polling, optimistic updates, and streaming updates .
- **Beginner-Friendly Explanation:** RTK Query handles data fetching for you. You define your API endpoints, and it gives you hooks that fetch, cache, and update data automatically.

**Purposes:**

- To eliminate manual loading and error state management.
- To automatically cache and deduplicate requests.
- To automatically refetch data after mutations via tags.
- To generate typed React hooks for each endpoint.
- To support optimistic updates and streaming.
- To replace hand-written `createAsyncThunk` + slice logic for server state.

**Syntax Rules and Structure:**

```js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Post', 'User'],
  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => '/posts',
      providesTags: ['Post'],
    }),
    addPost: builder.mutation({
      query: (post) => ({
        url: '/posts',
        method: 'POST',
        body: post,
      }),
      invalidatesTags: ['Post'],
    }),
  }),
});

export const { useGetPostsQuery, useAddPostMutation } = api;

// Store setup
const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});
```

**Syntax Rules:**

1. `createApi` defines the API slice with `baseQuery`, `tagTypes`, and `endpoints`.
2. `builder.query` defines GET endpoints; `builder.mutation` defines write endpoints.
3. `providesTags` on queries and `invalidatesTags` on mutations drive automatic refetching .
4. The generated hooks (`useGetPostsQuery`, `useAddPostMutation`) handle loading, error, and caching.
5. The API reducer and middleware must be added to the store.

**Constraints and Limitations:**

- Requires `api.middleware` in the store; omitting it is an anti-pattern .
- Cache invalidation relies on correct tag configuration.
- Overkill for very simple, one-off fetches.

**Annotated Code Examples:**

```js
const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['User'],
  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => '/users',
      providesTags: ['User'],
    }),
    addUser: builder.mutation({
      query: (user) => ({
        url: '/users',
        method: 'POST',
        body: user,
      }),
      invalidatesTags: ['User'],
    }),
  }),
});

function UserList() {
  const { data, isLoading, error } = useGetUsersQuery();
  const [addUser] = useAddUserMutation();

  if (isLoading) return <Spinner />;
  if (error) return <ErrorBanner />;
  return <List users={data} onAdd={addUser} />;
}
```

**Expected Output:** `useGetUsersQuery` fetches and caches the user list. After `addUser` succeeds, `invalidatesTags: ['User']` triggers a refetch of the user list automatically.

**Why this output:** RTK Query's tag system connects mutations to queries. Invalidating a tag marks the associated queries as stale and refetches them.

**Real-World Cases:**

- **CRUD applications:** Lists, details, create, update, delete.
- **Dashboards:** Cached stats with invalidation.
- **E-commerce:** Product catalogs with mutation-driven refresh.

**References:**

- RTK Query — Overview — https://redux-toolkit.js.org/rtk-query/overview
- Redux Essentials — RTK Query Advanced Patterns — https://redux.js.org/tutorials/essentials/part-8-rtk-query-advanced
- RTK Query — Automated Re-fetching — https://redux.js.org/tutorials/essentials/part-8-rtk-query-advanced

---

## 16. Redux DevTools

### Definitions

**Core Definition:** Redux DevTools is a browser extension and store enhancer that provides time-travel debugging, action inspection, and state tree visualization for Redux applications.

**Technical Definition:** Redux DevTools is a browser extension (Chrome, Firefox, Edge) that connects to the Redux store and records every dispatched action and resulting state. It provides: **time travel** (step back and forth through state history), **action inspection** (view action types, payloads, and diffs), **state tree inspection** (explore the entire state tree), and **dispatch replay** (re-dispatch recorded actions). In Redux Toolkit, `configureStore` enables DevTools automatically in development . In legacy setups, `composeWithDevTools` from `redux-devtools-extension` is used .

**Beginner-Friendly Explanation:** Redux DevTools is a browser extension that shows you every action that happened and how it changed the state. You can step back and forth through time to see exactly what went wrong.

### Purposes

- To debug state changes by inspecting every action.
- To time-travel through state history.
- To verify that reducers are pure and updates are correct.
- To replay actions for debugging.
- To visualize the entire state tree.

### Syntax Rules and Structure

**Redux Toolkit (automatic):**

```js
const store = configureStore({
  reducer: rootReducer,
  devTools: process.env.NODE_ENV !== 'production',
});
```

**Legacy (manual):**

```js
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';

const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk))
);
```

**Syntax Rules:**

1. Install the Redux DevTools browser extension.
2. Redux Toolkit enables DevTools automatically; `devTools: false` disables it.
3. For legacy setups, wrap the store enhancer with `composeWithDevTools`.
4. Disable DevTools in production for performance.
5. Use the DevTools panel to inspect actions, state, and diffs.

**Constraints and Limitations:**

- DevTools can slow down the app if left open with large state.
- Non-serializable state breaks DevTools serialization.
- DevTools should be disabled in production.

### Annotated Code Examples

```js
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: rootReducer,
  devTools: {
    name: 'My App',
    trace: true, // Show stack traces for actions
  },
});
```

**Expected Output:** The Redux DevTools panel shows every action with a stack trace, and the state tree can be time-traveled.

**Why this output:** `configureStore` enables the DevTools enhancer with the provided options.

### Real-World Cases

- **Debugging async actions:** Viewing pending, fulfilled, and rejected actions from thunks.
- **Time-travel debugging:** Stepping through state history to find a bug.
- **Performance:** Inspecting action frequency and state diffs.
- **Testing:** Replaying actions to reproduce bugs.

### References

- Redux DevTools — GitHub — https://github.com/reduxjs/redux-devtools
- Redux Toolkit — configureStore: devTools — https://redux-toolkit.js.org/api/configureStore
- Redux — DevTools — https://redux.js.org/usage/configuring-your-store#integrating-the-devtools-extension

---

## 17. Redux Best Practices

### Definitions

**Core Definition:** Redux best practices are the guidelines from the official Redux Style Guide for writing predictable, maintainable, and performant Redux applications.

**Technical Definition:** The Redux Style Guide divides recommendations into three priority categories: **Priority A: Essential** (prevent errors; e.g., do not mutate state, reducers must not have side effects, do not put non-serializable values in state or actions), **Priority B: Strongly Recommended** (improve readability and developer experience; e.g., use Redux Toolkit, write actions as model actions, use selectors for derived data), and **Priority C: Recommended** (consistency choices; e.g., action type naming conventions, file structure). The overarching best practice is to use Redux Toolkit, which bakes in many of these rules automatically .

**Beginner-Friendly Explanation:** The Redux Style Guide tells you how to write Redux code correctly. The most important rules are: do not mutate state, keep reducers pure, and use Redux Toolkit.

### Best Practices Table

| Priority | Practice | Description |
|----------|----------|-------------|
| **A: Essential** | Do not mutate state | Mutations break re-renders and DevTools  |
| **A: Essential** | Reducers must not have side effects | No async, randomness, or external mutations  |
| **A: Essential** | Do not put non-serializable values in state/actions | Breaks DevTools and persistence |
| **B: Strongly Recommended** | Use Redux Toolkit | Handles defaults, checks, and best practices  |
| **B: Strongly Recommended** | Use `createSlice` | Avoid switch statements and manual actions  |
| **B: Strongly Recommended** | Use `createAsyncThunk` or RTK Query for async | Standardizes loading/error states |
| **B: Strongly Recommended** | Use selectors for derived data | Memoize with `createSelector`  |
| **B: Strongly Recommended** | Treat reducers as transition logic | Not random field setters  |
| **C: Recommended** | Model actions as events | Name actions for what happened |
| **C: Recommended** | Avoid setter-shaped actions | Prevents invalid combinations  |

### Annotated Code Examples

**✅ Good Practices:**

```js
// Redux Toolkit slice with Immer
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    todoAdded(state, action) {
      state.push(action.payload); // Safe mutation via Immer
    },
  },
});

// Memoized selector
const selectCompletedCount = createSelector(
  [(state) => state.todos],
  (todos) => todos.filter((t) => t.completed).length
);
```

**❌ Bad Practices:**

```js
// ❌ Mutating state outside Immer
state.todos.push(newTodo);

// ❌ Side effect in reducer
function reducer(state, action) {
  fetch('/api/log'); // Side effect!
  return state;
}

// ❌ Storing derived state
const state = { todos: [], completedCount: 0 }; // Should be derived
```

**Expected Output:** The good practices produce predictable, debuggable state. The bad practices break re-renders, DevTools, and correctness.

**Why this output:** Mutation prevents React from detecting changes; side effects make reducers impure; derived state can become stale.

### Real-World Cases

- **All Redux applications:** The Style Guide applies universally.
- **Migration:** Moving from legacy Redux to Redux Toolkit.
- **Code reviews:** Checking for mutation and side effects.

### References

- Redux — Style Guide — https://redux.js.org/style-guide/
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview
- agents-inc/skills — Redux Toolkit Best Practices — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-redux-toolkit/SKILL.md

---

## 18. Redux Pitfalls

### Definitions

**Core Definition:** Redux pitfalls are the common mistakes that cause bugs, performance problems, and maintenance headaches in Redux applications.

**Technical Definition:** The most common Redux pitfalls are: **mutating state** (the most common cause of bugs in Redux applications, affecting over 70% of Redux apps; breaks re-renders and time-travel debugging) , **side effects in reducers**, **storing derived state** (causes synchronization bugs and unnecessary renders) , **duplicated request flags** instead of a single status field , **legacy patterns** (using `createStore`, `combineReducers` manually, switch statements) , **RTK Query without middleware** , **deep nesting in entity state** , **untyped hooks** , and **mutable action objects** (violates Redux's core design) .

**Beginner-Friendly Explanation:** These are the traps that catch almost everyone using Redux. The biggest one is mutating state directly — it breaks everything.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Mutating state** | No re-renders; DevTools breaks  | Use Immer via `createSlice`; return new objects |
| **Side effects in reducers** | Impure, unpredictable state  | Use middleware for async/side effects |
| **Storing derived state** | Stale data; unnecessary renders  | Use selectors with `createSelector` |
| **Duplicated request flags** | Invalid combinations (loading + error + success) | Use a single status field  |
| **Legacy Redux patterns** | Boilerplate; no checks  | Use `configureStore` and `createSlice` |
| **RTK Query without middleware** | Caching and invalidation break  | Add `api.middleware` to the store |
| **Deep nesting in entity state** | Hard to update and select  | Normalize with `createEntityAdapter` |
| **Untyped hooks** | Lost TypeScript safety  | Create typed `useAppSelector` / `useAppDispatch` |
| **Mutating action objects** | Violates Redux principles  | Treat actions as immutable |

### Annotated Code Examples

**Pitfall 1: Mutating state**

```js
// ❌ Mutation: no re-render
state.todos.push(newTodo);

// ✅ Correct: return new state
return { ...state, todos: [...state.todos, newTodo] };

// ✅ Redux Toolkit (Immer): safe "mutation"
state.todos.push(newTodo); // Inside createSlice reducer
```

**Pitfall 2: Storing derived state**

```js
// ❌ Storing derived state
const [completedCount, setCompletedCount] = useState(0);

// ✅ Deriving with selector
const completedCount = useSelector(
  (state) => state.todos.filter((t) => t.completed).length
);
```

**Expected Output:** The correct versions produce predictable, debuggable state. The incorrect versions cause stale UI and synchronization bugs.

**Why this output:** Mutation prevents React from detecting changes; stored derived state can drift from its source.

### Real-World Cases

- **Large codebases:** Mutation bugs are the most common Redux issue.
- **Performance:** Derived state and untyped hooks cause unnecessary re-renders.
- **Migration:** Legacy patterns are the main barrier to modern Redux.

### References

- Redux — Style Guide — https://redux.js.org/style-guide/
- TrackJS — Modern Redux Debugging: Common Bugs and Solutions — https://trackjs.com
- agents-inc/skills — Redux Toolkit Anti-Patterns — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-redux-toolkit/SKILL.md

---

## References

- Redux — Three Principles — https://redux.js.org/understanding/thinking-in-redux/three-principles
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview
- Redux Toolkit — createSlice — https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit — configureStore — https://redux-toolkit.js.org/api/configureStore
- Redux Toolkit — createAsyncThunk — https://redux-toolkit.js.org/api/createAsyncThunk
- Redux Toolkit — createEntityAdapter — https://redux-toolkit.js.org/api/createEntityAdapter
- Redux Toolkit — createSelector — https://redux-toolkit.js.org/api/createSelector
- RTK Query — Overview — https://redux-toolkit.js.org/rtk-query/overview
- Redux Essentials — RTK Query Advanced Patterns — https://redux.js.org/tutorials/essentials/part-8-rtk-query-advanced
- Redux — Style Guide — https://redux.js.org/style-guide/
- Redux — Middleware — https://redux.js.org/understanding/history-and-design/middleware
- Redux Thunk — GitHub — https://github.com/reduxjs/redux-thunk
- Redux Saga — Official Documentation — https://redux-saga.js.org/
- Redux Observable — Official Documentation — https://redux-observable.js.org/
- Redux DevTools — GitHub — https://github.com/reduxjs/redux-devtools
- React Redux — useDispatch — https://react-redux.js.org/api/hooks#usedispatch
- React Redux — useSelector — https://react-redux.js.org/api/hooks#useselector
- SitePoint — Async Operations in React Redux Applications — https://www.sitepoint.com/async-operations-react-redux-applications/
- agents-inc/skills — Redux Toolkit Best Practices — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-redux-toolkit/SKILL.md
- TrackJS — Modern Redux Debugging: Common Bugs and Solutions — https://trackjs.com
- Redux — Migrating to Modern Redux — https://redux.js.org/usage/migrating-to-modern-redux