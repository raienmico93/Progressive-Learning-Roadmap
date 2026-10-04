# React Design & Composition Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Design & Composition Patterns are the architectural techniques—compound components, custom hooks, controlled/uncontrolled components, render props, higher-order components, slot composition, and provider patterns—that enable developers to build flexible, reusable, and maintainable React component APIs.

**Technical Definition:** Composition patterns in React are structural techniques that leverage React's compositional nature to share state, behaviour, and rendering logic across components. These patterns solve recurring architectural problems: the **Compound Components** pattern manages implicit state sharing between parent and child components (e.g., `<Select>` and `<Option>`) through React Context; the **Custom Hooks** pattern extracts stateful logic and side effects from presentation layers into reusable functions; **Controlled vs. Uncontrolled Components** determine whether form state is managed by React or the DOM; **Render Props and Higher-Order Components** (HOCs) are legacy techniques for sharing cross-cutting concerns, now largely superseded by custom Hooks; **Slot Composition** passes pre-rendered elements as props to avoid prop drilling and prop explosion; and the **Provider Pattern** establishes dependency injection layers for global application contexts.

**Beginner-Friendly Explanation:** When you build React apps, you quickly run into the same problems: "How do I share state between related components?" "How do I reuse logic across components?" "How do I make a component flexible without adding 20 props?" React design patterns are the answers to these questions. They're proven techniques that make your components easier to use, easier to test, and easier to change. Some are old (like HOCs and render props), some are newer (like custom hooks and slot composition), and some are fundamental (like controlled components and providers). Knowing when to use each one is what separates a good React developer from a great one.

### Key Characteristics

- **Composition Over Inheritance:** All these patterns favour composing components together rather than extending class hierarchies.
- **Implicit State Sharing:** Compound components and providers share state implicitly through React Context, avoiding prop drilling.
- **Logic Extraction:** Custom hooks, render props, and HOCs extract reusable logic from components, but custom hooks are the modern preferred approach.
- **Flexibility vs. Simplicity:** Slot composition and compound components trade some simplicity for maximum flexibility, allowing consumers to compose UI exactly as needed.
- **Declarative APIs:** These patterns produce declarative component APIs that mirror the structure of the UI, making code easier to read and reason about.
- **Legacy vs. Modern:** Render props and HOCs are legacy patterns largely replaced by custom hooks, but understanding them is important for maintaining older codebases.

### Prerequisites

- Solid understanding of React components, props, state, and JSX.
- Familiarity with React Hooks (`useState`, `useEffect`, `useContext`).
- Basic understanding of React Context API.
- Experience building forms and handling user input in React.
- Awareness of JavaScript closures and higher-order functions.

### Related Programming Areas

- **React Context API:** The mechanism underlying compound components and provider patterns.
- **Custom Hooks:** The modern approach to logic reuse, replacing render props and HOCs.
- **Form Management:** Controlled vs. uncontrolled components are foundational to form handling.
- **State Management:** Provider patterns enable dependency injection for global state.
- **Component Libraries:** Design systems like Radix UI, Reach UI, and shadcn/ui are built on these patterns.

### Core Concepts / Features

1. Compound Components Pattern
2. Custom Hooks Pattern
3. Controlled vs. Uncontrolled Components
4. Render Props & Higher-Order Components (HOC)
5. Slot & Composition Pattern
6. Provider Pattern

---

## Core Concept 1: Compound Components Pattern

### Definitions

**Core Definition:** The Compound Components pattern is a technique where a parent component manages shared state and behaviour, and child components consume that state implicitly through React Context, working together to form a complete UI.

**Technical Definition:** Compound components are a set of components that work together to form a cohesive unit, implicitly sharing state and behaviour. The parent component (e.g., `<CustomSelect>`) manages the shared state and passes it down to its children using React's Context API. The child components (e.g., `<CustomOption>`) consume that context and interact with the parent as needed. This pattern mirrors the behaviour of native HTML elements like `<select>` and `<option>`, where the `<select>` manages the state and the `<option>`s are configuration. The pattern eliminates prop explosion and provides a flexible, declarative API where consumers compose the components exactly as they need.

**Beginner-Friendly Explanation:** Think of a native HTML `<select>` and `<option>`. By itself, `<select>` is useless. By itself, `<option>` is useless. But together, they're very useful—and they share state implicitly. The compound components pattern lets you build your own components that work the same way. A `<Tabs>` component manages which tab is active, and `<Tab>` and `<TabPanel>` components consume that information automatically. You don't have to pass the active tab as a prop to every child—they just know.

### Purposes

- To avoid prop explosion and keep component APIs clean and simple.
- To give users maximum flexibility in composing the UI exactly as they need.
- To share state implicitly between parent and child components without prop drilling.
- To create declarative APIs where the structure of the code matches the structure of the UI.
- To build extensible components where new features can be added without breaking existing APIs.
- To mimic the behaviour of native HTML elements that work together as a family.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import React, { createContext, useContext, useState } from 'react';

