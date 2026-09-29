# React Sibling Communication: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React sibling communication is the set of patterns and techniques for enabling two or more components at the same level of the component tree to share data, coordinate behaviour, and stay synchronised, despite React's unidirectional data flow.

**Technical Definition:** In React, sibling components cannot communicate directly because there is no built-in mechanism for one component to reference another sibling's state or props. Instead, sibling communication is achieved through one of four architectural patterns: (1) **lifted state**, where shared state is moved to the closest common ancestor and passed down via props and callbacks; (2) **shared parent state**, where the parent passes identical pieces of data or state-setters to multiple children; (3) **Context**, which bypasses manual prop-drilling by providing a shared data scope that any descendant can subscribe to; and (4) **external state stores**, which synchronise siblings by connecting them to a global store that lives outside the React component tree. Each pattern trades off simplicity against scalability: lifted state is the simplest and most explicit, while external stores are the most powerful but introduce additional dependencies and complexity.

**Beginner-Friendly Explanation:** Imagine two siblings in a family—they can't talk to each other directly without going through a parent. If the older sibling wants to tell the younger one something, they tell the parent, who then tells the younger sibling. That's lifted state: the parent holds the information and passes it down. Context is like a family group chat: instead of going through the parent every time, both siblings can read messages from a shared channel. External state stores are like a public bulletin board: anyone can read from it and write to it, without needing a parent to relay messages.

### Key Characteristics

- **No Direct Sibling-to-Sibling Communication:** React has no API for one component to directly access or modify a sibling's state.
- **Parent as Mediator:** In the lifted-state and shared-parent-state patterns, the parent acts as the central coordinator.
- **Explicit Data Flow:** Lifted state and shared parent state make data flow explicit and easy to trace.
- **Context as an Escape Hatch:** Context bypasses intermediate components, reducing prop-drilling at the cost of implicit data flow and potential re-render performance issues.
- **External Stores Decouple State from the Tree:** External stores (Redux, Zustand, Jotai) hold state outside React, allowing any component—sibling or not—to subscribe to it directly.
- **Scalability Spectrum:** Lifted state works for 2–3 siblings; Context works for feature-scoped sharing; external stores work for app-wide state.

### Prerequisites

- Solid understanding of React function components, JSX, and the `useState` Hook.
- Familiarity with props, callback props, and the component tree hierarchy.
- Working knowledge of the `useContext` Hook and Context API (for Context patterns).
- Basic understanding of external state management concepts (for store patterns).

### Related Programming Areas

- **State Management:** Coordinating state across multiple components.
- **Component Composition:** Building complex UIs from smaller, reusable pieces.
- **Prop Drilling:** The challenge of passing props through many intermediate components.
- **Performance Optimisation:** Avoiding unnecessary re-renders caused by context or store updates.
- **Architectural Patterns:** Choosing the right state ownership model for a given feature.

### Core Concepts / Features

1. Lifted State (Moving State to the Closest Common Ancestor)
2. Shared Parent State (Passing Identical Data and Setters to Siblings)
3. Context (Bypassing Prop Drilling for Shared Data Scope)
4. External State Stores (Synchronising Siblings via Global Stores)

---

## Core Concept 1: Lifted State (Moving State to the Closest Common Ancestor)

### Definitions

**Core Definition:** Lifted state is the pattern of removing state from sibling components and moving it to their closest common parent, then passing it down to both siblings via props and callbacks.

**Technical Definition:** When two sibling components need to share state, the state is "lifted" to their closest common ancestor. The parent component becomes the "single source of truth" for that state, passing the current value down to each sibling via props and providing callback functions that siblings invoke to request changes. This is one of the most common patterns in React development. The process involves three steps: (1) remove state from the child components, (2) pass hardcoded data from the common parent, and (3) add state to the common parent and pass it down together with event handlers. The parent coordinates the siblings: when one sibling triggers a state change, the parent re-renders with the new value and passes it to both siblings, keeping them synchronised.

**Beginner-Friendly Explanation:** Imagine two siblings who each have their own light switch in their bedrooms. If you want both lights to always turn on and off together, you can't have two separate switches—they'd get out of sync. Instead, you install one master switch in the hallway (the parent component). Both siblings' lights are connected to that one switch. When one sibling flips the switch, both lights change. That's lifted state: the state (the switch) is moved to a shared location (the parent), and both siblings read from it.

### Purposes

- To synchronise the state of two or more sibling components so they always reflect the same value.
- To establish a single source of truth for shared state, preventing state duplication and drift.
- To make data flow explicit and easy to trace: state flows down via props, changes flow up via callbacks.
- To coordinate sibling behaviour (e.g., only one panel expanded at a time).
- To avoid the complexity of Context or external stores when the shared state is small and local.

### Syntax Rules and Structure

**General Syntax (Parent Owns and Distributes State):**
```jsx
function Parent() {
  // Parent owns the shared state
  const [sharedValue, setSharedValue] = useState(initialValue);

  return (
    <>
      {/* Sibling 1 receives the value and a callback */}
      <SiblingA value={sharedValue} onChange={setSharedValue} />
      {/* Sibling 2 receives the same value and a callback */}
      <SiblingB value={sharedValue} onChange={setSharedValue} />
    </>
  );
}
```

