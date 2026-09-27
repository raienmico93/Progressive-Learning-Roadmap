# React Effect Design: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Effect Design is the disciplined practice of deciding when and how to use the `useEffect` Hook to synchronise React components with external systems, while avoiding unnecessary Effects, maintaining correct dependencies, handling asynchronous race conditions, and ensuring proper cleanup.

**Technical Definition:** React Effect Design encompasses the architectural patterns and rules that govern the `useEffect` Hook lifecycle in functional components. It distinguishes between logic caused by rendering (Effects) and logic caused by user interactions (event handlers), enforces correct dependency arrays to prevent stale closures, promotes derived state calculation during render rather than via state-syncing Effects, and mandates cleanup functions to release external resources. It also addresses the concurrency hazards inherent in asynchronous Effects, including race conditions and request cancellation.

**Beginner-Friendly Explanation:** React is great at figuring out what your screen should look like. But some things—like fetching data, setting timers, or connecting to a server—happen outside of that. `useEffect` is React's tool for handling those "outside" things. Effect Design is the art of using that tool *correctly*: knowing when you actually need it (and when you don't), making sure it re-runs at the right times, cleaning up after itself, and not getting confused when multiple things happen at once.

### Key Characteristics

- **Render-Triggered:** Effects are caused by rendering itself, not by specific user interactions.
- **Reactive:** Effects re-synchronise whenever any reactive value they read (props, state, context) changes.
- **Asynchronous by Nature:** Effects run after the commit phase, and asynchronous Effects require explicit guards against race conditions and unmounted-component updates.
- **Declarative, Not Imperative:** Effects describe *what* should be synchronised, not *when* to execute steps imperatively.
- **Cleanup-Oriented:** Every Effect that creates a resource should return a cleanup function that disposes of it.
- **Design-Sensitive:** The choice between an Effect and an event handler is a foundational design decision that affects correctness, performance, and maintainability.

### Prerequisites

- Solid understanding of JavaScript closures, promises, and asynchronous programming.
- Familiarity with React components, props, and state.
- Working knowledge of `useState`, `useRef`, and the render/commit lifecycle.
- Basic understanding of the browser's event model and Web APIs (fetch, AbortController).

### Related Programming Areas

- **Reactive Programming:** Managing data flow and change propagation.
- **Concurrency Control:** Handling asynchronous operations that may complete out of order.
- **Resource Management:** Preventing memory leaks and orphaned subscriptions.
- **Declarative UI:** Describing outcomes rather than imperative steps.
- **Performance Optimisation:** Avoiding unnecessary renders and expensive recalculations.

### Core Concepts / Features

1. Synchronisation versus Event Handling
2. Avoiding Unnecessary Effects
3. Effect Dependency Correctness
4. Derived Data Without Effects
5. Effect Cleanup
6. Race Conditions
7. Request Cancellation

---

## Core Concept 1: Synchronisation versus Event Handling

### Definitions

**Core Definition:** Synchronisation versus event handling is the design decision of whether a piece of logic should run automatically whenever the component is rendered and its reactive dependencies change (an Effect), or whether it should run only in response to a specific user interaction (an event handler).

**Technical Definition:** Event handlers are functions defined inside React components that execute in response to specific user interactions (clicks, typing, form submissions). They are *not* reactive—they capture the values from the render in which they were created and only re-run when the same interaction occurs again. Effects, by contrast, are *reactive*—they re-synchronise whenever any value they read from the render scope (props, state, context) changes. The rule is simple: ask *why* the code needs to run. If it runs because the component was displayed, use an Effect. If it runs because of a particular interaction, use an event handler.

**Beginner-Friendly Explanation:** Imagine a chat room. The act of *sending* a message happens because the user clicked the "Send" button—that's an event handler. But the act of *connecting* to the chat server happens because the component is visible on screen, regardless of how the user got there—that's an Effect. If you put the connection logic in an event handler, it would only connect when the user clicks something. If you put the send-message logic in an Effect, it would send messages at random times. Getting this distinction wrong is one of the most common sources of React bugs.

### Purposes

- To ensure that logic runs at the correct time: either automatically on render (Effect) or in response to user action (event handler).
- To prevent unintended executions of user-initiated actions.
- To ensure that necessary synchronisation happens regardless of which interaction caused the component to appear.
- To keep event handlers focused on specific user intents and Effects focused on external synchronisation.
- To make the code's intent clear to future maintainers.

### Syntax Rules and Structure

**General Syntax for an Event Handler:**
```javascript
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // Event handler: runs ONLY when the user clicks "Send"
  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <button onClick={handleSendClick}>Send</button>
  );
}
```

**Component Breakdown:**
- `handleSendClick`: A nested function inside the component.
- `onClick={handleSendClick}`: Attaches the handler to the button's click event.
- `sendMessage(message)`: Executes only when the button is clicked.

