# State Architecture & Synchronization Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** State Architecture & Synchronization Patterns in React is the discipline of deciding where state lives, how it flows between components, and how it stays consistent across local scopes, server caches, global stores, tabs, and asynchronous workflows.

**Technical Definition:** State architecture in React is a multi-dimensional problem. State must be classified by **ownership** (local UI state, server state, URL state, form state), by **scope** (component, subtree, application-global, cross-tab), and by **temporal characteristics** (synchronous ephemeral state, asynchronous cached state, persistent state). Synchronization patterns determine how state updates propagate: React Context for subtree-scoped state, data-fetching libraries for server-state caching and revalidation, global stores for shared client state, state machines for explicit transition logic, and cross-tab APIs (BroadcastChannel, `storage` events) for browser-wide consistency. The central architectural principle is **separation of concerns by state category**: server state belongs in TanStack Query or SWR, never duplicated in a global store; local state stays local as long as possible; global state is a last resort, not a default.  The companion principle is **derivation over synchronization**: values that can be computed from existing state during render should never be stored in separate state and kept in sync with `useEffect`. 

**Beginner-Friendly Explanation:** Your React app has many kinds of state: data from the server (user profiles, product lists), UI state (which tab is open, whether a modal is visible), form state (what the user typed), and URL state (the current page, filters). Each kind has a best home. Server data belongs in a caching library. UI state belongs in the component that needs it. Global state is only for things genuinely shared across the whole app. The big mistake is putting everything in one global store and syncing it manually—that leads to bugs, performance problems, and code that's hard to reason about. This cheat sheet teaches you where each kind of state belongs and how to keep it synchronized without fighting React.

### Key Characteristics

- **State Classification First:** Before choosing a tool, classify the state: Is it local or global? Server-owned or client-owned? Synchronous or asynchronous? Persistent or ephemeral? The classification determines the architecture.
- **Server-State is Not Application State:** Data fetched from an API is a cache of remote state, not application state. It belongs in TanStack Query or SWR, which handle caching, deduplication, background refetching, and optimistic updates out of the box. 
- **Context Splitting for Performance:** A single large context provider causes every consumer to re-render when any part of the context changes. Splitting contexts by domain—and separating state from actions—ensures that components only re-render when the data they actually use changes. 
- **Derivation Over Synchronization:** Derived values should be computed during render (with `useMemo` for expensive computations), never stored in `useState` and synced via `useEffect`. This eliminates double-render cycles, stale frames, and synchronization bugs. 
- **Explicit Transitions for Complex Flows:** For multi-step asynchronous workflows with branching logic and impossible-state prevention, state machines (XState) provide explicit, testable, visualizable transition logic. 
- **Cross-Tab Consistency:** Applications with multiple open tabs must synchronize auth state, theme, and other shared state using BroadcastChannel, `storage` events, or library-provided synchronization mechanisms. 

### Prerequisites

- Solid understanding of React Hooks (`useState`, `useEffect`, `useMemo`, `useCallback`, `useContext`, `useReducer`).
- Familiarity with React Context API and its performance characteristics.
- Basic understanding of asynchronous data fetching and Promises.
- Awareness of the distinction between client state and server state.
- Experience with at least one global state library (Zustand, Redux, Jotai) or willingness to learn.

### Related Programming Areas

- **Data Fetching:** TanStack Query, SWR, RTK Query, and Server Components.
- **State Management Libraries:** Zustand, Redux Toolkit, Jotai, Valtio, MobX.
- **State Machines:** XState and finite state machine modelling.
- **Browser APIs:** BroadcastChannel, `storage` events, `useSyncExternalStore`.
- **Performance Optimisation:** Re-render prevention, selector patterns, context splitting.
- **URL State Management:** Router search parameters, nuqs, and deep-linkable state.

### Core Concepts / Features

1. Local-First & Context-Based State
2. Server-State Separation
3. Global Stores & Event-Driven State
4. State Machines & Finite State Patterns
5. Derived State Patterns

---

## Core Concept 1: Local-First & Context-Based State

### Definitions

**Core Definition:** Local-first state architecture scopes state to the closest necessary parent component, using React Context only when state must be shared across a subtree, and splitting contexts to prevent performance regressions.

**Technical Definition:** Local-first state architecture begins with `useState` (or `useReducer`) inside the component that owns the state. State is lifted only when a sibling or descendant needs it. When lifting beyond one level becomes impractical, React Context provides a transport mechanism for sharing state across a subtree. However, Context has a critical performance characteristic: when the context value changes, **every consumer re-renders**, regardless of whether the changed portion of the value is relevant to that consumer. Two complementary techniques mitigate this: (1) **splitting contexts by domain** (e.g., separate `ThemeContext`, `UserContext`, `NotificationsContext`) so that a change in one domain does not re-render consumers of another;  and (2) **separating state from actions** (e.g., `TodoStateContext` and `TodoActionsContext`) so that components that only dispatch actions do not re-render when state changes, and vice versa. 

**Beginner-Friendly Explanation:** Start with state in the component that needs it. If another component needs it too, lift it up to the nearest common parent. If you have to pass it through many layers, use React Context instead of prop drilling. But be careful: when Context changes, everything that uses it re-renders. So split your contexts—one for theme, one for user, one for notifications—and put your state in one context and your update functions in another. This way, changing the theme doesn't re-render the notification list.

### Purposes

- To keep state as local as possible, reducing coupling and unintended re-renders.
- To use Context only when state genuinely needs to be shared across a subtree.
- To prevent performance regressions by splitting contexts by domain and by separating state from actions.
- To provide a clear decision framework: local state → lifted state → Context → global store.
- To stabilise context values with `useMemo` and `useCallback` to prevent unnecessary re-renders.
- To create strict contexts with custom hooks that throw when the provider is missing.

### Syntax Rules and Structure

**Decision Framework: Where Should State Live?**

```
Component-local state        → useState / useReducer
Sibling-shared state         → Lift to nearest common parent
Subtree-shared state         → React Context (split by domain)
App-global client state      → Zustand / Redux / Jotai
Server state                 → TanStack Query / SWR
URL state                    → Router search params / nuqs
```