**Component Breakdown:**
- `const [sharedValue, setSharedValue] = useState(initialValue)`: The parent owns the shared state.
- `value={sharedValue}`: The current value is passed to both siblings.
- `onChange={setSharedValue}`: The setter function is passed as a callback for siblings to request changes.

**General Syntax (Sibling Reads Value and Requests Change):**
```jsx
function SiblingA({ value, onChange }) {
  return (
    <button onClick={() => onChange('new value')}>
      Current: {value}
    </button>
  );
}
```

**Component Breakdown:**
- `value`: The sibling reads the current shared value.
- `onChange('new value')`: The sibling requests a change by calling the parent's callback.
- The sibling does not modify `value` directly.

**Syntax Rules:**
- Identify the closest common ancestor of the siblings that need to share state. That ancestor becomes the owner.
- Remove the state declaration from the sibling components (delete their `useState` calls).
- Add the state declaration to the parent component.
- Pass the state value down as a prop to each sibling.
- Pass the state setter (or a wrapper function) down as a callback prop to each sibling.
- The siblings should never mutate the value they receive; they should only read it and call the callback.
- Use descriptive callback names (e.g., `onTemperatureChange`, `onSelect`) rather than generic ones.

**Constraints and Limitations:**
- Lifting state up can lead to prop drilling if the state must travel through many intermediate components that do not use it.
- The parent becomes responsible for managing state that may conceptually belong to a child.
- Over-lifting state (storing everything in a top-level component) causes unnecessary re-renders and tight coupling.
- For deeply nested siblings, the closest common ancestor may be far removed, making the pattern cumbersome.

### Annotated Code Examples

**Example 1: Accordion with Coordinated Panels (Official React Pattern)**

```jsx
import { useState } from 'react';

// Child component: now a controlled component — isActive comes from the parent
function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}

// Parent component: owns the activeIndex state — the single source of truth
export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty is Kazakhstan's largest city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from the Kazakh word for "apple".
      </Panel>
    </>
  );
}
```

**Expected Output:** Two panels are displayed: "About" and "Etymology". Initially, the "About" panel is expanded (showing its content), and the "Etymology" panel shows a "Show" button. Clicking "Show" on the "Etymology" panel collapses "About" and expands "Etymology". Only one panel can be open at a time.

**Why This Output Occurs:** The `activeIndex` state lives in the parent (`Accordion`), making it the single source of truth. Each `Panel` receives `isActive` (a boolean derived from `activeIndex`) and an `onShow` callback that updates `activeIndex`. When the user clicks "Show" on the second panel, `onShow` calls `setActiveIndex(1)`, which re-renders the parent. The parent then passes `isActive={true}` to the second panel and `isActive={false}` to the first, collapsing the first and expanding the second. The two panels are now coordinated because they share the same source of truth.

**Example 2: Temperature Converter with Lifted State**

```jsx
import { useState } from 'react';

// Child component: controlled input that reports changes to the parent
function TemperatureInput({ temperature, onTemperatureChange, scale }) {
  function handleChange(e) {
    // Call the parent's callback with the new value
    onTemperatureChange(Number(e.target.value));
  }

  return (
    <fieldset>
      <legend>Enter temperature in {scale}:</legend>
      <input value={temperature} onChange={handleChange} />
    </fieldset>
  );
}

// Parent component: holds the shared temperature state
function TemperatureApp() {
  const [temperature, setTemperature] = useState(20);

  return (
    <div>
      <h1>Temperature: {temperature}°C</h1>
      <TemperatureInput
        temperature={temperature}
        onTemperatureChange={setTemperature}
        scale="Celsius"
      />
      <TemperatureInput
        temperature={temperature}
        onTemperatureChange={setTemperature}
        scale="Fahrenheit"
      />
    </div>
  );
}
```

**Expected Output:** Two temperature inputs are rendered (one labeled "Celsius" and one "Fahrenheit"). Both are pre-filled with the same value (20). When the user types in either input, both inputs update to the same value, and the heading updates accordingly.

**Why This Output Occurs:** The `temperature` state lives in the parent (`TemperatureApp`). Both `TemperatureInput` siblings receive the same `temperature` value and the same `onTemperatureChange` callback (which is `setTemperature`). When the user types in either input, the child calls `onTemperatureChange(newValue)`, which updates the parent's state. The parent re-renders, passing the updated value to both children. Both inputs stay in sync because they share the same source of truth.

### Real-World Cases

- **Tabbed interfaces:** The active tab index is owned by the parent; each tab button reports clicks via `onTabChange`, and the tab panel reads the active index.
- **Shopping carts:** The cart items state is owned by a top-level component; product list items and the cart summary both read it, and add/remove actions are reported via callbacks.
- **Form wizards:** The current step and form data are owned by the parent wizard component; each step child reads the data and reports changes via callbacks.
- **Filterable lists:** The filter criteria state is owned by the parent; the filter controls report changes, and the list component reads the filter to display filtered results.
- **Accordions and expandable panels:** The active panel index is owned by the parent; each panel reports clicks via `onShow`.

