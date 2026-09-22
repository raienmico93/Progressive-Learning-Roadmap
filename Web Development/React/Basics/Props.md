# React Props — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

Props (short for "properties") are the mechanism by which React components communicate with one another. A parent component passes information to a child component by providing props, which the child component reads as read-only inputs that influence its rendered output.

**Technical Definition**

In React's component model, props are plain JavaScript objects that are passed as the first argument to a functional component or accessed via `this.props` in a class component. When React encounters a user-defined component in JSX, it passes the JSX attributes as a single object to that component, and the component uses this object to determine its rendering logic and output. Props are immutable from the perspective of the receiving component; they flow unidirectionally from parent to child and cannot be modified by the child.

**Beginner-Friendly Explanation**

Think of props like the settings or arguments you give to a function. When you use a component in React, you can pass it information—like a name, a number, or even another piece of UI. The component then uses that information to decide what to display. Props let you customize a component without rewriting its code, making components reusable and predictable.

### Key Characteristics

- **Unidirectional Data Flow**: Props flow exclusively from parent components to child components. A child component cannot modify the props it receives.
- **Immutable**: Props are read-only. Modifying props directly violates React's core principles and produces unpredictable behavior.
- **Arbitrary JavaScript Values**: Props can carry strings, numbers, booleans, objects, arrays, functions, and even JSX elements.
- **Component Communication**: Props are the primary means by which components share data and coordinate behavior.
- **Resemblance to HTML Attributes**: The JSX syntax for passing props mirrors HTML attributes, making the mental model intuitive for web developers.

### Prerequisites

Before studying React Props, learners should be comfortable with:

- **JavaScript Fundamentals**: Variables, functions, objects, arrays, destructuring, and the spread operator.
- **ES6+ Syntax**: Arrow functions, template literals, and module imports/exports.
- **JSX**: The XML-like syntax extension used in React to describe UI structure.
- **React Components**: Functional components and the concept of composing components into hierarchies.
- **Rendering Concepts**: How React renders elements and updates the DOM.

### Related Programming Areas

- **React State Management**: Complements props by handling data that changes within a component over time.
- **Component Composition**: The architectural pattern of building complex UIs from smaller, focused components connected via props.
- **Type Systems (TypeScript/PropTypes)**: Tools for validating and documenting the shape of props.
- **Unidirectional Data Flow Architectures**: Patterns like Flux and Redux that extend React's one-way data flow to application state.

### Core Concepts / Features

- Props Definition
- Passing Data
- Reading Props
- Default Values
- Destructured Props
- Boolean Props
- Function Props
- Object and Array Props
- Children Prop

---

## 1. Props Definition

### Definitions

**Core Definition**

Props are the inputs that a React component receives from its parent. They are defined at the point of JSX usage and consumed inside the component to customize its behavior and output.

**Technical Definition**

When React processes a JSX element that represents a user-defined component (e.g., `<Avatar name="Sara" />`), it collects all attributes of that element into a single JavaScript object and passes this object as the first argument to the component function. This object is conventionally named `props`. In class components, the same object is accessible as `this.props`.

**Beginner-Friendly Explanation**

