# React Inline Styles — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Inline styles in React are CSS rules applied directly to an element through the `style` prop, which accepts a JavaScript object instead of a string.

**Technical Definition:** The `style` prop accepts an object whose keys are camelCase CSS property names and whose values are strings or numbers. React converts this object to the DOM element's `style` attribute. Numeric values are automatically suffixed with `px` for properties that accept pixel units, while unitless properties (such as `opacity`, `zIndex`, `flex`) remain as numbers. Vendor prefixes must be written manually in camelCase with a leading capital letter (except `ms`). The `style` prop is intended for dynamic styles where values depend on JavaScript variables; for static styles, `className` with CSS classes is recommended. The React documentation explicitly states: "We recommend using `style` only for dynamic styles where you don't know the style values ahead of time" .

**Beginner-Friendly Explanation:** Inline styles let you write CSS directly in your component as a JavaScript object. Instead of writing `style="background-color: blue"` like in HTML, you write `style={{ backgroundColor: 'blue' }}`. React converts this object into actual styles on the element. It is best for styles that change based on data, like the width of a progress bar or the color of a button based on a prop. For static styles, you should use CSS classes instead.

### Key Characteristics

- **Object-based:** The `style` prop accepts a JavaScript object, not a CSS string.
- **camelCase property names:** CSS properties like `background-color` become `backgroundColor`.
- **Automatic `px` for numbers:** Numeric values for pixel-accepting properties receive `px` automatically; unitless properties remain numbers.
- **Manual vendor prefixes:** React does not add vendor prefixes automatically; you must add them yourself in camelCase.
- **No pseudo-classes or media queries:** Inline styles cannot express `:hover`, `:focus`, media queries, or keyframe animations.
- **New object each render:** Style objects created in JSX are re-created on every render, which can affect memoization.

### Prerequisites

- JavaScript objects and camelCase naming conventions.
- React function components and JSX.
- Understanding of the `className` attribute for static styles.
- Familiarity with CSS properties and values.

### Related Programming Areas

- **CSS stylesheets** — the recommended approach for static styles.
- **CSS Modules** — scoped static styles with zero runtime cost.
- **CSS custom properties** — for dynamic theming with better performance than inline styles.
- **`useMemo`** — for memoizing style objects when passing them to memoized components.

---

## 1. Inline Styles

### Definitions

**Core Definition:** Inline styles are CSS rules applied directly to an element through the `style` prop as a JavaScript object.

**Technical Definition:** In React, inline styles are set using the `style` attribute, which accepts a JavaScript object with camelCased properties . This differs from HTML, where the `style` attribute takes a CSS string. React transpiles the JSX and converts the object into the DOM element's inline style. The React team explicitly warns against using inline styles as the primary means of styling: "using the `style` attribute as the primary means of styling elements is generally not recommended. In most cases, `className` should be used to reference classes defined in an external CSS stylesheet. `style` is most often used in React apps to add dynamically computed styles at render time" .

**Beginner-Friendly Explanation:** Inline styles let you apply CSS directly to an element inside your JavaScript code. You use an object instead of a string. It is convenient for styles that change based on data, but for everything else, CSS classes are better.

### Purposes

- To apply styles that depend on JavaScript variables or props.
- To avoid creating separate stylesheets for one-off dynamic styles.
- To compute styles at runtime based on component state.
- To override styles in specific situations without touching CSS files.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<div style={{ propertyName: value }} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `style={{ ... }}` | Outer `{}` is JSX expression syntax; inner `{}` creates the style object. |
| `propertyName` | camelCase CSS property name (e.g., `fontSize`, `backgroundColor`). |
| `value` | String or number; numbers receive `px` for pixel-accepting properties. |

**Syntax Rules:**