### References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Lifting State Up (Legacy): https://legacy.reactjs.org/docs/lifting-state-up.html
- CoreUI – How to Lift State Up in React: https://coreui.io/answers/how-to-lift-state-up-in-react/
- Epic React – Intro to Lifting State: https://www.epicreact.dev/

---

## Core Concept 2: Shared Parent State (Passing Identical Data and Setters to Siblings)

### Definitions

**Core Definition:** Shared parent state is the pattern of a parent component passing the same piece of state (or state-setter) down to multiple sibling components, enabling them to read the same value or request changes through the same mechanism.

**Technical Definition:** Shared parent state is a specific application of lifted state where the parent passes *identical* props or callbacks to multiple children. Unlike lifted state, which focuses on *moving* state to the parent, shared parent state emphasises the *distribution* of that state to siblings. The parent passes the same `value` prop to each sibling, and the same `onChange` callback (often the state setter itself, or a wrapper around it). This ensures that all siblings see the same value and that any change requested by one sibling is reflected in all others. The pattern is particularly useful when siblings need to display the same data in different formats (e.g., a temperature in Celsius and Fahrenheit) or when they need to coordinate their behaviour (e.g., a search input and a results list).

**Beginner-Friendly Explanation:** Imagine a parent giving each of their children a copy of the same newspaper. When the parent gets a new edition, all children get the updated copy. The children can also ask the parent to change what's in the newspaper, and the parent will update all copies. That's shared parent state: the parent holds the data, gives the same copy to each sibling, and handles requests for changes.

### Purposes

- To ensure that multiple sibling components always display the same piece of data.
- To synchronise sibling behaviour without requiring Context or external stores.
- To provide a simple, explicit mechanism for coordinating siblings when the shared state is small and local.
- To avoid the overhead of Context or external stores for simple sharing scenarios.
- To make the parent the single coordinator of shared state, keeping data flow predictable.

### Syntax Rules and Structure

**General Syntax (Parent Passes Identical Props to Siblings):**
```jsx
function Parent() {
  const [sharedValue, setSharedValue] = useState(initialValue);

  return (
    <>
      {/* Both siblings receive the same value and the same callback */}
      <SiblingA value={sharedValue} onChange={setSharedValue} />
      <SiblingB value={sharedValue} onChange={setSharedValue} />
    </>
  );
}
```

**Component Breakdown:**
- `sharedValue`: The same value is passed to both siblings.
- `setSharedValue`: The same setter is passed to both siblings.
- Both siblings have access to the same data and the same update mechanism.

**General Syntax (Sibling Reads and Writes Shared State):**
```jsx
function SiblingA({ value, onChange }) {
  return (
    <div>
      <p>Sibling A sees: {value}</p>
      <button onClick={() => onChange(value + 1)}>Increment</button>
    </div>
  );
}

function SiblingB({ value, onChange }) {
  return (
    <div>
      <p>Sibling B sees: {value}</p>
      <button onClick={() => onChange(value - 1)}>Decrement</button>
    </div>
  );
}
```

**Component Breakdown:**
- Both siblings read `value` and display it.
- Both siblings call `onChange` to request changes.
- The parent updates `sharedValue`, and both siblings re-render with the new value.

**Syntax Rules:**
- Pass the *same* value prop to each sibling that needs to read the shared state.
- Pass the *same* callback prop to each sibling that needs to request changes.
- If siblings need different update logic (e.g., one increments, one decrements), pass wrapper functions or use the same setter with different arguments.
- Use descriptive prop names that reflect the shared data (e.g., `temperature`, `selectedId`, `filterText`).
- Avoid passing the raw setter directly if the sibling could set an invalid value; wrap it in a validation function.

**Constraints and Limitations:**
- All siblings re-render when the shared state changes, even if some siblings don't use the changed value.
- The parent must manage all shared state, which can become unwieldy if the state grows complex.
- This pattern does not scale well for deeply nested siblings or app-wide state; use Context or external stores for those cases.
- Passing the same callback to multiple siblings can make it difficult to track which sibling initiated a change.

### Annotated Code Examples

**Example 1: Search Input and Results List (Shared Parent State)**

```jsx
import { useState } from 'react';

// Sibling 1: Search input — reports changes to the parent
function SearchInput({ query, onQueryChange }) {
  return (
    <input
      type="text"
      value={query}
      onChange={(e) => onQueryChange(e.target.value)}
      placeholder="Search..."
    />
  );
}

// Sibling 2: Results list — reads the query from the parent
function ResultsList({ query, items }) {
  const filtered = items.filter(item =>
    item.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <ul>
      {filtered.map((item, i) => (
        <li key={i}>{item}</li>
      ))}
    </ul>
  );
}

// Parent: owns the shared query state
export default function SearchApp() {
  const [query, setQuery] = useState('');
  const allItems = ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry'];

  return (
    <div>
      <h1>Fruit Search</h1>
      <SearchInput query={query} onQueryChange={setQuery} />
      <ResultsList query={query} items={allItems} />
    </div>
  );
}
```

