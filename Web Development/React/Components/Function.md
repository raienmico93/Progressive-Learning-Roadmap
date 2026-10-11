# React Function Components — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A function component is a JavaScript function that accepts a `props` object and returns a React element (typically JSX) describing what should appear on the screen.

**Technical Definition:** A function component is a plain JavaScript function (declared with `function` or arrow function syntax) that takes a single `props` argument and returns a React element. Since React 16.8, function components can use Hooks (`useState`, `useEffect`, etc.) to manage state and lifecycle behaviour, making them functionally equivalent to class components while requiring less boilerplate.

**Beginner-Friendly Explanation:** A function component is just a function that returns some JSX. It is the simplest and most modern way to write a React component. You give it some data (called "props"), and it gives back the HTML you want to see on screen.

### Key Characteristics

- **Concise:** Less code than class components; no `this` binding or `render()` method.
- **Hooks-enabled:** Can use `useState`, `useEffect`, and other Hooks for state and side effects.
- **Pure by default:** Should return the same output for the same props.
- **Composable:** Can be nested inside other components and combined to build complex UIs.
- **Standard:** The React team recommends function components with Hooks for all new code.

### Prerequisites

- JavaScript functions (declarations, expressions, arrow functions).
- Basic understanding of JSX syntax.
- Familiarity with ES modules (`import`/`export`).
- A React development environment (Vite, Next.js, or Create React App).

### Related Programming Areas

- **React Hooks** — state and lifecycle management in function components.
- **Component composition** — building complex UIs from simple function components.
- **Props and state** — the two primary data sources for function components.
- **JSX** — the markup syntax returned by function components.

---

## 1. Function Components

### Definitions

**Core Definition:** A function component is a JavaScript function that returns JSX.

**Technical Definition:** A function component is defined as a JavaScript function that accepts a single `props` object argument and returns a React element. It can be declared with `function` syntax or as an arrow function. It must be named with a capital letter to be treated as a component by JSX.

**Beginner-Friendly Explanation:** A function component is a function that gives back some HTML-like code (JSX). React calls this function whenever it needs to figure out what to show on screen.

### Purposes

- To define reusable UI building blocks in the simplest possible way.
- To use React Hooks for state and lifecycle management.
- To avoid the verbosity and `this`-binding issues of class components.
- To enable better tree-shaking and smaller bundle sizes.
- To align with modern React best practices and future features.

### Syntax Rules and Structure

**General Syntax (Function Declaration):**

```jsx
function ComponentName(props) {
  return <JSX />;
}
```

**General Syntax (Arrow Function):**

```jsx
const ComponentName = (props) => {
  return <JSX />;
};
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `function ComponentName` or `const ComponentName` | The component definition; name must be PascalCase. |
| `(props)` | The component's inputs (optional). |
| `return <JSX />` | The component's output; must return a single root element or fragment. |

**Syntax Rules:**

1. Component names must start with a capital letter. Lowercase names are treated as HTML tags.
2. Must return a single root element, a fragment, `null`, or an array with keys.
3. Hooks must be called at the top level of the component, never inside loops or conditions.
4. Props are read-only and must not be modified.

**Constraints and Limitations:**

- Cannot use class lifecycle methods directly; use `useEffect` instead.
- Hooks cannot be called conditionally.
- Component functions must never be called directly as regular functions.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Function Component with Props

```jsx
// Step 1: Define the component
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Step 2: Use the component
function App() {
  return (
    <div>
      <Welcome name="Alice" />
      <Welcome name="Bob" />
    </div>
  );
}
```

**Expected Output:**

```html
<div>
  <h1>Hello, Alice!</h1>
  <h1>Hello, Bob!</h1>
