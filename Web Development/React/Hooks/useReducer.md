# `useReducer` — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

`useReducer` is a React Hook that manages state through a reducer function, where state updates are triggered by dispatching actions that the reducer interprets to produce the next state.

**Technical Definition**

`useReducer(reducer, initialArg, init?)` accepts a reducer function and an initial state, returning the current state and a `dispatch` function. When `dispatch(action)` is called, React invokes the reducer with the current state and the action, and uses the returned value as the next state. The reducer must be a pure function—same inputs always produce the same output, with no side effects. React batches state updates within event handlers, and if the new state is identical to the current state (per `Object.is`), React skips re-rendering .

**Beginner-Friendly Explanation**

`useReducer` is like having a dedicated "state manager" for your component. Instead of directly changing state values, you send messages (actions) describing what happened, and the reducer decides how to update the state based on those messages. It's especially useful when your state has multiple related fields or complex update logic.

### Key Characteristics

- **Centralized Logic**: All state update logic lives in a single reducer function, separate from event handlers .
- **Action-Driven**: State changes are triggered by dispatching action objects that describe what happened .
- **Pure Reducer**: The reducer function must be pure—no mutations, no side effects .
- **Immutable Updates**: State must be updated by returning new objects/arrays, never by mutation .
- **Stable Dispatch**: The `dispatch` function has a stable identity across renders and can be safely omitted from dependency arrays .

### Prerequisites

- React Components and JSX
- `useState` and state management concepts
- JavaScript objects, arrays, and the spread operator
- Basic understanding of pure functions

### Related Programming Areas

- **Redux**: A state management library based on the same reducer pattern 
- **`useContext`**: Often combined with `useReducer` for global state management 
- **Immer**: A library that simplifies immutable updates in reducers 

### Core Concepts / Features

- Reducer Functions
- Actions
- Dispatch
- Initial State
- Complex State Transitions
- Reducer Composition

---

## 1. Reducer Functions

### Definitions

**Core Definition**

A reducer function is a pure function that takes the current state and an action, and returns the next state.

**Technical Definition**

The reducer function signature is `(state, action) => newState`. It must be pure: given the same `state` and `action`, it always returns the same `newState` without modifying its arguments or performing side effects. React calls the reducer during rendering when an action is dispatched. In Strict Mode, React calls the reducer twice in development to detect impurity .

**Beginner-Friendly Explanation**

A reducer is like a recipe book. You give it the current ingredients (state) and an instruction (action), and it tells you what the new ingredients should be. It doesn't change the old ingredients—it gives you a new set.

### Purposes