// 1. Create context for shared state
const ToggleContext = createContext(null);

// 2. Parent component manages state
function Toggle({ children }) {
  const [on, setOn] = useState(false);
  const toggle = () => setOn((prev) => !prev);

  return (
    <ToggleContext.Provider value={{ on, toggle }}>
      {children}
    </ToggleContext.Provider>
  );
}

// 3. Child components consume context
function ToggleOn({ children }) {
  const { on } = useContext(ToggleContext);
  return on ? children : null;
}

function ToggleOff({ children }) {
  const { on } = useContext(ToggleContext);
  return on ? null : children;
}

function ToggleButton() {
  const { on, toggle } = useContext(ToggleContext);
  return <button onClick={toggle}>{on ? 'ON' : 'OFF'}</button>;
}

// 4. Attach children as properties of parent
Toggle.On = ToggleOn;
Toggle.Off = ToggleOff;
Toggle.Button = ToggleButton;

// Usage
function App() {
  return (
    <Toggle>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <Toggle.Button />
    </Toggle>
  );
}
```

**Component Breakdown:**
- `ToggleContext`: The context that holds the shared state (`on`, `toggle`).
- `Toggle`: The parent component that manages state and provides it via context.
- `ToggleOn` / `ToggleOff`: Child components that conditionally render based on the shared state.
- `Toggle.Button`: A child component that calls the `toggle` function from context.
- `Toggle.On = ToggleOn`: Attaches the child components as properties of the parent, creating the compound component API.

**Syntax Rules:**
- The parent component must provide the shared state via a Context Provider.
- Child components must consume the context using `useContext`.
- Child components are attached to the parent as properties (`Parent.Child`) for a clean API.
- The parent should validate that child components are used within the parent (e.g., throw if context is `null`).
- Compound components should not be rendered outside their parent; the context will be `null`.

**Constraints and Limitations:**
- Compound components require more boilerplate than simple prop-based APIs.
- They can be harder to type in TypeScript (though libraries like `@bind-ts/react-bind` provide type-safe solutions).
- Overuse can lead to overly complex component APIs for simple use cases.
- The implicit state sharing can be confusing for developers unfamiliar with the pattern.

### Annotated Code Examples

**Example 1: Complete Compound Tabs Component**


**TabsContext.js**
This file holds the React context used to share the active tab state between the components.
```jsx
import { createContext } from 'react';
const TabsContext = createContext(null);
export default TabsContext;
```

**TabList.js**
The container wrapper for the individual tab buttons.
```jsx
import React from 'react';
function TabList({ children }) {
  return (
    <div className="tab-list" role="tablist">
      {children}
    </div>
  );
}
export default TabList;
```

**Tab.js**
The individual interactive button that selects a specific tab.
```jsx
import React, { useContext } from 'react';
import TabsContext from './TabsContext';

function Tab({ index, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  const isActive = activeTab === index;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      onClick={() => setActiveTab(index)}
      className={isActive ? 'tab active' : 'tab'}
    >
      {children}
    </button>
  );
}
export default Tab;
```

**TabPanel.js**
The wrapper that conditionally displays content based on the currently selected tab.
```jsx
import React, { useContext } from 'react';
import TabsContext from './TabsContext';

function TabPanel({ index, children }) {
  const { activeTab } = useContext(TabsContext);

  if (activeTab !== index) return null;

  return (
    <div role="tabpanel" className="tab-panel">
      {children}
    </div>
  );
}
export default TabPanel;
```

**Tabs.js**
This is the main parent component that manages the state and provides it to all the child sub-components attached to it.
```jsx
import React, { useState } from 'react';
import TabsContext from './TabsContext';
import TabList from './TabList';
import Tab from './Tab';
import TabPanel from './TabPanel';

function Tabs({ children, defaultTab = 0 }) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}
// Attach sub-components for the compound component pattern
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

export default Tabs;
```

**App.js**
Your main application entry point showcasing how to import and use the separated component.
```jsx
import React from 'react';
import Tabs from './Tabs';