</div>
```

**Why this output:** The `Welcome` component is called twice with different `name` props. React passes `{ name: 'Alice' }` and `{ name: 'Bob' }` as the props object to each instance.

#### Example 2: Function Component with Hooks

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**Expected Output:** A counter that increments by 1 on each button click.

**Why this output:** `useState` adds state to the function component. Clicking the button updates state, triggering a re-render with the new count.

### Real-World Cases

- **UI primitives:** Buttons, inputs, cards, and modals are almost always function components.
- **Layouts:** Page layouts, headers, and sidebars are built as function components.
- **Data-driven components:** Components that fetch and display data use `useEffect` inside function components.
- **Design systems:** Modern component libraries (Radix UI, shadcn/ui) use function components exclusively.

### References

- React — Your First Component — https://react.dev/learn/your-first-component
- React — Components and Props — https://legacy.reactjs.org/docs/components-and-props.html

---

## 2. Arrow Function Components

### Definitions

**Core Definition:** An arrow function component is a function component defined using JavaScript arrow function syntax.

**Technical Definition:** Arrow function components use the `const ComponentName = (props) => { ... }` syntax. They offer more concise syntax, implicit return for single-expression components, and lexical `this` binding (though `this` is irrelevant in function components). The arrow function syntax is preferred in modern React codebases for its conciseness.

**Beginner-Friendly Explanation:** An arrow function component is just a shorter way to write a function component. Instead of `function MyComponent() {}`, you write `const MyComponent = () => {}`.

### Purposes

- To write components with more concise syntax.
- To use implicit return for simple, single-expression components.
- To align with modern JavaScript conventions.
- To enable cleaner integration with TypeScript typing.

### Syntax Rules and Structure

**General Syntax (Block Body):**

```jsx
const ComponentName = (props) => {
  return <JSX />;
};
```

**General Syntax (Implicit Return):**

```jsx
const ComponentName = (props) => <JSX />;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `const ComponentName` | The component definition; PascalCase required. |
| `= (props) =>` | Arrow function syntax with optional props parameter. |
| `<JSX />` or `{ return <JSX /> }` | The component's output. |

**Syntax Rules:**

1. Implicit return works only for single expressions; for multi-line JSX, wrap in parentheses.
2. Arrow functions do not have their own `this`, but this is irrelevant in function components.
3. The component name must still be PascalCase.

**Constraints and Limitations:**

- Implicit return cannot contain statements (only expressions).
- Multi-line JSX with implicit return requires parentheses for readability.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Arrow Function with Explicit Return

```jsx
const Greeting = ({ name, age }) => {
  const message = `Hello, ${name}!`;
  return (
    <div>
      <h2>{message}</h2>
      <p>You are {age} years old.</p>
    </div>
  );
};

// Usage
<Greeting name="Alice" age={30} />
```

**Expected Output:**

```html
<div>
  <h2>Hello, Alice!</h2>
  <p>You are 30 years old.</p>
</div>
```

**Why this output:** The arrow function component destructures `name` and `age` from props and renders them.

#### Example 2: Arrow Function with Implicit Return

```jsx
const Button = ({ label, onClick }) => (
  <button className="btn" onClick={onClick}>
    {label}
  </button>
);

// Usage
<Button label="Submit" onClick={() => alert('Clicked!')} />
```

**Expected Output:**

```html
<button class="btn">Submit</button>
```

**Why this output:** The arrow function uses implicit return (parentheses around JSX) to return a single `<button>` element without a `return` keyword.

### Real-World Cases

- **Design system primitives:** Small, single-purpose components like buttons and icons.
- **TypeScript components:** Arrow functions work seamlessly with `React.FC<Props>` typing.
- **Modern codebases:** Most new React projects use arrow function components by default.

### References

- CoreUI — How to create a functional component in React — https://coreui.io/answers/how-to-create-a-functional-component-in-react/
- React — Responding to Events — https://react.dev/learn/responding-to-events

---

## 3. Component Definition

### Definitions

**Core Definition:** Component definition is the act of declaring a function component in JavaScript.

**Technical Definition:** Defining a function component means writing a JavaScript function (declaration or arrow function) that returns React elements. The definition includes the component's name, its props parameter, and its return statement. The component must be defined before it is used in JSX.

