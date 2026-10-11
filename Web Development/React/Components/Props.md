# React Props — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Props (short for "properties") are read-only inputs passed from a parent component to a child component in React. They are the primary mechanism for component communication.

**Technical Definition:** Props are plain JavaScript objects containing data passed to a React component. They are immutable from the child's perspective and flow unidirectionally from parent to child. React components accept a single `props` argument and access its properties either directly (`props.name`) or via destructuring (`{ name }`). Props can be any JavaScript value: strings, numbers, booleans, objects, arrays, functions, JSX elements, or React nodes. In JSX, props are written as attributes on a component tag. React treats props as immutable snapshots — a component must never mutate its own props. Type validation can be enforced at runtime with `prop-types` or at compile time with TypeScript.

**Beginner-Friendly Explanation:** Props are like settings you pass to a component. Just as you configure a button with a label and colour, you configure a React component with props. A parent component says "here is some data" and the child component uses it to decide what to render. You cannot change props from inside the child — they belong to the parent.

### Key Characteristics

- **Read-only:** Props are immutable snapshots; a component must never modify its own props.
- **Unidirectional:** Data flows one-way from parent to child.
- **Any JavaScript value:** Strings, numbers, objects, arrays, functions, JSX, and React nodes are all valid props.
- **Declarative:** Props are written as JSX attributes on the child component tag.
- **Defaultable:** Default values can be specified using ES6 default parameters.
- **Validatable:** Types can be checked at runtime (`prop-types`) or compile time (TypeScript).

### Prerequisites

- JavaScript fundamentals: functions, objects, arrays, destructuring, spread/rest operators.
- Basic JSX syntax.
- React component basics (function components).

### Related Programming Areas

- **State** — the other data source for React components; unlike props, state is local and mutable.
- **Context** — an alternative to prop drilling for sharing data across the component tree.
- **Component composition** — building complex UIs by nesting components and passing props.
- **Type systems** — TypeScript and Flow for static prop type checking.

### Core Concepts / Features

---

## 1. Props

### Definitions

**Core Definition:** Props are the inputs a React component receives from its parent.

**Technical Definition:** In React, a component is a function that accepts a single `props` object argument and returns React elements. Props are accessed as properties of that object. In JSX, props are passed as attributes on the component tag. React enforces that props are immutable — they are snapshots for a single render and must never be mutated directly.

**Beginner-Friendly Explanation:** Props are the information you pass into a component to tell it what to display or how to behave. Think of props like the arguments you pass to a function.

### Purposes

- To pass data from parent components to child components.
- To customize component appearance and behaviour.
- To enable component reuse with different configurations.
- To facilitate one-way data flow in React applications.
- To separate component logic from the data it operates on.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<ChildComponent propName={value} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `propName` | The name of the prop (camelCase). |
| `value` | Any JavaScript expression enclosed in curly braces. |

**Syntax Rules:**

1. Props are passed as JSX attributes on the component tag.
2. String props can be written with quotes: `name="Alice"`.
3. Dynamic values must use curly braces: `size={100}`.
4. Props are read-only; never assign to `props.something`.

**Constraints and Limitations:**

- Props cannot be modified by the child component.
- Prop names are case-sensitive.
- Passing an object directly requires double curly braces: `person={{ name: 'Alice' }}`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Props

```jsx
// Step 1: Define the child component
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Step 2: Pass props from the parent
function App() {
  return (
    <div>
      <Greeting name="Alice" />
      <Greeting name="Bob" />
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

**Why this output:** The `Greeting` component is called twice with different `name` props. React passes `{ name: 'Alice' }` and `{ name: 'Bob' }` as the props object to each instance.

#### Example 2: Passing Objects and Numbers as Props

```jsx
function Avatar({ person, size }) {
  return (
    <img
      src={person.imageUrl}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{ name: 'Lin Lanying', imageUrl: '/lin.jpg' }}
      />
      <Avatar
        size={80}
        person={{ name: 'Katsuko Saruhashi', imageUrl: '/katsuko.jpg' }}
      />
    </div>
  );
}
```

**Expected Output:**

```html
<div>
  <img src="/lin.jpg" alt="Lin Lanying" width="100" height="100">
  <img src="/katsuko.jpg" alt="Katsuko Saruhashi" width="80" height="80">
</div>
```

**Why this output:** The `person` prop is an object, passed with double curly braces. The `size` prop is a number, passed with single curly braces. Both are used inside `Avatar` to configure the `<img>` element.

### Real-World Cases

- **User profiles:** Passing user objects to profile card components.
- **Product cards:** Passing product data to e-commerce card components.
- **Navigation:** Passing route and label data to nav item components.

### References

- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component
- React — Props and State — https://react.dev/learn/state-a-components-memory

---

## 2. Prop Passing

### Definitions

**Core Definition:** Prop passing is the act of providing data to a child component through JSX attributes.

**Technical Definition:** Prop passing occurs when a parent component includes a child component in its JSX and specifies attributes on that child. Each attribute becomes a key-value pair in the child's `props` object. Props can be passed as static strings, dynamic expressions, or spread from an object.

**Beginner-Friendly Explanation:** Prop passing is how you give data to a component. You write the component tag and add attributes to it, like giving instructions.

### Purposes

- To provide data to child components.
- To configure child component behaviour.
- To pass event handlers and callbacks to children.
- To enable parent-child communication.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Child prop1="static" prop2={dynamic} prop3={expression} />
```

**Syntax Rules:**

1. Static strings use quotes: `label="Submit"`.
2. Dynamic values use curly braces: `count={items.length}`.
3. Boolean props can be shorthand: `<Input disabled />` means `disabled={true}`.
4. Multiple props are separated by spaces.

**Constraints and Limitations:**

- Only one props object is passed to each component.
- Prop names must be valid JavaScript identifiers.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Passing Multiple Prop Types

```jsx
function Button({ label, variant, disabled, onClick }) {
  return (
    <button
      className={`btn btn-${variant}`}
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
}

function App() {
  return (
    <Button
      label="Submit"                          // String prop
      variant="primary"                       // String prop
      disabled={false}                        // Boolean expression
      onClick={() => alert('Submitted!')}     // Function prop
    />
  );
}
```

