# JSX — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

JSX (JavaScript XML) is a syntax extension for JavaScript that allows developers to write HTML-like markup directly inside JavaScript files, enabling the co-location of rendering logic and UI structure within React components.

**Technical Definition**

JSX is a syntactic sugar that is transformed by a compiler (such as Babel, TypeScript, or SWC) into regular JavaScript function calls before reaching the browser. In the modern React 17+ automatic runtime, JSX compiles to calls to `jsx()` and `jsxs()` functions imported from `react/jsx-runtime`; in the legacy transform, it compiles to `React.createElement()` calls . JSX is not a template language but a JavaScript expression that produces `ReactElement` objects—plain JavaScript objects describing what should appear on screen .

**Beginner-Friendly Explanation**

JSX is like writing HTML inside your JavaScript code. Instead of creating elements with complicated function calls, you write what looks like HTML tags, and a tool (the compiler) translates them into JavaScript that the browser can understand. It makes building user interfaces feel natural and readable because your markup and logic live side by side.

### Key Characteristics

- **Syntax Extension**: JSX is not valid JavaScript on its own; it requires a compiler to transform it into executable code .
- **Expression-Based**: Every JSX element is a JavaScript expression that produces an object, meaning it can be assigned to variables, returned from functions, and passed as arguments.
- **HTML-Like but Stricter**: JSX resembles HTML but enforces stricter rules: all tags must be closed, attributes use camelCase, and multiple elements must be wrapped in a single root .
- **React-Independent**: JSX is a separate specification from React; other libraries like Preact and Solid also support JSX .

### Prerequisites

Before studying JSX, learners should be comfortable with:

- **JavaScript Fundamentals**: Variables, functions, expressions, and object literals.
- **Basic HTML Structure**: Tags, attributes, and nesting.
- **ES Modules**: Import and export syntax for sharing code between files.

### Related Programming Areas

- **React Elements**: The JavaScript objects that JSX produces, which describe UI structure.
- **Component Composition**: JSX enables components to render other components as children.
- **Build Tooling**: Babel, TypeScript, and bundlers (Vite, Webpack) handle JSX transformation.

### Core Concepts / Features

- JSX Definition
- JSX Expressions
- JavaScript Inside JSX
- JSX Attributes
- Nested Elements
- Fragments
- Self-Closing Elements
- JSX Restrictions
- JSX Transformation into JavaScript

---

## 1. JSX Definition

### Definitions

**Core Definition**

JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside a JavaScript file to describe UI structure.

**Technical Definition**

JSX is a specification (originally created by Facebook, now maintained by the React team) for embedding XML-like syntax within JavaScript. When a compiler encounters JSX, it parses the markup into JavaScript function calls that create React elements. JSX itself is not a language but a syntax layer that must be compiled before execution .

**Beginner-Friendly Explanation**

JSX is a way to write what looks like HTML directly in your JavaScript code. It's like a shorthand that lets you describe what your UI should look like, and a tool translates it into something the browser understands.

### Purposes

- To make UI code more readable and expressive by combining markup and logic in one place.
- To reduce the cognitive distance between the structure of the UI and the code that creates it.
- To provide a familiar HTML-like syntax for developers transitioning from web development.
- To enable static analysis and type checking of UI structure through tooling.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const element = <h1 className="title">Hello, world!</h1>;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<h1>` | The opening tag; creates an element of type `h1`. |
| `className="title"` | An attribute (prop) passed to the element. |
| `Hello, world!` | The children (text content) of the element. |
| `</h1>` | The closing tag. |
| `const element =` | JSX is an expression; it can be assigned to a variable. |

**Syntax Rules**

1. JSX must be compiled before execution; browsers do not understand JSX natively .
2. JSX elements must have a single root (see Nested Elements).
3. Component names must start with a capital letter; HTML tags are lowercase .

**Constraints and Limitations**

- JSX is not valid JavaScript; it cannot run without a build step.
- JSX is not React; it is a separate spec usable by other libraries .

### Annotated Complete Code Examples

**Example 1: Basic JSX Element**

```jsx
// A simple JSX element assigned to a constant
const greeting = <h1>Hello, JSX!</h1>;

// The compiled output (classic transform) is:
// const greeting = React.createElement('h1', null, 'Hello, JSX!');

// The compiled output (modern transform) is:
// import { jsx as _jsx } from 'react/jsx-runtime';
// const greeting = _jsx('h1', { children: 'Hello, JSX!' });
```

**Expected Output (when rendered)**

```
Hello, JSX!
```

**Why This Output Occurs**

The JSX `<h1>Hello, JSX!</h1>` is transformed by the compiler into a function call that creates a React element object. When React renders this element, it produces an `<h1>` DOM node containing the text "Hello, JSX!" .

**Example 2: JSX is an Expression**

```jsx
// JSX can be used anywhere a JavaScript expression can be used
function getGreeting() {
  return <span>Welcome back!</span>;
}

