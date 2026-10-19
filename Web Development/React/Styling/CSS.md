# React CSS Stylesheets — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** CSS stylesheets in React are external `.css` files that define global styling rules, applied to components via the `className` prop.

**Technical Definition:** In React, CSS stylesheets are imported into component modules (e.g., `import './Button.css'`) and bundled by the build tool. The styles are **global by default** — a class defined in one stylesheet applies to any element with that class name anywhere in the application. React uses `className` instead of `class` because `class` is a reserved JavaScript keyword. Stylesheets cascade according to the CSS cascade algorithm, which resolves conflicts through origin, layer, specificity, and order of appearance. For component-scoped styling, CSS Modules (`.module.css`) are the recommended alternative to global stylesheets.

**Beginner-Friendly Explanation:** CSS stylesheets are regular CSS files that you import into your React components. Once imported, the styles become global — any component can use the class names you defined. This is the traditional way of styling websites, and React supports it fully. You use `className` instead of `class` because React is JavaScript-based, and `class` is a special word in JavaScript. The main thing to remember is that these styles are not scoped to a single component unless you use a special tool like CSS Modules.

### Key Characteristics

- **Global scope by default:** Any imported CSS affects the entire application unless scoped with CSS Modules.
- **`className` instead of `class`:** React uses `className` because `class` is a reserved JavaScript keyword.
- **Cascade-driven:** Styles are applied according to CSS's cascade algorithm, considering origin, specificity, and order.
- **Import-based:** CSS files are imported into component modules and bundled by the build tool.
- **Zero runtime cost:** Unlike runtime CSS-in-JS, stylesheets are static and do not add per-render JavaScript overhead.
- **Compatible with Server Components:** Global CSS and CSS Modules work in React Server Components (RSC), unlike runtime CSS-in-JS.

### Prerequisites

- Basic CSS syntax (selectors, properties, values).
- React function components and JSX.
- Understanding of the `className` prop.
- Familiarity with ES modules (`import`).

### Related Programming Areas

- **CSS Modules** — component-scoped alternatives to global stylesheets.
- **CSS cascade and specificity** — the rules that determine which style wins.
- **CSS naming conventions** — BEM, SMACSS, OOCSS for maintainable styles.
- **Utility-first CSS** — Tailwind and similar frameworks.
- **CSS-in-JS** — runtime and compile-time styling solutions.

---

## 1. CSS Stylesheets

### Definitions

**Core Definition:** A CSS stylesheet is an external `.css` file containing styling rules that are applied to React elements via the `className` prop.

**Technical Definition:** A CSS stylesheet is a text file with a `.css` extension containing CSS rules. In React, it is imported into a JavaScript module using a side-effect import: `import './styles.css'`. The build tool (Vite, webpack, Next.js) processes the CSS and includes it in the application bundle. The rules are global — they apply to any matching element in the document, regardless of which component imported the file. CSS stylesheets support the full CSS feature set: selectors, pseudo-classes, media queries, keyframe animations, and cascade layers.

**Beginner-Friendly Explanation:** A CSS stylesheet is a regular CSS file. You write your styles in it, import it into your React component, and then use the class names with `className`. The styles apply everywhere in your app.

### Purposes

- To define global styles (resets, typography, base element styles).
- To apply static styling that does not change based on component state or props.
- To leverage the full power of CSS, including pseudo-classes and media queries.
- To separate styling concerns from component logic in a familiar, standards-based way.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import './Component.css';