**Expected Output:**

```html
<button class="btn btn-primary">Submit</button>
```

**Why this output:** Each prop type is handled appropriately: strings render as text, booleans control attributes, and functions become event handlers.

### Real-World Cases

- **Forms:** Passing values and handlers to input components.
- **Lists:** Passing item data to list item components.
- **Layouts:** Passing content and configuration to layout components.

### References

- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component

---

## 3. Prop Drilling

### Definitions

**Core Definition:** Prop drilling is the practice of passing props through multiple layers of intermediate components that do not themselves need the data.

**Technical Definition:** Prop drilling occurs when a deeply nested component needs data that originates from a distant ancestor. Because React props flow one-way from parent to child, the data must be passed explicitly through every intermediate component. This can become verbose and inconvenient, especially when many components need the same data. React's Context API was designed to solve this problem by allowing a parent to make data available to any descendant without passing props through intermediate layers.

**Beginner-Friendly Explanation:** Prop drilling is like passing a note through a long line of people to get it to the person at the end. Everyone in the middle has to handle the note even though it is not for them. React Context is like making an announcement that everyone can hear directly.

### Purposes

- (Prop drilling is a problem to avoid, not a purposeful feature.)
- To illustrate why Context exists as an alternative.
- To understand the trade-offs between explicit prop passing and implicit context.

### Syntax Rules and Structure

**Problematic Pattern:**

```jsx
function App() {
  const user = { name: 'Alice' };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;  // Does not use user
}

function Sidebar({ user }) {
  return <UserMenu user={user} />;  // Does not use user
}

function UserMenu({ user }) {
  return <span>{user.name}</span>;  // Finally uses user
}
```

**Context Alternative:**

```jsx
const UserContext = createContext();

function App() {
  const user = { name: 'Alice' };
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const user = useContext(UserContext);
  return <span>{user.name}</span>;
}
```

**Syntax Rules:**

1. Prop drilling requires every intermediate component to accept and forward the prop.
2. Context eliminates prop drilling by providing data directly to descendants.
3. Use Context when many components at different levels need the same data.

**Constraints and Limitations:**

- Prop drilling is acceptable for shallow trees (2–3 levels).
- Context introduces implicit dependencies, making component reuse harder.
- Overusing Context can make data flow harder to trace.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Prop Drilling (Before Context)

```jsx
function App() {
  const theme = 'dark';
  return <Header theme={theme} />;
}

function Header({ theme }) {
  return <Navigation theme={theme} />;
}

function Navigation({ theme }) {
  return <ThemeToggle theme={theme} />;
}

function ThemeToggle({ theme }) {
  return <button className={theme}>Toggle Theme</button>;
}
```

**Expected Output:**

```html
<button class="dark">Toggle Theme</button>
```

**Why this output:** The `theme` prop must be passed through `Header` and `Navigation` even though neither uses it. Only `ThemeToggle` needs it.

#### Example 2: Context Alternative

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Header />
    </ThemeContext.Provider>
  );
}

function Header() {
  return <Navigation />;
}

function Navigation() {
  return <ThemeToggle />;
}

function ThemeToggle() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Toggle Theme</button>;
}
```

**Expected Output:** Same as Example 1 but without prop drilling.

**Why this output:** `ThemeToggle` reads the theme directly from context, bypassing `Header` and `Navigation`.

### Real-World Cases

- **Theming:** Theme values (light/dark) needed by deeply nested components.
- **Authentication:** Current user data needed by many components.
- **Localisation:** Language and locale settings needed throughout the app.

### References

- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- React — Context — https://react.dev/reference/react/createContext

---

## 4. Prop Types

### Definitions

**Core Definition:** Prop types are the expected data types for a component's props, used for validation and documentation.

**Technical Definition:** Prop types can be enforced at runtime using the `prop-types` library (by assigning a `propTypes` static property to the component) or at compile time using TypeScript. Runtime prop types check values during development and log warnings when a prop does not match its declared type. TypeScript provides static type checking during development and compilation.

**Beginner-Friendly Explanation:** Prop types are like labels that say "this prop should be a string" or "this prop should be a number." They help catch mistakes early.

### Purposes

- To document the intended types of a component's props.
- To catch type errors during development.
- To improve code readability and maintainability.
- To provide inline documentation in editors (with TypeScript).

### Syntax Rules and Structure

**Runtime (`prop-types`):**

```jsx
import PropTypes from 'prop-types';

function Greeting({ name, age }) {
  return <h1>Hello, {name}! You are {age}.</h1>;
}

Greeting.propTypes = {
  name: PropTypes.string,
  age: PropTypes.number,
};
```

**Compile Time (TypeScript):**

```tsx
interface GreetingProps {
  name: string;
  age: number;
}

function Greeting({ name, age }: GreetingProps) {
  return <h1>Hello, {name}! You are {age}.</h1>;
}
```

**Syntax Rules:**

1. `propTypes` is assigned as a static property on the component.
2. Validators include `PropTypes.string`, `PropTypes.number`, `PropTypes.bool`, `PropTypes.func`, `PropTypes.object`, `PropTypes.array`, `PropTypes.node`, `PropTypes.element`, and more.
3. Append `.isRequired` to make a prop mandatory: `PropTypes.string.isRequired`.
4. TypeScript uses `interface` or `type` to describe props.

**Constraints and Limitations:**

- `prop-types` only runs in development mode; warnings are stripped in production.
- React 19 removed internal support for `propTypes` and `defaultProps` on function components; the `prop-types` library still works independently but React will not emit warnings automatically.
- TypeScript catches errors at compile time but requires a build step.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Runtime Prop Types with `prop-types`

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, isAdmin, hobbies }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <span>Admin</span>}
      <ul>
        {hobbies.map((h) => <li key={h}>{h}</li>)}
      </ul>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isAdmin: PropTypes.bool,
  hobbies: PropTypes.arrayOf(PropTypes.string),
};

UserCard.defaultProps = {
  age: 0,
  isAdmin: false,
  hobbies: [],
};
```