**General Syntax for an Effect:**
```javascript
function ChatRoom({ roomId }) {
  useEffect(() => {
    // Effect: runs whenever the component is displayed or roomId changes
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

**Component Breakdown:**
- `useEffect(() => { ... }, [roomId])`: Registers the Effect.
- `createConnection(serverUrl, roomId)`: Establishes the connection.
- `return () => { connection.disconnect() }`: Cleanup disconnects the connection.

**Syntax Rules:**
- Event handlers are defined as nested functions inside the component and attached to DOM events via `onClick`, `onChange`, `onSubmit`, etc.
- Effects are registered with `useEffect` and their dependencies listed in the dependency array.
- Never put user-initiated logic (sending a message, submitting a form) inside an Effect.
- Never put render-caused synchronisation (connecting to a server, setting up a subscription) inside an event handler.
- If logic is caused by *both* rendering and user interaction, separate it: put the render-caused part in an Effect and the interaction-caused part in an event handler.

**Constraints and Limitations:**
- Effects cannot know *which* interaction caused the component to render, so they cannot distinguish between different user actions.
- Event handlers capture values from the render in which they were created; if they read state that has changed, they may read a stale value unless the component has re-rendered.
- Effects run after every commit by default; without a dependency array, they run on every render.
- In React 19, the `useEffectEvent` hook provides a way to extract non-reactive logic from Effects, solving the "stale closure" problem for callbacks that need the latest props and state.

### Annotated Code Example: Chat Room with Both Patterns

```javascript
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ✅ EVENT HANDLER: Sending a message is caused by the user clicking "Send"
  function handleSendClick() {
    sendMessage(message); // Only runs when the button is clicked
    setMessage('');       // Clear the input after sending
  }

  // ✅ EFFECT: Connecting to the room is caused by the component being displayed
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect(); // Runs whenever roomId changes or component mounts

    return () => {
      connection.disconnect(); // Cleanup: disconnect when roomId changes or unmounts
    };
  }, [roomId]); // Re-run when roomId changes

  return (
    <>
      <input
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Type a message..."
      />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

**Expected Output:** The component renders an input field and a "Send" button. When the user types and clicks "Send", the message is sent and the input is cleared. Separately, whenever the component mounts or `roomId` changes, a connection is established to the corresponding chat room, and the previous connection is disconnected.

**Why This Output Occurs:** The `handleSendClick` function is an event handler—it only runs when the button is clicked. The `useEffect` is an Effect—it runs after render whenever `roomId` changes. Placing the connection logic in the Effect ensures that the component stays synchronised with the selected room regardless of how the user arrived at the room. Placing the send logic in the event handler ensures that messages are only sent when the user explicitly clicks "Send".

### Real-World Cases

- **E-commerce checkout:** Submitting an order is an event handler (triggered by a button click); loading the user's saved address is an Effect (triggered by the checkout page rendering).
- **Dashboard analytics:** Logging a page view on mount is an Effect; logging a button click is an event handler.
- **Search interface:** Fetching search results as the user types is an Effect (the query is reactive); triggering a search on "Enter" is an event handler.
- **Real-time notifications:** Setting up a WebSocket subscription on mount is an Effect; marking a notification as read on click is an event handler.

### References

- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – Separating Events from Effects: https://react.dev/learn/separating-events-from-effects
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect

---

## Core Concept 2: Avoiding Unnecessary Effects

### Definitions

**Core Definition:** Avoiding unnecessary Effects is the practice of identifying and eliminating `useEffect` calls that do not synchronise with an external system, replacing them with direct calculations during render, event handlers, or other idiomatic React patterns.

**Technical Definition:** Effects are an "escape hatch" for stepping outside React to synchronise with external systems such as browser APIs, third-party widgets, or network requests. When there is no external system involved—for example, when updating state based on props or state, transforming data for rendering, or handling user events—an Effect is usually unnecessary and harmful. Unnecessary Effects add extra render passes, complicate the dependency array, and introduce opportunities for bugs such as infinite loops and stale closures. Removing them makes code simpler, faster, and less error-prone.

**Beginner-Friendly Explanation:** When you first learn about `useEffect`, it's tempting to use it for everything: "I need to update this value when that value changes? I'll use an Effect!" But often, you can just calculate the value directly while rendering. For example, if you have `firstName` and `lastName`, you don't need an Effect to create `fullName`—you can just write `const fullName = firstName + ' ' + lastName`. Using an Effect for this causes an extra render and can lead to stale data. The rule is: if you're not talking to something *outside* of React (like a server, a browser API, or a third-party library), you probably don't need an Effect.

### Purposes

- To eliminate redundant state and the extra render passes caused by state-syncing Effects.
- To prevent infinite loops caused by Effects that set state which triggers themselves.
- To simplify code by removing unnecessary dependency arrays.
- To improve performance by avoiding unnecessary recalculations.
- To make component logic easier to follow and reason about.

### Syntax Rules and Structure

**Pattern 1: Transforming Data for Rendering (Calculate During Render)**

```javascript
// ❌ Unnecessary Effect: stores derived data in state and syncs with an Effect
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// ✅ Correct: calculate during render
const fullName = firstName + ' ' + lastName;
```

**Component Breakdown:**
- `const fullName = firstName + ' ' + lastName`: Derived value computed directly during the render pass.
- No state, no Effect, no dependency array needed.

**Pattern 2: Handling User Events (Use Event Handlers)**

```javascript
// ❌ Unnecessary Effect: reacting to a button click via state + Effect
const [submitted, setSubmitted] = useState(false);
useEffect(() => {
  if (submitted) {
    post('/api/register');
    setSubmitted(false);
  }
}, [submitted]);

// ✅ Correct: handle the event directly
function handleSubmit() {
  post('/api/register');
}
```

**Component Breakdown:**
- `handleSubmit`: An event handler that directly performs the action.
- `onClick={handleSubmit}`: Attaches the handler to the button.

**Pattern 3: Resetting State on Prop Change (Use key)**

```javascript
// ❌ Unnecessary Effect: resetting state when userId changes
useEffect(() => {
  setComment('');
}, [userId]);

// ✅ Correct: use key to reset the component's state
<Profile userId={userId} key={userId} />
```

**Component Breakdown:**
- `key={userId}`: Passing a different key causes React to treat the component as a new instance, resetting all its state.

**Syntax Rules:**
- Before writing an Effect, ask: "Is this synchronising with an external system?" If not, do not use an Effect.
- For derived data, calculate it during render. Use `useMemo` only if the calculation is genuinely expensive.
- For user-initiated actions, use event handlers.
- For resetting state when a prop changes, use the `key` prop instead of an Effect.
- For sharing logic between event handlers, extract a shared function rather than using an Effect.
- For chains of computations, calculate what you can during render and set the rest in the event handler.

**Constraints and Limitations:**
- Some logic genuinely requires an Effect: connecting to external systems, subscriptions, timers, and browser API interactions.
- The `key` approach resets *all* state in the component and its children, which may not be desired in every case.
- `useMemo` is a performance optimisation, not a semantic guarantee; React may discard memoised values.
- The React Compiler (in development) can automatically memoise expensive calculations, reducing the need for manual `useMemo`.

### Annotated Code Example: Before and After Removing an Unnecessary Effect

**❌ Before (with unnecessary Effect):**

```javascript
import { useState, useEffect } from 'react';

function ProductList({ products, filterText }) {
  const [filteredProducts, setFilteredProducts] = useState([]);

  // ❌ Unnecessary Effect: syncing derived data to state
  useEffect(() => {
    setFilteredProducts(
      products.filter(p =>
        p.name.toLowerCase().includes(filterText.toLowerCase())
      )
    );
  }, [products, filterText]);

  return (
    <ul>
      {filteredProducts.map(p => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

**✅ After (calculated during render):**

```javascript
import { useState, useMemo } from 'react';

function ProductList({ products, filterText }) {
  // ✅ Derived during render — no Effect, no extra state
  const filteredProducts = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(filterText.toLowerCase())
    ),
    [products, filterText]
  );

  return (
    <ul>
      {filteredProducts.map(p => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** Both versions render the same filtered list of products. However, the "before" version triggers an extra render: once when `products` or `filterText` changes (which updates `filteredProducts` state), and again when `filteredProducts` updates. The "after" version calculates the filtered list during the initial render, avoiding the extra render cycle.

**Why This Output Occurs:** In the "before" version, the Effect runs after the render, calls `setFilteredProducts`, and triggers a second render. This is wasteful. In the "after" version, `filteredProducts` is computed directly during render (and memoised with `useMemo` to avoid recalculating when dependencies haven't changed). The result is the same UI with one fewer render pass and no risk of stale state.

### Real-World Cases

- **Shopping cart total:** Calculating the total price from cart items during render instead of using an Effect to sync a total state variable.
- **Form validation:** Deriving whether a form is valid during render (`const isValid = email.includes('@') && password.length >= 8`) instead of using an Effect to set a `isValid` state.
- **User profile display:** Deriving the user's display name from `firstName` and `lastName` during render.
- **Filtered lists:** Filtering an array based on a search query during render rather than in an Effect.
- **Theme application:** Applying a CSS class based on a theme state during render rather than in an Effect.

### References

- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects

---

## Core Concept 3: Effect Dependency Correctness

### Definitions

**Core Definition:** Effect dependency correctness is the practice of including every reactive value that an Effect reads inside its dependency array, ensuring that React re-runs the Effect whenever any of those values change, and preventing stale closures.

**Technical Definition:** The dependency array is the second argument to `useEffect`. It tells React which values the Effect depends on. React compares the values in the array between renders using `Object.is`; if any value has changed, the Effect re-runs (after running the cleanup function from the previous run). A "reactive value" is any value that can change between renders: props, state, context, and variables derived from them. If a reactive value used inside the Effect is omitted from the dependency array, the Effect will "capture" an outdated value—a stale closure—leading to bugs where the Effect operates on old data. The `eslint-plugin-react-hooks` `exhaustive-deps` rule automatically validates that all reactive values are included.

**Beginner-Friendly Explanation:** When you write a `useEffect`, you often use values from outside the Effect—like a state variable or a prop. React needs to know: "If this value changes, should I re-run the Effect?" The dependency array is where you list those values. If you forget to list one, React won't re-run the Effect when that value changes, and your Effect will keep using the *old* value. This is called a "stale closure." The fix is simple: list everything the Effect uses that can change between renders.

### Purposes

- To ensure the Effect re-runs at the correct times when its reactive dependencies change.
- To prevent stale closures where the Effect uses outdated values.
- To make the Effect's dependencies explicit and understandable.
- To allow React to optimise by skipping unnecessary Effect executions.
- To comply with the `exhaustive-deps` lint rule and avoid subtle bugs.

### Syntax Rules and Structure

**General Syntax:**
```javascript
useEffect(() => {
  // Effect code that reads `prop`, `stateValue`, and `derivedValue`
}, [prop, stateValue, derivedValue]);
```

**Component Breakdown:**
- `[prop, stateValue, derivedValue]`: The dependency array. Every reactive value read inside the Effect must be listed.
- `Object.is`: React uses this comparison to determine whether a dependency has changed.
- If the array is empty (`[]`), the Effect runs only once on mount (and cleanup on unmount).

**Common Violations (from the exhaustive-deps rule):**

```javascript
// ❌ Missing dependency: `count` is used but not listed
useEffect(() => {
  console.log(count);
}, []); // Missing 'count'

// ❌ Missing prop: `userId` is used but not listed
useEffect(() => {
  fetchUser(userId);
}, []); // Missing 'userId'

// ❌ Incomplete dependencies: `sortOrder` is used but not listed
useMemo(() => {
  return items.sort(sortOrder);
}, [items]); // Missing 'sortOrder'
```

**Correct Examples:**

```javascript
// ✅ All dependencies included
useEffect(() => {
  console.log(count);
}, [count]);

// ✅ All dependencies included
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

**Syntax Rules:**
- Include every reactive value used inside the Effect in the dependency array.
- Do not "trick" the linter by omitting dependencies you don't want to trigger re-runs; instead, restructure the code.
- Functions defined inside the component that are used in the Effect should be either included in the dependency array (and memoised with `useCallback`) or defined *inside* the Effect.
- Objects and arrays created inside the component are new references on every render; including them in the dependency array will cause the Effect to run on every render. Use `useMemo` to stabilise them or move them inside the Effect.
- If an Effect genuinely needs to run only once, use a ref guard instead of lying to the linter with an empty array.

**Constraints and Limitations:**
- `Object.is` comparison means that two objects with the same content but different references are considered different.
- Functions and objects created during render are unstable and will cause the Effect to re-run on every render if included in the dependency array.
- Adding a missing dependency can sometimes cause an infinite loop if the dependency is a new object or function created on every render.
- The `exhaustive-deps` rule can be disabled, but this is strongly discouraged and often leads to bugs.

### Annotated Code Examples

**Example 1: Fixing a Missing Dependency**

```javascript
import { useState, useEffect } from 'react';

function UserGreeting({ userId }) {
  const [user, setUser] = useState(null);

  // ❌ Before: missing dependency causes stale closure
  // useEffect(() => {
  //   fetchUser(userId).then(setUser);
  // }, []); // 'userId' is missing — only fetches on mount

  // ✅ After: include userId so the Effect re-runs when it changes
  useEffect(() => {
    let ignore = false;

    fetchUser(userId).then(data => {
      if (!ignore) setUser(data);
    });

    return () => {
      ignore = true; // Prevent state update on unmounted component
    };
  }, [userId]); // ✅ Correct: userId is listed

  if (!user) return <p>Loading...</p>;
  return <p>Hello, {user.name}!</p>;
}
```

**Expected Output:** When `userId` changes, the component displays "Loading..." briefly, then fetches and displays the new user's name. Without the dependency, it would continue showing the old user's data even after `userId` changed.

**Why This Output Occurs:** With `[userId]` in the dependency array, React re-runs the Effect whenever `userId` changes. The cleanup function sets `ignore = true`, preventing the previous request's result from updating state if it resolves after the new request. Without the dependency, the Effect would only run on mount, and changing `userId` would have no effect.

**Example 2: Stabilising a Function Dependency**

```javascript
import { useState, useEffect, useCallback } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // ❌ Without useCallback: `performSearch` is a new function on every render
  // function performSearch() {
  //   fetch(`/api/search?q=${query}`).then(r => r.json()).then(setResults);
  // }

  // ✅ With useCallback: stable function reference
  const performSearch = useCallback(() => {
    fetch(`/api/search?q=${query}`)
      .then(r => r.json())
      .then(setResults);
  }, [query]); // Re-created only when query changes

  useEffect(() => {
    performSearch();
  }, [performSearch]); // ✅ Stable dependency

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**Expected Output:** The component renders a search input. As the user types, the `query` state changes, `performSearch` is re-created (because `query` changed), and the Effect re-runs, fetching new search results.

**Why This Output Occurs:** Without `useCallback`, `performSearch` would be a new function on every render, causing the Effect to run on every render. With `useCallback` and `[query]` as its dependency, `performSearch` is only re-created when `query` changes, so the Effect only re-runs when the query actually changes.

### Real-World Cases

- **Search autocomplete:** The Effect depends on the query string; every keystroke changes the query, re-running the search.
- **Chat room connection:** The Effect depends on `roomId`; switching rooms disconnects from the old room and connects to the new one.
- **Polling with interval:** The Effect depends on the polling interval value; changing the interval clears the old timer and sets a new one.
- **Document title updates:** The Effect depends on the page title string; changing the title updates the document title.
- **LocalStorage persistence:** The Effect depends on the state value being persisted; changing the value updates `localStorage`.

### References

- React Official Documentation – exhaustive-deps ESLint Rule: https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React Official Documentation – Removing Effect Dependencies: https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – Lifecycle of Reactive Effects: https://react.dev/learn/lifecycle-of-reactive-effects

---

## Core Concept 4: Derived Data Without Effects

### Definitions

**Core Definition:** Derived data without Effects is the practice of computing values directly during the render pass from existing props and state, rather than storing them in separate state and synchronising them with a `useEffect`.

**Technical Definition:** In React, state should contain only the *minimal* data necessary to represent the UI. Any value that can be computed from existing props or state is "derived data" and should be calculated during render. Storing derived data in state and syncing it with an Effect is an anti-pattern that causes: (1) an extra render pass (the Effect runs after the first render, sets state, and triggers a second render), (2) potential state drift (the derived state may become out of sync with its source), and (3) unnecessary complexity. For expensive calculations, `useMemo` can memoise the result so it is only recalculated when its dependencies change.

**Beginner-Friendly Explanation:** Imagine you have a `firstName` and a `lastName`, and you want to display the full name. You *could* create a `fullName` state variable and use an Effect to update it whenever `firstName` or `lastName` changes. But that's like writing down the answer to 2+2 on a sticky note and updating the sticky note whenever you see different numbers—instead of just calculating 2+2 when you need it. Derived data should be calculated on the spot, during render.

### Purposes

- To eliminate redundant state and the extra render passes it causes.
- To prevent state drift, where derived state becomes inconsistent with its source.
- To simplify component logic by removing unnecessary Effects and dependency arrays.
- To improve performance by avoiding unnecessary state updates.
- To make the component's data flow explicit and easy to reason about.

### Syntax Rules and Structure

**Pattern 1: Simple Derivation (Calculate During Render)**

```javascript
function UserProfile({ firstName, lastName }) {
  // ✅ Derived during render — no state, no Effect
  const fullName = firstName + ' ' + lastName;

  return <h1>{fullName}</h1>;
}
```

**Component Breakdown:**
- `const fullName = firstName + ' ' + lastName`: Computed directly during the render pass.
- No `useState`, no `useEffect`, no dependency array.

**Pattern 2: Expensive Derivation (Use useMemo)**

```javascript
import { useMemo } from 'react';

function TodoList({ todos, filter }) {
  // ✅ Memoised derivation — only recalculates when todos or filter change
  const visibleTodos = useMemo(
    () => todos.filter(todo => {
      if (filter === 'active') return !todo.completed;
      if (filter === 'completed') return todo.completed;
      return true;
    }),
    [todos, filter]
  );

  return (
    <ul>
      {visibleTodos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- `useMemo(() => ..., [todos, filter])`: Memoises the filtered list, recalculating only when `todos` or `filter` changes.
- The filter logic runs during render, not in an Effect.

**Pattern 3: Resetting State on Prop Change (Use key)**

```javascript
// ❌ Unnecessary Effect: resetting comment when userId changes
useEffect(() => {
  setComment('');
}, [userId]);

// ✅ Correct: use key to reset state
<Profile userId={userId} key={userId} />
```

**Component Breakdown:**
- `key={userId}`: When `userId` changes, React unmounts the old `Profile` instance and mounts a new one, resetting all its state.

**Pattern 4: Adjusting State on Prop Change (Conditional setState During Render)**

```javascript
function List({ items }) {
  const [selection, setSelection] = useState(null);
  const [prevItems, setPrevItems] = useState(items);

  // ✅ Adjust state during render when a prop changes
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null); // Reset selection when the items list changes
  }

  // ... render logic
}
```

**Component Breakdown:**
- `if (items !== prevItems)`: Compares the current prop to the previous prop.
- `setPrevItems(items)`: Updates the stored previous prop.
- `setSelection(null)`: Resets the selection state during render (React will re-render immediately).

**Syntax Rules:**
- If a value can be computed from current props or state, do not store it in state or update it in an Effect. Derive it during render.
- Use `useMemo` only for genuinely expensive calculations; measure with `console.time` before memoising.
- Do not set state in Effects solely in response to prop changes; prefer derived values or keyed resets.
- When resetting all state on a prop change, use the `key` prop.
- When adjusting *some* state on a prop change, use conditional `setState` during render (guarded by a comparison with the previous prop).

**Constraints and Limitations:**
- `useMemo` is a performance optimisation, not a semantic guarantee; React may discard memoised values in the future.
- Conditional `setState` during render must be guarded by a condition that compares the current prop to a stored previous prop, or it will cause an infinite render loop.
- The `key` approach resets *all* state in the component and its children, which may not be desired if only some state needs resetting.
- The React Compiler (experimental) can automatically memoise expensive calculations, reducing the need for manual `useMemo`.

### Annotated Code Examples

**Example 1: Deriving Filtered Data During Render**

```javascript
import { useState, useMemo } from 'react';

function ProductCatalog({ products, category }) {
  const [searchQuery, setSearchQuery] = useState('');

  // ✅ Derived during render: filter products by category and search query
  const filteredProducts = useMemo(() => {
    return products
      .filter(p => category === 'all' || p.category === category)
      .filter(p => p.name.toLowerCase().includes(searchQuery.toLowerCase()));
  }, [products, category, searchQuery]);

  return (
    <div>
      <input
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Search products..."
      />
      <p>{filteredProducts.length} products found</p>
      <ul>
        {filteredProducts.map(p => (
          <li key={p.id}>{p.name} — ${p.price}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** The component renders a search input and a list of products. As the user types in the search box, the list filters in real time. The product count updates accordingly.

**Why This Output Occurs:** The `filteredProducts` value is computed during render (memoised with `useMemo`). When `searchQuery` changes, the component re-renders, `useMemo` recalculates the filtered list, and the UI updates. No Effect, no extra state, no extra render pass.

**Example 2: Resetting State with key vs. Effect**

```javascript
import { useState, useEffect } from 'react';

// ❌ Version 1: Resetting state with an Effect
function ProfileWithEffect({ userId }) {
  const [comment, setComment] = useState('');

  useEffect(() => {
    setComment(''); // Reset comment when userId changes
  }, [userId]);

  return (
    <div>
      <p>User: {userId}</p>
      <input
        value={comment}
        onChange={(e) => setComment(e.target.value)}
        placeholder="Leave a comment..."
      />
    </div>
  );
}

// ✅ Version 2: Resetting state with key
function ProfileWithKey({ userId }) {
  const [comment, setComment] = useState('');

  return (
    <div>
      <p>User: {userId}</p>
      <input
        value={comment}
        onChange={(e) => setComment(e.target.value)}
        placeholder="Leave a comment..."
      />
    </div>
  );
}

function App() {
  const [userId, setUserId] = useState(1);

  return (
    <div>
      <button onClick={() => setUserId(u => u === 1 ? 2 : 1)}>
        Switch User
      </button>
      {/* ✅ Use key to reset ProfileWithKey's state when userId changes */}
      <ProfileWithKey userId={userId} key={userId} />
    </div>
  );
}
```

**Expected Output:** Both versions reset the comment input when the user is switched. However, the Effect version triggers an extra render (the Effect runs after the render, calls `setComment`, and triggers a second render). The `key` version resets the state immediately without an extra render.

**Why This Output Occurs:** In Version 1, changing `userId` causes a render with the *old* comment value, then the Effect runs and clears it, causing a second render with an empty comment. In Version 2, changing `userId` changes the `key` of `ProfileWithKey`, causing React to unmount the old instance (discarding its state) and mount a new instance with a fresh `comment` state of `''`.

### Real-World Cases

- **E-commerce filters:** Deriving the filtered product list from the full list and the selected filters during render.
- **Dashboard totals:** Calculating totals, averages, and percentages from raw data during render.
- **Form validation:** Deriving whether a form is valid (`isValid = email.includes('@') && password.length >= 8`) during render.
- **User display names:** Deriving `displayName` from `firstName` and `lastName` during render.
- **Theme styling:** Deriving CSS classes from a theme state value during render.

### References

- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Official Documentation – useMemo Reference: https://react.dev/reference/react/useMemo
- React Official Documentation – Preserving and Resetting State: https://react.dev/learn/preserving-and-resetting-state

---

## Core Concept 5: Effect Cleanup

### Definitions

**Core Definition:** Effect cleanup is the mechanism by which `useEffect` releases resources—timers, event listeners, network connections, subscriptions, and third-party instances—that were created during the Effect's execution, preventing memory leaks and state updates on unmounted components.

**Technical Definition:** The `useEffect` Hook accepts an optional return value: a cleanup function. React calls this function before the component is removed from the DOM (unmount) and before the next execution of the Effect (when dependencies change). The cleanup function runs in the same order as the Effects were defined. It cannot be async. The cleanup function is essential for preventing memory leaks, avoiding the "Can't perform a React state update on an unmounted component" warning, and ensuring that external resources are properly released. Common cleanup tasks include clearing timers (`clearTimeout`, `clearInterval`), removing event listeners (`removeEventListener`), cancelling network requests (`AbortController.abort()`), closing connections (`WebSocket.close()`), and destroying third-party instances (`chart.destroy()`).

**Beginner-Friendly Explanation:** When you start something in a `useEffect`—like a timer, an event listener, or a connection to a server—you need to stop it when your component is no longer needed. If you don't, those things keep running in the background, wasting memory and potentially causing bugs. React gives you a way to "clean up" after yourself by returning a function from `useEffect`. React calls this function automatically when the component disappears or when the Effect needs to run again.

### Purposes

- To prevent memory leaks caused by lingering timers, listeners, and connections.
- To avoid state updates on unmounted components.
- To ensure that external resources (network connections, subscriptions) are released.
- To maintain application performance by preventing unnecessary background work.
- To comply with React's requirement that Effects must be idempotent in StrictMode.

### Syntax Rules and Structure

**General Syntax:**
```javascript
useEffect(() => {
  // Setup: create resources
  const resource = createResource();

  // Return cleanup function
  return () => {
    // Cleanup: release resources
    resource.destroy();
  };
}, [dependencies]);
```

**Component Breakdown:**
- `useEffect(() => { ... }, [dependencies])`: The Effect Hook with a dependency array.
- Setup code: Runs after render when dependencies change or on mount.
- `return () => { ... }`: The cleanup function, called before the next Effect run and on unmount.
- `resource.destroy()`: Releases the resource (e.g., `clearTimeout`, `removeEventListener`, `socket.close()`).

**Syntax Rules:**
- The cleanup function must be returned from the Effect callback, not called directly.
- The cleanup function should mirror the setup: if you add a listener, remove it; if you create a timer, clear it; if you open a connection, close it.
- Do not include async operations in the cleanup function without careful handling.
- The cleanup function runs on every dependency change, not just on unmount.
- Multiple Effects should be used for unrelated concerns rather than combining them into one Effect with a complex cleanup.
- If the Effect creates multiple resources, the cleanup function should dispose of all of them.

**Constraints and Limitations:**
- The cleanup function cannot be async; if you need to await something during cleanup, you must handle it separately.
- In StrictMode, the cleanup function runs immediately after the Effect in development, which can cause issues if the cleanup is not idempotent.
- Some resources (e.g., `AbortController`) are not supported in older environments.
- Cleanup does not run when the browser tab is closed; use `beforeunload` for those scenarios.
- If the cleanup function throws an error, React will log it but will not prevent the component from unmounting.

### Annotated Code Examples

**Example 1: Cleanup for Multiple Resource Types**

```javascript
import { useState, useEffect, useRef } from 'react';

function Dashboard({ userId }) {
  const [data, setData] = useState(null);
  const [onlineStatus, setOnlineStatus] = useState(navigator.onLine);
  const socketRef = useRef(null);

  // Effect 1: Fetch data with AbortController
  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        const response = await fetch(
          `https://api.example.com/users/${userId}`,
          { signal: controller.signal }
        );
        const result = await response.json();
        setData(result);
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      }
    }

    fetchData();

    return () => {
      controller.abort(); // ✅ Cancel the fetch request
    };
  }, [userId]);

  // Effect 2: Online/offline event listeners
  useEffect(() => {
    function handleOnline() { setOnlineStatus(true); }
    function handleOffline() { setOnlineStatus(false); }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  // Effect 3: WebSocket connection
  useEffect(() => {
    const socket = new WebSocket(`wss://api.example.com/feed/${userId}`);
    socketRef.current = socket;

    socket.onmessage = (event) => {
      setData(prev => ({ ...prev, liveUpdate: event.data }));
    };

    return () => {
      socket.close(); // ✅ Close the WebSocket connection
      socketRef.current = null;
    };
  }, [userId]);

  return (
    <div>
      <h1>Dashboard</h1>
      <p>Status: {onlineStatus ? 'Online' : 'Offline'}</p>
      <p>User: {userId}</p>
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
}
```

**Expected Output:** The component displays user data, online/offline status, and live updates. When `userId` changes, the old fetch request is aborted, the old WebSocket is closed, and new resources are created. When the component unmounts, all three Effects run their cleanup functions, releasing all resources.

**Why This Output Occurs:** Each Effect is responsible for one concern and has its own cleanup function. The fetch Effect uses `AbortController` to cancel the request. The online/offline Effect removes its listeners. The WebSocket Effect closes the connection. This separation of concerns ensures that each resource is cleaned up correctly and independently.

**Example 2: Visualising Cleanup Order in StrictMode**

```javascript
import { useEffect } from 'react';

function CleanupDemo() {
  useEffect(() => {
    console.log('Effect 1: Setup');
    return () => console.log('Effect 1: Cleanup');
  }, []);

  useEffect(() => {
    console.log('Effect 2: Setup');
    return () => console.log('Effect 2: Cleanup');
  }, []);

  return <p>Check the console</p>;
}
```

**Expected Output (in development with StrictMode):**
```
Effect 1: Setup
Effect 2: Setup
Effect 1: Cleanup
Effect 2: Cleanup
Effect 1: Setup
Effect 2: Setup
```

When the component unmounts:
```
Effect 1: Cleanup
Effect 2: Cleanup
```

**Why This Output Occurs:** In StrictMode, React runs the setup, then immediately runs the cleanup, then runs the setup again. This is intentional: it helps developers detect missing cleanup logic. The cleanup functions run in the order the Effects were defined. On unmount, both cleanup functions run in the same order.

### Real-World Cases

- **SPA navigation:** Cleaning up listeners and connections when the user navigates between routes.
- **Modal components:** Removing keydown listeners and clearing timers when a modal closes.
- **Data fetching components:** Aborting in-flight requests when the component unmounts.
- **Real-time dashboards:** Closing WebSocket connections when the user leaves the dashboard.
- **Animation components:** Reverting GSAP contexts or cancelling `requestAnimationFrame` loops.

### References

- React Official Documentation – Synchronizing with Effects (Cleanup): https://react.dev/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed
- React Official Documentation – Lifecycle of Reactive Effects: https://react.dev/learn/lifecycle-of-reactive-effects
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController

---

## Core Concept 6: Race Conditions

### Definitions

**Core Definition:** Race conditions in React Effects occur when multiple asynchronous operations are triggered by rapid dependency changes, and an earlier operation resolves *after* a later one, causing stale data to overwrite fresh data.

**Technical Definition:** React does not cancel in-flight promises when an Effect re-runs or the component unmounts. If a slow request from an earlier render resolves after a faster request from a later render, the stale result can overwrite the fresh one unless the Effect explicitly guards against it. This is the documented root cause of "data flashes to a previous value when switching quickly." The standard solution is a boolean `ignore` flag set in the cleanup function and checked before every state update derived from the async result. `AbortController` is an equally valid, and more resource-efficient, variant when the underlying request API supports cancellation—it actually cancels the network request instead of only ignoring its result.

**Beginner-Friendly Explanation:** Imagine you're searching for something online. You type "cat", then quickly change your mind and type "dog". Both searches are sent to the server. But what if the "cat" search is slow and comes back *after* the "dog" search? Your screen would show cat results even though you asked for dogs. That's a race condition. React can't automatically cancel the first request, so you need to add a "guard" that says: "If this result is from an old search, ignore it."

### Purposes

- To prevent stale asynchronous results from overwriting fresh data.
- To ensure that the UI always reflects the most recent dependency values.
- To handle rapid dependency changes (fast navigation, fast typing, quick clicks) safely.
- To avoid the "data flashes to a previous value" bug.
- To make asynchronous Effects robust and production-safe.

### Syntax Rules and Structure

**Pattern 1: Ignore Flag (Official React Pattern)**

```javascript
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json); // Only update state if not ignored
    }
  }

  startFetching();

  return () => {
    ignore = true; // Mark this Effect's result as stale
  };
}, [userId]);
```

**Component Breakdown:**
- `let ignore = false`: A closure variable scoped to this Effect run.
- `if (!ignore) { setTodos(json) }`: Guards the state update.
- `return () => { ignore = true }`: The cleanup sets the flag, telling the async operation to discard its result.

**Pattern 2: AbortController (Resource-Efficient Variant)**

```javascript
useEffect(() => {
  const controller = new AbortController();

  async function fetchData() {
    try {
      const response = await fetch(`/api/items/${itemId}`, {
        signal: controller.signal,
      });
      const data = await response.json();
      setItems(data); // No guard needed: if aborted, fetch throws
    } catch (err) {
      if (err.name !== 'AbortError') {
        setError(err);
      }
    }
  }

  fetchData();

  return () => {
    controller.abort(); // Cancels the actual network request
  };
}, [itemId]);
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller.
- `{ signal: controller.signal }`: Passes the signal to `fetch`.
- `controller.abort()`: Cancels the request during cleanup.
- `if (err.name !== 'AbortError')`: Ignores the expected abort error.