function App() {
  return (
    <Tabs defaultTab={0}>
      <Tabs.List>
        <Tabs.Tab index={0}>Profile</Tabs.Tab>
        <Tabs.Tab index={1}>Settings</Tabs.Tab>
        <Tabs.Tab index={2}>Notifications</Tabs.Tab>
      </Tabs.List>
      
      <Tabs.Panel index={0}>Profile content here</Tabs.Panel>
      <Tabs.Panel index={1}>Settings content here</Tabs.Panel>
      <Tabs.Panel index={2}>Notifications content here</Tabs.Panel>
    </Tabs>
  );
}
export default App;
```

**Expected Output:** The `App` component renders a tabbed interface with three tabs ("Profile", "Settings", "Notifications"). Clicking a tab updates the active tab and displays the corresponding panel content. All child components share state implicitly through the `TabsContext`.

**Why This Output Occurs:** The `Tabs` parent component manages the `activeTab` state and provides it through context. The `Tab` components consume the context to determine if they are active and to set the active tab when clicked. The `TabPanel` components consume the context to determine whether to render their content. The children are attached to the parent (`Tabs.List`, `Tabs.Tab`, `Tabs.Panel`) creating a clean compound component API.

### Real-World Cases

- **Design systems:** Tabs, accordions, dropdowns, and menus are commonly built as compound components (e.g., Radix UI Primitives).
- **Form libraries:** Form field components (Form, Field, Label, Error) can be compound components that share validation state.
- **E-commerce:** Product gallery components (Gallery, Thumbnails, MainImage) share the selected image state.
- **SaaS dashboards:** Filter panels, date range pickers, and data table controls.
- **Navigation:** Breadcrumb components (Breadcrumb, BreadcrumbItem, BreadcrumbSeparator).

---

## Core Concept 2: Custom Hooks Pattern

### Definitions

**Core Definition:** A custom Hook is a JavaScript function whose name starts with `use` and that can call other Hooks, allowing stateful logic and side effects to be extracted from components and reused across multiple components.

**Technical Definition:** Custom Hooks are JavaScript functions that start with the prefix `use` and can call other Hooks. This naming convention signals to developers and React's linting rules that the function follows the rules of Hooks. Custom Hooks enable sharing stateful logic between components without requiring complex patterns like higher-order components or render props. They can encapsulate state management (`useToggle`, `useCounter`), side effects (`useFetch`, `useLocalStorage`), reducer logic (`useReducer`-based hooks), and composition of other hooks.

**Beginner-Friendly Explanation:** A custom Hook is like a reusable recipe for logic. If you find yourself writing the same `useState` and `useEffect` code in multiple components, you can extract it into a custom Hook. Then any component can use that Hook and get all the logic for free. The `use` prefix is important—it tells React that this function follows the rules of Hooks, and it tells other developers that this function contains React magic.

### Purposes

- To extract stateful logic from components into reusable functions.
- To reduce code duplication across components.
- To separate concerns: presentation in components, logic in hooks.
- To simplify complex components by moving logic into smaller, focused hooks.
- To enable testing of logic independently from UI components.
- To compose multiple hooks together for more complex behaviour.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useState, useCallback } from 'react';

// Custom hook: must start with "use"
function useToggle(initialState = false) {
  const [state, setState] = useState(initialState);

  const toggle = useCallback(() => {
    setState((prev) => !prev);
  }, []);

  const setOn = useCallback(() => setState(true), []);
  const setOff = useCallback(() => setState(false), []);

  return { state, toggle, setOn, setOff };
}

// Usage in a component
function Accordion() {
  const { state: isOpen, toggle } = useToggle(false);

  return (
    <div>
      <button onClick={toggle}>
        {isOpen ? 'Hide Content' : 'Show Content'}
      </button>
      {isOpen && <div className="content">Toggleable content</div>}
    </div>
  );
}
```

**Component Breakdown:**
- `useToggle`: A custom Hook that encapsulates toggle state logic.
- `useState`: The built-in Hook used inside the custom Hook.
- `useCallback`: Memoises the toggle function to prevent unnecessary re-renders.
- The Hook returns an object with the state and functions.
- The component uses the Hook like any other Hook.

**Syntax Rules:**
- Custom Hooks must start with the word `use`.
- Custom Hooks can call other Hooks (built-in or custom).
- Hooks must be called at the top level of a custom Hook, never inside conditions, loops, or nested functions.
- Custom Hooks should have narrow, clear responsibilities; avoid "god hooks" that handle multiple concerns.
- Return values should be stable (use `useMemo`/`useCallback` when returning objects or functions).
- Custom Hooks should be tested independently using `renderHook` from React Testing Library.

**Constraints and Limitations:**
- Custom Hooks cannot be called conditionally; the rules of Hooks apply.
- Over-extracting logic into hooks can lead to premature abstraction and harder-to-understand code.
- Custom Hooks do not share state between components; each call to a Hook gets its own state.
- Debugging custom Hooks can be harder than debugging components because the React DevTools may not show them clearly.

### Annotated Code Examples

**Example 1: `useFetch` Custom Hook**

**useFetch.js**
This file contains the reusable data-fetching hook logic.

```jsx
import { useState, useEffect } from 'react';

export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, {
          signal: abortController.signal,
        });

        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
          setData(null);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();

    return () => abortController.abort();
  }, [url]);

  return { data, loading, error };
}
```

**UserProfile.js**
This file contains the UI component and imports the hook to display user profile data.

