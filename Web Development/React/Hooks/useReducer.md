# `useReducer` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useReducer` is a React Hook that lets you manage a component's state using a reducer function, providing a structured approach to handling complex state logic.

**Technical Definition:** `useReducer(reducer, initialArg, init?)` returns an array with the current state and a `dispatch` function. It is an alternative to `useState` that consolidates all state update logic into a single pure function called a reducer. The reducer receives the current state and an action object, and returns the next state. React calls the reducer during the next render to compute the new state. `useReducer` is a Hook, so it can only be called at the top level of a component or custom Hook, never inside loops or conditions. The `dispatch` function has a stable identity, meaning it can be safely omitted from `useEffect` dependency arrays.

**Beginner-Friendly Explanation:** `useReducer` is a way to manage state that is more structured than `useState`. Instead of scattering state update logic across many event handlers, you put all of it in one function called a "reducer." When something happens (like a user clicking a button), you "dispatch" an action that describes what happened. The reducer looks at that action and decides how the state should change. It is like having a single control panel for all your state changes, rather than buttons scattered everywhere.

### Key Characteristics

- **Centralised logic:** All state transition logic lives in a single reducer function.
- **Action-based:** State changes are triggered by dispatching action objects.
- **Pure function:** The reducer must be pure — no side effects, no mutations.
- **Stable dispatch:** The `dispatch` function identity is stable across renders.
- **Predictable:** Given the same state and action, the reducer always returns the same next state.
- **Testable:** Reducers are pure functions and can be unit tested in isolation.
- **Scalable:** Preferred over `useState` when state has multiple sub-values or complex transitions.

### Prerequisites

- JavaScript fundamentals: functions, objects, arrays, switch statements, destructuring.
- Basic React function component syntax and JSX.
- Understanding of `useState` and React state fundamentals.
- Familiarity with the concept of pure functions.

### Related Programming Areas

- **`useState`** — simpler alternative for independent, simple state values.
- **`useContext`** — often combined with `useReducer` to share state across components.
- **Redux** — a global state management library that uses the same reducer pattern.
- **State machines** — reducers are closely related to finite state machines.

### Core Concepts / Features

---

## 1. `useReducer`

### Definitions

**Core Definition:** `useReducer` is a React Hook that manages component state using a reducer function.

**Technical Definition:** `useReducer(reducer, initialArg, init?)` returns an array with two values: the current state and a `dispatch` function. The `reducer` parameter is a pure function that takes the current state and an action, and returns the next state. The `initialArg` is the value from which the initial state is calculated. An optional `init` function can be provided for lazy initialisation, in which case the initial state is set to `init(initialArg)`.

**Beginner-Friendly Explanation:** `useReducer` is like `useState`, but instead of directly setting the new value, you dispatch an action. A reducer function then decides what the new state should be based on that action.

### Purposes

- To manage state with complex logic where the next state depends on the previous state.
- To consolidate multiple related state updates into a single reducer function.
- To make state transitions predictable and testable.
- To handle multi-step workflows and state machines.
- To avoid scattering state update logic across many event handlers.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducer, initialArg, init?);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `reducer` | Pure function `(state, action) => nextState`. |
| `initialArg` | Value from which the initial state is calculated. |
| `init` | Optional initializer function `(initialArg) => initialState`. |
| `state` | The current state for this render. |
| `dispatch` | Function that accepts an action and triggers a re-render. |

**Syntax Rules:**

1. `useReducer` must be called at the top level of a component or custom Hook.
2. The reducer must be pure and return the next state.
3. The `dispatch` function accepts a single `action` argument.
4. The initial state is set to `init(initialArg)` if `init` is provided, otherwise to `initialArg`.

**Constraints and Limitations:**

- Cannot be called inside loops or conditions.
- The reducer must not mutate the state; it must return a new value.
- In Strict Mode, React calls the reducer and initializer twice in development.

### Annotated Code Examples

```jsx
import { useReducer } from 'react';

// Step 1: Define the reducer function
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unknown action type');
  }
}

// Step 2: Use the reducer in the component
function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </div>
  );
}
```