**Syntax Rules:**
- Every async Effect that calls `setState` from a resolved promise must have a guard (ignore flag or AbortController). No exceptions.
- The guard must be checked immediately before *every* state update inside the async chain, not just the first one.
- An Effect that fetches and then makes a second dependent call must check `ignore` (or `signal.aborted`) before each `setState`.
- Prefer `AbortController` when the underlying request API supports cancellation; it actually cancels the network request, saving bandwidth and server resources.
- Use the ignore flag when the request API does not support cancellation or when the codebase already has an established ignore-flag convention.

**Constraints and Limitations:**
- The ignore flag prevents state updates but does *not* cancel the network request; the request continues to consume bandwidth and server resources.
- `AbortController` is not supported in all environments (though support is now widespread).
- In StrictMode, the Effect runs twice in development, which can trigger two requests; the cleanup must handle this gracefully.
- Race conditions are intermittent and hard to reproduce; the absence of a guard is a bug regardless of how unlikely the race seems.

### Annotated Code Examples

**Example 1: Search with Race Condition Guard**

```javascript
import { useState, useEffect } from 'react';

function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let ignore = false; // ✅ Guard flag for this Effect run

    async function fetchResults() {
      setLoading(true);
      try {
        const response = await fetch(`/api/search?q=${query}`);
        const data = await response.json();
        if (!ignore) {
          setResults(data);
        }
      } catch (error) {
        if (!ignore) {
          console.error('Search failed:', error);
        }
      } finally {
        if (!ignore) {
          setLoading(false);
        }
      }
    }

    fetchResults();

    return () => {
      ignore = true; // ✅ Mark this Effect's result as stale
    };
  }, [query]);

  return (
    <div>
      {loading && <p>Searching...</p>}
      <ul>
        {results.map(r => (
          <li key={r.id}>{r.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** As the user types a query, the component displays "Searching..." and then the results. If the user types quickly and a previous search resolves after a newer one, the stale results are ignored and only the latest search results are displayed.

**Why This Output Occurs:** Each time `query` changes, the Effect re-runs. The cleanup function from the previous Effect sets `ignore = true`, so when the previous request resolves, the `if (!ignore)` check prevents it from calling `setResults` with stale data. The new Effect's `ignore` is `false`, so its results are applied.

**Example 2: Item Switching with AbortController**

```javascript
import { useState, useEffect } from 'react';

function ItemDetail({ itemId }) {
  const [item, setItem] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController(); // ✅ Cancel actual request

    async function fetchItem() {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(`/api/items/${itemId}`, {
          signal: controller.signal,
        });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        const data = await response.json();
        setItem(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchItem();

    return () => {
      controller.abort(); // ✅ Cancel the request on cleanup
    };
  }, [itemId]);

  if (loading) return <p>Loading item...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!item) return null;

  return (
    <div>
      <h1>{item.name}</h1>
      <p>{item.description}</p>
    </div>
  );
}
```

**Expected Output:** When the user switches between items quickly, the previous item's request is cancelled and only the latest item's details are displayed. The "Loading item..." message appears briefly during each fetch.

**Why This Output Occurs:** The `AbortController` cancels the previous request when `itemId` changes. The `catch` block checks `err.name !== 'AbortError'` to avoid setting an error state for expected cancellations. The `finally` block sets `loading` to `false`, but note that this runs even for aborted requests; in practice, a guard may be needed to prevent the loading state from being set to `false` for an old request that was aborted. A more robust version would check `controller.signal.aborted` before setting state in the `finally` block.

### Real-World Cases

- **Search autocomplete:** Rapid typing triggers multiple searches; the latest query's results should win.
- **Tab switching:** Switching between tabs or pages quickly causes multiple data fetches; only the latest tab's data should be displayed.
- **Pagination:** Clicking "Next" rapidly triggers multiple page fetches; only the latest page's data should be shown.
- **User profile switching:** Selecting different users in a list; the latest user's profile should be displayed.
- **Dashboard filters:** Changing filters quickly; only the latest filter combination's data should be displayed.

### References

- React Official Documentation – Synchronizing with Effects (Race Conditions): https://react.dev/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events
- React Official Documentation – You Might Not Need an Effect (Fetching Data): https://react.dev/learn/you-might-not-need-an-effect#fetching-data
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController

---

## Core Concept 7: Request Cancellation

### Definitions

**Core Definition:** Request cancellation is the practice of using the `AbortController` API to cancel in-flight network requests when a React Effect is cleaned up, preventing memory leaks, wasted bandwidth, and state updates on unmounted components.

**Technical Definition:** `AbortController` is a Web API that provides a signal (`AbortSignal`) which can be passed to fetch requests (and other APIs) to cancel them. In React, an `AbortController` is created inside a `useEffect`, its `signal` is passed to `fetch`, and its `abort()` method is called in the cleanup function. When aborted, the `fetch` promise rejects with an `AbortError`, which can be caught and ignored. This is more resource-efficient than the ignore-flag pattern because it actually cancels the HTTP request rather than merely ignoring its result. The `AbortController` is supported in all modern browsers and in Node.js 15+.

**Beginner-Friendly Explanation:** When your React component fetches data from a server, the request might take a while. If the user navigates away or the component disappears before the request finishes, that request is still running in the background, wasting bandwidth. `AbortController` lets you "hang up" on that request—you tell the browser, "Never mind, I don't need this anymore." This saves resources and prevents errors from trying to update a component that no longer exists.

### Purposes

- To cancel in-flight network requests when the component unmounts or dependencies change.
- To prevent memory leaks and "state update on unmounted component" warnings.
- To save bandwidth and server resources by stopping unnecessary requests.
- To prevent race conditions by cancelling superseded requests.
- To provide a cleaner alternative to the ignore-flag pattern when the request API supports cancellation.

### Syntax Rules and Structure

**General Syntax:**
```javascript
useEffect(() => {
  const controller = new AbortController();

  async function fetchData() {
    try {
      const response = await fetch(url, {
        signal: controller.signal,
      });
      const data = await response.json();
      setData(data);
    } catch (err) {
      if (err.name === 'AbortError') {
        // Request was cancelled — ignore
        return;
      }
      setError(err.message);
    }
  }

  fetchData();

  return () => {
    controller.abort(); // Cancel the request on cleanup
  };
}, [url]);
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller with a `signal` property and an `abort()` method.
- `{ signal: controller.signal }`: Passes the signal to `fetch` (or to Axios via `{ signal }`).
- `controller.abort()`: Cancels the request; the `fetch` promise rejects with an `AbortError`.
- `if (err.name === 'AbortError')`: Ignores the expected cancellation error.