// Assigning JSX to a variable
const header = <h1>Dashboard</h1>;

// Passing JSX as a function argument
console.log(<div>Debug</div>);

// Returning JSX from a function
function Title() {
  return <h1>Page Title</h1>;
}
```

**Explanation**

Because JSX compiles to a JavaScript expression (a function call returning an object), it can be used in any context where expressions are valid: variable assignment, function return, function arguments, and object properties .

### Real-World Cases

**Case: Component Definition**

Every React component returns JSX to describe its rendered output. A `UserProfile` component returns JSX containing the user's avatar, name, and bio, all described declaratively.

**Case: Conditional Rendering**

JSX expressions are used with conditional operators to render different UI based on state: `{isLoggedIn ? <Dashboard /> : <LoginForm />}` .

### References

- React — Writing Markup with JSX - https://react.dev/learn/writing-markup-with-jsx
- React — Introducing the New JSX Transform - https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html

---

## 2. JSX Expressions

### Definitions

**Core Definition**

A JSX expression is any JSX element or fragment that evaluates to a JavaScript value (a React element object) at runtime.

**Technical Definition**

JSX expressions compile to JavaScript expressions that call `jsx()` or `React.createElement()`, producing `ReactElement` objects. Because they are expressions, JSX elements can be assigned, passed, returned, and embedded within other expressions .

**Beginner-Friendly Explanation**

A JSX expression is like a sentence in a language. It's a complete thought—a piece of UI—that you can use anywhere JavaScript expects a value.

### Purposes

- To create React elements as values that can be manipulated in JavaScript.
- To enable dynamic UI generation based on data and state.
- To allow composition by passing JSX as props or children.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const element = <div className="box">Content</div>;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<div>` | The element type. |
| `className="box"` | Attributes (props). |
| `Content` | Children. |

**Syntax Rules**

1. JSX expressions compile to function calls, producing objects .
2. They can be used anywhere JavaScript expressions are valid.
3. They must return a single root element or a fragment.

**Constraints and Limitations**

- JSX expressions cannot be used as element types directly: `<components[key] />` is invalid; assign to a capitalized variable first .

### Annotated Complete Code Examples

**Example 1: JSX as a Variable Value**

```jsx
// JSX expression assigned to a variable
const button = <button className="primary">Click Me</button>;

// The variable can then be used in other JSX
function Toolbar() {
  return (
    <div className="toolbar">
      {button}
      {button}
    </div>
  );
}
```

**Expected Output**

```
[Click Me button] [Click Me button]
```

**Why This Output Occurs**

The `button` variable holds a React element object. When used inside `{button}`, React renders the same element twice. The JSX expression is evaluated once and the resulting object is reused.

**Example 2: JSX in Conditional Expressions**

```jsx
function Status({ isOnline }) {
  // JSX expression inside a ternary
  return (
    <div>
      Status: {isOnline ? <span>Online</span> : <span>Offline</span>}
    </div>
  );
}
```

**Expected Output (isOnline = true)**

```
Status: Online
```

**Why This Output Occurs**

The ternary expression evaluates to the JSX `<span>Online</span>`, which React renders. JSX expressions work naturally within JavaScript conditional expressions .

### Real-World Cases

**Case: List Rendering**

A list component maps over an array and returns JSX expressions for each item: `items.map(item => <li key={item.id}>{item.name}</li>)`. The result is an array of JSX elements that React renders as a list .

### References

- React — Writing Markup with JSX - https://react.dev/learn/writing-markup-with-jsx
- GreatFrontEnd — What is JSX and how does it work? - https://github.com/greatfrontend/top-reactjs-interview-questions

---

## 3. JavaScript Inside JSX

### Definitions

**Core Definition**

JavaScript inside JSX refers to the use of curly braces `{}` to embed JavaScript expressions, variables, and logic directly within JSX markup.

**Technical Definition**

Curly braces in JSX create an "expression container" that allows any valid JavaScript expression to be evaluated and its result inserted into the JSX tree. This is the mechanism for dynamic content in JSX .

**Beginner-Friendly Explanation**

Curly braces are like a window from JSX into JavaScript. Whatever you put inside them gets calculated and the result appears in the UI.

### Purposes

- To display dynamic data (variables, function results) in the UI.
- To compute values inline within markup.
- To conditionally render content based on JavaScript logic.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* As children */}
<h1>{variableName}</h1>
<h1>{functionCall()}</h1>
<h1>{2 + 2}</h1>