1. The `style` prop accepts a JavaScript object, not a string.
2. CSS property names use camelCase: `fontSize`, not `font-size` .
3. Numeric values receive `px` automatically for pixel-accepting properties .
4. Unitless properties (e.g., `opacity`, `zIndex`, `lineHeight`, `fontWeight`) do not receive `px`.
5. String values must include units when needed: `{ width: '50%' }`.
6. Vendor prefixes use camelCase with a capital first letter: `WebkitTransition`, `msTransition` (the `ms` prefix is the only lowercase exception) .

**Constraints and Limitations:**

- Inline styles have higher specificity than CSS classes, making them harder to override.
- Pseudo-classes (`:hover`, `:focus`) and media queries cannot be used .
- Keyframe animations cannot be defined inline.
- A new object is created on every render if written inline, which can affect memoization .

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Inline Styles

```jsx
function ProfileCard() {
  const cardStyle = {
    display: 'flex',
    alignItems: 'center',
    padding: 20,               // → 20px
    borderRadius: 8,           // → 8px
    boxShadow: '0 0 3px 2px #cec7c759',
  };

  return (
    <div style={cardStyle}>
      <h2>Alice</h2>
      <p>Frontend Developer</p>
    </div>
  );
}
```

**Expected Output:** A flex card with 20px padding, 8px border radius, and a subtle shadow.

**Why this output:** `padding` and `borderRadius` are numeric and receive `px`. `display`, `alignItems`, and `boxShadow` are strings and are applied as-is. The `cardStyle` object is created once and reused.

#### Example 2: Dynamic Inline Styles

```jsx
function ProgressBar({ percent }) {
  return (
    <div
      style={{
        width: `${percent}%`,
        backgroundColor: percent > 80 ? 'green' : 'orange',
        transition: 'width 0.3s ease, background-color 0.3s ease',
      }}
    />
  );
}
```

**Expected Output:** A progress bar whose width and color depend on the `percent` prop.

**Why this output:** The `style` object is computed during render based on `percent`. React applies the resulting inline styles directly to the DOM node. The `transition` property is a string and is applied as-is.

### Real-World Cases

- **Progress indicators:** Setting width dynamically based on percentage.
- **Drag-and-drop:** Updating `transform` or `left`/`top` during dragging.
- **Theming:** Applying user-selected accent colors to specific elements.
- **Animation:** Computing interpolation values during a transition.

### References

- React — Common Components: style — https://18.react.dev/reference/react-dom/components/common
- Pluralsight — Inline styling with React — https://www.pluralsight.com/resources/blog/guides/inline-styling-with-react

---

## 2. `style` Prop

### Definitions

**Core Definition:** The `style` prop is the React prop that accepts a JavaScript object of CSS properties and applies them as inline styles to a DOM element.

**Technical Definition:** The `style` prop is a special React prop supported by all built-in browser components. It accepts an object whose keys are camelCase CSS property names and whose values are strings or numbers. React converts this object to the DOM element's inline style. The prop is documented in React's official reference: "`style`: An object with CSS styles, for example `{ fontWeight: 'bold', margin: 20 }`. Similarly to the DOM `style` property, the CSS property names need to be written as camelCase, for example `fontWeight` instead of `font-weight`. You can pass strings or numbers as values. If you pass a number, like `width: 100`, React will automatically append `px` ("pixels") to the value unless it's a unitless property" .

**Beginner-Friendly Explanation:** The `style` prop is how you pass an object of CSS rules to an element in React. You write it as `style={{ ... }}` and React turns it into actual styles on the element.

### Purposes