**Expected Output:** A search input and a list of five fruits are displayed. As the user types in the search input, the list filters in real time. For example, typing "an" filters the list to "Banana". Clearing the input restores the full list.

**Why This Output Occurs:** The `query` state lives in the parent (`SearchApp`). The `SearchInput` sibling receives `query` and `onQueryChange` (which is `setQuery`). When the user types, `SearchInput` calls `onQueryChange(newValue)`, updating the parent's state. The parent re-renders, passing the new `query` to both siblings. `ResultsList` reads the updated `query` and filters the items accordingly. Both siblings stay synchronised because they share the same parent state.

**Example 2: Coordinated Counter with Multiple Siblings**

```jsx
import { useState } from 'react';

// Sibling 1: Displays the count
function CountDisplay({ count }) {
  return <h2>Count: {count}</h2>;
}

// Sibling 2: Increments the count
function IncrementButton({ onIncrement }) {
  return <button onClick={onIncrement}>+1</button>;
}

// Sibling 3: Decrements the count
function DecrementButton({ onDecrement }) {
  return <button onClick={onDecrement}>-1</button>;
}

// Parent: owns the shared count state
export default function CounterApp() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <CountDisplay count={count} />
      <IncrementButton onIncrement={() => setCount(c => c + 1)} />
      <DecrementButton onDecrement={() => setCount(c => c - 1)} />
    </div>
  );
}
```

**Expected Output:** A heading displays "Count: 0" and two buttons ("+1" and "-1") are shown. Clicking "+1" increments the count and updates the heading. Clicking "-1" decrements it. Both buttons affect the same shared count.

**Why This Output Occurs:** The `count` state lives in the parent (`CounterApp`). `CountDisplay` reads it, while `IncrementButton` and `DecrementButton` receive wrapper callbacks that update the same state. When either button is clicked, the parent's state updates, and all three siblings re-render with the new count. The siblings are coordinated through the shared parent state.

### Real-World Cases

- **Search interfaces:** A search input sibling reports the query; a results list sibling reads it and filters data.
- **Shopping carts:** A cart summary sibling reads the cart items; product list siblings report add/remove actions.
- **Dashboard widgets:** A date-range picker sibling reports the selected range; chart siblings read it and update their data.
- **Form wizards:** A step indicator sibling reads the current step; navigation siblings report step changes.
- **Tabs:** A tab list sibling reports the active tab; a tab panel sibling reads it and displays the corresponding content.

### References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Managing State: https://react.dev/learn/managing-state
- Tsecurity.de – React Component Communication: Parent-Child and Child-Parent Interactions: https://tsecurity.de/de/2458759/IT+Programmierung/React+Component+Communication:+Parent-Child+and+Child-Parent+Interactions/

---

## Core Concept 3: Context (Bypassing Prop Drilling for Shared Data Scope)

### Definitions

**Core Definition:** React Context is a built-in feature that allows a parent component to make data available to any component in the tree below it—no matter how deep—without passing it explicitly through props.

**Technical Definition:** Context provides a way to pass data through the component tree without having to pass props down manually at every level. It is designed to share data that can be considered "global" for a tree of React components, such as the current authenticated user, theme, or preferred language. Context is created with `React.createContext()`, provided to a subtree via a `<Context.Provider value={...}>` component, and consumed by any descendant via the `useContext(Context)` Hook. When the provider's `value` changes, all consumers of that context re-render, regardless of whether they are direct children or deeply nested descendants. Context is an alternative to prop drilling: instead of passing a prop through every intermediate component, the provider "teleports" the value to the components that need it. However, Context is primarily a *dependency injection* mechanism, not a complete state management solution; for frequently changing state, selector-based external stores are often more performant.

**Beginner-Friendly Explanation:** Imagine a family group chat. Instead of each sibling asking the parent to relay a message, the parent posts the message in the group chat, and all siblings can read it directly. The parent doesn't have to pass the message through each sibling individually—any sibling in the group can see it. That's Context: a shared channel that any component in the tree can tap into, no matter how deep it is.

### Purposes

- To avoid prop drilling—passing props through many intermediate components that do not use them.
- To share data that is "global" to a subtree, such as theme, locale, or authenticated user.
- To provide a single source of truth for shared data across deeply nested components.
- To reduce the boilerplate of manually threading props through the component tree.
- To enable feature-scoped sharing without introducing an external state management library.

### Syntax Rules and Structure

**Step 1: Create the Context**
```jsx
import { createContext } from 'react';

const ThemeContext = createContext('light'); // Default value
```

**Component Breakdown:**
- `createContext(defaultValue)`: Creates a context object with an optional default value.
- The default value is used only when a component reads the context without a matching provider above it.

**Step 2: Provide the Context**
```jsx
function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

**Component Breakdown:**
- `<ThemeContext.Provider value={...}>`: Wraps the subtree that should have access to the context.
- `value={{ theme, setTheme }}`: The data and setter are passed as the context value.
- All descendants of `<Toolbar />` can read this value.

**Step 3: Consume the Context**
```jsx
import { useContext } from 'react';