Props are like the order you place at a restaurant. The menu (the component) stays the same, but what you order (the props) changes what you receive. The kitchen (the component's code) uses your order to prepare your specific dish.

### Purposes

- To enable communication between parent and child components.
- To make components configurable without modifying their internal code.
- To support component reusability across different contexts.
- To establish a clear, predictable contract for how components receive data.

### Syntax Rules and Structure

**Complete General Syntax (Function Component)**

```jsx
function ComponentName(props) {
  // Access props: props.propertyName
  return <div>{props.propertyName}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function ComponentName` | The component function declaration. Must begin with an uppercase letter to distinguish it from HTML tags. |
| `props` | The single argument containing all passed properties. |
| `props.propertyName` | Accessing an individual property from the props object. |

**Complete General Syntax (Class Component)**

```jsx
class ComponentName extends React.Component {
  render() {
    return <div>{this.props.propertyName}</div>;
  }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class ComponentName extends React.Component` | Class component declaration. |
| `this.props` | React automatically assigns the props object to the instance. |
| `render()` | Required method that returns the JSX using `this.props`. |

**Syntax Rules**

1. Component names must start with a capital letter; React treats lowercase names as DOM tags.
2. Props are passed as attributes in JSX: `<Component propName={value} />`.
3. String literals can use quotes (`name="Sara"`), while all other values use curly braces (`size={100}`).
4. Props are read-only; assigning to `props.x` throws an error in strict mode and violates React's contract.

**Constraints and Limitations**

- Props cannot be modified by the receiving component.
- The `props` object is the **only** argument a functional component receives (along with optional `context` in newer APIs, though this is separate from props).
- Changes to props trigger re-renders; React compares prop values to determine updates.

### Annotated Complete Code Examples

**Example 1: Basic Props Definition (Function Component)**

```jsx
// Define a Welcome component that accepts props
function Welcome(props) {
  // props is an object: { name: "Sara" }
  return <h1>Hello, {props.name}!</h1>;
}

// Parent component renders Welcome with a name prop
export default function App() {
  return (
    <div>
      {/* Passing the string "Sara" as the name prop */}
      <Welcome name="Sara" />
    </div>
  );
}
```

**Expected Output**

```
Hello, Sara!
```

**Why This Output Occurs**

React calls `Welcome` with the object `{ name: "Sara" }` as `props`. Inside the component, `props.name` evaluates to `"Sara"`, which is interpolated into the JSX. React then renders the `<h1>` element containing the text "Hello, Sara!".

**Example 2: Class Component with Props**

```jsx
import React from 'react';

// Class component definition
class Greeting extends React.Component {
  render() {
    // Access props via this.props
    return <p>Welcome, {this.props.user}!</p>;
  }
}

export default function App() {
  return (
    <div>
      <Greeting user="Alice" />
    </div>
  );
}
```

**Expected Output**

```
Welcome, Alice!
```

**Why This Output Occurs**

React instantiates the `Greeting` class and assigns the props object `{ user: "Alice" }` to `this.props`. The `render()` method references `this.props.user`, which evaluates to `"Alice"`. React renders the resulting `<p>` element.

### Real-World Cases

**Case: A Social Media Profile Card**

A `ProfileCard` component receives props for `username`, `avatarUrl`, `bio`, and `isOnline`. The same component renders differently for each user by changing the prop values. This allows the application to maintain a single card design while displaying unlimited user profiles.

**Case: An E-Commerce Product Tile**

A `ProductTile` component receives `product` (an object), `onAddToCart` (a function), and `currency` (a string). The product tile displays information and triggers the parent's cart logic when clicked, all through props.

### References

- React — Passing Props to a Component - https://react.dev/learn/passing-props-to-a-component
- React — Components and Props (Legacy) - https://legacy.reactjs.org/docs/components-and-props.html

---

## 2. Passing Data

### Definitions

**Core Definition**

Passing data refers to the act of providing values to a component through JSX attributes at the point where the component is used.

**Technical Definition**

Data is passed by adding attributes to a JSX element. React collects these attributes into a props object and delivers it to the component. String literals are passed using quotation marks; all other JavaScript expressions are passed using curly braces `{}`.

**Beginner-Friendly Explanation**

Passing data is like filling out a form before handing it to someone. You write down the information (data) in the fields (attributes), and the person (component) receives the completed form (props object).

### Purposes

- To provide components with the specific data they need to render.
- To customize component behavior from the outside without modifying internal code.
- To connect parent logic with child presentation.
- To enable dynamic rendering based on changing data.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<ComponentName
  stringProp="literal string"
  numberProp={42}
  booleanProp={true}
  objectProp={{ key: "value" }}
  arrayProp={[1, 2, 3]}
  expressionProp={variableName}
/>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `stringProp="literal string"` | String literals use double quotes; no braces needed. |
| `numberProp={42}` | Numbers, booleans, objects, arrays, and expressions use curly braces. |
| `objectProp={{ key: "value" }}` | Double braces: outer braces enter JSX expression mode, inner braces create a JavaScript object literal. |
| `arrayProp={[1, 2, 3]}` | Arrays are passed as expressions. |
| `expressionProp={variableName}` | Any JavaScript expression can be evaluated and passed. |

**Syntax Rules**

1. **Strings**: Use `propName="text"`. No curly braces.
2. **All Other Values**: Use `propName={expression}`. The expression can be a literal, variable, function call, or object/array literal.
3. **Objects**: Require double curly braces: `{{ }}`. The outer pair signals a JSX expression; the inner pair creates the object.
4. **Functions**: Passed without calling them: `onClick={handleClick}`, not `onClick={handleClick()}`.
5. **Spread Operator**: `{...obj}` passes all properties of `obj` as individual props.

**Constraints and Limitations**

- A component cannot pass props to itself; only parents pass props to children.
- Prop names must be valid JavaScript identifiers when destructured.
- Reserved words like `class` should be written as `className` in JSX.

### Annotated Complete Code Examples

**Example 1: Passing Multiple Data Types**

```jsx
// Child component receives props of various types
function UserProfile({ name, age, isActive, hobbies, address }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Status: {isActive ? 'Active' : 'Inactive'}</p>
      <p>Hobbies: {hobbies.join(', ')}</p>
      <p>City: {address.city}</p>
    </div>
  );
}

// Parent passes different data types
export default function App() {
  // A variable to pass as a prop
  const userAge = 28;
  
  return (
    <UserProfile
      name="John Doe"                                    // String literal
      age={userAge}                                      // Number variable
      isActive={true}                                    // Boolean literal
      hobbies={['Reading', 'Cycling', 'Coding']}         // Array literal
      address={{ city: 'San Francisco', zip: '94105' }} // Object literal
    />
  );
}
```

**Expected Output**

```
John Doe
Age: 28
Status: Active
Hobbies: Reading, Cycling, Coding
City: San Francisco
```

**Why This Output Occurs**

Each prop is evaluated by JavaScript: `name` becomes the string `"John Doe"`, `age` resolves the variable `userAge` to `28`, `isActive` becomes boolean `true`, the array and object literals are evaluated and passed as references. The `UserProfile` component destructures these values and renders them in the corresponding JSX elements.

**Example 2: Passing Data with the Spread Operator**

```jsx
function Card({ title, description, price }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <p>{description}</p>
      <span>${price}</span>
    </div>
  );
}

export default function App() {
  // An object containing multiple props
  const cardData = {
    title: 'Wireless Headphones',
    description: 'Noise-cancelling, 30-hour battery',
    price: 199
  };
  
  return (
    <div>
      {/* Spread operator passes each key as a separate prop */}
      <Card {...cardData} />
    </div>
  );
}
```

**Expected Output**

```
Wireless Headphones
Noise-cancelling, 30-hour battery
$199
```

**Why This Output Occurs**

The spread operator `{...cardData}` expands the object into individual props: `title="Wireless Headphones"`, `description="Noise-cancelling, 30-hour battery"`, and `price={199}`. This is equivalent to writing each prop manually but is more concise when the object structure aligns with the component's prop interface.

### Real-World Cases

**Case: Passing User Data Through a Dashboard**

A `Dashboard` component fetches user data from an API and passes relevant slices to child components: `<StatsPanel metrics={data.metrics} />`, `<ActivityFeed events={data.recentActivity} />`, and `<ProfileWidget user={data.user} />`. Each child receives exactly the data it needs, demonstrating prop drilling in a real application.

**Case: Configuration Objects for Reusable Components**

A `DataTable` component accepts a `columns` array prop and a `data` array prop. Different pages pass different configurations to the same component, enabling a single table implementation to serve diverse data display needs.

### References

- React — Passing Props to a Component - https://react.dev/learn/passing-props-to-a-component
- React — Spread Attributes (Legacy) - https://legacy.reactjs.org/docs/jsx-in-depth.html#spread-attributes

---

## 3. Reading Props

### Definitions

**Core Definition**

Reading props is the process by which a component accesses the values that were passed to it by its parent.

**Technical Definition**

In a functional component, props are accessed either through the `props` object (e.g., `props.name`) or by destructuring the props object in the function signature (e.g., `{ name }`). In class components, props are accessed via `this.props`.

**Beginner-Friendly Explanation**

Reading props is like opening the envelope someone handed you. You take out the information inside and use it to do your job.

### Purposes

- To access the data provided by the parent component.
- To use passed values in the component's rendering logic.
- To avoid hardcoding values and enable dynamic content.
- To establish the component's interface contract.

### Syntax Rules and Structure

**Complete General Syntax (Object Access)**

```jsx
function Component(props) {
  return <div>{props.propName}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `props` | The parameter name holding the props object. |
| `props.propName` | Dot notation to access a specific property. |

**Complete General Syntax (Destructuring)**

```jsx
function Component({ propName, anotherProp }) {
  return <div>{propName} {anotherProp}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `{ propName, anotherProp }` | Destructuring pattern extracts named properties directly into local variables. |
| `propName` | Now accessible as a standalone variable. |

**Syntax Rules**

1. Props can be read at any point within the component body, including inside callbacks and effects.
2. Destructuring is the idiomatic modern approach and is preferred for readability.
3. In class components, always use `this.props`; never destructure in the class body (use `render()` or lifecycle methods instead).
4. Props are available immediately upon component invocation; there is no asynchronous delay.

**Constraints and Limitations**

- You cannot read a prop that was never passed; accessing an undefined prop returns `undefined`.
- Destructuring assumes the props object is not `undefined`; if a parent renders `<Component />` without any props, the `props` argument defaults to an empty object in React (not `undefined`), so destructuring still works safely.

### Annotated Complete Code Examples

**Example 1: Reading Props with Object Access**

```jsx
// Reading props via dot notation
function Book(props) {
  // Access each prop individually
  const title = props.title;
  const author = props.author;
  const year = props.year;
  
  return (
    <div>
      <h3>{title}</h3>
      <p>by {author} ({year})</p>
    </div>
  );
}

export default function App() {
  return <Book title="The Great Gatsby" author="F. Scott Fitzgerald" year={1925} />;
}
```

**Expected Output**

```
The Great Gatsby
by F. Scott Fitzgerald (1925)
```

**Why This Output Occurs**

The `Book` component receives the props object `{ title: "The Great Gatsby", author: "F. Scott Fitzgerald", year: 1925 }`. Each property is read via dot notation and assigned to a local variable. These variables are then interpolated into the JSX output.

**Example 2: Reading Props with Destructuring**

```jsx
// Destructuring in the function signature
function Movie({ title, director, rating }) {
  // title, director, rating are now local variables
  return (
    <div>
      <h4>{title}</h4>
      <p>Directed by {director}</p>
      <p>Rating: {rating}/10</p>
    </div>
  );
}

export default function App() {
  return <Movie title="Inception" director="Christopher Nolan" rating={8.8} />;
}
```

**Expected Output**

```
Inception
Directed by Christopher Nolan
Rating: 8.8/10
```

**Why This Output Occurs**

The destructuring syntax `{ title, director, rating }` extracts the corresponding properties from the props object and creates local variables with the same names. This eliminates the need for `props.` prefixes and makes the JSX more readable.

### Real-World Cases

**Case: Reading Configuration in a UI Component**

A `Button` component reads `variant`, `size`, and `disabled` props to determine its CSS classes and HTML attributes. The component reads these props at the top of the function, applies conditional logic, and renders the appropriate `<button>` element.

**Case: Reading Nested Data in a Profile Component**

A `Profile` component receives a `user` object prop and reads `user.name`, `user.email`, and `user.avatarUrl`. It may also destructure at multiple levels: `const { name, email, avatarUrl } = user;` after receiving the `user` prop.

### References

- React — Passing Props to a Component (Reading Props section) - https://react.dev/learn/passing-props-to-a-component#step-2-read-props-inside-the-child-component
- MDN — Destructuring Assignment - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment

---

## 4. Default Values

### Definitions

**Core Definition**

Default values are fallback values assigned to props when the parent does not provide a value (or provides `undefined`).

**Technical Definition**

In functional components, default values are specified using JavaScript's default parameter syntax within the destructuring pattern: `{ propName = defaultValue }`. In class components, they are defined via the static `defaultProps` property.

**Beginner-Friendly Explanation**

Default values are like the "default settings" on a new phone. If you don't change anything, you get the standard configuration. If you do provide a setting, it overrides the default.

### Purposes

- To prevent components from breaking when optional props are omitted.
- To reduce boilerplate in parent components by not requiring every prop.
- To provide sensible fallback behavior.
- To document which props are optional and what they fall back to.

### Syntax Rules and Structure

**Complete General Syntax (Function Component)**

```jsx
function Component({ propName = defaultValue }) {
  return <div>{propName}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `propName` | The prop being defaulted. |
| `= defaultValue` | JavaScript default parameter syntax; applies when the value is `undefined`. |

**Complete General Syntax (Class Component — Legacy)**

```jsx
class Component extends React.Component {
  static defaultProps = {
    propName: defaultValue
  };
  render() {
    return <div>{this.props.propName}</div>;
  }
}
```

**Syntax Rules**

1. Default values apply only when the prop is `undefined`, **not** when it is `null`.
2. Default parameters can reference other variables in scope.
3. Default values can be any valid JavaScript expression, including function calls (though this may impact performance if called on every render).
4. For class components, `defaultProps` is a static property; it is applied before the component renders.

**Constraints and Limitations**

- `defaultProps` on class components is a legacy pattern; modern React code favors function components with default parameters.
- Default parameters are evaluated on every render if not memoized, though for simple literals this is negligible.
- `defaultProps` does **not** apply to `null` values; explicitly passing `prop={null}` overrides the default.

### Annotated Complete Code Examples

**Example 1: Default Values with Destructuring**

```jsx
// Default values for optional props
function Button({ label = 'Click Me', color = 'blue', size = 'medium' }) {
  return (
    <button style={{ backgroundColor: color }} className={`btn-${size}`}>
      {label}
    </button>
  );
}

export default function App() {
  return (
    <div>
      {/* No props passed — all defaults apply */}
      <Button />
      
      {/* Only label passed — color and size use defaults */}
      <Button label="Submit" />
      
      {/* All props passed — no defaults used */}
      <Button label="Cancel" color="red" size="large" />
    </div>
  );
}
```

**Expected Output**

```
[Button with blue background, medium size, text "Click Me"]
[Button with blue background, medium size, text "Submit"]
[Button with red background, large size, text "Cancel"]
```

**Why This Output Occurs**

When `<Button />` is rendered without props, the destructuring defaults kick in: `label` becomes `"Click Me"`, `color` becomes `"blue"`, and `size` becomes `"medium"`. When `<Button label="Submit" />` is rendered, only `label` is provided, so `color` and `size` fall back to their defaults. The third button receives all three values explicitly, overriding every default.

**Example 2: Default Value Does Not Apply to Null**

```jsx
function Alert({ message = 'Default message', type = 'info' }) {
  return <div className={`alert-${type}`}>{message}</div>;
}

export default function App() {
  return (
    <div>
      <Alert />                                  {/* Uses default message */}
      <Alert message={null} />                  {/* null overrides default! */}
      <Alert message="Custom" />                {/* Explicit value */}
    </div>
  );
}
```

**Expected Output**

```
<div class="alert-info">Default message</div>
<div class="alert-info"></div>
<div class="alert-info">Custom</div>
```

**Why This Output Occurs**

The second `<Alert>` explicitly passes `message={null}`. JavaScript's default parameter only applies when the argument is `undefined`, not `null`. Therefore, the default `'Default message'` is **not** used, and the component renders an empty alert.

### Real-World Cases

**Case: A Theme Provider with Default Theme**

A `ThemeProvider` component accepts a `theme` prop that defaults to a standard light theme. Applications that don't specify a theme automatically receive the default, while applications needing a dark theme pass `theme="dark"`.

**Case: An Avatar Component with Placeholder**

An `Avatar` component defaults `src` to a placeholder image when no image URL is provided. This prevents broken image icons when user avatars are missing.

### References

- React — Specifying a Default Value for a Prop - https://react.dev/learn/passing-props-to-a-component#specifying-a-default-value-for-a-prop
- MDN — Default Parameters - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters

---

## 5. Destructured Props

### Definitions

**Core Definition**

Destructured props refers to the practice of extracting individual properties from the props object directly in the component's function signature.

**Technical Definition**

Using JavaScript's destructuring assignment syntax, a component can unpack values from the `props` object into distinct local variables, reducing repetition and improving readability.

**Beginner-Friendly Explanation**

Instead of saying "props dot name" every time, you unpack the props like opening a toolbox and laying out the tools you need on the table.

### Purposes

- To reduce repetitive `props.` prefixing in component code.
- To make component interfaces explicit and self-documenting.
- To enable default values and renaming in a single expression.
- To improve code readability and maintainability.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function Component({ prop1, prop2, prop3 = defaultValue }) {
  // Use prop1, prop2, prop3 directly
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `{ prop1, prop2 }` | Destructuring pattern matching property names to local variables. |
| `prop3 = defaultValue` | Default value applied during destructuring. |
| `{ prop1: localName }` | Renaming: extracts `prop1` into a variable named `localName`. |

**Syntax Rules**

1. The destructuring pattern must match the property names passed by the parent.
2. Default values are specified with `=` inside the pattern.
3. Nested destructuring is possible but often reduces readability: `{ user: { name } }`.
4. The entire props object can still be accessed by not destructuring or by using a rest pattern.

**Constraints and Limitations**

- Destructuring in the signature means the full `props` object is not directly available unless explicitly captured.
- If a prop is optional and not defaulted, the destructured variable will be `undefined`.
- Renaming is useful when a prop name conflicts with a local variable or reserved word.

### Annotated Complete Code Examples

**Example 1: Basic Destructuring**

```jsx
// Without destructuring — more verbose
function GreetingNoDestructure(props) {
  return <h1>Hello, {props.name}! You have {props.messages} messages.</h1>;
}

// With destructuring — cleaner
function Greeting({ name, messages }) {
  return <h1>Hello, {name}! You have {messages} messages.</h1>;
}

export default function App() {
  return <Greeting name="Emma" messages={5} />;
}
```

**Expected Output**

```
Hello, Emma! You have 5 messages.
```

**Why This Output Occurs**

Both versions produce the same output. In the destructured version, `{ name, messages }` extracts these properties from the props object and makes them available as local variables, eliminating the `props.` prefix and making the JSX more readable.

**Example 2: Destructuring with Renaming and Defaults**

```jsx
function ProductCard({
  title: productTitle,        // Rename title to productTitle
  price = 0,                  // Default price to 0
  inStock = false             // Default inStock to false
}) {
  return (
    <div>
      <h3>{productTitle}</h3>
      <p>Price: ${price}</p>
      <p>{inStock ? 'Available' : 'Out of Stock'}</p>
    </div>
  );
}

export default function App() {
  return (
    <div>
      {/* Only title provided; price and inStock use defaults */}
      <ProductCard title="Laptop" />
      
      {/* All provided; rename applies, defaults overridden */}
      <ProductCard title="Phone" price={699} inStock={true} />
    </div>
  );
}
```

**Expected Output**

```
Laptop
Price: $0
Out of Stock

Phone
Price: $699
Available
```

**Why This Output Occurs**

The first `ProductCard` receives only `title="Laptop"`. The destructuring renames it to `productTitle`, and `price` and `inStock` fall back to `0` and `false` respectively. The second card provides all props, so the rename applies and the defaults are overridden.

### Real-World Cases

**Case: A Form Input Component**

A `TextInput` component destructures `{ label, value, onChange, placeholder = '' }`. The concise signature makes the component's API immediately clear to any developer reading the code, and the default placeholder prevents `undefined` from appearing in the DOM.

**Case: A Layout Component**

A `PageLayout` component destructures `{ header, sidebar, children, footer }`. The destructuring makes it evident that this component expects four specific content regions, serving as living documentation.

### References

- React — Passing Props to a Component (Destructuring) - https://react.dev/learn/passing-props-to-a-component
- MDN — Object Destructuring - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#object_destructuring

---

## 6. Boolean Props

### Definitions

**Core Definition**

Boolean props are props whose values are either `true` or `false`, typically used to toggle features, states, or behaviors.

**Technical Definition**

In JSX, boolean props can be passed explicitly with `{true}` or `{false}`, or they can be passed using shorthand syntax where the mere presence of the prop implies `true`.

**Beginner-Friendly Explanation**

Boolean props are like light switches. You either turn them on (`true`) or off (`false`). If you just say "this switch exists," it's automatically on.

### Purposes

- To enable or disable component features conditionally.
- To toggle visual states like visibility, disabled status, or active state.
- To provide simple yes/no configuration without explicit values.
- To control behavior in a binary, unambiguous manner.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* Explicit boolean */}
<Component isActive={true} />
<Component isActive={false} />

{/* Shorthand — presence implies true */}
<Component isActive />

{/* Conditional expression */}
<Component isActive={someCondition} />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `isActive={true}` | Explicitly passes `true`. |
| `isActive` | Shorthand; equivalent to `isActive={true}`. |
| `isActive={false}` | Explicitly passes `false`. |
| `isActive={someCondition}` | Passes the result of a JavaScript expression. |

**Syntax Rules**

1. The shorthand `<Component propName />` is equivalent to `<Component propName={true} />`.
2. To pass `false`, you **must** use the explicit form: `<Component propName={false} />`.
3. Boolean props are commonly used for HTML attributes like `disabled`, `hidden`, and `readOnly`.

**Constraints and Limitations**

- When spreading props onto a DOM element, passing `false` for certain HTML attributes may produce warnings; some DOM attributes require the prop to be omitted entirely rather than set to `false`.
- Boolean props should be named with a clear prefix like `is`, `has`, or `should` for readability.

### Annotated Complete Code Examples

**Example 1: Boolean Props for Conditional Rendering**

```jsx
function Modal({ isOpen, title, children }) {
  // Early return if modal is not open
  if (!isOpen) {
    return null; // Render nothing
  }
  
  return (
    <div className="modal-overlay">
      <div className="modal">
        <h2>{title}</h2>
        {children}
      </div>
    </div>
  );
}

export default function App() {
  return (
    <div>
      {/* Modal will not render because isOpen is false */}
      <Modal isOpen={false} title="Hidden Modal">
        <p>You can't see me</p>
      </Modal>
      
      {/* Modal will render because isOpen is true */}
      <Modal isOpen={true} title="Visible Modal">
        <p>I am visible!</p>
      </Modal>
    </div>
  );
}
```

**Expected Output**

```
[Only the second modal is rendered]
Visible Modal
I am visible!
```

**Why This Output Occurs**

The first `Modal` receives `isOpen={false}`. Inside the component, `if (!isOpen)` evaluates to `true`, causing an early return of `null`. React renders nothing for this component. The second `Modal` receives `isOpen={true}`, so the condition fails and the full modal JSX is rendered.

**Example 2: Boolean Shorthand**

```jsx
function ToggleButton({ label, isDisabled }) {
  return (
    <button disabled={isDisabled}>
      {label}
    </button>
  );
}

export default function App() {
  return (
    <div>
      {/* Shorthand: isDisabled is true */}
      <ToggleButton label="Disabled Button" isDisabled />
      
      {/* Explicit false */}
      <ToggleButton label="Enabled Button" isDisabled={false} />
    </div>
  );
}
```

**Expected Output**

```
[Disabled button with text "Disabled Button"]
[Enabled button with text "Enabled Button"]
```

**Why This Output Occurs**

The first `ToggleButton` uses the shorthand `isDisabled`, which React interprets as `isDisabled={true}`. The `disabled` attribute is applied to the `<button>`, making it non-interactive. The second button explicitly passes `isDisabled={false}`, so the `disabled` attribute is omitted and the button is clickable.

### Real-World Cases

**Case: A Loading Spinner Component**

A `Spinner` component accepts an `isLoading` boolean prop. When `true`, it renders the spinner animation; when `false`, it renders nothing or the loaded content. Parents toggle this prop based on data-fetching state.

**Case: A Navigation Menu with Active States**

A `NavItem` component receives an `isActive` boolean prop. The active item receives different CSS styling. The parent determines which item is active based on the current route.

### References

- React — Passing Props to a Component (Boolean Props) - https://react.dev/learn/passing-props-to-a-component
- React DOM — Common Components (Boolean Attributes) - https://react.dev/reference/react-dom/components/common

---

## 7. Function Props

### Definitions

**Core Definition**

Function props are functions passed from a parent component to a child component as props, enabling the child to trigger behavior or send data back to the parent.

**Technical Definition**

In JSX, a function is passed as a prop using curly braces without invoking it: `onAction={handleAction}`. The child component receives this function as a value and can call it later, typically in response to an event.

**Beginner-Friendly Explanation**

Function props are like giving someone your phone number. You're not calling them right now; you're giving them the ability to call you later when they need something.

### Purposes

- To allow child components to communicate events back to parents.
- To enable customizable behavior from the parent without modifying child internals.
- To pass callbacks that execute when specific child events occur.
- To support the "lifting state up" pattern where children notify parents of changes.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* Parent passes a function */}
<Child onAction={handleAction} />

{/* Child receives and calls the function */}
function Child({ onAction }) {
  return <button onClick={onAction}>Click</button>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `onAction={handleAction}` | Passes a reference to the function (no parentheses). |
| `onAction` | Destructured prop containing the function. |
| `onClick={onAction}` | Passes the function to a DOM event handler. |
| `onAction()` | Calls the function when invoked inside the child. |

**Syntax Rules**

1. **Never call the function when passing it**: Use `onAction={handleAction}`, not `onAction={handleAction()}`.
2. To pass arguments, use an arrow function wrapper: `onClick={() => handleAction(id)}`.
3. Function props conventionally use names beginning with `on`, like `onClick`, `onSubmit`, `onChange`.
4. In class components, event handler methods may need to be bound to the instance, though arrow function class properties avoid this.

**Constraints and Limitations**

- Passing a function that creates a new reference on every render (e.g., inline arrow functions) can cause performance issues in memoized child components.
- Functions passed as props should be stable references when possible to avoid unnecessary re-renders.
- The child must call the function; merely passing it does not execute it.

### Annotated Complete Code Examples

**Example 1: Passing a Function to a Child**

```jsx
// Child component receives and invokes a function prop
function DeleteButton({ onDelete, itemId }) {
  return (
    <button onClick={() => onDelete(itemId)}>
      Delete Item {itemId}
    </button>
  );
}

// Parent component defines the handler
export default function App() {
  function handleDelete(id) {
    alert(`Deleting item with ID: ${id}`);
  }
  
  return (
    <div>
      <DeleteButton onDelete={handleDelete} itemId={42} />
    </div>
  );
}
```

**Expected Output**

```
[Button with text "Delete Item 42"]
When clicked: Alert box displays "Deleting item with ID: 42"
```

**Why This Output Occurs**

The parent passes `handleDelete` as the `onDelete` prop. The child renders a button with an `onClick` handler that wraps `onDelete` in an arrow function to pass `itemId`. When clicked, the arrow function executes, calling `handleDelete(42)`, which triggers the alert.

**Example 2: Class Component Function Props (Legacy Pattern)**

```jsx
import React from 'react';

class ParentComponent extends React.Component {
  // Arrow function class property avoids manual binding
  handleSave = (data) => {
    console.log('Saving:', data);
  };
  
  render() {
    return <ChildForm onSave={this.handleSave} />;
  }
}

class ChildForm extends React.Component {
  handleSubmit = () => {
    // Call the function prop with data
    this.props.onSave({ name: 'John', age: 30 });
  };
  
  render() {
    return (
      <button onClick={this.handleSubmit}>
        Save User
      </button>
    );
  }
}
```

**Expected Output**

```
[Button with text "Save User"]
When clicked: Console logs "Saving: { name: 'John', age: 30 }"
```

**Why This Output Occurs**

The parent class defines `handleSave` as an arrow function class property, which automatically binds `this` to the component instance. This function is passed as `onSave` to `ChildForm`. The child's `handleSubmit` method (also an arrow function property) calls `this.props.onSave()` with the user data object, which invokes the parent's `handleSave` and logs the data.

### Real-World Cases

**Case: A Form with onSubmit Callback**

A `LoginForm` component receives an `onSubmit` function prop. When the user submits the form, the component calls `onSubmit(credentials)`, allowing the parent to handle authentication logic without the form needing to know about API calls.

**Case: A Pagination Component**

A `Pagination` component receives an `onPageChange` function prop. When the user clicks a page number, the component calls `onPageChange(newPage)`, and the parent updates its state to fetch and display the corresponding data.

### References

- React — Passing Functions to Components (FAQ) - https://legacy.reactjs.org/docs/faq-functions.html
- React — Event Handling - https://react.dev/learn/responding-to-events

---

## 8. Object and Array Props

### Definitions

**Core Definition**

Object and array props are props whose values are JavaScript objects or arrays, allowing complex, structured data to be passed to components.

**Technical Definition**

Objects are passed using double curly braces `{{ }}` and arrays with single curly braces `{[ ]}`. The component receives references to these data structures and can access their properties or iterate over their elements.

**Beginner-Friendly Explanation**

Object props are like passing a file folder containing multiple documents. Array props are like passing a list with multiple items. The component can open the folder or read the list to get the details.

### Purposes

- To pass structured, related data as a single prop.
- To enable rendering lists of data.
- To provide configuration objects with multiple settings.
- To pass nested data from API responses to presentational components.

### Syntax Rules and Structure

**Complete General Syntax (Object Prop)**

```jsx
<Component config={{ key1: 'value1', key2: 'value2' }} />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `config` | The prop name. |
| `{{ ... }}` | Outer braces: enter JSX expression. Inner braces: object literal. |

**Complete General Syntax (Array Prop)**

```jsx
<Component items={['item1', 'item2', 'item3']} />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `items` | The prop name. |
| `{[...]}` | Curly braces containing an array literal. |

**Syntax Rules**

1. Objects require **double curly braces**: `{{ }}`. The outer pair tells JSX to evaluate an expression; the inner pair creates the object.
2. Arrays use **single curly braces**: `{[ ]}`.
3. When rendering arrays, each element should have a unique `key` prop for React's reconciliation algorithm.
4. Object props can be destructured at multiple levels.
5. Arrays are often rendered using the `.map()` method.

**Constraints and Limitations**

- Props passed as objects or arrays are **references**, not copies. Mutating the original object in the parent affects the child (though this violates React's immutability principles).
- Inline object and array literals create new references on every render, potentially causing unnecessary re-renders in memoized components.
- Deeply nested objects can make prop interfaces difficult to understand; consider flattening or splitting into multiple props.

### Annotated Complete Code Examples

**Example 1: Object Prop for User Data**

```jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <img src={user.avatarUrl} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <p>{user.location}</p>
    </div>
  );
}