**General Syntax for Local State (Start Here):**

```jsx
function SearchBox() {
  const [query, setQuery] = useState('');

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**Component Breakdown:**
- State starts local. No context, no global store, no prop drilling.
- Only lift or globalise when another component genuinely needs the state.

**General Syntax for Split Contexts (By Domain):**

```jsx
// ❌ DON'T: Single large context
const AppContext = createContext({
  user: null,
  theme: 'light',
  notifications: [],
  settings: {},
});

// ✅ DO: Split by domain
const ThemeContext = createContext('light');
const UserContext = createContext(null);
const NotificationsContext = createContext([]);
const SettingsContext = createContext({});

function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);
  const [notifications, setNotifications] = useState([]);

  return (
    <ThemeContext.Provider value={theme}>
      <UserContext.Provider value={user}>
        <NotificationsContext.Provider value={notifications}>
          <Header />        {/* Only re-renders when theme changes */}
          <Notifications />  {/* Only re-renders when notifications change */}
          <Settings />       {/* Only re-renders when settings change */}
        </NotificationsContext.Provider>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}
```

**Component Breakdown:**
- Each domain has its own context.
- A change in `theme` re-renders only `Header`, not `Notifications` or `Settings`.
- This is the single most impactful optimisation for context-heavy applications. 

**General Syntax for Separating State from Actions:**

```jsx
// Two contexts: state and actions
const TodoStateContext = createContext([]);
const TodoActionsContext = createContext(null);

function TodoProvider({ children }) {
  const [todos, setTodos] = useState([]);

  // Memoise actions so they have stable references
  const actions = useMemo(() => ({
    addTodo: (text) =>
      setTodos((prev) => [...prev, { id: crypto.randomUUID(), text, completed: false }]),
    toggleTodo: (id) =>
      setTodos((prev) =>
        prev.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t))
      ),
    deleteTodo: (id) =>
      setTodos((prev) => prev.filter((t) => t.id !== id)),
  }), []);

  return (
    <TodoStateContext.Provider value={todos}>
      <TodoActionsContext.Provider value={actions}>
        {children}
      </TodoActionsContext.Provider>
    </TodoStateContext.Provider>
  );
}
```

**Component Breakdown:**
- `TodoStateContext` carries the state (`todos`).
- `TodoActionsContext` carries the action functions.
- Components that only need actions (`addTodo`) do not re-render when `todos` changes.
- Components that only read state (`todos`) do not re-render when actions are recreated.
- Actions are memoised with `useMemo` and an empty dependency array because they only use the stable `setTodos` function. 

**Syntax Rules:**
- Start with `useState` in the component that owns the state.
- Lift state only when a sibling or ancestor needs it.
- Use Context only when prop drilling becomes impractical.
- Split contexts by domain (theme, user, notifications) to prevent cross-domain re-renders.
- Separate state and actions into two contexts for maximum performance.
- Memoise context values with `useMemo` and actions with `useCallback`.
- Use a custom hook (`useTheme()`) that throws if the context is missing (strict context).

**Constraints and Limitations:**
- Context is a transport, not a storage—it does not decide how state changes.
- Context causes all consumers to re-render when the context value changes; splitting is the mitigation.
- Over-splitting contexts can lead to a "provider pyramid" that is hard to read and maintain.
- Context is not a replacement for a state management library for complex global state.

### Annotated Code Examples

**Example 1: Split Contexts for Theme, User, and Notifications**

```jsx
import React, { createContext, useContext, useState, useMemo } from 'react';

// Three separate contexts
const ThemeContext = createContext('light');
const UserContext = createContext(null);
const NotificationsContext = createContext([]);

// Custom hooks with strict context checking
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) throw new Error('useUser must be used within UserProvider');
  return context;
}

function useNotifications() {
  const context = useContext(NotificationsContext);
  if (context === undefined) throw new Error('useNotifications must be used within NotificationsProvider');
  return context;
}

function AppProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState({ name: 'Alice' });
  const [notifications, setNotifications] = useState([]);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <NotificationsContext.Provider value={{ notifications, setNotifications }}>
          {children}
        </NotificationsContext.Provider>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

function Header() {
  const { theme } = useTheme();
  return <header className={theme}>Header — only re-renders when theme changes</header>;
}

function NotificationBadge() {
  const { notifications } = useNotifications();
  return <span>{notifications.length} notifications</span>;
}

function App() {
  return (
    <AppProvider>
      <Header />
      <NotificationBadge />
    </AppProvider>
  );
}
```

**Expected Output:** The `Header` component re-renders only when the theme changes. The `NotificationBadge` re-renders only when notifications change. Changing the theme does not re-render the notification badge, and vice versa.

**Why This Output Occurs:** Each context is scoped to a single domain. React's context propagation only notifies consumers of the context whose value changed. Because `Header` consumes `ThemeContext` and `NotificationBadge` consumes `NotificationsContext`, they are independent. This is the core performance optimisation for context-based state. 

**Example 2: Separating State and Actions in a Todo App**

```jsx
import React, { createContext, useContext, useState, useMemo, useCallback } from 'react';

const TodoStateContext = createContext([]);
const TodoActionsContext = createContext(null);

function useTodoState() {
  const context = useContext(TodoStateContext);
  if (context === undefined) throw new Error('useTodoState must be used within TodoProvider');
  return context;
}

function useTodoActions() {
  const context = useContext(TodoActionsContext);
  if (context === undefined) throw new Error('useTodoActions must be used within TodoProvider');
  return context;
}

function TodoProvider({ children }) {
  const [todos, setTodos] = useState([]);

  const actions = useMemo(() => ({
    addTodo: (text) => setTodos((prev) => [...prev, { id: crypto.randomUUID(), text, completed: false }]),
    toggleTodo: (id) => setTodos((prev) => prev.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t))),
    deleteTodo: (id) => setTodos((prev) => prev.filter((t) => t.id !== id)),
  }), []);

  return (
    <TodoStateContext.Provider value={todos}>
      <TodoActionsContext.Provider value={actions}>
        {children}
      </TodoActionsContext.Provider>
    </TodoStateContext.Provider>
  );
}