```jsx
import React from 'react';
import { useFetch } from './useFetch'; // Adjust the import path as needed

export function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(
    `https://api.example.com/users/${userId}`
  );

  if (loading) return <p>Loading...</p>;
  if (error) return <p role="alert">Error: {error}</p>;
  if (!data) return null;

  return <h2>{data.name}</h2>;
}
```

**Expected Output:** The `UserProfile` component displays "Loading..." initially, then the user's name when the data arrives, or an error message if the fetch fails.

**Why This Output Occurs:** The `useFetch` Hook encapsulates all the logic for fetching data, managing loading state, handling errors, and aborting the request on cleanup. The component only needs to call the Hook and render the appropriate UI based on the returned state. This separates the data-fetching concern from the presentation concern.

### Real-World Cases

- **Data fetching:** `useFetch`, `useQuery`, `useMutation` for API interactions.
- **Local storage:** `useLocalStorage` for persisting state to localStorage.
- **Form handling:** `useForm` for managing form state and validation.
- **Media queries:** `useMediaQuery` for responsive design.
- **Debouncing:** `useDebounce` for delaying expensive operations.
- **Event listeners:** `useEventListener` for adding and cleaning up DOM event listeners.

---

## Core Concept 3: Controlled vs. Uncontrolled Components

### Definitions

**Core Definition:** Controlled components have their form data managed by React state, while uncontrolled components have their form data managed by the DOM itself.

**Technical Definition:** In a controlled component, form data is handled by a React component, with the component's state as the single source of truth. The input's `value` prop is set from React state, and the `onChange` handler updates that state. In an uncontrolled component, form data is handled by the DOM, and the component uses a `ref` to access the current value when needed. Controlled components provide more control over form data and make it easier to implement complex validation and business logic, but require more code. Uncontrolled components are simpler to implement and can be faster for large forms, but offer less control.

**Beginner-Friendly Explanation:** With a controlled component, React is the boss—it decides what the input's value is at all times. You type a letter, React updates the state, and the input displays the new state. With an uncontrolled component, the DOM is the boss—the input keeps its own value, and you ask for it when you need it (like on form submission). Controlled is more powerful but requires more code; uncontrolled is simpler but less flexible.

### Purposes

- To manage form state with React's state management (controlled).
- To access form values on demand without re-rendering on every keystroke (uncontrolled).
- To implement real-time validation and conditional form logic (controlled).
- To integrate with non-React code more easily (uncontrolled).
- To handle large forms with better performance (uncontrolled).
- To provide a single source of truth for form data (controlled).

### Syntax Rules and Structure

**General Syntax for Controlled Component:**

```jsx
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

**Component Breakdown:**
- `useState('')`: The single source of truth for the input's value.
- `value={value}`: The input's value is controlled by React state.
- `onChange={(e) => setValue(e.target.value)}`: Updates the state on every keystroke.

**General Syntax for Uncontrolled Component:**

```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);

  function handleSubmit() {
    console.log(inputRef.current.value);
  }

  return <input type="text" ref={inputRef} defaultValue="initial" />;
}
```

**Component Breakdown:**
- `useRef(null)`: A ref to access the DOM node directly.
- `defaultValue="initial"`: The initial value (not controlled after mount).
- `inputRef.current.value`: Access the current value when needed (e.g., on submit).

**Syntax Rules:**
- Controlled components must have both `value` and `onChange` props.
- Uncontrolled components use `ref` and `defaultValue` (not `value`).
- A component can be controlled if it has an `on` prop (e.g., `onChange`), as seen in the Controlled Props pattern.
- Prefer controlled components for forms that need validation, conditional logic, or dynamic behaviour.
- Use uncontrolled components for simple forms, file inputs, or when integrating with non-React code.

**Constraints and Limitations:**
- Controlled components re-render on every keystroke, which can be slow for large forms.
- Uncontrolled components do not re-render on input change, making real-time validation harder.
- `defaultValue` is only used on the initial render; subsequent changes are not reflected.
- The `ref` API requires direct DOM access, which is generally discouraged in React.
- Controlled components require more boilerplate code than uncontrolled components.

### Annotated Code Examples

**Example 1: Controlled Form with Real-Time Validation**

```jsx
import React, { useState } from 'react';

function RegistrationForm() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  function handleEmailChange(e) {
    const value = e.target.value;
    setEmail(value);

    // Real-time validation
    if (value && !value.includes('@')) {
      setError('Please enter a valid email address');
    } else {
      setError('');
    }
  }

  function handleSubmit(e) {
    e.preventDefault();
    if (!email.includes('@')) {
      setError('Please enter a valid email address');
      return;
    }
    console.log('Submitting:', email);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={handleEmailChange}
      />
      {error && <p role="alert">{error}</p>}
      <button type="submit">Register</button>
    </form>
  );
}
```

**Expected Output:** As the user types, the email is validated in real-time. If the email is invalid, an error message appears immediately. On submit, the form validates again before submitting.

**Why This Output Occurs:** The `email` state is the single source of truth. Every keystroke updates the state via `onChange`, and the validation logic runs on each change. The error message is displayed conditionally based on the validation result. This real-time feedback is only possible because the component is controlled.

**Example 2: Uncontrolled Form for Simple Submission**