export default function App() {
  // Object literal passed as the user prop
  const userData = {
    name: 'Sarah Chen',
    email: '[email protected]',
    avatarUrl: '/avatars/sarah.jpg',
    location: 'Seattle, WA'
  };
  
  return <UserCard user={userData} />;
}
```

**Expected Output**

```
[User card with Sarah's avatar, name, email, and location]
```

**Why This Output Occurs**

The `userData` object is passed as the `user` prop. Inside `UserCard`, the component accesses `user.name`, `user.email`, and other properties to render the card. This approach keeps the parent's code organized and lets the child component define how to display the data.

**Example 2: Array Prop for Rendering Lists**

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        // Each list item needs a unique key prop
        <li key={index} style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

export default function App() {
  const myTodos = [
    { text: 'Learn React props', completed: true },
    { text: 'Build a project', completed: false },
    { text: 'Review hooks', completed: false }
  ];
  
  return <TodoList todos={myTodos} />;
}
```

**Expected Output**

```
• Learn React props (strikethrough)
• Build a project
• Review hooks
```

**Why This Output Occurs**

The `myTodos` array is passed as the `todos` prop. The `TodoList` component calls `todos.map()` to iterate over each todo object. For each item, it renders an `<li>` with the text and applies a conditional strikethrough style based on the `completed` property. The `key` prop helps React identify which items have changed.

### Real-World Cases

**Case: Passing API Response Data**

After fetching data from a REST API, a parent component receives a JSON object containing arrays and nested objects. It passes slices of this data to child components: `<UserList users={data.users} />`, `<StatsPanel stats={data.statistics} />`.

**Case: Configuration Object for a Chart Component**

A `LineChart` component receives a `config` object prop with properties like `xAxisLabel`, `yAxisLabel`, `colors`, and `showGrid`. This single prop encapsulates all chart customization.

### References

- React — Rendering Lists - https://react.dev/learn/rendering-lists
- React — Passing Props to a Component (Objects and Arrays) - https://react.dev/learn/passing-props-to-a-component

---

## 9. Children Prop

### Definitions

**Core Definition**

The `children` prop is a special prop that contains whatever JSX is written between the opening and closing tags of a component.

**Technical Definition**

In React, when a component is used with nested JSX content (e.g., `<Card>content</Card>`), React automatically passes that content as the `children` property of the props object. The `children` prop can contain strings, numbers, elements, components, fragments, or arrays of these.

**Beginner-Friendly Explanation**

The `children` prop is like a container component: you put things inside the box, and the box displays them wherever its design specifies. You don't have to name the content—it's just "the children."

### Purposes

- To create wrapper or layout components that render arbitrary nested content.
- To enable composition patterns where a component's content is provided at the usage site.
- To build reusable UI containers like cards, modals, and layouts.
- To pass JSX as a prop without needing an explicit prop name.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* Parent usage */}
<Wrapper>
  <p>Child content here</p>