**Expected Output:** A user card. If `name` is missing or not a string, a console warning appears in development.

**Why this output:** `propTypes` validates each prop. `isRequired` on `name` warns if it is missing. `arrayOf` validates that `hobbies` is an array of strings.

#### Example 2: TypeScript Props

```tsx
interface UserCardProps {
  name: string;
  age?: number;
  isAdmin?: boolean;
  hobbies?: string[];
}

function UserCard({ name, age = 0, isAdmin = false, hobbies = [] }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <span>Admin</span>}
      <ul>
        {hobbies.map((h) => <li key={h}>{h}</li>)}
      </ul>
    </div>
  );
}
```

**Expected Output:** Same as Example 1, but TypeScript catches errors at compile time.

**Why this output:** The `interface` defines the expected shape of props. Optional props are marked with `?`. Default values are provided via ES6 default parameters.

### Real-World Cases

- **Component libraries:** Prop types document the public API of reusable components.
- **Large teams:** Type checking prevents integration bugs.
- **Refactoring:** Types catch breakages when props change.

### References

- React — Typechecking With PropTypes — https://legacy.reactjs.org/docs/typechecking-with-proptypes.html
- React — Using TypeScript — https://react.dev/learn/typescript
- prop-types — npm — https://www.npmjs.com/package/prop-types

---

## 5. Default Props

### Definitions

**Core Definition:** Default props are fallback values used when a prop is not provided by the parent.

**Technical Definition:** Default props can be specified using ES6 default parameters in the function signature (modern approach) or the legacy `defaultProps` static property (class components and older function components). Default values apply only when the prop is `undefined`; passing `null` will remain `null`. React 19 removed support for `defaultProps` on function components; ES6 default parameters are now the recommended approach.

**Beginner-Friendly Explanation:** Default props are like backup values. If the parent does not provide a prop, the component uses the default instead.

### Purposes

- To provide sensible fallback values for optional props.
- To make components more robust against missing data.
- To reduce the need for conditional checks inside components.
- To document which props are optional.

### Syntax Rules and Structure

**Modern Approach (ES6 Default Parameters):**

```jsx
function Button({ label = 'Click me', variant = 'primary', disabled = false }) {
  return <button className={`btn btn-${variant}`} disabled={disabled}>{label}</button>;
}
```

**Legacy Approach (`defaultProps`):**

```jsx
function Button({ label, variant, disabled }) {
  return <button className={`btn btn-${variant}`} disabled={disabled}>{label}</button>;
}

Button.defaultProps = {
  label: 'Click me',
  variant: 'primary',
  disabled: false,
};
```

**Syntax Rules:**

1. Default parameters are applied when the prop is `undefined`.
2. `defaultProps` is assigned as a static property on the component.
3. Default values do not apply if `null` is passed.

**Constraints and Limitations:**

- React 19 removed `defaultProps` support for function components; use ES6 default parameters instead.
- Class components still support `defaultProps` since there is no ES6 alternative.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: ES6 Default Parameters

```jsx
function Alert({ type = 'info', message = 'Something happened.', dismissible = false }) {
  return (
    <div className={`alert alert-${type}`}>
      {message}
      {dismissible && <button>Dismiss</button>}
    </div>
  );
}

// Usage without props
<Alert />

// Usage with some props
<Alert type="error" message="Something went wrong." />
```

**Expected Output (first usage):**

```html
<div class="alert alert-info">Something happened.</div>
```

**Expected Output (second usage):**

```html
<div class="alert alert-error">Something went wrong.</div>
```

**Why this output:** When props are omitted, the default values are used. When props are provided, they override the defaults.

#### Example 2: TypeScript with Default Parameters

```tsx
interface AlertProps {
  type?: 'info' | 'warning' | 'error';
  message?: string;
  dismissible?: boolean;
}

function Alert({
  type = 'info',
  message = 'Something happened.',
  dismissible = false,
}: AlertProps) {
  return (
    <div className={`alert alert-${type}`}>
      {message}
      {dismissible && <button>Dismiss</button>}
    </div>
  );
}
```

**Expected Output:** Same as Example 1. TypeScript marks all props as optional with `?`.

**Why this output:** The interface marks props as optional, and the default parameters provide fallback values.

### Real-World Cases

- **UI primitives:** Buttons, inputs, and alerts with sensible defaults.
- **Configuration components:** Components with many optional settings.
- **Backward compatibility:** Adding new props without breaking existing usage.

### References

- React — Specifying a Default Value for a Prop — https://react.dev/learn/passing-props-to-a-component#specifying-a-default-value-for-a-prop
- CoreUI — How to set default props in React — https://coreui.io/answers/how-to-set-default-props-in-react/

---

## 6. Destructuring Props

### Definitions

**Core Definition:** Destructuring props is a JavaScript syntax that extracts specific properties from the `props` object into individual variables.

**Technical Definition:** Destructuring can be performed in the function parameter list (`function Component({ name, age })`) or inside the function body (`const { name, age } = props`). It is an ES6 feature that makes component code cleaner by eliminating repetitive `props.` prefixes. Destructuring also supports default values and renaming.

**Beginner-Friendly Explanation:** Destructuring lets you pull out just the props you need and use them as variables, instead of writing `props.name` every time.

### Purposes

- To write cleaner, more readable component code.
- To avoid repetitive `props.` prefixes.
- To combine prop extraction with default values.
- To rename props when there is a naming conflict.

### Syntax Rules and Structure

**General Syntax (Parameter Destructuring):**

```jsx
function Component({ prop1, prop2, prop3 }) {
  // prop1, prop2, prop3 are available as variables
}
```

**General Syntax (Body Destructuring):**

```jsx
function Component(props) {
  const { prop1, prop2 } = props;
}
```

**Syntax Rules:**

1. Destructuring in the parameter list is the most common pattern.
2. Default values can be combined: `function Component({ name = 'Guest' })`.
3. Props can be renamed: `function Component({ name: userName })`.
4. Nested destructuring is supported: `function Component({ user: { name } })`.

**Constraints and Limitations:**

- Destructuring creates local variables; changes to them do not affect `props`.
- Deeply nested destructuring can reduce readability.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Destructuring

```jsx
function UserProfile({ name, email, age }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <p>Age: {age}</p>
    </div>
  );
}

// Usage
<UserProfile name="Alice" email="alice@example.com" age={30} />
```

**Expected Output:**

```html
<div>
  <h2>Alice</h2>
  <p>alice@example.com</p>
  <p>Age: 30</p>
</div>
```

**Why this output:** The three props are destructured in the parameter list and used directly as variables.

#### Example 2: Destructuring with Defaults and Renaming

```jsx
function Button({ label: buttonLabel = 'Click', variant = 'primary', ...rest }) {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {buttonLabel}
    </button>
  );
}
```

**Expected Output:**

```html
<button class="btn btn-primary">Click</button>
```

**Why this output:** `label` is renamed to `buttonLabel` with a default of `'Click'`. `variant` has a default of `'primary'`. The rest props are spread onto the button.

### Real-World Cases

- **All modern React components:** Destructuring is the standard way to access props.
- **UI library components:** Components with many props benefit most from destructuring.
- **Custom hooks:** Destructuring is also used for hook return values.

### References

- Destructuring — Learn Destructuring — https://github.com/oliverjam/learn-destructuring
- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component

---

## 7. Rest Props

### Definitions

**Core Definition:** Rest props (or "rest parameters") collect the remaining props that were not explicitly destructured into a single object.

**Technical Definition:** The rest operator (`...`) in a destructuring pattern gathers all remaining enumerable properties into a new object. In React, this is commonly used to separate known props from unknown ones, so the unknown props can be forwarded to a child element or DOM node. The rest object is typically named `rest` or `props`.

**Beginner-Friendly Explanation:** Rest props let you say "give me these specific props, and put everything else in a bag." You can then pass that bag to another element.

### Purposes

- To separate known props from unknown ones.
- To forward remaining props to a child or DOM element.
- To create wrapper components that pass through props.
- To avoid manually listing every prop a component should forward.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Component({ knownProp, ...rest }) {
  return <div {...rest} />;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `knownProp` | A prop that is explicitly extracted. |
| `...rest` | An object containing all other props. |
| `{...rest}` | JSX spread that applies the rest props as attributes. |

**Syntax Rules:**

1. The rest element must be the last item in the destructuring pattern.
2. `rest` is a plain JavaScript object.
3. Spread the rest object onto a JSX element using `{...rest}`.

**Constraints and Limitations:**

- Spreading unknown props onto a DOM element can pass invalid HTML attributes.
- The rest object is a new object each render; memoization may be affected.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Forwarding Rest Props to a DOM Element

```jsx
function TextInput({ id, label, ...inputProps }) {
  return (
    <label htmlFor={id}>
      {label}
      <input id={id} {...inputProps} />
    </label>
  );
}

// Usage
<TextInput
  id="email"
  label="Email:"
  type="email"
  placeholder="Enter email"
  required
/>
```

**Expected Output:**

```html
<label for="email">
  Email:
  <input id="email" type="email" placeholder="Enter email" required>
</label>
```

**Why this output:** `id` and `label` are extracted explicitly. The remaining props (`type`, `placeholder`, `required`) are spread onto the `<input>` element.

#### Example 2: Rest Props with TypeScript

```tsx
interface TextInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
}

function TextInput({ label, ...inputProps }: TextInputProps) {
  return (
    <label>
      {label}
      <input {...inputProps} />
    </label>
  );
}
```

**Expected Output:** Same as Example 1. TypeScript ensures the rest props match input attributes.

**Why this output:** The interface extends `React.InputHTMLAttributes`, so TypeScript knows the rest props are valid input attributes.

### Real-World Cases

- **Design system wrappers:** Wrapping native elements and forwarding props.
- **Form components:** Extracting label and id, forwarding the rest to the input.
- **Layout components:** Forwarding className, style, and data attributes.

### References

- Destructuring — Rest Operator — https://github.com/oliverjam/learn-destructuring
- React Patterns — Destructuring, Rest, and Spread — https://reactpatterns.com/

---

## 8. Spread Props

### Definitions

**Core Definition:** Spread props use the JavaScript spread operator to pass all properties of an object as individual JSX attributes.

**Technical Definition:** The spread syntax `{...object}` in JSX expands an object's enumerable properties into individual props. It is syntactic sugar for manually writing each property as an attribute. Spread props are commonly combined with rest props to forward unknown props to child elements. The spread operator can also merge objects: `<Component {...defaults} {...overrides} />`.

**Beginner-Friendly Explanation:** Spreading props lets you pass an entire object of props at once, instead of writing each prop individually.

### Purposes

- To pass a large number of props without listing each one.
- To forward props from a parent to a child.
- To merge default props with override props.
- To enable higher-order component patterns.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Component {...propsObject} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{...propsObject}` | Spreads all enumerable properties of `propsObject` as JSX attributes. |

**Syntax Rules:**

1. Spread attributes can be combined with regular attributes.
2. Later attributes override earlier ones: `<Comp {...props} className="override" />`.
3. Order matters: attributes after the spread override those in the spread.

**Constraints and Limitations:**

- Spreading unknown objects can pass invalid HTML attributes to DOM elements.
- The spread creates a new props object each render.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Spreading Props from an Object

```jsx
function App() {
  const buttonProps = {
    label: 'Submit',
    variant: 'primary',
    disabled: false,
    onClick: () => alert('Clicked!'),
  };

  return <Button {...buttonProps} />;
}

function Button({ label, variant, disabled, onClick }) {
  return (
    <button className={`btn btn-${variant}`} disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}
```

**Expected Output:**

```html
<button class="btn btn-primary">Submit</button>
```

**Why this output:** The `buttonProps` object is spread into individual props on the `Button` component.

#### Example 2: Merging Defaults with Overrides

```jsx
const defaultProps = { variant: 'primary', size: 'medium', disabled: false };

<Button {...defaultProps} variant="danger" />
// Result: variant="danger", size="medium", disabled={false}
```

**Expected Output:** A danger button with medium size, enabled.

**Why this output:** The `variant` prop after the spread overrides the spread `variant`.

### Real-World Cases

- **Configuration objects:** Passing API response data as props.
- **HOC wrappers:** Forwarding all original props to the wrapped component.
- **Default prop merging:** Applying defaults then overriding specific values.

### References

- React — JSX Spread Attributes — https://legacy.reactjs.org/docs/jsx-in-depth.html#spread-attributes
- React Patterns — Spread Attributes — https://reactpatterns.com/#jsx-spread-attributes

---

## 9. Children Prop

### Definitions

**Core Definition:** The `children` prop is a special React prop that contains the nested content between a component's opening and closing tags.

**Technical Definition:** `children` is a reserved prop name in React. When JSX content is placed between opening and closing tags of a component, React automatically passes that content as the `children` prop. The value can be a React element, string, number, array, fragment, function, or `null`. Components can render `children` using `{props.children}`.

**Beginner-Friendly Explanation:** The `children` prop is everything you put between a component's opening and closing tags. It lets you pass content into a component.

### Purposes

- To pass arbitrary content to wrapper components.
- To enable composition without knowledge of what will be rendered.
- To create reusable layout components.
- To support the slot pattern for multiple content regions.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Parent>
  <Child />
  text content
  {expression}
</Parent>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<Parent>` | The containing component. |
| Content between tags | Becomes the `children` prop. |
| `{props.children}` | Renders the children inside the parent. |

**Syntax Rules:**

1. `children` is automatically provided by React.
2. It can be a single element, multiple elements, text, or a function.
3. Components render `children` with `{props.children}` or `{children}` after destructuring.
4. `children` can be explicitly passed as a prop: `<Comp children="text" />`.

**Constraints and Limitations:**

- A component cannot use both `children` and `dangerouslySetInnerHTML` simultaneously.
- If `children` is a function, the component must call it.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Children Prop

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-body">{children}</div>
    </div>
  );
}

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
  <h3>Welcome</h3>
  <div class="card-body">
    <p>This is the card content.</p>
    <button>Learn more</button>
  </div>
</div>
```

**Why this output:** The `<p>` and `<button>` are passed as `children` to `Card`, which renders them inside `card-body`.

#### Example 2: Children as a Function

```jsx
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return children(position);
}

// Usage
<MouseTracker>
  {({ x, y }) => <p>Mouse: {x}, {y}</p>}
</MouseTracker>
```

**Expected Output:** The mouse position updates as the user moves the mouse.

**Why this output:** `children` is a function that receives the position and returns JSX.

### Real-World Cases

- **Layout components:** Headers, sidebars, and modals use children for content.
- **Design systems:** Card, Dialog, and Accordion components use children.
- **Render props alternative:** Children as a function replaces the render prop pattern.

### References

- React — Passing JSX as Children — https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React — Children Prop — https://react.dev/reference/react/Children

---

## 10. Render Props

### Definitions

**Core Definition:** A render prop is a prop whose value is a function that returns JSX, allowing a component to share logic while delegating rendering to the caller.

**Technical Definition:** The render prop pattern involves passing a function as a prop (conventionally named `render`) to a component. The component calls this function with data or state, and the function returns React elements. This inverts control: the component provides the logic, and the consumer decides what to render. The pattern is less common in modern React (replaced by custom Hooks) but remains useful for sharing stateful logic.

**Beginner-Friendly Explanation:** A render prop is a function you pass to a component that tells it what to render. The component provides the data, and your function decides how to display it.

### Purposes

- To share stateful logic between components.
- To invert control of rendering to the consumer.
- To provide flexibility in what a component renders.
- To avoid inheritance for code reuse.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<DataProvider render={(data) => <Child data={data} />} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `render` | The prop that accepts a function. |
| `(data) =>` | The function receives data from the provider. |
| `<Child data={data} />` | The function returns JSX. |

**Syntax Rules:**

1. The render prop is a function, not a JSX element.
2. The component calls the function with data or state.
3. The function returns React elements.

**Constraints and Limitations:**

- Render props can be less readable than custom Hooks.
- Nested render props can create "wrapper hell."
- Modern React favours custom Hooks for logic reuse.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Render Prop for Mouse Tracking

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

#### Example 2: Render Prop with Data Fetching

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading });
}

// Usage
<DataFetcher
  url="/api/user"
  render={({ data, loading }) => (
    loading ? <Spinner /> : <UserProfile user={data} />
  )}
/>
```

**Expected Output:** A spinner while loading, then the user profile.

**Why this output:** `DataFetcher` handles the fetching logic and passes `data` and `loading` to the render function.

### Real-World Cases

- **Legacy codebases:** Render props are common in older React libraries.
- **Library components:** Some libraries use render props for flexible APIs.
- **Custom Hooks migration:** Render props can often be replaced by custom Hooks.

### References

- React — Render Props — https://legacy.reactjs.org/docs/render-props.html
- Vercel — Prefer Children Over Render Props — https://github.com/vercel-labs/agent-skills

---

## 11. Function as Children

### Definitions

**Core Definition:** Function as children is a pattern where the `children` prop is a function instead of JSX, allowing a component to call the function with data.

**Technical Definition:** When `children` is a function, the component calls `children(data)` and uses the returned JSX. This is essentially the render prop pattern using the `children` prop instead of a named prop. It is convenient because `children` is a reserved prop that does not require a custom prop name.

**Beginner-Friendly Explanation:** Instead of passing JSX as children, you pass a function. The component calls your function with some data, and your function returns what to show.

### Purposes

- To combine the children prop with the render prop pattern.
- To provide data to children without a named render prop.
- To simplify APIs where only one render function is needed.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Component>
  {(data) => <Child data={data} />}
</Component>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<Component>` | The component that calls `children`. |
| `{(data) => ...}` | A function passed as children. |
| `data` | Data provided by the component. |

**Syntax Rules:**

1. The component must check if `children` is a function and call it.
2. The function receives data from the component.
3. The function returns React elements.

**Constraints and Limitations:**

- Only one children function can be passed per component.
- Less explicit than a named render prop.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Function as Children

```jsx
function PageWidth({ children }) {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return children(width);
}

