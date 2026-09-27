# React State Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React state architecture is the discipline of classifying application state into distinct categories—local, shared, server, URL, form, derived, and persistent—and assigning each category to the appropriate management tool, lifecycle, and ownership boundary.

**Technical Definition:** React state architecture refers to the systematic classification and governance of state within a React application based on its origin, scope, lifecycle, and mutation patterns. Rather than treating all state as a monolithic concern (the "one global store" anti-pattern), a scalable architecture distinguishes between: UI-local state (component-scoped ephemeral values), shared client state (cross-component client data), server state (asynchronous remote data with its own cache lifecycle), URL state (navigation-synchronized parameters), form state (transient high-frequency user inputs), derived state (computed values), and persistent state (storage-backed long-lived data). Each category has distinct failure modes, performance characteristics, and appropriate tools. The best state decision is often not between Redux and Context, but between client state versus server state, and local versus shared.

**Beginner-Friendly Explanation:** Imagine you are organising a large office. Some documents are personal notes you keep on your own desk (local state). Some are shared with your team in a common area (shared state). Some come from an external source and need to be filed with a tracking system (server state). Some are just the current page number in a manual (URL state). Some are quick scribbles on a form you are filling out (form state). Some are calculations you can do in your head instead of writing down (derived state). And some are important records you lock in a safe so they survive a power outage (persistent state). React state architecture is about knowing which document goes where—and using the right container for each.

### Key Characteristics

- **Taxonomy-Driven:** State is classified by origin (client vs. server), scope (local vs. shared), lifecycle (ephemeral vs. persistent), and mutability (direct vs. derived).
- **Tool Matching:** Each state category maps to a specific React primitive or library: `useState`/`useReducer` for local, Context/Zustand for shared, TanStack Query for server, `useSearchParams` for URL, React Hook Form for forms, render-time computation for derived, and Zustand `persist` middleware for persistent.
- **Failure-Mode Awareness:** Different state categories have different failure modes. Treating them all the same produces either a giant store (high coupling) or a jungle of contexts (uncontrolled re-renders).
- **Minimal State Principle:** Store the minimal amount of state needed, then derive everything else. Redundant and duplicate data in state leads to synchronization bugs and unnecessary re-renders.
- **Boundary Enforcement:** Architectural boundaries (domain, feature, page) determine where state should live, preventing over-coupling and maintainability degradation.

### Prerequisites

- Solid understanding of React function components, JSX, and the `useState` Hook.
- Familiarity with the component tree hierarchy and prop passing.
- Working knowledge of `useEffect`, `useReducer`, and `useContext`.
- Basic understanding of asynchronous programming (promises, `async`/`await`).
- Awareness of browser storage APIs (`localStorage`, `IndexedDB`).

### Related Programming Areas

- **State Management:** Coordinating state across components and features.
- **Caching:** Managing server-state cache lifecycles (staleness, garbage collection, invalidation).
- **Navigation:** Synchronizing UI state with the browser's address bar.
- **Form Processing:** Capturing, validating, and submitting user input.
- **Performance Optimisation:** Avoiding unnecessary re-renders and expensive recalculations.

### Core Concepts / Features

1. Local State (`useState` / `useReducer`)
2. Shared State (Context API, Zustand, Redux)
3. Server State (TanStack Query)
4. URL State (`useSearchParams`)
5. Form State (React Hook Form)
6. Derived State (Render-Time Computation)
7. Persistent State (localStorage, IndexedDB, Zustand Persist)

---

## Core Concept 1: Local State (`useState` / `useReducer`)

### Definitions

**Core Definition:** Local state is UI state that is scoped entirely within a single component and is not shared with other components, typically managed via the `useState` or `useReducer` Hooks.

**Technical Definition:** Local state is component-scoped state declared with `useState` or `useReducer` inside a function component. It is not accessible to any other component unless explicitly passed down via props or lifted to a parent. `useState` is the standard Hook for simple, independent state values (form inputs, toggles, counters). `useReducer` is an alternative Hook for state logic that involves multiple sub-values, interdependent fields, or complex transitions; it consolidates state update logic into a single reducer function outside the component. The choice between them follows a clear heuristic: use `useState` for simple, independent values; use `useReducer` when multiple state fields change together and must remain consistent, or when transitions are event-driven ("submit", "cancel", "retry").

**Beginner-Friendly Explanation:** Think of a local variable in a JavaScript function. It exists only while that function is running and is not visible anywhere else. Local state is the same idea, but it persists across renders. A toggle button that tracks whether it is "on" or "off" is local state—no other component needs to know or care. If you find that two components need the same piece of state, it is no longer local; it needs to be lifted.

### Purposes

- To track ephemeral UI values that belong to exactly one component (open/close flags, selected tabs, hover states, modal visibility).
- To manage simple form fragment values within a single component.
- To handle complex component-local workflows (multi-step forms, wizards) with `useReducer` when transitions are event-driven.
- To keep state colocated with the component that uses it, maximising cohesion and minimising coupling.
- To avoid the overhead of Context or external stores for state that does not need to be shared.

### Syntax Rules and Structure

**General Syntax (`useState`):**
```jsx
const [state, setState] = useState(initialValue);
```

**Component Breakdown:**
- `state`: The current value.
- `setState`: The function used to update the value.
- `initialValue`: The value used only on the first render.

**General Syntax (`useReducer`):**
```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

**Component Breakdown:**
- `reducer(state, action)`: A pure function that returns the next state based on the current state and an action.
- `dispatch(action)`: A function that sends an action to the reducer.
- `initialState`: The initial state object.

**Syntax Rules:**
- Use `useState` for simple, independent values (booleans, numbers, strings, small objects).
- Use `useReducer` when multiple state fields change together, when the next state depends on the previous state in complex ways, or when state transitions are event-driven.
- Never mutate state directly; always use the setter or dispatch an action.
- For object state, remember to copy the other fields when updating only one field.
- Avoid redundant state: if a value can be calculated from props or existing state during rendering, do not store it in state.

**Constraints and Limitations:**
- Local state is invisible to other components; sharing requires lifting state up or using Context/stores.
- `useState` does not provide a structured way to handle complex transitions; `useReducer` adds structure but also boilerplate.
- Overusing local state for values that should be shared leads to prop drilling or duplicated state.
- State updates are asynchronous and batched; reading state immediately after setting it will return the old value.

### Annotated Code Example: Moving Dot with `useState` vs. `useReducer`

```jsx
import { useState, useReducer } from 'react';