- To apply dynamic styles computed at runtime.
- To avoid creating separate stylesheets for one-off styles.
- To override styles in specific situations.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<div style={{ color: 'blue', fontSize: 16 }} />
```

**Syntax Rules:**

1. The prop name is `style` (lowercase).
2. The value is a JavaScript object.
3. The object keys are camelCase CSS property names.
4. The object values are strings or numbers.

**Constraints and Limitations:**

- The `style` prop is intended for dynamic styles only .
- Static styles should use `className` with CSS classes.

### Annotated Code Examples

```jsx
function Button({ variant }) {
  const baseStyle = {
    padding: '8px 16px',
    border: 'none',
    borderRadius: 4,
    cursor: 'pointer',
  };

  const variantStyle = {
    primary: { background: 'blue', color: 'white' },
    secondary: { background: 'gray', color: 'white' },
  }[variant];

  return (
    <button style={{ ...baseStyle, ...variantStyle }}>
      Click
    </button>
  );
}
```

**Expected Output:** A button with base styles plus the styles for the specified variant.

**Why this output:** The `style` prop receives a merged object. `baseStyle` provides common styles, and `variantStyle` overrides `background` and `color`. The spread operator creates a new object each render, which is acceptable for this dynamic use case.

### Real-World Cases

- **Variant-driven components:** Buttons, badges, and alerts with dynamic colors.
- **Layout adjustments:** Dynamically setting `width`, `height`, or `transform`.

### References

- React — Common Components: style — https://18.react.dev/reference/react-dom/components/common

---

## 3. Style Objects

### Definitions

**Core Definition:** A style object is a plain JavaScript object whose keys are camelCase CSS property names and whose values are CSS values (strings or numbers).

**Technical Definition:** A style object is the value passed to the `style` prop. It is not a CSS string — it is a JavaScript object that React converts to inline styles. Because it is a regular JavaScript object, it can be created outside the component (for static styles), computed inside the component (for dynamic styles), or merged from multiple sources using the spread operator. The object itself is not reactive — React reads its properties during render and applies them to the DOM.

**Beginner-Friendly Explanation:** A style object is just a JavaScript object that describes CSS. You write `{ color: 'blue', fontSize: 16 }` instead of `"color: blue; font-size: 16px"`.

### Purposes

- To define styles as reusable JavaScript objects.
- To compose styles from multiple sources with the spread operator.
- To compute styles dynamically based on props or state.
- To extract style definitions outside the component for readability.

### Syntax Rules and Structure

```jsx
const myStyle = {
  color: 'blue',
  lineHeight: 10,    // → 10px (lineHeight is unitless, so 10 remains unitless? Actually lineHeight is unitless, so no px.)
  padding: '1.5em',  // String with unit
};
```

**Syntax Rules:**

1. Style objects use camelCase keys.
2. Values can be strings (`'1.5em'`, `'50%'`) or numbers (`16`, `0.9`).
3. Numbers receive `px` for pixel-accepting properties; unitless properties remain numbers.
4. Style objects can be defined outside the component for static styles .
5. Style objects can be merged with `{ ...baseStyle, ...overrideStyle }`.

### Annotated Code Examples

```jsx
// Static style object outside the component
const cardStyles = {
  container: {
    display: 'flex',
    height: 100,
    width: 400,
    boxShadow: '0 0 3px 2px #cec7c759',
    alignItems: 'center',
    padding: 20,
    borderRadius: 20,
  },
  profilePicture: {
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'orange',
    color: 'white',
    height: 20,
    width: 20,
    borderRadius: '50%',
    padding: 10,
    fontWeight: 'bold',
  },
};

