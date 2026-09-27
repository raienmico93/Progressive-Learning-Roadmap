# React CSS and Native Styling in React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React CSS and native styling is the discipline of applying, scoping, and managing styles within a React component architecture, using native browser CSS features (cascade layers, `@scope`, custom properties, container queries) and build-tool-based scoping mechanisms (CSS Modules) to avoid the global-scope pitfalls of traditional CSS.

**Technical Definition:** React CSS and native styling encompasses the architectural decisions and browser-native mechanisms for styling component-based applications. It addresses the fundamental conflict between CSS's global, document-oriented design and React's component-scoped, modular architecture. Key mechanisms include **CSS Cascade Layers** (`@layer`), which decouple style priority from selector specificity by establishing explicit layer ordering; **CSS Modules**, which compile class names to locally-scoped unique identifiers; **CSS Custom Properties** (CSS Variables), which enable dynamic theming at runtime through the cascade; **Container Queries** (`@container`), which allow components to respond to their own container's size rather than the viewport; and the **`@scope` at-rule**, which restricts selectors to a DOM subtree without tooling. Together, these features enable component-level styling that is portable, maintainable, and free from global leakage.

**Beginner-Friendly Explanation:** CSS was designed for single HTML pages, where every style rule applies globally. React builds applications out of small, reusable components—but a global stylesheet means one component's styles can accidentally break another's. React CSS and native styling is about using modern CSS features to keep each component's styles contained to that component, like giving every component its own private style notebook instead of writing on a shared whiteboard. Cascade layers, CSS Modules, custom properties, container queries, and `@scope` are the tools that make this possible.

### Key Characteristics

- **Global Scope is the Default:** Standard CSS imports in React are global; a class name defined in one component's stylesheet can affect any other component in the application.
- **Specificity Wars:** Global CSS combined with third-party libraries and legacy styles forces developers to add `!important` flags, creating brittle stylesheets.
- **Cascade Layers Decouple Priority from Specificity:** `@layer` establishes explicit priority ordering; a simple class in a higher layer beats a heavy selector in a lower layer.
- **CSS Modules Provide Local Scoping by Default:** All class names and animation names are scoped locally; the import returns a mapping object from local names to global names.
- **Custom Properties Enable Runtime Theming:** CSS variables scoped to `:root` or `[data-theme]` allow theme switching without re-rendering components.
- **Container Queries Make Components Portable:** A component responds to its own container's size, not the viewport, so the same component works in a sidebar and a main column without modifier classes.
- **`@scope` Restricts Selectors to Subtrees:** The `@scope` at-rule limits selectors to a DOM subtree without build tools, though browser support is still maturing.

### Prerequisites

- Solid understanding of CSS selectors, specificity, the cascade, and inheritance.
- Familiarity with React function components, JSX, and the `className` prop.
- Working knowledge of a build tool (Vite, Webpack, Next.js) and how it handles CSS imports.
- Basic understanding of React Context and the `useState`/`useEffect` Hooks (for theming).
- Awareness of browser developer tools for debugging styles.

### Related Programming Areas

- **CSS Architecture:** BEM, SMACSS, and other naming methodologies.
- **Design Systems:** Design tokens, CSS custom properties, and component libraries.
- **Theming:** Light/dark mode, brand themes, and runtime theme switching.
- **Responsive Design:** Media queries, container queries, and fluid typography.
- **Build Tooling:** CSS Modules, PostCSS, and preprocessor integration.

### Core Concepts / Features

1. Global CSS Injection, Reset Layers, and Stylesheet Ordering Bugs
2. Component-Level Styles and the Limits of Scope Insulation
3. CSS Modules Architecture (Local Scoping, Class Composition, Dynamic Class Joining)
4. CSS Naming Strategies (BEM, SMACSS) vs. CSS-Scoping Mechanisms
5. Modern Native CSS Integration: CSS Variables for Dynamic Theme Switching
6. Responsive Styling Foundations: Media Queries vs. Container Queries

---

## Core Concept 1: Global CSS Injection, Reset Layers, and Stylesheet Ordering Bugs

### Definitions

**Core Definition:** Global CSS injection is the default behaviour of imported stylesheets in React, where all styles become globally scoped; reset layers are CSS resets (e.g., `normalize.css`) that override component styles; and stylesheet ordering bugs occur when the order of CSS imports in the bundle does not match the intended cascade priority.

**Technical Definition:** When a CSS file is imported in a React component, the bundler (Vite, Webpack) injects it into the document, making all its rules globally available. This breaks component encapsulation: "CSS imports are global, which breaks decoupling, because a component will have access to styles that are not within its own scope". A CSS reset (e.g., `body { margin: 0; }`) declared outside a `@layer` takes precedence over layered styles, because "unlayered styles always beat any layered style regardless of specificity". Stylesheet ordering bugs arise when the import order in the bundle does not match the intended cascade order—for example, when a third-party library's styles are injected after your own, overriding your components.

**Beginner-Friendly Explanation:** Imagine you and your roommates share a whiteboard. Everyone writes on the same board—your notes, their notes, and the landlord's rules. That's global CSS. A CSS reset is like the landlord writing "no writing on the walls" on the whiteboard, which covers up your notes. Stylesheet ordering bugs happen when someone writes over your section because they wrote after you. CSS Cascade Layers fix this by giving everyone their own whiteboard, stacked in a fixed order: the landlord's rules are at the bottom, yours are in the middle, and urgent notes are on top.

### Purposes

- **Global CSS injection:** To apply base styles (resets, typography) across the entire application.
- **Reset layers:** To normalise browser defaults without overriding component styles.
- **Cascade layers:** To establish explicit priority ordering so components are not overridden by third-party or legacy styles.
- **Stylesheet ordering:** To ensure that the intended cascade order is preserved regardless of import order in the bundle.

### Syntax Rules and Structure

**Cascade Layer Declaration (Top of Global Stylesheet):**
```css
/* app/globals.css — declare layer order at the very top */
@layer reset, vendor, design-system, utilities;
```

**Component Breakdown:**
- `@layer reset, vendor, design-system, utilities`: Declares four layers in order from lowest to highest priority.
- Later layers win over earlier layers, regardless of selector specificity.
- Unlayered styles always beat layered styles.

**Assigning Styles to Layers:**
```css
@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  body { margin: 0; }
}

@layer vendor {
  .external-legacy-widget table td > div {
    background-color: #888888;
    padding: 24px;
  }
}

@layer design-system {
  .custom-dashboard-card {
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  }
}

@layer utilities {
  .force-brand-purple { background-color: #6366f1 !important; }
}
```