```jsx
import React, { useRef } from 'react';

function SimpleForm() {
  const nameRef = useRef(null);
  const emailRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    const name = nameRef.current.value;
    const email = emailRef.current.value;
    console.log('Name:', name, 'Email:', email);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="name">Name</label>
      <input id="name" type="text" ref={nameRef} defaultValue="" />

      <label htmlFor="email">Email</label>
      <input id="email" type="email" ref={emailRef} defaultValue="" />

      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** The form does not re-render as the user types. On submit, the current values are read from the DOM refs and logged.

**Why This Output Occurs:** The inputs are uncontrolled; their values are managed by the DOM. The `ref` objects provide direct access to the DOM nodes, allowing the values to be read on demand. No React state is involved, so no re-renders occur during typing.

### Real-World Cases

- **Complex forms:** Controlled components for multi-step forms with validation and conditional fields.
- **Simple forms:** Uncontrolled components for basic contact forms or search inputs.
- **File inputs:** Uncontrolled components because `<input type="file" />` is inherently uncontrolled.
- **Large forms:** Uncontrolled components for better performance with many fields.
- **Third-party integration:** Uncontrolled components when integrating with non-React libraries.

---

## Core Concept 4: Render Props & Higher-Order Components (HOC)

### Definitions

**Core Definition:** Render props and Higher-Order Components (HOCs) are legacy React patterns for sharing code between components, largely superseded by custom Hooks in modern React.

**Technical Definition:** A **render prop** is a technique for sharing code between React components using a prop whose value is a function that returns a React element. A component with a render prop takes a function and calls it instead of implementing its own render logic. A **Higher-Order Component (HOC)** is a function that takes a component and returns a new component, adding behaviour or props to the wrapped component. Both patterns were the primary means of sharing cross-cutting concerns (authentication, data fetching, theming) before Hooks were introduced in React 16.8. Today, custom Hooks are the preferred approach for logic reuse, and render props are only used when the consumer needs full control over rendering.

**Beginner-Friendly Explanation:** Render props and HOCs are the "old way" of sharing logic in React. Think of a HOC like a gift wrapper: you give it a component, and it gives you back a new component with extra features. A render prop is like giving someone a recipe: you tell them what to render by passing a function. These patterns still work, but they're not used much anymore because custom Hooks are simpler and more flexible. You'll still see them in older codebases and some libraries.

### Purposes

- To share cross-cutting concerns (authentication, data fetching, theming) across components (legacy).
- To provide reusable logic without repeating code (legacy).
- To give consumers full control over rendering when needed (render props).
- To maintain older codebases that use these patterns.
- To understand the evolution of React patterns and why Hooks were introduced.
- To recognise these patterns when encountered in libraries and legacy code.

### Syntax Rules and Structure

**General Syntax for Render Props:**

```jsx
function DataProvider({ render }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data')
      .then((res) => res.json())
      .then(setData);
  }, []);

  return render(data);
}

// Usage
function App() {
  return (
    <DataProvider
      render={(data) => (
        <div>{data ? data.name : 'Loading...'}</div>
      )}
    />
  );
}
```

**Component Breakdown:**
- `DataProvider`: A component that accepts a `render` prop.
- `render(data)`: Calls the render prop with the current data.
- The consumer provides a function that returns JSX.

**General Syntax for HOC:**

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { user } = useAuth();

    if (!user) {
      return <Navigate to="/login" />;
    }

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
```

**Component Breakdown:**
- `withAuth(WrappedComponent)`: A function that takes a component and returns a new one.
- `AuthenticatedComponent`: The new component that checks authentication and renders the wrapped component with the `user` prop.
- `ProtectedDashboard`: The enhanced component that can be used like a regular component.

**Syntax Rules:**
- Render props: the prop can be named anything (`render`, `children`, or a custom name).
- HOCs: the returned component must forward props (`{...props}`) to the wrapped component.
- HOCs: set a `displayName` for debugging.
- HOCs: do not mutate the original component; return a new one.
- Prefer custom Hooks over render props and HOCs for logic reuse.
- Use render props only when the consumer needs full control over rendering.

**Constraints and Limitations:**
- HOCs can lead to "wrapper hell" with deeply nested component trees.
- HOCs can cause prop naming collisions.
- Render props can lead to deeply nested JSX ("callback hell").
- Both patterns are harder to type in TypeScript than custom Hooks.
- Neither pattern shares state between component instances; each instance has its own state.
- Custom Hooks solve the same problems with less boilerplate and better composition.

### Annotated Code Examples

**Example 1: Render Prop for Mouse Tracking**

```jsx
import React, { useState, useEffect } from 'react';

function Mouse({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMouseMove(event) {
      setPosition({ x: event.clientX, y: event.clientY });
    }
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

// Usage
function App() {
  return (
    <Mouse
      render={({ x, y }) => (
        <p>
          Mouse position: ({x}, {y})
        </p>
      )}
    />
  );
}
```