**Expected Output:** A counter with increment and decrement buttons.

**Why this output:** `useReducer` initialises state to `{ count: 0 }`. Clicking the "+" button dispatches `{ type: 'increment' }`, which the reducer processes to return `{ count: 1 }`.

### Real-World Cases

- **Complex forms:** Managing multiple form fields with validation.
- **Shopping carts:** Adding, removing, and updating cart items.
- **Multi-step wizards:** Tracking the current step and form data.
- **Undo/redo:** Storing a history of state transitions.

### References

- React — useReducer — https://react.dev/reference/react/useReducer
- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer

---

## 2. Reducer Function

### Definitions

**Core Definition:** A reducer function is a pure function that takes the current state and an action, and returns the next state.

**Technical Definition:** A reducer is a function with the signature `(state, action) => newState`. It must be pure: given the same inputs, it must always return the same output, and it must not produce side effects. The reducer uses the action's `type` property to determine how to compute the next state. If the action is not recognised, the reducer should return the current state unchanged, or throw an error for invalid actions.

**Beginner-Friendly Explanation:** A reducer is a function that decides how the state changes. You give it the current state and an action, and it tells you what the new state should be.

### Purposes

- To centralise all state update logic in one place.
- To make state transitions explicit and predictable.
- To enable unit testing of state logic in isolation.
- To handle complex state transitions with clear, readable code.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'ACTION_TYPE':
      return nextState;
    default:
      return state;
  }
}
```

**Syntax Rules:**

1. The reducer must be pure.
2. State and action can be of any type.
3. Use `switch` statements or if-else chains to handle action types.
4. Always return the current state for unrecognised actions (or throw).

**Constraints and Limitations:**

- Must not mutate the state; return a new object or array.
- Must not perform side effects (API calls, logging, etc.).
- In Strict Mode, the reducer runs twice in development to detect impurities.

### Annotated Code Examples

```jsx
function todosReducer(state, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...state,
        { id: action.id, text: action.text, done: false },
      ];
    }
    case 'changed': {
      return state.map((t) =>
        t.id === action.todo.id ? action.todo : t
      );
    }
    case 'deleted': {
      return state.filter((t) => t.id !== action.id);
    }
    default: {
      throw new Error('Unknown action: ' + action.type);
    }
  }
}
```

**Expected Output:** The reducer correctly handles adding, changing, and deleting todos.

**Why this output:** Each action type maps to a pure transformation of the state array. The spread operator creates new arrays without mutating the original.

### Real-World Cases

- **Todo lists:** Adding, editing, and deleting items.
- **Shopping carts:** Adding products, updating quantities, removing items.
- **Form state:** Updating fields, resetting, submitting.
- **Game state:** Moving pieces, tracking scores, managing turns.

### References

- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer
- React — useReducer: reducer — https://react.dev/reference/react/useReducer#parameters

---

## 3. Initial State

### Definitions

**Core Definition:** The initial state is the value the reducer uses on the first render before any actions have been dispatched.

**Technical Definition:** The initial state is calculated from `initialArg` or, if an `init` function is provided, from `init(initialArg)`. It is used only during the first render and is ignored on subsequent renders. The initial state can be of any type: a primitive, object, array, or even `null`.

**Beginner-Friendly Explanation:** The initial state is what the component "starts with" before the user does anything.

### Purposes

- To define the starting point of the state.
- To provide a predictable baseline for the component.
- To enable lazy initialisation for expensive initial values.

### Syntax Rules and Structure

```jsx
// Without init function
const [state, dispatch] = useReducer(reducer, { count: 0 });