// Usage
<PageWidth>
  {(width) => (
    <div>
      <p>Page width: {width}px</p>
      {width > 768 ? <DesktopLayout /> : <MobileLayout />}
    </div>
  )}
</PageWidth>
```

**Expected Output:** The page width is displayed and the layout changes based on width.

**Why this output:** `PageWidth` provides the width to the children function, which decides what to render.

### Real-World Cases

- **Responsive components:** Providing screen dimensions to children.
- **Data providers:** Providing fetched data to children functions.
- **State providers:** Providing state and setters to children functions.

### References

- Function as Child Component — https://github.com/ericelliott/react-function-as-child-component
- Educative — Functions as a Child and Render Props — https://www.educative.io/courses/advanced-react-patterns

---

## 12. Prop Validation

### Definitions

**Core Definition:** Prop validation is the process of checking that props passed to a component match the expected types and constraints.

**Technical Definition:** Prop validation can be performed at runtime using the `prop-types` library, which logs warnings in development when a prop does not match its declared type. It can also be performed at compile time using TypeScript or Flow, which catch type errors before the code runs. React 19 removed internal support for `propTypes` and `defaultProps` on function components, but the `prop-types` library still works when invoked explicitly.

**Beginner-Friendly Explanation:** Prop validation checks that you are passing the right kinds of data to a component. It helps catch mistakes early.

### Purposes

- To catch type errors during development.
- To document the expected shape of a component's props.
- To improve code reliability and maintainability.
- To provide better developer experience with editor hints.

### Syntax Rules and Structure

**Runtime (`prop-types`):**

```jsx
import PropTypes from 'prop-types';

Component.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  hobbies: PropTypes.arrayOf(PropTypes.string),
  status: PropTypes.oneOf(['active', 'inactive']),
};
```

**Compile Time (TypeScript):**

```tsx
interface ComponentProps {
  name: string;
  age?: number;
  hobbies?: string[];
  status?: 'active' | 'inactive';
}
```

**Syntax Rules:**

1. `propTypes` is assigned as a static property on the component.
2. Append `.isRequired` to make a prop mandatory.
3. Use `oneOf` for enum-like props, `arrayOf` for typed arrays, `shape` for nested objects.
4. TypeScript uses `interface` or `type` with optional (`?`) markers.

**Constraints and Limitations:**

- `prop-types` only runs in development; no runtime cost in production.
- React 19 does not automatically invoke `propTypes` warnings on function components.
- TypeScript requires a build step and `.tsx` files.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Runtime Validation with `prop-types`

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, status, hobbies }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Status: {status}</p>
      <ul>{hobbies.map((h) => <li key={h}>{h}</li>)}</ul>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  status: PropTypes.oneOf(['active', 'inactive']),
  hobbies: PropTypes.arrayOf(PropTypes.string),
};
```

**Expected Output:** A user card. In development, if `name` is missing or `status` is invalid, a console warning appears.

**Why this output:** `propTypes` validates each prop during development and logs warnings for mismatches.

#### Example 2: Compile-Time Validation with TypeScript

```tsx
interface UserCardProps {
  name: string;
  age?: number;
  status?: 'active' | 'inactive';
  hobbies?: string[];
}

function UserCard({ name, age, status, hobbies = [] }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Status: {status}</p>
      <ul>{hobbies.map((h) => <li key={h}>{h}</li>)}</ul>
    </div>
  );
}

// TypeScript error: Property 'name' is missing
<UserCard age={30} />
```

**Expected Output:** A compile-time error if required props are missing or types are wrong.

**Why this output:** TypeScript checks prop types at compile time and prevents invalid usage.

### Real-World Cases

- **Component libraries:** Validation documents the public API.
- **Team projects:** Type checking prevents integration bugs.
- **Refactoring:** Types catch breakages when props change.

### References

- React — Typechecking With PropTypes — https://legacy.reactjs.org/docs/typechecking-with-proptypes.html
- React — Using TypeScript — https://react.dev/learn/typescript

---

## 13. `prop-types`

### Definitions

**Core Definition:** `prop-types` is the npm package that provides runtime type checking for React props.

**Technical Definition:** The `prop-types` library exports a set of validator functions (`PropTypes.string`, `PropTypes.number`, etc.) that can be assigned to a component's `propTypes` static property. React calls these validators in development mode and logs a console warning when a prop does not match its declared type. The library was extracted from React core in React 15.5.

**Beginner-Friendly Explanation:** `prop-types` is a tool that checks whether you are passing the right types of props to your components. It warns you in the console if something is wrong.

### Purposes

- To validate prop types at runtime during development.
- To document the expected types of a component's props.
- To catch common mistakes like passing a string instead of a number.
- To provide a lightweight alternative to TypeScript for small projects.

### Syntax Rules and Structure

**Installation:**

```bash
npm install prop-types
```

**Usage:**

```jsx
import PropTypes from 'prop-types';

Component.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  onClick: PropTypes.func,
  items: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    text: PropTypes.string,
  })),
};
```

**Common Validators:**

| Validator | Description |
|-----------|-------------|
| `PropTypes.string` | String |
| `PropTypes.number` | Number |
| `PropTypes.bool` | Boolean |
| `PropTypes.func` | Function |
| `PropTypes.object` | Object |
| `PropTypes.array` | Array |
| `PropTypes.node` | Anything renderable |
| `PropTypes.element` | React element |
| `PropTypes.oneOf([...])` | Enum-like values |
| `PropTypes.oneOfType([...])` | Union types |
| `PropTypes.arrayOf(...)` | Typed array |
| `PropTypes.shape({...})` | Object with specific shape |

**Syntax Rules:**

1. Import `PropTypes` from `prop-types`.
2. Assign `Component.propTypes`.
3. Append `.isRequired` for mandatory props.
4. Custom validators can be defined as functions.

**Constraints and Limitations:**