function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button
      style={{ background: theme === 'dark' ? '#333' : '#fff' }}
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      Toggle Theme
    </button>
  );
}
```

**Component Breakdown:**
- `useContext(ThemeContext)`: Reads the nearest provider's value.
- `{ theme, setTheme }`: Destructures the context value.
- The button reads `theme` and calls `setTheme` to request changes.

**Syntax Rules:**
- Create the context with `createContext()`. The default value is optional but recommended for testing and fallback.
- Wrap the subtree that needs access in `<Context.Provider value={...}>`.
- Consume the context with `useContext(Context)` in any descendant component.
- When the provider's `value` changes, all consumers re-render. To avoid unnecessary re-renders, memoise the value object with `useMemo` or split the context into separate state and dispatch contexts.
- Context is best for data that is truly "global" to a subtree (theme, locale, auth). Avoid using it for state that changes frequently and is only needed by a few components.
- For state + dispatch patterns, use `useReducer` with Context: put the reducer's `state` and `dispatch` into two separate contexts to prevent dispatch-only consumers from re-rendering when state changes.

**Constraints and Limitations:**
- All consumers of a context re-render when the provider's `value` changes, even if they only read a small part of the value.
- Context makes component reuse more difficult because consumers are coupled to the context's shape.
- Context is not a complete state management solution; it does not provide selectors, middleware, or time-travel debugging.
- For frequently changing state (e.g., form inputs, counters), Context can cause performance issues; external stores with selectors are often better.
- Context should be used sparingly; for simple prop drilling, component composition (passing JSX as children) is often a simpler solution.

### Annotated Code Examples

**Example 1: Theme Context for Sibling Components**

```jsx
import { createContext, useContext, useState } from 'react';

// Step 1: Create the context
const ThemeContext = createContext('light');

// Sibling 1: A button that toggles the theme
function ThemeToggleButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}

// Sibling 2: A panel that displays the current theme
function ThemeDisplayPanel() {
  const { theme } = useContext(ThemeContext);

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

// Parent: provides the context to both siblings
export default function ThemeApp() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <h1>Theme Switcher</h1>
      <ThemeToggleButton />
      <ThemeDisplayPanel />
    </ThemeContext.Provider>
  );
}
```

**Expected Output:** A heading "Theme Switcher" is displayed, followed by a button ("Switch to dark mode") and a panel showing "Current theme: light" with a white background. Clicking the button changes the theme to dark, updates the button text to "Switch to light mode", and changes the panel's background to dark with white text.

**Why This Output Occurs:** The `theme` state lives in the parent (`ThemeApp`), which provides it via `ThemeContext.Provider`. Both `ThemeToggleButton` and `ThemeDisplayPanel` are siblings inside the provider. They both consume the context with `useContext(ThemeContext)`. When the button calls `setTheme('dark')`, the provider's value changes, and both consumers re-render with the new theme. The siblings are synchronised through the shared context without any props being passed between them.

**Example 2: Context with useReducer for Complex Shared State**

```jsx
import { createContext, useContext, useReducer } from 'react';

// Step 1: Create the contexts (state and dispatch separated)
const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

// Reducer: consolidates state update logic
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added':
      return [...tasks, { id: action.id, text: action.text, done: false }];
    case 'toggled':
      return tasks.map(t =>
        t.id === action.id ? { ...t, done: !t.done } : t
      );
    case 'deleted':
      return tasks.filter(t => t.id !== action.id);
    default:
      throw new Error('Unknown action: ' + action.type);
  }
}

// Parent: provides state and dispatch via separate contexts
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// Custom hooks for consuming the contexts
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

// Sibling 1: Task list — reads tasks
function TaskList() {
  const tasks = useTasks();

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          {task.text} {task.done ? '✓' : ''}
        </li>
      ))}
    </ul>
  );
}

// Sibling 2: Add task form — dispatches actions
function AddTask() {
  const dispatch = useTasksDispatch();
  let nextId = 3;

  return (
    <button onClick={() => {
      dispatch({ type: 'added', id: nextId++, text: 'New Task' });
    }}>
      Add Task
    </button>
  );
}