// With init function (lazy initialisation)
const [state, dispatch] = useReducer(reducer, props.initialCount, init);
```

**Syntax Rules:**

1. The initial state is used only on the first render.
2. `init` is optional; if omitted, the initial state is `initialArg`.
3. `init` receives `initialArg` as its argument and returns the initial state.

### Annotated Code Examples

```jsx
function init(initialCount) {
  return { count: initialCount };
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'reset':
      return init(action.payload);
    default:
      return state;
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset', payload: initialCount })}>
        Reset
      </button>
    </div>
  );
}
```

**Expected Output:** A counter that starts at `initialCount` and can be reset to it.

**Why this output:** The `init` function transforms `initialCount` into the initial state object. The reset action reuses `init` to restore the original count.

### Real-World Cases

- **Forms:** Initialising with default values from props.
- **Counters:** Starting from a configurable value.
- **Filters:** Initialising with default filter criteria.

### References

- React — useReducer: initialArg — https://react.dev/reference/react/useReducer#parameters
- React — Avoiding recreating the initial state — https://react.dev/reference/react/useReducer#avoiding-recreating-the-initial-state

---

## 4. Action Objects

### Definitions

**Core Definition:** An action object is a plain JavaScript object that describes what the user did, dispatched to the reducer to trigger a state change.

**Technical Definition:** An action is a value of any type, but by convention it is an object with a `type` property that identifies the action, and optionally other properties (such as `payload`) that carry additional data. The reducer reads `action.type` to determine how to update the state. Actions are dispatched via the `dispatch` function.

**Beginner-Friendly Explanation:** An action is a message that says "this happened." It usually has a type (like `'increment'`) and may carry extra data (like how much to increment by).

### Purposes

- To describe state changes in a declarative way.
- To decouple event handlers from state update logic.
- To provide a clear, readable record of what happened.
- To enable action creators for reusable action generation.

### Syntax Rules and Structure

**General Syntax:**

```jsx
dispatch({ type: 'ACTION_TYPE', payload: data });
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `type` | String identifying the action (required by convention). |
| `payload` | Optional additional data. |

**Syntax Rules:**

1. Actions are usually objects with a `type` property.
2. The `type` should be a descriptive string (e.g., `'added'`, `'deleted'`).
3. Additional data can be included as other properties.

### Annotated Code Examples

```jsx
// Action creators
const addTodo = (text) => ({ type: 'added', id: nextId++, text });
const deleteTodo = (id) => ({ type: 'deleted', id });
const toggleTodo = (todo) => ({ type: 'changed', todo: { ...todo, done: !todo.done } });

// Dispatching actions
dispatch(addTodo('Learn useReducer'));
dispatch(deleteTodo(1));
dispatch(toggleTodo(todo));
```

**Expected Output:** Actions are created and dispatched to update the state.

**Why this output:** Action creators are functions that return action objects, making it easy to create consistent actions with the correct shape.

### Real-World Cases

- **Todo apps:** Add, delete, toggle, edit actions.
- **Shopping carts:** Add item, remove item, update quantity.
- **Form wizards:** Next step, previous step, reset.

### References

- React — dispatch function — https://react.dev/reference/react/useReducer#dispatch-function
- Refine — React useReducer Hook: The Basics — https://refine.dev/blog/react-usereducer-hook

---

## 5. Action Types

### Definitions

**Core Definition:** Action types are string constants that identify the kind of action being dispatched.

**Technical Definition:** Action types are the `type` property values in action objects. They are conventionally written in uppercase with underscores (e.g., `'ADD_TODO'`, `'REMOVE_ITEM'`) or lowercase (e.g., `'added'`, `'deleted'`). Using constants for action types prevents typos and enables autocompletion. It is common to define them in a separate file or as a frozen object.

**Beginner-Friendly Explanation:** Action types are labels that tell the reducer what kind of action happened. Using constants instead of raw strings helps avoid spelling mistakes.

### Purposes

- To identify the kind of action being dispatched.
- To prevent typos by using constants.
- To enable editor autocompletion and refactoring.
- To document all possible actions in one place.

### Syntax Rules and Structure

```jsx
// Action type constants
const ACTIONS = {
  ADD_TODO: 'ADD_TODO',
  DELETE_TODO: 'DELETE_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO',
};

dispatch({ type: ACTIONS.ADD_TODO, text: 'Learn' });
```

**Syntax Rules:**

1. Action types should be descriptive and unique.
2. Use constants to avoid typos.
3. Define them in a separate file for larger applications.