**Component Breakdown:**
- `@layer reset { ... }`: Base normalisation; lowest priority.
- `@layer vendor { ... }`: Third-party styles; low priority, cannot override the design system.
- `@layer design-system { ... }`: Your component styles; higher priority than vendor.
- `@layer utilities { ... }`: Utility classes that must always win; highest priority.

**Fix for Mantine Styles Being Overridden:**
```js
// ❌ Wrong order – Mantine styles override your styles
import './styles.css';
import '@mantine/core/styles.css';
```
```js
// ✅ Correct order – your styles override Mantine styles
import '@mantine/core/styles.css';
import './styles.css';
```

**Using Mantine's Layer Version:**
```javascript
import '@mantine/core/styles.layer.css';
```

**Syntax Rules:**
- Declare `@layer` order at the very top of your global stylesheet, before any other rules.
- Put resets in a `@layer reset` (or `@layer base`) so they do not override component styles.
- Put third-party library styles in a `@layer vendor` so your design system can override them.
- Put your component styles in a higher layer (e.g., `@layer design-system`).
- Put utility classes in the highest layer (`@layer utilities`).
- If a library provides a `.layer.css` version, use it instead of the unlayered version.
- If you cannot use `@layer`, ensure your styles are imported after the library's styles.

**Constraints and Limitations:**
- `@layer` has broad browser support (Baseline 2022+) but may not work in very old browsers.
- Unlayered styles always beat layered styles, so a stray unlayered reset will override everything.
- `!important` reverses layer order: an `!important` declaration in a lower layer beats a normal declaration in a higher layer.
- Not all build tools handle `@layer` correctly; some may reorder imports.
- Safari had bugs with `all: revert-layer`; React Spectrum removed it in v0.7.0.

### Annotated Code Example: Layer Order Fix

```css
/* app/globals.css */

/* 1. Declare layer order — reset first, utilities last */
@layer reset, vendor, design-system, utilities;

/* 2. Reset layer — normalises browser defaults */
@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
  }
  body {
    margin: 0;
    font-family: system-ui, sans-serif;
  }
}

/* 3. Vendor layer — third-party widget styles */
@layer vendor {
  /* A heavy legacy selector that would normally win */
  .legacy-calendar table td > div {
    background-color: #888;
    padding: 24px;
  }
}

/* 4. Design system layer — your component styles */
@layer design-system {
  .card {
    background: white;
    border-radius: 12px;
    padding: 16px;
  }
}

/* 5. Utilities layer — highest priority */
@layer utilities {
  .text-brand {
    color: #6366f1 !important;
  }
}
```

```jsx
// DashboardCard.tsx
export function DashboardCard() {
  return (
    <div className="card">
      <h2 className="text-brand">Revenue</h2>
      {/* The legacy widget is quarantined in the vendor layer */}
      <div className="legacy-calendar">
        <LegacyCalendarEngine />
      </div>
    </div>
  );
}
```

**Expected Output:** The `.card` styles from `@layer design-system` override the `.legacy-calendar` styles from `@layer vendor`, even though the legacy selector is more specific. The `.text-brand` utility always wins.

**Why This Output Occurs:** The layer order `reset, vendor, design-system, utilities` establishes priority: `utilities` > `design-system` > `vendor` > `reset`. A simple class in `design-system` beats a heavy selector in `vendor` because layer priority is evaluated before specificity.

### Real-World Cases

- **Design systems + Tailwind:** MUI + Tailwind coexistence via cascade layers: `@layer theme, base, mui, components, utilities` so Tailwind utilities beat MUI.
- **Legacy + modern styles:** A new design system layered above a legacy vendor stylesheet.
- **Multi-tenant white-label:** Theme layers that can be reordered per tenant.
- **React Spectrum:** Uses `@layer` internally to avoid specificity issues.

### References

- MDN Web Docs – CSS Cascade Layers: https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Cascade_layers
- Mantine – Styles Order: https://help.mantine.dev/q/styles-order
- SmartTechDevs – Mastering CSS Cascade Layers in React: https://smarttechdevs.in/blog/nextjs-css-cascade-layers-style-isolation
- React Spectrum – v0.7.0 Release Notes: https://react-spectrum.adobe.com/releases/v0-7-0.md

---

## Core Concept 2: Component-Level Styles and the Limits of Scope Insulation

### Definitions

**Core Definition:** Component-level styles are styles intended to apply only to a specific component; scope insulation is the degree to which those styles are prevented from leaking out or being affected by external styles—and it has inherent limits.

**Technical Definition:** Component-level styles aim to achieve encapsulation: "Properly encapsulated components will not have any implicit dependencies on each other". However, CSS inheritance and the global cascade mean that true style isolation is never absolute. Properties like `font-family`, `color`, and `line-height` inherit from ancestors, so a component's inherited styles can be affected by its parent. Global styles (resets, third-party CSS) always affect components unless explicitly reset. Even CSS Modules, which scope class names, do not isolate inherited properties or prevent global styles from affecting the component's elements. The `@scope` at-rule limits selectors to a subtree, but inherited properties still cross the scope boundary. The limits of insulation mean that component-level styles must be designed defensively: use explicit reset rules for inherited properties, avoid relying on the absence of global styles, and use cascade layers to manage priority.

**Beginner-Friendly Explanation:** Imagine you live in an apartment with shared walls. You can paint your own walls (component styles), but noise from neighbours (global styles) still comes through, and the building's heating system (inherited properties) affects your apartment regardless of what you do. Component-level styles are like your interior design—they make your apartment look how you want, but they cannot fully block out the shared infrastructure. The best you can do is soundproof (reset inherited properties) and coordinate with the building management (cascade layers).

### Purposes

- To apply styles to a single component without affecting others.
- To understand the boundaries of scope insulation so components can be designed defensively.
- To use `@scope` for selector isolation without build tooling.
- To recognise that inherited properties cross all scope boundaries and must be reset explicitly.
- To combine multiple insulation mechanisms (CSS Modules + cascade layers + `@scope`) for maximum isolation.

### Syntax Rules and Structure

**`@scope` for Selector Isolation:**
```css
/* Styles inside @scope only apply within .card */
@scope (.card) {
  :scope {
    border: 1px solid #ccc;
    border-radius: 8px;
  }
  p {
    color: #333;
    line-height: 1.5;
  }
  img {
    max-width: 100%;
  }
}
```