// App: uses the provider and renders siblings
export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Task Manager</h1>
      <TaskList />
      <AddTask />
    </TasksProvider>
  );
}
```

**Expected Output:** A heading "Task Manager" is displayed, followed by an empty task list and an "Add Task" button. Clicking "Add Task" adds a new task to the list. The task list and the add form are siblings that communicate through the shared context.

**Why This Output Occurs:** The `tasks` state and `dispatch` function are provided via two separate contexts. `TaskList` reads `tasks` via `useTasks()`, and `AddTask` calls `dispatch` via `useTasksDispatch()`. When `AddTask` dispatches an action, the reducer updates the `tasks` state, the provider's value changes, and `TaskList` re-renders with the new task. Separating state and dispatch contexts prevents `AddTask` (which only needs `dispatch`) from re-rendering when `tasks` changes.

### Real-World Cases

- **Theme switching:** A theme context provides the current theme and a toggle function to all components in the app.
- **User authentication:** An auth context provides the current user and login/logout functions to the entire app.
- **Locale/internationalisation:** A locale context provides the current language and translation function to all components.
- **Feature flags:** A feature-flag context provides enabled/disabled flags to components throughout the app.
- **Shopping cart:** A cart context provides cart items and add/remove functions to product pages and the cart sidebar.

### References

- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – useContext: https://react.dev/reference/react/useContext
- React Official Documentation – Scaling Up with Reducer and Context: https://react.dev/learn/scaling-up-with-reducer-and-context
- React Legacy Documentation – Context: https://legacy.reactjs.org/docs/context.html
- LogRocket – React Context tutorial: Complete guide with practical examples: https://blog.logrocket.com/react-context-tutorial/

---

## Core Concept 4: External State Stores (Synchronising Siblings via Global Stores)

### Definitions

**Core Definition:** External state stores are libraries that hold application state outside of the React component tree, allowing any component—including siblings—to subscribe to and update shared state directly without relying on the parent component as a mediator.

**Technical Definition:** External state stores decouple state from the component hierarchy by maintaining a global store that lives outside React. Components subscribe to slices of this store via hooks (e.g., `useSelector` in Redux, `useStore` in Zustand, `useAtom` in Jotai) and re-render only when their selected slice changes. This eliminates the prop-drilling and re-render issues associated with lifted state and Context. The three dominant paradigms are: (1) **Flux/Centralised (Redux Toolkit)** — a single global store with dispatched actions and pure reducers, offering predictable state transitions and time-travel debugging; (2) **Hook-based/Minimal (Zustand)** — a simple external store consumed via hooks, with no providers, actions, or reducers required; and (3) **Atomic/Bottom-up (Jotai)** — independent state atoms composed together, enabling fine-grained reactivity and derived state. Each library solves sibling communication by making shared state globally accessible while allowing components to subscribe only to the specific data they need.

**Beginner-Friendly Explanation:** Imagine a public bulletin board in a town square. Anyone can pin a message to it, and anyone can read from it. Siblings don't need to go through their parent to communicate—they just read from and write to the bulletin board. That's an external state store: a shared place for data that lives outside the family (component tree). Redux is like a formal government office with strict procedures; Zustand is like a casual community whiteboard; Jotai is like a set of interconnected sticky notes.

### Purposes

- To synchronise sibling components without requiring a common parent to mediate.
- To avoid prop drilling and the re-render performance issues of Context.
- To provide a single source of truth for app-wide state.
- To enable fine-grained subscriptions so components only re-render when their specific data changes.
- To support complex state management patterns such as middleware, devtools, persistence, and time-travel debugging.

### Syntax Rules and Structure

**Pattern 1: Zustand (Minimal Hook-Based Store)**

```javascript
import { create } from 'zustand';

// Create the store (lives outside React)
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// Sibling 1: reads and writes
function SiblingA() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);

  return <button onClick={increment}>Count: {count}</button>;
}

// Sibling 2: reads the same state
function SiblingB() {
  const count = useStore((state) => state.count);
  return <p>Sibling B sees: {count}</p>;
}

// Parent: renders both siblings — no props needed
function App() {
  return (
    <>
      <SiblingA />
      <SiblingB />
    </>
  );
}
```

**Component Breakdown:**
- `create((set) => ({ ... }))`: Creates a Zustand store with state and actions.
- `useStore((state) => state.count)`: Selects the `count` slice; only re-renders when `count` changes.
- `useStore((state) => state.increment)`: Selects the `increment` action.
- No providers are needed; the store is a hook that can be used anywhere.

**Pattern 2: Redux Toolkit (Centralised Store with Slices)**

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// Create a slice (reducer + actions)
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
  },
});

const { increment, decrement } = counterSlice.actions;
const store = configureStore({ reducer: { counter: counterSlice.reducer } });

// Sibling 1: dispatches actions
function SiblingA() {
  const dispatch = useDispatch();
  return (
    <button onClick={() => dispatch(increment())}>Increment</button>
  );
}

// Sibling 2: reads state
function SiblingB() {
  const count = useSelector((state) => state.counter.value);
  return <p>Count: {count}</p>;
}

// App: provides the store
function App() {
  return (
    <Provider store={store}>
      <SiblingA />
      <SiblingB />
    </Provider>
  );
}
```

**Component Breakdown:**
- `configureStore`: Creates the Redux store.
- `<Provider store={store}>`: Makes the store available to all components.
- `useDispatch()`: Returns the dispatch function for actions.
- `useSelector((state) => state.counter.value)`: Selects the `value` slice.

**Pattern 3: Jotai (Atomic State)**

```javascript
import { atom, useAtom } from 'jotai';

// Create atoms (independent state pieces)
const countAtom = atom(0);

// Sibling 1: reads and writes the atom
function SiblingA() {
  const [count, setCount] = useAtom(countAtom);
  return (
    <button onClick={() => setCount(count + 1)}>Count: {count}</button>
  );
}

// Sibling 2: reads the same atom
function SiblingB() {
  const [count] = useAtom(countAtom);
  return <p>Sibling B sees: {count}</p>;
}

// App: no provider needed for basic usage
function App() {
  return (
    <>
      <SiblingA />
      <SiblingB />
    </>
  );
}
```