**Beginner-Friendly Explanation:** Defining a component means writing the function that describes what the component should look like.

### Purposes

- To create a reusable UI building block.
- To encapsulate rendering logic in a named entity.
- To enable the component to be imported and used elsewhere.

### Syntax Rules and Structure

**Definition Patterns:**

| Pattern | Syntax |
|---------|--------|
| Function declaration | `function Name(props) { return <JSX />; }` |
| Function expression | `const Name = function(props) { return <JSX />; };` |
| Arrow function (block) | `const Name = (props) => { return <JSX />; };` |
| Arrow function (implicit) | `const Name = (props) => <JSX />;` |

**Syntax Rules:**

1. The component name must be PascalCase.
2. The function must return a React element, fragment, `null`, or array.
3. Props are optional; a component can be defined without parameters.

### Annotated Code Examples

```jsx
// Definition: Function declaration
function UserCard({ user }) {
  return (
    <div className="card">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Definition: Arrow function (implicit return)
const Badge = ({ text, color }) => (
  <span className={`badge badge-${color}`}>{text}</span>
);
```

**Expected Output:** A user card and a colored badge.

**Why this output:** Each definition creates a component that can be used in JSX.

### Real-World Cases

- **Component libraries:** Defining reusable components for a design system.
- **Page components:** Defining the top-level component for each route.
- **Utility components:** Defining small helpers like badges and labels.

### References

- React — Your First Component — https://react.dev/learn/your-first-component

---

## 4. Component Invocation

### Definitions

**Core Definition:** Component invocation is the act of using a component in JSX by writing it as a tag.

**Technical Definition:** Components are invoked by writing them as JSX tags: `<ComponentName />`. React then calls the component function during rendering, passing the JSX attributes as props. Component functions must **never** be called directly as regular functions (e.g., `ComponentName()`) because this bypasses React's rendering, reconciliation, and Hook orchestration.

**Beginner-Friendly Explanation:** To use a component, you write it like an HTML tag: `<MyComponent />`. You do not call it like a function.

### Purposes

- To render a component instance within another component or the app root.
- To pass props to a component.
- To let React manage the component's lifecycle and re-renders.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<ComponentName prop1={value1} prop2={value2} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<ComponentName` | The component tag; PascalCase name. |
| `prop1={value1}` | Props passed to the component. |
| `/>` | Self-closing tag (or `>children</ComponentName>`). |

**Syntax Rules:**

1. Always use JSX syntax (`<Component />`), never direct function calls (`Component()`). 
2. Props are passed as JSX attributes.
3. Children are placed between opening and closing tags.

**Constraints and Limitations:**

- Direct function calls bypass React's reconciliation and Hook rules.
- Calling a component directly inside a loop or condition can violate the Rules of Hooks.

### Annotated Code Examples

```jsx
function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}

function App() {
  return (
    <div>
      {/* ✅ Correct: JSX invocation */}
      <Button label="Submit" onClick={() => alert('Submitted')} />

      {/* ❌ Wrong: Direct function call */}
      {/* {Button({ label: 'Cancel' })} */}
    </div>
  );
}
```

**Expected Output:** A submit button is rendered.

**Why this output:** The `<Button />` JSX tag tells React to call the `Button` function during rendering. The direct call `Button()` would bypass React's rendering pipeline.

### Real-World Cases

- **Parent-child composition:** Parents invoke child components in their JSX.
- **Conditional rendering:** Components are invoked inside ternary expressions or `&&` conditions.
- **List rendering:** Components are invoked inside `.map()` callbacks.

### References

- React — React calls Components and Hooks — https://react.dev/reference/rules/react-calls-components-and-hooks

---

## 5. Component Return Values

### Definitions

**Core Definition:** A component's return value is the React element (or other valid React node) that the component function produces.

**Technical Definition:** A function component can return: a single React element (JSX), a fragment, `null`, `undefined` (treated like `null`), a boolean, a string, a number, an array of React elements (each with a unique key), or a portal. The return value describes what React should render to the DOM.