**Expected Output:** The component displays the current mouse position, updating as the mouse moves.

**Why This Output Occurs:** The `Mouse` component encapsulates the logic for tracking mouse position and shares it via the `render` prop. The consumer provides a function that renders the position. This is the classic render props example from the React documentation.

**Example 2: HOC for Authentication**

```jsx
import React from 'react';
import { Navigate } from 'react-router-dom';

function withAuth(WrappedComponent) {
  function AuthenticatedComponent(props) {
    const { user, isLoading } = useAuth();

    if (isLoading) return <p>Loading...</p>;
    if (!user) return <Navigate to="/login" replace />;

    return <WrappedComponent {...props} user={user} />;
  }

  AuthenticatedComponent.displayName = `withAuth(${WrappedComponent.displayName || WrappedComponent.name})`;

  return AuthenticatedComponent;
}

// Usage
function Dashboard({ user }) {
  return <h1>Welcome, {user.name}</h1>;
}

const ProtectedDashboard = withAuth(Dashboard);
```

**Expected Output:** If the user is authenticated, the dashboard renders with the user's name. If not, the user is redirected to the login page.

**Why This Output Occurs:** The `withAuth` HOC wraps the `Dashboard` component with authentication logic. The `AuthenticatedComponent` checks the auth state and either renders the wrapped component with the `user` prop or redirects to login. The `displayName` is set for debugging.

### Real-World Cases

- **Legacy codebases:** Maintaining applications built before React Hooks (pre-16.8).
- **React Router:** The `Route` component's `render` prop is a classic render prop example.
- **Redux:** The `connect` HOC was the standard way to connect components to the Redux store.
- **Apollo GraphQL:** The `graphql` HOC was used to inject query data into components.
- **Downshift:** A popular library that uses render props for building autocomplete components.

---

## Core Concept 5: Slot & Composition Pattern

### Definitions

**Core Definition:** The Slot & Composition pattern passes pre-rendered elements as props (or children) to components, avoiding deep prop drilling and prop explosion by letting consumers compose exactly the content they need.

**Technical Definition:** Slot composition is a pattern where a component exposes named "slots" (header, footer, actions, icon, etc.) that consumers fill with their own JSX. Instead of passing numerous boolean props and render functions to configure the component, consumers pass pre-rendered elements as children or named props. This pattern is used extensively in design systems like shadcn/ui and Radix UI, where components like `Notification`, `Card`, and `Dialog` expose slots for different content areas. The pattern eliminates prop explosion, reduces unnecessary re-renders (because pre-rendered elements are not re-created on every render of the parent), and provides maximum flexibility.

**Beginner-Friendly Explanation:** Imagine you're building a notification component. The old way would be to add a prop for every possible variation: `showIcon`, `icon`, `showDismiss`, `onDismiss`, `showAction`, `actionLabel`, `renderFooter`, and so on. That's a mess. The slot pattern says: just let the user pass in whatever they want. You provide `Notification.Icon`, `Notification.Title`, `Notification.Description`, `Notification.Actions`, and the user composes exactly what they need. No prop explosion, no guessing what the API supports.

### Purposes

- To avoid prop explosion by letting consumers compose exactly what they need.
- To reduce unnecessary re-renders by passing pre-rendered elements as props.
- To provide maximum flexibility without a complex configuration API.
- To create intuitive component APIs that mirror the structure of the UI.
- To enable design system components that are both consistent and customisable.
- To avoid deep prop drilling by passing content directly where it is needed.

### Syntax Rules and Structure

**General Syntax:**

```jsx
// Card component with slots
function Card({ children, className = '' }) {
  return (
    <div className={`card ${className}`}>
      {children}
    </div>
  );
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

function CardFooter({ children }) {
  return <div className="card-footer">{children}</div>;
}

// Attach slots to parent
Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

// Usage
function App() {
  return (
    <Card>
      <Card.Header>
        <h2>Card Title</h2>
      </Card.Header>
      <Card.Body>
        <p>This is the card body content.</p>
      </Card.Body>
      <Card.Footer>
        <Button>Save</Button>
        <Button variant="outline">Cancel</Button>
      </Card.Footer>
    </Card>
  );
}
```

**Component Breakdown:**
- `Card`: The parent component that renders a container and accepts `children`.
- `Card.Header`, `Card.Body`, `Card.Footer`: Slot components that render their children in specific styled containers.
- `Card.Header = CardHeader`: Attaches the slot components to the parent for a clean API.
- The consumer composes the card by placing slot components as children.

**Syntax Rules:**
- The parent component should render its `children` without imposing structure.
- Slot components should be lightweight wrappers that apply styling and semantics.
- Slots can be attached as properties of the parent (`Parent.Slot`) or passed as named props.
- Use `children` for the primary content area and named slots for secondary areas.
- Pre-rendered elements passed as props are not re-created on parent re-render, reducing unnecessary work.