**Component Breakdown:**
- `atom(0)`: Creates an atom with an initial value.
- `useAtom(countAtom)`: Returns the current value and a setter.
- Atoms are globally accessible; no provider is needed for basic usage.

**Syntax Rules:**
- **Zustand:** Create a store with `create()`. Use selector functions to subscribe to specific slices. Use `useShallow` for selecting multiple values. No provider is needed.
- **Redux Toolkit:** Create slices with `createSlice()`. Configure the store with `configureStore()`. Wrap the app in `<Provider store={store}>`. Use `useSelector` to read and `useDispatch` to write.
- **Jotai:** Create atoms with `atom()`. Use `useAtom` to read and write. Atoms can derive from other atoms for computed state.
- Choose the library based on complexity: Zustand for simple to medium apps, Redux Toolkit for large apps with complex state, Jotai for fine-grained atomic state.
- External stores are best for *client* state (UI flags, user preferences, cart contents). For *server* state (API data), use TanStack Query or SWR.

**Constraints and Limitations:**
- External stores add dependencies and bundle size. Redux is the heaviest (6,200 kB), while Zustand and Jotai are lightweight.
- Redux requires significant boilerplate (slices, actions, reducers) compared to Zustand and Jotai.
- Zustand and Jotai do not provide the strict predictability and time-travel debugging of Redux out of the box.
- External stores are overkill for simple sibling communication that can be handled with lifted state or Context.
- Server-side rendering (SSR) requires careful handling of store initialisation and hydration.

### Annotated Code Examples

**Example 1: Zustand Store for Sibling Communication**

```javascript
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

// Sibling 1: Add todo form
function AddTodo() {
    const addTodo = useTodoStore((state) => state.addTodo);
    const [text, setText] = useState('');
  
    function handleSubmit(e) {
        e.preventDefault();
        if (text.trim()) {
            addTodo(text.trim());
            setText('');
        }
    }
  
    return (
        <form onSubmit={handleSubmit}>
            <input value={text} onChange={(e) => setText(e.target.value)} />
            <button type="submit">Add</button>
        </form>
    );
}

// Sibling 2: Todo list
function TodoList() {
    const todos = useTodoStore((state) => state.todos);
    const toggleTodo = useTodoStore((state) => state.toggleTodo);
  
    return (
        <ul>
            {todos.map(todo => (
                <li
                    key={todo.id}
                    onClick={() => toggleTodo(todo.id)}
                    style={{ textDecoration: todo.done ? 'line-through' : 'none' }}
                >
                    {todo.text}
                </li>
            ))}
        </ul>
    );
}

// Parent: renders both siblings — no props passed
export default function TodoApp() {
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

**Why This Output Occurs:** The `useTodoStore` Zustand store holds all todo state outside React. `AddTodo` selects the `addTodo` action and calls it on form submission. `TodoList` selects the `todos` array and the `toggleTodo` action. When `addTodo` updates the store, `TodoList` re-renders with the new todo. The siblings communicate directly through the store—no parent mediation or props are needed.

**Example 2: Redux Toolkit for Coordinated Siblings**

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// Create a filter slice
const filterSlice = createSlice({
  name: 'filter',
  initialState: { category: 'all', search: '' },
  reducers: {
    setCategory: (state, action) => { state.category = action.payload; },
    setSearch: (state, action) => { state.search = action.payload; },
  },
});

const { setCategory, setSearch } = filterSlice.actions;
const store = configureStore({ reducer: { filter: filterSlice.reducer } });

// Sibling 1: Filter controls
function FilterControls() {
  const dispatch = useDispatch();
  const { category, search } = useSelector((state) => state.filter);

  return (
    <div>
      <select
        value={category}
        onChange={(e) => dispatch(setCategory(e.target.value))}
      >
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      <input
        value={search}
        onChange={(e) => dispatch(setSearch(e.target.value))}
        placeholder="Search..."
      />
    </div>
  );
}

// Sibling 2: Filtered results
function FilteredResults() {
  const { category, search } = useSelector((state) => state.filter);
  const allItems = [
    { name: 'Laptop', category: 'electronics' },
    { name: 'T-Shirt', category: 'clothing' },
    { name: 'Phone', category: 'electronics' },
    { name: 'Jeans', category: 'clothing' },
  ];

  const filtered = allItems.filter(item =>
    (category === 'all' || item.category === category) &&
    item.name.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <ul>
      {filtered.map((item, i) => (
        <li key={i}>{item.name} — {item.category}</li>
      ))}
    </ul>
  );
}

// App: provides the store
export default function ShopApp() {
  return (
    <Provider store={store}>
      <h1>Product Filter</h1>
      <FilterControls />
      <FilteredResults />
    </Provider>
  );
}
```

**Expected Output:** A dropdown ("All", "Electronics", "Clothing") and a search input are displayed, followed by a list of four products. Selecting "Electronics" filters the list to Laptop and Phone. Typing "lap" in the search box further filters to just "Laptop".