function Component() {
  return <div className="my-class">Content</div>;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `import './Component.css'` | Side-effect import that includes the stylesheet in the bundle. |
| `className="my-class"` | Applies the CSS class to the element. |

**Syntax Rules:**

1. Import the CSS file at the top of the component module.
2. Use `className` (not `class`) to apply classes.
3. Class names are global; use a naming convention to avoid collisions.
4. Multiple classes are separated by spaces: `className="btn btn-primary"`.

**Constraints and Limitations:**

- Global scope can cause naming collisions in large applications.
- Styles cannot be dynamically computed without adding or removing classes.
- Unused CSS is not automatically tree-shaken unless the build tool supports it.
- `@import` from external URLs is render-blocking and can cause flash of unstyled content (FOUC).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Global Stylesheet

**`Button.css`:**

```css
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  background: blue;
  color: white;
}
```

**`Button.jsx`:**

```jsx
import './Button.css';

function Button({ variant = 'primary', children }) {
  return (
    <button className={`btn btn-${variant}`}>
      {children}
    </button>
  );
}
```

**Expected Output:** A blue primary button with padding and rounded corners.

**Why this output:** The imported CSS file defines `.btn` and `.btn-primary`. The template literal combines them into `"btn btn-primary"`, applying both rules.

#### Example 2: Global Reset Stylesheet

**`global.css`:**

```css
/* Reset */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: system-ui, sans-serif;
  line-height: 1.5;
  color: #333;
}
```

**`layout.tsx` (Next.js):**

```tsx
import './global.css';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

**Expected Output:** All elements use border-box sizing, zero margins, and the system font.

**Why this output:** The global stylesheet is imported at the root layout, applying the reset and base typography to every route in the application.

### Real-World Cases

- **Application resets:** Normalising browser default styles.
- **Base typography:** Setting font families, sizes, and line heights globally.
- **Design tokens:** Defining CSS custom properties for colors and spacing.
- **Third-party CSS:** Importing Bootstrap or other CSS libraries from packages.

### References

- Next.js — CSS Styling — https://nextjs.org/learn/dashboard-app/css-styling
- Vercel — NO_EXTERNAL_CSS_AT_IMPORTS — https://vercel.com/docs/conformance/rules/NO_EXTERNAL_CSS_AT_IMPORTS
- TanStack — CSS Styling — https://tanstack.com/router/latest/docs/start/framework/react/guide/css-styling

---

## 2. Importing CSS

### Definitions

**Core Definition:** Importing CSS is the act of including a `.css` file in a JavaScript module so its rules become part of the application bundle.

**Technical Definition:** CSS is imported using a side-effect import: `import './styles.css'`. The build tool processes the import, resolves the file path, and includes the CSS in the final bundle. In frameworks like Next.js, global CSS can be imported in the root layout (`app/layout.tsx`) to apply to all routes. For component-scoped styles, CSS Modules use a default import: `import styles from './Component.module.css'`. Relative imports are processed by the framework and avoid the render-blocking issues of external `@import` URLs.

**Beginner-Friendly Explanation:** Importing CSS means telling your component to use a CSS file. You write `import './styles.css'` at the top of your file, and the styles become available.

### Purposes

- To include stylesheet rules in the application bundle.
- To apply global or component-scoped styles.
- To load third-party CSS from packages or local files.

### Syntax Rules and Structure

**Global CSS Import:**

```jsx
import './global.css';
```

**CSS Module Import:**

```jsx
import styles from './Component.module.css';
```

**Third-Party CSS Import:**

```jsx
import 'bootstrap/dist/css/bootstrap.css';
```

**Syntax Rules:**

1. Global CSS uses a side-effect import (no variable assigned).
2. CSS Modules use a default import to access the class name mapping.
3. Third-party CSS can be imported from `node_modules` packages.
4. Relative imports are processed by the build tool and are safe from render-blocking issues.

**Constraints and Limitations:**

- External `@import` URLs are render-blocking and can cause FOUC.
- Global CSS imported in a component applies to the entire application, not just that component.
- In React Server Components, page-specific global CSS should be avoided in broad `'use client'` entry points.

### Annotated Code Examples

```jsx
// Global CSS — applies everywhere
import './globals.css';

// Third-party CSS from node_modules
import 'bootstrap/dist/css/bootstrap.css';

// CSS Module — scoped to this component
import styles from './Button.module.css';

function Button() {
  return <button className={styles.primary}>Click</button>;
}
```

**Expected Output:** Global styles apply app-wide; the CSS Module class is scoped and unique.

**Why this output:** The side-effect imports include the CSS in the bundle. The CSS Module import returns an object mapping class names to generated unique names, ensuring scoping.

### Real-World Cases

- **Root layout:** Importing `global.css` in `app/layout.tsx` for Next.js applications.
- **Third-party libraries:** Importing Bootstrap or Tailwind base styles from packages.
- **Component styles:** Importing CSS Modules for scoped component styling.

### References

- Next.js — Global styles — https://nextjs.org/learn/dashboard-app/css-styling
- Vercel — Importing CSS from packages — https://vercel.com/docs/conformance/rules/NO_EXTERNAL_CSS_AT_IMPORTS
- TanStack — Global route CSS — https://tanstack.com/router/latest/docs/start/framework/react/guide/css-styling

---

## 3. Global CSS

### Definitions

**Core Definition:** Global CSS refers to stylesheet rules that apply to the entire application, regardless of which component imported them.

**Technical Definition:** In React, all CSS imported through a standard `import './styles.css'` statement becomes global. A class defined in one file applies to any element with that class name anywhere in the document. Global CSS is best limited to foundational styles: resets, base typography, brand colors, and CSS custom properties. Next.js recommends importing global CSS in the root layout to apply styles to every route. For component-specific styling, CSS Modules or utility-first CSS should be used instead to avoid collisions.

**Beginner-Friendly Explanation:** Global CSS means the styles apply to the whole app. If you define a `.button` class, any element with `className="button"` will get those styles, no matter where it is. This is useful for resets and base styles, but can cause conflicts if you are not careful.

### Purposes

- To apply foundational styles (resets, typography, base element rules) across the application.
- To define design tokens (CSS custom properties) that are available everywhere.
- To load third-party CSS frameworks that require global scope.
- To ensure consistent baseline styling across all routes.

### Syntax Rules and Structure

**General Syntax:**

```jsx
// app/layout.tsx (Next.js root layout)
import './global.css';

export default function RootLayout({ children }) {
  return <html><body>{children}</body></html>;
}
```

**Syntax Rules:**

1. Import global CSS at the application root (root layout or top-level component).
2. Global CSS should contain only truly global rules: resets, base typography, brand colors.
3. Avoid attaching styles to raw HTML elements unless part of the design system.
4. Use CSS Modules or utility classes for component-specific styles.

**Constraints and Limitations:**

- Global styles cannot be scoped to a single component.
- Importing global CSS in many components can cause style leakage and unexpected overrides.
- In React Server Components, page-specific global CSS in broad `'use client'` entry points can make unrelated pages wait on that CSS.

### Annotated Code Examples

**`global.css`:**

```css
:root {
  --color-primary: #2563eb;
  --spacing-unit: 8px;
}

* {
  box-sizing: border-box;
  margin: 0;
}

body {
  font-family: system-ui, sans-serif;
  background: #fafafa;
}
```

**`app/layout.tsx`:**

```tsx
import './global.css';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

**Expected Output:** Every route inherits the reset, base typography, and CSS custom properties.

**Why this output:** The global stylesheet is imported once at the root layout. Its rules apply to the entire document, and the CSS custom properties are available to all components.

### Real-World Cases

- **Application reset:** Normalising browser defaults across all pages.
- **Design tokens:** Defining `--color-primary`, `--spacing-unit` globally.
- **Tailwind base styles:** Importing `@tailwind base;` globally.

### References

- Next.js — Global styles — https://nextjs.org/learn/dashboard-app/css-styling
- React on Rails — Global CSS and RSC — https://reactonrails.com
- RT Camp — Minimal Global Styles — https://rtcamp.com/handbook/react-best-practices/styling/

---

## 4. CSS Specificity

### Definitions

**Core Definition:** CSS specificity is the weight browsers use to determine which CSS declaration is most relevant to an element when multiple rules conflict.

**Technical Definition:** Specificity is the weight that browsers use in the cascade algorithm to determine the CSS declaration that is the most relevant to an element, which in turn determines the property value to apply. The specificity value is a three-column count: **ID – CLASS – TYPE**. ID selectors add 1-0-0, class selectors (including attribute selectors and pseudo-classes) add 0-1-0, and type selectors (elements and pseudo-elements) add 0-0-1. The declaration with the highest specificity wins. If specificities are equal, the later rule in the source order wins. Specificity is compared only between selectors from the same cascade origin and layer.

**Beginner-Friendly Explanation:** Specificity is CSS's way of deciding which style wins when two rules target the same element. An ID is stronger than a class, and a class is stronger than a tag name. If two rules have the same strength, the one written later wins.

### Purposes

- To understand which style rule is applied when multiple rules conflict.
- To write selectors that are predictable and maintainable.
- To avoid specificity wars and unnecessary `!important` flags.

### Syntax Rules and Structure

**Specificity Weight Categories:**

| Category | Selectors | Weight |
|----------|-----------|--------|
| ID | `#header` | 1-0-0 |
| Class | `.btn`, `[type="radio"]`, `:hover` | 0-1-0 |
| Type | `div`, `::before` | 0-0-1 |

**Syntax Rules:**

1. Inline styles have the highest specificity (1-0-0-0) and override all selectors.
2. `!important` overrides normal declarations but should be avoided.
3. The universal selector `*` has zero specificity (0-0-0).
4. Nesting adds specificity equal to the parent selector.

**Constraints and Limitations:**

- High-specificity selectors are hard to override without `!important`.
- ID selectors create very high specificity and are discouraged in component styling.
- Specificity is compared only within the same cascade layer and origin.

### Annotated Code Examples

```css
/* Specificity: 0-0-1 */
p {
  color: blue;
}

/* Specificity: 0-1-0 */
.text {
  color: red;
}

/* Specificity: 1-0-0 */
#intro {
  color: green;
}
```

```jsx
<p id="intro" className="text">Hello</p>
```

**Expected Output:** The text is green because `#intro` has the highest specificity (1-0-0).

**Why this output:** The ID selector outranks both the class and type selectors. Even though `.text` appears later in the stylesheet, specificity takes precedence over source order.

### Real-World Cases

- **Debugging style conflicts:** Understanding why a style is not applying.
- **Writing low-specificity CSS:** Using classes and avoiding IDs for easier overrides.
- **Cascade layers:** Using `@layer` to control specificity across groups of rules.

### References

- MDN — Specificity — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Specificity
- MDN — Handling conflicts — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Introduction

---

## 5. CSS Cascade

### Definitions

**Core Definition:** The CSS cascade is the algorithm that determines which CSS rule is applied when multiple rules target the same element.

**Technical Definition:** The cascade is an algorithm that defines how user agents combine property values originating from different sources. It defines the origin and layer that takes precedence when declarations in more than one origin, cascade layer, or `@scope` block set a value for a property. The cascade considers, in order: **origin and importance** (user-agent, user, author), **cascade layers**, **specificity**, and **order of appearance**. The cascade lies at the core of CSS, as emphasized by the name: Cascading Style Sheets.

**Beginner-Friendly Explanation:** The cascade is CSS's decision-making process. When several rules could apply to the same element, the cascade figures out which one actually wins. It looks at where the rule came from, what layer it is in, how specific the selector is, and which rule was written last.

### Purposes

- To resolve conflicts between multiple CSS declarations.
- To determine the final value of a property when multiple rules apply.
- To enable predictable and maintainable styling through origin, layers, and specificity.

### Syntax Rules and Structure

**Cascade Order (Highest to Lowest Precedence):**

| Step | Factor |
|------|--------|
| 1 | Origin and importance (user-agent → user → author) |
| 2 | Cascade layers (`@layer`) |
| 3 | Specificity (ID > class > type) |
| 4 | Order of appearance (later wins) |

**Syntax Rules:**

1. Author styles override user-agent styles unless the user-agent uses `!important`.
2. Cascade layers (`@layer`) control precedence between groups of rules.
3. Specificity is compared after origin and layer.
4. If all else is equal, the later rule wins.

### Annotated Code Examples

```css
@layer reset, base, components;

@layer base {
  p { color: blue; }
}

@layer components {
  .text { color: red; }
}

/* Unlayered */
p { color: green; }
```

```jsx
<p className="text">Hello</p>
```

**Expected Output:** The text is green because unlayered styles override layered styles.

**Why this output:** The unlayered `p` rule has the highest cascade priority, even though `.text` has higher specificity. Origin and layer precedence come before specificity in the cascade.

### Real-World Cases

- **Cascade layers:** Organising styles into reset, base, components, and utilities layers.
- **Third-party CSS:** Placing third-party styles in a lower-priority layer.
- **Debugging:** Understanding why a style is not applying despite high specificity.

### References

- MDN — Introduction to the CSS cascade — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Introduction
- MDN — Cascade layers — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Introduction

---

## 6. CSS Inheritance

### Definitions

**Core Definition:** CSS inheritance is the mechanism by which certain properties are passed from a parent element to its children when no value is specified for the child.

**Technical Definition:** Inheritance controls what happens when no value is specified for a property on an element. Some properties are **inherited** by default (e.g., `color`, `font-family`, `line-height`), meaning the child receives the computed value from its parent. Other properties are **non-inherited** (e.g., `margin`, `padding`, `border`), meaning the child uses the property's initial value if no declaration applies. The `inherit` keyword explicitly forces inheritance, and `initial`, `unset`, and `revert` control the fallback behaviour. Inheritance is always from the parent element in the document tree, even when the parent is not the containing block.

**Beginner-Friendly Explanation:** Inheritance means some styles automatically pass down from parent elements to their children. For example, if you set `color` on a `<div>`, all text inside it inherits that color unless something else overrides it. Other properties, like `margin`, do not inherit — each element starts with its own default.

### Purposes

- To propagate typography and color settings through the component tree.
- To reduce redundant declarations by relying on inherited values.
- To control which properties inherit and which do not.

### Syntax Rules and Structure

**Inherited Properties (Examples):**

| Property | Inherited? |
|----------|-----------|
| `color` | Yes |
| `font-family` | Yes |
| `font-size` | Yes |
| `line-height` | Yes |
| `visibility` | Yes |
| `margin` | No |
| `padding` | No |
| `border` | No |
| `background` | No |

**Syntax Rules:**

1. Inherited properties automatically receive the parent's computed value when no declaration applies.
2. Non-inherited properties use their initial value when no declaration applies.
3. The `inherit` keyword explicitly forces inheritance on any property.
4. `initial` resets to the property's initial value; `unset` behaves as `inherit` for inherited properties and `initial` for non-inherited ones.

### Annotated Code Examples

```css
.parent {
  color: blue;
  margin: 16px;
}

.child {
  /* color is inherited (blue) */
  /* margin is not inherited (uses initial 0) */
}
```

```jsx
<div className="parent">
  Parent
  <div className="child">Child</div>
</div>
```

**Expected Output:** Both "Parent" and "Child" text are blue. The child has no inherited margin.

**Why this output:** `color` is an inherited property, so the child receives `blue` from the parent. `margin` is non-inherited, so the child uses its initial value (0) unless explicitly declared.

### Real-World Cases

- **Typography:** Setting `font-family` and `color` once on `body` and letting all elements inherit.
- **Component theming:** Using `color` inheritance to propagate theme colors.
- **Resetting styles:** Using `all: unset` or `all: initial` to clear inheritance.

### References

- MDN — CSS cascade and inheritance — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade
- MDN — Inheritance — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Inheritance

---

## 7. CSS Naming Conventions

### Definitions

**Core Definition:** CSS naming conventions are agreed-upon rules for naming CSS classes to improve readability, maintainability, and collaboration.

**Technical Definition:** CSS naming conventions are guidelines that assist developers in writing consistent and understandable CSS code. They advocate using specific naming patterns for CSS classes, IDs, and selectors to increase code readability, maintainability, and collaboration. Popular conventions include **BEM** (Block Element Modifier), **OOCSS** (Object-Oriented CSS), **SMACSS** (Scalable and Modular Architecture for CSS), and SUIT CSS. Choosing the appropriate convention depends on the project's scope and collaboration practices.

**Beginner-Friendly Explanation:** Naming conventions are rules for naming your CSS classes. Good names make it obvious what a class does and where it belongs. This helps teams avoid conflicts and keeps the codebase organised.

### Purposes

- To make class names self-documenting and predictable.
- To avoid naming collisions in large codebases.
- To improve collaboration between developers.
- To make it easier to find where styles are defined.

### Syntax Rules and Structure

**Comparison of Naming Conventions:**

| Convention | Focus | Example |
|------------|-------|---------|
| BEM | Block, Element, Modifier naming | `card__title--large` |
| OOCSS | Separation of structure and skin | `.button` + `.button--primary` |
| SMACSS | Categorisation into five rule types | `l-header`, `is-active`, `module-card` |

**Syntax Rules:**

1. Choose one convention and apply it consistently across the project.
2. Use descriptive, functional names rather than abstract labels.
3. Avoid descendant selectors and location-dependent styling.

### Annotated Code Examples

```css
/* BEM naming */
.card { }
.card__title { }
.card__title--large { }

/* SMACSS naming */
.l-header { }      /* Layout */
.is-active { }     /* State */
.module-card { }   /* Module */
```

**Expected Output:** Class names clearly communicate their role and scope.

**Why this output:** BEM separates blocks, elements, and modifiers with `__` and `--`. SMACSS prefixes layout (`l-`), state (`is-`), and module classes for immediate recognition.

### Real-World Cases

- **Large teams:** Consistent naming reduces merge conflicts and confusion.
- **Design systems:** Component classes follow a predictable pattern.
- **Legacy codebases:** Naming conventions make refactoring safer.

### References

- Frontend Mentor — Understanding CSS naming conventions — https://www.frontendmentor.io/articles/understanding-css-naming-conventions-bem-oocss-smacss-and-suit-css-V6ZZUYs1xz
- SMACSS — Categorizing CSS Rules — http://smacss.com/book/categorizing

---

## 8. BEM (Block Element Modifier)

### Definitions

**Core Definition:** BEM is a naming convention that divides UI into independent blocks, each with elements and modifiers, using `__` and `--` separators.

**Technical Definition:** BEM (Block Element Modifier) is a popular naming convention for CSS classes that helps developers create maintainable, reusable, and self-documenting code. It is based on the concept of dividing user interfaces into independent blocks, each of which can have elements and modifiers. **Blocks** are standalone components (e.g., `.nav`). **Elements** are parts of a block (e.g., `.nav__list`). **Modifiers** are variations or states (e.g., `.nav__item--active`). The naming pattern is `block__element--modifier`.

**Beginner-Friendly Explanation:** BEM gives you a formula for naming classes. You name the main component (the block), then its parts (elements), then variations (modifiers). The double underscores and double hyphens make the structure obvious.

### Purposes

- To create self-documenting class names that reveal the component structure.
- To avoid naming collisions through unique prefixes.
- To make styles reusable across different contexts.
- To reduce specificity issues by using flat class selectors.

### Syntax Rules and Structure

**BEM Syntax:**

```css
.block { }
.block__element { }
.block__element--modifier { }
```

**Syntax Rules:**

1. Block names are lowercase with hyphens: `.nav`.
2. Elements are separated by double underscores: `.nav__list`.
3. Modifiers are separated by double hyphens: `.nav__item--active`.
4. Class names should not depend on other blocks (no `.nav .button`).

### Annotated Code Examples

```css
.nav {
  display: flex;
}

.nav__list {
  list-style: none;
}

.nav__item--active {
  font-weight: bold;
}
```

```jsx
<nav className="nav">
  <ul className="nav__list">
    <li className="nav__item nav__item--active">Home</li>
  </ul>
</nav>
```

**Expected Output:** A horizontal navigation with a bold active item.

**Why this output:** `.nav` defines the block. `.nav__list` styles the element. `.nav__item--active` applies the modifier for the active state.

### Real-World Cases

- **Component libraries:** BEM is widely used in design systems.
- **Large applications:** Unique class names prevent collisions.
- **Team collaboration:** The naming pattern is easy to learn and apply.

### References

- Frontend Mentor — BEM — https://www.frontendmentor.io/articles/understanding-css-naming-conventions-bem-oocss-smacss-and-suit-css-V6ZZUYs1xz
- Get BEM — Official site — https://getbem.com/

---

## 9. SMACSS (Scalable and Modular Architecture for CSS)

### Definitions

**Core Definition:** SMACSS is a CSS architecture that categorises rules into five types — Base, Layout, Module, State, and Theme — to improve scalability and maintainability.

**Technical Definition:** SMACSS (pronounced "smacks") is more of a style guide than a rigid framework. At its core is **categorisation**: by categorizing CSS rules, developers begin to see patterns and can define better practices around each pattern. The five categories are: **Base** (defaults, single-element selectors), **Layout** (page sections), **Module** (reusable components), **State** (how modules/layouts look in a particular state), and **Theme** (visual variations). Naming prefixes like `l-` for layout and `is-` for state help identify each category.

**Beginner-Friendly Explanation:** SMACSS organises your CSS into five buckets: base styles, layout, modules, states, and themes. Each bucket has its own naming prefix so you can tell at a glance what kind of rule you are looking at.

### Purposes

- To categorise CSS rules for better organisation.
- To reduce code duplication and simplify maintenance.
- To make it easy to find where a style belongs.
- To scale CSS across large projects.

### Syntax Rules and Structure

**SMACSS Categories:**

| Category | Purpose | Example Prefix |
|----------|---------|----------------|
| Base | Default element styles | (none) |
| Layout | Page sections | `l-` |
| Module | Reusable components | (module name) |
| State | Active/inactive states | `is-` |
| Theme | Visual variations | (theme name) |

**Syntax Rules:**

1. Base rules use single element selectors: `html`, `body`, `a`.
2. Layout rules use the `l-` prefix: `.l-header`, `.l-sidebar`.
3. State rules use the `is-` prefix: `.is-hidden`, `.is-collapsed`.
4. Modules are the bulk of the styles and use descriptive names.

### Annotated Code Examples

```css
/* Base */
body { margin: 0; font-family: sans-serif; }

/* Layout */
.l-header { height: 60px; }
.l-sidebar { width: 250px; }

/* Module */
.card { padding: 16px; border: 1px solid #ddd; }

/* State */
.is-hidden { display: none; }
.is-active { font-weight: bold; }
```

**Expected Output:** Styles are clearly organised into SMACSS categories.

**Why this output:** The prefixes (`l-`, `is-`) and the categorisation make it immediately clear which layer each rule belongs to.

### Real-World Cases

- **Large-scale applications:** SMACSS keeps CSS organised as the project grows.
- **Team projects:** The five categories provide a shared vocabulary.
- **Combining methodologies:** SMACSS can be combined with BEM for naming.

### References

- SMACSS — Categorizing CSS Rules — http://smacss.com/book/categorizing
- Frontend Mentor — SMACSS overview — https://www.frontendmentor.io/articles/understanding-css-naming-conventions-bem-oocss-smacss-and-suit-css-V6ZZUYs1xz

---

## 10. OOCSS (Object-Oriented CSS)

### Definitions

**Core Definition:** OOCSS is a CSS methodology that encourages reusable, modular styles by separating structure from skin and container from content.

**Technical Definition:** OOCSS (Object-Oriented CSS) draws upon object-oriented programming concepts. It has two major underlying principles: **separation of structure and skin** (layout vs. visual appearance) and **separation of container and content** (styles should not depend on where an element is located). The goal is to create reusable "objects" — CSS classes that can be applied anywhere and behave consistently, regardless of context. OOCSS also promotes the open/closed principle: classes should be open for extension but closed for modification.

**Beginner-Friendly Explanation:** OOCSS says you should separate how an element is laid out from how it looks, and you should not tie styles to a specific location. This makes your CSS more reusable.

### Purposes

- To encourage reuse of CSS snippets independent of location.
- To separate structure (layout) from skin (visual appearance).
- To avoid location-dependent styling.
- To create simple, reusable base objects.

### Syntax Rules and Structure

**OOCSS Principles:**

| Principle | Description | Example |
|-----------|-------------|---------|
| Separate structure and skin | Layout classes vs. visual classes | `.media` (structure) + `.media--rounded` (skin) |
| Separate container and content | Styles should not depend on parent | `.btn` works anywhere, not just inside `.header` |

**Syntax Rules:**

1. Create separate classes for layout and visual styling.
2. Avoid descendant selectors that tie styles to a container.
3. Keep base objects simple and extend them with modifiers.

### Annotated Code Examples

```css
/* Structure (separate from skin) */
.media {
  display: flex;
  align-items: flex-start;
}

/* Skin (separate from structure) */
.media--bordered {
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* Reusable button object */
.btn {
  display: inline-block;
  padding: 8px 16px;
}

.btn--primary {
  background: blue;
  color: white;
}
```

```jsx
<div className="media media--bordered">
  <img src="avatar.jpg" alt="Avatar" />
  <div>Content</div>
</div>

<button className="btn btn--primary">Click</button>
```

**Expected Output:** The media object and button can be reused in any context.

**Why this output:** Structure (`.media`, `.btn`) is separated from skin (`.media--bordered`, `.btn--primary`). Neither depends on a specific container, making them reusable anywhere.

### Real-World Cases

- **Design systems:** OOCSS principles underpin many component libraries.
- **Reusable utilities:** Layout objects that work in any context.
- **Scalable CSS:** Reducing duplication through reusable classes.

### References

- KeyCDN — OOCSS: The Future of Writing CSS — https://www.keycdn.com/blog/oocss
- CSS-Tricks — Methods to Organize CSS — https://css-tricks.com/methods-organize-css/

---

## 11. CSS Best Practices

### Definitions

**Core Definition:** CSS best practices are guidelines for writing maintainable, performant, and scalable stylesheets in React applications.

**Technical Definition:** CSS best practices in React encompass: limiting global styles to foundational rules (resets, typography, design tokens), using CSS Modules or utility-first CSS for component-scoped styles, avoiding inline styles for anything other than dynamic values, following a consistent naming convention, avoiding multiple competing styling solutions, and leveraging modern CSS features (cascade layers, nesting, container queries). The goal is a predictable, scalable, and easy-to-reason-about styling system.

**Beginner-Friendly Explanation:** Best practices are the habits that keep your CSS clean and maintainable. They include keeping global styles minimal, using the right tool for each job, and being consistent.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Minimal global styles** | Limit global CSS to resets, base typography, and design tokens. |
| **Use CSS Modules for component styles** | Scope styles to avoid collisions. |
| **Avoid inline styles** | Use them only for dynamic values, not static styling. |
| **Stick to one styling approach** | Avoid mixing CSS Modules, SASS, and CSS-in-JS. |
| **Use design tokens** | Reference semantic classes like `text-primary` instead of hardcoded values. |
| **Use descriptive class names** | Prefer `btn-primary` over `box1` or `redText`. |
| **Avoid location-dependent styling** | Do not nest selectors unnecessarily. |
| **Use cascade layers for organisation** | Group styles into reset, base, components, utilities. |

### Annotated Code Examples

**✅ Good Practices:**

```css
/* global.css — minimal global styles */
:root {
  --color-primary: #2563eb;
  --spacing-4: 16px;
}

* { box-sizing: border-box; }
body { font-family: system-ui, sans-serif; }
```

```jsx
// Button.module.css — scoped component styles
import styles from './Button.module.css';

function Button({ variant = 'primary' }) {
  return <button className={styles[variant]}>Click</button>;
}
```

**❌ Bad Practices:**

```jsx
// ❌ Inline static styles
<div style={{ padding: 16, backgroundColor: 'blue' }} />

// ❌ Mixing multiple styling solutions
import styles from './Button.module.css';
import styled from 'styled-components';

// ❌ Global styles for component-specific rules
.button { padding: 16px; } // Applied everywhere!
```

**Expected Output:** The good practices produce predictable, scoped, and maintainable styling.

**Why this output:** Global styles are minimal and foundational. Component styles are scoped with CSS Modules. Inline styles are avoided for static styling. A single styling approach is used consistently.

### Real-World Cases

- **All React applications:** Best practices apply universally.
- **Design systems:** Consistency and scalability are critical.
- **Large teams:** A single styling approach prevents conflicts.

### References

- RT Camp — Styling Best Practice for React-based projects — https://rtcamp.com/handbook/react-best-practices/styling/
- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- MDN — Organizing your CSS — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Organizing

---

## References

- Next.js — CSS Styling — https://nextjs.org/learn/dashboard-app/css-styling
- Vercel — NO_EXTERNAL_CSS_AT_IMPORTS — https://vercel.com/docs/conformance/rules/NO_EXTERNAL_CSS_AT_IMPORTS
- TanStack — CSS Styling — https://tanstack.com/router/latest/docs/start/framework/react/guide/css-styling
- MDN — Specificity — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Specificity
- MDN — Introduction to the CSS cascade — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Introduction
- MDN — CSS cascade and inheritance — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade
- MDN — Inheritance — https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Inheritance
- MDN — Organizing your CSS — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Organizing
- SMACSS — Categorizing CSS Rules — http://smacss.com/book/categorizing
- Frontend Mentor — Understanding CSS naming conventions — https://www.frontendmentor.io/articles/understanding-css-naming-conventions-bem-oocss-smacss-and-suit-css-V6ZZUYs1xz
- KeyCDN — OOCSS: The Future of Writing CSS — https://www.keycdn.com/blog/oocss
- CSS-Tricks — Methods to Organize CSS — https://css-tricks.com/methods-organize-css/
- RT Camp — Styling Best Practice for React-based projects — https://rtcamp.com/handbook/react-best-practices/styling/
- Impertio Studio — React Implementation Styling — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- React on Rails — CSS and Styling with React Server Components — https://reactonrails.com
- Get BEM — Official site — https://getbem.com/