**Constraints and Limitations:**
- Slot patterns can be more verbose than prop-based APIs for simple components.
- The slot API is implicit; developers need to know which slots are available.
- TypeScript typing for slots can be challenging (though libraries like `@mikrostack/rst` provide type-safe slot systems).
- Overusing slots can lead to overly complex component APIs.

### Annotated Code Examples

**Example 1: Notification Component with Slots**

```jsx
function Notification({ children, className = '' }) {
  return (
    <div className={`notification ${className}`} role="alert">
      {children}
    </div>
  );
}

function NotificationIcon({ children }) {
  return <div className="notification-icon">{children}</div>;
}

function NotificationContent({ children }) {
  return <div className="notification-content">{children}</div>;
}

function NotificationTitle({ children }) {
  return <h4 className="notification-title">{children}</h4>;
}

function NotificationDescription({ children }) {
  return <p className="notification-description">{children}</p>;
}

function NotificationActions({ children }) {
  return <div className="notification-actions">{children}</div>;
}

function NotificationDismiss({ onDismiss }) {
  return (
    <button className="notification-dismiss" onClick={onDismiss}>
      ×
    </button>
  );
}

Notification.Icon = NotificationIcon;
Notification.Content = NotificationContent;
Notification.Title = NotificationTitle;
Notification.Description = NotificationDescription;
Notification.Actions = NotificationActions;
Notification.Dismiss = NotificationDismiss;

// Usage
function App() {
  return (
    <Notification className="notification-success">
      <Notification.Icon>
        <CheckCircle className="icon-success" />
      </Notification.Icon>
      <Notification.Content>
        <Notification.Title>Success!</Notification.Title>
        <Notification.Description>
          Your changes have been saved.
        </Notification.Description>
        <Notification.Actions>
          <Button size="sm">View</Button>
          <Button size="sm" variant="outline">Undo</Button>
        </Notification.Actions>
      </Notification.Content>
      <Notification.Dismiss onDismiss={() => console.log('Dismissed')} />
    </Notification>
  );
}
```

**Expected Output:** A styled notification with an icon, title, description, action buttons, and a dismiss button. The consumer composes exactly what they need without passing a single boolean prop.

**Why This Output Occurs:** The `Notification` component provides a container and slot components. The consumer composes the notification by placing the slots they need as children. Each slot component applies its own styling. No prop explosion, no configuration API—just composition.

### Real-World Cases

- **Design systems:** Cards, notifications, dialogs, and modals with header/body/footer slots.
- **Layout components:** Page layouts with slots for header, sidebar, main content, and footer.
- **Dashboard widgets:** Widget components with slots for title, actions, and content.
- **Marketing pages:** Section components with slots for headline, body, image, and CTA.
- **Form components:** Form field components with slots for label, input, and error message.

---

## Core Concept 6: Provider Pattern

### Definitions

**Core Definition:** The Provider Pattern uses React Context to establish a dependency injection layer that makes shared data and services available to any component in the tree without prop drilling.

**Technical Definition:** The Provider Pattern leverages React's Context API to create a provider component that wraps a subtree and makes a value available to all descendants via `useContext`. The pattern involves three steps: (1) creating a context with `createContext`, (2) wrapping the component tree with `<Context.Provider value={...}>`, and (3) consuming the value in descendants with `useContext(Context)`. In production applications, the pattern is often enhanced with a custom hook that throws if the context is missing (strict context), and the provider value is stabilised with `useMemo` and `useCallback` to prevent unnecessary re-renders. The pattern is used for cross-cutting concerns like theming, authentication, internationalisation, and dependency injection of services.

**Beginner-Friendly Explanation:** Imagine you need to share the current user's information with every component in your app. Without the Provider Pattern, you'd have to pass the user prop through every single component in the tree—even components that don't need it. The Provider Pattern solves this by wrapping your app in a `UserProvider` that makes the user available to any component that asks for it. It's like a radio broadcast: the provider broadcasts the user data, and any component with a receiver (the `useContext` hook) can tune in.

### Purposes

- To avoid prop drilling by making shared data available to any component in the tree.
- To establish dependency injection layers for services, APIs, and configuration.
- To provide a single source of truth for cross-cutting concerns (theme, auth, locale).
- To enable clean separation of concerns: the provider owns the state, consumers use it.
- To create a public API for dependencies that is friendly to refactoring and testing.
- To scope context to specific subtrees rather than making it global by default.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import React, { createContext, useContext, useMemo, useState } from 'react';

// 1. Create context with a strict default
const ThemeContext = createContext(undefined);

// 2. Create a custom hook that throws if context is missing
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// 3. Provider component
function ThemeProvider({ children, initialTheme = 'light' }) {
  const [theme, setTheme] = useState(initialTheme);

  // Stabilise the context value to prevent unnecessary re-renders
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 4. Usage in components
function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}