**Beginner-Friendly Explanation:** Whatever a component returns is what React shows on screen. You can return one thing, multiple things in a fragment, or nothing at all.

### Purposes

- To describe the UI that the component produces.
- To conditionally return different UIs based on props or state.
- To return nothing (`null`) when the component should render nothing.

### Syntax Rules and Structure

**Valid Return Types:**

| Type | Renders |
|------|---------|
| JSX element | The element |
| Fragment (`<>...</>`) | Children without wrapper |
| `null` | Nothing |
| `undefined` | Nothing (React 18+; warning in older versions) |
| `false` / `true` | Nothing |
| String / Number | Text node |
| Array (with keys) | Multiple elements |
| Portal | Rendered elsewhere in the DOM |

**Syntax Rules:**

1. A component must return a single value (but that value can be a fragment or array).
2. `null` is the recommended way to render nothing.
3. Arrays must have a `key` prop on each element.

### Annotated Code Examples

```jsx
// Returns a single element
function Single() {
  return <h1>Hello</h1>;
}

// Returns a fragment (multiple elements)
function Multiple() {
  return (
    <>
      <h1>Hello</h1>
      <p>World</p>
    </>
  );
}

// Returns null (renders nothing)
function Nothing({ show }) {
  if (!show) return null;
  return <p>Visible</p>;
}
```

**Expected Output:** `Single` renders an `<h1>`. `Multiple` renders an `<h1>` and a `<p>`. `Nothing` renders nothing when `show` is false.

**Why this output:** React handles each return type appropriately: elements render to DOM, fragments group without a wrapper, and `null` produces no output.

### Real-World Cases

- **Conditional rendering:** Returning `null` to hide components.
- **Fragment returns:** Returning multiple sibling elements from a component.
- **Array returns:** Returning lists of elements from a component.

### References

- React — Returning `null` — https://react.dev/learn/conditional-rendering#conditionally-returning-nothing-with-null
- React Blog — React v16.0 — https://legacy.reactjs.org/blog/2017/09/26/react-v16.0.html

---

## 6. Returning `null`

### Definitions

**Core Definition:** Returning `null` from a component renders nothing to the DOM while keeping the component mounted.

**Technical Definition:** When a function component returns `null`, React renders nothing but does not unmount the component. The component's Hooks continue to run, and its position in the component tree is preserved. This is the recommended way to conditionally render nothing.

**Beginner-Friendly Explanation:** If you want a component to show nothing, return `null`. It is like saying "render nothing here."

### Purposes

- To conditionally hide a component without unmounting it.
- To render nothing for permission gates or feature toggles.
- To handle empty data states.
- To maintain component lifecycle and state while invisible.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Component({ show }) {
  if (!show) return null;
  return <div>Content</div>;
}
```

**Syntax Rules:**

1. `return null;` renders nothing.
2. The component remains mounted; its state and effects are preserved.
3. `undefined` behaves like `null` in React 18+, but produces a warning in older versions.

**Constraints and Limitations:**

- Returning `null` keeps the component mounted, so `useEffect` cleanup does not run.
- If you want to fully unmount, use conditional rendering (`{show && <Component />}`) instead.

### Annotated Code Examples

```jsx
function PermissionGate({ userRole, requiredRole, children }) {
  const hasPermission = userRole === requiredRole;

  if (!hasPermission) {
    return null;  // Render nothing
  }

  return <>{children}</>;
}

// Usage
<PermissionGate userRole="viewer" requiredRole="admin">
  <AdminPanel />
