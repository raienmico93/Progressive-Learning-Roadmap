# React Component Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React component patterns are reusable architectural solutions for common component design problems in React applications.

**Technical Definition:** React component patterns are established structural approaches used to organize components, manage state, and define clear responsibility boundaries within a React application. They are not features of the framework itself but conventions adopted by developers to address recurring structural challenges. When similar coordination problems appear across projects—such as duplicated logic, unclear state ownership, or tightly coupled components—patterns provide consistent, repeatable solutions. Patterns determine how code is arranged, clarify where logic belongs, how components interact, and how data flows across the tree.

**Beginner-Friendly Explanation:** Component patterns are like recipes for building components. They are proven ways to solve common problems, such as "how do I share logic between components?" or "how do I make a component flexible?" Instead of inventing your own solution each time, you can use a pattern that other developers have already figured out.

### Key Characteristics

- **Composition over configuration:** Prefer composing smaller components over adding boolean props.
- **Separation of concerns:** Clear boundaries between logic, rendering, and state.
- **Reusability:** Patterns enable logic and UI reuse across components.
- **Scalability:** Patterns help codebases grow without becoming tangled.
- **Flexibility:** Patterns provide APIs that consumers can adapt to their needs.

### Prerequisites

- JavaScript functions, objects, and ES6 syntax.
- React function components and props.
- JSX syntax and expressions.
- React state and event handling.
- Basic understanding of React Context.

### Related Programming Areas

- **Composition** — the foundation of all React patterns.
- **Props** — the primary data-passing mechanism.
- **Context** — sharing state without prop drilling.
- **Hooks** — reusable stateful logic.
- **State management** — reducers, providers, and lifting state.

---

## 1. Container/Presentational Pattern

### Definitions

**Core Definition:** The Container/Presentational pattern separates components into two categories: containers that handle logic and presentational components that handle rendering.

**Technical Definition:** The Container and presentation pattern aims to separate the presentation logic from the business logic in a React codebase, thereby making it modular, testable, and one that follows the separation of concerns principle. The container component acts as the component responsible for data fetching or computation. The presentation component's job is to render the fetched data or computed value on the UI.

**Beginner-Friendly Explanation:** Container components are the "brains" that fetch data and manage logic. Presentational components are the "looks" that just display what they are given.

### Purposes

- To separate data fetching and business logic from UI rendering.
- To make UI components reusable across different data sources.
- To simplify testing of pure rendering logic.
- To allow designers and developers to work on UI independently.

### Syntax Rules and Structure

**General Syntax (Container):**

```jsx
function ContainerComponent() {
  const [data, setData] = useState(null);
  useEffect(() => { fetchData().then(setData); }, []);
  return <PresentationalComponent data={data} />;
}
```

**General Syntax (Presentational):**

```jsx
function PresentationalComponent({ data }) {
  return <div>{data ? data.name : 'Loading...'}</div>;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Container | Fetches data, manages state, passes props. |
| Presentational | Renders UI purely from props. |

**Syntax Rules:**

1. Container components manage state and side effects.
2. Presentational components receive data exclusively via props.
3. Presentational components have no data fetching logic.

**Constraints and Limitations:**

- With Hooks, the boundary between container and presentational has blurred.
- Over-splitting can lead to too many small files.

### Annotated Code Examples

```jsx
// Container: fetches characters from an API
function StarWarsCharactersContainer() {
  const [characters, setCharacters] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(false);

  useEffect(() => {
    setIsLoading(true);
    fetch('https://akabab.github.io/starwars-api/api/all.json')
      .then((res) => res.json())
      .then((data) => { setCharacters(data); setIsLoading(false); })
      .catch(() => { setError(true); setIsLoading(false); });
  }, []);

  return (
    <CharacterList
      loading={isLoading}
      error={error}
      characters={characters}
    />
  );
}

