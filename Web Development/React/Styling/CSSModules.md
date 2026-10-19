# CSS Modules — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** CSS Modules are CSS files in which all class names and animation names are scoped locally by default, preventing style collisions across components.

**Technical Definition:** A CSS Module is a CSS file (conventionally named with the `.module.css` extension) where all class names and animation names are scoped locally by default. When imported from a JavaScript module, a CSS Module exports an object containing all mappings from local names to generated global names. The build tool (Vite, webpack, Next.js) transforms local class names into unique hashed identifiers at build time, ensuring that `.button` in one module never conflicts with `.button` in another. CSS Modules compile to ICSS (Interoperable CSS) but are written like normal CSS files. Global scope can be explicitly accessed using the `:global` pseudo-class, and local scope can be re-established with `:local`.

**Beginner-Friendly Explanation:** CSS Modules are like regular CSS files, but with a superpower: every class name you write is automatically made unique. If you write `.button` in one file and `.button` in another, they will never clash because the build tool renames them to something like `Button_button__a1b2c` and `Card_button__x9y8z`. You write normal CSS, import the file into your component, and access classes through the imported `styles` object. It is the safest, most performance-friendly way to style React components because it has zero runtime cost and works everywhere — including React Server Components.

### Key Characteristics

- **Local by default:** All class names and animation names are scoped to the file unless explicitly marked global.
- **Unique hashed names:** The build tool generates unique identifiers at build time, eliminating naming collisions.
- **Exported mapping object:** Importing a CSS Module returns an object mapping original class names to generated names.
- **Zero runtime cost:** CSS Modules are compiled to static CSS at build time; no JavaScript runs to generate styles.
- **RSC-compatible:** Because there is no runtime, CSS Modules work in React Server Components.
- **Composition support:** Classes can be composed from other classes within the same file or from other modules.
- **`:global` and `:local`:** The scope can be switched explicitly when interoperating with global CSS or third-party libraries.

### Prerequisites

- Basic CSS syntax (selectors, properties, values).
- React function components and JSX.
- Understanding of the `className` prop.
- A build tool that supports CSS Modules (Vite, Next.js, Create React App, or webpack).

### Related Programming Areas

- **CSS architecture** — BEM, SMACSS, OOCSS naming conventions.
- **CSS-in-JS** — runtime and compile-time styling alternatives.
- **Utility-first CSS** — Tailwind and similar frameworks.
- **React Server Components** — CSS Modules are the recommended styling approach for RSC.
- **TypeScript** — type declarations for module imports.

---

## 1. CSS Modules

### Definitions

**Core Definition:** A CSS Module is a CSS file in which all class names and animation names are scoped locally by default.

**Technical Definition:** A CSS Module is a CSS file where all class names and animation names are scoped locally by default. All URLs (`url(...)`) and `@imports` are in module request format (`./xxx` and `../xxx` mean relative, `xxx` and `xxx/yyy` mean in the `node_modules` folder). CSS Modules compile to ICSS (Interoperable CSS) but are written like normal CSS files. When importing the CSS Module from a JavaScript module, it exports an object with all mappings from local names to global names. The React ecosystem supports CSS Modules out of the box in frameworks like Next.js, which has built-in support for `.module.css` files.

**Beginner-Friendly Explanation:** A CSS Module is just a CSS file with a special extension (`.module.css`). You write normal CSS in it, but when you import it into your component, the class names become properties of an object. You use those properties with `className` instead of writing string class names. This keeps your styles scoped and collision-free.

### Purposes

- To scope CSS class names locally and prevent style leakage across components.
- To enable component-level styling without naming collisions.
- To provide a zero-runtime styling solution compatible with React Server Components.
- To leverage the full power of CSS (pseudo-classes, media queries, animations) in a scoped manner.
- To create explicit, traceable dependencies between components and their styles.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import styles from './Component.module.css';