{/* As attributes */}
<img src={imageUrl} alt={description} />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `{variableName}` | Embeds the value of a variable. |
| `{functionCall()}` | Embeds the result of a function call. |
| `{2 + 2}` | Embeds the result of an expression. |

**Syntax Rules**

1. Curly braces can only be used **as text children** or **immediately after `=` in attributes** .
2. Any valid JavaScript expression works inside curly braces: variables, function calls, arithmetic, ternaries, array methods.
3. Statements (`if`, `for`, `while`) are not expressions and cannot be used directly inside curly braces .

**Constraints and Limitations**

- `{tag}` as an element type is invalid: `<{tag}>` will not compile .
- `src="{avatar}"` passes the literal string `"{avatar}"`, not the variable value .

### Annotated Complete Code Examples

**Example 1: Displaying Variables and Expressions**

```jsx
function Profile() {
  const name = 'Hedy Lamarr';
  const age = 30;
  
  return (
    <div>
      {/* Variable */}
      <h1>{name}</h1>
      {/* Expression */}
      <p>In 5 years: {age + 5}</p>
      {/* Function call */}
      <p>Uppercase: {name.toUpperCase()}</p>
    </div>
  );
}
```

**Expected Output**

```
Hedy Lamarr
In 5 years: 35
Uppercase: HEDY LAMARR
```

**Why This Output Occurs**

Each `{}` evaluates a JavaScript expression. `name` resolves to the string. `age + 5` computes 35. `name.toUpperCase()` calls the string method. The results are embedded as text content in the JSX .

**Example 2: Conditional Expressions in JSX**

```jsx
function Welcome({ isLoggedIn, username }) {
  return (
    <div>
      {/* Ternary expression */}
      {isLoggedIn ? <h1>Welcome, {username}!</h1> : <h1>Please log in.</h1>}
      
      {/* Logical AND for conditional rendering */}
      {isLoggedIn && <button>Log Out</button>}
    </div>
  );
}
```

**Expected Output (isLoggedIn = true, username = "Alice")**

```
Welcome, Alice!
[Log Out button]
```

**Why This Output Occurs**

The ternary evaluates to the JSX for the logged-in state. The `&&` expression evaluates to the button element when `isLoggedIn` is truthy. Both are standard JavaScript expressions used within JSX curly braces .

### Real-World Cases

**Case: Dynamic Class Names**

A component applies conditional CSS classes: `<div className={isActive ? 'active' : 'inactive'}>`. The className is determined by a JavaScript expression .

**Case: Inline Styles**

A component sets dynamic inline styles: `<div style={{ backgroundColor: theme.color }}>`. The double braces are a JavaScript object literal inside the JSX curly braces .

### References

- React — JavaScript in JSX with Curly Braces - https://react.dev/learn/javascript-in-jsx-with-curly-braces
- React — Conditional Rendering - https://react.dev/learn/conditional-rendering

---

## 4. JSX Attributes

### Definitions

**Core Definition**

JSX attributes are the properties passed to JSX elements, written similarly to HTML attributes but with JavaScript expression support and camelCase naming conventions.

**Technical Definition**

JSX attributes become the `props` (properties) of the created React element. They are passed as the second argument to `jsx()` or `React.createElement()`. String literals are passed with quotes; all other values use curly braces .

**Beginner-Friendly Explanation**

Attributes are the settings you give to a JSX element—like telling an image what source to use or telling a button what to do when clicked.

### Purposes

- To configure the behavior and appearance of elements.
- To pass data to components.
- To attach event handlers.
- To set HTML attributes like `src`, `href`, and `className`.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* String literal */}
<img src="photo.jpg" alt="A photo" />

{/* JavaScript expression */}
<img src={user.avatarUrl} alt={user.name} />

{/* Boolean shorthand */}
<button disabled />

{/* Event handler */}
<button onClick={handleClick}>Click</button>

{/* Object (double braces) */}
<div style={{ color: 'red' }}>Text</div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `src="photo.jpg"` | String literal in quotes. |
| `src={user.avatarUrl}` | JavaScript expression in curly braces. |
| `disabled` | Boolean attribute defaults to `true`. |
| `onClick={handleClick}` | Event handler (function reference). |
| `style={{ ... }}` | Object literal inside JSX braces. |

**Syntax Rules**

1. **String literals** use quotes: `prop="value"` .
2. **JavaScript expressions** use curly braces: `prop={expression}` .
3. **camelCase**: HTML attributes use camelCase in JSX: `className` not `class`, `htmlFor` not `for`, `tabIndex` not `tabindex` .
4. **Boolean shorthand**: A prop without a value defaults to `true`: `<input disabled />` .
5. **Exception**: `aria-*` and `data-*` attributes remain hyphenated .

**Constraints and Limitations**