// 5. Wrap the app
function App() {
  return (
    <ThemeProvider initialTheme="light">
      <ThemeToggle />
    </ThemeProvider>
  );
}
```

**Component Breakdown:**
- `ThemeContext`: The context object created with `createContext(undefined)`.
- `useTheme`: A custom hook that consumes the context and throws if it is used outside the provider.
- `ThemeProvider`: The provider component that manages the theme state and provides it via context.
- `useMemo`: Stabilises the context value to prevent re-renders when `theme` hasn't changed.
- `ThemeToggle`: A consumer component that uses the `useTheme` hook.

**Syntax Rules:**
- Use `createContext(undefined)` and a custom hook that throws if the context is missing (strict context).
- Stabilise the provider value with `useMemo` and `useCallback` to prevent unnecessary re-renders.
- Export the context, the provider, and the custom hook from the same module.
- Consumers should use the custom hook (`useTheme`) rather than `useContext(ThemeContext)` directly.
- Wrap the provider around the smallest subtree that needs the context, not always the entire app.
- Use `useReducer` with context for complex state management (explicit state transitions).

**Constraints and Limitations:**
- Context causes all consumers to re-render when the context value changes.
- Overusing context can lead to "global state by accident" and hidden coupling.
- Context is a transport, not a storage—it does not decide how state changes or when updates happen.
- Splitting context into multiple providers (one for state, one for dispatch) can reduce re-renders.
- Context is not a replacement for a state management library for complex global state.

### Annotated Code Examples

**Example 1: Authentication Provider with Strict Context**

```jsx
import React, { createContext, useContext, useState, useMemo, useCallback } from 'react';

const AuthContext = createContext(undefined);

function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  const login = useCallback(async (email, password) => {
    const res = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });
    if (!res.ok) throw new Error('Invalid credentials');
    const userData = await res.json();
    setUser(userData);
  }, []);

  const logout = useCallback(() => setUser(null), []);

  const value = useMemo(
    () => ({ user, isLoading, login, logout, isAuthenticated: !!user }),
    [user, isLoading, login, logout]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// Usage
function Dashboard() {
  const { user, logout } = useAuth();
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <button onClick={logout}>Log Out</button>
    </div>
  );
}

function App() {
  return (
    <AuthProvider>
      <Dashboard />
    </AuthProvider>
  );
}
```

**Expected Output:** The `Dashboard` component displays the user's name and a logout button. The `useAuth` hook provides the user data and logout function from the `AuthProvider`.

**Why This Output Occurs:** The `AuthProvider` manages the authentication state and provides it via context. The `useAuth` hook consumes the context and throws if the provider is missing. The `useMemo` ensures the context value only changes when the auth state changes. The `Dashboard` component uses the hook to access the user and logout function.

### Real-World Cases

- **Theming:** Providing the current theme (light/dark) to all components.
- **Authentication:** Providing the current user and auth functions to the component tree.
- **Internationalisation:** Providing the current locale and translation functions.
- **Feature flags:** Providing feature flag values to components.
- **Dependency injection:** Providing API clients, analytics services, and configuration objects.
- **State management:** Combining `useReducer` with context for complex global state.

---

## References

- Compound Components: Truly Flexible React APIs – Epic React by Kent C. Dodds: https://www.epicreact.dev/compound-components
- Intro to Compound Components – Epic React by Kent C. Dodds: https://www.epicreact.dev/modules/advanced-react-patterns-v1/compound-components-patterns-intro
- React Custom Hooks Patterns – Compile-N-Run (GitHub): https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/react/12-react-advanced-patterns/3-react-custom-hooks-patterns.mdx
- React Hooks Best Practices – rtCamp: https://rtcamp.com/handbook/react-best-practices/hooks/
- Sharing State Between Components – React Documentation: https://react.dev/learn/sharing-state-between-components
- Uncontrolled Components – React Documentation: https://react.dev/reference/react-dom/components/input#controlling-an-input-with-a-state-variable
- Render Props – React Documentation (Legacy): https://legacy.reactjs.org/docs/render-props.html
- Higher-Order Components – React Documentation (Legacy): https://legacy.reactjs.org/docs/higher-order-components.html
- Typing Higher-Order Components Without Tears – Steve Kinney: https://stevekinney.com/courses/react-typescript/typing-higher-order-components
- Use Slot Pattern for Flexible Content Areas – shadcn (GitHub): https://github.com/shipshitdev/v0/blob/master/.agents/skills/shadcn/references/comp-use-slot-pattern-for-flexibility.md
- Slot – Radix UI Primitives: https://www.radix-ui.com/primitives/docs/utilities/slot
- React's Context API: Friend or Architectural Foe? – Feature-Sliced Design: https://feature-sliced.design/blog/react-context-api-guide
- Provider Pattern – PatternsDev (GitHub): https://github.com/PatternsDev/skills/blob/main/skills/javascript/provider-pattern/SKILL.md
- createContext – React Documentation: https://react.dev/reference/react/createContext
- useContext – React Documentation: https://react.dev/reference/react/useContext