- Only runs in development mode.
- React 19 removed automatic invocation on function components; explicit invocation is required.
- Does not catch errors at compile time.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic `prop-types` Usage

```jsx
import PropTypes from 'prop-types';

function Button({ label, variant, disabled, onClick }) {
  return (
    <button className={`btn btn-${variant}`} disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}

Button.propTypes = {
  label: PropTypes.string.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
  disabled: PropTypes.bool,
  onClick: PropTypes.func,
};

Button.defaultProps = {
  variant: 'primary',
  disabled: false,
};
```

**Expected Output:** A button. In development, warnings appear for invalid prop types.

**Why this output:** `propTypes` validates each prop and logs warnings for mismatches.

#### Example 2: Shape Validation

```jsx
import PropTypes from 'prop-types';

function UserCard({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

UserCard.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string,
    address: PropTypes.shape({
      city: PropTypes.string,
      country: PropTypes.string,
    }),
  }).isRequired,
};
```

**Expected Output:** A user card. Warnings appear if the `user` object is missing required fields.

**Why this output:** `shape` validates the structure of the object, including nested objects.

### Real-World Cases

- **Component libraries:** Validate the public API of reusable components.
- **Small projects:** Type checking without TypeScript.
- **Legacy codebases:** Existing `propTypes` documentation.

### References

- React — Typechecking With PropTypes — https://legacy.reactjs.org/docs/typechecking-with-proptypes.html
- prop-types — npm — https://www.npmjs.com/package/prop-types

---

## 14. TypeScript Props

### Definitions

**Core Definition:** TypeScript props are statically typed props defined using TypeScript interfaces or type aliases.

**Technical Definition:** In TypeScript, a component's props are typed by defining an `interface` or `type` that describes the shape of the props object. The type is applied to the function component's parameter. TypeScript checks prop usage at compile time, providing autocompletion and error detection in editors. Props should be described as object types using `type` or `interface`.

**Beginner-Friendly Explanation:** TypeScript props let you define exactly what types of props a component expects. If you pass the wrong type, TypeScript tells you before you run the code.

### Purposes

- To catch prop type errors at compile time.
- To provide editor autocompletion and inline documentation.
- To make component APIs self-documenting.
- To enable safer refactoring.

### Syntax Rules and Structure

**Inline Type:**

```tsx
function MyButton({ title }: { title: string }) {
  return <button>{title}</button>;
}
```

**Interface:**

```tsx
interface MyButtonProps {
  title: string;
  disabled?: boolean;
}

function MyButton({ title, disabled }: MyButtonProps) {
  return <button disabled={disabled}>{title}</button>;
}
```

**Type Alias:**

```tsx
type MyButtonProps = {
  title: string;
  disabled?: boolean;
};
```

**Syntax Rules:**

1. Use `.tsx` extension for files containing JSX.
2. Install `@types/react` and `@types/react-dom`.
3. Props should be an object type described with `type` or `interface`.
4. Use `?` for optional props.
5. Use union types for props with limited values: `'primary' | 'secondary'`.

**Constraints and Limitations:**

- TypeScript requires a build step.
- Type errors do not appear at runtime; they are compile-time only.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic TypeScript Props

```tsx
interface UserCardProps {
  name: string;
  age: number;
  email?: string;
  isAdmin?: boolean;
}

function UserCard({ name, age, email, isAdmin = false }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {email && <p>{email}</p>}
      {isAdmin && <span>Admin</span>}
    </div>
  );
}

// Usage
<UserCard name="Alice" age={30} />
<UserCard name="Bob" age={25} email="bob@example.com" isAdmin />
```

**Expected Output:** Two user cards with different configurations.

**Why this output:** TypeScript enforces that `name` and `age` are provided and are the correct types.

#### Example 2: Typing Children and Event Handlers

```tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

function Button({ children, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}

// Usage
<Button onClick={() => alert('Clicked')}>Click me</Button>
```

**Expected Output:** A button with the specified variant and click handler.

**Why this output:** `React.ReactNode` types the children prop. The union type constrains `variant` to specific values.

### Real-World Cases

- **Large applications:** Type safety prevents integration bugs.
- **Design systems:** Types document the component API.
- **Team collaboration:** Types serve as contracts between components.

### References

- React — Using TypeScript — https://react.dev/learn/typescript
- CoreUI — How to type props in React with TypeScript — https://coreui.io/blog/how-to-type-props-in-react-with-typescript/

---

## 15. Props Immutability

### Definitions

**Core Definition:** Props immutability means that a component must never modify its own props.

**Technical Definition:** In React, props are immutable snapshots with respect to a single render. A component must never mutate its props directly. Instead, it should pass new props down to children or use state to manage changes. Mutating props directly is a violation of React's purity rules and can lead to bugs, inconsistent UI, and missed re-renders because React cannot detect the change.

**Beginner-Friendly Explanation:** Props are read-only. You can look at them, but you must not change them. If you need to change data, use state instead.

### Purposes

- To ensure predictable component behaviour.
- To prevent bugs caused by unexpected data mutations.
- To enable React's rendering optimisations.
- To maintain one-way data flow.

### Syntax Rules and Structure

**Correct (No Mutation):**

```jsx
function UserCard({ user }) {
  // Read-only access
  return <h2>{user.name}</h2>;
}
```

**Incorrect (Mutation):**

```jsx
function UserCard({ user }) {
  user.name = 'Bob';  // ❌ Mutation!
  return <h2>{user.name}</h2>;
}
```

**Syntax Rules:**

1. Never assign to `props.something` or mutate nested objects.
2. To "change" props, pass new props from the parent.
3. Use state (`useState`) for data that needs to change over time.

**Constraints and Limitations:**

- JavaScript objects are mutable by default; there is no runtime enforcement.
- ESLint plugins (e.g., `eslint-plugin-react-hooks`) can catch prop mutations.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Correct vs Incorrect Prop Mutation