- To centralize all state update logic in one place .
- To make state transitions predictable and testable .
- To separate "what happened" (action) from "how state changes" (reducer logic) .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'action_type': {
      return newState;
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `state` | Current state value |
| `action` | Object describing what happened |
| `action.type` | String identifying the action |
| `return newState` | The next state (must be a new object/array for objects/arrays) |

**Syntax Rules**

1. **Must be pure**: No mutations, no side effects, no async operations .
2. **Use `switch` for readability**: Convention is to use a `switch` statement on `action.type` .
3. **Wrap cases in braces**: Prevents variable collisions between cases .
4. **Always return**: Every case must return the new state. Missing returns cause the state to become `undefined` .
5. **Throw on unknown actions**: The `default` case should throw an error to catch mistakes .

**Constraints and Limitations**

- The reducer cannot dispatch actions or perform side effects.
- It cannot call other Hooks or access component scope.
- It must return the exact same state reference if no changes are needed (React skips re-render).

### Annotated Complete Code Examples

**Example 1: Basic Counter Reducer**

```jsx
import { useReducer } from 'react';

// Reducer function: pure, no mutations
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment': {
      // Return a new object, do not mutate state
      return { count: state.count + 1 };
    }
    case 'decrement': {
      return { count: state.count - 1 };
    }
    case 'reset': {
      return { count: 0 };
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

**Expected Output**

```
Count: 0
[+ button] [- button] [Reset button]
After clicking +: Count: 1
```

**Why This Output Occurs**

Each `dispatch` call sends an action to `counterReducer`. The reducer matches `action.type` and returns a new state object. React re-renders `Counter` with the updated `state.count` .

### Real-World Cases

**Case: Form State Management**

A form reducer manages multiple fields (`name`, `email`, `age`) with actions like `changed_name`, `changed_email`, and `submitted`, centralizing all form logic in one place .

### References

- React — useReducer API Reference (Reducer function) - https://react.dev/reference/react/useReducer
- React — Extracting State Logic into a Reducer - https://18.react.dev/learn/extracting-state-logic-into-a-reducer

---

## 2. Actions

### Definitions

**Core Definition**

An action is a plain JavaScript object that describes what happened, containing at minimum a `type` property and optionally additional data.

**Technical Definition**

An action is dispatched to the reducer and must include a `type` field (a string) that identifies the action. Additional fields (often grouped under `payload`) carry the data needed to compute the new state. The `type` is specific to the component; naming conventions include `'added'`, `'changed'`, or `'incremented_age'` .

**Beginner-Friendly Explanation**

An action is like a note that says "this happened." It has a label (the type) and maybe some details (like which item or what the new value is). You send this note to the reducer, and it figures out what to do.

### Purposes

- To describe user interactions or events in a declarative way .
- To carry the data needed for state updates.
- To decouple event handlers from state update logic .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
dispatch({
  type: 'action_type',
  // optional additional fields
  payload: data
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type` | String identifying the action (required) |
| Other fields | Data needed by the reducer (optional) |

**Syntax Rules**

1. **`type` is required**: The reducer switches on `action.type` .
2. **Type is component-specific**: `'added'` and `'added_task'` are both valid; choose descriptive names .
3. **Actions can carry any data**: The shape is up to you; common patterns include `payload`, `id`, `text`, `task` .

**Constraints and Limitations**

- Actions should be serializable (no functions or DOM nodes).
- The reducer must handle all action types it might receive.

### Annotated Complete Code Example

**Example: Task Actions**

```javascript
// Action for adding a task
dispatch({
  type: 'added',
  id: nextId++,
  text: text
});

// Action for changing a task
dispatch({
  type: 'changed',
  task: updatedTask
});

// Action for deleting a task
dispatch({
  type: 'deleted',
  id: taskId
});
```

**Explanation**

Each action describes a single interaction. The `type` tells the reducer what kind of change occurred, and the additional fields provide the data needed to perform that change .

### Real-World Cases

**Case: E-Commerce Cart**

Actions like `'added_item'`, `'removed_item'`, and `'cleared_cart'` describe cart operations. The reducer interprets these and returns the updated cart state.

### References

- React — Extracting State Logic into a Reducer (Actions) - https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer
- React — useReducer (Actions) - https://hi.react.dev/reference/react/useReducer

---

## 3. Dispatch

### Definitions

**Core Definition**

`dispatch` is the function returned by `useReducer` that sends actions to the reducer and triggers a re-render with the updated state.

**Technical Definition**

`dispatch(action)` calls the reducer with the current state and the given action, then schedules a re-render if the returned state differs from the current state. The `dispatch` function has a stable identity across renders and does not change .

**Beginner-Friendly Explanation**

`dispatch` is like a messenger. You give it a note (action), and it takes that note to the reducer, which reads it and updates the state accordingly.

### Purposes

- To trigger state updates by sending actions .
- To decouple event handlers from state logic .
- To provide a stable function for use in Effects and callbacks .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: 'action_type' });
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `dispatch` | Function returned by `useReducer` |
| `dispatch(action)` | Sends action to the reducer |

**Syntax Rules**

1. **Dispatch is stable**: Safe to omit from dependency arrays .
2. **No return value**: `dispatch` returns nothing.
3. **Batched**: React batches multiple dispatches within event handlers .

### Annotated Complete Code Example

**Example: Dispatching from Event Handlers**

```jsx
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  function handleIncrement() {
    dispatch({ type: 'increment' });
  }

  return <button onClick={handleIncrement}>+</button>;
}
```

**Explanation**

`dispatch` is called inside the event handler with an action object. React invokes the reducer with the current state and the action, then re-renders with the new state .

### Real-World Cases

**Case: Login Form**

A login form dispatches `'field'` actions on every input change, `'login'` on submit, and `'success'`/`'error'` based on the API response. The reducer handles all state transitions .

### References

- React — useReducer (dispatch function) - https://zh-hans.react.dev/reference/react/useReducer

---

## 4. Initial State

### Definitions

**Core Definition**

Initial state is the value the reducer state holds on the first render, provided as the second argument to `useReducer`.

**Technical Definition**

`useReducer(reducer, initialArg, init?)` uses `initialArg` as the initial state. If `init` is provided, the initial state is `init(initialArg)`. The initial state is computed once and ignored on subsequent renders. For expensive initialization, pass a function to avoid recomputation .