</Wrapper>

{/* Child component definition */}
function Wrapper({ children }) {
  return <div className="wrapper">{children}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<Wrapper>...</Wrapper>` | Opening and closing tags; content between them becomes `children`. |
| `{ children }` | Destructured prop containing the nested JSX. |
| `{children}` | Renders the children content inside the wrapper's JSX. |

**Syntax Rules**

1. The `children` prop is automatically provided by React; it does not need to be passed explicitly.
2. Content between opening and closing tags—including text, elements, and other components—is collected as `children`.
3. `children` can be rendered using `{children}` anywhere inside the component's JSX.
4. Self-closing components `<Component />` have `children` as `undefined`.
5. When manipulating `children` programmatically (counting, mapping, converting to array), use the `React.Children` utilities because the `children` data structure is opaque.

**Constraints and Limitations**

- `children` is **not** a regular array; its structure is implementation-dependent. Do not assume it is an array.
- You cannot access the rendered output of nested components through `children`; you only get the JSX elements as passed.
- Manipulating `children` directly (e.g., `children.map()`) is discouraged in favor of `React.Children.map()` or restructuring the component API.

### Annotated Complete Code Examples

**Example 1: Basic Children Prop**

```jsx
// Card component renders whatever is placed between its tags
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

export default function App() {
  return (
    <Card>
      <h2>Welcome</h2>
      <p>This content is passed as children.</p>
      <button>Learn More</button>
    </Card>
  );
}
```

**Expected Output**

```
[Card container]
  Welcome (heading)
  This content is passed as children. (paragraph)
  [Learn More button]
```

**Why This Output Occurs**

All JSX written between `<Card>` and `</Card>` is collected by React into the `children` prop. The `Card` component renders `{children}` inside its `card-body` div. React then renders the nested content—the heading, paragraph, and button—in their original order.

**Example 2: Children with Multiple Elements and Composition**

```jsx
function Layout({ header, children, footer }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <main>{children}</main>
      <footer>{footer}</footer>
    </div>
  );
}