function UserCard() {
  return (
    <div style={cardStyles.container}>
      <span style={cardStyles.profilePicture}>D</span>
      <div style={{ marginLeft: 10 }}>
        <p style={{ fontWeight: 'bold' }}>Desmond Nyamador</p>
        <p>I just learned an easy way to style React Components</p>
      </div>
    </div>
  );
}
```

**Expected Output:** A user card with a profile picture and bio.

**Why this output:** The static style objects are defined outside the component and reused. Dynamic styles (like `marginLeft`) are defined inline. This keeps the component readable and avoids re-creating static objects on every render .

### Real-World Cases

- **Design systems:** Style objects for component variants.
- **Reusable layouts:** Shared style objects for common layouts.
- **Dynamic styling:** Merging base styles with prop-driven overrides.

### References

- Pluralsight — Inline styling with React — https://www.pluralsight.com/resources/blog/guides/inline-styling-with-react

---

## 4. Style Values

### Definitions

**Core Definition:** Style values are the CSS values assigned to properties in a style object, expressed as either strings or numbers.

**Technical Definition:** In React inline styles, values can be strings (e.g., `'1.5em'`, `'50%'`, `'blue'`) or numbers (e.g., `16`, `0.9`). For numeric values, React automatically appends `px` to properties that accept pixel units, unless the property is unitless. If you need a unit other than `px` (e.g., `em`, `rem`, `%`), you must provide a string with the unit explicitly. The `style` prop documentation states: "You can pass strings or numbers as values. If you pass a number, like `width: 100`, React will automatically append `px` ("pixels") to the value unless it's a unitless property" .

**Beginner-Friendly Explanation:** You can write style values as numbers or strings. If you write a number for something like `width`, React adds `px` for you. If you need `em` or `%`, you write it as a string.

### Purposes

- To provide flexibility in how CSS values are expressed.
- To avoid manually appending `px` for pixel values.
- To support units other than pixels when needed.

### Syntax Rules and Structure

**Value Types:**

| Type | Example | Behaviour |
|------|---------|-----------|
| Number (pixel-accepting) | `width: 100` | → `100px` |
| Number (unitless) | `opacity: 0.9` | → `0.9` (no `px`) |
| String with unit | `padding: '1.5em'` | → `1.5em` |
| String without unit (keywords) | `color: 'blue'` | → `blue` |
| String with unit (percentage) | `width: '50%'` | → `50%` |

**Syntax Rules:**

1. Numeric values receive `px` for pixel-accepting properties.
2. Unitless properties (e.g., `opacity`, `zIndex`, `flex`, `lineHeight`, `fontWeight`) do not receive `px` .
3. For units other than `px`, use a string with the unit: `'1.5em'`, `'50%'` .
4. String values for colors, keywords, and functions are applied as-is.

### Annotated Code Examples

```jsx
function StyleValuesExample() {
  return (
    <div
      style={{
        width: 200,               // → 200px
        height: '50vh',            // → 50vh (string with unit)
        opacity: 0.8,              // → 0.8 (unitless)
        lineHeight: 1.5,           // → 1.5 (unitless)
        padding: '1.5em',          // → 1.5em (string with unit)
        backgroundColor: 'blue',   // → blue (keyword)
      }}
    />
  );
}
```

**Expected Output:** A 200px-wide, 50vh-tall element with 0.8 opacity, 1.5 line height, 1.5em padding, and a blue background.

**Why this output:** `width: 200` receives `px`. `height: '50vh'` uses the string unit. `opacity` and `lineHeight` are unitless. `padding` uses a string with `em`. `backgroundColor` is a keyword string.

### Real-World Cases

- **Responsive sizing:** Using `vh`, `vw`, `%`, or `rem` for dimensions.
- **Typography:** Using unitless `lineHeight` and `fontWeight`.
- **Animation:** Using unitless `opacity` and `zIndex`.

### References

- React — Common Components: style — https://18.react.dev/reference/react-dom/components/common
- Stack Overflow — In a React tutorial, how does React know that the number specified as a dimension is in pixels? — https://stackoverflow.com/questions/71444133/in-a-react-tutorial-how-does-react-know-that-the-number-specified-as-a-dimension

---

## 5. Style camelCase

### Definitions

**Core Definition:** Style camelCase is the convention of writing CSS property names in camelCase (e.g., `backgroundColor`) instead of kebab-case (e.g., `background-color`) in React inline style objects.

**Technical Definition:** React requires CSS property names in the `style` object to be written in camelCase. This matches the property names of the DOM's `style` object (e.g., `element.style.backgroundColor`). Hyphenated CSS property names are not valid JavaScript identifiers, so camelCase is used for consistency and to allow dot notation. The React documentation states: "the CSS property names need to be written as camelCase, for example `fontWeight` instead of `font-weight`" .

**Beginner-Friendly Explanation:** In React inline styles, you write `backgroundColor` instead of `background-color`. You remove the hyphen and capitalize the next letter.

### Purposes

- To conform to JavaScript identifier rules (no hyphens).
- To match the DOM `style` object's property names.
- To enable dot notation and autocompletion in editors.

### Syntax Rules and Structure

**Common Conversions:**

| CSS (kebab-case) | React (camelCase) |
|------------------|-------------------|
| `background-color` | `backgroundColor` |
| `font-size` | `fontSize` |
| `border-radius` | `borderRadius` |
| `margin-top` | `marginTop` |
| `z-index` | `zIndex` |
| `flex-direction` | `flexDirection` |

**Syntax Rules:**

1. Remove hyphens and capitalize the following letter.
2. Vendor prefixes follow special rules (see next section).
3. Custom properties (CSS variables) are the exception — they use the `--` prefix as-is.

### Annotated Code Examples

```jsx
function CamelCaseExample() {
  return (
    <div
      style={{
        backgroundColor: 'blue',      // Not 'background-color'
        fontSize: 16,                 // Not 'font-size'
        borderRadius: 8,              // Not 'border-radius'
        zIndex: 10,                   // Not 'z-index'
      }}
    >
      Content
    </div>
  );
}
```

**Expected Output:** A blue box with 16px font, 8px border radius, and z-index 10.

**Why this output:** All property names are in camelCase, which React requires for the `style` object.

### Real-World Cases

- **All React inline styles:** camelCase is mandatory.
- **CSS-in-JS libraries:** Many libraries also use camelCase object syntax.

### References

- React — Common Components: style — https://18.react.dev/reference/react-dom/components/common
- Codecademy — React Styles Cheatsheet — https://www.codecademy.com/learn/react-101/modules/react-101-jsx-u/cheatsheet

---

## 6. Style Vendor Prefixes

### Definitions

**Core Definition:** Style vendor prefixes are browser-specific prefixes (e.g., `Webkit`, `ms`) added to CSS properties to support experimental or non-standard features across different browsers.

**Technical Definition:** React does not automatically add vendor prefixes to inline styles. You must add them manually using camelCase, with a leading capital letter for most prefixes. The exception is the `ms` prefix, which is the only lowercase vendor prefix. For example, `-webkit-transition` becomes `WebkitTransition`, and `-ms-transition` becomes `msTransition` . The React documentation and community resources confirm: "these styles are not vendor-prefixed automatically, so you have to add vendor prefixes manually" .

**Beginner-Friendly Explanation:** Some CSS features need special prefixes for different browsers, like `-webkit-` for Safari. React does not add these for you. You have to write them yourself in camelCase, like `WebkitTransition`.

### Purposes

- To support browser-specific CSS features in inline styles.
- To ensure compatibility with older browsers that require prefixes.
- To give developers explicit control over vendor prefixes.

### Syntax Rules and Structure

**Vendor Prefix Conversion:**

| CSS (kebab-case) | React (camelCase) | Rule |
|------------------|-------------------|------|
| `-webkit-transition` | `WebkitTransition` | Capital `W` |
| `-webkit-box-shadow` | `WebkitBoxShadow` | Capital `W` |
| `-moz-user-select` | `MozUserSelect` | Capital `M` |
| `-ms-transition` | `msTransition` | Lowercase `ms` (only exception) |

**Syntax Rules:**

1. Vendor prefixes are written in camelCase with a leading capital letter (except `ms`) .
2. The `ms` prefix is the only lowercase vendor prefix .
3. React does not add prefixes automatically — you must include them manually .

**Constraints and Limitations:**

- Manual prefixing is error-prone and increases verbosity.
- Some modern browsers no longer require prefixes for many features.
- Automatic prefixing can be achieved with build tools like Autoprefixer for external CSS, but not for inline styles.

### Annotated Code Examples

```jsx
function VendorPrefixExample() {
  return (
    <div
      style={{
        WebkitTransition: 'all 0.3s ease',   // -webkit-transition
        msTransition: 'all 0.3s ease',       // -ms-transition
        WebkitBoxShadow: '0 0 5px rgba(0,0,0,0.3)',
        userSelect: 'none',                   // Standard property (no prefix)
      }}
    >
      Content
    </div>
  );
}
```

**Expected Output:** A div with transition and box-shadow that work across browsers that require prefixes.

**Why this output:** The vendor prefixes are written manually in camelCase. `WebkitTransition` and `msTransition` correspond to the prefixed CSS properties. `userSelect` is the standard property and does not need a prefix for modern browsers.

### Real-World Cases

- **Legacy browser support:** Ensuring transitions, transforms, and shadows work in older Safari or IE.
- **Experimental CSS features:** Using prefixed properties for cutting-edge CSS.
- **Cross-browser consistency:** Explicitly handling prefix requirements.

### References

- Stack Overflow — How do I apply vendor prefixes to inline styles in reactjs? — https://stackoverflow.com/questions/32183931/how-do-i-apply-vendor-prefixes-to-inline-styles-in-reactjs
- Pluralsight — Inline styling with React — https://www.pluralsight.com/resources/blog/guides/inline-styling-with-react

---

## 7. Inline Style Performance

### Definitions

**Core Definition:** Inline style performance refers to the computational and memory costs associated with creating and applying style objects during React rendering.

**Technical Definition:** Inline styles have a minimal runtime cost compared to CSS-in-JS libraries, but they introduce two performance considerations: (1) style objects created inside the component body are re-created on every render, causing unnecessary object allocations ; and (2) passing a new style object to a memoized child component (`React.memo`) breaks referential equality, causing the child to re-render even when the style values are unchanged . For frequently re-rendering components, the recommendation is to use CSS classes for static styles and `useMemo` for dynamic style objects . Inline styles are "less performant than CSS modules, but still performant enough to be used in most cases" .

**Beginner-Friendly Explanation:** Inline styles are fast enough for most use cases, but they can cause performance problems if you create new style objects on every render or pass them to memoized child components. Use `useMemo` if the style object is dynamic and the component re-renders often.

### Purposes

- To understand the cost of inline styles.
- To avoid unnecessary object allocations.
- To maintain referential stability for memoized components.

### Syntax Rules and Structure

**Performance Considerations:**

| Issue | Impact | Solution |
|-------|--------|----------|
| New object each render | Object allocation, GC pressure | Define static styles outside component; use `useMemo` for dynamic |
| Breaks `React.memo` | Child re-renders unnecessarily | Stabilize style object with `useMemo` |
| Increased HTML size | Larger DOM | Use CSS classes for static styles |

**Syntax Rules:**

1. Define static style objects outside the component to avoid re-creation .
2. For dynamic style objects passed to memoized children, use `useMemo` with the relevant dependencies .
3. Use CSS classes for static styles to reduce bundle and HTML size .

### Annotated Code Examples

**❌ Inline Object Re-created on Every Render:**

```jsx
function TaskItem({ task }) {
  return (
    <div style={{ width: `${task.progress}%` }}>  {/* New object each render */}
      {task.name}
    </div>
  );
}
```

**✅ Memoized Style Object:**

```jsx
import { useMemo } from 'react';