**Component Breakdown:**
- `@scope (.card) { ... }`: Limits all selectors inside to descendants of `.card`.
- `:scope`: Selects the scoping root itself (`.card`).
- `p`, `img`: Only match elements inside `.card`, not elsewhere in the document.

**Proximity in `@scope`:**
```css
@scope (.card) to (.card-footer) {
  p { color: #333; }
}
```

**Component Breakdown:**
- `to (.card-footer)`: Stops the scope at `.card-footer`; `p` elements inside `.card-footer` are not styled by this rule.
- This allows fine-grained control over which subtree a rule applies to.

**CSS Modules Scoping:**
```css
/* Button.module.css */
.button {
  background: blue;
  color: white;
}
```

```jsx
import styles from './Button.module.css';

function Button() {
  return <button className={styles.button}>Click</button>;
}
// Renders: <button class="Button_button__a1b2c">Click</button>
```

**Component Breakdown:**
- `styles.button`: The compiled, locally-scoped class name.
- The original `.button` selector is transformed to a unique global name (e.g., `Button_button__a1b2c`).

**Syntax Rules:**
- Use `@scope` for selector isolation without build tools; it is supported in Baseline 2023+ browsers (Chrome, Edge, Safari; Firefox support is in progress).
- Use `:scope` inside `@scope` to target the scoping root.
- Use `to (selector)` to limit the scope's reach.
- Use CSS Modules for class-name scoping; it transforms local names to unique global names.
- Always reset inherited properties (`color`, `font-family`, `line-height`) at the component root if strict isolation is required.
- Recognise that `@scope` isolates *selectors*, not *inherited properties*; inherited values still cross the boundary.
- Combine `@scope` with `@layer` for both selector and priority isolation.

**Constraints and Limitations:**
- CSS inheritance crosses all scope boundaries; `@scope` does not prevent inherited properties from affecting children beyond the scope limit.
- Firefox does not yet fully support `@scope` (as of early 2026).
- CSS Modules do not isolate inherited properties or prevent global styles from affecting the component.
- Styled-components and other CSS-in-JS libraries have runtime performance overhead and SSR complexity.
- Even with insulation, global resets and third-party CSS can still affect components unless layered or scoped.

### Annotated Code Example: `@scope` + CSS Modules

```css
/* Card.module.css */
.card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
}
```