**Beginner-Friendly Explanation**

Initial state is what the reducer starts with before any actions are dispatched. It's the starting point for all state transitions.

### Purposes

- To provide the starting value for the reducer state .
- To avoid expensive initialization on every render .
- To compute initial state from props via an initializer function .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Direct initial state
const [state, dispatch] = useReducer(reducer, { count: 0 });

// With initializer function
const [state, dispatch] = useReducer(reducer, props, createInitialState);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `initialArg` | Value or argument for the initializer |
| `init` | Optional function that computes initial state from `initialArg` |

**Syntax Rules**

1. **Initial state is ignored after first render**: Like `useState`, it's only used initially .
2. **Pass the function, not the call**: `createInitialState` not `createInitialState()` .
3. **Initializer receives the second argument**: `init(initialArg)` .

### Annotated Complete Code Example

**Example: Lazy Initialization**

```jsx
function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return { draft: '', todos: initialTodos };
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    username,
    createInitialState
  );
  // createInitialState runs only once during initialization
}
```

**Explanation**

Passing `createInitialState` (the function) as the third argument means it only runs during initialization. If you passed `createInitialState(username)`, it would run on every render .

### Real-World Cases

**Case: User-Specific Initial State**

A todo list initializes tasks based on the logged-in username, using the initializer function to compute the state only once .

### References

- React — useReducer (Avoiding recreating initial state) - https://hi.react.dev/reference/react/useReducer
- React — useReducer (init function) - https://mintlify.wiki/facebook/react/api/hooks/use-reducer

---

## 5. Complex State Transitions

### Definitions

**Core Definition**

Complex state transitions are state updates involving multiple related fields, conditional logic, or multi-step processes that are difficult to manage with multiple `useState` calls.

**Technical Definition**

When state has multiple fields that must change together (e.g., `isSubmitting`, `isSuccess`, `error`), independent `useState` calls can produce impossible combinations (e.g., `isSubmitting: true` and `isSuccess: true` simultaneously). A single reducer with typed actions ensures every transition produces a valid state, centralizing the transition logic .

**Beginner-Friendly Explanation**

Complex state is like a traffic light—you can't have both green and red at the same time. With `useReducer`, you define all the valid combinations, and the reducer ensures you never end up in an impossible state.

### Purposes

- To prevent impossible state combinations .
- To centralize multi-step transition logic .
- To make state changes traceable and debuggable .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'SUBMIT': {
      return { ...state, status: 'submitting', error: null };
    }
    case 'SUCCESS': {
      return { ...state, status: 'success', data: action.data };
    }
    case 'ERROR': {
      return { ...state, status: 'error', error: action.error };
    }
    default: {
      throw Error('Unknown action');
    }
  }
}
```

### Annotated Complete Code Example

**Example: Checkout Wizard State**

```jsx
// Define possible states as a discriminated union (TypeScript)
type WizardState = {
  currentStep: number;
  completedSteps: Set<number>;
} & (
  | { status: "idle"; error: null }
  | { status: "submitting"; error: null }
  | { status: "error"; error: string }
  | { status: "success"; error: null }
);

function wizardReducer(state, action) {
  switch (action.type) {
    case "SUBMIT_STEP":
      return { ...state, status: "submitting", error: null };
    case "STEP_VALIDATED":
      return {
        ...state,
        status: "idle",
        error: null,
        currentStep: state.currentStep + 1,
        completedSteps: new Set(state.completedSteps).add(state.currentStep)
      };
    case "STEP_FAILED":
      return { ...state, status: "error", error: action.error };
    case "GO_BACK":
      return { ...state, status: "idle", error: null, currentStep: Math.max(0, state.currentStep - 1) };
    default:
      throw Error('Unknown action');
  }
}
```

**Explanation**

The reducer ensures that when `STEP_VALIDATED` is dispatched, `status` always resets to `"idle"`, preventing `isSubmitting` from getting stuck. Each transition produces a valid state combination .

### Real-World Cases

**Case: Login Flow**

A login reducer manages `username`, `password`, `isLoading`, `error`, and `isLoggedIn`. Actions like `'field'`, `'login'`, `'success'`, and `'error'` drive the transitions, ensuring no invalid combinations occur .

### References

- GitHub — Use useReducer for Multi-Field State Transitions - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/react-refactor/references/state-reducer-for-complex.md
- LogRocket — A guide to the React useReducer Hook - https://blog.logrocket.com/react-usereducer-hook-ultimate-guide/

---

## 6. Reducer Composition

### Definitions

**Core Definition**

Reducer composition is the practice of combining multiple reducers or combining a reducer with `useContext` to scale state management across an application.

**Technical Definition**

Reducer composition can mean (1) combining multiple small reducers into one root reducer, or (2) combining `useReducer` with `useContext` to provide state and `dispatch` to any component in the tree. The latter pattern moves state to a top-level provider and uses custom Hooks to read state and dispatch from context .

**Beginner-Friendly Explanation**

Reducer composition is like organizing a large company. Instead of one person (component) managing everything, you have departments (reducers) that handle specific areas, and a central office (provider) that coordinates them.

### Purposes

- To scale state management across large component trees .
- To avoid prop drilling for state and dispatch .
- To keep components focused on display rather than data fetching .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Create contexts
const StateContext = createContext(null);
const DispatchContext = createContext(null);

// Provider component
function AppProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <StateContext value={state}>
      <DispatchContext value={dispatch}>
        {children}
      </DispatchContext>
    </StateContext>
  );
}

// Custom hooks for reading
function useAppState() {
  return useContext(StateContext);
}

function useAppDispatch() {
  return useContext(DispatchContext);
}
```