// --- useState version: simple independent values ---
function MovingDotUseState() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div
      onPointerMove={e => {
        setPosition({ x: e.clientX, y: e.clientY });
      }}
      style={{ position: 'relative', width: '100vw', height: '100vh' }}
    >
      <div
        style={{
          position: 'absolute',
          backgroundColor: 'red',
          borderRadius: '50%',
          transform: `translate(${position.x}px, ${position.y}px)`,
          left: -10, top: -10, width: 20, height: 20,
        }}
      />
    </div>
  );
}

// --- useReducer version: complex transitions ---
function positionReducer(state, action) {
  switch (action.type) {
    case 'moved':
      return { x: action.x, y: action.y };
    case 'reset':
      return { x: 0, y: 0 };
    default:
      throw new Error('Unknown action: ' + action.type);
  }
}

function MovingDotUseReducer() {
  const [position, dispatch] = useReducer(positionReducer, { x: 0, y: 0 });

  return (
    <div
      onPointerMove={e => {
        dispatch({ type: 'moved', x: e.clientX, y: e.clientY });
      }}
      onDoubleClick={() => dispatch({ type: 'reset' })}
      style={{ position: 'relative', width: '100vw', height: '100vh' }}
    >
      <div
        style={{
          position: 'absolute',
          backgroundColor: 'blue',
          borderRadius: '50%',
          transform: `translate(${position.x}px, ${position.y}px)`,
          left: -10, top: -10, width: 20, height: 20,
        }}
      />
    </div>
  );
}
```

**Expected Output:** Both components render a circle that follows the mouse cursor. The `useReducer` version additionally resets the dot to `(0, 0)` on double-click, demonstrating how `useReducer` makes it easy to add new actions without cluttering the component with multiple `setState` calls.

**Why This Output Occurs:** The `useState` version stores `{ x, y }` in a single state variable, which is appropriate because the two values always change together. The `useReducer` version moves the update logic into a pure reducer function, making the state transitions explicit and testable. This is the recommended pattern when state logic becomes complex or when multiple actions affect the same state.

### Real-World Cases

- **Modal visibility:** A modal component tracks its own `isOpen` state with `useState`.
- **Accordion panels:** Each panel tracks its own `isExpanded` state locally.
- **Multi-step wizard:** A wizard component uses `useReducer` to manage step transitions, validation state, and navigation actions.
- **Toggle switches:** A toggle component tracks `isOn` locally and reports changes to the parent via a callback.
- **Temporary form fragments:** A search input manages its own query string locally before submitting to the parent.

### References

- React Official Documentation – useState: https://react.dev/reference/react/useState
- React Official Documentation – useReducer: https://react.dev/reference/react/useReducer
- React Official Documentation – Extracting State Logic into a Reducer: https://react.dev/learn/extracting-state-logic-into-a-reducer
- React Official Documentation – Choosing the State Structure: https://18.react.dev/learn/choosing-the-state-structure
- Feature-Sliced Design – React State Management: https://feature-sliced.design/blog/scalable-react-state-patterns
- RTcamp – Choosing the Right React State Management Strategy: https://rtcamp.com/tutorials/react-state-management/

---

## Core Concept 2: Shared State (Context API, Zustand, Redux)

### Definitions

**Core Definition:** Shared state is client-side data that multiple independent components need to access or modify simultaneously, managed through the Context API, external stores (Zustand, Redux), or state-lifting to a common ancestor.

**Technical Definition:** Shared state is state that is read or written by more than one component, where those components may be siblings, deeply nested, or in separate branches of the component tree. React provides the Context API as a built-in mechanism for sharing data without prop drilling, but Context has performance limitations: every consumer re-renders when the provider value changes. External state-management libraries (Zustand, Redux Toolkit, Jotai) hold state outside the React tree and expose it via hooks with selector-based subscriptions, allowing fine-grained re-render control. The choice between Context and an external store depends on how frequently the shared state changes, how many components consume it, and whether selectors, middleware, or devtools are needed. Context is a dependency injection mechanism for shared values, not a complete state management solution. For small to medium applications, Context integrates natively and requires less code; for larger applications with complex state, external stores are preferable.

**Beginner-Friendly Explanation:** Imagine you and your roommates share a whiteboard. When one person writes on it, everyone can see the update. That is shared state. React's Context API is like a whiteboard mounted in the hallway—anyone in the apartment can read it. An external store like Zustand is like a shared digital document—you can subscribe to only the parts you care about, so you do not get notified when someone edits a section you are not looking at.

### Purposes

- To share client-side state (session, theme, feature flags, user preferences, active workspace) across unrelated components.
- To avoid prop drilling through intermediate components that do not use the data.
- To provide a single source of truth for cross-component data.
- To enable selector-based subscriptions so components only re-render when their specific slice of state changes.
- To support middleware, devtools, persistence, and time-travel debugging (Redux).

### Syntax Rules and Structure

**Pattern 1: Context API (Built-in)**

```jsx
import { createContext, useContext, useState, useMemo } from 'react';

const ThemeContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  // Memoise the value to prevent unnecessary consumer re-renders
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

**Component Breakdown:**
- `createContext(null)`: Creates the context object.
- `useMemo(() => ({ theme, setTheme }), [theme])`: Memoises the value object so it only changes when `theme` changes.
- `useTheme()`: A custom hook that reads the context and throws if used outside the provider.

**Pattern 2: Zustand (Minimal External Store)**

```javascript
import { create } from 'zustand';

const useCartStore = create((set) => ({
  items: [],
  addItem: (product) => set((state) => ({
    items: [...state.items, product],
  })),
  removeItem: (id) => set((state) => ({
    items: state.items.filter(i => i.id !== id),
  })),
}));

function CartSummary() {
  const items = useCartStore((state) => state.items);
  const total = items.reduce((sum, item) => sum + item.price, 0);

  return <p>Total: ${total.toFixed(2)}</p>;
}
```

**Component Breakdown:**
- `create((set) => ({ ... }))`: Creates the Zustand store.
- `useCartStore((state) => state.items)`: Selects only the `items` slice; the component re-renders only when `items` changes.
- No provider is needed; the store is a hook usable anywhere.

**Syntax Rules:**
- **Context API:** Use for rarely changing, global-to-a-subtree values (theme, locale, auth). Memoise the value object with `useMemo`. Split state and dispatch into separate contexts for state + reducer patterns.
- **Zustand:** Use for client state shared across features. Select only the slices you need. Use `useShallow` for selecting multiple values.
- **Redux Toolkit:** Use for large applications with complex, predictable state. Create slices with `createSlice`, configure the store with `configureStore`, and wrap the app in `<Provider>`. Use `useSelector` to read and `useDispatch` to write.
- Never copy server data into a client store; server data belongs in TanStack Query.

**Constraints and Limitations:**
- Context causes all consumers to re-render when the provider value changes; split contexts and memoise values to mitigate.
- Context makes component reuse more difficult because consumers are coupled to the context's shape.
- External stores add dependencies; Redux is the heaviest, Zustand is lightweight.
- External stores are overkill for simple sibling communication that can be handled with lifted state.

### Annotated Code Example: Zustand Store for Cross-Branch Sharing

```jsx
import { create } from 'zustand';

// Create the store — lives outside React
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

// Component in one branch of the tree
function AddTodo() {
  const addTodo = useTodoStore((state) => state.addTodo);
  const [text, setText] = useState('');

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      if (text.trim()) { addTodo(text.trim()); setText(''); }
    }}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

// Component in a completely different branch
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

// Parent: renders both branches — no props passed
export default function TodoApp() {
  return (
    <div>
      <h1>Todo List</h1>
      <Sidebar><AddTodo /></Sidebar>
      <MainContent><TodoList /></MainContent>
    </div>
  );
}
```

**Expected Output:** A form with an input and "Add" button, and an empty todo list. Typing a task and clicking "Add" adds it to the list. Clicking a task toggles its done state (strikethrough).

**Why This Output Occurs:** The Zustand store holds all todo state outside React. `AddTodo` and `TodoList` are in separate branches of the component tree, separated by intermediate components that know nothing about the store. Both subscribe to the store directly—`AddTodo` selects `addTodo`, and `TodoList` selects `todos` and `toggleTodo`. When `addTodo` updates the store, `TodoList` re-renders with the new todo. The components communicate through the store without parent mediation or prop drilling.

### Real-World Cases

- **Shopping cart:** Cart items stored in a Zustand store; product pages add items and the cart sidebar reads them.
- **User authentication:** Auth state (user, token, login status) stored globally; navbar and protected routes read it.
- **Theme switching:** Theme context provides the current theme and a toggle function to all components.
- **Dashboard filters:** Filter state stored in a shared store; multiple chart widgets read the same filters and update together.
- **Multi-step forms:** Form data stored in a global store; each step reads and writes its portion.

### References

- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – Scaling Up with Reducer and Context: https://react.dev/learn/scaling-up-with-reducer-and-context
- Zustand Documentation – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- Redux Toolkit – Quick Start: https://redux.js.org/tutorials/quick-start
- Feature-Sliced Design – React's Context API: Friend or Architectural Foe?: https://feature-sliced.design/blog/context-api-performance
- Steve Kinney – Separating Actions from State with Two Contexts: https://stevekinney.com/courses/react-performance/separating-actions-from-state-with-two-contexts

---

## Core Concept 3: Server State (TanStack Query)

### Definitions

**Core Definition:** Server state is asynchronous data fetched from a remote server, characterised by its separate cache lifecycle, loading states, error states, and remote ownership—managed by libraries like TanStack Query (React Query) or SWR.