### Annotated Code Examples

```jsx
export const CART_ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY',
  CLEAR_CART: 'CLEAR_CART',
};

function cartReducer(state, action) {
  switch (action.type) {
    case CART_ACTIONS.ADD_ITEM:
      return { ...state, items: [...state.items, action.payload] };
    case CART_ACTIONS.REMOVE_ITEM:
      return { ...state, items: state.items.filter((i) => i.id !== action.payload) };
    case CART_ACTIONS.CLEAR_CART:
      return { ...state, items: [] };
    default:
      return state;
  }
}
```

**Expected Output:** The reducer handles each action type correctly.

**Why this output:** Using constants prevents typos and makes the reducer easier to maintain.

### Real-World Cases

- **Shopping carts:** Add, remove, clear actions.
- **Todo lists:** Add, delete, toggle, edit actions.
- **Authentication:** Login, logout, register actions.

### References

- FreeCodeCamp — Shared State Complexity in React — https://www.freecodecamp.org

---

## 6. Dispatching Actions

### Definitions

**Core Definition:** Dispatching an action is the act of calling the `dispatch` function with an action object to trigger a state change.

**Technical Definition:** The `dispatch` function returned by `useReducer` accepts a single argument — the action — and tells React to call the reducer with the current state and that action. React then schedules a re-render with the new state. The `dispatch` function has a stable identity, so it can be safely omitted from dependency arrays. It does not return a value, and the state variable is not updated immediately after calling `dispatch`.

**Beginner-Friendly Explanation:** Dispatching is how you tell the reducer "this happened." You call `dispatch` with an action, and React updates the state.

### Purposes

- To trigger state changes from event handlers.
- To decouple event handlers from state update logic.
- To provide a consistent interface for all state updates.

### Syntax Rules and Structure

```jsx
function handleClick() {
  dispatch({ type: 'increment' });
}
```

**Syntax Rules:**

1. `dispatch` accepts exactly one argument (the action).
2. `dispatch` does not return a value.
3. The state variable is not updated immediately after calling `dispatch`.
4. `dispatch` has a stable identity across renders.

### Annotated Code Examples

```jsx
function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  function handleIncrement() {
    dispatch({ type: 'increment' });
    console.log(state.count); // Still old value!
  }

  return <button onClick={handleIncrement}>Count: {state.count}</button>;
}
```

**Expected Output:** Clicking the button increments the count, but the `console.log` shows the old value.

**Why this output:** `dispatch` schedules a re-render. The state variable in the current render still holds the old value.

### Real-World Cases

- **All components using `useReducer`:** Dispatching is the standard way to trigger state changes.
- **Async operations:** Dispatching after a fetch completes.

### References

- React — dispatch function — https://react.dev/reference/react/useReducer#dispatch-function
- React — I've dispatched an action but logging gives me the old state value — https://react.dev/reference/react/useReducer#ive-dispatched-an-action-but-logging-gives-me-the-old-state-value

---

## 7. State Transitions

### Definitions

**Core Definition:** State transitions are the changes from one state value to another, driven by dispatching actions and processed by the reducer.

**Technical Definition:** A state transition occurs when the reducer receives the current state and an action, and returns the next state. Reducers model state transitions as pure functions, making them predictable and testable. Unlike `useState`, where state transitions are scattered across event handlers, `useReducer` centralises all transitions in one place, ensuring that invalid state combinations are impossible.

**Beginner-Friendly Explanation:** State transitions are how the state changes from one value to another. The reducer decides what the next state should be based on the action.

### Purposes

- To ensure every state transition produces a valid state.
- To centralise transition logic for maintainability.
- To prevent impossible state combinations (e.g., `isLoading: true` and `isSuccess: true`).