// Presentational: renders characters
function CharacterList({ loading, error, characters }) {
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error loading characters.</p>;
  return (
    <ul>
      {characters.map((c) => (
        <li key={c.id}>{c.name}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** A loading message, then a list of Star Wars character names.

**Why this output:** The container fetches data and passes `loading`, `error`, and `characters` as props. The presentational component renders based on these props.

### Real-World Cases

- **Data fetching pages:** User lists, product catalogs, dashboards.
- **Design systems:** All UI primitives are presentational.
- **Storybook:** Presentational components are perfect for Storybook.

### References

- Refine — React Design Patterns — https://raw.githubusercontent.com/refinedev/refine/refs/heads/main/documentation/blog/2024-09-05-react-patterns.md
- Patterns.dev — Container/Presentational Pattern — https://www.patterns.dev/vue/container-presentational

---

## 2. Higher-Order Component (HOC)

### Definitions

**Core Definition:** A higher-order component is a function that takes a component and returns a new, enhanced component.

**Technical Definition:** A HOC is a pure JavaScript function that accepts one component as an argument and returns another after injecting or adding additional data and functionality. It essentially acts as a JavaScript decorator function. The fundamental concept of HOCs aligns with React's nature, which favors composition over inheritance.

**Beginner-Friendly Explanation:** A HOC is a function that wraps a component to give it extra abilities, like authentication or data fetching.

### Purposes

- To reuse component logic across multiple components.
- To handle cross-cutting concerns (authentication, logging, data fetching).
- To add props or behavior to a component without modifying it.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function withExtraProps(WrappedComponent) {
  return function EnhancedComponent(props) {
    const extraProps = { ... };
    return <WrappedComponent {...props} {...extraProps} />;
  };
}
```

**Syntax Rules:**

1. HOCs are pure functions with no side effects.
2. Pass unrelated props through to the wrapped component.
3. Do not mutate the original component.
4. Use `displayName` for easier debugging.

**Constraints and Limitations:**

- HOC nesting can create "wrapper hell."
- Props name conflicts can occur when composing multiple HOCs.
- Modern React prefers custom Hooks over HOCs.

### Annotated Code Examples

```jsx
function withAuth(WrappedComponent) {
  return function WithAuth(props) {
    const isAuthenticated = checkAuth();
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    return <WrappedComponent {...props} />;
  };
}

// Usage
function Dashboard() {
  return <h1>Dashboard</h1>;
}

const ProtectedDashboard = withAuth(Dashboard);
```

**Expected Output:** The dashboard renders only if the user is authenticated.

**Why this output:** `withAuth` wraps `Dashboard` and checks authentication before rendering. If authenticated, it renders the wrapped component with all original props.

### Real-World Cases

- **Authentication:** `withAuth`, `withPermission`.
- **Data fetching:** `withData`, `withSubscription`.
- **Redux:** `connect`.
- **Logging:** `withLogger`.

### References

- Syncfusion — React Design Patterns — https://www.syncfusion.com/blogs/post/react-design-patterns
- React — Higher-Order Components — https://ru.react.js.org/docs/higher-order-components.html

---

## 3. Render Props

### Definitions

**Core Definition:** A render prop is a prop whose value is a function that returns JSX, allowing a component to share logic while delegating rendering to the caller.

**Technical Definition:** The render prop pattern involves passing a function as a prop (conventionally named `render`) to a component. The component calls this function with data or state, and the function returns React elements. This inverts control: the component provides the logic, and the consumer decides what to render.

**Beginner-Friendly Explanation:** A render prop is a function you pass to a component that tells it what to render. The component gives your function some data, and your function returns what to show.

### Purposes

- To share stateful logic between components.
- To invert control of rendering to the consumer.
- To provide flexible APIs where the consumer controls rendering.
- To avoid HOC nesting ("wrapper hell").

### Syntax Rules and Structure

**General Syntax:**

```jsx
<DataProvider render={(data) => <Child data={data} />} />
```

**Syntax Rules:**

1. The render prop is a function, not JSX.
2. The component calls the function with data or state.
3. The function returns React elements.

**Constraints and Limitations:**

- Render props can be less readable than custom Hooks.
- Nested render props can create "wrapper hell."
- Modern React prefers custom Hooks for logic reuse.

### Annotated Code Examples

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return render(position);
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <p>Mouse position: {x}, {y}</p>
  )}
/>
```

**Expected Output:** The mouse position updates as the user moves the mouse.

**Why this output:** `MouseTracker` provides the mouse position to the render function, which decides what to render.

### Real-World Cases

- **Legacy libraries:** Many older React libraries use render props.
- **Custom Hooks migration:** Render props can be replaced by custom Hooks.
- **Flexible rendering:** When a component needs to delegate rendering decisions.

### References

- React — Render Props — https://legacy.reactjs.org/docs/render-props.html

---

## 4. Compound Components

### Definitions

**Core Definition:** Compound components are a set of related components that work together, sharing implicit state through context.

**Technical Definition:** Compound components structure complex components as a group of smaller, focused subcomponents that share state via React Context. The parent component owns the state, and children consume it through context. This pattern is used by libraries like Radix UI and Headless UI. It avoids boolean prop proliferation and provides a clean, expressive API.

**Beginner-Friendly Explanation:** Compound components are like a set of related parts that work together. For example, a `<Tabs>` component comes with `<Tabs.List>`, `<Tabs.Tab>`, and `<Tabs.Panel>`.

### Purposes

- To create expressive, self-documenting component APIs.
- To share state between related components without prop drilling.
- To avoid boolean prop proliferation.
- To enable flexible composition of complex UI patterns.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
</Tabs>
```

**Syntax Rules:**

1. The parent component provides state via context.
2. Subcomponents consume context to access and modify state.
3. Subcomponents are exported as static properties of the parent.

### Annotated Code Examples

```jsx
import { createContext, useContext, useState } from 'react';

const TabsContext = createContext();

function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

function Tab({ value, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      aria-selected={activeTab === value}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanel({ value, children }) {
  const { activeTab } = useContext(TabsContext);
  return activeTab === value ? <div>{children}</div> : null;
}

Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage
<Tabs defaultTab="profile">
  <Tabs.Tab value="profile">Profile</Tabs.Tab>
  <Tabs.Tab value="settings">Settings</Tabs.Tab>
  <Tabs.Panel value="profile">Profile content</Tabs.Panel>
  <Tabs.Panel value="settings">Settings content</Tabs.Panel>
</Tabs>
```

**Expected Output:** A tabbed interface with Profile and Settings tabs.

**Why this output:** `Tabs` manages the active tab state and shares it via context. `Tab` and `TabPanel` consume the context to render correctly.

### Real-World Cases

- **Radix UI:** All primitives use compound components.
- **Headless UI:** Tabs, Dialog, Listbox, Menu.
- **Design systems:** Accordion, Dropdown, Select.

### References

- Patterns.dev — Compound Components — https://github.com/PatternsDev/skills/blob/main/react/react-composition-2026/SKILL.md
- LobeHub — Building Compound Components — https://lobehub.com/skills/tambo-ai-tambo-compound-components

---

## 5. Provider Pattern

### Definitions

**Core Definition:** The provider pattern uses React Context to make data available to all components in a subtree without prop drilling.

**Technical Definition:** The Provider pattern uses `React.createContext()` to create a context object, and a Provider component to supply a value to all descendant components. Consumers use `useContext()` to read the value. This pattern is ideal for sharing global data like themes, authentication, or localization.

**Beginner-Friendly Explanation:** The provider pattern lets you share data with many components without passing props through every level.

### Purposes

- To share data across many components without prop drilling.
- To provide theme, authentication, or localization data.
- To decouple data provision from data consumption.

### Syntax Rules and Structure

**General Syntax:**

```jsx
const MyContext = createContext();

function MyProvider({ children }) {
  const value = { ... };
  return <MyContext.Provider value={value}>{children}</MyContext.Provider>;
}

function useMyContext() {
  return useContext(MyContext);
}
```

**Syntax Rules:**

1. Create a context with `createContext()`.
2. Wrap the subtree with `<Context.Provider value={...}>`.
3. Consume with `useContext()`.

### Annotated Code Examples

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = { theme, setTheme };
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  return useContext(ThemeContext);
}

// Usage
function App() {
  return (
    <ThemeProvider>
      <Toolbar />
    </ThemeProvider>
  );
}

function Toolbar() {
  const { theme, setTheme } = useTheme();
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

**Expected Output:** A button that toggles between light and dark themes.

**Why this output:** `ThemeProvider` provides the theme state to all descendants. `Toolbar` consumes it via `useTheme()`.

### Real-World Cases

- **Theming:** Light/dark mode, brand colors.
- **Authentication:** Current user, permissions.
- **Localization:** Language, locale settings.

### References

- Syncfusion — React Design Patterns — https://www.syncfusion.com/blogs/post/react-design-patterns
- code-b.dev — React Design Patterns — https://code-b.dev/blog/react-design-patterns

---

## 6. Context Pattern

### Definitions

**Core Definition:** The context pattern is the broader practice of using React Context to share state, often combined with custom hooks and reducers.

**Technical Definition:** Context patterns include Context + Custom Hook (encapsulating context access in a hook), Scoped Provider (multiple instances of the same context), and Context with Reducer (combining `useReducer` with context for complex state). Custom hooks encapsulate context access and error handling, and separated contexts split state and actions to prevent unnecessary re-renders.

**Beginner-Friendly Explanation:** Context patterns are different ways to use React Context effectively, often with custom hooks and reducers.

### Purposes

- To encapsulate context access in a clean API.
- To manage complex state with reducers.
- To prevent unnecessary re-renders with split contexts.

### Syntax Rules and Structure

**General Syntax (Context + Hook + Reducer):**

```jsx
const StateContext = createContext();
const DispatchContext = createContext();

function Provider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

function useAppState() {
  const context = useContext(StateContext);
  if (!context) throw new Error('useAppState must be used within Provider');
  return context;
}
```

**Syntax Rules:**

1. Split state and dispatch into separate contexts to reduce re-renders.
2. Use custom hooks to encapsulate context access and error handling.
3. Use a null default with an error boundary to prevent silent failures.

### Annotated Code Examples

```jsx
import { createContext, useContext, useReducer } from 'react';

const StateContext = createContext(null);
const DispatchContext = createContext(null);

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.payload] };
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter((i) => i.id !== action.payload) };
    default:
      return state;
  }
}

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

function useCartState() {
  const context = useContext(StateContext);
  if (!context) throw new Error('useCartState must be used within CartProvider');
  return context;
}

function useCartDispatch() {
  const context = useContext(DispatchContext);
  if (!context) throw new Error('useCartDispatch must be used within CartProvider');
  return context;
}
```

**Expected Output:** A cart state provider that separates state and dispatch.

**Why this output:** Splitting state and dispatch into separate contexts prevents components that only dispatch from re-rendering when state changes.

### Real-World Cases

- **Shopping carts:** State and dispatch separated for performance.
- **Form wizards:** Complex state managed with reducers.
- **Notification systems:** Add/remove notifications via dispatch.

### References

- code-b.dev — React Design Patterns — https://code-b.dev/blog/react-design-patterns
- voku/agent-skills — TypeScript React Patterns — https://raw.githubusercontent.com/voku/agent-skills/be55a29c8c5fd14144130f96b8653d4c94cf7927/skills/typescript-react-patterns/AGENTS.md

---

## 7. Custom Hooks Pattern

### Definitions

**Core Definition:** Custom hooks are JavaScript functions that start with `use` and encapsulate reusable stateful logic.

**Technical Definition:** Custom hooks are functions that use React Hooks internally to encapsulate stateful logic. They allow logic to be shared across components without changing the component hierarchy. Custom hooks are the modern replacement for HOCs and render props for logic reuse. A custom hook is a function whose name starts with `use` and that may call other Hooks.

**Beginner-Friendly Explanation:** A custom hook is a reusable function that contains logic you want to share between components, like form handling or data fetching.

### Purposes

- To extract and reuse stateful logic across components.
- To replace HOCs and render props for logic reuse.
- To keep components focused on rendering.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function useCustomHook(initialValue) {
  const [value, setValue] = useState(initialValue);
  // Logic here
  return { value, setValue };
}
```

**Syntax Rules:**

1. Custom hook names must start with `use`.
2. Hooks must be called at the top level of the custom hook.
3. Custom hooks can call other Hooks.

### Annotated Code Examples

```jsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Usage
function App() {
  const [name, setName] = useLocalStorage('name', '');
  return (
    <input value={name} onChange={(e) => setName(e.target.value)} />
  );
}
```

**Expected Output:** An input whose value is persisted to localStorage.

**Why this output:** `useLocalStorage` encapsulates the logic for reading from and writing to localStorage, making it reusable across components.

### Real-World Cases

- **Form handling:** `useForm`, `useInput`.
- **Data fetching:** `useFetch`, `useQuery`.
- **Local storage:** `useLocalStorage`.
- **Debouncing:** `useDebounce`.

### References

- code-b.dev — React Design Patterns — https://code-b.dev/blog/react-design-patterns
- LinkedIn — React Component Patterns 2025 — https://www.linkedin.com

---

## 8. State Reducer Pattern

### Definitions

**Core Definition:** The state reducer pattern allows consumers to control how a component's state updates by providing a custom reducer function.

**Technical Definition:** The state reducer pattern inverts control of state updates. Instead of the component hardcoding state transitions, the consumer provides a reducer function that receives the current state and an action, and returns the new state. This allows consumers to override or extend the component's default behavior. It keeps components declarative and testable.

**Beginner-Friendly Explanation:** The state reducer pattern lets the consumer decide how a component's state changes, giving them more control.

### Purposes

- To allow consumers to customize state update behavior.
- To provide flexibility without complex prop APIs.
- To keep state logic testable and declarative.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function useToggle({ reducer = defaultReducer } = {}) {
  const [state, dispatch] = useReducer(reducer, { on: false });
  const toggle = () => dispatch({ type: 'toggle' });
  return { ...state, toggle };
}
```

**Syntax Rules:**

1. The component provides a default reducer.
2. The consumer can pass a custom reducer via props.
3. The custom reducer can override or extend default behavior.

### Annotated Code Examples

```jsx
function defaultReducer(state, action) {
  switch (action.type) {
    case 'toggle':
      return { on: !state.on };
    default:
      return state;
  }
}

function useToggle({ reducer = defaultReducer } = {}) {
  const [state, dispatch] = useReducer(reducer, { on: false });
  const toggle = () => dispatch({ type: 'toggle' });
  return { ...state, toggle };
}

// Usage with custom reducer that prevents toggling more than 3 times
function App() {
  const customReducer = (state, action) => {
    if (action.type === 'toggle' && state.count >= 3) return state;
    return defaultReducer(state, action);
  };

  const { on, toggle } = useToggle({ reducer: customReducer });
  return <button onClick={toggle}>{on ? 'ON' : 'OFF'}</button>;
}
```

**Expected Output:** A toggle button that stops toggling after 3 times.

**Why this output:** The custom reducer overrides the default behavior when the toggle count exceeds 3.

### Real-World Cases

- **Toggle components:** Allowing consumers to control toggle limits.
- **Accordions:** Allowing consumers to control which panels can open.
- **Dropdowns:** Allowing consumers to control open/close behavior.

### References

- LinkedIn — React Component Patterns 2025 — https://www.linkedin.com
- code-b.dev — React Design Patterns — https://code-b.dev/blog/react-design-patterns

---

## 9. Controlled Props Pattern

### Definitions

**Core Definition:** The controlled props pattern allows a parent component to control the state of a child component through props.

**Technical Definition:** Controlled props is a pattern where a component's state is controlled by its parent through props instead of being managed internally. The component receives its current value and a change handler from the parent. This pattern is found in built-in components like `<input />` through the `value` and `onChange` props. It gives users complete control over the state of the component.

**Beginner-Friendly Explanation:** With controlled props, the parent decides what the child's state is. The child asks the parent to change it.

### Purposes

- To give parent components full control over child state.
- To enable sharing state between sibling components.
- To provide a consistent API for controlled and uncontrolled usage.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Toggle({ isOn, onToggle }) {
  return <button onClick={() => onToggle(!isOn)}>{isOn ? 'ON' : 'OFF'}</button>;
}

// Parent controls state
function App() {
  const [isOn, setIsOn] = useState(false);
  return <Toggle isOn={isOn} onToggle={setIsOn} />;
}
```

**Syntax Rules:**

1. The component receives its value via a prop.
2. Changes are requested via a callback prop.
3. The parent owns the state.

### Annotated Code Examples

```jsx
function Toggle({ isOn, onToggle }) {
  return (
    <button onClick={() => onToggle(!isOn)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}

// Controlled usage
function App() {
  const [isOn, setIsOn] = useState(false);
  return (
    <div>
      <Toggle isOn={isOn} onToggle={setIsOn} />
      <p>Status: {isOn ? 'Active' : 'Inactive'}</p>
    </div>
  );
}
```

**Expected Output:** A toggle button and a status message that update together.

**Why this output:** The parent component owns the `isOn` state and passes it to `Toggle`. When toggled, the parent updates state, which re-renders both the button and the status message.

### Real-World Cases

- **Form inputs:** `<input value={value} onChange={...} />`.
- **Custom selects:** `<Select value={value} onChange={...} />`.
- **Accordions:** `<Accordion open={open} onToggle={...} />`.

### References

- Epic React — Control Props — https://www.epicreact.dev
- Compile-N-Run — Controlled Props — https://github.com/Compile-N-Run/Compile-N-Run

---

## 10. Proxy Component Pattern

### Definitions

**Core Definition:** A proxy component is a placeholder component that can be rendered to or from another component, often used for code splitting or lazy loading.

**Technical Definition:** A proxy component is a reusable component that acts as a placeholder, delegating rendering to another component. This pattern is commonly used for code splitting (loading components on demand) and for hot module replacement during development. The proxy delays loading the actual component until it is needed.

**Beginner-Friendly Explanation:** A proxy component is a stand-in that loads the real component when needed, which helps with performance.

### Purposes

- To enable code splitting by loading components on demand.
- To support hot module replacement during development.
- To defer loading expensive components until they are needed.

### Syntax Rules and Structure

**General Syntax:**

```jsx
const LazyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <LazyComponent />
    </Suspense>
  );
}
```

**Syntax Rules:**

1. Use `React.lazy()` to create a proxy component.
2. Wrap in `<Suspense>` with a fallback.
3. The actual component loads when rendered.

### Annotated Code Examples

```jsx
import React, { Suspense, lazy } from 'react';

// Proxy component: loads HeavyComponent on demand
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**Expected Output:** "Loading..." appears briefly, then the heavy component renders.

**Why this output:** `React.lazy()` creates a proxy that loads the actual component only when it is rendered.

### Real-World Cases

- **Code splitting:** Loading routes or heavy components on demand.
- **Hot module replacement:** Proxy components for development reloading.
- **Performance optimization:** Deferring non-critical components.

### References

- reactpatterns — Proxy Component — https://github.com/reactpatterns/reactpatterns

---

## 11. Layout Components

### Definitions

**Core Definition:** Layout components are components that define the structural arrangement of their children without controlling the content.

**Technical Definition:** Layout components focus on how elements are arranged on the page, using CSS Grid, Flexbox, or other layout systems. They typically accept `children` and render them in a specific structure. The layout pattern enables state persistence because the React component tree is maintained between page transitions. Layout components are often implemented as compound components to provide flexibility.

**Beginner-Friendly Explanation:** Layout components arrange other components on the page, like a grid or a sidebar layout.

### Purposes

- To define page structure without controlling content.
- To enable consistent layouts across pages.
- To persist state between page transitions.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function PageLayout({ header, sidebar, content }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{content}</main>
    </div>
  );
}
```

**Syntax Rules:**

1. Layout components accept content via props or children.
2. They define structure, not content.
3. They are often composed with other layout components.

### Annotated Code Examples

```jsx
function Stack({ direction = 'column', spacing = 8, children }) {
  return (
    <div
      style={{
        display: 'flex',
        flexDirection: direction,
        gap: spacing,
      }}
    >
      {children}
    </div>
  );
}

// Usage
<Stack direction="row" spacing={16}>
  <Card>Card 1</Card>
  <Card>Card 2</Card>
  <Card>Card 3</Card>
</Stack>
```

**Expected Output:** Three cards arranged horizontally with 16px gap.

**Why this output:** `Stack` is a layout component that arranges its children using flexbox. The consumer controls the content.

### Real-World Cases

- **Page layouts:** Header, sidebar, content, footer.
- **Grid systems:** Dashboard grids, card grids.
- **Stack components:** Vertical and horizontal stacking.

### References

- Next.js — Layouts — https://www.nextjs.cn
- StackOverflow — React Layout Component — https://stackoverflow.com

---

## 12. Headless Components

### Definitions

**Core Definition:** Headless components expose behavior and state without prescribing any UI markup or styling.

**Technical Definition:** A headless component does not render DOM directly. Instead, it provides an API—often through hooks or compound components—that consumers use to attach behavior to their own markup. The pattern separates the behavior layer (state, keyboard interactions, focus management, accessibility semantics) from the presentation layer (layout, styles, animation). This enables maximum flexibility: different visual structures can reuse the same behavior, styling systems remain independent, and interaction logic stays centralized and testable.

**Beginner-Friendly Explanation:** Headless components give you the logic but not the look. You build your own UI and attach the headless component's behavior to it.

### Purposes

- To separate behavior from presentation.
- To enable multiple UI implementations of the same logic.
- To maintain accessibility without prescribing markup.
- To avoid rigid, over-configured components.

### Syntax Rules and Structure

**General Syntax (Hook-based):**

```jsx
const { isOpen, toggle, getTriggerProps } = useDropdown();

<button {...getTriggerProps()}>Toggle</button>
{isOpen && <div>Content</div>}
```

**Syntax Rules:**

1. Headless components expose state and event bindings.
2. Consumers attach these to their own markup.
3. No styling is prescribed.
4. Accessibility is handled by the behavior layer.

### Annotated Code Examples

```jsx
function useDropdown() {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen((o) => !o);
  const close = () => setIsOpen(false);

  return {
    isOpen,
    toggle,
    close,
    getTriggerProps: () => ({
      onClick: toggle,
      'aria-expanded': isOpen,
    }),
    getMenuProps: () => ({
      role: 'menu',
    }),
    getItemProps: () => ({
      role: 'menuitem',
      onClick: close,
    }),
  };
}

// Usage
function MyDropdown() {
  const { isOpen, getTriggerProps, getMenuProps, getItemProps } = useDropdown();

  return (
    <div>
      <button {...getTriggerProps()}>Menu</button>
      {isOpen && (
        <ul {...getMenuProps()}>
          <li {...getItemProps()}>Item 1</li>
          <li {...getItemProps()}>Item 2</li>
        </ul>
      )}
    </div>
  );
}
```

**Expected Output:** A dropdown menu with a toggle button and items.

**Why this output:** `useDropdown` provides the behavior (open/close state, ARIA attributes, event handlers). The consumer provides the markup and styling.

### Real-World Cases

- **Downshift:** Headless autocomplete/select library.
- **React Aria:** Adobe's headless accessibility hooks.
- **TanStack Table:** Headless table logic.
- **Radix UI:** Headless UI primitives.

### References

- MUI Base UI — Headless Component Guidelines — https://github.com/pproenca/dot-skills
- Headless Component APIs — Educative — https://www.educative.io/courses/learn-react/lta/headless-component-apis

---

## 13. Polymorphic Components

### Definitions

**Core Definition:** Polymorphic components are components that can render as different HTML elements or components based on an `as` prop.

**Technical Definition:** The `as` prop is a fundamental pattern in modern React component libraries that allows you to change the underlying HTML element or component that gets rendered. For example, a `<Button as="a" href="...">` renders an anchor tag instead of a button. This pattern requires careful TypeScript typing to ensure the correct props are accepted for the rendered element. The `as` prop pattern is often compared with Radix UI's Slot-based `asChild` approach.

**Beginner-Friendly Explanation:** A polymorphic component can become a different element. For example, a Button can become a link when needed.

### Purposes

- To render the same component as different HTML elements.
- To maintain a consistent API while changing semantics.
- To improve accessibility by using the correct element.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Button as="a" href="/home">Home</Button>
<Button as="button" onClick={handleClick}>Click</Button>
```

**TypeScript Syntax:**

```tsx
type PolymorphicProps<T extends React.ElementType> = {
  as?: T;
} & React.ComponentPropsWithoutRef<T>;

function Button<T extends React.ElementType = 'button'>({
  as,
  ...props
}: PolymorphicProps<T>) {
  const Component = as || 'button';
  return <Component {...props} />;
}
```

**Syntax Rules:**

1. Use an `as` prop to specify the element type.
2. TypeScript generics ensure correct prop types.
3. Default to a sensible element (e.g., `'button'`).

### Annotated Code Examples

```tsx
type ButtonProps<T extends React.ElementType = 'button'> = {
  as?: T;
  variant?: 'primary' | 'secondary';
} & React.ComponentPropsWithoutRef<T>;

function Button<T extends React.ElementType = 'button'>({
  as,
  variant = 'primary',
  ...props
}: ButtonProps<T>) {
  const Component = as || 'button';
  return <Component className={`btn btn-${variant}`} {...props} />;
}

// Usage
<Button>Click me</Button>
<Button as="a" href="/home">Home</Button>
<Button as="button" onClick={() => alert('Clicked')}>Alert</Button>
```

**Expected Output:** A button, an anchor link, and a clickable button.

**Why this output:** The `as` prop changes the rendered element. TypeScript ensures the correct props are passed for each element type.

### Real-World Cases

- **Design systems:** Buttons that can be links.
- **Typography:** Text components that can be headings, paragraphs, or spans.
- **Layout:** Box components that can be divs, sections, or articles.

### References

- Steve Kinney — Polymorphic Components — https://stevekinney.com
- Vercel — Polymorphism — https://github.com/vercel/components.build

---

## 14. Slot Pattern

### Definitions

**Core Definition:** The slot pattern uses named props or children to pass JSX elements into specific regions of a layout component.

**Technical Definition:** The canonical name is Slot and Children as API, a composition technique where a component exposes one or more placeholders that the consumer fills with their own markup, rather than the component controlling all content internally. This pattern is used by Radix UI through its `asChild` prop, which merges props onto a consumer-supplied child element. The slot pattern solves the problem of passing 50 props to configure different content areas.

**Beginner-Friendly Explanation:** The slot pattern lets you pass content into specific "slots" of a component, like a header slot and a footer slot.

### Purposes

- To define multiple content regions in a component.
- To allow consumers to control content placement.
- To avoid boolean prop proliferation.
- To create flexible, configurable layouts.

### Syntax Rules and Structure

**General Syntax (Named Slots):**

```jsx
<Modal
  title="Confirm"
  body={<p>Are you sure?</p>}
  actions={<button>Delete</button>}
/>
```

**General Syntax (asChild):**

```jsx
<Button asChild>
  <a href="/home">Home</a>
</Button>
```

**Syntax Rules:**

1. Named slots are props that accept JSX elements.
2. The `asChild` prop merges the component's props onto its child.
3. Slots are optional; handle missing slots gracefully.

### Annotated Code Examples

```jsx
function Modal({ title, body, actions }) {
  return (
    <div className="modal">
      <div className="modal-header">
        <h2>{title}</h2>
      </div>
      <div className="modal-body">{body}</div>
      <div className="modal-footer">{actions}</div>
    </div>
  );
}

// Usage
<Modal
  title="Confirm Delete"
  body={<p>Are you sure you want to delete this item?</p>}
  actions={
    <>
      <button>Cancel</button>
      <button className="danger">Delete</button>
    </>
  }
/>
```

**Expected Output:** A modal with a title, body, and action buttons.

**Why this output:** Three named slots (`title`, `body`, `actions`) each receive JSX and are rendered in their specific locations.

### Real-World Cases

- **Modal dialogs:** Title, body, and action buttons.
- **Card components:** Header, body, and footer.
- **Page layouts:** Header, sidebar, content, footer.

### References

- tour-kit — Unified Slot — https://github.com/domidex01/tour-kit
- wolfgang/agents — Slot Pattern — https://github.com/manusco/wolfgang

---

## 15. Factory Pattern

### Definitions

**Core Definition:** The factory pattern uses functions to create components or hooks with pre-configured behavior.

**Technical Definition:** The factory pattern creates reusable component factories with isolated dependencies. It is often implemented using Higher-Order Components (HOCs) or custom hooks. A factory function returns a component or hook that has been pre-configured with specific dependencies. This pattern is useful for creating families of related components with shared behavior.

**Beginner-Friendly Explanation:** A factory function creates components for you, pre-configured with specific behavior.

### Purposes

- To create reusable component factories with isolated dependencies.
- To reduce boilerplate when creating similar components.
- To enable dependency injection in components.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function createComponent(defaultValue) {
  return function Component({ value = defaultValue }) {
    return <div>{value}</div>;
  };
}

const RedComponent = createComponent('red');
const BlueComponent = createComponent('blue');
```

**Syntax Rules:**

1. Factory functions return components or hooks.
2. Factory functions accept configuration parameters.
3. Use factories for dependency injection.

### Annotated Code Examples

```jsx
function createDataHook(baseURL) {
  return function useData(endpoint) {
    const [data, setData] = useState(null);

    useEffect(() => {
      fetch(`${baseURL}${endpoint}`)
        .then((res) => res.json())
        .then(setData);
    }, [endpoint]);

    return data;
  };
}

const useAPI = createDataHook('https://api.example.com');
const useAdminAPI = createDataHook('https://admin.example.com');

// Usage
function App() {
  const users = useAPI('/users');
  const logs = useAdminAPI('/logs');
  // ...
}
```

**Expected Output:** Data hooks pre-configured with different base URLs.

**Why this output:** The factory creates hooks with different base URLs, avoiding repetitive code.

### Real-World Cases

- **API hooks:** Pre-configured data-fetching hooks.
- **Component variants:** Creating families of components with shared behavior.
- **Testing:** Factories with mock dependencies.

### References

- React — Component Hook Factories — https://de.react.dev
- @cocajs/atomic-react — Factory Pattern — https://www.npmjs.com/package/@cocajs/atomic-react

---

## 16. Component Composition Best Practices

### Definitions

**Core Definition:** Component composition best practices are guidelines for building flexible, maintainable component APIs using composition.

**Technical Definition:** The core principle is composition over configuration. Instead of adding boolean props and conditional branches to handle every variant, compose smaller, focused components together. This makes components easier to understand, test, and extend. Key practices include replacing boolean props with composition, building compound components with context, creating explicit variant components, and using children over render props for composition.

**Beginner-Friendly Explanation:** Best practices for composition are the "do's and don'ts" that help you build components that are easy to use and maintain.

### Purposes

- To avoid boolean prop proliferation.
- To create flexible, composable component APIs.
- To keep components maintainable as requirements grow.
- To align with modern React best practices.

### Syntax Rules and Structure

**Best Practices:**

| Practice | Description |
|----------|-------------|
| Avoid boolean props | Replace `showHeader`, `showFooter` with composition. |
| Use compound components | Structure as compound components with shared context. |
| Create explicit variants | `ThreadComposer`, `EditComposer` instead of `Composer` with `isThread`. |
| Prefer children over render props | Use children for composition instead of `renderX` props. |
| Lift state | State in providers, not trapped in components. |
| Compose internals | Subcomponents access context, not props. |

**Syntax Rules:**

1. If a component has more than 3-4 boolean props, refactor to composition.
2. Compound components share state via context.
3. Explicit variants are more readable than boolean modes.

### Annotated Code Examples

```jsx
// ❌ Boolean prop proliferation
<Card
  showHeader
  showFooter
  collapsible
  bordered
  withShadow
  headerAction="close"
  size="large"
/>

// ✅ Explicit composition
<Card variant="bordered" shadow="md">
  <Card.Header>
    <h3>Title</h3>
    <Card.CloseButton />
  </Card.Header>
  <Card.Body collapsible>
    <p>Content here</p>
  </Card.Body>
  <Card.Footer>
    <Button>Save</Button>
  </Card.Footer>
</Card>
```

**Expected Output:** A card with explicit, composable parts.

**Why this output:** Each piece is explicit, testable, and independently optional. The API is self-documenting.

### Real-World Cases

- **Design systems:** All modern component libraries follow these patterns.
- **Scalable applications:** Patterns prevent code rot as features grow.
- **Team collaboration:** Clear patterns reduce inconsistency.

### References

- Patterns.dev — React Composition Patterns — https://github.com/PatternsDev/skills
- Vercel — Composition Patterns — https://raw.githubusercontent.com/vercel-labs/agent-skills

---

## References

- Refine — React Design Patterns — https://raw.githubusercontent.com/refinedev/refine/refs/heads/main/documentation/blog/2024-09-05-react-patterns.md
- Syncfusion — React Design Patterns: A Practical Guide — https://www.syncfusion.com/blogs/post/react-design-patterns
- code-b.dev — React Design Patterns for Scalable Frontend Architecture — https://code-b.dev/blog/react-design-patterns
- Patterns.dev — Modern React Composition Patterns — https://github.com/PatternsDev/skills/blob/main/react/react-composition-2026/SKILL.md
- Vercel — React Composition Patterns — https://raw.githubusercontent.com/vercel-labs/agent-skills
- LobeHub — Building Compound Components — https://lobehub.com/skills/tambo-ai-tambo-compound-components
- MUI Base UI — Headless Component Guidelines — https://github.com/pproenca/dot-skills
- Epic React — Control Props — https://www.epicreact.dev
- Steve Kinney — Polymorphic Components and the as Prop — https://stevekinney.com
- Vercel — Polymorphism — https://github.com/vercel/components.build
- reactpatterns — Proxy Component — https://github.com/reactpatterns/reactpatterns
- tour-kit — Unified Slot (Radix + Base UI) — https://github.com/domidex01/tour-kit
- wolfgang/agents — Slot Pattern (Radix Style) — https://github.com/manusco/wolfgang
- voku/agent-skills — TypeScript React Patterns — https://raw.githubusercontent.com/voku/agent-skills/be55a29c8c5fd14144130f96b8653d4c94cf7927/skills/typescript-react-patterns/AGENTS.md
- React — Render Props — https://legacy.reactjs.org/docs/render-props.html
- React — Higher-Order Components — https://ru.react.js.org/docs/higher-order-components.html
- Next.js — Layouts — https://www.nextjs.cn
- @cocajs/atomic-react — Factory Pattern — https://www.npmjs.com/package/@cocajs/atomic-react