**Technical Definition:** Server state refers to data that is owned by a remote server and fetched into the client for display. It differs fundamentally from client state in several ways: it is asynchronous (requires loading and error states), it can become stale (the server's copy may change independently), it has a cache lifecycle (staleness, garbage collection, refetching), and it is not owned by any single component. TanStack Query v5 is the dominant library for server-state management, providing a stale-while-revalidate cache keyed by deterministic arrays (query keys). It ships with production-ready defaults: `staleTime: 0` (data is instantly stale), `gcTime: 5 * 60_000` (unused cache kept for 5 minutes), `retry: 3` (3 retries with exponential backoff), `refetchOnWindowFocus: true`, and `refetchOnReconnect: true`. The philosophy is that React Query is an **async state manager, not a data fetcher**: you provide the Promise, and Query manages caching, background updates, and synchronization. A critical rule is to never copy server data into a client state store; TanStack Query is the single source of truth for API data.

**Beginner-Friendly Explanation:** Imagine you order a book from an online store. The store (server) owns the book. You (the client) get a copy delivered to your house (the cache). The copy might be outdated if the store updates the book's contents. You do not "own" the book—you have a cached copy. TanStack Query is like a librarian who tracks which copies are fresh, automatically re-orders updated versions, and tells you when a copy is being fetched. You do not need to manually manage when to re-order or which copy is stale—the librarian handles it all.

### Purposes

- To fetch remote data and display it with automatic caching, background refetching, and stale-while-revalidate semantics.
- To handle loading, error, and success states declaratively without manual `useState` + `useEffect` boilerplate.
- To manage cache invalidation and optimistic updates after mutations.
- To deduplicate identical requests across components.
- To provide pagination, infinite queries, and dependent queries with built-in support.
- To synchronise server data across routes and components with a shared cache.

### Syntax Rules and Structure

**General Syntax (`useQuery`):**
```jsx
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60_000, // Data considered fresh for 5 minutes
  });

  if (isPending) return <p>Loading...</p>;
  if (isError) return <p>Error: {error.message}</p>;

  return <h1>{data.name}</h1>;
}
```

**Component Breakdown:**
- `queryKey: ['user', userId]`: A deterministic array that uniquely identifies the query. It is used for caching and invalidation.
- `queryFn: () => fetchUser(userId)`: The function that returns a Promise resolving to the data.
- `staleTime`: How long the data is considered fresh before it is eligible for refetching.
- `isPending`: `true` when there is no cached data and the query is loading.
- `isError`: `true` when the query has failed.

**General Syntax (`useMutation`):**
```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodo() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newTodo) => fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify(newTodo),
    }),
    onSuccess: () => {
      // Invalidate and refetch the todos query
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button onClick={() => mutation.mutate({ text: 'New Todo' })}>
      Add Todo
    </button>
  );
}
```

**Component Breakdown:**
- `mutationFn`: The function that performs the mutation.
- `onSuccess`: A callback that runs after a successful mutation; used to invalidate related queries.
- `queryClient.invalidateQueries`: Marks cached queries as stale and triggers refetching.

**Syntax Rules:**
- Use a query-key factory for complex applications: `const userKeys = { all: ['users'], detail: (id) => ['users', id] }`.
- Set `staleTime` deliberately (not the default `0`) based on how frequently the data changes.
- On mutations, always cancel-then-snapshot-then-write before optimistic updates.
- Invalidate queries by prefix (`queryClient.invalidateQueries({ queryKey: ['todos'] })`) to refetch all related queries.
- Never copy server data into a client store; TanStack Query is the source of truth for API data.
- Use `useSuspenseQuery` for first-class Suspense support (React 18+).

**Constraints and Limitations:**
- TanStack Query adds a dependency and a learning curve.
- The default `staleTime: 0` means data is instantly stale and will refetch on mount/focus unless raised.
- For frequently changing data (e.g., real-time feeds), consider WebSockets or subscriptions instead.
- Server-state libraries are not designed for client state (UI flags, form inputs); use them only for remote data.

### Annotated Code Example: Dependent Queries with Gating

```jsx
import { useQuery } from '@tanstack/react-query';

function UserProjects({ email }) {
  // Step 1: Fetch the user by email
  const { data: user, isLoading: userLoading } = useQuery({
    queryKey: ['user', email],
    queryFn: () => fetchUserByEmail(email),
  });

  const userId = user?.id;

  // Step 2: Fetch projects, but only when userId is available
  const { data: projects, isLoading: projectsLoading } = useQuery({
    queryKey: ['projects', userId],
    queryFn: () => fetchProjectsByUser(userId),
    enabled: !!userId, // ✅ Gated: runs only when userId is defined
  });

  if (userLoading) return <p>Loading user...</p>;
  if (projectsLoading) return <p>Loading projects...</p>;

  return (
    <ul>
      {projects?.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

**Expected Output:** The component first displays "Loading user..." while the user is fetched. Once the user is available and `userId` is set, the projects query becomes enabled, and "Loading projects..." is displayed. When projects resolve, the list is rendered.

**Why This Output Occurs:** The `enabled: !!userId` option tells TanStack Query not to execute the projects query until `userId` is truthy. TanStack Query handles the sequencing declaratively: the first query runs, and when its data is available, the second query is automatically enabled and initiated.

### Real-World Cases

- **Product listings:** Fetching products with pagination, caching, and background refetching.
- **User dashboards:** Loading user statistics, activity feeds, and notifications with independent cache lifecycles.
- **Search autocomplete:** Fetching search suggestions with debouncing and request cancellation.
- **Dashboard analytics:** Fetching chart data with `staleTime` configured per chart based on data volatility.
- **Dependent data:** Fetching a user first, then fetching their orders, with the second query gated on the first.

### References

- TanStack Query v5 – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query v5 – Query Keys: https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query v5 – Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- TanStack Query v5 – Dependent Queries: https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- TanStack Query v5 – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- Vercel – How to use TanStack Query for server state: https://vercel.com/docs/guides/tanstack-query

---

## Core Concept 4: URL State (`useSearchParams`)

### Definitions

**Core Definition:** URL state refers to application parameters (such as pagination indices, search filters, sort order, and active tabs) that are synchronised with the browser's address bar, managed via React Router's `useSearchParams` Hook or similar navigation APIs.

**Technical Definition:** URL state is state that is stored in the browser's URL query string (`?key=value`), making it shareable, bookmarkable, and navigable via the browser's back/forward buttons. React Router v6 provides the `useSearchParams` Hook, which returns a tuple of the current `URLSearchParams` object and a setter function. Setting the search params causes a navigation (a new history entry). The setter accepts a query string (`"?tab=1"`), a shorthand object (`{ tab: "1" }`), an array of tuples (`[["tab", "1"]]`), or a `URLSearchParams` object. The `searchParams` object is a stable reference, making it safe to use as a dependency in `useEffect`. However, it is mutable: changing the object without calling `setSearchParams` will not update the URL. URL state is ideal for filters, search queries, pagination, sort order, and any state that should be shareable via link.

**Beginner-Friendly Explanation:** Have you ever shared a link to a filtered product listing and noticed that the filters are preserved when the recipient opens it? That is URL state. Instead of storing the filter in a hidden variable (which disappears when you close the tab), the filter is written into the URL itself. This makes the state shareable, bookmarkable, and compatible with the browser's back button. React Router's `useSearchParams` Hook is the bridge between your React components and the URL.

### Purposes

- To make application state shareable via link (e.g., sending a colleague a link to a filtered view).
- To enable bookmarking of specific application states.
- To synchronise UI state with browser navigation (back/forward buttons).
- To persist filter, sort, and pagination state across page reloads without using storage APIs.
- To decouple UI state from component state, making it accessible to any component via the URL.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { useSearchParams } from 'react-router';

function ProductFilters() {
  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get('category') ?? 'all';
  const sort = searchParams.get('sort') ?? 'name';

  function handleCategoryChange(newCategory) {
    setSearchParams({ category: newCategory, sort });
  }

  return (
    <div>
      <select value={category} onChange={(e) => handleCategoryChange(e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
      </select>
      <p>Current sort: {sort}</p>
    </div>
  );
}
```

**Component Breakdown:**
- `useSearchParams()`: Returns `[searchParams, setSearchParams]`.
- `searchParams.get('category')`: Reads a single query parameter (returns `null` if absent).
- `setSearchParams({ category: newCategory, sort })`: Updates the query string and causes a navigation.

**Syntax Rules:**
- Use `searchParams.get(key)` to read a single parameter; use `searchParams.getAll(key)` for multi-value parameters.
- Use the functional callback form for updates that depend on the previous value: `setSearchParams(prev => { prev.set('tab', '2'); return prev; })`. Note that this does not support React's `setState` queueing logic.
- Use `searchParams` as a `useEffect` dependency; it is a stable reference.
- Provide default values when reading parameters to avoid `null` checks.
- For TypeScript projects, consider using `react-zod-url-state` to validate and synchronise URL state with schemas.

**Constraints and Limitations:**
- URL state is inherently public—do not store sensitive data (tokens, personal information) in the URL.
- URL length is limited (browsers typically support 2,000–8,000 characters); do not store large objects.
- Every `setSearchParams` call triggers a navigation, which may cause a full route re-render.
- The `searchParams` object is mutable; changing it without calling `setSearchParams` does not update the URL.
- Complex nested objects cannot be represented in the URL without serialisation.

### Annotated Code Example: Pagination with URL State

```jsx
import { useSearchParams } from 'react-router';

function PaginatedList({ items }) {
  const [searchParams, setSearchParams] = useSearchParams();

  const page = Number(searchParams.get('page') ?? '1');
  const pageSize = 10;
  const totalPages = Math.ceil(items.length / pageSize);
  const start = (page - 1) * pageSize;
  const currentItems = items.slice(start, start + pageSize);

  function goToPage(newPage) {
    setSearchParams({ page: String(newPage) });
  }

  return (
    <div>
      <ul>
        {currentItems.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
      <button disabled={page <= 1} onClick={() => goToPage(page - 1)}>
        Previous
      </button>
      <span>Page {page} of {totalPages}</span>
      <button disabled={page >= totalPages} onClick={() => goToPage(page + 1)}>
        Next
      </button>
    </div>
  );
}
```

**Expected Output:** A list of 10 items with "Previous" and "Next" buttons. Clicking "Next" advances to the next page, updating both the displayed items and the URL (`?page=2`). The browser's back button returns to the previous page.

**Why This Output Occurs:** The `page` parameter is read from the URL via `searchParams.get('page')`. Clicking "Next" calls `setSearchParams({ page: '2' })`, which causes a navigation and re-renders the component with the new page number. The items are sliced based on the page number, and the URL reflects the current page. Because the page is in the URL, the state is shareable and bookmarked.

### Real-World Cases

- **E-commerce filters:** Category, price range, and sort order stored in the URL for shareable filtered views.
- **Search results:** The search query and page number stored in the URL for bookmarkable searches.
- **Dashboard tabs:** The active tab stored in the URL so refreshing the page preserves the active tab.
- **Data tables:** Sort column and sort direction stored in the URL for shareable table configurations.
- **Multi-step forms:** The current step stored in the URL so users can navigate back and forward between steps.

### References

- React Router – useSearchParams: https://api.reactrouter.com/v7/functions/react-router.useSearchParams.html
- React Router – useSearchParams (main branch): https://reactrouter.com/api/hooks/useSearchParams
- LogRocket – Why URL state matters: A guide to useSearchParams in React: https://blog.logrocket.com/why-url-state-matters-guide-usesearchparams-react/
- Socket.dev – react-zod-url-state: https://socket.dev/npm/package/react-zod-url-state

---

## Core Concept 5: Form State (React Hook Form)

### Definitions

**Core Definition:** Form state refers to the transient, high-frequency state that captures user input values, validation errors, dirty/touched flags, and submission lifecycle status before the data is committed to the server or a parent component.

**Technical Definition:** Form state is a specialised category of client state characterised by its transient nature (it exists only while the user is filling out the form), high update frequency (every keystroke can trigger an update), and complex lifecycle (pristine → dirty → validating → submitting → submitted). Managing form state manually with `useState` and `onChange` handlers leads to excessive re-renders and boilerplate as the form grows. React Hook Form (RHF) is the dominant library for form-state management. It uses uncontrolled inputs with refs, only re-rendering on validation errors or submission, which dramatically reduces re-renders compared to controlled inputs. The `useForm()` Hook returns a `register` function for connecting inputs, `handleSubmit` for submission, and a `formState` object containing `errors`, `isDirty`, `isSubmitting`, `isValid`, and other lifecycle flags. For complex validation, RHF integrates with schema validation libraries (Zod, Yup) via resolvers.

**Beginner-Friendly Explanation:** Filling out a form is like writing on a whiteboard that gets erased after you submit. Every keystroke is a tiny change, and you need to track what you have typed, whether it is valid, and whether you have submitted yet. Managing all of this by hand is like trying to write down every keystroke on a separate sticky note—it gets messy fast. React Hook Form is like a smart whiteboard that tracks everything for you and only bothers you when something is wrong.

### Purposes

- To capture and manage user input values without excessive re-renders.
- To validate form fields (on change, on blur, or on submit) and display error messages.
- To track the form's lifecycle state (pristine, dirty, touched, submitting, submitted).
- To handle form submission and integrate with server mutations.
- To support complex form structures (nested fields, field arrays, multi-step wizards).
- To integrate with schema validation libraries (Zod, Yup) for type-safe validation.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { useForm } from 'react-hook-form';

function MyForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    defaultValues: { email: '', password: '' },
  });

  async function onSubmit(data) {
    await fetch('/api/register', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: 'Email is required' })} />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        type="password"
        {...register('password', { minLength: { value: 8, message: 'Too short' } })}
      />
      {errors.password && <span>{errors.password.message}</span>}

      <button disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