function TodoList() {
  const todos = useTodoState(); // Only re-renders when todos change
  return <ul>{todos.map((t) => <li key={t.id}>{t.text}</li>)}</ul>;
}

function AddTodoButton() {
  const { addTodo } = useTodoActions(); // Never re-renders from state changes
  return <button onClick={() => addTodo('New todo')}>Add</button>;
}
```

**Expected Output:** The `TodoList` re-renders when todos change. The `AddTodoButton` never re-renders from state changes because it only consumes the actions context, whose value is stable.

**Why This Output Occurs:** The actions context value is memoised with an empty dependency array, so it never changes. The state context value changes when `todos` changes. Components consuming only actions are never notified of state changes. This pattern is particularly valuable when action functions are expensive to create or when many components need to dispatch but not read. 

### Real-World Cases

- **SaaS dashboards:** Splitting `ThemeContext`, `AuthContext`, and `DashboardConfigContext` so that theme changes do not re-render the entire dashboard.
- **E-commerce:** Separating `CartStateContext` from `CartActionsContext` so that the "Add to Cart" button (which dispatches) does not re-render when the cart contents change.
- **Design systems:** Providing a `LocaleContext` for internationalisation that is separate from the `ThemeContext` to prevent unnecessary re-renders.
- **Multi-step forms:** Using local state for the current step and context only for shared form data across steps.
- **Notification systems:** Separating notification state from notification actions to keep the notification bell (which reads count) independent of the notification list (which reads items).

---

## Core Concept 2: Server-State Separation

### Definitions

**Core Definition:** Server-state separation is the architectural practice of managing all API-fetched data through dedicated data-fetching libraries (TanStack Query, SWR, RTK Query) rather than duplicating it in global client stores.

**Technical Definition:** Server state is data that originates from a remote source (API, database), is asynchronous, cached, and owned by the server. Client state is data that originates in the browser, is synchronous, ephemeral, and owned by the client. These two categories have fundamentally different lifecycles: server state needs caching, deduplication, background refetching, stale-while-revalidate logic, loading and error states, and optimistic updates; client state needs none of these.  Mixing server state into a global client store (Redux, Zustand) forces manual cache invalidation, loading state tracking, and stale-while-revalidate logic that a dedicated server-state library handles automatically.  TanStack Query and SWR provide declarative hooks (`useQuery`, `useMutation`, `useSWR`) that manage the full server-state lifecycle, while the client store remains focused on UI state (modals, tabs, form drafts, preferences). 

**Beginner-Friendly Explanation:** Data from your API is not the same as UI state. API data needs caching, refetching, and loading states—a lot of complex logic. UI state is simple: which tab is open, whether a modal is visible. Don't put API data in Redux or Zustand. Use TanStack Query or SWR instead. They handle all the hard parts for you: caching, deduplication, background updates, and optimistic mutations. Your global store should only hold actual client state—things that live only in the browser.

### Purposes

- To eliminate manual cache invalidation, loading state tracking, and stale-while-revalidate logic from global stores.
- To provide automatic caching, deduplication, and background refetching for API data.
- To enable optimistic updates with automatic rollback on error.
- To separate concerns: server cache management in one library, UI state in another.
- To prevent the common anti-pattern of duplicating API data in multiple stores.
- To leverage framework-level features like query keys, stale times, and prefetching.

### Syntax Rules and Structure

**General Syntax with TanStack Query:**

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query: fetch and cache server data
function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((res) => res.json()),
    staleTime: 5 * 60 * 1000, // Data is fresh for 5 minutes
  });
}

// Mutation: update server data with optimistic updates
function useAddUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (newUser) =>
      fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(newUser),
      }).then((res) => res.json()),
    onMutate: async (newUser) => {
      await queryClient.cancelQueries({ queryKey: ['users'] });
      const previous = queryClient.getQueryData(['users']);
      queryClient.setQueryData(['users'], (old) => [...old, newUser]);
      return { previous };
    },
    onError: (err, newUser, context) => {
      queryClient.setQueryData(['users'], context.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

// Component
function UserList() {
  const { data: users, isLoading, isError } = useUsers();
  const addUser = useAddUser();

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Failed to load users.</p>;

  return (
    <div>
      <button onClick={() => addUser.mutate({ name: 'New User' })}>Add User</button>
      <ul>{users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>
    </div>
  );
}
```

**Component Breakdown:**
- `useQuery`: Fetches and caches server data. `queryKey` uniquely identifies the data.
- `staleTime`: How long data is considered fresh before refetching.
- `useMutation`: Handles server mutations (POST, PUT, DELETE).
- `onMutate`: Applies an optimistic update before the server responds.
- `onError`: Rolls back the optimistic update if the mutation fails.
- `onSettled`: Invalidates the query cache to refetch fresh data after the mutation.

**General Syntax with SWR:**

```jsx
import useSWR, { mutate } from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Failed to load.</p>;

  return <h1>{data.name}</h1>;
}
```

**Component Breakdown:**
- `useSWR(key, fetcher)`: Fetches data, caches it, and revalidates on focus/reconnect.
- `mutate`: Imperatively updates the cache.
- SWR is lighter than TanStack Query but offers fewer features (no built-in mutation hook, simpler cache control).

**Syntax Rules:**
- Never duplicate API data in a global client store. If data comes from the server, it belongs in TanStack Query or SWR.
- Use `queryKey` (TanStack Query) or a unique URL (SWR) to identify cached data.
- Set `staleTime` appropriately: 0 for real-time data, 5–30 minutes for relatively static data.
- Use `onMutate` / `onError` / `onSettled` for optimistic updates with rollback.
- Invalidate queries after mutations to trigger refetching.
- Keep client state (UI state) in Zustand/Redux; keep server state in the data-fetching library. 

**Constraints and Limitations:**
- TanStack Query and SWR add bundle size and learning curve.
- Server state libraries do not replace a global store for client state; both are needed in complex apps.
- Optimistic updates require careful rollback logic and can cause UI flicker if not implemented correctly.
- Cache invalidation across related queries (e.g., a mutation on "posts" affecting "post-detail") must be managed manually.
- Server Components in Next.js provide an alternative to client-side server-state libraries for initial data loading.