- Reserved words like `class` cannot be used as attribute names; use `className` instead .
- Attributes containing dashes (like `stroke-width`) must be camelCased (`strokeWidth`), except `data-*` and `aria-*` .

### Annotated Complete Code Examples

**Example 1: String vs. Expression Attributes**

```jsx
function Avatar({ user }) {
  return (
    <img
      // String literal: the string "avatar" is passed as className
      className="avatar"
      // Expression: the value of user.imageUrl is passed as src
      src={user.imageUrl}
      // Expression: a string concatenation
      alt={'Photo of ' + user.name}
      // Expression: a number
      width={90}
    />
  );
}
```

**Expected Output (user = { name: 'Hedy', imageUrl: '/hedy.jpg' })**

```
[Image with class "avatar", src "/hedy.jpg", alt "Photo of Hedy", width 90]
```

**Why This Output Occurs**

`className="avatar"` passes the literal string. `src={user.imageUrl}` evaluates the expression and passes the URL. `alt={'Photo of ' + user.name}` computes the concatenated string. `width={90}` passes the number 90 .

**Example 2: Inline Styles with Double Braces**

```jsx
function Box() {
  const boxStyle = {
    backgroundColor: 'lightblue',
    padding: '20px',
    borderRadius: '8px'
  };
  
  return (
    <div style={boxStyle}>
      Styled with an object
    </div>
  );
}
```

**Expected Output**

```
[Div with light blue background, 20px padding, 8px border radius]
```

**Why This Output Occurs**

The `style` attribute expects an object. `style={boxStyle}` passes the object variable. The double-brace syntax `style={{ backgroundColor: 'red' }}` is simply an object literal inside JSX curly braces—the outer braces are JSX expression syntax, the inner braces are the JavaScript object .

### Real-World Cases

**Case: Form Inputs**

A form component passes `value`, `onChange`, `placeholder`, and `type` attributes to input elements, controlling their behavior and appearance.

**Case: Navigation Links**

An anchor element receives `href` and `className` attributes, with `href` often computed from route data: `<a href={route.path}>{route.label}</a>`.

### References

- React — Writing Markup with JSX (Attributes) - https://react.dev/learn/writing-markup-with-jsx
- React — JSX In Depth (Props in JSX) - https://legacy.reactjs.org/docs/jsx-in-depth.html

---

## 5. Nested Elements

### Definitions

**Core Definition**

Nested elements are JSX elements contained within other JSX elements, forming a hierarchical tree structure that mirrors the desired UI composition.

**Technical Definition**

In JSX, elements can contain other elements as children, creating a tree of React elements. The compiler transforms this nesting into nested function calls, where the children become the `children` prop of the parent element .

**Beginner-Friendly Explanation**

Nesting is putting elements inside other elements, like boxes inside boxes. A `<div>` can contain a `<h1>` and a `<p>`, and those can contain text and other elements.

### Purposes

- To build complex UI structures from simple elements.
- To establish parent-child relationships for layout and styling.
- To compose components hierarchically.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<div className="container">
  <h1>Title</h1>
  <p>Paragraph with <strong>bold</strong> text.</p>
</div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<div className="container">` | Parent element. |
| `<h1>Title</h1>` | First child. |
| `<p>...</p>` | Second child, itself containing nested elements. |
| `<strong>bold</strong>` | Grandchild inside the paragraph. |

**Syntax Rules**

1. **Single root**: A component must return a single root element wrapping all children .
2. **Proper closing**: All tags must be explicitly closed; nesting must be well-formed .
3. **Depth**: Nesting can be arbitrarily deep, but readability suffers with excessive depth.

**Constraints and Limitations**

- Two sibling elements at the root of a return statement are not allowed without a wrapper .
- Nesting components too deeply can make the component tree hard to debug.

### Annotated Complete Code Examples

**Example 1: Nested HTML Elements**

```jsx
function Article() {
  return (
    <article className="post">
      <header>
        <h1>Understanding JSX</h1>
        <p className="meta">Published on <time>2025-01-15</time></p>
      </header>
      <section>
        <p>
          JSX allows you to write <em>HTML-like</em> markup
          with <strong>JavaScript expressions</strong> embedded.
        </p>
      </section>
    </article>
  );
}
```

**Expected Output**

```
Understanding JSX
Published on 2025-01-15
JSX allows you to write HTML-like markup with JavaScript expressions embedded.
```

**Why This Output Occurs**

The nesting creates a tree: `article` contains `header` and `section`; `header` contains `h1` and `p`; `p` contains `time`. The compiler transforms this into nested function calls, each creating an element with its children as the `children` prop .

**Example 2: Nested Components**

```jsx
function Header() {
  return <h1>My App</h1>;
}