**Syntax Rules**

1. **Two separate contexts**: State and dispatch in separate contexts to minimize re-renders .
2. **Provider high in the tree**: Place the provider near the root .
3. **Custom hooks**: Export `useAppState` and `useAppDispatch` for clean consumption .

### Annotated Complete Code Example

**Example: Task App with Reducer + Context**

```jsx
// TasksContext.js
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

```jsx
// TaskApp.jsx
import { TasksProvider, useTasks, useTasksDispatch } from './TasksContext';

function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}

function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map(task => (
        <Task key={task.id} task={task} />
      ))}
    </ul>
  );
}

function Task({ task }) {
  const dispatch = useTasksDispatch();

  function handleDelete() {
    dispatch({ type: 'deleted', id: task.id });
  }

  return (
    <li>
      {task.text}
      <button onClick={handleDelete}>Delete</button>
    </li>
  );
}
```

**Explanation**

The `TasksProvider` manages state with `useReducer` and provides `tasks` and `dispatch` via context. `TaskList` reads `tasks` with `useTasks()`, and `Task` dispatches actions with `useTasksDispatch()`. No props are passed down .

### Real-World Cases

**Case: Theme + Auth Global State**

An app provides theme and authentication state via separate reducer-context pairs, allowing any component to read or update them without prop drilling.

### References

- React — Scaling Up with Reducer and Context - https://ar.react.dev/learn/scaling-up-with-reducer-and-context
- GitHub — Scaling Up with Reducer and Context (Source) - https://github.com/reactjs/react.dev/blob/c0af2d01/src/content/learn/scaling-up-with-reducer-and-context.md

---

## References

- React Official Documentation — useReducer API Reference - https://react.dev/reference/react/useReducer
- React Official Documentation — useReducer (Chinese) - https://zh-hans.react.dev/reference/react/useReducer
- React Official Documentation — Extracting State Logic into a Reducer - https://18.react.dev/learn/extracting-state-logic-into-a-reducer
- React Official Documentation — Scaling Up with Reducer and Context - https://ar.react.dev/learn/scaling-up-with-reducer-and-context
- React Official Documentation — Scaling Up (Ukrainian) - https://uk.react.dev/learn/scaling-up-with-reducer-and-context
- React Official Documentation — Extracting State Logic (Chinese) - https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer
- React Official Documentation — useReducer (Hindi) - https://hi.react.dev/reference/react/useReducer
- React Official Documentation — useReducer (Mintlify) - https://mintlify.wiki/facebook/react/api/hooks/use-reducer
- Refine — React useReducer Hook - The Basics - https://refine.dev/blog/react-usereducer/
- LogRocket — A guide to the React useReducer Hook - https://blog.logrocket.com/react-usereducer-hook-ultimate-guide/
- GitHub — Use useReducer for Multi-Field State Transitions - https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/react-refactor/references/state-reducer-for-complex.md
- CoreUI — How to Use useReducer in React - https://coreui.io/answers/how-to-use-usereducer-in-react/
- Educative — Managing complex state with React useReducer hook - https://www.educative.io/courses/react-beginner-to-advanced/usereducer
- GitHub — Scaling Up with Reducer and Context (React Source) - https://github.com/reactjs/react.dev/blob/c0af2d01/src/content/learn/scaling-up-with-reducer-and-context.md