**Syntax Rules:**
- Create the `AbortController` inside the `useEffect`, not outside.
- Pass `controller.signal` to the `fetch` call (or to the request library).
- Call `controller.abort()` in the cleanup function.
- Catch `AbortError` and ignore it; do not set error state for expected cancellations.
- If using Axios, pass `{ signal: controller.signal }` as a request config option, and check `axios.isCancel(error)`.
- If the request library does not support `AbortSignal`, fall back to the ignore-flag pattern.

**Constraints and Limitations:**
- `AbortController` is not supported in some older environments (though support is now widespread).
- Aborting a request may not immediately release all resources; the browser handles the actual cancellation asynchronously.
- The `AbortSignal` can only be used once; if you need to abort multiple requests, create multiple controllers or use `AbortSignal.timeout()`.
- Some polyfills or custom fetch wrappers may not forward the `signal` to the underlying `fetch`; verify that the signal is actually passed through.

### Annotated Code Examples

**Example 1: Basic AbortController with fetch**

```javascript
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUser() {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
          { signal: controller.signal }
        );

        if (!response.ok) {
          throw new Error(`HTTP error: ${response.status}`);
        }

        const data = await response.json();
        setUser(data);
      } catch (err) {
        if (err.name === 'AbortError') {
          // Request was cancelled — ignore this error
          return;
        }
        setError(err.message);
      } finally {
        // Only update loading if the request was not aborted
        if (!controller.signal.aborted) {
          setLoading(false);
        }
      }
    }

    fetchUser();

    return () => {
      controller.abort(); // ✅ Cancel the request on cleanup
    };
  }, [userId]);

  if (loading) return <p>Loading user...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Phone: {user.phone}</p>
    </div>
  );
}
```