```jsx
// Card.tsx
import styles from './Card.module.css';

export function Card({ children }) {
  return (
    <div className={styles.card}>
      <style>{`
        @scope (.${styles.card}) {
          p { color: #333; line-height: 1.5; }
          img { max-width: 100%; border-radius: 4px; }
        }
      `}</style>
      {children}
    </div>
  );
}
```

**Expected Output:** The `p` and `img` styles only apply inside this card, even though the selectors are generic. The card's border and padding come from the CSS Module class. Inherited properties (e.g., `font-family`) still come from the document, and must be reset explicitly if needed.

**Why This Output Occurs:** The CSS Module scopes `.card` to a unique class name. The `@scope` rule uses that unique class name as its scoping root, limiting `p` and `img` selectors to descendants of the card. However, `font-family` and other inherited properties still cross the boundary because CSS inheritance is not blocked by `@scope`.

### Real-World Cases

- **Embedding third-party HTML:** `@scope` is ideal for styling email previews or CMS content without leaking styles.
- **Design system components:** CSS Modules + `@scope` for maximum isolation.
- **Legacy integration:** `@scope` to contain old CSS selectors to a specific subtree.
- **Defensive component design:** Explicitly resetting inherited properties at the component root.

### References

- Smashing Magazine – CSS @scope: https://www.smashingmagazine.com/2026/02/css-scope-alternative-naming-conventions/
- Frontend Masters – @scope and HTML Style Blocks: https://frontendmasters.com/blog/reminder-that-scope-and-html-style-blocks-are-a-potent-combo/
- MDN Web Docs – @scope: https://developer.mozilla.org/en-US/docs/Web/CSS/@scope
- W3C CSSWG – CSS Cascade 6: https://drafts.csswg.org/css-cascade-6/#scoped-styles
- Stack Overflow – How to limit style to component level in React: https://stackoverflow.com/posts/77510767/revisions

---

## Core Concept 3: CSS Modules Architecture (Local Scoping, Class Composition, and Dynamic Class Joining)

### Definitions

**Core Definition:** CSS Modules is a build-time tool that scopes all class names and animation names locally by default, exporting a mapping object from local names to globally unique names; class composition allows one class to inherit the styles of another; dynamic class joining combines multiple class names conditionally at runtime.

**Technical Definition:** A CSS Module is "a CSS file in which all class names and animation names are scoped locally by default". When imported from JavaScript, it exports an object with all mappings from local names to global names: `import styles from "./style.css"` gives `styles.className` as the compiled unique name. The `composes` property allows a class to inherit the rules of another class: `.otherClassName { composes: className; color: yellow; }`. `composes` must appear before other rules in the class body. Classes can compose from other CSS Modules: `composes: className from "./style.css"`. Dynamic class joining in React is achieved by conditionally constructing a `className` string from the `styles` object, often with a utility library like `clsx` or `classnames`.

**Beginner-Friendly Explanation:** CSS Modules are like giving each component its own private vocabulary. When you write `.button` in a CSS Module, the build tool renames it to something unique like `Button_button__a1b2c`, so it cannot clash with any other `.button` in the application. `composes` is like saying "this class should also include everything from that class"—like mixing paint colours. Dynamic class joining is what you do when a component's appearance depends on its state (e.g., `active`, `disabled`): you build the `className` string conditionally in JavaScript.

### Purposes

- To eliminate class name collisions by scoping all class names locally.
- To make component styles modular and reusable without global namespace pollution.
- To compose styles from multiple classes, promoting DRY (Don't Repeat Yourself) principles.
- To compose classes across files for shared design primitives.
- To dynamically join class names based on component state (active, disabled, variant).
- To achieve explicit dependencies between styles and components.

### Syntax Rules and Structure

**Basic CSS Module:**
```css
/* Button.module.css */
.button {
  background: blue;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
}

.button--large {
  font-size: 20px;
  padding: 12px 24px;
}
```

```jsx
import styles from './Button.module.css';

function Button({ size = 'md', children }) {
  const className = size === 'lg'
    ? `${styles.button} ${styles['button--large']}`
    : styles.button;

  return <button className={className}>{children}</button>;
}
```

**Class Composition (`composes`):**
```css
/* styles.module.css */
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

.secondary {
  composes: base;
  background: gray;
  color: white;
}
```

**Component Breakdown:**
- `.primary` composes `.base`, inheriting its padding, border, and border-radius.
- The compiled `.primary` element receives both class names: `styles.primary` and `styles.base`.
- `composes` must appear before other rules in the class body.

**Composing from Another CSS Module:**
```css
.otherClassName {
  composes: className from "./style.css";
}
```

**Component Breakdown:**
- `composes: className from "./style.css"`: Pulls in a class from another module.
- The composition order across files is undefined; do not define conflicting properties in composed classes from different files.

**Dynamic Class Joining with `clsx`:**
```jsx
import clsx from 'clsx';
import styles from './Button.module.css';

function Button({ variant = 'primary', size = 'md', disabled, children }) {
  return (
    <button
      className={clsx(
        styles.button,
        styles[`button--${variant}`],
        size === 'lg' && styles['button--large'],
        disabled && styles['button--disabled']
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
}
```

**Component Breakdown:**
- `clsx(...)`: Joins class names, ignoring falsy values.
- `styles[\`button--${variant}\`]`: Dynamically accesses the compiled class name.
- `disabled && styles['button--disabled']`: Conditionally includes a class.

**Syntax Rules:**
- Name CSS Module files with the `.module.css` extension so the bundler treats them as modules.
- Import the module as `styles` (or any name) and reference `styles.className`.
- Use camelCase for local class names (recommended).
- Use `composes` to inherit styles from other classes; it must be the first declaration in the class body.
- Use `:global(...)` to escape to global scope for a specific selector.
- Use `clsx` or `classnames` for dynamic class joining.
- Do not use multiple CSS Modules to describe a single element; compose classes within one module instead.

**Constraints and Limitations:**
- `composes` only works for single class selectors, not complex selectors or elements.
- Composition order across files is undefined; avoid conflicting properties in composed classes from different files.
- CSS Modules do not isolate inherited properties; a global `body { font-family: ... }` still affects all components.
- Dynamic class names (computed at runtime) cannot be used with `composes` in CSS; only static class names can be composed.
- CSS Modules add a build step; they are not usable in plain HTML without a bundler.

### Annotated Code Example: Button with Composition and Dynamic Joining

```css
/* Button.module.css */
.base {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;
}

.primary {
  composes: base;
  background: #007bff;
  color: white;
}

.primary:hover {
  background: #0069d9;
}

.secondary {
  composes: base;
  background: #6c757d;
  color: white;
}

.large {
  composes: base;
  padding: 12px 24px;
  font-size: 20px;
}

.disabled {
  composes: base;
  opacity: 0.5;
  cursor: not-allowed;
}
```

```jsx
import clsx from 'clsx';
import styles from './Button.module.css';

export function Button({
  variant = 'primary',
  size = 'md',
  disabled = false,
  children,
}) {
  return (
    <button
      className={clsx(
        styles[variant],           // .primary or .secondary (composes .base)
        size === 'lg' && styles.large,
        disabled && styles.disabled
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
}
```

**Expected Output:** A `<button>` with the combined styles of `.base` and `.primary` (or `.secondary`), plus `.large` and `.disabled` if applicable. The compiled class names are unique (e.g., `Button_base__a1b2c Button_primary__d3e4f`).

**Why This Output Occurs:** `.primary` composes `.base`, so the element receives both the `.primary` and `.base` compiled class names. `clsx` joins the selected classes. The `disabled` attribute is set separately for accessibility, while the `.disabled` class provides the visual state.

### Real-World Cases

- **Design systems:** Buttons, inputs, and cards composed from shared base classes.
- **Variant-driven components:** Buttons with `primary`, `secondary`, `danger`, `ghost` variants.
- **Size-driven components:** Inputs with `sm`, `md`, `lg` sizes.
- **State-driven components:** Disabled, loading, error, and success states.

### References

- CSS Modules – README: https://raw.githubusercontent.com/css-modules/css-modules/8d33532b29327386173b28051d289e9d25545e1f/README.md
- CSS Modules – Composition: https://github.com/Terro216/css-modules/blob/master/docs/composition.md
- Lightning CSS – CSS Modules: https://lightningcss.dev
- MakeUseOf – How to Style React Components Using CSS Modules: https://www.makeuseof.com

---

## Core Concept 4: CSS Naming Strategies (BEM, SMACSS) vs. CSS-Scoping Mechanisms

### Definitions

**Core Definition:** BEM (Block, Element, Modifier) and SMACSS (Scalable and Modular Architecture for CSS) are naming conventions that manage global CSS by enforcing structured class names; CSS-scoping mechanisms (CSS Modules, `@scope`, cascade layers) use browser or build-tool features to isolate styles without naming conventions.

**Technical Definition:** BEM is a naming convention where a **Block** is a standalone component (`button`), an **Element** is a part of a block (`button__icon`), and a **Modifier** is a variant (`button--primary`). SMACSS categorises CSS rules into five types: **Base** (defaults), **Layout** (page structure), **Module** (reusable components), **State** (JavaScript-dependent states), and **Theme** (visual themes). Both are naming-based solutions to the global scope problem: "Rigid class name conventions, such as BEM, are one theoretical solution to this issue". However, "in the real world, it doesn't always work out like that"—BEM class names can become long and unwieldy (e.g., `app-user-overview__status--is-authenticating`), and small HTML changes require many CSS revisions. CSS-scoping mechanisms, by contrast, use technology (local scoping, `@scope`) rather than convention, eliminating the need for lengthy prefixes.

**Beginner-Friendly Explanation:** BEM and SMACSS are like agreeing on a naming system for everyone in a shared office: "All documents about Project X start with 'X-', and drafts end with '-draft'." It works if everyone follows the rules, but it is easy to break. CSS Modules and `@scope` are like giving everyone their own office with a locked door: no naming rules are needed because no one can accidentally touch your documents. Naming conventions are a social solution; scoping mechanisms are a technical solution.

### Purposes

- **BEM:** To provide a systematic, predictable naming convention that reduces class name collisions.
- **BEM:** To make the relationship between blocks, elements, and modifiers explicit in the class name.
- **SMACSS:** To categorise CSS rules by purpose (base, layout, module, state, theme) for better organisation.
- **SMACSS:** To separate concerns between layout, components, and states.
- **CSS-scoping mechanisms:** To provide technical isolation without relying on developer discipline.
- **Hybrid approach:** To combine naming conventions with scoping mechanisms for both clarity and isolation.

### Syntax Rules and Structure

**BEM Naming:**
```css
/* Block */
.button { ... }

/* Element (part of a block) */
.button__icon { ... }

/* Modifier (variant) */
.button--primary { ... }
.button--large { ... }

/* Element with modifier */
.button__icon--spin { ... }
```

```jsx
function Button({ variant = 'default', size, children }) {
  const className = [
    'button',
    variant !== 'default' && `button--${variant}`,
    size && `button--${size}`,
  ].filter(Boolean).join(' ');

  return (
    <button className={className}>
      <span className="button__icon">{/* icon */}</span>
      {children}
    </button>
  );
}
```

**Component Breakdown:**
- `.button`: The Block (standalone component).
- `.button__icon`: The Element (part of the block).
- `.button--primary`: The Modifier (variant).
- Class names are constructed as strings in the component.

**SMACSS Categories:**
```css
/* Base — defaults, resets */
body { margin: 0; font-family: system-ui; }

/* Layout — page structure */
.l-header { ... }
.l-sidebar { ... }
.l-main { ... }

/* Module — reusable components */
.card { ... }
.card__title { ... }

/* State — JavaScript-dependent states */
.is-active { ... }
.is-disabled { ... }

/* Theme — visual themes */
.theme-dark { ... }
.theme-light { ... }
```

**Component Breakdown:**
- `.l-` prefix: Layout modules (page-level structure).
- `.is-` prefix: State classes applied dynamically via JavaScript.
- `.theme-` prefix: Theme classes applied at the root.
- Module class names are the component's base class.

**CSS-Scoping Mechanisms (Comparison):**
```css
/* CSS Modules — technical isolation */
.button {
  background: blue;
}
/* Compiles to: .Button_button__a1b2c */

/* @scope — selector isolation */
@scope (.card) {
  p { color: #333; }
}

/* Cascade layers — priority isolation */
@layer reset, vendor, design-system, utilities;
```

**Component Breakdown:**
- CSS Modules: No naming convention needed; the build tool generates unique names.
- `@scope`: No naming convention needed; selectors are limited to a subtree.
- Cascade layers: No naming convention needed; priority is controlled by layer order.

**Syntax Rules:**
- **BEM:** Use `block__element--modifier` syntax; avoid nesting more than one level of elements.
- **SMACSS:** Categorise rules into Base, Layout, Module, State, Theme; use prefixes (`l-` for layout, `is-` for state).
- **CSS Modules:** Use `.module.css` files and import the styles object.
- **`@scope`:** Use `@scope (root) to (limit) { ... }` for selector isolation.
- **Cascade Layers:** Declare `@layer` order at the top of the global stylesheet.
- **Hybrid:** Use BEM for component class names and `@layer` for priority control.

**Constraints and Limitations:**
- BEM class names become long and unwieldy for deeply nested components (e.g., `app-user-overview__status--is-authenticating`).
- BEM requires discipline; "not fully adhering to the naming rules breaks the system's structure".
- SMACSS provides categories but "does not present any specific syntax"; it is a philosophy, not a strict convention.
- CSS-scoping mechanisms require build tooling (CSS Modules) or browser support (`@scope`, `@layer`).
- Naming conventions do not provide technical isolation; a developer can still accidentally use the wrong class name.

### Annotated Code Example: BEM + Cascade Layers

```css
/* app/globals.css */
@layer reset, vendor, components, utilities;

/* BEM naming inside a layer */
@layer components {
  .button {
    display: inline-flex;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
  }

  .button--primary {
    background: #007bff;
    color: white;
  }

  .button--secondary {
    background: #6c757d;
    color: white;
  }

  .button__icon {
    margin-right: 8px;
  }

  .button--disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
```

```jsx
function Button({ variant = 'primary', disabled, children }) {
  return (
    <button
      className={`button button--${variant} ${disabled ? 'button--disabled' : ''}`}
      disabled={disabled}
    >
      <span className="button__icon">{/* icon */}</span>
      {children}
    </button>
  );
}
```

**Expected Output:** A `<button>` with BEM class names (`.button`, `.button--primary`, `.button__icon`) inside the `components` layer, so it overrides vendor styles but is overridden by utilities.

**Why This Output Occurs:** BEM provides predictable class names. The `@layer components` ensures these styles have higher priority than `vendor` but lower than `utilities`. The combination gives both naming clarity and priority control.

### Real-World Cases

- **Large teams:** BEM provides a shared vocabulary across teams.
- **Design systems:** SMACSS categories map to design system layers (base tokens, layout, components, states).
- **Legacy modernisation:** Introducing `@layer` to control BEM styles relative to legacy vendor CSS.
- **Component libraries:** CSS Modules + BEM-like naming for components shipped to multiple apps.

### References

- Encapsulation (UNSW) – BEM and CSS Modules: https://cgi.cse.unsw.edu.au/~cs6080/raw/lectures/react-css-custom.pdf
- SMACSS – Categorizing CSS Rules: https://smacss.com/book/categorizing
- Smashing Magazine – CSS @scope: https://www.smashingmagazine.com/2026/02/css-scope-alternative-naming-conventions/
- SmartTechDevs – Mastering CSS Cascade Layers in React: https://smarttechdevs.in/blog/nextjs-css-cascade-layers-style-isolation

---

## Core Concept 5: Modern Native CSS Integration — CSS Variables for Dynamic Theme Switching

### Definitions

**Core Definition:** CSS Custom Properties (CSS Variables) are runtime-dynamic values defined with the `--` prefix that can be read and written by JavaScript, enabling theme switching by changing variable values on the document root.

**Technical Definition:** CSS custom properties are "reusable values in CSS files… preceded by `--` followed by an identifier and can store values such as colors, lengths, or fonts". They are inherited by descendants and can be scoped to any selector, including `:root`, `[data-theme="dark"]`, or a component class. Dynamic theme switching is implemented by defining theme-specific values under selectors like `.dark` and `.light` (or `[data-theme="dark"]` and `[data-theme="light"]`), and toggling the class or attribute on the `<html>` element. React manages the theme state (usually via Context API) and persists the preference in `localStorage`; a `useEffect` hook updates the `data-theme` attribute on the document root. CSS rules reference `var(--color-primary)` instead of hardcoded values, so switching the attribute re-points all variables in the same frame.

**Beginner-Friendly Explanation:** Imagine your app's colours are written on sticky notes. When you switch from light to dark mode, you don't repaint the walls—you just swap the sticky notes for a different set. CSS variables are the sticky notes: `--bg-color`, `--text-color`, `--primary-color`. React flips the switch (changes the `data-theme` attribute), and every component that references `var(--bg-color)` instantly shows the new colour. No component needs to re-render—the browser handles the change.

### Purposes

- To define reusable, semantic colour tokens (e.g., `--color-bg-primary`) instead of hardcoded values.
- To enable runtime theme switching (light/dark, brand themes) without re-rendering components.
- To scope theme values to different subtrees (e.g., a dark sidebar inside a light page).
- To persist theme preference across sessions via `localStorage`.
- To integrate with design systems that generate CSS variables from design tokens.

### Syntax Rules and Structure

**Defining Theme Variables:**
```css
/* app/globals.css */
:root {
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f5f5f5;
  --color-text-primary: #1a1a1a;
  --color-text-secondary: #6c757d;
  --color-border: #dee2e6;
  --color-brand: #007bff;
}

[data-theme="dark"] {
  --color-bg-primary: #1a1a1a;
  --color-bg-secondary: #2d2d2d;
  --color-text-primary: #f5f5f5;
  --color-text-secondary: #adb5bd;
  --color-border: #495057;
  --color-brand: #4dabf7;
}
```

**Component Breakdown:**
- `:root { ... }`: Default (light) theme variables.
- `[data-theme="dark"] { ... }`: Dark theme overrides.
- All components reference `var(--color-...)`.

**Using Variables in Components:**
```css
/* Card.module.css */
.card {
  background: var(--color-bg-primary);
  color: var(--color-text-primary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 16px;
}

.title {
  color: var(--color-brand);
}
```

**React Theme Provider:**
```jsx
import { createContext, useContext, useEffect, useState } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [isDark, setIsDark] = useState(() => {
    const saved = localStorage.getItem('theme');
    return saved === 'dark';
  });

  useEffect(() => {
    document.documentElement.setAttribute(
      'data-theme',
      isDark ? 'dark' : 'light'
    );
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [isDark]);

  const toggleTheme = () => setIsDark(!isDark);

  return (
    <ThemeContext.Provider value={{ isDark, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

**Component Breakdown:**
- `isDark`: Boolean theme state, initialised from `localStorage`.
- `useEffect`: Sets `data-theme` on `<html>` and persists to `localStorage`.
- `toggleTheme`: Flips the theme state.
- `useTheme`: Custom hook for consuming the theme.

**Theme Toggle Component:**
```jsx
function ThemeToggle() {
  const { isDark, toggleTheme } = useTheme();
  return (
    <button onClick={toggleTheme}>
      {isDark ? '☀️ Light Mode' : '🌙 Dark Mode'}
    </button>
  );
}
```

**Component Breakdown:**
- `isDark`: Reads the current theme.
- `toggleTheme`: Changes the theme.
- The button label reflects the current theme.

**Syntax Rules:**
- Define theme variables under `:root` (default) and `[data-theme="dark"]` (dark).
- Use semantic variable names (`--color-bg-primary`) rather than literal names (`--blue-500`).
- Reference variables with `var(--variable-name)` in all component styles.
- Use React Context to manage the theme state globally.
- Persist the theme preference in `localStorage` and initialise from it.
- Set `data-theme` on `document.documentElement` (the `<html>` element).
- Avoid inline `style` for theming; use variables in CSS files for maintainability.
- For SSR, inject the theme attribute server-side to avoid a flash of incorrect theme.

**Constraints and Limitations:**
- CSS variables are inherited; a variable set on `:root` is available everywhere, but a variable set on a component is only available to its descendants.
- CSS variables do not work in IE11 or other very old browsers.
- Changing a variable triggers a style recalculation but does not re-render React components; this is a performance advantage but means React state and CSS state can diverge if not carefully managed.
- For SSR, the server must render the correct theme attribute; otherwise, the client will flash the default theme.
- CSS variables cannot be used in media query conditions (e.g., `@media (min-width: var(--breakpoint))` is invalid).

### Annotated Code Example: Complete Theme System

```css
/* app/globals.css */
:root {
  --color-bg: #ffffff;
  --color-surface: #f8f9fa;
  --color-text: #212529;
  --color-text-muted: #6c757d;
  --color-primary: #007bff;
  --color-border: #dee2e6;
  --radius: 8px;
  --shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

[data-theme="dark"] {
  --color-bg: #121212;
  --color-surface: #1e1e1e;
  --color-text: #e9ecef;
  --color-text-muted: #adb5bd;
  --color-primary: #4dabf7;
  --color-border: #343a40;
  --shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
}
```

```css
/* Card.module.css */
.card {
  background: var(--color-surface);
  color: var(--color-text);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 16px;
}

.title {
  color: var(--color-primary);
  margin: 0 0 8px;
}
```

```jsx
// App.tsx
import { ThemeProvider, useTheme } from './ThemeProvider';
import styles from './Card.module.css';

function Card() {
  return (
    <div className={styles.card}>
      <h2 className={styles.title}>Revenue</h2>
      <p>Monthly recurring revenue is up 12%.</p>
    </div>
  );
}

function ThemeToggle() {
  const { isDark, toggleTheme } = useTheme();
  return (
    <button onClick={toggleTheme}>
      {isDark ? 'Switch to Light' : 'Switch to Dark'}
    </button>
  );
}

export default function App() {
  return (
    <ThemeProvider>
      <ThemeToggle />
      <Card />
    </ThemeProvider>
  );
}
```

**Expected Output:** A card with a title and text. Clicking the toggle switches between light and dark themes. The card's background, text, border, and shadow all change instantly without a page reload or component re-render.

**Why This Output Occurs:** The `ThemeProvider` sets `data-theme="dark"` or `data-theme="light"` on `<html>`. The CSS variables in `[data-theme="dark"]` override those in `:root`. The card's styles reference `var(--color-surface)`, `var(--color-text)`, etc., so they automatically update when the theme attribute changes. React only re-renders the `ThemeToggle` button (because its state changed); the `Card` component does not re-render, yet its appearance changes.

### Real-World Cases

- **Light/dark mode:** The most common use case; persists across sessions with `localStorage`.
- **Multi-brand theming:** Different brands (e.g., "Blue Business", "Green Eco", "Purple Creative") defined as separate variable sets.
- **White-label apps:** Tenants can customise colours, fonts, and radii without code changes.
- **Design system tokens:** CSS variables generated from design tokens (Figma, Style Dictionary).
- **Scoped theming:** A dark sidebar inside a light page by scoping variables to the sidebar's container.

### References

- CoreUI – How to Implement Dark Mode in React: https://coreui.io/answers/how-to-implement-dark-mode-in-react/
- Basedash – How We Built Light Mode Without Tailwind's dark: Class: https://old.basedash.com
- Syncfusion – Themes using CSS Variables in React: https://ej2.syncfusion.com
- Tencent Cloud – Dynamic Theme Switching in Next.js: https://cloud.tencent.cn/developer/article/2609520

---

## Core Concept 6: Responsive Styling Foundations — Media Queries vs. Container Queries

### Definitions

**Core Definition:** Media queries respond to the viewport size (or user preferences like `prefers-color-scheme`), while container queries respond to the size of a specific ancestor container, enabling truly portable, context-aware components.

**Technical Definition:** Media queries (`@media`) evaluate the browser viewport's dimensions and user preferences. Container queries (`@container`) evaluate the size of a specific ancestor element that has been declared as a query container via `container-type: inline-size` (or `size`). Container queries make components "self-contained and truly reusable" because the component adapts to its own container, not the viewport. Container query units (`cqi`, `cqb`, `cqw`, `cqh`) allow fluid typography and spacing relative to the container. Style queries (`@container style(--density: compact)`) query custom property values. Browser support for size queries is Baseline 2023+, while style queries are newer.

**Beginner-Friendly Explanation:** Media queries ask: "How wide is the browser window?" Container queries ask: "How wide is the box I'm in?" This distinction matters for reusable components. A card in a sidebar should look different from a card in the main content area, even if the browser window is the same width. Media queries cannot tell the difference; container queries can. Use media queries for page-level layout (navigation, grid columns) and container queries for reusable components (cards, widgets, forms).

### Purposes

- **Media queries:** To make page-level layout decisions (hide navigation, change grid columns, adjust root font size).
- **Media queries:** To respond to user preferences (`prefers-color-scheme`, `prefers-reduced-motion`, `hover`, `orientation`).
- **Container queries:** To make reusable components adapt to their available space.
- **Container queries:** To eliminate modifier class explosion (`.card--compact`, `.card--sidebar`) by letting the component own its breakpoints.
- **Container query units:** To create fluid typography and spacing that scales with the component, not the window.
- **Style queries:** To query custom property values (e.g., `--density: compact`) for component variants.

### Syntax Rules and Structure

**Media Query (Page-Level):**
```css
/* Page layout changes at viewport breakpoints */
@media (min-width: 768px) {
  .sidebar {
    display: block;
    width: 250px;
  }
  .main {
    display: flex;
    flex: 1;
  }
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #121212;
  }
}
```

**Component Breakdown:**
- `@media (min-width: 768px)`: Viewport is at least 768px wide.
- `@media (prefers-color-scheme: dark)`: User prefers a dark colour scheme.
- Media queries are global; every component responds to the same viewport.

**Container Query (Component-Level):**
```css
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
    gap: 16px;
  }
  .card__image {
    width: 150px;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
  .card__image {
    width: 100%;
  }
}
```

**Component Breakdown:**
- `container-type: inline-size`: Declares the element as a query container for inline-size queries.
- `container-name: card`: Optional name to disambiguate in nested containers.
- `@container card (min-width: 400px)`: Applies when the container is at least 400px wide.
- The component adapts to its own container, not the viewport.

**Container Query Units:**
```css
.card__title {
  font-size: clamp(1rem, 4cqi, 1.5rem);
}
```

**Component Breakdown:**
- `cqi`: 1% of the container's inline size.
- `clamp(1rem, 4cqi, 1.5rem)`: Fluid font size that scales with the container, bounded by min and max.
- The title's font size adapts to the card's width, not the viewport.

**Style Queries:**
```css
.card-slot {
  container-name: card;
  --density: comfortable;
}

@container card style(--density: compact) {
  .card {
    padding: 0.5rem;
  }
}
```

**Component Breakdown:**
- `style(--density: compact)`: Queries the container's custom property value.
- Useful for density variants without modifier classes.

**React Component with Container Queries (Tailwind):**
```jsx
function Card() {
  return (
    <div className="@container">
      <div className="flex flex-col @md:flex-row gap-4">
        <img className="w-full @md:w-40" src={img} alt="" />
        <div>
          <h3 className="text-lg @md:text-xl">{title}</h3>
          <p>{description}</p>
        </div>
      </div>
    </div>
  );
}
```

**Component Breakdown:**
- `@container`: Tailwind's container query root.
- `@md:flex-row`: Applies `flex-row` when the container is at least `md` width.
- The same component works in a sidebar and a main column without modifier classes.

**Syntax Rules:**
- Use **media queries** for page chrome, global typography, `prefers-*` features, print, and orientation.
- Use **container queries** for reusable components (cards, widgets, forms) that can appear in multiple widths.
- Declare `container-type: inline-size` on the wrapper element; forgetting it means queries never match.
- Use `container-name` for nested containers to avoid ambiguity.
- Use `cqi`/`cqw` for fluid typography and spacing relative to the container.
- Use `clamp()` with `cqi` to create fluid scales with min/max bounds.
- Use `@supports (container-type: inline-size)` for progressive enhancement.
- Style queries are newer than size queries; treat them as progressive enhancement.

**Constraints and Limitations:**
- Container queries require a defined container (`container-type`); forgetting it means the query never matches.
- Size containment (`container-type: size`) can affect how percentages and overflowing content behave; `inline-size` is usually safer.
- Container queries are not a full replacement for media queries; dark mode, reduced motion, and print still belong in `@media`.
- Nested containers need clear `container-name`s or queries may attach to the wrong ancestor.
- Browser support for `@container` is Baseline 2023+; Firefox supports it, but older browsers do not.
- Media queries remain better for page-level decisions and user preference features.

### Annotated Code Example: Responsive Card with Container Queries

```css
/* Card.module.css */
.wrapper {
  container-type: inline-size;
  container-name: card;
}

.card {
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  overflow: hidden;
}

/* Narrow container: vertical layout */
@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
  .image {
    width: 100%;
    aspect-ratio: 16 / 9;
    object-fit: cover;
  }
  .title {
    font-size: clamp(1rem, 4cqi, 1.25rem);
  }
}