function TaskItem({ task }) {
  const progressStyle = useMemo(
    () => ({ width: `${task.progress}%` }),
    [task.progress]
  );

  return <div style={progressStyle}>{task.name}</div>;
}
```

**Expected Output:** The memoized version avoids creating a new object on every render, reducing allocations.

**Why this output:** `useMemo` caches the style object and only recreates it when `task.progress` changes. This maintains referential stability and prevents unnecessary allocations .

### Real-World Cases

- **Frequently re-rendering lists:** Memoizing style objects for list items.
- **Memoized components:** Passing stable style objects to `React.memo` children.
- **High-frequency updates:** Animations and real-time dashboards.

### References

- Chakra UI — Styling Performance — https://next.chakra-ui.com/guides/styling-performance
- GitHub Issue — Performance: Inline style objects in TaskManagement components — https://github.com/cpa03/ai-first/issues/1076

---

## 8. Inline Style Limitations

### Definitions

**Core Definition:** Inline style limitations are the CSS features and behaviours that cannot be expressed through the `style` prop.

**Technical Definition:** Inline styles only support a subset of CSS. Because they are applied directly to an element's `style` attribute, they cannot express pseudo-classes (`:hover`, `:focus`), pseudo-elements (`::before`, `::after`), media queries (`@media`), keyframe animations (`@keyframes`), or complex selectors . They also cannot express multiple values for the same property (e.g., fallbacks in older CSS). These limitations make inline styles unsuitable for interactive or responsive styling; CSS classes or CSS-in-JS solutions should be used instead .

**Beginner-Friendly Explanation:** Inline styles cannot do everything CSS can do. You cannot use `:hover`, media queries, or animations with inline styles. For those, you need CSS classes.

### Purposes

- To understand when inline styles are insufficient.
- To choose the right styling approach for interactive or responsive design.
- To avoid attempting to implement unsupported features inline.

### Syntax Rules and Structure

**What Inline Styles Cannot Do:**

| Feature | Example | Alternative |
|---------|---------|-------------|
| Pseudo-classes | `:hover`, `:focus` | CSS classes or CSS-in-JS |
| Pseudo-elements | `::before`, `::after` | CSS classes |
| Media queries | `@media (min-width: 768px)` | CSS classes or container queries |
| Keyframe animations | `@keyframes` | CSS classes |
| Fallback values | Multiple values for same property | CSS classes |

**Syntax Rules:**

1. Inline styles cannot express pseudo-classes or pseudo-elements .
2. Inline styles cannot express media queries or container queries.
3. Inline styles cannot define keyframe animations.
4. For these features, use CSS Modules, utility-first CSS, or CSS-in-JS.

### Annotated Code Examples

**❌ Attempting Inline `:hover` (Impossible):**

```jsx
// This does not work — inline styles cannot express :hover
<div style={{ ':hover': { color: 'red' } }}>Hover me</div>
```

**✅ Using CSS Classes for `:hover`:**

```css
/* Button.css */
.button:hover {
  color: red;
}
```

```jsx
import './Button.css';