### Syntax Rules and Structure

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, status: 'loading', error: null };
    case 'FETCH_SUCCESS':
      return { ...state, status: 'success', data: action.payload };
    case 'FETCH_ERROR':
      return { ...state, status: 'error', error: action.payload };
    default:
      return state;
  }
}
```

**Syntax Rules:**

1. Each action type maps to a specific state transition.
2. Transitions should be pure and deterministic.
3. Invalid transitions should return the current state or throw.

### Annotated Code Examples

```jsx
function checkoutReducer(state, action) {
  switch (action.type) {
    case 'NEXT_STEP':
      return { ...state, step: state.step + 1, isSubmitting: false };
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true, isSuccess: false, error: null };
    case 'SUBMIT_SUCCESS':
      return { ...state, isSubmitting: false, isSuccess: true, step: 4 };
    case 'SUBMIT_ERROR':
      return { ...state, isSubmitting: false, isSuccess: false, error: action.payload };
    default:
      return state;
  }
}
```

**Expected Output:** The checkout wizard transitions between steps and submission states correctly.

**Why this output:** Each action type produces a specific, valid state transition. It is impossible to be both `isSubmitting` and `isSuccess` simultaneously.

### Real-World Cases

- **Checkout flows:** Multi-step wizards with validation.
- **Authentication:** Login start, success, error transitions.
- **Data fetching:** Loading, success, error states.

### References

- shipshitdev — Use useReducer for Multi-Field State Transitions — https://github.com/shipshitdev/skills

---

## 8. Complex State Logic

### Definitions

**Core Definition:** Complex state logic refers to state management scenarios where multiple related values change together, transitions depend on previous state, or many actions affect the same state.

**Technical Definition:** Complex state logic arises when state has multiple sub-values that must remain consistent with each other, when the next state depends on the previous state in non-trivial ways, or when multiple event handlers need to update the same state. `useReducer` is preferred over `useState` in these scenarios because it centralises transition logic, prevents invalid state combinations, and makes the code more maintainable.

**Beginner-Friendly Explanation:** Complex state logic is when managing state becomes difficult because many things are connected. `useReducer` helps by putting all the logic in one place.

### Purposes

- To manage state objects with multiple sub-values.
- To handle multi-step workflows and state machines.
- To prevent invalid state combinations.
- To centralise state update logic for maintainability.

### Syntax Rules and Structure

```jsx
const [state, dispatch] = useReducer(reducer, {
  step: 1,
  formData: {},
  isSubmitting: false,
  isSuccess: false,
  error: null,
});
```

**Syntax Rules:**

1. Use `useReducer` when state has 3+ related fields.
2. Use `useReducer` when transitions depend on previous state.
3. Use `useReducer` when multiple actions affect the same state.

### Annotated Code Examples

```jsx
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find((i) => i.id === action.payload.id);
      if (existing) {
        return {
          ...state,
          items: state.items.map((i) =>
            i.id === action.payload.id
              ? { ...i, quantity: i.quantity + 1 }
              : i
          ),
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
      };
    }
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter((i) => i.id !== action.payload),
      };
    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map((i) =>
          i.id === action.payload.id
            ? { ...i, quantity: action.payload.quantity }
            : i
        ),
      };
    case 'CLEAR_CART':
      return { ...state, items: [] };
    default:
      throw new Error('Unknown action: ' + action.type);
  }
}
```

**Expected Output:** A shopping cart that correctly handles adding, removing, and updating items.

**Why this output:** All cart logic is centralised in one reducer. Each action type produces a valid cart state.

### Real-World Cases

- **Shopping carts:** Adding, removing, and updating items.
- **Checkout wizards:** Multi-step forms with validation.
- **Dashboard filters:** Multiple filter criteria that interact.
- **Game state:** Complex game rules and transitions.

### References

- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer
- CoreUI — How to build an e-commerce cart in React — https://coreui.io

---

## 9. `useReducer` vs `useState`

### Definitions

**Core Definition:** `useState` and `useReducer` are two React Hooks for managing state, with `useState` being simpler and `useReducer` providing more structure for complex logic.

**Technical Definition:** `useState` declares a state variable that can be updated directly with a setter function. `useReducer` declares a state variable with update logic inside a reducer function. The main difference is structural: `useState` scatters update logic across event handlers, while `useReducer` centralises it in a pure function. Both are optimised similarly by React in terms of batching.

**Beginner-Friendly Explanation:** `useState` is for simple state (like a toggle or a counter). `useReducer` is for complex state where many things change together or the logic is complicated.

### Purposes

- **`useState`:** To manage simple, independent state values.
- **`useReducer`:** To manage complex state with multiple sub-values or dependent transitions.
- **`useReducer`:** To centralise state logic and make it testable.

### Comparison Table

| Aspect | `useState` | `useReducer` |
|--------|-----------|--------------|
| Complexity | Simple | Complex |
| State shape | Primitives, simple objects | Objects with multiple sub-values |
| Update logic | Direct setter calls | Reducer function |
| Testability | Harder to test in isolation | Easy to unit test |
| Code organisation | Logic scattered in handlers | Logic centralised in reducer |
| When to use | Independent values, toggles | Related values, multi-step workflows |

**Syntax Rules:**

1. Start with `useState`; switch to `useReducer` when logic becomes complex.
2. Use `useReducer` when state has 3+ related fields.
3. Use `useReducer` when the next state depends on the previous state.

### Annotated Code Examples

**useState (Simple):**

```jsx
const [isOn, setIsOn] = useState(false);
<button onClick={() => setIsOn(!isOn)}>{isOn ? 'ON' : 'OFF'}</button>
```

**useReducer (Complex):**

```jsx
const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
// Multiple related values with complex update logic
```

**Expected Output:** Both manage state, but `useReducer` is better for complex scenarios.

**Why this output:** `useState` is sufficient for simple toggles. `useReducer` provides structure for related values and complex transitions.

### Real-World Cases

- **Simple forms:** Use `useState` for individual fields.
- **Complex forms:** Use `useReducer` for multi-step wizards.
- **Toggles:** Use `useState`.
- **Shopping carts:** Use `useReducer`.

### References

- React — Comparing useState and useReducer — https://react.dev/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer
- LogRocket — React Context tutorial: Complete guide — https://blog.logrocket.com

---

## 10. Lazy Initialization

### Definitions

**Core Definition:** Lazy initialisation is the practice of passing an `init` function to `useReducer` so the initial state is computed only when needed.

**Technical Definition:** `useReducer` accepts an optional third argument, `init`, which is a function that receives `initialArg` and returns the initial state. This is useful when the initial state needs to be computed from props, when the computation is expensive, or when you want to extract the logic for resetting state. The `init` function is called only once during the first render.

**Beginner-Friendly Explanation:** If computing the starting state is slow or depends on props, you can give `useReducer` a function instead of a value. React will call that function only once.

### Purposes

- To avoid expensive computations on every render.
- To compute initial state from props.
- To extract reset logic into a reusable function.

### Syntax Rules and Structure

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init);

function init(initialArg) {
  return { /* computed initial state */ };
}
```

**Syntax Rules:**

1. `init` is the optional third argument to `useReducer`.
2. `init` receives `initialArg` and returns the initial state.
3. `init` is called only once during the first render.

### Annotated Code Examples

```jsx
function createInitialState(username) {
  return {
    username,
    todos: loadTodosFromStorage(username),
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'reset':
      return createInitialState(action.username);
    // ... other actions
  }
}

function TodoApp({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    username,
    createInitialState
  );
  // state is initialised lazily from username
}
```

**Expected Output:** The initial state is computed from `username` only once.

**Why this output:** `createInitialState` is passed as the `init` function. React calls it with `username` during the first render and uses the result as the initial state. The same function can be reused for reset actions.

### Real-World Cases

- **Forms:** Initialising from props or persisted data.
- **Counters:** Starting from a configurable value.
- **Reset functionality:** Reusing the init function to reset state.

### References

- React — useReducer: init — https://react.dev/reference/react/useReducer#parameters
- Epic React — Advanced State Management — https://advanced-apis.epicreact.dev

---

## 11. Reducer Composition

### Definitions

**Core Definition:** Reducer composition is the practice of combining multiple reducers into a single reducer, each managing a slice of the state.

**Technical Definition:** Large applications often have state with multiple independent slices. Reducer composition allows you to write a separate reducer for each slice and combine them into a root reducer. This pattern is common in Redux and can be applied to `useReducer`. It keeps reducers focused and testable, and makes it easy to add new slices.