function Nav() {
  return (
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  );
}

function App() {
  return (
    <div>
      <Header />
      <Nav />
      <main>Content here</main>
    </div>
  );
}
```

**Expected Output**

```
My App
Home | About
Content here
```

**Why This Output Occurs**

`App` nests `<Header />`, `<Nav />`, and `<main>`. Each component renders its own JSX. React recursively renders the tree: `App` → `Header` + `Nav` + `main` → their respective DOM elements .

### Real-World Cases

**Case: Layout Components**

A `PageLayout` component nests `Header`, `Sidebar`, and `Content` components, establishing the application's visual hierarchy.

**Case: Form Structures**

A form nests `FieldSet` components, each containing `Label`, `Input`, and `Error` components, creating accessible form structures.

### References

- React — Writing Markup with JSX (Rules of JSX) - https://react.dev/learn/writing-markup-with-jsx
- React — JSX In Depth (Children in JSX) - https://legacy.reactjs.org/docs/jsx-in-depth.html

---

## 6. Fragments

### Definitions

**Core Definition**

A Fragment is a special JSX syntax that groups multiple elements without adding an extra node to the DOM.

**Technical Definition**

`React.Fragment` (or the shorthand `<>...</>`) is a component that allows returning multiple elements from a component without wrapping them in a DOM element. Fragments are compiled to `React.Fragment` or, in the modern transform, to a fragment element type, and they do not produce a corresponding DOM node .

**Beginner-Friendly Explanation**

A Fragment is like an invisible box. You can put multiple elements inside it, and they stay grouped together, but the box itself doesn't appear in the final HTML.

### Purposes

- To return multiple sibling elements without adding an unnecessary wrapper `<div>`.
- To avoid extra DOM nodes that can interfere with CSS layouts (Flexbox, Grid).
- To keep the DOM tree cleaner and more efficient.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* Shorthand syntax */}
<>
  <h1>Title</h1>
  <p>Paragraph</p>
</>

{/* Explicit syntax (requires import) */}
import { Fragment } from 'react';
<Fragment>
  <h1>Title</h1>
  <p>Paragraph</p>
</Fragment>

{/* With key (only explicit syntax supports key) */}
<Fragment key={item.id}>
  <dt>{item.term}</dt>
  <dd>{item.description}</dd>
</Fragment>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<>` | Shorthand opening fragment. |
| `</>` | Shorthand closing fragment. |
| `<Fragment>` | Explicit fragment component (supports `key` prop). |

**Syntax Rules**

1. The shorthand `<>...</>` cannot take attributes, including `key` .
2. The explicit `<Fragment>` form can take a `key` prop when used in lists.
3. Fragments compile to a special element type that React renders as nothing in the DOM.

**Constraints and Limitations**

- Shorthand fragments cannot have `key` or any other attributes.
- Fragments cannot be styled directly; if you need a styled wrapper, use a `<div>` or similar element.

### Annotated Complete Code Examples

**Example 1: Fragment vs. Div Wrapper**

```jsx
{/* Using a div adds an extra DOM node */}
function WithDiv() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

{/* Using a Fragment adds no DOM node */}
function WithFragment() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}
```

**Rendered HTML Comparison**

```
<!-- WithDiv renders: -->
<div>
  <h1>Title</h1>
  <p>Content</p>
</div>

<!-- WithFragment renders: -->
<h1>Title</h1>
<p>Content</p>
```

**Why This Difference Occurs**

The `<div>` creates an actual DOM node. The Fragment `<>...</>` is compiled to a React Fragment element type, which React renders by placing children directly into the parent DOM node without creating a wrapper .

**Example 2: Fragment with Key in Lists**