**Why This Output Occurs:** The `filter` slice holds `category` and `search` state in the Redux store. `FilterControls` dispatches actions to update these values. `FilteredResults` reads them via `useSelector` and filters the items. Both siblings subscribe to the same slice and re-render when it changes. The store acts as the single source of truth, and the siblings are synchronised without any parent mediation.

### Real-World Cases

- **E-commerce carts:** The cart state is stored in a global store; product pages add items, and the cart sidebar reads them.
- **User authentication:** Auth state (user, token, login status) is stored globally; navbar and protected routes read it.
- **Notification systems:** Notification state is stored globally; a notification bell and a notification panel both read it.
- **Dashboard filters:** Filter state is stored globally; multiple chart widgets read the same filters and update together.
- **Multi-step forms:** Form data is stored in a global store; each step reads and writes its portion, and a summary step reads all of it.

### References

- Zustand Documentation – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- Zustand GitHub – Bear necessities for state management in React: https://github.com/pmndrs/zustand
- Redux Toolkit – Getting Started: https://redux-toolkit.js.org/introduction/getting-started
- Redux Toolkit – Quick Start: https://redux.js.org/tutorials/quick-start
- Jotai – Primitive and flexible state management for React: https://jotai.org/
- DEV Community – Redux Toolkit vs Zustand vs Jotai: https://dev.to/zeeshanali0704/frontend-system-design-redux-toolkit-vs-zustand-vs-jotai-1npn
- IEEE Xplore – Evaluation of State Management Libraries: https://ieeexplore.ieee.org/document/10293011
- RTCamp – Choosing the Right React State Management Strategy: https://rtcamp.com/tutorials/react-state-management/

---

## Comparison and Decision Guidance

| Pattern | Best For | Data Flow | Re-render Granularity | Provider Required | Boilerplate |
|---|---|---|---|---|---|
| **Lifted State** | 2–3 siblings with simple shared state | Explicit: parent → child | All siblings re-render | No | Low |
| **Shared Parent State** | Multiple siblings reading the same data | Explicit: parent → child | All siblings re-render | No | Low |
| **Context** | Feature-scoped sharing (theme, auth, locale) | Implicit: provider → consumers | All consumers re-render | Yes | Medium |
| **Redux Toolkit** | Large apps with complex, predictable state | Centralised: dispatch → reducer → store | Selector-based (fine-grained) | Yes | High |
| **Zustand** | Medium apps needing simple global state | Hook-based: store → hook | Selector-based (fine-grained) | No | Low |
| **Jotai** | Apps with fine-grained atomic state | Atomic: atom → useAtom | Atom-based (very fine-grained) | No (optional) | Low |

**Decision Guidance:**
- **Start with lifted state.** For most sibling communication, lifting state to the closest common parent is sufficient, explicit, and requires no dependencies.
- **Use Context when prop drilling becomes painful.** When the closest common ancestor is far removed from the siblings that need the data, Context eliminates the drilling.
- **Use an external store when state grows complex.** When multiple features need to read and write the same state, or when you need devtools, middleware, or persistence, choose an external store.
- **Choose Zustand for simplicity.** It has the smallest API surface, no providers, and selector-based subscriptions. It is the recommended default for new client stores.
- **Choose Redux Toolkit for large teams and complex state.** Its strict unidirectional flow, middleware, and time-travel debugging make it suitable for enterprise applications.
- **Choose Jotai for atomic, derived state.** When many tiny pieces of state derive from each other (graph-shaped state), atoms provide the most natural model.

---

## References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – Scaling Up with Reducer and Context: https://react.dev/learn/scaling-up-with-reducer-and-context
- React Official Documentation – useContext: https://react.dev/reference/react/useContext
- React Legacy Documentation – Lifting State Up: https://legacy.reactjs.org/docs/lifting-state-up.html
- React Legacy Documentation – Context: https://legacy.reactjs.org/docs/context.html
- Zustand Documentation – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- Zustand GitHub: https://github.com/pmndrs/zustand
- Redux Toolkit – Getting Started: https://redux-toolkit.js.org/introduction/getting-started
- Redux – Quick Start: https://redux.js.org/tutorials/quick-start
- Jotai – Primitive and flexible state management for React: https://jotai.org/
- DEV Community – Redux Toolkit vs Zustand vs Jotai: https://dev.to/zeeshanali0704/frontend-system-design-redux-toolkit-vs-zustand-vs-jotai-1npn
- IEEE Xplore – Evaluation of State Management Libraries: https://ieeexplore.ieee.org/document/10293011
- RTCamp – Choosing the Right React State Management Strategy: https://rtcamp.com/tutorials/react-state-management/
- CoreUI – How to Lift State Up in React: https://coreui.io/answers/how-to-lift-state-up-in-react/
- Tsecurity.de – React Component Communication: Parent-Child and Child-Parent Interactions: https://tsecurity.de/de/2458759/IT+Programmierung/React+Component+Communication:+Parent-Child+and+Child-Parent+Interactions/
- LogRocket – React Context tutorial: Complete guide with practical examples: https://blog.logrocket.com/react-context-tutorial/