**Beginner-Friendly Explanation:** Instead of one giant reducer, you can have several smaller reducers, each handling a different part of the state. Then you combine them.

### Purposes

- To keep reducers focused and manageable.
- To enable independent testing of each slice.
- To make it easy to add new state slices.
- To mirror the structure of the state in the reducer structure.

### Syntax Rules and Structure

```jsx
function rootReducer(state, action) {
  return {
    todos: todosReducer(state.todos, action),
    filter: filterReducer(state.filter, action),
  };
}
```

**Syntax Rules:**

1. Each slice reducer manages a portion of the state.
2. The root reducer delegates to each slice reducer.
3. Slice reducers return their portion of the state.

### Annotated Code Examples

```jsx
function todosReducer(todos, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...todos, action.payload];
    case 'DELETE_TODO':
      return todos.filter((t) => t.id !== action.payload);
    default:
      return todos;
  }
}

function filterReducer(filter, action) {
  switch (action.type) {
    case 'SET_FILTER':
      return action.payload;
    default:
      return filter;
  }
}

function rootReducer(state, action) {
  return {
    todos: todosReducer(state.todos, action),
    filter: filterReducer(state.filter, action),
  };
}

const [state, dispatch] = useReducer(rootReducer, {
  todos: [],
  filter: 'all',
});
```

**Expected Output:** The root reducer delegates to the appropriate slice reducer.

**Why this output:** Each action is passed to every slice reducer. Only the relevant slice responds, and the others return their current state unchanged.

### Real-World Cases

- **Large applications:** Multiple state slices (users, products, orders).
- **Redux-style architecture:** Combining reducers for different features.
- **Modular codebases:** Keeping state logic organised by domain.

### References

- Redux — Reducer Composition — https://redux.js.org
- Educative — Combining Architectural Patterns in React — https://www.educative.io

---

## 12. Reducer Testing

### Definitions

**Core Definition:** Reducer testing is the practice of unit testing reducer functions in isolation, without rendering any React components.

**Technical Definition:** Reducers are pure functions, which makes them easy to test. A unit test for a reducer calls it with a given state and action, and asserts that the returned state matches the expected next state. This approach tests state transition logic directly, without the overhead of rendering components. Component tests can then verify that dispatching actions triggers the correct reducer updates.

**Beginner-Friendly Explanation:** Because reducers are just functions, you can test them directly without needing to render anything. You call the reducer with a state and an action, and check the result.

### Purposes

- To test state transition logic in isolation.
- To catch bugs in reducer logic before integration.
- To document expected behaviour through tests.
- To enable safe refactoring of reducer logic.

### Syntax Rules and Structure

```jsx
test('increment increases count by 1', () => {
  const initialState = { count: 0 };
  const action = { type: 'increment' };
  const nextState = reducer(initialState, action);
  expect(nextState).toEqual({ count: 1 });
});
```

**Syntax Rules:**

1. Import the reducer function directly.
2. Call it with a known state and action.
3. Assert the returned state matches expectations.

### Annotated Code Examples

```jsx
// reducer.js
export function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// reducer.test.js
import { counterReducer } from './reducer';

describe('counterReducer', () => {
  test('increments count', () => {
    const state = { count: 0 };
    const next = counterReducer(state, { type: 'increment' });
    expect(next).toEqual({ count: 1 });
  });

  test('returns current state for unknown action', () => {
    const state = { count: 5 };
    const next = counterReducer(state, { type: 'unknown' });
    expect(next).toBe(state);
  });
});
```

**Expected Output:** All tests pass.

**Why this output:** The reducer is pure, so calling it with a known state and action produces a predictable result that can be asserted.

### Real-World Cases

- **CI/CD pipelines:** Automated reducer tests catch regressions.
- **TDD:** Writing tests before implementing reducer logic.
- **Refactoring:** Ensuring reducer behaviour is preserved.