**Component Breakdown:**
- `useForm({ defaultValues })`: Initialises the form with default values.
- `register('email', { required: '...' })`: Connects an input to the form and applies validation rules.
- `handleSubmit(onSubmit)`: Wraps the submission handler, validating the form before calling `onSubmit`.
- `formState.errors`: Contains validation errors keyed by field name.
- `formState.isSubmitting`: `true` while the submission handler is running.

**Syntax Rules:**
- Always provide `defaultValues` in `useForm` so RHF has a single source of truth for determining dirty state.
- Use `register` for native HTML inputs; use `Controller` for third-party or custom input components.
- Prefer schema validation (Zod) with `zodResolver` for complex forms; keep schemas in a centralised location (e.g., `lib/schemas`).
- Read `formState` properties during render (not in effects) to enable subscription.
- Avoid `isValid` with `onSubmit` mode for button state; use `isSubmitted` for showing validation state.
- Wrap async submit handlers in `try/catch` to handle server errors.

**Constraints and Limitations:**
- React Hook Form uses uncontrolled inputs by default; controlled components (e.g., for custom UI libraries) require the `Controller` wrapper.
- Large forms (300+ fields) with a resolver and `formState` reads can freeze during registration; use `useFormState` for isolated subscriptions.
- Schema validation adds a dependency (Zod/Yup) but provides type safety and reusable schemas.
- RHF does not manage server state; use TanStack Query for mutations and cache invalidation.

### Annotated Code Example: Multi-Step Form with Validation

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Define the validation schema
const checkoutSchema = z.object({
  email: z.string().email('Invalid email'),
  address: z.string().min(5, 'Address too short'),
  cardNumber: z.string().regex(/^\d{16}$/, 'Must be 16 digits'),
});

function CheckoutForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isSubmitted },
  } = useForm({
    resolver: zodResolver(checkoutSchema),
    defaultValues: { email: '', address: '', cardNumber: '' },
  });

  async function onSubmit(data) {
    try {
      await fetch('/api/checkout', {
        method: 'POST',
        body: JSON.stringify(data),
      });
    } catch (err) {
      alert('Checkout failed: ' + err.message);
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Email</label>
        <input {...register('email')} />
        {errors.email && <span style={{ color: 'red' }}>{errors.email.message}</span>}
      </div>

      <div>
        <label>Address</label>
        <input {...register('address')} />
        {errors.address && <span style={{ color: 'red' }}>{errors.address.message}</span>}
      </div>

      <div>
        <label>Card Number</label>
        <input {...register('cardNumber')} />
        {errors.cardNumber && <span style={{ color: 'red' }}>{errors.cardNumber.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Processing...' : 'Checkout'}
      </button>

      {isSubmitted && Object.keys(errors).length === 0 && (
        <p>Order placed successfully!</p>
      )}
    </form>
  );
}
```

**Expected Output:** A checkout form with email, address, and card number fields. Submitting the form with invalid data displays red error messages below each field. Submitting with valid data shows a "Processing..." button state and then "Order placed successfully!".

**Why This Output Occurs:** The `zodResolver` validates the form data against the `checkoutSchema` on submission. If validation fails, `errors` contains field-specific messages that are displayed conditionally. If validation passes, `onSubmit` is called with the valid data. The `isSubmitting` flag disables the button during the async request, and `isSubmitted` combined with no errors shows the success message.

### Real-World Cases

- **Registration forms:** Email, password, and confirmation fields with validation and error display.
- **Checkout forms:** Shipping address, payment details, and order summary with multi-step navigation.
- **Profile editing:** Name, bio, and avatar upload with dirty state tracking.
- **Survey forms:** Dynamic field arrays and conditional fields with schema validation.
- **Login forms:** Email and password with server error handling and submission state.

### References

- React Hook Form – Documentation: https://react-hook-form.com/
- React Hook Form – formState: https://react-hook-form.com/docs/useform/formstate
- React Hook Form – Register: https://react-hook-form.com/docs/useform/register
- Refine – Essentials of Managing Form State with React Hook Form: https://refine.dev/blog/react-hook-form/
- Oakoss – Agent Skills – Forms: https://github.com/oakoss/agent-skills

---

## Core Concept 6: Derived State (Render-Time Computation)

### Definitions

**Core Definition:** Derived state is computed information calculated strictly on the fly during the render pass using existing state or props, avoiding redundant storage and synchronization bugs.

**Technical Definition:** Derived state is any value that can be computed from existing props or state during rendering. Storing derived values in separate state variables is an anti-pattern: it causes extra re-renders (one for the source state change, another for the derived state update), introduces synchronization bugs if the derived value is not updated consistently, and wastes memory. The golden rule is: **store the minimal amount of state needed, then derive everything else** during render. React's official documentation states: "If you can calculate some information from the component's props or its existing state variables during rendering, you should not put that information into that component's state". For expensive calculations, `useMemo` can memoise the result so it is only recomputed when its dependencies change. Computing derived values directly during render produces the correct result in a single pass, whereas using `useEffect` to synchronise derived state causes a double-render cycle.

**Beginner-Friendly Explanation:** Imagine you have a shopping cart with items and prices. Instead of writing down the total on a separate sticky note and updating it every time an item is added, you just calculate the total when you need it. The total is "derived" from the items. If you wrote it on a sticky note, you might forget to update it when an item is removed—and then your sticky note would be wrong. Deriving the total on the spot guarantees it is always correct.

### Purposes

- To avoid redundant state variables and the synchronization bugs they cause.
- To eliminate the extra re-renders caused by setting state in `useEffect` for derived values.
- To keep the state structure minimal and easy to reason about.
- To compute filtered lists, totals, counts, formatted dates, and boolean conditions on demand.
- To improve performance by memoising expensive calculations with `useMemo`.

### Syntax Rules and Structure

**General Syntax (Simple Derivation):**
```jsx
function ShoppingCart({ items }) {
  // ✅ Derived during render — no state, no Effect
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const itemCount = items.length;

  return (
    <div>
      <p>Items: {itemCount}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

**Component Breakdown:**
- `total`: Computed from `items` during render.
- `itemCount`: Computed from `items` during render.
- No `useState` or `useEffect` is used for these values.

**General Syntax (Expensive Derivation with `useMemo`):**
```jsx
import { useMemo } from 'react';

function ProductList({ products, filter }) {
  // ✅ Memoised derivation — only recalculates when products or filter change
  const filteredProducts = useMemo(
    () => products.filter(p =>
      filter === 'all' || p.category === filter
    ),
    [products, filter]
  );

  return (
    <ul>
      {filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

**Component Breakdown:**
- `useMemo(() => ..., [products, filter])`: Memoises the filtered list, recalculating only when `products` or `filter` changes.
- The filter logic runs during render, not in an Effect.

**Syntax Rules:**
- If a value can be computed from current props or state, derive it during render instead of storing it in state.
- Use `useMemo` only for genuinely expensive calculations; measure with `console.time` before memoising.
- Do not set state in `useEffect` solely in response to prop changes; derive the value instead.
- When resetting all state on a prop change, use the `key` prop instead of an Effect.
- When adjusting *some* state on a prop change, use conditional `setState` during render (guarded by a comparison with the previous prop).

**Constraints and Limitations:**
- `useMemo` is a performance optimisation, not a semantic guarantee; React may discard memoised values.
- Conditional `setState` during render must be guarded by a comparison with a stored previous prop, or it will cause an infinite render loop.
- The `key` approach resets *all* state in the component and its children, which may not be desired if only some state needs resetting.
- Derived state cannot be used for values that need to be mutated independently of their source.

### Annotated Code Example: Derived State vs. Synced State

```jsx
import { useState } from 'react';

// ❌ Anti-pattern: storing derived values in state
function ShoppingCartBad({ items }) {
  const [total, setTotal] = useState(0);
  const [itemCount, setItemCount] = useState(0);

  // Effect to sync derived state — causes extra renders and sync bugs
  useEffect(() => {
    setTotal(items.reduce((sum, i) => sum + i.price, 0));
    setItemCount(items.length);
  }, [items]);

  return (
    <div>
      <p>Items: {itemCount}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}

// ✅ Correct: deriving values during render
function ShoppingCartGood({ items }) {
  const total = items.reduce((sum, i) => sum + i.price, 0);
  const itemCount = items.length;

  return (
    <div>
      <p>Items: {itemCount}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

**Expected Output:** Both components render the same output: the item count and total price. However, the bad version triggers an extra render every time `items` changes (one for the `items` change, one for the `setTotal`/`setItemCount` call). The good version computes the values during render in a single pass.

**Why This Output Occurs:** In the bad version, the `useEffect` runs after render, calls `setTotal` and `setItemCount`, and triggers a second render. This is wasteful and can lead to synchronization bugs if the Effect is not perfectly maintained. In the good version, `total` and `itemCount` are computed directly during render from `items`, which is the single source of truth. No extra render, no sync bugs.

### Real-World Cases

- **Shopping cart totals:** Computing the total price from cart items during render.
- **Filtered lists:** Filtering an array based on a search query during render.
- **Form validation:** Deriving `isValid` from form values during render (`const isValid = email.includes('@') && password.length >= 8`).
- **User display names:** Deriving `displayName` from `firstName` and `lastName` during render.
- **Chart data:** Computing chart data points from raw data during render.
- **Dashboard metrics:** Computing averages, percentages, and counts from raw data during render.

### References

- React Official Documentation – Choosing the State Structure (Avoid Redundant State): https://18.react.dev/learn/choosing-the-state-structure
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- Steve Kinney – Derived vs. Stored State: https://stevekinney.com/courses/react-performance/derived-vs-stored-state
- Epic React – Derive State: https://www.epicreact.dev/
- Vercel – Derive State Instead of Syncing: https://github.com/vercel-labs/agent-skills

---

## Core Concept 7: Persistent State (localStorage, IndexedDB, Zustand Persist)

### Definitions

**Core Definition:** Persistent state is long-lived data that is written to browser storage APIs (such as `localStorage`, `sessionStorage`, or `IndexedDB`) so that it survives page reloads, tab closures, and browser restarts.

**Technical Definition:** Persistent state bridges the gap between ephemeral client state and durable storage. The browser provides three primary storage mechanisms: `localStorage` (synchronous, string-only, ~5–10 MB limit), `sessionStorage` (synchronous, string-only, cleared when the tab closes), and `IndexedDB` (asynchronous, structured data, large storage limits). `localStorage` is the simplest but has hard limits that many applications outgrow quickly; it is the "default reach" but is unsuitable for large data volumes. `IndexedDB` is the browser's real database: it handles structured data, has far higher size limits, and its async API keeps large reads and writes off the main thread. In React, persistence is typically implemented via the Zustand `persist` middleware, which wraps store state in a storage adapter (`localStorage`, `IndexedDB`, etc.) and automatically hydrates state on initialization. Each persisted store defines a `migrate` function to handle older persisted shapes on load.

**Beginner-Friendly Explanation:** Imagine you are writing a document on a computer without saving it. When you close the program, the document is gone. Persistent state is like saving the document to a file—when you reopen the program, the document is still there. `localStorage` is like a small notebook you can write short notes in. `IndexedDB` is like a filing cabinet that can hold large folders of documents. Zustand's `persist` middleware is like an automatic save feature that writes your state to the notebook or filing cabinet without you having to remember to do it.

### Purposes

- To preserve user preferences (theme, language, sidebar state) across browser sessions.
- To cache large datasets (documents, images, offline data) for offline access or performance.
- To persist form drafts so users do not lose their work if they close the tab.
- To store authentication tokens and session data (with security considerations).
- To synchronise state across browser tabs (via `localStorage` events or Zustand persist).
- To provide a "remember me" experience for returning users.

### Syntax Rules and Structure

**Pattern 1: Zustand Persist Middleware (Recommended)**

```javascript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useThemeStore = create(
  persist(
    (set) => ({
      theme: 'light',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'theme-storage', // unique key in storage
      storage: createJSONStorage(() => localStorage),
    }
  )
);
```

**Component Breakdown:**
- `persist((set) => ({ ... }), { ... })`: Wraps the store in the persist middleware.
- `name: 'theme-storage'`: The storage key under which the state is saved.
- `storage: createJSONStorage(() => localStorage)`: The storage adapter (localStorage by default).
- State is automatically hydrated on initialization and written on every change.

**Pattern 2: Manual localStorage (Simple Cases)**

```jsx
import { useState, useEffect } from 'react';

function usePersistentState(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

**Component Breakdown:**
- `useState(() => { ... })`: Lazy initialiser reads from `localStorage` on first render.
- `useEffect`: Writes to `localStorage` whenever the value changes.
- `JSON.parse` / `JSON.stringify`: Serialises and deserialises the value.

**Pattern 3: IndexedDB for Large Data (Custom Adapter)**

```javascript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { get, set, del } from 'idb-keyval';

const indexedDBStorage = {
  getItem: (name) => get(name),
  setItem: (name, value) => set(name, value),
  removeItem: (name) => del(name),
};

const useDocumentStore = create(
  persist(
    (set) => ({
      documents: [],
      addDocument: (doc) => set((state) => ({
        documents: [...state.documents, doc],
      })),
    }),
    {
      name: 'document-store',
      storage: createJSONStorage(() => indexedDBStorage),
    }
  )
);
```

**Component Breakdown:**
- `indexedDBStorage`: A custom storage adapter wrapping `idb-keyval` for IndexedDB.
- `createJSONStorage(() => indexedDBStorage)`: Tells Zustand persist to use IndexedDB.
- The store hydrates asynchronously (IndexedDB is async), so components may need a hydration flag.

**Syntax Rules:**
- Use `localStorage` for small, simple values (theme, preferences, tokens).
- Use `IndexedDB` for large, structured data (documents, images, offline datasets).
- Use Zustand's `persist` middleware for declarative persistence; it supports both sync and async storages.
- Define a `migrate` function for each persisted store to handle schema changes across versions.
- Never store sensitive data (passwords, personal information) in `localStorage`; it is accessible to any script on the page.
- For SSR, guard storage access with `typeof window !== 'undefined'`.

**Constraints and Limitations:**
- `localStorage` is synchronous and blocks the main thread; it has a ~5–10 MB limit.
- `localStorage` only stores strings; complex objects must be serialised.
- `IndexedDB` has an awkward native API; use a wrapper library (`idb-keyval`, `localforage`) for ergonomics.
- IndexedDB is asynchronous, so hydration may not be complete on the first render; use a hydration flag to avoid rendering stale data.
- Persisted state is per-browser-profile; clearing site data or switching browsers loses the state.
- Migrations are the store's responsibility; changing the persisted shape without a migration breaks existing saves.

### Annotated Code Example: Persisting Theme with Zustand

```jsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

// Create a persisted theme store
const useThemeStore = create(
  persist(
    (set) => ({
      theme: 'light',
      toggleTheme: () => set((state) => ({
        theme: state.theme === 'light' ? 'dark' : 'light',
      })),
    }),
    {
      name: 'app-theme',
      storage: createJSONStorage(() => localStorage),
    }
  )
);

function ThemeToggle() {
  const theme = useThemeStore((state) => state.theme);
  const toggleTheme = useThemeStore((state) => state.toggleTheme);

  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}

function ThemedPanel() {
  const theme = useThemeStore((state) => state.theme);

  return (
    <div style={{
      background: theme === 'dark' ? '#333' : '#fff',
      color: theme === 'dark' ? '#fff' : '#333',
      padding: '20px',
    }}>
      Current theme: {theme}
    </div>
  );
}
```

**Expected Output:** A toggle button and a themed panel. Clicking the button switches the theme between light and dark, updating both components. Reloading the page preserves the selected theme because it is persisted in `localStorage`.

**Why This Output Occurs:** The `persist` middleware automatically writes the store's state to `localStorage` under the key `'app-theme'` on every change, and hydrates the state from `localStorage` on initialization. Both `ThemeToggle` and `ThemedPanel` subscribe to the store and re-render when the theme changes. After a reload, the store hydrates with the persisted theme, so the correct theme is displayed immediately.

### Real-World Cases

- **Theme preferences:** Persisting the user's light/dark mode choice across sessions.
- **Shopping cart:** Persisting cart contents so the user does not lose items on reload.
- **Form drafts:** Auto-saving form input to `localStorage` so users can resume after an accidental navigation.
- **Authentication tokens:** Storing JWT tokens for session persistence (with security caveats).
- **Offline documents:** Caching large documents in IndexedDB for offline access.
- **User preferences:** Persisting sidebar collapsed state, table column widths, and dashboard layout.

### References

- Zustand – Persisting store data: https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- Zustand – persist Middleware: https://zustand.docs.pmnd.rs/middlewares/persist
- MDN Web Docs – localStorage: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- MDN Web Docs – IndexedDB API: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
- Narraitor – ADR-004: IndexedDB for client-side persistence: https://github.com/jerseycheese/Narraitor/blob/develop/public_docs/architecture/ADR-004-indexeddb-persistence.md
- idb-keyval – Documentation: https://github.com/jakearchibald/idb-keyval

---

## State Architecture Decision Guidance

| State Type | Primary Tool | Scope | Lifecycle | Persistence | Re-render Granularity |
|---|---|---|---|---|---|
| **Local State** | `useState` / `useReducer` | Single component | Component lifetime | None (ephemeral) | Component-only |
| **Shared State** | Context / Zustand / Redux | Multiple components | App or feature lifetime | Optional (Zustand persist) | Selector-based (Zustand/Redux) or all consumers (Context) |
| **Server State** | TanStack Query | Multiple components | Cache lifecycle (stale/gc) | Cache in memory | Query-key based |
| **URL State** | `useSearchParams` | Multiple components / route | Navigation lifetime | URL (shareable) | Route re-render |
| **Form State** | React Hook Form | Form component tree | Form session | Optional (draft persistence) | Field-level (uncontrolled) |
| **Derived State** | Render-time computation / `useMemo` | Any | Render pass | None | None (computed during render) |
| **Persistent State** | Zustand persist / localStorage / IndexedDB | App-wide | Across sessions | Storage API | Store subscription |

**Decision Tree:**

1. **Can the value be computed from existing props/state?** → Derived state (compute during render).
2. **Is the value from the server (API data)?** → Server state (TanStack Query).
3. **Should the value be shareable via link or survive reloads?** → URL state (`useSearchParams`).
4. **Is the value user input being collected in a form?** → Form state (React Hook Form).
5. **Is the value needed by only one component?** → Local state (`useState`/`useReducer`).
6. **Is the value needed across multiple components?** → Shared state (Context / Zustand).
7. **Should the value survive browser restarts?** → Persistent state (Zustand persist).

**The Anti-Goal:** One global store for everything. A global store that holds server data, form state, URL state, and derived values becomes a dumping ground with high coupling, low cohesion, and performance traps. Classify first, then choose the tool.

---

## References

- React Official Documentation – Choosing the State Structure: https://18.react.dev/learn/choosing-the-state-structure
- React Official Documentation – Managing State: https://react.dev/learn/managing-state
- React Official Documentation – useState: https://react.dev/reference/react/useState
- React Official Documentation – useReducer: https://react.dev/reference/react/useReducer
- React Official Documentation – Extracting State Logic into a Reducer: https://react.dev/learn/extracting-state-logic-into-a-reducer
- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – Scaling Up with Reducer and Context: https://react.dev/learn/scaling-up-with-reducer-and-context
- TanStack Query v5 – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query v5 – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query v5 – Dependent Queries: https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- React Router – useSearchParams: https://api.reactrouter.com/v7/functions/react-router.useSearchParams.html
- React Hook Form – Documentation: https://react-hook-form.com/
- React Hook Form – formState: https://react-hook-form.com/docs/useform/formstate
- Zustand – Persisting store data: https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- Zustand – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- Redux Toolkit – Quick Start: https://redux.js.org/tutorials/quick-start
- Feature-Sliced Design – React State Management: https://feature-sliced.design/blog/scalable-react-state-patterns
- Feature-Sliced Design – React's Context API: Friend or Architectural Foe?: https://feature-sliced.design/blog/context-api-performance
- Steve Kinney – Derived vs. Stored State: https://stevekinney.com/courses/react-performance/derived-vs-stored-state
- Steve Kinney – Separating Actions from State with Two Contexts: https://stevekinney.com/courses/react-performance/separating-actions-from-state-with-two-contexts
- LogRocket – Why URL state matters: A guide to useSearchParams in React: https://blog.logrocket.com/why-url-state-matters-guide-usesearchparams-react/
- Refine – Essentials of Managing Form State with React Hook Form: https://refine.dev/blog/react-hook-form/
- MDN Web Docs – localStorage: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- MDN Web Docs – IndexedDB API: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
- Narraitor – ADR-004: IndexedDB for client-side persistence: https://github.com/jerseycheese/Narraitor/blob/develop/public_docs/architecture/ADR-004-indexeddb-persistence.md
- RTcamp – Choosing the Right React State Management Strategy: https://rtcamp.com/tutorials/react-state-management/
- Vercel – How to use TanStack Query for server state: https://vercel.com/docs/guides/tanstack-query