</PermissionGate>
```

**Expected Output:** Nothing is rendered for a viewer (no `AdminPanel`).

**Why this output:** `PermissionGate` returns `null` when the user does not have the required role, so `AdminPanel` is never rendered.

### Real-World Cases

- **Permission gates:** Hiding UI for users without permission.
- **Feature toggles:** Hiding components behind feature flags.
- **Loading states:** Returning `null` while data is loading (if a parent handles the spinner).
- **Empty states:** Returning `null` when a list is empty.

### References

- CoreUI — How to Render Null in React — https://coreui.io/answers/how-to-render-null-in-react/
- React — Conditionally Returning Nothing with `null` — https://react.dev/learn/conditional-rendering

---

## 7. Returning Fragments

### Definitions

**Core Definition:** A fragment is a special React component that groups multiple elements without adding an extra node to the DOM.

**Technical Definition:** Fragments are declared with `<React.Fragment>` or the shorthand `<>...</>`. They allow a component to return multiple sibling elements without wrapping them in a parent `<div>`. The shorthand syntax does not support `key` or other attributes; the explicit `<React.Fragment>` syntax supports the `key` prop.

**Beginner-Friendly Explanation:** A fragment lets you return multiple elements from a component without adding an unnecessary wrapper `<div>` around them.

### Purposes

- To return multiple elements from a component without extra DOM nodes.
- To avoid breaking CSS layouts that depend on parent-child structure.
- To enable keyed fragments for rendering lists of groups.
- To reduce DOM bloat.

### Syntax Rules and Structure

**General Syntax (Shorthand):**

```jsx
<>
  <ChildA />
  <ChildB />
</>
```

**General Syntax (Explicit):**

```jsx
<React.Fragment key={id}>
  <ChildA />
  <ChildB />