```jsx
import { Fragment } from 'react';

function Glossary({ items }) {
  return (
    <dl>
      {items.map(item => (
        // Explicit Fragment allows key prop
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.definition}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

**Expected Output (items = [{ id: 1, term: 'JSX', definition: 'Syntax extension' }])**

```
JSX
Syntax extension
```

**Why This Output Occurs**

Each `<Fragment key={...}>` groups the `<dt>` and `<dd>` elements as siblings without a wrapper. The `key` helps React's reconciliation identify each group. The shorthand `<>` cannot be used here because it doesn't support the `key` prop .

### Real-World Cases

**Case: Table Rows**

A component returns multiple `<tr>` elements from a single component, using a Fragment to avoid wrapping them in a `<tbody>` or `<div>`: `<>{rows.map(row => <tr>...</tr>)}</>`.

**Case: CSS Grid/Flex Layouts**

When a component returns multiple items that need to be direct children of a grid or flex container, a Fragment avoids the wrapper that would break the layout.

### References

- React — Writing Markup with JSX (Fragments) - https://react.dev/learn/writing-markup-with-jsx
- React — Fragments - https://react.dev/reference/react/Fragment

---

## 7. Self-Closing Elements

### Definitions

**Core Definition**

Self-closing elements are JSX elements with no children, written with a forward slash before the closing angle bracket: `<element />`.

**Technical Definition**

In JSX, all elements must be explicitly closed. Elements that have no children use the self-closing syntax `<tag />`, which is equivalent to `<tag></tag>`. This applies to void HTML elements like `<img>`, `<br>`, and `<input>`, as well as components .

**Beginner-Friendly Explanation**

Self-closing elements are tags that close themselves. Instead of writing `<img></img>`, you write `<img />`. It's a shorthand for "this element has no content."

### Purposes

- To close elements that have no children in a concise syntax.
- To satisfy JSX's requirement that all tags be closed.
- To distinguish void elements (images, inputs, line breaks) that have no content.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
{/* HTML void elements */}
<img src="photo.jpg" alt="Photo" />
<br />
<input type="text" />

{/* Components with no children */}
<Divider />
<Spacer />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<img ... />` | Self-closing image element. |
| `<br />` | Self-closing line break. |
| `<Component />` | Self-closing component. |

**Syntax Rules**

1. The forward slash `/` before `>` is required for elements with no children .
2. Both `<tag />` and `<tag></tag>` are valid; the self-closing form is preferred for empty elements.
3. Components can be self-closing: `<Header />` is equivalent to `<Header></Header>`.

**Constraints and Limitations**

- Self-closing syntax cannot be used for elements that need children: `<div />` is valid but renders an empty div.
- In JSX, `<br>` is invalid; it must be `<br />` .

### Annotated Complete Code Examples

**Example 1: HTML Void Elements**

```jsx
function Profile({ user }) {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>Bio line 1<br />Bio line 2</p>
      <input type="email" placeholder="Email" />
    </div>
  );
}
```

**Expected Output**

```
[Avatar image]
John Doe
Bio line 1
Bio line 2
[Email input field]
```

**Why This Output Occurs**

`<img />`, `<br />`, and `<input />` are all self-closing because they have no children. Without the `/`, React would throw a syntax error for `<img>` or `<br>` because JSX requires all tags to be closed .

**Example 2: Self-Closing Components**

```jsx
function Header() {
  return <h1>My App</h1>;
}

function Footer() {
  return <footer>© 2025</footer>;
}

function App() {
  return (
    <div>
      <Header />
      <main>Content</main>
      <Footer />
    </div>
  );
}
```

**Explanation**

`<Header />` and `<Footer />` are self-closing because they have no children in this usage. This is equivalent to `<Header></Header>` and `<Footer></Footer>`. The self-closing form is more concise and clearly indicates that no children are passed.

### Real-World Cases

**Case: Icon Components**

Icon components typically accept no children and are always self-closing: `<SearchIcon />`, `<CloseIcon size={24} />`.

**Case: Input Fields**

Form inputs are void elements and must be self-closing: `<input type="text" value={value} onChange={handleChange} />`.

### References

- React — Writing Markup with JSX (Closing Tags) - https://react.dev/learn/writing-markup-with-jsx
- LabEx — React JSX Basics (Self-closing Tags) - https://labex.io/tutorials/react-react-jsx-basics-601739

---

## 8. JSX Restrictions

### Definitions

**Core Definition**

JSX restrictions are the syntax rules and limitations that distinguish JSX from HTML and ensure it compiles correctly to JavaScript.

**Technical Definition**

JSX enforces stricter rules than HTML: single root elements, explicit tag closing, camelCase attributes, expression-only content in curly braces, and capitalized component names. These restrictions exist because JSX compiles to JavaScript function calls with specific structural requirements .

**Beginner-Friendly Explanation**

JSX has rules you must follow, like a game. You can't just write anything—tags must be closed, you can only have one main container, and attribute names are slightly different from HTML.

### Purposes

- To ensure JSX compiles correctly to JavaScript.
- To prevent ambiguous or invalid markup.
- To maintain consistency with JavaScript naming conventions.

### Syntax Rules and Structure

**Restriction 1: Single Root Element**

```jsx
// ❌ Invalid: two sibling elements
return (
  <h1>Title</h1>
  <p>Content</p>
);

// ✅ Valid: wrapped in a parent
return (
  <div>
    <h1>Title</h1>
    <p>Content</p>
  </div>
);

// ✅ Valid: wrapped in a Fragment
return (
  <>
    <h1>Title</h1>
    <p>Content</p>
  </>
);
```

**Restriction 2: Close All Tags**