function Button() {
  return <button className="button">Hover me</button>;
}
```

**Expected Output:** The button turns red on hover.

**Why this output:** The `:hover` pseudo-class is defined in a CSS class, not in an inline style. Inline styles cannot express pseudo-classes .

### Real-World Cases

- **Interactive buttons:** Hover and focus states require CSS classes.
- **Responsive layouts:** Media queries require CSS classes or container queries.
- **Animations:** Keyframe animations require CSS classes.

### References

- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Stack Overflow — What is the actual difference between using an inline-style and using a css-in-js library? — https://stackoverflow.com/questions/49658288/what-is-the-actual-difference-between-using-an-inline-style-and-using-a-css-in-js

---

## 9. Inline Style Best Practices

### Definitions

**Core Definition:** Inline style best practices are guidelines for using the `style` prop effectively, safely, and performantly.

**Technical Definition:** Best practices for inline styles in React include: using inline styles only for dynamic values computed at runtime (not for static styling) ; typing style objects with `React.CSSProperties` in TypeScript ; memoizing dynamic style objects with `useMemo` when passing them to memoized components ; defining static style objects outside the component ; avoiding inline styles for pseudo-classes, media queries, and animations ; and preferring CSS classes, CSS Modules, or utility-first CSS for static and interactive styles.

**Beginner-Friendly Explanation:** Use inline styles only when the styles depend on data that changes. For everything else, use CSS classes. If you use TypeScript, type your style objects with `React.CSSProperties`. If the style object is dynamic and the component re-renders often, use `useMemo`.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use only for dynamic values** | Inline styles are for runtime-computed values, not static styling . |
| **Type with `React.CSSProperties`** | Ensures type safety and autocompletion in TypeScript . |
| **Memoize dynamic objects** | Use `useMemo` when passing style objects to memoized components . |
| **Define static objects outside** | Avoid re-creating static style objects on every render . |
| **Never use for pseudo-classes/media queries** | Use CSS classes for interactive and responsive styles . |
| **Prefer CSS classes for static styles** | Reduces bundle size and improves maintainability . |
| **Use CSS custom properties for theming** | Better performance and flexibility than inline styles for dynamic theming. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
import { useMemo } from 'react';
import type { CSSProperties } from 'react';

// Static style object outside the component
const staticCardStyle: CSSProperties = {
  padding: 16,
  borderRadius: 8,
  boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
};

function Card({ percent }: { percent: number }) {
  // Dynamic style object memoized
  const progressStyle = useMemo<CSSProperties>(
    () => ({ width: `${percent}%` }),
    [percent]
  );

  return (
    <div style={staticCardStyle}>
      <div style={progressStyle} />
    </div>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ Inline style for static styling
<div style={{ padding: 16, borderRadius: 8 }} />

// ❌ Inline style for :hover (does not work)
<div style={{ ':hover': { color: 'red' } }} />

// ❌ New style object every render passed to memoized child
<MemoizedChild style={{ width: percent }} />
```