</React.Fragment>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<>` or `<React.Fragment>` | Opening fragment tag. |
| `</>` or `</React.Fragment>` | Closing fragment tag. |
| `key={id}` | Only allowed on `<React.Fragment>`. |

**Syntax Rules:**

1. The shorthand `<>...</>` cannot accept any attributes, including `key`.
2. To pass a `key`, use the explicit `<React.Fragment key={...}>` syntax.
3. Fragments produce no DOM node.

**Constraints and Limitations:**

- Shorthand fragments cannot be keyed.
- Some older tooling may not support the shorthand syntax; use `<React.Fragment>` for maximum compatibility.

### Annotated Code Examples

#### Example 1: Basic Fragment

```jsx
function Columns() {
  return (
    <>
      <td>Hello</td>
      <td>World</td>
    </>
  );
}
```

**Expected Output (inside a `<table>`):**

```html
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```

**Why this output:** The fragment groups the two `<td>` elements without adding a `<div>` between `<tr>` and `<td>`, keeping the HTML valid.

#### Example 2: Keyed Fragment in a List

```jsx
function Glossary({ items }) {
  return (
    <dl>
      {items.map((item) => (
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.definition}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

**Expected Output:**

```html
<dl>
  <dt>React</dt>
  <dd>A JavaScript library for building UIs.</dd>
  <dt>JSX</dt>
  <dd>A syntax extension for JavaScript.</dd>
</dl>
```

**Why this output:** Each `<React.Fragment key={...}>` groups a `<dt>` and `<dd>` pair. The `key` helps React identify items in the list.

### Real-World Cases

- **Table rows:** Returning multiple `<td>` elements without a wrapper.
- **Definition lists:** Grouping `<dt>` and `<dd>` pairs.
- **Flexbox layouts:** Returning flex items without an extra wrapper div.

### References

- React — `<Fragment>` — https://react.dev/reference/react/Fragment
- React Blog — React v16.2.0: Improved Support for Fragments — https://legacy.reactjs.org/blog/2017/11/28/react-v16.2.0-fragment-support.html

---

## 8. Returning Arrays

### Definitions

**Core Definition:** A component can return an array of React elements, each with a unique `key` prop.

**Technical Definition:** Since React 16, function components can return an array of elements. Each element in the array must have a unique `key` prop to help React's reconciliation algorithm identify which items have changed. The array is rendered in order, and no wrapper element is added.

**Beginner-Friendly Explanation:** Instead of returning one element, you can return an array of elements. React renders them all, but you must give each one a unique `key` so React can track them.

### Purposes

- To return multiple elements without a wrapper element.
- To render lists where the array itself is the component's return value.
- To avoid extra DOM nodes (similar to fragments but with explicit keying).

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Component({ items }) {
  return items.map((item) => (
    <Element key={item.id} />
  ));
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `items.map(...)` | Returns an array of React elements. |
| `key={item.id}` | Required unique identifier for each element. |

**Syntax Rules:**

1. Every element in the array must have a unique `key` prop.
2. The array can contain elements, strings, numbers, or `null`/`false`.
3. React renders array elements in order.

**Constraints and Limitations:**

- Missing keys produce a React warning and may cause incorrect rendering.
- Arrays are less readable than fragments in JSX; prefer fragments when keys are not needed.

### Annotated Code Examples

```jsx
function TodoList({ todos }) {
  return todos.map((todo) => (
    <li key={todo.id}>
      {todo.text}
    </li>
  ));
}

// Usage
<TodoList todos={[
  { id: 1, text: 'Learn React' },
  { id: 2, text: 'Build an app' },
]} />
```

**Expected Output:**

```html
<li>Learn React</li>
<li>Build an app</li>
```

**Why this output:** The component returns an array of `<li>` elements, each with a unique `key`. React renders them as siblings without a wrapper.

### Real-World Cases

- **List components:** Components whose sole purpose is to render a list of items.
- **Performance-sensitive lists:** When avoiding an extra wrapper DOM node is critical.
- **Dynamic content:** When the number of elements varies based on data.

### References

- React Blog — React v16.2.0: Improved Support for Fragments — https://legacy.reactjs.org/blog/2017/11/28/react-v16.2.0-fragment-support.html
- React — Rendering Lists — https://react.dev/learn/rendering-lists

---

## 9. Component Naming Conventions

### Definitions

**Core Definition:** Component naming conventions are the rules for naming React components and their files.

**Technical Definition:** React components must use PascalCase (e.g., `UserProfile`, `SearchBar`) so that JSX distinguishes them from HTML tags. Component files should match the component name (e.g., `UserProfile.jsx`). Props use camelCase, event handlers use the `on`/`handle` prefixes, and hooks use the `use` prefix.

**Beginner-Friendly Explanation:** Always name your components with a capital letter at the start of each word. This tells React that it is a component, not a regular HTML tag.

### Purposes

- To distinguish components from HTML tags in JSX.
- To make code readable and predictable.
- To follow community standards.
- To avoid naming conflicts.

### Syntax Rules and Structure

**Naming Conventions Table:**

| Element | Convention | Example |
|---------|-----------|---------|
| Component name | PascalCase | `UserProfile` |
| Component file | PascalCase | `UserProfile.jsx` |
| Props | camelCase | `userId`, `isAdmin` |
| Boolean props | `is`/`has`/`should` prefix | `isDisabled`, `hasError` |
| Event handler prop | `on` prefix | `onClick`, `onSubmit` |
| Event handler function | `handle` prefix | `handleClick` |
| Custom hook | `use` prefix | `useAuth` |
| HOC | `with` prefix | `withAuth` |

**Syntax Rules:**

1. Components must use PascalCase.
2. Files should match the component name.
3. Hooks must start with `use`.

**Constraints and Limitations:**

- Lowercase component names are treated as HTML tags.
- Inconsistent naming reduces readability.

### Annotated Code Examples

```jsx
// ✅ Good: PascalCase component
function UserProfile({ userId, isAdmin }) {
  function handleClick() {
    console.log('Clicked');
  }
  return <button onClick={handleClick}>Edit</button>;
}

// ❌ Bad: lowercase component (treated as HTML tag)
function userProfile() {
  return <div>Not a component</div>;
}
```

**Expected Output:** `UserProfile` renders a button. `userProfile` is treated as an unknown HTML tag.

**Why this output:** React uses capitalization to distinguish components from DOM tags.

### Real-World Cases

- **Team collaboration:** Consistent naming makes code reviews easier.
- **Design systems:** Component names become part of the public API.
- **Testing:** Test files are named after components.

### References

- React — Naming Conventions — https://react.dev/learn/your-first-component#naming-conventions
- Eufemia — Naming Convention — https://eufemia.dnb.no/

---

## 10. Component File Organization

### Definitions

**Core Definition:** Component file organization is the practice of structuring files and folders for React components in a maintainable way.

**Technical Definition:** React does not enforce a specific file structure, but common approaches include grouping by feature/route (co-locating CSS, JS, and tests) or grouping by file type. Each component typically lives in its own folder with its component file, tests, styles, and an index file for clean imports.

**Beginner-Friendly Explanation:** How you organize your files is up to you, but most React projects put each component in its own folder with related files.

### Purposes

- To keep related files close together (co-location).
- To make it easy to find and modify components.
- To scale the project structure as the codebase grows.
- To enable clean imports via index files.

### Syntax Rules and Structure

**Common Structures:**

**Grouping by Feature/Route:**

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.module.css
│   │   └── index.ts
│   └── UserProfile/
│       ├── UserProfile.tsx
│       └── index.ts
├── hooks/
├── utils/
└── App.tsx
```

**Component Folder Structure:**

```
Button/
├── Button.tsx          # Component implementation
├── Button.test.tsx     # Tests
├── Button.stories.tsx  # Storybook
├── Button.module.css   # Scoped styles
└── index.ts            # Re-export
```

**Syntax Rules:**

1. Each component should have its own folder.
2. Co-locate tests, styles, and stories with the component.
3. Use an `index.ts` file to re-export the component for clean imports.

**Constraints and Limitations:**

- Avoid deeply nested folder structures (limit to 3–4 levels).
- Do not over-think folder structure at the start of a project.

### Annotated Code Examples

**File: `src/components/Button/Button.tsx`**

```jsx
export function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}
```

**File: `src/components/Button/index.ts`**

```ts
export { Button } from './Button';
```

**File: `src/App.tsx`**

```jsx
import { Button } from './components/Button';

function App() {
  return <Button label="Submit" onClick={() => alert('Clicked')} />;
}
```

**Expected Output:** A submit button is rendered.

**Why this output:** The `index.ts` re-exports the `Button` component, allowing clean imports without specifying the full file path.

### Real-World Cases

- **Large applications:** Feature-based folders keep related code together.
- **Design systems:** Each component has its own folder with tests and stories.
- **Monorepos:** Components are organized into packages with their own folder structures.

### References

- React — File Structure FAQ — https://legacy.reactjs.org/docs/faq-structure.html
- RT Camp — Organizing React Projects for Scalability — https://rtcamp.com/handbook/react-best-practices/project-structure/

---

## References

- React — Your First Component — https://react.dev/learn/your-first-component
- React — Components and Props — https://legacy.reactjs.org/docs/components-and-props.html
- React — React calls Components and Hooks — https://react.dev/reference/rules/react-calls-components-and-hooks
- React — `<Fragment>` — https://react.dev/reference/react/Fragment
- React — Rendering Lists — https://react.dev/learn/rendering-lists
- React — Conditionally Returning Nothing with `null` — https://react.dev/learn/conditional-rendering
- React Blog — React v16.0 — https://legacy.reactjs.org/blog/2017/09/26/react-v16.0.html
- React Blog — React v16.2.0: Improved Support for Fragments — https://legacy.reactjs.org/blog/2017/11/28/react-v16.2.0-fragment-support.html
- React — File Structure FAQ — https://legacy.reactjs.org/docs/faq-structure.html
- CoreUI — How to create a functional component in React — https://coreui.io/answers/how-to-create-a-functional-component-in-react/
- CoreUI — How to Render Null in React — https://coreui.io/answers/how-to-render-null-in-react/
- RT Camp — Organizing React Projects for Scalability — https://rtcamp.com/handbook/react-best-practices/project-structure/
- Telerik — React Design Patterns and Best Practices for 2025 — https://www.telerik.com/blogs/react-design-patterns-best-practices
- Eufemia — Naming Convention — https://eufemia.dnb.no/