### Annotated Code Examples

**Example 1: Separating Server State (TanStack Query) from Client State (Zustand)**

```jsx
// server-state.ts — TanStack Query
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then((res) => res.json()),
    staleTime: 10 * 60 * 1000,
  });
}

export function useAddProduct() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (product) =>
      fetch('/api/products', { method: 'POST', body: JSON.stringify(product) }).then((res) => res.json()),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['products'] }),
  });
}
```

```jsx
// client-state.ts — Zustand (UI state only)
import { create } from 'zustand';

export const useUIStore = create((set) => ({
  cartOpen: false,
  toggleCart: () => set((state) => ({ cartOpen: !state.cartOpen })),
  selectedCategory: 'all',
  setSelectedCategory: (category) => set({ selectedCategory: category }),
}));
```

```jsx
// ProductPage.tsx — Combines both
import { useProducts, useAddProduct } from './server-state';
import { useUIStore } from './client-state';

function ProductPage() {
  const { data: products, isLoading } = useProducts();
  const addProduct = useAddProduct();
  const { selectedCategory, setSelectedCategory } = useUIStore();

  if (isLoading) return <p>Loading products...</p>;

  const filtered = products.filter(
    (p) => selectedCategory === 'all' || p.category === selectedCategory
  );

  return (
    <div>
      <select value={selectedCategory} onChange={(e) => setSelectedCategory(e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
      </select>
      <button onClick={() => addProduct.mutate({ name: 'New Product' })}>Add Product</button>
      <ul>{filtered.map((p) => <li key={p.id}>{p.name}</li>)}</ul>
    </div>
  );
}
```

**Expected Output:** The product list loads from the server and is cached by TanStack Query. The category filter is stored in Zustand (client state) and does not require a server round-trip. Adding a product triggers a mutation and invalidates the product query, causing a refetch.

**Why This Output Occurs:** Server data (`products`) lives in TanStack Query, which handles caching, loading states, and refetching. Client UI state (`selectedCategory`, `cartOpen`) lives in Zustand, which is synchronous and ephemeral. The two state sources are independent: changing the category does not affect the server cache, and refetching products does not reset the selected category. 

### Real-World Cases

- **E-commerce:** Product catalog in TanStack Query; cart UI state (open/closed) in Zustand.
- **SaaS dashboards:** Analytics data in TanStack Query; dashboard layout preferences in Zustand.
- **Social media:** Feed data in TanStack Query; notification panel state in Zustand.
- **Admin panels:** User lists in TanStack Query; filter sidebar state in a local store.
- **Multi-tenant apps:** Tenant data in TanStack Query; current tenant selection in URL state.

---

## Core Concept 3: Global Stores & Event-Driven State

### Definitions

**Core Definition:** Global stores are libraries that manage client-side state shared across the entire application, with different architectural approaches: flux (Redux Toolkit), simplified store (Zustand), atomic (Jotai), and proxy-based (Valtio).