### References

- Stack Overflow — How test a component using the useReducer hook? — https://stackoverflow.com
- Redux — Writing Tests — https://redux.js.org/usage/writing-tests

---

## 13. `useReducer` Best Practices

### Definitions

**Core Definition:** `useReducer` best practices are guidelines for using the Hook effectively, safely, and maintainably.

**Technical Definition:** Best practices encompass keeping reducers pure, using action type constants, typing actions with TypeScript, combining with context for global state, avoiding prop drilling of `dispatch`, and writing reducers that are easy to test. These practices improve code quality, prevent bugs, and align with React's recommendations.

**Beginner-Friendly Explanation:** Best practices are the "do's and don'ts" that experienced React developers follow to keep their code clean and bug-free.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| Keep reducers pure | No side effects, no mutations, deterministic output. |
| Use action type constants | Avoid typos, enable autocompletion. |
| Type actions with TypeScript | Discriminated unions for type safety. |
| Combine with context | Share state and dispatch without prop drilling. |
| Don't prop-drill dispatch | Put dispatch in context or pass it explicitly. |
| Write testable reducers | Pure functions are easy to unit test. |
| Use meaningful action types | Descriptive names that explain what happened. |
| Return new state objects | Never mutate the state; spread and create new objects. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Action type constants
export const ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
} as const;

// TypeScript discriminated union
type Action =
  | { type: 'ADD_ITEM'; payload: Item }
  | { type: 'REMOVE_ITEM'; id: string };

// Pure reducer with new state objects
function cartReducer(state: CartState, action: Action): CartState {
  switch (action.type) {
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.payload] };
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter((i) => i.id !== action.id) };
    default:
      return state;
  }
}
```

**❌ Bad Practices:**

```jsx
// Raw string action types (typo risk)
dispatch({ type: 'addItem' }); // typo!
dispatch({ type: 'ADD_ITEM' });

// Mutating state
function badReducer(state, action) {
  state.items.push(action.payload); // ❌ Mutation!
  return state;
}
```

**Expected Output:** The good practices produce predictable, testable, type-safe code.

**Why this output:** Constants prevent typos. TypeScript catches errors at compile time. Pure reducers ensure React detects changes.

### Real-World Cases

- **All applications using `useReducer`:** These practices apply universally.
- **Team projects:** Consistency reduces integration bugs.
- **Large codebases:** Best practices keep code maintainable.

### References

- Refine — React useReducer Hook: The Basics — https://refine.dev/blog/react-usereducer-hook
- Steve Kinney — From Passing Dispatch to Context — https://stevekinney.com
- shipshitdev — Use useReducer for Multi-Field State Transitions — https://github.com/shipshitdev/skills

---

## References

- React — useReducer — https://react.dev/reference/react/useReducer
- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer
- React — Scaling Up with Reducer and Context — https://react.dev/learn/scaling-up-with-reducer-and-context
- React — Comparing useState and useReducer — https://react.dev/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer
- React — Built-in React Hooks — https://react.dev/reference/react/hooks
- Refine — React useReducer Hook: The Basics — https://refine.dev/blog/react-usereducer-hook
- LogRocket — A guide to the React useReducer Hook — https://blog.logrocket.com
- LogRocket — React Context tutorial: Complete guide with practical examples — https://blog.logrocket.com
- Stack Overflow — Why choose useReducer over useState in React? — https://stackoverflow.com
- Stack Overflow — How test a component using the useReducer hook? — https://stackoverflow.com
- Redux — Writing Tests — https://redux.js.org/usage/writing-tests
- Epic React — Advanced State Management — https://advanced-apis.epicreact.dev
- shipshitdev — Use useReducer for Multi-Field State Transitions — https://github.com/shipshitdev/skills
- Steve Kinney — From Passing Dispatch to Context — https://stevekinney.com
- FreeCodeCamp — Shared State Complexity in React — https://www.freecodecamp.org
- CoreUI — How to build an e-commerce cart in React — https://coreui.io