export default function App() {
  return (
    <Layout
      header={<h1>My App</h1>}
      footer={<p>© 2025</p>}
    >
      {/* This content becomes the children prop */}
      <p>Main content goes here.</p>
      <p>Multiple elements are allowed.</p>
    </Layout>
  );
}
```

**Expected Output**

```
[Header: My App]
[Main: Main content goes here. Multiple elements are allowed.]
[Footer: © 2025]
```

**Why This Output Occurs**

This example demonstrates a common pattern: named props (`header`, `footer`) for explicit content slots, and `children` for the primary content. React collects the JSX between the opening and closing `<Layout>` tags as `children`, while `header` and `footer` are passed explicitly. The `Layout` component renders each in its designated location.

### Real-World Cases

**Case: Modal Component**

A `Modal` component receives `isOpen`, `onClose`, and `children`. The children contain the modal's body content, which varies with each usage. The modal provides the overlay, positioning, and close behavior, while the parent provides the content.

**Case: Accordion or Tabs Component**

An `Accordion` component receives multiple `AccordionItem` children. The accordion manages the open/close state, and each item's content is provided as children of the `AccordionItem`. This composition pattern is idiomatic React.

### References

- React — Passing JSX as Children - https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React — Children API Reference - https://react.dev/reference/react/Children

---

## References

- React Official Documentation — Passing Props to a Component - https://react.dev/learn/passing-props-to-a-component
- React Official Documentation — Components and Props (Legacy) - https://legacy.reactjs.org/docs/components-and-props.html
- React Official Documentation — Passing Functions to Components (FAQ) - https://legacy.reactjs.org/docs/faq-functions.html
- React Official Documentation — Children API Reference - https://react.dev/reference/react/Children
- React Official Documentation — Rendering Lists - https://react.dev/learn/rendering-lists
- React Official Documentation — Responding to Events - https://react.dev/learn/responding-to-events
- React Official Documentation — Common Components (DOM Elements) - https://react.dev/reference/react-dom/components/common
- MDN Web Docs — Destructuring Assignment - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
- MDN Web Docs — Default Parameters - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
- CoreUI — How to Pass Props in React - https://coreui.io/answers/how-to-pass-props-in-react/
- CoreUI — How to Type Props in React with TypeScript - https://coreui.io/answers/how-to-type-props-in-react-with-typescript/