```jsx
// ❌ Incorrect: mutating props
function BadComponent({ items }) {
  items.push('new item');  // Mutates the array!
  return <ul>{items.map((i) => <li key={i}>{i}</li>)}</ul>;
}

// ✅ Correct: creating a new array
function GoodComponent({ items }) {
  const newItems = [...items, 'new item'];  // New array
  return <ul>{newItems.map((i) => <li key={i}>{i}</li>)}</ul>;
}
```

**Expected Output:** Both render a list, but the bad version mutates the parent's data.

**Why this output:** `push` mutates the original array. The spread operator creates a new array without affecting the original.

#### Example 2: Using State Instead of Prop Mutation

```jsx
function Counter({ initialCount }) {
  // ❌ Wrong: trying to mutate the prop
  // initialCount = 10;

  // ✅ Correct: using state initialised from the prop
  const [count, setCount] = useState(initialCount);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Expected Output:** A counter initialised with the prop value, then updated via state.

**Why this output:** The prop provides the initial value. State manages subsequent changes.

### Real-World Cases

- **Debugging:** Prop mutations cause hard-to-trace bugs.
- **Performance:** React cannot optimise mutated props.
- **Strict Mode:** React Strict Mode double-renders to detect mutations.

### References

- React — Props and State Are Immutable — https://react.dev/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable
- React — Immutability ESLint Rule — https://react.dev/reference/eslint-plugin-react-hooks/lints/immutability

---

## 16. Props Best Practices

### Definitions

**Core Definition:** Props best practices are guidelines for using props effectively, safely, and maintainably.

**Technical Definition:** Best practices encompass immutability, type validation, default values, destructuring, avoiding prop drilling, using children appropriately, and following React's purity rules. These practices improve code quality, prevent bugs, and align with the React team's recommendations.

**Beginner-Friendly Explanation:** Best practices are the "do's and don'ts" that experienced React developers follow to keep their code clean and bug-free.

### Purposes

- To write clean, maintainable component APIs.
- To prevent common props-related bugs.
- To improve application performance.
- To ensure consistency across a codebase.

### Syntax Rules and Structure

**Best Practices Table:**

| Practice | Description |
|----------|-------------|
| Never mutate props | Treat props as read-only. |
| Use destructuring | Extract props in the parameter list for readability. |
| Provide default values | Use ES6 default parameters for optional props. |
| Validate types | Use `prop-types` or TypeScript. |
| Avoid prop drilling | Use Context for deeply shared data. |
| Prefer children over render props | Children are more readable and composable. |
| Spread with care | Do not pass invalid HTML attributes to DOM elements. |
| Keep props minimal | Pass only what the component needs. |
| Use descriptive prop names | Avoid generic names like `data` or `value`. |
| Document props | Use TypeScript types or JSDoc comments. |

**Syntax Rules:**

1. Props must never be mutated.
2. Type validation should be applied to all reusable components.
3. Context should be used when prop drilling exceeds 2–3 levels.
4. Children are preferred over render props for static composition.

**Constraints and Limitations:**

- Overusing Context can make data flow harder to trace.
- Memoisation can be broken by new object/function props each render.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Good vs Bad Prop Usage

```jsx
// ❌ Bad: mutating props, no validation, prop drilling
function BadUserCard(props) {
  props.user.name = 'Bob';  // Mutation!
  return <h2>{props.user.name}</h2>;
}

// ✅ Good: no mutation, destructuring, validation
import PropTypes from 'prop-types';

function GoodUserCard({ user, onEdit }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
}

GoodUserCard.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
  }).isRequired,
  onEdit: PropTypes.func.isRequired,
};
```

**Expected Output:** A user card that does not mutate data and validates props.

**Why this output:** The good version follows immutability, destructuring, and type validation practices.

#### Example 2: Context Instead of Prop Drilling

```jsx
// ❌ Bad: prop drilling through 3 levels
function App() {
  const user = { name: 'Alice' };
  return <Layout user={user} />;
}
function Layout({ user }) {
  return <Sidebar user={user} />;
}
function Sidebar({ user }) {
  return <UserMenu user={user} />;
}

// ✅ Good: Context
const UserContext = createContext();

function App() {
  const user = { name: 'Alice' };
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}
function Layout() {
  return <Sidebar />;
}
function Sidebar() {
  return <UserMenu />;
}
function UserMenu() {
  const user = useContext(UserContext);
  return <span>{user.name}</span>;
}
```

**Expected Output:** Both render the user name, but the Context version avoids prop drilling.

**Why this output:** Context provides the user data directly to `UserMenu` without intermediate props.

### Real-World Cases

- **Code reviews:** Best practices are enforced through ESLint and review checklists.
- **Design systems:** Component libraries follow strict props best practices.
- **Performance optimisation:** Avoiding mutations and unnecessary prop spreading improves re-renders.

### References

- React — Rules of React — https://react.dev/reference/rules
- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component
- React — Props and State Are Immutable — https://react.dev/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable
- Vercel — Prefer Children Over Render Props — https://github.com/vercel-labs/agent-skills

---

## References

- React — Passing Props to a Component — https://react.dev/learn/passing-props-to-a-component
- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- React — Typechecking With PropTypes — https://legacy.reactjs.org/docs/typechecking-with-proptypes.html
- React — Using TypeScript — https://react.dev/learn/typescript
- React — Rules of React — https://react.dev/reference/rules
- React — Props and State Are Immutable — https://react.dev/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable
- React — Immutability ESLint Rule — https://react.dev/reference/eslint-plugin-react-hooks/lints/immutability
- React — Render Props — https://legacy.reactjs.org/docs/render-props.html
- React — JSX Spread Attributes — https://legacy.reactjs.org/docs/jsx-in-depth.html#spread-attributes
- React — Children Prop — https://react.dev/reference/react/Children
- prop-types — npm — https://www.npmjs.com/package/prop-types
- CoreUI — How to set default props in React — https://coreui.io/answers/how-to-set-default-props-in-react/
- CoreUI — How to type props in React with TypeScript — https://coreui.io/blog/how-to-type-props-in-react-with-typescript/
- Vercel — Prefer Children Over Render Props — https://github.com/vercel-labs/agent-skills
- Destructuring — Learn Destructuring — https://github.com/oliverjam/learn-destructuring