```jsx
// ❌ Invalid
<img src="photo.jpg">
<br>
<input type="text">

// ✅ Valid
<img src="photo.jpg" />
<br />
<input type="text" />
```

**Restriction 3: camelCase Attributes**

```jsx
// ❌ Invalid (HTML names)
<div class="box" tabindex="0">

// ✅ Valid (camelCase)
<div className="box" tabIndex={0} />

// ❌ Invalid
<label for="email">

// ✅ Valid
<label htmlFor="email">
```

**Restriction 4: Capitalized Component Names**

```jsx
// ❌ Invalid: lowercase treated as HTML tag
function hello() { return <div>Hi</div>; }
<hello />

// ✅ Valid: capitalized
function Hello() { return <div>Hi</div>; }
<Hello />
```

**Restriction 5: No Statements in Curly Braces**

```jsx
// ❌ Invalid: if is a statement
<div>{if (x) { return <span>Yes</span>; }}</div>

// ✅ Valid: ternary (expression)
<div>{x ? <span>Yes</span> : null}</div>
```

**Restriction 6: No Expression as Element Type**

```jsx
// ❌ Invalid
return <components[props.type] />;

// ✅ Valid
const SpecificComponent = components[props.type];
return <SpecificComponent />;
```

**Syntax Rules**

1. **Single root**: Wrap multiple elements in a parent or Fragment .
2. **Close all tags**: Self-closing syntax for void elements .
3. **camelCase**: `className`, `htmlFor`, `tabIndex`, `onClick` .
4. **Capitalized components**: Distinguishes components from HTML tags .
5. **Expressions only**: `if` and `for` statements are not allowed in curly braces .

**Constraints and Limitations**

- `data-*` and `aria-*` attributes are exceptions to camelCase .
- HTML entities can be used in JSX text: `&copy;`, `&amp;`, etc.

### Annotated Complete Code Examples

**Example 1: Fixing Common JSX Errors**

```jsx
// ❌ Original (invalid JSX)
function BadComponent() {
  return (
    <div class="container">
      <img src="photo.jpg">
      <label for="name">Name</label>
      <input type="text" id="name">
    </div>
  );
}

// ✅ Fixed (valid JSX)
function GoodComponent() {
  return (
    <div className="container">
      <img src="photo.jpg" />
      <label htmlFor="name">Name</label>
      <input type="text" id="name" />
    </div>
  );
}
```

**Explanation**

The invalid version uses HTML-style attribute names (`class`, `for`) and unclosed void elements (`<img>`, `<input>`). The fixed version uses camelCase (`className`, `htmlFor`) and self-closing syntax (`<img />`, `<input />`) .

**Example 2: Conditional Rendering Without Statements**

```jsx
// ❌ Invalid: if statement inside JSX
function BadStatus({ isOnline }) {
  return <div>{if (isOnline) { return <span>Online</span>; }}</div>;
}

// ✅ Valid: ternary expression
function GoodStatus({ isOnline }) {
  return <div>{isOnline ? <span>Online</span> : <span>Offline</span>}</div>;
}

// ✅ Valid: logical AND for single branch
function SimpleStatus({ isOnline }) {
  return <div>{isOnline && <span>Online</span>}</div>;
}
```

**Explanation**

`if` is a statement, not an expression, so it cannot be used inside JSX curly braces. Ternary (`? :`) and logical AND (`&&`) are expressions and work correctly .

### Real-World Cases

**Case: Migrating HTML to JSX**

When converting existing HTML templates to JSX, developers must rename attributes (`class` → `className`), close void elements, and wrap multiple roots in a Fragment or div .

**Case: Linting and Formatting**

ESLint and Prettier enforce JSX restrictions, catching errors like unclosed tags and incorrect attribute names before runtime.

### References

- React — Writing Markup with JSX (Rules of JSX) - https://react.dev/learn/writing-markup-with-jsx
- React — JSX In Depth - https://legacy.reactjs.org/docs/jsx-in-depth.html

---

## 9. JSX Transformation into JavaScript

### Definitions

**Core Definition**

JSX transformation is the compilation process that converts JSX syntax into standard JavaScript function calls that browsers can execute.

**Technical Definition**

A compiler (Babel, TypeScript, SWC) parses JSX and transforms it into JavaScript. The **classic transform** (React < 17) converts JSX to `React.createElement(type, props, ...children)`. The **automatic transform** (React 17+) converts JSX to calls to `jsx()` or `jsxs()` from `react/jsx-runtime`, eliminating the need to import React .

**Beginner-Friendly Explanation**

Browsers don't understand JSX, so a tool translates it into regular JavaScript function calls. It's like translating a sentence from one language to another so the browser can understand it.

### Purposes

- To make JSX executable in the browser.
- To create React element objects from declarative markup.
- To enable tooling like Babel to support JSX syntax.

