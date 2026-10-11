# React Component Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React components are the fundamental building blocks of React applications. They are reusable, self-contained units of code that describe a portion of the user interface.

**Technical Definition:** A React component is a JavaScript function or class that accepts arbitrary inputs (called "props") and returns React elements (typically JSX) describing what should appear on the screen. Components are the unit of reuse in React and are composed together to form complex UIs. 

**Beginner-Friendly Explanation:** Components are like custom HTML tags that you create yourself. Instead of writing the same HTML over and over, you create a component once and reuse it wherever needed. For example, you can create a `<Button>` component and use it throughout your app.

### Key Characteristics

- **Reusable:** Components can be used multiple times across an application.
- **Composable:** Components can be nested inside other components.
- **Independent:** Each component manages its own logic and rendering.
- **Declarative:** You describe what the UI should look like, not how to build it.
- **Props-driven:** Components receive data through props from their parent.
- **Stateful or Stateless:** Components can manage internal state or be purely presentational.

### Prerequisites

- JavaScript fundamentals (functions, objects, arrays, ES6+ syntax).
- Basic understanding of JSX.
- Familiarity with ES modules (`import`/`export`).
- A React development environment (Vite, Next.js, or Create React App).

### Related Programming Areas

- **State Management:** Redux, Zustand, React Context.
- **Routing:** React Router, TanStack Router.
- **Data Fetching:** TanStack Query, SWR.
- **Testing:** Jest, React Testing Library.
- **Styling:** CSS Modules, Tailwind CSS, styled-components.

### Core Concepts / Features

---

## 1. Components

### Definitions

**Core Definition:** A component is a reusable piece of UI that encapsulates structure, logic, and styling into a single unit.

**Technical Definition:** A React component is a function that takes a `props` object and returns a React element (JSX). Components can be defined as functions (function components) or ES6 classes (class components). Components must return a single root element (or a fragment) and their names must start with a capital letter. 

**Beginner-Friendly Explanation:** Think of a component as a custom HTML element that you design. It has its own appearance and behavior. You can use it like `<MyComponent />` anywhere in your app.

### Purposes

- To break complex UIs into manageable, reusable pieces.
- To encapsulate logic and presentation together.
- To enable composition of larger UIs from smaller parts.
- To promote code reuse and maintainability.
- To separate concerns in an application.

### Syntax Rules and Structure

**General Syntax (Function Component):**

```jsx
function ComponentName(props) {
  return <JSX />;
}
```

**General Syntax (Arrow Function Component):**

```jsx
const ComponentName = (props) => {
  return <JSX />;
};
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `function ComponentName` | The component definition; name must be PascalCase. |
| `(props)` | The component's inputs (optional). |
| `return <JSX />` | The component's output; must return a single root element. |

**Syntax Rules:**

1. Component names must start with a capital letter. Lowercase names are treated as HTML tags. 
2. Components must return a single root element or a fragment. 
3. Components can accept props as arguments. 
4. Components can be defined as functions or classes. 

**Constraints and Limitations:**

- Components cannot return multiple sibling elements without a wrapper.
- Component names must be valid JavaScript identifiers.
- Components must be pure with respect to their props. 

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Function Component

```jsx
// Step 1: Define the component
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// Step 2: Use the component
function App() {
  return (
    <div>
      <Welcome />
      <Welcome />
    </div>
  );
}
```

**Expected Output:**

```html
<div>
  <h1>Hello, World!</h1>
  <h1>Hello, World!</h1>
</div>
```

**Why this output:** The `Welcome` component is called twice, each producing an `<h1>` element. React renders both instances.

#### Example 2: Component with Props

```jsx
function Greeting({ name, age }) {
  return (
    <div>
      <h2>Hello, {name}!</h2>
      <p>You are {age} years old.</p>
    </div>
  );
}

function App() {
  return (
    <div>
      <Greeting name="Alice" age={30} />
      <Greeting name="Bob" age={25} />
    </div>
  );
}
```

**Expected Output:**

```html
<div>
  <div>
    <h2>Hello, Alice!</h2>
    <p>You are 30 years old.</p>
  </div>
  <div>
    <h2>Hello, Bob!</h2>
    <p>You are 25 years old.</p>
  </div>