**Expected Output:** When `userId` changes, the previous request is cancelled and a new one begins. The component displays "Loading user..." briefly, then the new user's details. If the component unmounts before the request completes, the request is cancelled and no state update occurs.

**Why This Output Occurs:** The `AbortController` is created inside the Effect, and its `signal` is passed to `fetch`. When `userId` changes (or the component unmounts), the cleanup function calls `controller.abort()`, which causes the `fetch` promise to reject with an `AbortError`. The catch block ignores this error, and the `finally` block checks `controller.signal.aborted` before updating the loading state.

**Example 2: AbortController with Axios**

```javascript
import { useState, useEffect } from 'react';
import axios from 'axios';

function PostList({ categoryId }) {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchPosts() {
      setLoading(true);

      try {
        const response = await axios.get(
          `https://jsonplaceholder.typicode.com/posts?userId=${categoryId}`,
          { signal: controller.signal } // ✅ Pass the signal to Axios
        );
        setPosts(response.data);
      } catch (error) {
        if (axios.isCancel(error)) {
          // Request was cancelled — ignore
          return;
        }
        console.error('Failed to fetch posts:', error);
      } finally {
        if (!controller.signal.aborted) {
          setLoading(false);
        }
      }
    }

    fetchPosts();

    return () => {
      controller.abort(); // ✅ Cancel the request on cleanup
    };
  }, [categoryId]);

  if (loading) return <p>Loading posts...</p>;

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** When `categoryId` changes, the previous Axios request is cancelled and a new one begins. The loading indicator appears briefly, then the new posts are displayed.