**Expected Output:** The good practices produce type-safe, performant, and maintainable code.

**Why this output:** Static styles are outside the component. Dynamic styles are memoized. TypeScript types ensure correctness. Pseudo-classes are handled by CSS classes .

### Real-World Cases

- **Design systems:** Using `React.CSSProperties` for component style props.
- **Performance-sensitive lists:** Memoizing dynamic style objects.
- **Theming:** Using CSS custom properties instead of inline styles for better performance.

### References

- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Steve Kinney — Styling Csstype And Css Modules — https://stevekinney.com/courses/react-typescript/styling-csstype-and-css-modules
- RT Camp — Styling Best Practice for React-based projects — https://rtcamp.com

---

## References

- React — Common Components: style — https://18.react.dev/reference/react-dom/components/common
- Pluralsight — Inline styling with React — https://www.pluralsight.com/resources/blog/guides/inline-styling-with-react
- Chakra UI — Styling Performance — https://next.chakra-ui.com/guides/styling-performance
- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Steve Kinney — Styling Csstype And Css Modules — https://stevekinney.com/courses/react-typescript/styling-csstype-and-css-modules
- GitHub Issue — Performance: Inline style objects — https://github.com/cpa03/ai-first/issues/1076
- Stack Overflow — How do I apply vendor prefixes to inline styles in reactjs? — https://stackoverflow.com/questions/32183931/how-do-i-apply-vendor-prefixes-to-inline-styles-in-reactjs
- Stack Overflow — What is the actual difference between using an inline-style and using a css-in-js library? — https://stackoverflow.com/questions/49658288/what-is-the-actual-difference-between-using-an-inline-style-and-using-a-css-in-js
- Stack Overflow — In a React tutorial, how does React know that the number specified as a dimension is in pixels? — https://stackoverflow.com/questions/71444133/in-a-react-tutorial-how-does-react-know-that-the-number-specified-as-a-dimension
- RT Camp — Styling Best Practice for React-based projects — https://rtcamp.com
- Codecademy — React Styles Cheatsheet — https://www.codecademy.com/learn/react-101/modules/react-101-jsx-u/cheatsheet