</div>
```

**Why this output:** Each `Greeting` instance receives different props and renders different content.

### Real-World Cases

- **Design systems:** Buttons, inputs, modals, and cards are all components.
- **Layouts:** Headers, sidebars, and footers are composed of components.
- **Pages:** Each page in a React app is typically a component.
- **Forms:** Form fields, validation messages, and submit buttons are components.

### References

- React — Your First Component — https://react.dev/learn/your-first-component
- React — Components and Props — https://legacy.reactjs.org/docs/components-and-props.html

---

## 2. Component Types

### Definitions

**Core Definition:** React supports two ways of defining components: function components and class components.

**Technical Definition:** Function components are plain JavaScript functions that return JSX. Class components are ES6 classes that extend `React.Component` and implement a `render()` method. Since React 16.8 (introduction of Hooks), function components can use state and lifecycle features, making them the recommended approach for new code. 

**Beginner-Friendly Explanation:** You can write components either as functions or as classes. Functions are simpler and are now the standard way to write React components. Classes are older and still work, but new code should use functions.

### Purposes

- To provide flexibility in how components are defined.
- To allow migration from older class-based codebases.
- To enable the use of Hooks in function components.
- To support legacy patterns in existing applications.

### Sub-Features

---

#### 2.1 Function Components

##### Definitions

**Core Definition:** A function component is a JavaScript function that returns JSX.

**Technical Definition:** Function components are functions that accept `props` as their first argument and return React elements. They can use Hooks (`useState`, `useEffect`, etc.) to manage state and side effects. They are the recommended way to write components in modern React. 

**Beginner-Friendly Explanation:** A function component is just a function that returns some JSX. It is the simplest way to write a component.

##### Purposes

- To write simple, concise components.
- To use Hooks for state and lifecycle management.
- To avoid `this` binding issues.
- To reduce bundle size compared to class components.
- To enable React Server Components and future React features.

##### Syntax Rules and Structure

**General Syntax:**

```jsx
function ComponentName(props) {
  // Hooks can be used here
  return <JSX />;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `function ComponentName` | Function declaration. |
| `props` | The props object (optional). |
| `return <JSX />` | The rendered output. |

**Syntax Rules:**

1. Function components can use Hooks. 
2. No `this` keyword is needed. 
3. Props are accessed directly as function arguments. 
4. Must return a single root element or fragment.

**Constraints and Limitations:**

- Cannot use class lifecycle methods directly; use `useEffect` instead. 
- Hooks must be called at the top level of the component, not inside loops or conditions. 

##### Annotated Code Examples

```jsx
import { useState, useEffect } from 'react';

function Counter() {
  // State Hook
  const [count, setCount] = useState(0);

  // Effect Hook (runs after render)
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Expected Output:** A button that increments a counter and updates the document title.

**Why this output:** `useState` creates a state variable. `useEffect` runs after render and updates the title. Clicking the button updates state, triggering a re-render.

##### Real-World Cases

- **Interactive UIs:** Counter, toggle, and form components.
- **Data fetching:** Components that fetch data with `useEffect`.
- **Context consumers:** Components that use `useContext`.

##### References

- React — Your First Component — https://react.dev/learn/your-first-component
- React — Using the State Hook — https://react.dev/reference/react/useState

---

#### 2.2 Class Components

##### Definitions

**Core Definition:** A class component is an ES6 class that extends `React.Component` and implements a `render()` method.

**Technical Definition:** Class components extend `React.Component` (or `React.PureComponent`) and must define a `render()` method. They can access props via `this.props`, manage state via `this.state` and `this.setState()`, and use lifecycle methods like `componentDidMount()`. Class components are not deprecated but are no longer recommended for new code. 

**Beginner-Friendly Explanation:** A class component is like a blueprint for a component written as a class. It has special methods for handling state and lifecycle events. It is older and more verbose than function components.

##### Purposes

- To support legacy codebases that use class components.
- To use lifecycle methods in older React versions.
- To serve as error boundaries (which require class components). 

##### Syntax Rules and Structure

**General Syntax:**

```jsx
class ComponentName extends React.Component {
  constructor(props) {
    super(props);
    this.state = { ... };
  }

  render() {
    return <JSX />;
  }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class ComponentName extends React.Component` | Class declaration. |
| `constructor(props)` | Initializes state and binds methods. |
| `this.state` | The component's internal state. |
| `this.props` | The component's props. |
| `render()` | Required method that returns JSX. |

**Syntax Rules:**

1. Must extend `React.Component` or `React.PureComponent`. 
2. Must define a `render()` method. 
3. State is initialized in the constructor. 
4. Event handlers must be bound to `this`. 
5. Lifecycle methods are used for side effects. 

**Constraints and Limitations:**

- Cannot use Hooks. 
- More verbose than function components. 
- `this` binding can cause bugs. 
- Not recommended for new code. 

##### Annotated Code Examples

```jsx
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  componentDidMount() {
    console.log('Component mounted');
  }

  componentDidUpdate(prevProps, prevState) {
    console.log(`Count changed from ${prevState.count} to ${this.state.count}`);
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}
```

**Expected Output:** A counter that logs lifecycle events.

**Why this output:** `componentDidMount` runs after the first render. `componentDidUpdate` runs after each state update. `this.setState` triggers a re-render.

##### Real-World Cases

- **Legacy applications:** Maintaining older React codebases.
- **Error boundaries:** Class components are required for error boundaries. 
- **Lifecycle-heavy components:** Components that need fine-grained lifecycle control.

##### References

- React — React.Component — https://react.dev/reference/react/Component
- React — Migrating from Class Components to Functions — https://react.dev/reference/react/Component#alternatives

---

## 3. Component Naming

### Definitions

**Core Definition:** Component naming refers to the conventions for naming React components, files, props, and event handlers.

**Technical Definition:** React components must use PascalCase (e.g., `UserProfile`, `SearchBar`). File names should match the component name. Props use camelCase, and event handler props use the `on` prefix (e.g., `onClick`). Hooks use the `use` prefix. 

**Beginner-Friendly Explanation:** Always name your components with a capital letter at the start of each word (PascalCase). This helps React tell the difference between your components and regular HTML tags.

### Purposes

- To distinguish components from HTML tags.
- To make code readable and predictable.
- To follow community standards and conventions.
- To avoid naming conflicts.

### Syntax Rules and Structure

**Naming Conventions Table:**

| Element | Convention | Example |
|---------|-----------|---------|
| Component | PascalCase | `UserProfile`, `SearchBar` |
| Component file | PascalCase | `UserProfile.jsx` |
| Props | camelCase | `userId`, `isAdmin` |
| Boolean props | `is`/`has`/`should` prefix | `isDisabled`, `hasIcon` |
| Event handler prop | `on` prefix | `onClick`, `onSubmit` |
| Event handler function | `handle` prefix | `handleClick`, `handleSubmit` |
| Custom hook | `use` prefix | `useFormInput`, `useAuth` |
| HOC | `with` prefix | `withAuth`, `withRouter` |

**Syntax Rules:**

1. Components must use PascalCase. 
2. Files should match the component name. 
3. Hooks must start with `use`. 
4. HOCs use the `with` prefix. 

**Constraints and Limitations:**

- Lowercase component names are treated as HTML tags. 
- Inconsistent naming reduces readability.

### Annotated Code Examples

```jsx
// Good: PascalCase component
function UserProfile({ userId, isAdmin }) {
  function handleClick() {
    console.log('Clicked');
  }

  return <button onClick={handleClick}>Edit</button>;
}

// Bad: lowercase component (treated as HTML tag)
function userProfile() {
  return <div>Not a component</div>;
}
```

**Expected Output:** The `UserProfile` component renders a button. The `userProfile` function is treated as an unknown HTML tag.

### Real-World Cases

- **Team collaboration:** Consistent naming makes code reviews easier.
- **Design systems:** Component names become part of the public API.
- **Testing:** Test files are named after components.

### References

- React — Naming Conventions — https://react.dev/learn/your-first-component#naming-conventions
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks

---

## 4. Component Composition

### Definitions

**Core Definition:** Component composition is the practice of building complex components by combining simpler ones.

**Technical Definition:** Composition in React is achieved by nesting components inside one another, typically using the `children` prop or by passing components as props. React recommends composition over inheritance for code reuse. 

**Beginner-Friendly Explanation:** Instead of making one giant component, you build small components and put them together like LEGO blocks to create something bigger.

### Purposes

- To reuse code without inheritance.
- To create flexible, configurable components.
- To separate concerns and improve maintainability.
- To enable the "slot" pattern for layout components.

### Syntax Rules and Structure

**General Syntax (Children Prop):**

```jsx
<Parent>
  <Child />
</Parent>
```

**General Syntax (Named Slots):**

```jsx
<Layout
  header={<Header />}
  sidebar={<Sidebar />}
  content={<Content />}
/>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `children` | Special prop containing nested JSX. |
| Named props | Custom props that accept JSX elements. |

**Syntax Rules:**

1. Components can render their `children` using `{props.children}`. 
2. Any JSX can be passed as a prop. 
3. Composition is preferred over inheritance. 

**Constraints and Limitations:**

- Passing too many named slots can make APIs complex.
- `children` is a single prop; for multiple slots, use named props.

### Annotated Code Examples

```jsx
// Layout component using children
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Usage
function App() {
  return (
    <Card title="Welcome">
      <p>This is the card content.</p>
      <button>Learn more</button>
    </Card>
  );
}
```

**Expected Output:**

```html
<div class="card">
  <h2>Welcome</h2>
  <div class="card-body">
    <p>This is the card content.</p>
    <button>Learn more</button>
  </div>
</div>
```

**Why this output:** The `<p>` and `<button>` are passed as `children` to `Card`, which renders them inside `card-body`.

### Real-World Cases

- **Layout components:** Headers, sidebars, and footers use children for content.
- **Modal components:** Modal body content is passed as children.
- **Design systems:** Components like `<Card>` and `<Dialog>` use composition.

### References

- React — Composition vs Inheritance — https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component

---

## 5. Component Reuse

### Definitions

**Core Definition:** Component reuse is the ability to use the same component in multiple places within an application.

**Technical Definition:** React enables reuse through composition, props, higher-order components (HOCs), render props, and custom hooks. A component can be instantiated multiple times with different props to produce different outputs. 

**Beginner-Friendly Explanation:** Write a component once, use it many times. For example, a `<Button>` component can be used for "Submit", "Cancel", and "Delete" buttons.

### Purposes

- To reduce code duplication.
- To ensure consistency across the UI.
- To simplify maintenance and updates.
- To enable testing in isolation.

### Syntax Rules and Structure

**Reuse Patterns:**

| Pattern | Description |
|---------|-------------|
| Props | Pass different data to the same component. |
| Composition | Nest components to build complex UIs. |
| HOCs | Wrap components to add functionality. |
| Render Props | Pass a function to control rendering. |
| Custom Hooks | Extract reusable logic. |

**Syntax Rules:**

1. Components can be rendered multiple times. 
2. Each instance has its own state and lifecycle. 
3. Props determine the output of each instance. 

### Annotated Code Examples

```jsx
function Button({ label, variant, onClick }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
}

function App() {
  return (
    <div>
      <Button label="Submit" variant="primary" onClick={() => alert('Submitted')} />
      <Button label="Cancel" variant="secondary" onClick={() => alert('Cancelled')} />
      <Button label="Delete" variant="danger" onClick={() => alert('Deleted')} />
    </div>
  );
}
```

**Expected Output:** Three buttons with different labels, styles, and handlers.

**Why this output:** The same `Button` component is reused with different props.

### Real-World Cases

- **Form buttons:** Submit, cancel, and reset buttons use the same component.
- **Product cards:** A `ProductCard` component is reused for each product.
- **Navigation items:** Menu items use a shared `NavItem` component.

### References

- React — Reusing Logic with Custom Hooks — https://react.dev/learn/reusing-logic-with-custom-hooks
- React — Higher-Order Components — https://legacy.reactjs.org/docs/higher-order-components.html

---

## 6. Component Lifecycle

### Definitions

**Core Definition:** The component lifecycle describes the phases a component goes through from creation to removal.

**Technical Definition:** React components go through three main phases: Mounting (creation and insertion into the DOM), Updating (re-rendering due to props or state changes), and Unmounting (removal from the DOM). In class components, lifecycle methods like `componentDidMount()`, `componentDidUpdate()`, and `componentWillUnmount()` are called at each phase. In function components, `useEffect` handles these phases. 

**Beginner-Friendly Explanation:** Every component has a life: it is born (mounted), it grows and changes (updates), and eventually it dies (unmounts). You can run code at each of these stages.

### Purposes

- To run side effects at specific points in a component's life.
- To fetch data when a component mounts.
- To clean up resources when a component unmounts.
- To respond to prop or state changes.

### Syntax Rules and Structure

**Class Component Lifecycle:**

| Phase | Method | Purpose |
|-------|--------|---------|
| Mounting | `constructor()` | Initialize state. |
| Mounting | `render()` | Return JSX. |
| Mounting | `componentDidMount()` | Run after first render. |
| Updating | `shouldComponentUpdate()` | Control re-renders. |
| Updating | `render()` | Re-render. |
| Updating | `componentDidUpdate()` | Run after update. |
| Unmounting | `componentWillUnmount()` | Cleanup. |

**Function Component Lifecycle (useEffect):**

```jsx
useEffect(() => {
  // Runs after every render
  return () => {
    // Cleanup (runs before next effect or unmount)
  };
}, [dependencies]);
```

**Syntax Rules:**

1. `useEffect` runs after render. 
2. The dependency array controls when the effect re-runs. 
3. The cleanup function runs before the next effect and on unmount. 

### Annotated Code Examples

```jsx
import { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // Mounting: start timer
    const interval = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);

    // Unmounting: cleanup
    return () => clearInterval(interval);
  }, []);

  return <p>Seconds: {seconds}</p>;
}
```

**Expected Output:** A timer that increments every second and cleans up when unmounted.

**Why this output:** `useEffect` with an empty dependency array runs once on mount. The cleanup function clears the interval on unmount. 

### Real-World Cases

- **Data fetching:** Fetch data in `useEffect` when the component mounts.
- **Subscriptions:** Subscribe to WebSocket events and clean up on unmount.
- **Timers:** Start and stop timers based on component lifecycle.

### References

- React — Lifecycle of Reactive Effects — https://react.dev/learn/lifecycle-of-reactive-effects
- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects

---

## 7. Component State

### Definitions

**Core Definition:** State is data that a component manages internally and that can change over time.

**Technical Definition:** State is a component's memory. It is initialized with `useState` (function components) or `this.state` (class components) and updated with `setState` functions. State changes trigger re-renders. State is local to the component and cannot be accessed by other components directly. 

**Beginner-Friendly Explanation:** State is like a component's personal notebook. It can write things down, change them, and read them back. When the notebook changes, the component updates what it shows on screen.

### Purposes

- To store data that changes over time (user input, fetched data).
- To trigger re-renders when data changes.
- To manage component-specific UI state (open/closed, active/inactive).
- To persist values between renders.

### Syntax Rules and Structure

**Function Component State:**

```jsx
const [state, setState] = useState(initialValue);
```

**Class Component State:**

```jsx
this.state = { key: value };
this.setState({ key: newValue });
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `useState(initialValue)` | Declares a state variable. |
| `state` | The current state value. |
| `setState` | Function to update state. |
| `this.state` | Class component state object. |
| `this.setState()` | Method to update class state. |

**Syntax Rules:**

1. State updates are asynchronous. 
2. State updates are batched in React 18+. 
3. Use updater functions for state that depends on previous state: `setCount(c => c + 1)`. 
4. Never mutate state directly. 

**Constraints and Limitations:**

- State updates may be batched; do not rely on immediate updates. 
- State is local; to share state, lift it up. 
- State should not be mutated directly. 

### Annotated Code Examples

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  function handleIncrement() {
    // Updater function ensures correct value
    setCount((prevCount) => prevCount + 1);
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

**Expected Output:** A counter that increments by 1 on each click.

**Why this output:** `setCount` with an updater function ensures the state is updated based on the previous value, even with batching. 

### Real-World Cases

- **Form inputs:** Storing user input values.
- **Toggles:** Open/closed states for modals and dropdowns.
- **Counters:** Shopping cart item counts.
- **Data fetching:** Storing fetched data.

### References

- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory
- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates

---

## 8. Component Props

### Definitions

**Core Definition:** Props (short for "properties") are read-only inputs passed from a parent component to a child component.

**Technical Definition:** Props are JavaScript objects containing data passed to a component. They are immutable from the child's perspective and flow one-way (parent to child). Props can be any JavaScript value: strings, numbers, objects, arrays, functions, or JSX elements. 

**Beginner-Friendly Explanation:** Props are like arguments you pass to a function. They let you customize how a component looks and behaves. A parent component gives props to a child component.

### Purposes

- To pass data from parent to child components.
- To customize component behavior and appearance.
- To pass event handlers and callbacks.
- To enable component reuse with different configurations.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<ChildComponent propName={value} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `propName` | The name of the prop (camelCase). |
| `value` | The value to pass (any JavaScript expression). |

**Syntax Rules:**

1. Props are read-only; never modify them. 
2. Props flow one-way from parent to child. 
3. To pass data back to the parent, use callback props. 
4. Props can be destructured in function components: `function Child({ name, age })`. 

**Constraints and Limitations:**

- Props cannot be modified by the child. 
- Prop drilling (passing props through many levels) can become cumbersome. 
- Props are compared by reference for memoization. 

### Annotated Code Examples

```jsx
function UserCard({ user, onEdit }) {
  return (
    <div className="card">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
}

function App() {
  const user = { id: 1, name: 'Alice', email: 'alice@example.com' };

  function handleEdit(id) {
    console.log(`Editing user ${id}`);
  }

  return <UserCard user={user} onEdit={handleEdit} />;
}
```

**Expected Output:** A card showing user details with an edit button.

**Why this output:** The `user` object and `onEdit` callback are passed as props. The child uses them to render content and handle clicks.

### Real-World Cases

- **Lists:** Passing item data to list item components.
- **Forms:** Passing form values and handlers to input components.
- **Layouts:** Passing content and configuration to layout components.

### References

- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component
- React — Components and Props — https://legacy.reactjs.org/docs/components-and-props.html

---

## 9. Component Rendering

### Definitions

**Core Definition:** Rendering is the process by which React calls components to determine what should appear on screen.

**Technical Definition:** React rendering has three steps: Trigger (initial render or state update), Render (React calls components to compute the UI), and Commit (React applies changes to the DOM). Rendering must be pure: given the same inputs, a component must return the same output. 

**Beginner-Friendly Explanation:** Rendering is React figuring out what your UI should look like and then updating the screen to match.

### Purposes

- To compute the UI based on current state and props.
- To determine what needs to change in the DOM.
- To ensure the UI is consistent with application state.

### Syntax Rules and Structure

**Render Process:**

| Step | Description |
|------|-------------|
| Trigger | Initial render or state update. |
| Render | React calls components to compute JSX. |
| Commit | React updates the DOM. |

**Syntax Rules:**

1. Rendering must be pure. 
2. Rendering is recursive: React renders child components. 
3. Rendering does not always produce DOM updates. 

### Annotated Code Examples

```jsx
function App() {
  const [count, setCount] = useState(0);

  console.log('Rendering App');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Expected Output:** "Rendering App" logged on each render.

**Why this output:** React calls `App` on initial render and on each state update, logging the message each time.

### Real-World Cases

- **Performance monitoring:** Tracking render frequency with `console.log`.
- **Debugging:** Understanding why components re-render.

### References

- React — Render and Commit — https://react.dev/learn/render-and-commit
- React — Keeping Components Pure — https://react.dev/learn/keeping-components-pure

---

## 10. Component Updates

### Definitions

**Core Definition:** Component updates occur when a component's state or props change, triggering a re-render.

**Technical Definition:** React updates components when their state changes (via `setState` or `useState` updater) or when their parent re-renders with new props. Updates are batched for performance. React's reconciliation algorithm determines the minimal DOM changes needed. 

**Beginner-Friendly Explanation:** When something changes in a component (like the user clicks a button), the component updates to reflect the new data.

### Purposes

- To reflect changes in state or props in the UI.
- To re-render components efficiently.
- To batch multiple updates into a single render.

### Syntax Rules and Structure

**State Update:**

```jsx
setState(newValue);
setState(prev => prev + 1); // Updater function
```

**Syntax Rules:**

1. State updates trigger re-renders. 
2. React batches updates in event handlers (React 18+). 
3. Use updater functions when new state depends on previous state. 

### Annotated Code Examples

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    // Batched: only one re-render
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return <button onClick={handleClick}>+3 (but only +1)</button>;
}
```

**Expected Output:** The counter increments by 1, not 3, because of batching and stale state.

**Why this output:** React batches the three `setCount` calls. Each call uses the same `count` value (0), so the final state is 1. To fix, use `setCount(c => c + 1)`. 

### Real-World Cases

- **Form submissions:** Updating state with form data.
- **Shopping carts:** Adding/removing items.
- **Filters:** Updating filter criteria and re-rendering lists.

### References

- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React — State as a Snapshot — https://react.dev/learn/state-as-a-snapshot

---

## 11. Component Unmounting

### Definitions

**Core Definition:** Unmounting is the phase when a component is removed from the DOM.

**Technical Definition:** Unmounting occurs when React removes a component from the tree. In class components, `componentWillUnmount()` is called. In function components, the cleanup function returned from `useEffect` runs. This is the place to cancel timers, abort fetches, and unsubscribe from events. 

**Beginner-Friendly Explanation:** Unmounting is when a component is removed from the screen. You should clean up anything the component was doing (like timers) at this point.

### Purposes

- To clean up resources (timers, subscriptions, event listeners).
- To prevent memory leaks.
- To cancel in-flight requests.

### Syntax Rules and Structure

**Class Component:**

```jsx
componentWillUnmount() {
  // Cleanup
}
```

**Function Component:**

```jsx
useEffect(() => {
  // Setup
  return () => {
    // Cleanup on unmount
  };
}, []);
```

**Syntax Rules:**

1. Cleanup runs before the component is removed. 
2. Return a cleanup function from `useEffect`. 
3. Cleanup runs before the next effect execution as well. 

### Annotated Code Examples

```jsx
function DataFetcher({ url }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    fetch(url, { signal: controller.signal })
      .then((res) => res.json())
      .then(setData)
      .catch((err) => {
        if (err.name !== 'AbortError') console.error(err);
      });

    return () => controller.abort();
  }, [url]);

  return <div>{data ? data.title : 'Loading...'}</div>;
}
```

**Expected Output:** Data is fetched and displayed. If the component unmounts or `url` changes, the fetch is aborted.

**Why this output:** The cleanup function calls `controller.abort()`, cancelling the fetch and preventing state updates on an unmounted component. 

### Real-World Cases

- **Data fetching:** Aborting fetch requests on unmount.
- **Subscriptions:** Unsubscribing from WebSocket or event emitters.
- **Timers:** Clearing intervals and timeouts.

### References

- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects
- React — useEffect — https://react.dev/reference/react/useEffect

---

## 12. Pure Components

### Definitions

**Core Definition:** A pure component is a component that always returns the same output for the same props and state.

**Technical Definition:** Pure components in React are components whose rendering logic is pure: given the same inputs, they produce the same output. React provides `React.memo` (for function components) and `React.PureComponent` (for class components) to skip re-renders when props and state have not changed (shallow comparison). 

**Beginner-Friendly Explanation:** A pure component is like a vending machine: if you press the same button, you get the same snack. It does not change its behavior based on outside factors.

### Purposes

- To improve performance by skipping unnecessary re-renders.
- To ensure predictable component behavior.
- To enable React's optimization features.

### Syntax Rules and Structure

**Function Component (React.memo):**

```jsx
const MemoizedComponent = memo(function Component(props) {
  return <JSX />;
});
```

**Class Component (PureComponent):**

```jsx
class Component extends React.PureComponent {
  render() {
    return <JSX />;
  }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `memo()` | HOC that memoizes a function component. |
| `React.PureComponent` | Class that implements `shouldComponentUpdate` with shallow comparison. |

**Syntax Rules:**

1. Pure components should not have side effects during render. 
2. `memo` compares props with `Object.is`. 
3. `PureComponent` compares props and state shallowly. 

**Constraints and Limitations:**

- Shallow comparison may miss deep changes. 
- Memoization is a performance optimization, not a guarantee. 
- Do not use memo if the component always renders differently.

### Annotated Code Examples

```jsx
import { memo, useState } from 'react';

const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting rendered');
  return <h1>Hello, {name}!</h1>;
});

function App() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" />
      <input value={address} onChange={(e) => setAddress(e.target.value)} placeholder="Address" />
      <Greeting name={name} />
    </div>
  );
}
```

**Expected Output:** "Greeting rendered" logs only when `name` changes, not when `address` changes.

**Why this output:** `memo` prevents `Greeting` from re-rendering when its props (`name`) have not changed. 

### Real-World Cases

- **Large lists:** Memoizing list items to prevent re-renders.
- **Expensive components:** Components with heavy rendering logic.
- **Design systems:** Memoizing components used in many places.

### References

- React — memo — https://react.dev/reference/react/memo
- React — PureComponent — https://react.dev/reference/react/PureComponent

---

## 13. Impure Components

### Definitions

**Core Definition:** An impure component is a component that produces different output for the same props and state, or has side effects during render.

**Technical Definition:** Impure components violate React's purity rules. They may call functions like `Math.random()` or `Date.now()` during render, mutate external variables, or perform side effects (e.g., modifying the DOM, setting timers). Impure rendering breaks React's assumptions and leads to bugs, hydration mismatches, and incorrect memoization. 

**Beginner-Friendly Explanation:** An impure component is unpredictable. If you give it the same inputs, it might produce different outputs, which confuses React.

### Purposes

- (Impure components are an anti-pattern; they do not have legitimate purposes.)
- To illustrate what NOT to do in React rendering.

### Syntax Rules and Structure

**Impure Examples:**

```jsx
// ❌ Impure: uses Math.random()
function Component() {
  const id = Math.random();
  return <div key={id}>Content</div>;
}

// ❌ Impure: mutates external variable
let count = 0;
function Component() {
  count++;
  return <div>{count}</div>;
}
```

**Syntax Rules:**

1. Rendering must be pure. 
2. Side effects must run outside render (in `useEffect`). 
3. Do not call impure functions during render. 

### Annotated Code Examples

```jsx
// ❌ Impure component
function ImpureClock() {
  return <div>Current time: {Date.now()}</div>;
}

// ✅ Pure component (side effect in useEffect)
function PureClock() {
  const [time, setTime] = useState(() => Date.now());

  useEffect(() => {
    const interval = setInterval(() => setTime(Date.now()), 1000);
    return () => clearInterval(interval);
  }, []);

  return <div>Current time: {time}</div>;
}
```

**Expected Output:** The impure clock re-renders with a new time on every render. The pure clock updates only once per second.

**Why this output:** `Date.now()` in render produces a new value each time. The pure version stores the time in state and updates it via `useEffect`.

### Real-World Cases

- **Debugging:** Identifying impure components helps fix hydration errors.
- **Strict Mode:** React Strict Mode double-renders to detect impurity. 

### References

- React — Keeping Components Pure — https://react.dev/learn/keeping-components-pure
- React — Components and Hooks Must Be Pure — https://react.dev/reference/rules/components-and-hooks-must-be-pure

---

## 14. Component Boundaries

### Definitions

**Core Definition:** Component boundaries are React components that catch JavaScript errors anywhere in their child component tree and display a fallback UI.

**Technical Definition:** Error boundaries are class components that implement `static getDerivedStateFromError()` and/or `componentDidCatch()`. They catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them. They do not catch errors in event handlers, asynchronous code, or server-side rendering. 

**Beginner-Friendly Explanation:** Error boundaries are like safety nets. If a component crashes, the error boundary catches the error and shows a friendly message instead of a broken page.

### Purposes

- To prevent a single component error from crashing the entire app.
- To display a fallback UI when errors occur.
- To log errors to an error reporting service.
- To isolate errors to specific parts of the UI.

### Syntax Rules and Structure

**General Syntax:**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `getDerivedStateFromError` | Updates state to show fallback UI. |
| `componentDidCatch` | Logs error information. |
| `this.props.children` | Renders children when no error. |

**Syntax Rules:**

1. Only class components can be error boundaries. 
2. Must implement `getDerivedStateFromError` or `componentDidCatch`. 
3. Error boundaries only catch errors in their child tree. 
4. They do not catch errors in event handlers, async code, or SSR. 

**Constraints and Limitations:**

- Cannot be function components (unless using a library like `react-error-boundary`).
- Do not catch all error types. 
- Errors in the error boundary itself are not caught. 

### Annotated Code Examples

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

function BuggyComponent() {
  throw new Error('I crashed!');
}

function App() {
  return (
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

**Expected Output:** "Something went wrong." is displayed instead of a crashed app.

**Why this output:** `BuggyComponent` throws an error during render. The `ErrorBoundary` catches it and renders the fallback UI. 

### Real-World Cases

- **Route-level boundaries:** Wrapping each route in an error boundary.
- **Widget-level boundaries:** Wrapping individual widgets to isolate failures.
- **Third-party integrations:** Catching errors from external components.

### References

- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React — react-error-boundary — https://www.npmjs.com/package/react-error-boundary

---

## 15. Component Patterns

### Definitions

**Core Definition:** Component patterns are proven solutions to recurring component design problems in React.

**Technical Definition:** React component patterns include Compound Components (components that share implicit state via context), Render Props (components that accept a function prop to control rendering), Higher-Order Components (functions that wrap a component and return a new component), and Slot Patterns (components that accept named props for different UI regions). 

**Beginner-Friendly Explanation:** Patterns are like recipes for building components. They help you solve common problems in a consistent, reusable way.

### Purposes

- To provide reusable solutions to common design problems.
- To improve code readability and maintainability.
- To enable flexible and composable component APIs.
- To share state and logic between components.

### Sub-Features

---

#### 15.1 Compound Components

##### Definitions

**Core Definition:** Compound components are multiple components designed to work together as a single unit, sharing implicit state.

**Technical Definition:** Compound components use React Context to share state between a parent component and its child components. The parent owns the state, and children consume it via context. This pattern is used by libraries like Radix UI and Headless UI. 

**Beginner-Friendly Explanation:** Compound components are like a set of related components that work together, such as `<Tabs>`, `<Tabs.List>`, and `<Tabs.Content>`.

##### Annotated Code Example

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

function TabList({ children }) {
  return <div role="tablist">{children}</div>;
}

function Tab({ value, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      role="tab"
      aria-selected={activeTab === value}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanel({ value, children }) {
  const { activeTab } = useContext(TabsContext);
  return activeTab === value ? <div role="tabpanel">{children}</div> : null;
}

// Usage
function App() {
  return (
    <Tabs defaultTab="profile">
      <TabList>
        <Tab value="profile">Profile</Tab>
        <Tab value="settings">Settings</Tab>
      </TabList>
      <TabPanel value="profile">Profile content</TabPanel>
      <TabPanel value="settings">Settings content</TabPanel>
    </Tabs>
  );
}
```

**Expected Output:** A tabbed interface with Profile and Settings tabs.

**Why this output:** The `Tabs` component manages the active tab state and shares it via context. `Tab` and `TabPanel` consume the context to render correctly. 

##### References

- Radix UI — Compound Components — https://www.radix-ui.com/primitives/docs/overview/introduction
- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context

---

#### 15.2 Render Props

##### Definitions

**Core Definition:** Render props is a pattern where a component accepts a function prop that returns JSX.

**Technical Definition:** A component with a render prop takes a function as a prop and calls it to determine what to render. This inverts control: the consumer decides what to render while the component provides the data or behavior. 

**Beginner-Friendly Explanation:** A render prop is a function you pass to a component that tells it what to render.

##### Annotated Code Example

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <p>Mouse position: {x}, {y}</p>
      )}
    />
  );
}
```

**Expected Output:** The mouse position updates as the user moves the mouse.

**Why this output:** `MouseTracker` tracks the mouse and calls the `render` prop with the position. The consumer decides how to display it.

##### References

- React — Render Props — https://legacy.reactjs.org/docs/render-props.html

---

#### 15.3 Higher-Order Components (HOC)

##### Definitions

**Core Definition:** A higher-order component is a function that takes a component and returns a new component with additional props or behavior.

**Technical Definition:** HOCs are functions like `withAuth(Component)` that wrap the original component in a container that provides extra functionality. They are used for cross-cutting concerns like authentication, logging, or data fetching. 

**Beginner-Friendly Explanation:** A HOC is a function that wraps a component to give it superpowers.

##### Annotated Code Example

```jsx
function withAuth(Component) {
  return function WithAuth(props) {
    const isAuthenticated = checkAuth();
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    return <Component {...props} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
```

**Expected Output:** The `Dashboard` is rendered only if the user is authenticated.

**Why this output:** `withAuth` wraps `Dashboard` and checks authentication before rendering it.

##### References

- React — Higher-Order Components — https://legacy.reactjs.org/docs/higher-order-components.html

---

#### 15.4 Slot Pattern

##### Definitions

**Core Definition:** The slot pattern uses named props to pass JSX elements into specific regions of a layout component.

**Technical Definition:** Instead of using a single `children` prop, layout components accept multiple named props (e.g., `header`, `sidebar`, `content`) that accept JSX elements. This allows the parent to control structure while the consumer controls content. 

**Beginner-Friendly Explanation:** Slots are like placeholder areas in a layout. You fill each slot with your own content.

##### Annotated Code Example

```jsx
function Layout({ header, sidebar, content }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{content}</main>
    </div>
  );
}

// Usage
function App() {
  return (
    <Layout
      header={<h1>My App</h1>}
      sidebar={<nav>Navigation</nav>}
      content={<p>Main content</p>}
    />
  );
}
```

**Expected Output:** A layout with header, sidebar, and main content.

**Why this output:** Each named prop is rendered in its designated location in the layout.

##### References

- React — Passing JSX as Children — https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children

---

### Real-World Cases

- **Compound components:** Radix UI, Headless UI, Reach UI.
- **Render props:** react-hook-form `<Controller>`, TanStack Query.
- **HOCs:** react-redux `connect`, `withRouter`.
- **Slot patterns:** Next.js layouts, Remix `<Outlet />`.

### References

- React — Composition vs Inheritance — https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- Patterns.dev — React Patterns — https://www.patterns.dev/react

---

## References

- React — Your First Component — https://react.dev/learn/your-first-component
- React — Components and Props — https://legacy.reactjs.org/docs/components-and-props.html
- React — State: A Component's Memory — https://react.dev/learn/state-a-components-memory
- React — Lifecycle of Reactive Effects — https://react.dev/learn/lifecycle-of-reactive-effects
- React — Render and Commit — https://react.dev/learn/render-and-commit
- React — Queueing a Series of State Updates — https://react.dev/learn/queueing-a-series-of-state-updates
- React — Keeping Components Pure — https://react.dev/learn/keeping-components-pure
- React — memo — https://react.dev/reference/react/memo
- React — PureComponent — https://react.dev/reference/react/PureComponent
- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component
- React — Composition vs Inheritance — https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- React — Higher-Order Components — https://legacy.reactjs.org/docs/higher-order-components.html
- React — Render Props — https://legacy.reactjs.org/docs/render-props.html
- React — Using the State Hook — https://react.dev/reference/react/useState
- React — Rules of Hooks — https://react.dev/reference/rules/rules-of-hooks
- Patterns.dev — React Patterns — https://www.patterns.dev/react