**Why This Output Occurs:** Axios supports the `AbortController` signal via the `signal` config option. When `controller.abort()` is called, Axios rejects the promise with a cancellation error, which is identified by `axios.isCancel(error)`. The `finally` block checks `controller.signal.aborted` before setting loading to `false`, preventing a stale loading state from being applied.

### Real-World Cases

- **Search autocomplete:** Cancelling the previous search request when the user types a new query.
- **Tab navigation:** Cancelling the previous tab's data request when the user switches tabs quickly.
- **Infinite scroll:** Cancelling pending page requests when the user scrolls rapidly.
- **Form submission with navigation:** Cancelling a POST request if the user navigates away before it completes.
- **Dashboard filters:** Cancelling data fetches for old filter combinations when the user changes filters rapidly.

### References

- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – AbortSignal: https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- Axios Documentation – Cancellation: https://axios-http.com/docs/cancellation
- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect

---

## References

- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – Separating Events from Effects: https://react.dev/learn/separating-events-from-effects
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Official Documentation – Lifecycle of Reactive Effects: https://react.dev/learn/lifecycle-of-reactive-effects
- React Official Documentation – Removing Effect Dependencies: https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – exhaustive-deps ESLint Rule: https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React Official Documentation – useMemo Reference: https://react.dev/reference/react/useMemo
- React Official Documentation – Preserving and Resetting State: https://react.dev/learn/preserving-and-resetting-state
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – AbortSignal: https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- MDN Web Docs – fetch(): https://developer.mozilla.org/en-US/docs/Web/API/fetch
- Axios Documentation – Cancellation: https://axios-http.com/docs/cancellation
- React 19 – useEffectEvent (Experimental): https://react.dev/reference/react/useEffectEvent