**Technical Definition:** Global state management libraries differ in their core model. **Redux Toolkit** implements the Flux pattern with actions and reducers, providing predictable state transitions, DevTools time-travel debugging, and a massive ecosystem. **Zustand** provides a simplified store: a single `create` call returns a hook, with no actions, reducers, or providers required. **Jotai** uses an atomic model: small atoms of state compose bottom-up, with derived atoms computed from other atoms. **Valtio** uses a mutable proxy: state is a plain JavaScript object wrapped in a proxy, and components subscribe to snapshots. Performance characteristics differ: Zustand re-renders only subscribing components, Jotai re-renders only atom-reading components, and Valtio re-renders only snapshot-reading components.  Cross-tab synchronization is handled by the BroadcastChannel API (with `localStorage` fallback) or by library-specific middleware (e.g., Zustand's persist middleware with `storage` events). 

**Beginner-Friendly Explanation:** You have four main choices for global state. Redux Toolkit is the structured, enterprise option—great for large teams and complex workflows, but verbose. Zustand is the simple, lightweight option—one file, one store, no boilerplate. Jotai is the atomic option—define small pieces of state that compose together, like LEGO bricks. Valtio is the proxy option—you mutate a plain object and components update automatically. For cross-tab sync, use BroadcastChannel to broadcast state changes to all open tabs.

### Purposes

- To share client state (UI preferences, modals, drafts) across the entire application.
- To choose the right store architecture based on team size, complexity, and performance needs.
- To prevent unnecessary re-renders through selectors and fine-grained subscriptions.
- To synchronize state across multiple browser tabs and windows.
- To persist state across page reloads with middleware.
- To integrate with DevTools for debugging and time-travel.

### Syntax Rules and Structure

**Zustand (Simplified Store):**

```jsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useStore = create(
  persist(
    (set, get) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 })),
      decrement: () => set((state) => ({ count: state.count - 1 })),
      reset: () => set({ count: 0 }),
    }),
    { name: 'counter-storage' }
  )
);

// Usage — no Provider needed
function Counter() {
  const count = useStore((state) => state.count); // Selector: only re-renders when count changes
  const increment = useStore((state) => state.increment);

  return <button onClick={increment}>{count}</button>;
}
```

**Component Breakdown:**
- `create((set, get) => ({ ... }))`: Creates a store with state and actions.
- `persist`: Middleware that persists state to `localStorage`.
- `useStore((state) => state.count)`: Selector that subscribes only to `count`.
- No Provider component is required; the store is global by default.

**Jotai (Atomic State):**

```jsx
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);
const doubledAtom = atom((get) => get(countAtom) * 2);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [doubled] = useAtom(doubledAtom);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubled}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </div>
  );
}
```

**Component Breakdown:**
- `atom(0)`: Creates a primitive atom with initial value 0.
- `atom((get) => get(countAtom) * 2)`: Creates a derived atom that computes from `countAtom`.
- `useAtom(countAtom)`: Subscribes the component to the atom.
- Only components reading an atom re-render when that atom changes.

**Redux Toolkit (Flux Pattern):**

```jsx
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
  },
});

const store = configureStore({ reducer: { counter: counterSlice.reducer } });

// Usage with useSelector
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return <button onClick={() => dispatch(counterSlice.actions.increment())}>{count}</button>;
}
```

**Component Breakdown:**
- `createSlice`: Defines reducers and actions together.
- `configureStore`: Creates the Redux store.
- `useSelector`: Selects a slice of state; only re-renders when the selected value changes.
- Redux Toolkit requires a `<Provider store={store}>` at the app root.

**Cross-Tab Synchronization with BroadcastChannel:**

```javascript
// cross-tab-sync.js
const channel = new BroadcastChannel('app-state');

export function broadcastState(key, value) {
  channel.postMessage({ key, value });
}

export function subscribeToState(callback) {
  channel.onmessage = (event) => {
    callback(event.data.key, event.data.value);
  };
}
```

```jsx
// In a Zustand store
import { create } from 'zustand';
import { broadcastState, subscribeToState } from './cross-tab-sync';

const useStore = create((set) => ({
  theme: 'light',
  setTheme: (theme) => {
    set({ theme });
    broadcastState('theme', theme); // Broadcast to other tabs
  },
}));

// Subscribe to changes from other tabs
subscribeToState((key, value) => {
  if (key === 'theme') useStore.setState({ theme: value });
});
```

**Component Breakdown:**
- `BroadcastChannel('app-state')`: Creates a named channel for cross-tab communication.
- `channel.postMessage(...)`: Broadcasts state changes to all tabs on the same origin.
- `channel.onmessage`: Receives messages from other tabs and updates the local store.
- `localStorage` fallback: For browsers without BroadcastChannel, use the `storage` event.

**Syntax Rules:**
- Choose the store architecture based on project needs: Zustand for simplicity, Redux Toolkit for structure, Jotai for atomic composition, Valtio for mutable proxy.
- Use selectors (Zustand, Redux) or atoms (Jotai) to subscribe only to the state a component needs.
- Use `persist` middleware (Zustand) or equivalent for state persistence across reloads.
- Use BroadcastChannel (with `localStorage` fallback) for cross-tab state synchronization.
- Keep global state minimal: server state belongs in TanStack Query; local state stays local.
- Use Redux Toolkit's `createEntityAdapter` for normalized collections.
- Use Jotai for fine-grained, derived state that composes from small atoms.

**Constraints and Limitations:**
- Redux Toolkit has the steepest learning curve and the most boilerplate.
- Zustand has less structure for very large apps and a smaller middleware ecosystem than Redux.
- Jotai's atomic model can be confusing for developers used to centralized stores.
- Valtio's mutable proxy model can lead to accidental mutations if not disciplined.
- Cross-tab synchronization requires careful handling of conflict resolution.
- Persisting state to `localStorage` is not appropriate for sensitive data (tokens, PII).

### Annotated Code Examples

**Example 1: Zustand Store with Selectors and Persist**

```jsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

export const useAppStore = create(
  persist(
    (set, get) => ({
      theme: 'light',
      sidebarOpen: true,
      user: null,
      setTheme: (theme) => set({ theme }),
      toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
      setUser: (user) => set({ user }),
      logout: () => set({ user: null }),
    }),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        theme: state.theme,
        sidebarOpen: state.sidebarOpen,
      }),
    }
  )
);

// Component using selectors
function Sidebar() {
  const sidebarOpen = useAppStore((state) => state.sidebarOpen);
  const toggleSidebar = useAppStore((state) => state.toggleSidebar);

  return (
    <aside className={sidebarOpen ? 'open' : 'closed'}>
      <button onClick={toggleSidebar}>Toggle</button>
    </aside>
  );
}

function ThemeToggle() {
  const theme = useAppStore((state) => state.theme);
  const setTheme = useAppStore((state) => state.setTheme);

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      {theme}
    </button>
  );
}
```

**Expected Output:** The sidebar and theme toggle are independent: toggling the sidebar does not re-render the theme toggle, and vice versa. The theme and sidebar state persist across page reloads, but the user is not persisted (due to `partialize`).

**Why This Output Occurs:** Each component subscribes to a specific slice of the store using a selector. Zustand only re-renders a component when its selected slice changes. The `partialize` option ensures only non-sensitive state (theme, sidebar) is persisted to `localStorage`, while the user is kept in memory only. 

**Example 2: Jotai Atoms with Derived State**

```jsx
import { atom, useAtom } from 'jotai';

const todosAtom = atom([]);
const filterAtom = atom('all');

// Derived atom: filtered todos
const filteredTodosAtom = atom((get) => {
  const todos = get(todosAtom);
  const filter = get(filterAtom);

  if (filter === 'active') return todos.filter((t) => !t.completed);
  if (filter === 'completed') return todos.filter((t) => t.completed);
  return todos;
});

function TodoApp() {
  const [todos, setTodos] = useAtom(todosAtom);
  const [filter, setFilter] = useAtom(filterAtom);
  const [filtered] = useAtom(filteredTodosAtom);

  return (
    <div>
      <select value={filter} onChange={(e) => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="active">Active</option>
        <option value="completed">Completed</option>
      </select>
      <ul>{filtered.map((t) => <li key={t.id}>{t.text}</li>)}</ul>
    </div>
  );
}
```

**Expected Output:** Changing the filter updates the filtered list without re-rendering the entire app. Only components reading the affected atoms re-render.

**Why This Output Occurs:** Jotai's atomic model means that `filteredTodosAtom` is derived from `todosAtom` and `filterAtom`. When `filterAtom` changes, Jotai recomputes `filteredTodosAtom` and notifies only the components that read it. This is fine-grained reactivity without selectors or memoisation. 

### Real-World Cases

- **E-commerce:** Zustand for cart UI state (open/closed, sidebar), with BroadcastChannel to sync cart across tabs.
- **SaaS dashboards:** Redux Toolkit for complex business workflows with audit trails and DevTools debugging.
- **Design systems:** Jotai for atomic theme and locale state that composes from small pieces.
- **Interactive editors:** Valtio for mutable proxy-based state that mirrors the editor's document model.
- **Multi-tab apps:** BroadcastChannel to sync auth state, theme, and notifications across all open tabs.

---

## Core Concept 4: State Machines & Finite State Patterns

### Definitions

**Core Definition:** State machines and finite state patterns model application logic as a finite set of states, with explicit transitions between states triggered by events, eliminating impossible states and making complex flows testable and visualizable.

**Technical Definition:** A **finite state machine (FSM)** is a mathematical model with a finite set of states, a set of events, and a transition function that maps (state, event) pairs to new states. In React, XState is the primary library for state machines and statecharts (hierarchical state machines). Instead of letting state change freely through multiple `useState` calls, XState defines a machine with explicit states, events, and transitions. Each state can have entry actions, exit actions, and nested states. Statecharts add hierarchy (nested states), parallelism (concurrent states), and communication (actors). XState prevents impossible states (e.g., "loading" and "error" simultaneously) by enforcing that only one state is active at a time, and it makes complex flows testable through model-based testing. 

**Beginner-Friendly Explanation:** Imagine a checkout flow: it has states like "cart", "shipping", "payment", "review", and "confirmation". Some transitions are allowed (cart → shipping), and some are not (cart → confirmation). A state machine defines exactly which transitions are allowed. This prevents bugs like "the user somehow reached confirmation without paying." XState lets you define these machines visually and use them in React with a simple hook. It's overkill for a simple toggle but incredibly valuable for multi-step async flows.

### Purposes

- To model complex, multi-step asynchronous flows with explicit states and transitions.
- To eliminate impossible states by enforcing that only one state is active at a time.
- To make complex logic testable through model-based testing and visualizable through state diagrams.
- To handle loading, success, error, and retry states for async operations cleanly.
- To support conditional branching, guards, and actions within transitions.
- To provide a single source of truth for the entire flow's behaviour.

### Syntax Rules and Structure

**General Syntax with XState v5:**

```jsx
import { createMachine } from 'xstate';
import { useMachine } from '@xstate/react';

// Define the machine
const checkoutMachine = createMachine({
  id: 'checkout',
  initial: 'cart',
  states: {
    cart: {
      on: { CHECKOUT: 'shipping' },
    },
    shipping: {
      on: { BACK: 'cart', CONTINUE: 'payment' },
    },
    payment: {
      on: { BACK: 'shipping', PAY: 'processing' },
    },
    processing: {
      on: {
        SUCCESS: 'confirmation',
        FAILURE: 'payment',
      },
    },
    confirmation: {
      type: 'final',
    },
  },
});

// Use in a component
function CheckoutFlow() {
  const [state, send] = useMachine(checkoutMachine);

  return (
    <div>
      <p>Current step: {state.value}</p>

      {state.matches('cart') && (
        <button onClick={() => send({ type: 'CHECKOUT' })}>Proceed to Shipping</button>
      )}

      {state.matches('shipping') && (
        <div>
          <button onClick={() => send({ type: 'BACK' })}>Back</button>
          <button onClick={() => send({ type: 'CONTINUE' })}>Continue to Payment</button>
        </div>
      )}

      {state.matches('payment') && (
        <div>
          <button onClick={() => send({ type: 'BACK' })}>Back</button>
          <button onClick={() => send({ type: 'PAY' })}>Pay Now</button>
        </div>
      )}

      {state.matches('processing') && <p>Processing payment...</p>}
      {state.matches('confirmation') && <p>Order confirmed!</p>}
    </div>
  );
}
```

**Component Breakdown:**
- `createMachine({ id, initial, states })`: Defines the machine with states and transitions.
- `initial: 'cart'`: The starting state.
- `states.cart.on.CHECKOUT`: Defines that the `CHECKOUT` event transitions from `cart` to `shipping`.
- `useMachine(checkoutMachine)`: Returns the current state and a `send` function.
- `state.matches('cart')`: Checks if the machine is in the `cart` state.
- `send({ type: 'CHECKOUT' })`: Sends an event to trigger a transition.
- `type: 'final'`: Marks a terminal state where the machine stops.

**General Syntax for Async States (Loading/Success/Error):**

```jsx
const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      invoke: {
        src: 'fetchData',
        onDone: { target: 'success', actions: 'setData' },
        onError: { target: 'failure', actions: 'setError' },
      },
    },
    success: { on: { FETCH: 'loading' } },
    failure: { on: { RETRY: 'loading' } },
  },
});
```

**Component Breakdown:**
- `invoke`: Runs an async function (`fetchData`) when entering the `loading` state.
- `onDone`: Transitions to `success` and runs the `setData` action when the promise resolves.
- `onError`: Transitions to `failure` and runs the `setError` action when the promise rejects.
- This pattern cleanly models the full async lifecycle without multiple `useState` calls.

**Syntax Rules:**
- Use state machines for flows with three or more states and non-trivial transition logic.
- Use `createMachine` to define states, events, and transitions declaratively.
- Use `useMachine` from `@xstate/react` to connect the machine to a component.
- Use `state.matches('stateName')` to check the current state in JSX.
- Use `send({ type: 'EVENT' })` to trigger transitions.
- Use `invoke` for asynchronous operations (API calls, timers) within states.
- Use guards (`cond`) for conditional transitions.
- Use actions (`entry`, `exit`) for side effects tied to state transitions.
- Use `type: 'final'` for terminal states.

**Constraints and Limitations:**
- XState adds a learning curve and bundle size (~30KB).
- For simple toggles and booleans, `useState` is simpler and more appropriate.
- Overusing state machines for every piece of state leads to unnecessary complexity.
- XState v5 changed the API from v4; older tutorials may show `Machine()` instead of `createMachine()`.
- Testing state machines requires different patterns than testing components; use model-based testing for comprehensive coverage.

### Annotated Code Examples

**Example 1: Toggle Machine (Simple)**

```jsx
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

function Toggler() {
  const [state, send] = useMachine(toggleMachine);

  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.matches('inactive') ? 'Off' : 'On'}
    </button>
  );
}
```

**Expected Output:** The button displays "Off" initially. Clicking it sends the `TOGGLE` event, transitioning to the `active` state and displaying "On". Clicking again returns to "Off".

**Why This Output Occurs:** The machine has two states (`inactive` and `active`) and one event (`TOGGLE`) that transitions between them. `state.matches('inactive')` determines the button label. The machine enforces that only valid transitions occur—there is no way to reach an undefined state. 

**Example 2: Authentication Flow Machine (Complex)**

```jsx
import { createMachine, assign } from 'xstate';
import { useMachine } from '@xstate/react';

const authMachine = createMachine({
  id: 'auth',
  initial: 'idle',
  context: { user: null, error: null },
  states: {
    idle: {
      on: { LOGIN: 'authenticating' },
    },
    authenticating: {
      invoke: {
        src: 'loginUser',
        onDone: {
          target: 'authenticated',
          actions: assign({ user: (_, event) => event.data, error: null }),
        },
        onError: {
          target: 'error',
          actions: assign({ error: (_, event) => event.data.message }),
        },
      },
    },
    authenticated: {
      on: { LOGOUT: 'idle' },
    },
    error: {
      on: { RETRY: 'authenticating', LOGIN: 'authenticating' },
    },
  },
});

function AuthFlow() {
  const [state, send] = useMachine(authMachine, {
    services: {
      loginUser: async (context, event) => {
        const res = await fetch('/api/login', {
          method: 'POST',
          body: JSON.stringify(event.credentials),
        });
        if (!res.ok) throw new Error('Invalid credentials');
        return res.json();
      },
    },
  });

  return (
    <div>
      {state.matches('idle') && (
        <button onClick={() => send({ type: 'LOGIN', credentials: { email: 'a@b.com', password: 'secret' } })}>
          Log In
        </button>
      )}
      {state.matches('authenticating') && <p>Logging in...</p>}
      {state.matches('authenticated') && (
        <div>
          <p>Welcome, {state.context.user.name}</p>
          <button onClick={() => send({ type: 'LOGOUT' })}>Log Out</button>
        </div>
      )}
      {state.matches('error') && (
        <div>
          <p role="alert">{state.context.error}</p>
          <button onClick={() => send({ type: 'RETRY' })}>Try Again</button>
        </div>
      )}
    </div>
  );
}
```

**Expected Output:** The component displays "Log In" initially. Clicking it transitions to "Logging in..." then either "Welcome, Alice" or an error message with a "Try Again" button. The machine enforces that the user cannot be in two states at once (e.g., both authenticated and error).

**Why This Output Occurs:** The `invoke` block calls the `loginUser` service. If it resolves, the machine transitions to `authenticated` and stores the user in context. If it rejects, it transitions to `error` and stores the error message. The `assign` action updates the context. This pattern cleanly separates the async operation, state transitions, and context management without multiple `useState` hooks. 

### Real-World Cases

- **E-commerce checkout:** Multi-step flows with cart, shipping, payment, review, and confirmation states.
- **Authentication:** Login, MFA challenge, token refresh, logout flows with error and retry states.
- **Multi-step forms:** Onboarding wizards with conditional branching and validation states.
- **Media players:** Play, pause, buffering, error, and ended states.
- **File uploads:** Idle, uploading, processing, success, error, and retry states.
- **Payment processing:** Payment initiation, 3D Secure challenge, authorisation, capture, and refund states.

---

## Core Concept 5: Derived State Patterns

### Definitions

**Core Definition:** Derived state is any value that can be computed from existing state or props during render; the derived state pattern states that such values should be computed on-the-fly (with `useMemo` for expensive computations) rather than stored in separate state and synchronised via `useEffect`.

**Technical Definition:** Derived state is a value that is a pure function of other state or props. Examples include filtered lists, formatted strings, computed totals, and validation errors. The anti-pattern is storing derived state in `useState` and using `useEffect` to keep it in sync with the source state. This causes a **double-render cycle**: the first render uses stale derived state, then the effect fires and sets state again, triggering a second render. It also introduces **stale frames** (the UI briefly shows outdated derived values) and **drift** (the derived state can become out of sync with its source).  The fix is to compute derived values during render, using `useMemo` only when the computation is expensive. This eliminates the extra render, prevents stale frames, and ensures a single source of truth. 

**Beginner-Friendly Explanation:** If you can calculate something from other state, don't store it in separate state. For example, if you have a list of items and a search query, the filtered list is derived from both—don't create a `filteredItems` state and update it with an effect every time the query changes. Just calculate it during render. If the calculation is heavy (filtering 10,000 items), wrap it in `useMemo`. This avoids double renders, stale data, and synchronization bugs.

### Purposes

- To eliminate double-render cycles caused by `useEffect` + `useState` synchronization.
- To prevent stale frames where the UI briefly shows outdated derived values.
- To maintain a single source of truth for each piece of state.
- To simplify component logic by removing synchronization effects.
- To improve performance by avoiding unnecessary state updates and re-renders.
- To make data flow explicit and predictable: derived values are computed, not stored.

### Syntax Rules and Structure

**The Anti-Pattern (Don't Do This):**

```jsx
// ❌ ANTI-PATTERN: Syncing derived state with useEffect
function FilteredList({ items }) {
  const [query, setQuery] = useState('');
  const [filteredItems, setFilteredItems] = useState(items);

  // This effect runs AFTER render, causing a double render
  useEffect(() => {
    setFilteredItems(items.filter((item) => item.includes(query)));
  }, [items, query]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>{filteredItems.map((item) => <li key={item}>{item}</li>)}</ul>
    </div>
  );
}
```

**Component Breakdown:**
- `filteredItems` is derived from `items` and `query`.
- Storing it in state and syncing via `useEffect` causes a double render: first with stale `filteredItems`, then with the updated value.
- The user may see a stale frame (old filtered items briefly) before the effect runs.

**The Fix (Compute During Render):**

```jsx
// ✅ CORRECT: Compute during render (with useMemo for expensive computations)
function FilteredList({ items }) {
  const [query, setQuery] = useState('');

  // Derived value computed during render — no effect, no extra state
  const filteredItems = useMemo(
    () => items.filter((item) => item.includes(query)),
    [items, query]
  );

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>{filteredItems.map((item) => <li key={item}>{item}</li>)}</ul>
    </div>
  );
}
```

**Component Breakdown:**
- `filteredItems` is computed during render using `useMemo`.
- No `useEffect`, no `useState` for the derived value.
- The computation only runs when `items` or `query` changes.
- No double render, no stale frame, no drift.

**General Syntax for Expensive Derived State:**

```jsx
const expensiveResult = useMemo(() => {
  // Expensive computation
  return heavyFunction(inputA, inputB);
}, [inputA, inputB]);
```

**Component Breakdown:**
- `useMemo` caches the result and only recomputes when dependencies change.
- For cheap computations, `useMemo` is unnecessary overhead—just compute inline.
- For very expensive computations, `useMemo` prevents recomputation on unrelated re-renders.

**Syntax Rules:**
- Never store derived state in `useState` and sync it with `useEffect`.
- Compute derived values inline for cheap computations.
- Use `useMemo` for expensive computations (filtering large arrays, complex formatting).
- Derived state should be a pure function of existing state or props.
- If a value cannot be derived (e.g., it comes from an API), it is source state, not derived state.
- URL state should be read directly, not copied into local state. 

**Constraints and Limitations:**
- `useMemo` is not a guarantee—React may discard cached values.
- Over-using `useMemo` for cheap computations adds unnecessary overhead.
- Derived state patterns do not apply to asynchronous derivations (those require a different approach).
- For very complex derived state, consider moving the computation to a web worker or a state machine.

### Annotated Code Examples

**Example 1: Filtered List (Anti-Pattern vs. Fix)**

```jsx
// ❌ ANTI-PATTERN: Double render + stale frame
function SearchResults({ items }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  useEffect(() => {
    setResults(items.filter((item) => item.name.includes(query)));
  }, [items, query]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>{results.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
    </div>
  );
}

// ✅ FIX: Compute during render
function SearchResults({ items }) {
  const [query, setQuery] = useState('');

  const results = useMemo(
    () => items.filter((item) => item.name.includes(query)),
    [items, query]
  );

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>{results.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
    </div>
  );
}
```

**Expected Output:** The anti-pattern renders twice per keystroke: once with the old `results`, then again after the effect updates `results`. The fix renders once per keystroke with the correct `results`.

**Why This Output Occurs:** In the anti-pattern, `setQuery` triggers a render with the new query but the old `results`. Then the effect runs, calls `setResults`, and triggers a second render with the updated results. In the fix, `results` is computed during the first render, so only one render occurs. 

**Example 2: Computed Total (No State, No Effect)**

```jsx
// ❌ ANTI-PATTERN: Derived total stored in state
function Cart({ items }) {
  const [total, setTotal] = useState(0);

  useEffect(() => {
    setTotal(items.reduce((sum, item) => sum + item.price * item.quantity, 0));
  }, [items]);

  return <p>Total: ${total}</p>;
}

// ✅ FIX: Compute during render
function Cart({ items }) {
  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);

  return <p>Total: ${total}</p>;
}
```

**Expected Output:** The anti-pattern shows "Total: $0" on the first render (stale), then updates to the correct total. The fix shows the correct total immediately.

**Why This Output Occurs:** The anti-pattern initialises `total` to 0 and only updates it in an effect, causing a stale first render. The fix computes the total during render, so it is always correct. 

### Real-World Cases

- **E-commerce:** Computing cart totals, discount amounts, and shipping estimates from line items.
- **Search interfaces:** Filtering and sorting lists based on query and sort criteria.
- **Dashboards:** Computing aggregate metrics from raw data points.
- **Forms:** Deriving validation errors and field states from form values.
- **Data tables:** Computing pagination slices, sorted orders, and filtered rows.
- **URL state:** Deriving the current page from URL search parameters without copying them into state.

---

## References

- Separating Actions from State with Two Contexts – Steve Kinney: https://stevekinney.com/courses/react-performance/separating-actions-from-state-two-contexts
- Context Performance Optimization – belos-street/skill-kit (GitHub): https://github.com/belos-street/skill-kit/blob/main/skills/react-best-practices/reference/context-performance-optimization.md
- State Management – rnavarych/alpha-engineer (GitHub): https://github.com/rnavarych/alpha-engineer/blob/main/plugins/roles/role-frontend/skills/state-management/SKILL.md
- State Management in 2026: Beyond Redux – PkgPulse: https://www.pkgpulse.com/guides/state-management-2026
- Managing State in DHTMLX React Gantt and React Scheduler with XState – DHTMLX: https://dhtmlx.com/blog/using-xstate-in-react-gantt-and-scheduler-apps-for-complex-state-scenarios/
- derived-state: compute in render vs sync with useEffect – dev48v/derived-state (GitHub): https://github.com/dev48v/derived-state
- TanStack Query v5 and Zustand v5 Patterns – voku/agent-skills (GitHub): https://github.com/voku/agent-skills/blob/main/skills/state-management/SKILL.md
- Server-State vs Client-State Separation – pproenca/dot-skills (GitHub): https://github.com/pproenca/dot-skills/blob/main/skills/.experimental/react-optimise/references/sub-server-client-separation.md
- XState React Integration – XState Documentation: https://xstate.js.org/docs/recipes/react.html
- @xstate/react – XState Documentation: https://xstate.js.org/docs/packages/xstate-react/
- React State Management Compared: Redux vs Zustand vs Jotai vs Valtio – DEV Community: https://dev.to/this-is-learning/react-state-management-compared-redux-vs-zustand-vs-jotai-vs-valtio-2b3d
- Cross-Tab State Synchronization – tabstatesync (npm): https://www.npmjs.com/package/tabstatesync
- Cross-Tab State Synchronization – @tkhdev/cross-tab (npm): https://www.npmjs.com/package/@tkhdev/cross-tab
- Zustand Documentation – Persist Middleware: https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- Jotai Documentation – Atoms: https://jotai.org/docs/core/atom
- Redux Toolkit Documentation: https://redux-toolkit.js.org/
- TanStack Query Documentation – Optimistic Updates: https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- SWR Documentation: https://swr.vercel.app/
- React Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure