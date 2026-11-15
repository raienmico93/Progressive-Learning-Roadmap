# Redux Toolkit — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Redux Toolkit (RTK) is the official, opinionated, batteries-included toolset for Redux that eliminates boilerplate and bakes in best practices by default.

**Technical Definition:** Redux Toolkit is the official, opinionated, batteries-included toolset for efficient Redux development. It is intended to be the standard way to write Redux logic, and the Redux team strongly recommends it. RTK is a wrapper around the core Redux library that includes APIs for store setup (`configureStore`), reducer and action creation (`createSlice`, `createReducer`, `createAction`), async logic (`createAsyncThunk`), normalized state (`createEntityAdapter`), memoized selectors (`createSelector`), side-effect orchestration (`createListenerMiddleware`), and data fetching and caching (**RTK Query**). It integrates Immer so reducers can be written with "mutating" syntax that is safely converted to immutable updates, and it enables development-mode checks for accidental mutations and non-serializable values. It addresses the three most common complaints about legacy Redux: complicated store configuration, too many packages, and too much boilerplate.

**Beginner-Friendly Explanation:** Redux Toolkit is the modern, official way to use Redux. Redux itself is very powerful but requires a lot of repetitive code — you have to write action types, action creators, and reducers by hand, and it is easy to accidentally mutate state. Redux Toolkit gives you shortcuts for all of that: `createSlice` writes the actions and reducer for you, `configureStore` sets up the store correctly, `createAsyncThunk` handles loading and error states for API calls, and RTK Query handles fetching and caching entirely. It also includes Immer, so you can write simpler reducer code that looks like it mutates state but is actually safely immutable.

### Key Characteristics

- **Official and standard:** The Redux team recommends RTK for all Redux logic and actively maintains it.
- **Batteries included:** One package covers store setup, reducers, actions, async, normalization, selectors, listeners, and data fetching.
- **Immer-powered reducers:** `createSlice` and `createReducer` allow "mutating" syntax that produces immutable updates.
- **Good defaults:** `configureStore` includes thunk, DevTools, immutability checks, and serializability checks automatically.
- **Typed end-to-end:** TypeScript inference works through slices, thunks, selectors, and RTK Query hooks.
- **RTK Query:** A complete data fetching and caching layer that eliminates manual loading and error state.
- **Modern builder callbacks:** RTK 2.0 requires the builder callback pattern for `createReducer` and `extraReducers`.
- **Listener middleware:** A built-in way to run side effects in response to actions without adding thunk/saga/observable.

### Prerequisites

- JavaScript fundamentals: objects, arrays, functions, and immutability concepts.
- React function components and Hooks.
- Redux concepts: store, actions, reducers, dispatch, and selectors.
- TypeScript (strongly recommended; RTK's inference is a major benefit).
- Basic understanding of async/await and HTTP.

### Related Programming Areas

- **Redux** — the underlying state container.
- **React-Redux** — `useSelector`, `useDispatch`, and `Provider`.
- **RTK Query** — the built-in data fetching layer.
- **Reselect** — memoized selector library re-exported by RTK.
- **Immer** — the immutability library used by `createSlice`.

---

## 1. Redux Toolkit

### Definitions

**Core Definition:** Redux Toolkit is the official toolset that provides utility functions to simplify Redux store setup, reducer and action creation, async logic, and data fetching.

**Technical Definition:** Redux Toolkit is a package (`@reduxjs/toolkit`) that wraps Redux, Reselect, Redux Thunk, and Immer into a single dependency with a cohesive API. It exports `configureStore`, `createSlice`, `createReducer`, `createAction`, `createAsyncThunk`, `createEntityAdapter`, `createSelector`, `createListenerMiddleware`, `createApi`, `fetchBaseQuery`, and utility functions like `nanoid`, `isAnyOf`, and `current`. It is designed to be the standard way to write Redux logic, and it bakes in the Redux Style Guide's recommendations.

**Beginner-Friendly Explanation:** Redux Toolkit is a single package that contains everything you need to write Redux correctly, without the boilerplate.

### Purposes

- To eliminate the boilerplate of handwritten action types, action creators, and reducers.
- To provide a single dependency that includes Redux, thunk, Immer, and Reselect.
- To enforce the Redux Style Guide's best practices by default.
- To simplify async logic with `createAsyncThunk` and RTK Query.
- To provide good TypeScript inference across the entire Redux layer.
- To make store setup a one-liner with sensible defaults.

### Syntax Rules and Structure

**Installation:**

```bash
npm install @reduxjs/toolkit react-redux
```

**Package exports:**

| Export | Purpose |
|--------|---------|
| `configureStore` | Store setup with defaults |
| `createSlice` | Reducers + action creators + types |
| `createReducer` | Reducer with Immer |
| `createAction` | Action creator with `.type` and `.match` |
| `createAsyncThunk` | Async lifecycle actions |
| `createEntityAdapter` | Normalized collections |
| `createSelector` | Memoized selectors |
| `createListenerMiddleware` | Side effects in response to actions |
| `createApi`, `fetchBaseQuery` | RTK Query |

**Syntax Rules:**

1. Use `configureStore` for store creation; never use `createStore`.
2. Use `createSlice` for reducers and actions; never write switch statements by hand.
3. Use `createAsyncThunk` or RTK Query for async logic.
4. Use the builder callback pattern for `createReducer` and `extraReducers` (required in RTK 2.0).
5. Use typed hooks (`useAppSelector`, `useAppDispatch`) in TypeScript projects.

**Constraints and Limitations:**

- Immer is included, but it does not protect state that lives outside slices.
- RTK is opinionated; the API surface is larger than plain Redux.
- Some advanced patterns (e.g., custom middleware) still require understanding core Redux.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Minimal Redux Toolkit Setup

```js
// store.js
import { configureStore, createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
  },
});

export const { increment } = counterSlice.actions;
export const store = configureStore({ reducer: { counter: counterSlice.reducer } });
```

**Expected Output:** A store with a single `counter` slice and a generated `increment` action creator.

**Why this output:** `createSlice` generates the reducer, action creator, and action type. `configureStore` combines them with defaults.

#### Example 2: Feature Slice with Async Thunk

```js
// todosSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchTodos = createAsyncThunk('todos/fetchTodos', async () => {
  const res = await fetch('/api/todos');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
});

const todosSlice = createSlice({
  name: 'todos',
  initialState: { items: [], status: 'idle', error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => { state.status = 'loading'; })
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

export default todosSlice.reducer;
```

**Expected Output:** Dispatching `fetchTodos()` sets `status` to `loading`, then `succeeded` with items, or `failed` with an error.

**Why this output:** `createAsyncThunk` generates the three lifecycle actions; `extraReducers` handles each state transition with Immer-powered updates.

### Real-World Cases

- **Any new Redux application:** RTK is the standard.
- **Migrating legacy Redux:** Incrementally replace `createStore` with `configureStore` and `createSlice`.
- **Server state:** RTK Query replaces hand-written fetching logic.

### References

- Redux Toolkit — Official Documentation — https://redux-toolkit.js.org/
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview
- Redux — Migrating to Modern Redux — https://redux.js.org/usage/migrating-to-modern-redux

---

## 2. `configureStore`

### Definitions

**Core Definition:** `configureStore` is RTK's store factory that sets up a Redux store with the recommended defaults — thunk, DevTools, immutability checks, and serializability checks — in a single call.

**Technical Definition:** `configureStore({ reducer, middleware?, devTools?, preloadedState?, enhancers? })` creates a store. It automatically combines slice reducers when `reducer` is an object, includes the thunk middleware, adds development-mode checks for accidental mutations and non-serializable values, and enables the Redux DevTools Extension. Custom middleware is added with the `middleware` callback (which receives `getDefaultMiddleware`), and custom enhancers with the `enhancers` callback. It replaces the legacy `createStore` + `combineReducers` + `applyMiddleware` + `composeWithDevTools` setup.

**Beginner-Friendly Explanation:** `configureStore` creates your Redux store and sets up everything you need — async support, DevTools, and safety checks — with almost no configuration.

### Purposes

- To create a store with the recommended defaults in one call.
- To automatically combine slice reducers.
- To include thunk middleware for async logic.
- To enable development-mode immutability and serializability checks.
- To enable the Redux DevTools Extension automatically.
- To support custom middleware and enhancers when needed.
- To provide typed store, state, and dispatch for TypeScript.

### Syntax Rules and Structure

**General Syntax:**

```js
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: {
    todos: todosReducer,
    counter: counterReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(customMiddleware),
  devTools: process.env.NODE_ENV !== 'production',
  preloadedState: {},
  enhancers: (getDefaultEnhancers) =>
    getDefaultEnhancers().concat(customEnhancer),
});
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `reducer` | A single reducer function or an object of slice reducers |
| `middleware` | Callback receiving `getDefaultMiddleware`; returns the middleware array |
| `devTools` | `false` to disable, or an options object for the DevTools extension |
| `preloadedState` | Initial state, useful for SSR and tests |
| `enhancers` | Callback receiving `getDefaultEnhancers`; returns the enhancer array |

**Syntax Rules:**

1. `reducer` can be a function or an object of slice reducers; an object is automatically combined.
2. The default middleware includes thunk, immutability checks, and serializability checks.
3. Use the callback form of `middleware` to add custom middleware while preserving defaults.
4. RTK Query's middleware must be appended: `getDefaultMiddleware().concat(api.middleware)`.
5. `devTools` defaults to `true` in development.
6. Export the store and derive `RootState` and `AppDispatch` types for TypeScript.

**Constraints and Limitations:**

- The serializability check warns on non-serializable values (e.g., `Date`, `File`, `Promise`); configure the check if needed.
- Custom middleware ordering matters; the callback form gives precise control.
- Only one store per application is the Redux convention.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Store with RTK Query Middleware

```js
// store.js
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './todosSlice';
import { api } from './api';

export const store = configureStore({
  reducer: {
    todos: todosReducer,
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});
```

**Expected Output:** A store with the todos slice and the RTK Query cache, with RTK Query's middleware handling caching and invalidation.

**Why this output:** RTK Query requires both its reducer (`api.reducer`) and its middleware (`api.middleware`). `configureStore` combines them with the defaults.

#### Example 2: Typed Store for TypeScript

```ts
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './todosSlice';

export const store = configureStore({
  reducer: { todos: todosReducer },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// hooks.ts
import { useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

**Expected Output:** Fully typed `useAppSelector` and `useAppDispatch` hooks that infer state and dispatch types.

**Why this output:** `RootState` is derived from `store.getState`, and `AppDispatch` from `store.dispatch`. `.withTypes()` creates typed hooks without repeating generics at every call site.

### Real-World Cases

- **All RTK applications:** The standard store creation method.
- **SSR:** Create a new store per request with `preloadedState`.
- **Testing:** Create a fresh store per test with overridden state.

### References

- Redux Toolkit — configureStore — https://redux-toolkit.js.org/api/configureStore
- Redux Toolkit — Usage with TypeScript — https://redux-toolkit.js.org/usage/usage-with-typescript
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview

---

## 3. `createSlice`

### Definitions

**Core Definition:** `createSlice` accepts a slice name, an initial state, and an object of reducer functions, and automatically generates the slice reducer, action creators, action types, and selectors.

**Technical Definition:** `createSlice({ name, initialState, reducers, extraReducers?, selectors?, reducerPath? })` returns a slice object with `name`, `reducer`, `actions`, `caseReducers`, `getInitialState`, `selectors`, and `getSelectors`. The `name` is used as the prefix for generated action types (e.g., `'todos/todoAdded'`). Reducers can be written with "mutating" syntax thanks to Immer, which safely converts the mutations into immutable updates. `extraReducers` handles actions defined outside the slice (e.g., from `createAsyncThunk`). In RTK 2.0, `reducerPath` is optional and defaults to `name`.

**Beginner-Friendly Explanation:** `createSlice` creates a "slice" of your Redux state — one feature's reducer, actions, and selectors — all in one place. You write the reducer functions, and it generates everything else.

### Purposes

- To eliminate handwritten action types and action creators.
- To write reducers with simpler "mutating" syntax via Immer.
- To group related state, reducers, and selectors in one module.
- To handle external actions (from thunks or other slices) with `extraReducers`.
- To generate typed selectors with the `selectors` field (RTK 2.0+).
- To provide a single file per feature that is easy to test and reason about.

### Syntax Rules and Structure

**General Syntax:**

```js
import { createSlice } from '@reduxjs/toolkit';

const todosSlice = createSlice({
  name: 'todos',
  initialState: { items: [], status: 'idle' },
  reducers: {
    todoAdded(state, action) {
      state.items.push({ id: action.payload.id, text: action.payload.text, completed: false });
    },
    todoToggled(state, action) {
      const todo = state.items.find((t) => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },
  },
  extraReducers: (builder) => {
    builder.addCase(fetchTodos.fulfilled, (state, action) => {
      state.items = action.payload;
    });
  },
  selectors: {
    selectTodos: (state) => state.items,
    selectCompleted: (state) => state.items.filter((t) => t.completed),
  },
});

export const { todoAdded, todoToggled } = todosSlice.actions;
export const { selectTodos, selectCompleted } = todosSlice.selectors;
export default todosSlice.reducer;
```

**Component Breakdown:**

| Field | Description |
|-------|-------------|
| `name` | Prefix for generated action types |
| `initialState` | Starting state for the slice |
| `reducers` | Object of reducer functions; each generates an action creator |
| `extraReducers` | Builder callback for external actions |
| `selectors` | (RTK 2.0+) Object of selectors that receive the slice state |

**Syntax Rules:**

1. `name` becomes the action type prefix: `todoAdded` → `'todos/todoAdded'`.
2. Reducers receive `(state, action)`; Immer allows direct mutation of the draft.
3. `extraReducers` must use the builder callback in RTK 2.0.
4. Selectors in `selectors` receive the **slice state**, not the root state.
5. Export action creators from `slice.actions` and the reducer as default.
6. The slice reducer is used in `configureStore` under its key.

**Constraints and Limitations:**

- `createSlice` assumes Immer; do not manually mutate state outside the slice.
- Selectors defined in `selectors` receive the slice state; use `getSelectors` for root-state selectors.
- Action type collisions are possible if two slices share a `name`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Slice with Immer and Generated Actions

```js
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment(state) { state.value += 1; },
    addBy(state, action) { state.value += action.payload; },
  },
});

console.log(counterSlice.actions.increment()); // { type: 'counter/increment' }
console.log(counterSlice.actions.addBy(5));     // { type: 'counter/addBy', payload: 5 }
console.log(counterSlice.reducer(undefined, { type: '@@INIT' })); // { value: 0 }
```

**Expected Output:** `increment()` returns `{ type: 'counter/increment' }`; the reducer returns `{ value: 0 }` for its initial state.

**Why this output:** `createSlice` generates action creators from the reducer names and uses `initialState` for the first call.

#### Example 2: Slice with Async Thunk and Selectors

```js
const usersSlice = createSlice({
  name: 'users',
  initialState: { entities: [], status: 'idle' },
  reducers: {
    userAdded(state, action) { state.entities.push(action.payload); },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => { state.status = 'loading'; })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.entities = action.payload;
      })
      .addCase(fetchUsers.rejected, (state) => { state.status = 'failed'; });
  },
  selectors: {
    selectUsers: (state) => state.entities,
    selectStatus: (state) => state.status,
  },
});
```

**Expected Output:** The slice handles `fetchUsers` lifecycle actions and exposes `selectUsers` and `selectStatus` selectors that receive the slice state.

**Why this output:** `extraReducers` handles external actions; the `selectors` field generates selectors scoped to the slice.

### Real-World Cases

- **Feature slices:** One slice per feature (todos, users, cart).
- **Entity slices:** Combined with `createEntityAdapter`.
- **Async slices:** `extraReducers` for `createAsyncThunk` actions.

### References

- Redux Toolkit — createSlice — https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit — Usage with TypeScript: createSlice — https://redux-toolkit.js.org/usage/usage-with-typescript#createslice
- Redux Essentials — Writing Slices — https://redux.js.org/tutorials/essentials/part-2-app-structure

---

## 4. `createReducer`

### Definitions

**Core Definition:** `createReducer` is a utility that lets you write a reducer as a lookup table of case reducers rather than a switch statement, using Immer for immutable updates.

**Technical Definition:** `createReducer(initialState, builderCallback)` accepts an initial state and a builder callback that defines how state changes in response to specific action types. The builder provides `addCase`, `addMatcher`, and `addDefaultCase`. In RTK 2.0, the builder callback pattern is **required**; the earlier object form was removed. `createReducer` uses Immer, so case reducers can mutate the draft state safely. It is used internally by `createSlice` and is useful when you need a reducer outside a slice or with more complex matching logic.

**Beginner-Friendly Explanation:** `createReducer` is how you write a reducer without a giant `switch` statement. You register handlers for specific actions, and RTK uses Immer to handle immutability.

### Purposes

- To define reducers as a table of action handlers instead of a switch.
- To use Immer for immutable updates without manual spreading.
- To support pattern matching with `addMatcher` for cross-cutting action groups.
- To define standalone reducers outside a slice.
- To share case reducers across multiple slices.

### Syntax Rules and Structure

**General Syntax:**

```js
import { createReducer } from '@reduxjs/toolkit';

const initialState = { value: 0 };

const counterReducer = createReducer(initialState, (builder) => {
  builder
    .addCase(increment, (state, action) => { state.value += 1; })
    .addCase(decrement, (state, action) => { state.value -= 1; })
    .addCase(addBy, (state, action) => { state.value += action.payload; })
    .addDefaultCase((state, action) => {});
});
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `addCase(actionCreator, reducer)` | Handles one specific action type |
| `addMatcher(predicate, reducer)` | Handles any action matching the predicate |
| `addDefaultCase(reducer)` | Handles any action not matched above |

**Syntax Rules:**

1. The builder callback is required in RTK 2.0.
2. `addCase` accepts an action creator (from `createAction` or `createSlice`) or an action type string.
3. `addMatcher` receives `(action) => boolean` and is useful for matching action groups.
4. `addDefaultCase` runs for unmatched actions.
5. Case reducers can mutate the draft state thanks to Immer.

**Constraints and Limitations:**

- Case reducers must not return `undefined`; return nothing to keep the mutated draft.
- Multiple `addMatcher` handlers that match the same action run in order; later handlers receive the state produced by earlier ones.
- The object form (`{ [actionType]: reducer }`) is removed in RTK 2.0.

### Annotated Code Examples

```js
import { createReducer, createAction, isAnyOf } from '@reduxjs/toolkit';

const increment = createAction('counter/increment');
const decrement = createAction('counter/decrement');

const counterReducer = createReducer({ value: 0 }, (builder) => {
  builder
    .addCase(increment, (state) => { state.value += 1; })
    .addCase(decrement, (state) => { state.value -= 1; })
    .addMatcher(
      isAnyOf(increment, decrement),
      (state) => { state.lastUpdated = Date.now(); }
    );
});
```

**Expected Output:** `increment` updates both `value` and `lastUpdated`; `decrement` updates both. The matcher runs after the specific case.

**Why this output:** `addCase` handles the specific action; `addMatcher` with `isAnyOf` handles the group. Both run for matching actions, in registration order.

### Real-World Cases

- **Standalone reducers:** Reducers used outside slices or in `combineReducers` directly.
- **Shared case reducers:** Reducer logic reused across multiple slices.
- **Cross-cutting updates:** `addMatcher` for groups of actions (e.g., all `todos/*` actions).

### References

- Redux Toolkit — createReducer — https://redux-toolkit.js.org/api/createReducer
- Redux Toolkit — createAction: isAnyOf — https://redux-toolkit.js.org/api/createAction#isanyof

---

## 5. `createAction`

### Definitions

**Core Definition:** `createAction` generates an action creator function for a given action type, with the `type` string attached and a `.match` method for type checking.

**Technical Definition:** `createAction(type, prepareCallback?)` returns an action creator. Calling it with no arguments returns `{ type }`; calling it with a payload returns `{ type, payload }`; a `prepare` callback can customise the payload and add a `meta` field. The returned action creator has `.type` (the action type string) and `.match(action)` (returns `true` if the action has that type). `createSlice` uses `createAction` internally to generate its action creators.

**Beginner-Friendly Explanation:** `createAction` creates a function that builds an action object for you. Instead of writing `{ type: 'todos/todoAdded', payload }` every time, you call `todoAdded(payload)`.

### Purposes

- To generate action creators without hand-writing action objects.
- To attach the action `type` string to the creator for use in reducers and matchers.
- To provide a `.match` method for type-safe action checking.
- To use a `prepare` callback for custom payload shapes and metadata.
- To support reducers defined with `createReducer` and `extraReducers`.

### Syntax Rules and Structure

**General Syntax:**

```js
import { createAction } from '@reduxjs/toolkit';

const increment = createAction('counter/increment');
const addBy = createAction('counter/addBy', (amount) => ({
  payload: amount,
  meta: { timestamp: Date.now() },
}));
```

**Component Breakdown:**

| Property / Argument | Description |
|----------------------|-------------|
| `type` | The action type string |
| `prepareCallback` | Optional function returning `{ payload, meta?, error? }` |
| `.type` | The action type string, accessible on the creator |
| `.match(action)` | Returns `true` if the action matches the type |

**Syntax Rules:**

1. The `type` string should be namespaced (e.g., `'feature/event'`).
2. Calling the creator with no arguments returns `{ type }`.
3. Calling with one argument returns `{ type, payload }`.
4. A `prepare` callback overrides the payload shape and can add `meta`.
5. `.match(action)` is useful for `addMatcher` and type guards.

**Constraints and Limitations:**

- The payload must be serializable.
- `createAction` does not create a reducer; pair it with `createReducer`.
- `.match` checks only the `type` field, not the payload shape.

### Annotated Code Examples

```js
const todoAdded = createAction('todos/todoAdded', (text) => ({
  payload: { id: nanoid(), text, completed: false },
}));

todoAdded('Learn Redux');
// { type: 'todos/todoAdded', payload: { id: '...', text: 'Learn Redux', completed: false } }

todoAdded.type;                 // 'todos/todoAdded'
todoAdded.match({ type: 'todos/todoAdded' }); // true
```

**Expected Output:** The action creator generates an action with a generated ID, the text, and `completed: false`. `.type` and `.match` are available on the creator.

**Why this output:** The prepare callback builds the payload; `createAction` attaches `.type` and `.match`.

### Real-World Cases

- **Standalone actions:** Actions used outside slices with `createReducer`.
- **Shared actions:** Action types reused across slices.
- **Matchers:** Grouping actions with `isAnyOf` in `addMatcher`.

### References

- Redux Toolkit — createAction — https://redux-toolkit.js.org/api/createAction
- Redux Essentials — Actions — https://redux.js.org/tutorials/essentials/part-2-app-structure

---

## 6. `createAsyncThunk`

### Definitions

**Core Definition:** `createAsyncThunk` generates a thunk that performs async work and dispatches `pending`, `fulfilled`, and `rejected` lifecycle actions.

**Technical Definition:** `createAsyncThunk(typePrefix, payloadCreator, options?)` returns a thunk action creator. The `payloadCreator` is an async function receiving `(arg, thunkAPI)` that returns the fulfilled value or rejects with an error. The generated thunk dispatches `typePrefix/pending` when called, `typePrefix/fulfilled` on success (with `action.payload`), and `typePrefix/rejected` on failure (with `action.error` or `action.payload` if `rejectWithValue` is used). The thunk also exposes `.pending`, `.fulfilled`, and `.rejected` action creators for use in `extraReducers`. It supports request cancellation via `signal` and conditional execution via `condition`.

**Beginner-Friendly Explanation:** `createAsyncThunk` handles the three states of an API call — loading, success, and failure — automatically. You give it the async function, and it generates the actions.

### Purposes

- To standardise async action lifecycles (pending, fulfilled, rejected).
- To avoid hand-writing loading, success, and error actions.
- To handle errors and cancellations consistently.
- To integrate with `extraReducers` for state updates.
- To access `dispatch`, `getState`, and `signal` inside the async function.
- To support conditional execution and request deduplication.

### Syntax Rules and Structure

**General Syntax:**

```js
import { createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'users/fetchUser',
  async (userId, { rejectWithValue, getState, dispatch, signal }) => {
    try {
      const res = await fetch(`/api/users/${userId}`, { signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `typePrefix` | Base action type (e.g., `'users/fetchUser'`) |
| `payloadCreator` | Async function `(arg, thunkAPI) => Promise` |
| `arg` | The argument passed to the thunk |
| `thunkAPI` | `{ dispatch, getState, rejectWithValue, signal, extra }` |
| `.pending` / `.fulfilled` / `.rejected` | Lifecycle action creators |

**Syntax Rules:**

1. `typePrefix` is the base type; lifecycle types are appended (`/pending`, etc.).
2. `payloadCreator` must return a Promise.
3. Use `rejectWithValue` to return a custom rejection payload instead of a generic error.
4. Use `signal` for cancellation and `getState` for accessing state.
5. Handle the three lifecycle actions in `extraReducers` with the builder callback.

**Constraints and Limitations:**

- The payload must be serializable.
- `rejectWithValue` must be called exactly once; returning both a value and a rejection is invalid.
- `condition` can abort the request, but it is not a replacement for cancellation inside the thunk.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Fetch Thunk

```js
export const fetchTodos = createAsyncThunk('todos/fetchTodos', async () => {
  const res = await fetch('/api/todos');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
});

// In a slice
extraReducers: (builder) => {
  builder
    .addCase(fetchTodos.pending, (state) => { state.status = 'loading'; })
    .addCase(fetchTodos.fulfilled, (state, action) => {
      state.status = 'succeeded';
      state.items = action.payload;
    })
    .addCase(fetchTodos.rejected, (state, action) => {
      state.status = 'failed';
      state.error = action.error.message;
    });
}
```

**Expected Output:** Dispatching `fetchTodos()` transitions status from `idle` → `loading` → `succeeded` (with items) or `failed` (with error).

**Why this output:** The thunk dispatches `pending` immediately, then `fulfilled` with the parsed data or `rejected` with the error.

#### Example 2: Thunk with `rejectWithValue` and Cancellation

```js
export const searchUsers = createAsyncThunk(
  'users/search',
  async (query, { signal, rejectWithValue }) => {
    try {
      const res = await fetch(`/api/users?q=${encodeURIComponent(query)}`, { signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (err) {
      if (err.name === 'AbortError') throw err;
      return rejectWithValue({ message: err.message, query });
    }
  },
  {
    condition: (query, { getState }) => {
      return query.length >= 3; // Skip short queries
    },
  }
);
```

**Expected Output:** Short queries dispatch nothing; long queries dispatch pending/fulfilled, or rejected with `{ message, query }` on failure. Aborts re-throw without a rejection action.

**Why this output:** `condition` skips the thunk for short queries. `rejectWithValue` provides structured error data. Aborts are re-thrown so the thunk resolves as a cancellation.

### Real-World Cases

- **API fetching:** Loading users, posts, and products.
- **Form submission:** Sending data with loading and error states.
- **Authentication:** Login and logout flows.
- **Search:** Debounced or conditional requests.

### References

- Redux Toolkit — createAsyncThunk — https://redux-toolkit.js.org/api/createAsyncThunk
- Redux Essentials — Async Logic and Data Fetching — https://redux.js.org/tutorials/essentials/part-5-async-logic

---

## 7. `createEntityAdapter`

### Definitions

**Core Definition:** `createEntityAdapter` generates reusable reducers and selectors for managing normalized collections of items in the store.

**Technical Definition:** `createEntityAdapter({ selectId?, sortComparer? })` returns an adapter with methods for managing a normalized state shape `{ ids: [], entities: {} }`: `addOne`, `addMany`, `setAll`, `upsertOne`, `upsertMany`, `updateOne`, `updateMany`, `removeOne`, `removeMany`, `removeAll`, and `getInitialState`. It also provides `getSelectors(selectSliceState)` which returns `selectAll`, `selectById`, `selectIds`, and `selectTotal`, and (in RTK 2.0) `getSelectors` can be used with the slice's `selectors` field. `sortComparer` sorts the `ids` array; `selectId` customises the ID field (default `id`).

**Beginner-Friendly Explanation:** `createEntityAdapter` helps you store collections of items (like users or todos) efficiently. You get ready-made functions for adding, updating, and selecting items.

### Purposes

- To manage normalized collections efficiently.
- To avoid writing repetitive add/update/remove reducers.
- To provide memoized selectors for entities.
- To support sorting and custom ID fields.
- To keep entity state flat and easy to update.
- To integrate with `createSlice` and RTK Query.

### Syntax Rules and Structure

**General Syntax:**

```js
import { createEntityAdapter, createSlice } from '@reduxjs/toolkit';

const usersAdapter = createEntityAdapter({
  selectId: (user) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name),
});

const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState(),
  reducers: {
    userAdded: usersAdapter.addOne,
    usersReceived: usersAdapter.setAll,
    userUpdated: usersAdapter.updateOne,
    userRemoved: usersAdapter.removeOne,
  },
});

export const { selectAll, selectById, selectIds, selectTotal } =
  usersAdapter.getSelectors((state) => state.users);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `selectId` | Function returning the unique ID (default `id`) |
| `sortComparer` | Function sorting the `ids` array |
| `getInitialState(additionalState?)` | Returns `{ ids: [], entities: {} }` plus additional fields |
| `addOne` / `addMany` / `setAll` | Add or replace entities |
| `upsertOne` / `upsertMany` | Insert or update |
| `updateOne` / `updateMany` | Update fields of existing entities |
| `removeOne` / `removeMany` / `removeAll` | Remove entities |
| `getSelectors(selectSliceState)` | Returns memoized selectors |

**Syntax Rules:**

1. `selectId` defaults to `(entity) => entity.id`.
2. `sortComparer` sorts the `ids` array; it is not applied to the `entities` object.
3. `getInitialState()` can accept additional state fields.
4. Adapter methods can be used directly as case reducers.
5. `getSelectors` receives a selector for the slice state, not the root state.

**Constraints and Limitations:**

- Deeply nested entity state complicates updates; normalize instead.
- The normalized shape (`ids` + `entities`) is opinionated.
- `sortComparer` must be deterministic and stable.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Entity Slice

```js
const booksAdapter = createEntityAdapter();
const booksSlice = createSlice({
  name: 'books',
  initialState: booksAdapter.getInitialState(),
  reducers: {
    bookAdded: booksAdapter.addOne,
    booksReceived: booksAdapter.setAll,
    bookRemoved: booksAdapter.removeOne,
  },
});

// State shape
// { ids: [1, 2], entities: { 1: { id: 1, title: '...' }, 2: {...} } }
```

**Expected Output:** `bookAdded({ id: 1, title: '...' })` adds the book; `bookRemoved(1)` removes it.

**Why this output:** The adapter generates reducers that maintain the normalized shape.

#### Example 2: Entity Slice with Sorting and Selectors

```js
const usersAdapter = createEntityAdapter({
  sortComparer: (a, b) => a.name.localeCompare(b.name),
});

const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState(),
  reducers: {
    userAdded: usersAdapter.addOne,
    usersReceived: usersAdapter.setAll,
  },
});

export const { selectAll, selectById, selectTotal } =
  usersAdapter.getSelectors((state) => state.users);
```

**Expected Output:** `selectAll` returns users sorted by name; `selectById` returns a single user; `selectTotal` returns the count.

**Why this output:** `sortComparer` sorts the `ids` array; the generated selectors use it.

### Real-World Cases

- **User lists:** Normalized users by ID.
- **Product catalogs:** Products with sorting.
- **Todo lists:** Todos with add/update/remove.

### References

- Redux Toolkit — createEntityAdapter — https://redux-toolkit.js.org/api/createEntityAdapter
- Redux Essentials — Normalizing State — https://redux.js.org/tutorials/essentials/part-6-performance-normalization

---

## 8. `createSelector`

### Definitions

**Core Definition:** `createSelector` is RTK's re-export of Reselect's memoized selector creator, used to compute derived data efficiently.

**Technical Definition:** `createSelector(inputSelectors, resultFunc)` creates a memoized selector. It caches the last inputs and output; if the inputs are unchanged (by reference), it returns the cached output instead of recomputing. Reselect 5 (included with RTK 2.0) also accepts an options object with `memoize` and `argsMemoize` for custom memoization functions, and adds development-mode checks for common mistakes. Selectors can be composed: one selector can use another as an input.

**Beginner-Friendly Explanation:** `createSelector` remembers the last result of a computation. If the inputs are the same, it returns the cached result instead of recomputing.

### Purposes

- To memoize derived data computations.
- To prevent unnecessary re-renders in React.
- To compose complex selectors from simpler ones.
- To improve performance for expensive filtering or aggregation.
- To centralize derived-data logic.

### Syntax Rules and Structure

**General Syntax:**

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

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `inputSelectors` | Array of selectors whose results are passed to the result function |
| `resultFunc` | Function receiving the input results and returning the derived value |
| Cache | Stores the last inputs and output; recomputes only when inputs change |

**Syntax Rules:**

1. Input selectors receive the same arguments as the memoized selector.
2. The result function receives the input selectors' results as arguments.
3. Inputs are compared by reference; stable references are required for memoization.
4. Selectors can be composed by using one as an input to another.
5. Reselect 5 (RTK 2.0) accepts a development-mode options object for custom memoization.

**Constraints and Limitations:**

- Input selectors must return stable references; new objects on every call defeat memoization.
- Do not store selector results back into Redux state.
- Memoization is per-selector-instance; creating selectors inside components defeats the cache.

### Annotated Code Examples

```js
const selectTodos = (state) => state.todos;

const selectCompletedCount = createSelector(
  [selectTodos],
  (todos) => todos.filter((t) => t.completed).length
);

const selectHasCompleted = createSelector(
  [selectCompletedCount],
  (count) => count > 0
);
```

**Expected Output:** `selectCompletedCount` recomputes only when `todos` changes; `selectHasCompleted` recomputes only when the count changes.

**Why this output:** Each selector caches its last inputs and output. Composing selectors propagates memoization.

### Real-World Cases

- **Filtered lists:** Filtering todos by status.
- **Aggregations:** Counting completed items.
- **Derived entities:** Joining normalized data.
- **Performance:** Preventing re-renders with memoized selectors.

### References

- Redux Toolkit — createSelector — https://redux-toolkit.js.org/api/createSelector
- Reselect — GitHub — https://github.com/reduxjs/reselect
- Redux Essentials — Memoized Selectors — https://redux.js.org/tutorials/essentials/part-6-performance-normalization

---

## 9. `createListenerMiddleware`

### Definitions

**Core Definition:** `createListenerMiddleware` creates a middleware that lets you run side effects in response to dispatched actions, with access to `dispatch`, `getState`, cancellation, and delays.

**Technical Definition:** `createListenerMiddleware()` returns a middleware with `startListening`, `addListener`, `stopListening`, `clearListeners`, and `middleware`. `startListening({ actionCreator | matcher | predicate, effect })` registers a listener. The `effect` callback receives `(action, listenerApi)`, where `listenerApi` provides `dispatch`, `getState`, `getOriginalState`, `cancelActiveListeners`, `cancel`, `delay`, `pause`, `resume`, `signal`, `fork`, `condition`, `take`, `subscribe`, `unsubscribe`, and `extra`. Listeners run **after** the reducer processes the action, so they see the updated state. `withTypes()` creates typed versions of `startListening` and `addListener` for TypeScript.

**Beginner-Friendly Explanation:** `createListenerMiddleware` lets you run code in response to specific actions — like saving to `localStorage` after a todo is added. It is a lightweight alternative to thunks and sagas for side effects.

### Purposes

- To run side effects in response to actions without adding thunk/saga/observable.
- To access `dispatch` and `getState` inside a listener.
- To cancel, delay, or debounce work inside a listener.
- To coordinate multiple actions with `take` and `condition`.
- To keep side-effect logic colocated with slices.
- To provide typed listeners in TypeScript.

### Syntax Rules and Structure

**General Syntax:**

```js
import { createListenerMiddleware, isAnyOf } from '@reduxjs/toolkit';

export const listenerMiddleware = createListenerMiddleware();

export const startAppListening = listenerMiddleware.startListening.withTypes();

startAppListening({
  matcher: isAnyOf(todoAdded, todoToggled),
  effect: async (action, listenerApi) => {
    listenerApi.cancelActiveListeners();
    await listenerApi.delay(500);
    const state = listenerApi.getState();
    localStorage.setItem('todos', JSON.stringify(state.todos));
  },
});
```

**Component Breakdown:**

| Field / Method | Description |
|----------------|-------------|
| `actionCreator` | Listen for a specific action creator |
| `matcher` | Listen for actions matching a predicate |
| `predicate` | Listen for actions matching a function |
| `effect(action, listenerApi)` | The side effect to run |
| `listenerApi.dispatch` | Dispatch additional actions |
| `listenerApi.getState` | Read the current state (after the reducer) |
| `listenerApi.cancelActiveListeners` | Cancel earlier running instances |
| `listenerApi.delay(ms)` | Pause the listener |
| `listenerApi.signal` | Abort signal for cancellation |
| `listenerApi.fork` | Start a child task |
| `listenerApi.take` | Wait for another action |

**Syntax Rules:**

1. Register the middleware in `configureStore` with `prepend` or `concat`.
2. Use `actionCreator`, `matcher`, or `predicate` to select actions.
3. Listeners run after the reducer, so `getState` reflects the new state.
4. `cancelActiveListeners()` stops earlier runs of the same listener (useful for debouncing).
5. `withTypes()` creates typed `startListening` and `addListener`.
6. Use `listenerApi.signal` to abort fetch requests.

**Constraints and Limitations:**

- Listeners do not replace reducers; they run after the state has changed.
- Overusing listeners scatters side-effect logic; prefer thunks or RTK Query for data fetching.
- Listener cancellation semantics must be understood to avoid race conditions.

### Annotated Code Examples

```js
import { createListenerMiddleware, isAnyOf } from '@reduxjs/toolkit';
import { todoAdded, todoToggled } from './todosSlice';

export const listenerMiddleware = createListenerMiddleware();

listenerMiddleware.startListening({
  matcher: isAnyOf(todoAdded, todoToggled),
  effect: async (action, listenerApi) => {
    listenerApi.cancelActiveListeners();
    await listenerApi.delay(500);
    const todos = listenerApi.getState().todos;
    localStorage.setItem('todos', JSON.stringify(todos));
  },
});
```

**Expected Output:** After any `todoAdded` or `todoToggled`, the todos are saved to `localStorage` 500ms after the last action, with earlier pending saves cancelled.

**Why this output:** `isAnyOf` matches both actions; `cancelActiveListeners` and `delay` debounce the save; `getState` reads the updated todos.

### Real-World Cases

- **Persistence:** Saving state to `localStorage` after actions.
- **Analytics:** Sending events after specific actions.
- **Debounced side effects:** Saving drafts after typing stops.
- **Cross-slice coordination:** Reacting to one slice's action in another slice.

### References

- Redux Toolkit — createListenerMiddleware — https://redux-toolkit.js.org/api/createListenerMiddleware
- Redux Toolkit — Listener Middleware Usage — https://redux-toolkit.js.org/usage/usage-guide

---

## 10. RTK Query

### Definitions

**Core Definition:** RTK Query is a powerful data fetching and caching layer built into Redux Toolkit that eliminates the need to hand-write loading state, caching, and data fetching logic.

**Technical Definition:** RTK Query is a data fetching and caching layer built for Redux. You define API endpoints with `createApi` and a `baseQuery` (typically `fetchBaseQuery`), and it generates reducers, middleware, and React hooks for each endpoint. It manages caching, deduplication, polling, and cache invalidation through a **tag** system: queries declare `providesTags`, mutations declare `invalidatesTags`, and affected queries refetch automatically. It supports automatic caching (`keepUnusedDataFor`, default 60 seconds), `refetchOnMountOrArgChange`, `refetchOnFocus`, `refetchOnReconnect`, optimistic updates (`onQueryStarted` with `updateQueryData`), and streaming updates (`onCacheEntryAdded`).

**Beginner-Friendly Explanation:** RTK Query handles data fetching for you. You define your API endpoints, and it gives you hooks that fetch, cache, and update data automatically.

### Purposes

- To eliminate manual loading and error state management.
- To automatically cache and deduplicate requests.
- To automatically refetch data after mutations via tags.
- To generate typed React hooks for each endpoint.
- To support optimistic updates and streaming updates.
- To replace hand-written `createAsyncThunk` + slice logic for server state.
- To provide a consistent API for REST and GraphQL endpoints.

### Syntax Rules and Structure

**General Syntax:**

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
    getPost: builder.query({
      query: (id) => `/posts/${id}`,
      providesTags: (result, error, id) => [{ type: 'Post', id }],
    }),
    addPost: builder.mutation({
      query: (post) => ({ url: '/posts', method: 'POST', body: post }),
      invalidatesTags: ['Post'],
    }),
    updatePost: builder.mutation({
      query: (post) => ({ url: `/posts/${post.id}`, method: 'PATCH', body: post }),
      invalidatesTags: (result, error, post) => [{ type: 'Post', id: post.id }],
    }),
  }),
});

export const {
  useGetPostsQuery,
  useGetPostQuery,
  useAddPostMutation,
  useUpdatePostMutation,
} = api;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `reducerPath` | Key for the API reducer in the store |
| `baseQuery` | The base query function (`fetchBaseQuery` or custom) |
| `tagTypes` | Declares the tag categories for invalidation |
| `builder.query` | Defines a GET endpoint; generates a `useXQuery` hook |
| `builder.mutation` | Defines a write endpoint; generates a `useXMutation` hook |
| `providesTags` | Declares which tags a query provides |
| `invalidatesTags` | Declares which tags a mutation invalidates |

**Syntax Rules:**

1. Add `[api.reducerPath]: api.reducer` to the store's reducers.
2. Add `api.middleware` to the store's middleware.
3. `providesTags` on queries and `invalidatesTags` on mutations drive automatic refetching.
4. Tags can be strings (`'Post'`) or objects with `type` and `id` for per-entity invalidation.
5. Generated hooks return `{ data, isLoading, isFetching, isError, error, refetch }`.
6. Use `keepUnusedDataFor` (default 60) to control cache retention.
7. Use `onQueryStarted` for optimistic updates with `updateQueryData` and `patchResult.undo()` for rollback.

**Constraints and Limitations:**

- Requires both the API reducer and middleware in the store; omitting either breaks caching.
- Cache invalidation relies on correct tag configuration.
- Overkill for very simple, one-off fetches.
- Does not support WebSocket subscriptions directly; use `onCacheEntryAdded` for streaming.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Query and Mutation with Tags

```js
export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['User'],
  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => '/users',
      providesTags: ['User'],
    }),
    addUser: builder.mutation({
      query: (user) => ({ url: '/users', method: 'POST', body: user }),
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

**Expected Output:** `useGetUsersQuery` fetches and caches the list. After `addUser` succeeds, `invalidatesTags: ['User']` refetches the list automatically.

**Why this output:** The tag system connects the mutation to the query. Invalidating `'User'` marks the query as stale and refetches it.

#### Example 2: Optimistic Update with Rollback

```js
updatePost: builder.mutation({
  query: (post) => ({ url: `/posts/${post.id}`, method: 'PATCH', body: post }),
  async onQueryStarted(post, { dispatch, queryFulfilled }) {
    const patchResult = dispatch(
      api.util.updateQueryData('getPost', post.id, (draft) => {
        Object.assign(draft, post);
      })
    );
    try {
      await queryFulfilled;
    } catch {
      patchResult.undo();
    }
  },
}),
```

**Expected Output:** The post updates instantly in the cache; if the request fails, the change is rolled back.

**Why this output:** `updateQueryData` applies the optimistic change; `patchResult.undo()` reverses it on failure.

### Real-World Cases

- **CRUD applications:** Lists, details, create, update, delete.
- **Dashboards:** Cached stats with invalidation.
- **E-commerce:** Product catalogs with mutation-driven refresh.
- **Streaming:** `onCacheEntryAdded` for WebSocket updates.

### References

- RTK Query — Overview — https://redux-toolkit.js.org/rtk-query/overview
- RTK Query — Queries — https://redux-toolkit.js.org/rtk-query/usage/queries
- RTK Query — Mutations — https://redux-toolkit.js.org/rtk-query/usage/mutations
- RTK Query — Automated Re-fetching — https://redux-toolkit.js.org/rtk-query/usage/automated-refetching
- Redux Essentials — RTK Query Advanced Patterns — https://redux.js.org/tutorials/essentials/part-8-rtk-query-advanced

---

## 11. Redux Toolkit Best Practices

### Definitions

**Core Definition:** Redux Toolkit best practices are the guidelines from the official Redux Style Guide for writing predictable, maintainable, and performant Redux applications with RTK.

**Technical Definition:** The Redux Style Guide divides recommendations into three priority categories: **Priority A: Essential** (prevent errors; e.g., do not mutate state, reducers must not have side effects, do not put non-serializable values in state or actions), **Priority B: Strongly Recommended** (improve readability and developer experience; e.g., use Redux Toolkit, use `createSlice`, use `createAsyncThunk` or RTK Query for async, use selectors for derived data), and **Priority C: Recommended** (consistency choices; e.g., action type naming, file structure). RTK bakes in many of these rules automatically, but explicit discipline is still required for the rest.

**Beginner-Friendly Explanation:** The Redux Style Guide tells you how to write Redux correctly. The most important rules are: do not mutate state, keep reducers pure, use Redux Toolkit, and use selectors for derived data.

### Best Practices Table

| Priority | Practice | Description |
|----------|----------|-------------|
| **A: Essential** | Do not mutate state | Mutations break re-renders and DevTools |
| **A: Essential** | Reducers must not have side effects | No async, randomness, or external mutations |
| **A: Essential** | Do not put non-serializable values in state/actions | Breaks DevTools and persistence |
| **A: Essential** | Only one store per app | One source of truth |
| **B: Strongly Recommended** | Use Redux Toolkit | Handles defaults, checks, and best practices |
| **B: Strongly Recommended** | Use `createSlice` | Avoid switch statements and manual actions |
| **B: Strongly Recommended** | Use `createAsyncThunk` or RTK Query | Standardizes loading/error states |
| **B: Strongly Recommended** | Use selectors for derived data | Memoize with `createSelector` |
| **B: Strongly Recommended** | Model actions as events | Name actions for what happened |
| **B: Strongly Recommended** | Avoid setter-shaped actions | Prevents invalid combinations |
| **B: Strongly Recommended** | Normalize state with `createEntityAdapter` | Flat, ID-keyed collections |
| **B: Strongly Recommended** | Use RTK Query for server state | Do not duplicate server data in slices |
| **C: Recommended** | Use typed hooks | `useAppSelector` / `useAppDispatch` |
| **C: Recommended** | Colocate slice files by feature | Feature folders |
| **C: Recommended** | Use `createListenerMiddleware` for side effects | Debounced persistence, analytics |

### Annotated Code Examples

**✅ Good Practices:**

```ts
// Feature slice with Immer, async thunk, entity adapter, and selectors
const usersAdapter = createEntityAdapter<User>();

const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState({ status: 'idle' }),
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => { state.status = 'loading'; })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        usersAdapter.setAll(state, action.payload);
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
  selectors: {
    selectAllUsers: usersAdapter.getSelectors().selectAll,
    selectStatus: (state) => state.status,
  },
});
```

**❌ Bad Practices:**

```ts
// ❌ Mutating state outside Immer
state.users.push(newUser);

// ❌ Side effect in reducer
function reducer(state, action) {
  fetch('/api/log'); // Side effect!
  return state;
}

// ❌ Storing derived state
const state = { users: [], userCount: 0 };

// ❌ RTK Query without middleware
configureStore({ reducer: { [api.reducerPath]: api.reducer } }); // Missing api.middleware

// ❌ Legacy createStore
import { createStore } from 'redux'; // Use configureStore
```

**Expected Output:** The good practices produce predictable, debuggable, performant state. The bad practices break re-renders, DevTools, caching, and correctness.

**Why this output:** Mutation prevents React from detecting changes; side effects make reducers impure; derived state goes stale; missing middleware breaks RTK Query; `createStore` lacks defaults and checks.

### Real-World Cases

- **All RTK applications:** The Style Guide applies universally.
- **Migration:** Moving from legacy Redux to Redux Toolkit.
- **Code reviews:** Checking for mutation, side effects, and missing middleware.

### References

- Redux — Style Guide — https://redux.js.org/style-guide/
- Redux Toolkit — Usage Guide — https://redux-toolkit.js.org/usage/usage-guide
- Redux Essentials — Redux Toolkit Best Practices — https://redux.js.org/tutorials/essentials/part-1-overview-concepts

---

## References

- Redux Toolkit — Official Documentation — https://redux-toolkit.js.org/
- Redux Toolkit — Overview — https://redux.js.org/redux-toolkit/overview
- Redux Toolkit — configureStore — https://redux-toolkit.js.org/api/configureStore
- Redux Toolkit — createSlice — https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit — createReducer — https://redux-toolkit.js.org/api/createReducer
- Redux Toolkit — createAction — https://redux-toolkit.js.org/api/createAction
- Redux Toolkit — createAsyncThunk — https://redux-toolkit.js.org/api/createAsyncThunk
- Redux Toolkit — createEntityAdapter — https://redux-toolkit.js.org/api/createEntityAdapter
- Redux Toolkit — createSelector — https://redux-toolkit.js.org/api/createSelector
- Redux Toolkit — createListenerMiddleware — https://redux-toolkit.js.org/api/createListenerMiddleware
- Redux Toolkit — Usage Guide — https://redux-toolkit.js.org/usage/usage-guide
- Redux Toolkit — Usage with TypeScript — https://redux-toolkit.js.org/usage/usage-with-typescript
- RTK Query — Overview — https://redux-toolkit.js.org/rtk-query/overview
- RTK Query — Queries — https://redux-toolkit.js.org/rtk-query/usage/queries
- RTK Query — Mutations — https://redux-toolkit.js.org/rtk-query/usage/mutations
- RTK Query — Automated Re-fetching — https://redux-toolkit.js.org/rtk-query/usage/automated-refetching
- Redux Essentials — Part 1: Overview — https://redux.js.org/tutorials/essentials/part-1-overview-concepts
- Redux Essentials — Part 2: App Structure — https://redux.js.org/tutorials/essentials/part-2-app-structure
- Redux Essentials — Part 5: Async Logic — https://redux.js.org/tutorials/essentials/part-5-async-logic
- Redux Essentials — Part 6: Performance and Normalization — https://redux.js.org/tutorials/essentials/part-6-performance-normalization
- Redux Essentials — Part 8: RTK Query Advanced Patterns — https://redux.js.org/tutorials/essentials/part-8-rtk-query-advanced
- Redux — Style Guide — https://redux.js.org/style-guide/
- Redux — Migrating to Modern Redux — https://redux.js.org/usage/migrating-to-modern-redux
- Reselect — GitHub — https://github.com/reduxjs/reselect
- Immer — GitHub — https://github.com/immerjs/immer