### Syntax Rules and Structure

**Complete General Syntax (Classic Transform)**

```javascript
// JSX source
const element = <h1 className="title">Hello</h1>;

// Compiled output (classic)
const element = React.createElement(
  'h1',
  { className: 'title' },
  'Hello'
);
```

**Complete General Syntax (Modern/Automatic Transform)**

```javascript
// JSX source
const element = <h1 className="title">Hello</h1>;

// Compiled output (automatic)
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', {
  className: 'title',
  children: 'Hello'
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `'h1'` | The element type (string for HTML, function for components). |
| `{ className: 'title' }` | The props object. |
| `'Hello'` | The children (third argument in classic, part of props in modern). |

**Syntax Rules**

1. **Classic transform**: JSX → `React.createElement()`. Requires `import React` .
2. **Automatic transform**: JSX → `jsx()` / `jsxs()` from `react/jsx-runtime`. No React import needed .
3. **Children handling**: Multiple children in automatic transform use `jsxs()`; single child uses `jsx()`.
4. **Key extraction**: The `key` prop is extracted as a separate argument, not part of the props object.

**Constraints and Limitations**

- The transform is done at build time, not runtime.
- The automatic transform requires React 17+ or backported support in React 16.14+ .
- The `jsx-runtime` functions are for compiler use only; manual element creation should use `React.createElement` .

### Annotated Complete Code Examples

**Example 1: Classic Transform with Multiple Children**

```jsx
// Source JSX
const list = (
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
);

// Classic compiled output
const list = React.createElement(
  'ul',
  null,
  React.createElement('li', null, 'Item 1'),
  React.createElement('li', null, 'Item 2')
);
```

**Explanation**

Each JSX element becomes a `React.createElement` call. Children are passed as additional arguments after the props object. Nesting becomes nested function calls .

**Example 2: Modern Transform with Automatic Runtime**

```jsx
// Source JSX
function App() {
  return <h1>Hello World</h1>;
}

// Modern compiled output
import { jsx as _jsx } from 'react/jsx-runtime';

function App() {
  return _jsx('h1', { children: 'Hello World' });
}
```

**Explanation**

The automatic transform imports `jsx` from `react/jsx-runtime` and calls it with the type and props. Children are included in the props object. Notice the source code does **not** import React—only Hooks or other exports would need importing .

**Example 3: Component with Props and Key**

```jsx
// Source JSX
const items = data.map(item => (
  <ListItem key={item.id} value={item.value} />
));

// Modern compiled output (simplified)
import { jsx as _jsx } from 'react/jsx-runtime';

const items = data.map(item => _jsx(ListItem, {
  value: item.value
}, item.id)); // Key is passed as third argument
```

**Explanation**

The `key` prop is special: it is extracted and passed as a separate argument to the `jsx()` function, not included in the props object. This is why `key` is not accessible via `props.key` inside the component .

### Real-World Cases

**Case: Babel Configuration**

A project's `babel.config.js` uses `@babel/preset-react` with `{ runtime: 'automatic' }` to enable the modern transform, eliminating the need for `import React` in every file.

**Case: TypeScript with JSX**

TypeScript's `jsx` compiler option controls the transform: `"react"` for classic, `"react-jsx"` for automatic. The `jsxFactory` option customizes the factory function for non-React libraries .

### References

- React — Introducing the New JSX Transform - https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html
- TypeScript — JSX Documentation - https://www.typescriptlang.org/docs/handbook/jsx.html

---

## References

- React Official Documentation — Writing Markup with JSX - https://react.dev/learn/writing-markup-with-jsx
- React Official Documentation — JavaScript in JSX with Curly Braces - https://react.dev/learn/javascript-in-jsx-with-curly-braces
- React Official Documentation — JSX In Depth (Legacy) - https://legacy.reactjs.org/docs/jsx-in-depth.html
- React Official Documentation — Introducing the New JSX Transform - https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html
- React Official Documentation — Fragments API Reference - https://react.dev/reference/react/Fragment
- React Official Documentation — Conditional Rendering - https://react.dev/learn/conditional-rendering
- TypeScript Documentation — JSX - https://www.typescriptlang.org/docs/handbook/jsx.html
- GreatFrontEnd — What is JSX and how does it work? - https://github.com/greatfrontend/top-reactjs-interview-questions
- LabEx — React JSX Basics - https://labex.io/tutorials/react-react-jsx-basics-601739
- GitHub — Web Dev Notes: JSX and React - https://raw.githubusercontent.com/bgoonz/web-dev-notes-resource-site/snyk-fix-bab6b746986dc8c0050747799742c46e/2-content/blog-posts/ciriculumn/week-14/w14/mds.html/w14_react.md