function Component() {
  return <div className={styles.container}>Content</div>;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `./Component.module.css` | The CSS Module file; the `.module.css` extension is required for scoping. |
| `styles` | The imported object mapping local class names to generated global names. |
| `styles.container` | The generated unique class name for `.container`. |

**Syntax Rules:**

1. File names must end with `.module.css` (or `.module.scss`, `.module.less`) for the build tool to apply scoping.
2. Import the module as a default import: `import styles from './Component.module.css'`.
3. Access classes as properties of the imported object: `styles.className`.
4. Classes can be combined using template literals or utility libraries like `clsx`: `` `${styles.a} ${styles.b}` ``.
5. Class names in CSS are written normally (e.g., `.container`, `.button-primary`).

**Constraints and Limitations:**

- Class names cannot be constructed dynamically (e.g., `styles[`btn-${variant}`]`) without additional tooling; use explicit mapping or `clsx`.
- Global styles (resets, design tokens) still require a separate global stylesheet.
- TypeScript requires module declarations for type safety.
- CSS Modules add a slight build-time overhead compared to plain global CSS, but zero runtime cost.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Scoped Button

**`Button.module.css`:**

```css
.button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primary {
  background: blue;
  color: white;
}
```

**`Button.jsx`:**

```jsx
import styles from './Button.module.css';

function Button({ children }) {
  return (
    <button className={`${styles.button} ${styles.primary}`}>
      {children}
    </button>
  );
}
```

**Expected Output:** A blue button with padding and rounded corners. The rendered HTML class name is a unique hash (e.g., `Button_button__a1b2c Button_primary__x9y8z`).

**Why this output:** The build tool transforms `.button` and `.primary` into unique class names and exports the mapping. The component retrieves the generated names via `styles.button` and `styles.primary`, ensuring no collision with other `.button` or `.primary` classes elsewhere.

#### Example 2: Using `clsx` for Conditional Classes

```jsx
import clsx from 'clsx';
import styles from './Alert.module.css';

function Alert({ type, children }) {
  return (
    <div className={clsx(styles.alert, styles[type])}>
      {children}
    </div>
  );
}
```

**`Alert.module.css`:**

```css
.alert {
  padding: 12px;
  border-radius: 4px;
}

.info {
  background: #e0f2fe;
}

.error {
  background: #fee2e2;
}
```

**Expected Output:** An alert div with the base `alert` class and the appropriate type class (`info` or `error`).

**Why this output:** `clsx` combines `styles.alert` with `styles[type]`, where `type` is `'info'` or `'error'`. The CSS Module mapping provides the generated names for each class.

### Real-World Cases

- **Design systems:** Component libraries use CSS Modules to scope styles per component.
- **Large applications:** Multiple teams can use the same class names without conflicts.
- **Server-rendered apps:** CSS Modules work seamlessly with React Server Components because they have zero runtime cost.
- **Teams familiar with CSS:** CSS Modules require minimal learning for developers who already know CSS.

### References

- CSS Modules — Official README — https://github.com/css-modules/css-modules
- Next.js — CSS Modules — https://nextjs.org/docs/14/app/building-your-application/styling/css-modules
- Bun — CSS Modules — https://bun.sh/docs/bundler/css-modules

---

## 2. Module Scoping

### Definitions

**Core Definition:** Module scoping is the mechanism by which CSS Modules automatically rename class names to unique identifiers, confining their effect to the module where they are defined.

**Technical Definition:** Module scoping is achieved at build time. The bundler (Vite, webpack, Next.js) transforms every local class name in a `.module.css` file into a unique string, typically a hash combined with the original name and file name (e.g., `Button_button__a1b2c`). The generated CSS file contains these unique names, and the imported `styles` object maps the original names to the generated ones. Because the generated names are unique, two different modules can both define `.button` without any conflict. This is the core innovation of CSS Modules: local scope by default.

**Beginner-Friendly Explanation:** Module scoping means the build tool renames your classes so they cannot clash. When you write `.button`, the tool turns it into something like `.Button_button__a1b2c`. Nobody else has that exact name, so your styles stay where you put them.

### Purposes

- To eliminate global class name collisions.
- To confine styles to the component that imports the module.
- To enable multiple components to use the same class names safely.
- To make style dependencies explicit and traceable.

### Syntax Rules and Structure

**How Scoping Works:**

| Step | What Happens |
|------|--------------|
| 1 | You write `.button` in `Button.module.css`. |
| 2 | The build tool renames it to `Button_button__a1b2c`. |
| 3 | The generated CSS contains `.Button_button__a1b2c`. |
| 4 | The imported `styles` object maps `button` → `Button_button__a1b2c`. |
| 5 | The component uses `styles.button` to apply the unique class. |

**Syntax Rules:**

1. Scoping is automatic for all class names in `.module.css` files.
2. No naming conventions (like BEM) are required to avoid collisions.
3. The generated names are deterministic for a given file and class name.
4. Scoping applies to class selectors and animation names; ID selectors are not scoped by default.

**Constraints and Limitations:**

- Scoping does not apply to element selectors (e.g., `div`, `p`); those remain global unless nested inside a scoped class.
- Global styles must still be handled separately.

### Annotated Code Examples

**`Button.module.css`:**

```css
.button {
  padding: 8px 16px;
}
```

**`Card.module.css`:**

```css
.button {
  border: 1px solid #ddd;
}
```

**`Button.jsx`:**

```jsx
import styles from './Button.module.css';
// styles.button = "Button_button__a1b2c"

function Button() {
  return <button className={styles.button}>Click</button>;
}
```

**`Card.jsx`:**

```jsx
import styles from './Card.module.css';
// styles.button = "Card_button__x9y8z"

function CardButton() {
  return <button className={styles.button}>Card Button</button>;
}
```

**Expected Output:** Two different buttons with different generated class names and different styles, despite both using `.button` in their CSS files.

**Why this output:** The build tool generates unique names based on the file name and class name. `Button.module.css` and `Card.module.css` produce different hashes, so the `.button` styles do not conflict.

### Real-World Cases

- **Component libraries:** Each component has its own `.module.css` file with no risk of global collisions.
- **Monorepos:** Packages can define the same class names without coordination.
- **Third-party integrations:** Scoped styles do not accidentally override library styles.

### References

- Next.js — CSS Modules: Locally scoped CSS — https://nextjs.org/docs/14/app/building-your-application/styling/css-modules
- CSS Modules — Why CSS Modules? — https://github.com/css-modules/css-modules

---

## 3. Local Class Names

### Definitions

**Core Definition:** Local class names are CSS class names defined inside a CSS Module that are automatically scoped to that module and exported through the `styles` object.

**Technical Definition:** By default, every class name in a CSS Module is **local**. This means the build tool renames it to a unique identifier and includes it in the exported mapping object. Local class names are the primary mechanism for component-scoped styling. They can be composed with other local classes using the `composes` property, and they can be nested or combined with pseudo-classes. The `:local` pseudo-class can be used to explicitly declare a class as local, although this is rarely necessary since local is the default.

**Beginner-Friendly Explanation:** Local class names are the classes you write in your CSS Module that get scoped automatically. They are the normal way you style components with CSS Modules.

### Purposes

- To define styles that are scoped to a single component.
- To enable safe reuse of class names across different modules.
- To provide the mapping entries in the imported `styles` object.
- To compose styles from multiple local classes.

### Syntax Rules and Structure

**Local by Default:**

```css
/* Button.module.css */
.button {          /* Local class */
  padding: 8px;
}

.primary {         /* Local class */
  background: blue;
}
```

**Explicit Local:**

```css
:local(.button) {  /* Explicitly local */
  padding: 8px;
}
```

**Syntax Rules:**

1. All class names in a `.module.css` file are local by default.
2. `:local(.className)` explicitly marks a class as local (rarely needed).
3. Local classes are accessible via `styles.className` after import.
4. Local classes can be composed using `composes: otherLocalClass;`.

### Annotated Code Examples

**`Card.module.css`:**

```css
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
}

.title {
  font-size: 1.25rem;
  font-weight: bold;
}
```

**`Card.jsx`:**

```jsx
import styles from './Card.module.css';

function Card({ title, children }) {
  return (
    <div className={styles.card}>
      <h3 className={styles.title}>{title}</h3>
      {children}
    </div>
  );
}
```

**Expected Output:** A card with a border, rounded corners, padding, and a bold title. The HTML class names are unique hashes.

**Why this output:** `.card` and `.title` are local classes. The build tool renames them and exports the mapping. The component uses `styles.card` and `styles.title` to apply the generated names.

### Real-World Cases

- **All components using CSS Modules:** Local class names are the foundation of scoped styling.
- **Design system primitives:** Buttons, cards, and inputs define local classes for their styles.

### References

- CSS Modules — Local Scope — https://github.com/css-modules/css-modules/blob/master/docs/local-scope.md
- Rspack — CSS Modules: Local scope by default — https://rspack.rs

---

## 4. Global Class Names

### Definitions

**Core Definition:** Global class names are CSS class names that are not scoped by CSS Modules and apply to the entire document, accessed via the `:global` pseudo-class.

**Technical Definition:** CSS Modules provides the `:global` pseudo-class to switch a selector or identifier into global scope. `:global(.xxx)` declares the class `.xxx` in the global scope, meaning it is not renamed and can target elements outside the module. This is essential for interacting with third-party libraries that use global class names (e.g., Material UI's `.Mui-selected`) or for writing global styles (resets, base typography) within a module. The `:global` block can also contain multiple rules, and `:local` can be used inside a `:global` block to switch back to local scope.

**Beginner-Friendly Explanation:** Global class names are classes that are not scoped. You use `:global` to tell CSS Modules: "Do not rename this class; let it apply everywhere." This is useful for overriding third-party library styles or writing global styles.

### Purposes

- To target global class names used by third-party libraries.
- To define global styles within a CSS Module file.
- To override styles from global CSS frameworks.
- To interoperate with legacy CSS that uses global class names.

### Syntax Rules and Structure

**Global Class Selector:**

```css
:global(.global-class) {
  color: red;
}
```

**Global Block:**

```css
:global {
  .global-class {
    color: red;
  }
  .another-global {
    color: blue;
  }
}
```

**Switching Back to Local:**

```css
:global .global-parent :local(.local-child) {
  color: green;
}
```

**Syntax Rules:**

1. `:global(.className)` marks the selector as global.
2. `:global { ... }` wraps multiple global rules.
3. Inside a `:global` block, `:local(.className)` switches back to local scope.
4. Global class names are not included in the `styles` mapping object.

### Annotated Code Examples

**Targeting a Third-Party Global Class:**

```css
/* Override Material UI's selected state */
:global(.Mui-selected) {
  background-color: #e0f2fe;
}
```

**Combining Global and Local:**

```css
.card :global(.MuiButton-root) {
  margin-top: 8px;
}
```

**`Card.jsx`:**

```jsx
import styles from './Card.module.css';

function Card() {
  return (
    <div className={styles.card}>
      <button className="MuiButton-root">Click</button>
    </div>
  );
}
```

**Expected Output:** The Material UI button inside the card receives a top margin. The global `.MuiButton-root` class is targeted without being renamed.

**Why this output:** The `:global(.MuiButton-root)` selector targets the global class, allowing the CSS Module to style third-party components that use global class names.

### Real-World Cases

- **Third-party library integration:** Overriding Material UI, Bootstrap, or other global class names.
- **Global resets:** Adding global reset rules inside a module.
- **Legacy CSS migration:** Gradually scoping legacy global CSS.

### References

- CSS Modules — Exceptions: `:global` — https://github.com/css-modules/css-modules
- Material UI — Targeting State Classes with CSS Modules — https://mui.com/material-ui/integrations/interoperability/
- Rspack — Global scope with `:global` — https://rspack.rs

---

## 5. Composing Classes

### Definitions

**Core Definition:** Composing classes is the CSS Modules feature that allows one class to include the styles of another class using the `composes` property.

**Technical Definition:** The `composes` property allows a class to inherit all the styles of one or more other classes. When a class composes another class, the CSS Module exports both class names for the local class. This can result in multiple class names being applied to a single element. `composes` rules must come before other rules in the class, can compose multiple classes in one declaration (`composes: classNameA classNameB;`), and can compose classes from other CSS Modules (`composes: className from "./style.css";`). Composition only works for locally scoped simple class selectors (single class name).

**Beginner-Friendly Explanation:** Composing classes lets you reuse styles by saying "this class also includes that class." For example, a `.primary-button` class can compose `.button` to get all the base button styles, then add its own styles on top.

### Purposes

- To reuse style rules across multiple classes without duplicating CSS.
- To create modifier classes that extend a base class.
- To share styles across different CSS Modules.
- To keep CSS DRY and maintainable.

### Syntax Rules and Structure

**Composing Within the Same File:**

```css
.base {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
}

.primary {
  composes: base;
  background: blue;
  color: white;
}
```

**Composing Multiple Classes:**

```css
.combined {
  composes: base primary;
}
```

**Composing from Another Module:**

```css
.other {
  composes: base from "./shared.module.css";
}
```

**Syntax Rules:**

1. `composes` must appear before any regular CSS properties in the class.
2. Composition only works for single-class simple selectors (e.g., `.base`), not complex selectors.
3. Multiple classes can be composed in one declaration, separated by spaces.
4. Composition from other files uses the syntax `composes: className from "./file.css"`.
5. When composing, the CSS Module exports both the composed class names and the local class name.

### Annotated Code Examples

**`Button.module.css`:**

```css
.base {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primary {
  composes: base;
  background: blue;
  color: white;
}

.secondary {
  composes: base;
  background: gray;
  color: black;
}
```

**`Button.jsx`:**

```jsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children }) {
  const className = variant === 'primary'
    ? styles.primary
    : styles.secondary;

  return <button className={className}>{children}</button>;
}
```

**Expected Output:** Primary and secondary buttons both have the base padding, border removal, and border radius. Primary is blue; secondary is gray.

**Why this output:** `.primary` and `.secondary` both compose `.base`, inheriting its styles. The CSS Module exports the combined class names, so `styles.primary` includes both `base` and `primary` class names.

**Composing from Another Module:**

**`shared.module.css`:**

```css
.clickable {
  cursor: pointer;
  user-select: none;
}
```

**`Card.module.css`:**

```css
.cardButton {
  composes: clickable from "./shared.module.css";
  padding: 12px;
  border: 1px solid #ddd;
}
```

**Expected Output:** The card button has the clickable cursor and user-select behavior from the shared module, plus its own padding and border.

**Why this output:** `composes: clickable from "./shared.module.css"` imports the `.clickable` class from the shared module and applies it alongside `.cardButton`.

### Real-World Cases

- **Design systems:** Base classes composed into modifier classes for buttons, alerts, and badges.
- **Shared utilities:** Common styles (clickable, visually-hidden) composed across modules.
- **Theming:** Base component styles composed with theme-specific modifiers.

### References

- CSS Modules — Composition — https://github.com/css-modules/css-modules/blob/master/docs/composition.md
- Lightning CSS — CSS Modules: Class composition — https://lightningcss.dev

---

## 6. CSS Modules with TypeScript

### Definitions

**Core Definition:** CSS Modules with TypeScript involves providing type declarations for `.module.css` imports so that TypeScript understands the shape of the imported `styles` object.

**Technical Definition:** TypeScript does not natively know how to handle `.module.css` imports. Without type declarations, importing a CSS Module results in a "Cannot find module" error. The solution is to create a declaration file (e.g., `types/css-modules.d.ts`) that tells TypeScript the module exports an object with string keys and string values. For stronger type safety, tools like `typed-css-modules` or `vite-plugin-css-module-types` can generate specific `.d.ts` files for each CSS Module, providing autocompletion and validation for actual class names.

**Beginner-Friendly Explanation:** TypeScript needs to be told what a CSS Module looks like. You create a small file that says "any `.module.css` file exports an object of class names." Then TypeScript stops complaining and gives you autocomplete for your class names.

### Purposes

- To prevent TypeScript "Cannot find module" errors when importing CSS Modules.
- To provide autocompletion for CSS class names in the editor.
- To catch typos in class names at compile time.
- To improve developer experience and code reliability.

### Syntax Rules and Structure

**Basic Declaration (Global):**

```ts
// types/css-modules.d.ts
declare module '*.module.css' {
  const classes: { [key: string]: string };
  export default classes;
}
```

**Generated Declaration (Specific):**

```ts
// Button.module.css.d.ts (auto-generated)
declare const styles: {
  readonly button: string;
  readonly primary: string;
  readonly secondary: string;
};
export default styles;
```

**Syntax Rules:**

1. Create a `types/css-modules.d.ts` (or similar) file in your project.
2. Use `declare module '*.module.css'` to declare the module shape.
3. The default export is an object whose keys are class names and whose values are strings.
4. For per-file types, use a generator tool like `typed-css-modules`.
5. Vite projects can use `vite-plugin-css-module-types` for automatic declarations.

### Annotated Code Examples

**Basic Declaration:**

```ts
// src/types/css-modules.d.ts
declare module '*.module.css' {
  const classes: { readonly [key: string]: string };
  export default classes;
}
```

**Component:**

```tsx
import styles from './Button.module.css';

function Button() {
  return <button className={styles.primary}>Click</button>;
  // TypeScript accepts styles.primary as string
}
```

**Generated Declaration with `typed-css-modules`:**

```bash
npx typed-css-modules src/**/*.module.css
```

**Generated `Button.module.css.d.ts`:**

```ts
declare const styles: {
  readonly button: string;
  readonly primary: string;
  readonly secondary: string;
};
export default styles;
```

**Expected Output:** TypeScript knows the exact class names available in `Button.module.css`. Typing `styles.primry` (typo) causes a compile-time error.

**Why this output:** The generated declaration file lists the actual class names. TypeScript checks property access against this type, catching typos and providing autocompletion.

### Real-World Cases

- **TypeScript React projects:** All CSS Modules usage requires declarations.
- **Design systems:** Generated declarations ensure consumers use the correct class names.
- **Large codebases:** Type safety prevents class name typos that would silently produce broken styles.

### References

- Steve Kinney — Modules and Declaration Files for React: CSS Modules — https://stevekinney.com/courses/react-typescript/typescript-modules-declarations
- typed-css-modules — npm — https://www.npmjs.com/package/typed-css-modules
- vite-plugin-css-module-types — npm — https://www.npmjs.com/package/vite-plugin-css-module-types

---

## 7. CSS Modules Best Practices

### Definitions

**Core Definition:** CSS Modules best practices are guidelines for using CSS Modules effectively, safely, and performantly in React applications.

**Technical Definition:** Best practices for CSS Modules include: using CSS Modules as the default styling approach for component-scoped styles because they are the safest, zero-runtime, and best-performing option; always using the `.module.css` extension to ensure scoping; defining TypeScript declarations for type safety; using `composes` to reuse styles rather than duplicating CSS; using `clsx` or `cn` for conditional class composition instead of string interpolation; avoiding global CSS for component styles; and leveraging CSS custom properties for theming within modules.

**Beginner-Friendly Explanation:** Best practices are the habits that keep your CSS Modules clean and bug-free. They include always using the `.module.css` extension, setting up TypeScript declarations, using `composes` to reuse styles, and using a helper like `clsx` for conditional classes.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use CSS Modules by default** | The safest, zero-runtime, best-performing styling approach. |
| **Always use `.module.css`** | The extension is required for scoping. |
| **Define TypeScript declarations** | Prevent import errors and enable autocomplete. |
| **Use `composes` for reuse** | Avoid duplicating CSS by composing base classes. |
| **Use `clsx` or `cn`** | Never use string interpolation for conditional classes. |
| **Keep global styles separate** | Use a global stylesheet for resets and design tokens. |
| **Use `:global` sparingly** | Only for third-party integration or deliberate global overrides. |
| **Prefer CSS custom properties for theming** | Works well with CSS Modules and avoids runtime cost. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
import clsx from 'clsx';
import styles from './Button.module.css';

function Button({ variant = 'primary', disabled }) {
  return (
    <button
      className={clsx(
        styles.button,
        styles[variant],
        { [styles.disabled]: disabled }
      )}
      disabled={disabled}
    >
      Click
    </button>
  );
}
```

**`Button.module.css`:**

```css
.button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
}

.primary {
  composes: button;
  background: blue;
  color: white;
}

.secondary {
  composes: button;
  background: gray;
  color: white;
}

.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

**❌ Bad Practices:**

```jsx
// ❌ String interpolation for conditional classes
<div className={`${styles.button} ${disabled ? styles.disabled : ''}`} />

// ❌ Plain .css import (global, not scoped)
import './Button.css';

// ❌ No TypeScript declaration
import styles from './Button.module.css'; // TypeScript error

// ❌ Global class for component-specific style
:global(.my-button) { padding: 8px; }
```

**Expected Output:** The good practices produce type-safe, scoped, and maintainable styling.

**Why this output:** `clsx` handles conditional classes cleanly. `composes` avoids duplication. The `.module.css` extension ensures scoping. TypeScript declarations prevent import errors. Global classes are reserved for deliberate global styling.

### Real-World Cases

- **All React applications using CSS Modules:** These practices apply universally.
- **Design systems:** CSS Modules with `composes` and TypeScript declarations are ideal for component libraries.
- **Server-rendered applications:** Zero-runtime CSS Modules are compatible with React Server Components.

### References

- Impertio Studio — React Implementation Styling: CSS Modules (Primary Recommendation) — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- CSS Modules — Official README: Why CSS Modules? — https://github.com/css-modules/css-modules
- Next.js — CSS Modules: Global Styles — https://nextjs.org/docs/14/app/building-your-application/styling/css-modules

---

## References

- CSS Modules — Official README — https://github.com/css-modules/css-modules
- CSS Modules — Composition — https://github.com/css-modules/css-modules/blob/master/docs/composition.md
- CSS Modules — Local Scope — https://github.com/css-modules/css-modules/blob/master/docs/local-scope.md
- Next.js — CSS Modules — https://nextjs.org/docs/14/app/building-your-application/styling/css-modules
- Steve Kinney — Modules and Declaration Files for React: CSS Modules — https://stevekinney.com/courses/react-typescript/typescript-modules-declarations
- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- Lightning CSS — CSS Modules — https://lightningcss.dev
- Rspack — CSS Modules — https://rspack.rs
- Material UI — Targeting State Classes with CSS Modules — https://mui.com/material-ui/integrations/interoperability/
- typed-css-modules — npm — https://www.npmjs.com/package/typed-css-modules
- vite-plugin-css-module-types — npm — https://www.npmjs.com/package/vite-plugin-css-module-types
- GitHub — Styling in React: A Comparison — https://github.com/mayaaedlund/styling-in-react
- GitNation — Rethinking React Styling for the RSC Era — https://gitnation.com
- Bun — CSS Modules — https://bun.sh/docs/bundler/css-modules