/* Wide container: horizontal layout */
@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
    gap: 16px;
  }
  .image {
    width: 160px;
    height: 160px;
    object-fit: cover;
    flex-shrink: 0;
  }
  .title {
    font-size: clamp(1.25rem, 3cqi, 1.5rem);
  }
}
```

```jsx
import styles from './Card.module.css';

export function Card({ image, title, description }) {
  return (
    <div className={styles.wrapper}>
      <div className={styles.card}>
        <img className={styles.image} src={image} alt="" />
        <div className={styles.content}>
          <h3 className={styles.title}>{title}</h3>
          <p>{description}</p>
        </div>
      </div>
    </div>
  );
}
```

**Expected Output:** The card renders vertically in a narrow container (sidebar) and horizontally in a wide container (main column). The title font size scales fluidly with the container width via `cqi`.

**Why This Output Occurs:** The `.wrapper` declares `container-type: inline-size`, making it a query container. The `@container card (max-width: 399px)` and `@container card (min-width: 400px)` rules apply based on the container's width, not the viewport. The `cqi` unit in `clamp()` makes the title's font size relative to the container's inline size. The same component works in both contexts without modifier classes.

### Real-World Cases

- **Design system cards:** Cards that adapt to sidebars, modals, and main columns without variant classes.
- **Dashboard widgets:** Widgets that reflow based on their grid cell width.
- **E-commerce product cards:** Product cards that switch between horizontal and vertical layouts.
- **CMS components:** Modular components that can be placed in different container widths.
- **Netflix UI:** Uses container queries for dynamic layouts across devices.

### References

- Smashing Magazine – Container Queries: https://www.smashingmagazine.com/2026/09/
- web.dev – Unlocking the Power of CSS Container Queries: https://web.dev
- GitHub – CSS Container Queries Guide (LucaNerlich): https://github.com/LucaNerlich/lucanerlich.com/pull/53
- SitePoint – CSS Container Queries + Subgrid: https://www.sitepoint.com
- OrchestKit – Responsive Patterns: https://raw.githubusercontent.com/yonatangross/orchestkit/main/src/skills/responsive-patterns/SKILL.md

---

## Comparison and Decision Guidance

| Concern | Recommended Approach | When to Use | Key Risk |
|---|---|---|---|
| **Global CSS priority** | Cascade Layers (`@layer`) | Any app with third-party CSS or resets | Unlayered styles beat layered styles |
| **Component style isolation** | CSS Modules | Component libraries, design systems | Does not isolate inherited properties |
| **Selector isolation without tooling** | `@scope` | Embedding third-party HTML, email previews | Firefox support still maturing |
| **Naming conventions** | BEM / SMACSS | Large teams, legacy codebases | Verbose; discipline required |
| **Dynamic theming** | CSS Custom Properties + Context | Light/dark mode, multi-brand | SSR flash without server-side attribute |
| **Component responsiveness** | Container Queries | Cards, widgets, reusable components | Requires `container-type` on wrapper |
| **Page-level responsiveness** | Media Queries | Layout, navigation, `prefers-*` | Cannot respond to component context |
| **Fluid typography** | `clamp()` + `cqi` | Component-scoped type scales | Over-reliance can harm readability |

**Decision Guidance:**
- **Start with cascade layers** to establish explicit priority order between resets, vendor styles, your design system, and utilities.
- **Use CSS Modules** for component-level style scoping; they eliminate class name collisions without naming conventions.
- **Use `@scope`** for selector isolation when you cannot use a build tool, or for embedding third-party HTML.
- **Use BEM or SMACSS** alongside scoping mechanisms for naming clarity, but do not rely on them alone for isolation.
- **Use CSS Custom Properties** for all theming; define semantic tokens under `:root` and `[data-theme="dark"]`.
- **Use container queries** for reusable components that must adapt to their container; use media queries for page-level layout and user preferences.
- **Combine `clamp()` with `cqi`** for fluid typography that scales with the component.
- **Always declare `container-type: inline-size`** on wrappers where container queries are used.
- **Test with `@supports`** for progressive enhancement of newer features.

---

## References

- MDN Web Docs – CSS Cascade Layers: https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Cascade_layers
- MDN Web Docs – @scope: https://developer.mozilla.org/en-US/docs/Web/CSS/@scope
- MDN Web Docs – CSS Custom Properties: https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties
- MDN Web Docs – Container Queries: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries
- CSS Modules – README: https://raw.githubusercontent.com/css-modules/css-modules/8d33532b29327386173b28051d289e9d25545e1f/README.md
- CSS Modules – Composition: https://github.com/Terro216/css-modules/blob/master/docs/composition.md
- SMACSS – Categorizing CSS Rules: https://smacss.com/book/categorizing
- Smashing Magazine – CSS @scope: https://www.smashingmagazine.com/2026/02/css-scope-alternative-naming-conventions/
- SmartTechDevs – Mastering CSS Cascade Layers in React: https://smarttechdevs.in/blog/nextjs-css-cascade-layers-style-isolation
- Mantine – Styles Order: https://help.mantine.dev/q/styles-order
- React Spectrum – v0.7.0 Release Notes: https://react-spectrum.adobe.com/releases/v0-7-0.md
- CoreUI – How to Implement Dark Mode in React: https://coreui.io/answers/how-to-implement-dark-mode-in-react/
- Frontend Masters – @scope and HTML Style Blocks: https://frontendmasters.com/blog/reminder-that-scope-and-html-style-blocks-are-a-potent-combo/
- W3C CSSWG – CSS Cascade 6: https://drafts.csswg.org/css-cascade-6/#scoped-styles
- GitHub – CSS Container Queries Guide (LucaNerlich): https://github.com/LucaNerlich/lucanerlich.com/pull/53
- web.dev – Unlocking the Power of CSS Container Queries: https://web.dev
- SitePoint – CSS Container Queries + Subgrid: https://www.sitepoint.com
- OrchestKit – Responsive Patterns: https://raw.githubusercontent.com/yonatangross/orchestkit/main/src/skills/responsive-patterns/SKILL.md
- Caisy – Styled Components vs CSS Modules: https://caisy.io
- Syncfusion – Themes using CSS Variables in React: https://ej2.syncfusion.com