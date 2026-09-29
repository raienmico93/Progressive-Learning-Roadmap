# CSS Variable Scope — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Variable Scope is the system of rules that determines which custom properties are visible to which elements, how values cascade and inherit through the DOM tree, how variables pierce Shadow DOM boundaries, how dependency chains between variables are resolved, and how authors can isolate internal component variables from external interference.

**Technical Definition:** Custom properties are scoped to the element(s) they are declared on, and participate in the cascade: the value of such a custom property is that from the declaration decided by the cascading algorithm. The selector given to the ruleset defines the scope in which the custom property can be used. Custom properties inherit by default from parent to child, and any declaration that does not have an inherited value available falls back to the property's initial value (or the registered `initial-value` if using `@property`). Custom properties are the exception to Shadow DOM encapsulation: they pierce the shadow boundary, allowing external styles to pass values into components. Dependency chains are resolved at computed-value time; if a dependency cycle is created, all the declarations that directly contribute to the cycle define invalid variables. The private property pattern uses a naming convention (typically a leading underscore, e.g., `--_padding`) to signal that a custom property is internal to a component and should not be overridden externally.

**Beginner-Friendly Explanation:** A CSS custom property is like a labelled box of values. The scope of that box determines who can open it. If you put the box on the `<html>` element, everyone can see it — that is a global scope. If you put it on a specific card element, only that card and its children can see it — that is a local scope. Custom properties also inherit, so a child element automatically sees its parent's custom properties. In Web Components, custom properties are the only CSS feature that can cross the Shadow DOM wall, making them the standard way to theme components from the outside. Dependency chains let one property reference another, but if you create a loop (A depends on B, B depends on A), the browser marks both as invalid. And the private property pattern lets you mark a property as "internal use only" with an underscore prefix, so other developers know not to override it.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Cascade-driven** | Custom properties participate in the cascade like any other CSS property. |
| **Inherited by default** | Custom properties inherit from parent to child unless `inherits: false` is registered. |
| **Selector-scoped** | The selector on which a custom property is declared defines its scope. |
| **Shadow-piercing** | Custom properties are the only CSS feature that crosses Shadow DOM boundaries. |
| **Cycle-aware** | Dependency cycles invalidate all declarations in the cycle. |
| **Convention-based privacy** | The `--_` prefix is a naming convention, not a language-enforced privacy mechanism. |

---

### Prerequisites

Before studying CSS Variable Scope, you should understand:

- **CSS Custom Properties** — declaration, `var()`, and `@property`.
- **CSS Cascade and Specificity** — how declarations compete and win.
- **CSS Inheritance** — how properties pass from parent to child.
- **Web Components and Shadow DOM** — the encapsulation model for custom elements.
- **CSS Selectors** — class, element, attribute, and pseudo-class selectors.

---

### Related Programming Areas

- **Web Components** — custom properties are the theming API for shadow-encapsulated components.
- **Design Systems** — scoping tokens appropriately prevents leakage and conflicts.
- **CSS Architecture** — private properties and scoped variables are core to component encapsulation.
- **JavaScript Frameworks** — Vue, React, and Svelte manage scope through components and CSS-in-JS.

---

### Core Concepts / Features

1. Global vs. Local Scopes: `:root` vs. Scoped Blocks
2. Inheritance and Shadow DOM Boundaries
3. Dependency Chains and Cycles
4. The Private Property Pattern

---

## 1. Global vs. Local Scopes: Targeting the Root Context Using `:root` Compared to Scoping Blocks Within Classes, Elements, or Pseudo-States

### Definitions

**Core Definition:** Global scope refers to custom properties declared on the `:root` (or `html`) element, making them available throughout the entire document via inheritance. Local scope refers to custom properties declared on a specific selector (class, element, or pseudo-state), making them available only within that element's subtree.

**Technical Definition:** The `:root` pseudo-class matches the document's root element (typically `<html>`). Custom properties declared there cascade to every element in the document, because every element is a descendant of the root. Local scoping is achieved by declaring custom properties on any other selector. Because custom properties inherit, the declaration on a parent element makes the value available to all descendants of that element, but not to siblings or ancestors. The cascade determines which declaration wins when the same custom property is declared at multiple levels; a declaration closer to the element (higher specificity or later in the cascade) overrides an inherited value.

**Beginner-Friendly Explanation:** Global custom properties live on the `<html>` element and are visible everywhere. Local custom properties live on a specific element — like a card or a navigation bar — and are only visible inside that element. If you declare `--color: blue` on a card, only the card and its children see blue. The rest of the page is unaffected. If the card also inherits a `--color` from `:root`, the card's own declaration wins because it is closer to the element. This is how you create component-specific themes without breaking the rest of the page.

---

### Purposes

- To make design tokens available throughout the entire document (`:root`).
- To scope component-specific values to a single component or subtree.
- To override global tokens for a specific section or component.
- To use the cascade to determine which declaration wins.
- To reduce global namespace pollution by keeping values local.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Global scope: available everywhere via inheritance */
:root {
    --global-color: #3498db;
}

/* Local scope: available only within .card and its descendants */
.card {
    --card-color: #e74c3c;
}

/* Local scope with pseudo-state: only when hovering */
.button {
    --button-bg: #3498db;
}
.button:hover {
    --button-bg: #2980b9;
}

/* Local scope within a media query */
@media (min-width: 768px) {
    :root {
        --spacing: 2rem;
    }
}
```

#### Component Breakdown

| Scope | Selector | Visibility | Use Case |
|---|---|---|---|
| Global | `:root` | Entire document | Design tokens, theme colours, spacing scales |
| Component | `.card` | Card and descendants | Component-specific overrides |
| State | `:hover`, `:focus` | Element in that state | Hover/focus variants |
| Conditional | `@media` | Matching viewport | Responsive token changes |

#### Syntax Rules

1. `:root` is a pseudo-class matching the document root element; it has higher specificity than `html`.
2. Custom properties declared on `:root` are inherited by every element in the document.
3. Custom properties declared on any other selector are inherited only by that element's descendants.
4. The cascade determines the winning declaration when the same property is declared at multiple levels.
5. Local declarations override inherited values for the declaring element and its descendants.
6. The `@property` at-rule can set `inherits: false`, preventing inheritance even when the property is declared on a parent.
7. Media queries and container queries can change custom property values conditionally.

#### Constraints and Limitations

- **No true global scope** — even `:root` declarations are inherited, not global. A custom property declared on `:root` is not available to elements that do not descend from the root (e.g., elements in a separate document or a detached tree).
- **Specificity matters** — a local declaration with higher specificity overrides a `:root` declaration.
- **Media query limitations** — `var()` cannot be used inside media query conditions, so breakpoints must be literal values.
- **No leak prevention without `@property`** — unregistered custom properties always inherit unless `inherits: false` is registered.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Global vs. Local Scoping

**HTML File (`scope.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Variable Scope</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="scope.css">
</head>
<body>
    <!-- Card with local scope override -->
    <div class="card card-danger">
        <h2>Danger Card</h2>
        <p>This card overrides --accent locally.</p>
    </div>
    <!-- Card with default global accent -->
    <div class="card">
        <h2>Default Card</h2>
        <p>This card uses the global accent.</p>
    </div>
</body>
</html>
```

**CSS File (`scope.css`):**

```css
/* GLOBAL SCOPE: available everywhere */
:root {
    --accent: #3498db;
    --text-primary: #111827;
    --surface: #ffffff;
    --radius: 12px;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card {
    /* The card inherits global tokens but can override them */
    background-color: var(--surface);
    color: var(--text-primary);
    border-left: 4px solid var(--accent);
    border-radius: var(--radius);
    padding: 30px;
    margin-bottom: 20px;
    max-width: 400px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card h2 {
    color: var(--accent);
    margin-top: 0;
}

/* LOCAL SCOPE: overrides --accent only within .card-danger */
.card-danger {
    --accent: #e74c3c;
    /* --text-primary and --surface are still inherited from :root */
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `scope.html`.
3. Save the CSS code as `scope.css` in the same folder.
4. Open `scope.html` in a web browser.
5. Observe: the first card has a red accent (local override), and the second card has a blue accent (global value).

**Expected Output:** Two cards. The first card uses the locally scoped `--accent` (red), while the second card inherits the global `--accent` (blue). Both cards share the same `--text-primary`, `--surface`, and `--radius` from `:root`.

**Why This Works:** The `:root` selector declares global tokens. The `.card-danger` class overrides only `--accent`, creating a local scope. Because custom properties inherit, the override affects only the card and its descendants — the second card is unaffected.

---

### Real-World Cases

- **Global design tokens:** Colours, spacing, and typography declared on `:root`.
- **Component variants:** `.button--danger` overrides `--button-bg` locally.
- **Responsive theming:** Media queries change `--spacing` on `:root` for different viewports.
- **State-based styling:** `:hover` and `:focus` change custom property values for interactive states.

---

## 2. Inheritance and Shadow DOM Boundaries: How Variables Cascade Down the DOM Tree and Pierce Encapsulated Shadow DOM Boundaries

### Definitions

**Core Definition:** Custom properties inherit from parent to child through the DOM tree, including across Shadow DOM boundaries. This makes them the primary mechanism for theming Web Components from the outside, even though Shadow DOM is designed to encapsulate styles.

**Technical Definition:** CSS custom properties are inherited properties. When an element does not have a declared value for a custom property, it inherits the computed value from its parent. This inheritance chain crosses Shadow DOM boundaries: the top-level elements of a shadow tree inherit from their host element. Custom CSS properties are able to pierce the Shadow DOM boundary and can be used to style elements from outside of a component itself. To prevent inheritance, a custom property must be registered with `inherits: false` using `@property` or `CSS.registerProperty()`. The `:host` selector within a shadow tree can read inherited custom properties and apply them to the component's internal elements.

**Beginner-Friendly Explanation:** Imagine a Web Component as a sealed box with its own private CSS. Nothing from outside can reach inside — except custom properties. They slip through the cracks because they inherit. If you set `--brand-color: blue` on the page, and the component uses `var(--brand-color)` internally, the component sees blue. This is why custom properties are the standard way to theme Web Components: they are the only CSS feature that crosses the Shadow DOM wall.

---

### Purposes

- To provide a theming API for Web Components without breaking encapsulation.
- To allow external styles to pass values into shadow-encapsulated components.
- To inherit design tokens from the document root into component internals.
- To control whether a custom property inherits using `@property`.
- To support fallback values when no external value is provided.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Outside the component */
:root {
    --brand-color: #3498db;
}

/* Inside the component's shadow DOM */
:host {
    display: block;
    background-color: var(--brand-color, #ccc);
}

.internal-element {
    color: var(--brand-color);
}

/* Preventing inheritance with @property */
@property --internal-only {
    syntax: "<color>";
    inherits: false;
    initial-value: #333;
}
```

#### Component Breakdown

| Concept | Description |
|---|---|
| Inheritance | Custom properties inherit from parent to child by default. |
| Shadow DOM piercing | Custom properties cross the shadow boundary; regular CSS does not. |
| `:host` | The shadow host element; reads inherited custom properties. |
| `inherits: false` | Prevents a registered property from inheriting. |
| Fallback | `var(--prop, fallback)` provides a default if the property is not set. |

#### Syntax Rules

1. Custom properties inherit by default from parent to child.
2. Inheritance crosses Shadow DOM boundaries: shadow tree top-level elements inherit from the host.
3. Custom properties are the **only** CSS feature that pierces Shadow DOM encapsulation.
4. `@property` with `inherits: false` prevents inheritance for registered properties.
5. The `:host` selector reads inherited custom properties inside the shadow tree.
6. Fallback values in `var()` provide defaults when the property is not set externally.
7. Unregistered custom properties always inherit; registration is required to disable inheritance.

#### Constraints and Limitations

- **No other CSS feature pierces Shadow DOM** — selectors, `!important`, and regular properties do not cross the boundary.
- **`all: initial` does not reset custom properties** — resetting a component's styles with `all: initial` does not reset custom properties; they must be reset explicitly.
- **Inheritance cannot be selectively blocked** — without `@property`, all custom properties inherit.
- **Performance** — inherited custom properties cause style recalculations in descendants when changed; `inherits: false` narrows the scope of recalculation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Theming a Web Component with Custom Properties

**HTML File (`shadow.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Shadow DOM Theming</title>
    <style>
        :root {
            --card-accent: #3498db;
            --card-bg: #ffffff;
        }
        .dark-context {
            --card-accent: #e74c3c;
            --card-bg: #1a1a1a;
        }
    </style>
</head>
<body>
    <my-card></my-card>
    <div class="dark-context">
        <my-card></my-card>
    </div>

    <script>
        class MyCard extends HTMLElement {
            connectedCallback() {
                const shadow = this.attachShadow({ mode: 'open' });
                shadow.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            background-color: var(--card-bg, white);
                            border-left: 4px solid var(--card-accent, #ccc);
                            border-radius: 12px;
                            padding: 24px;
                            margin: 16px 0;
                            font-family: system-ui, sans-serif;
                        }
                        h2 {
                            color: var(--card-accent, #ccc);
                            margin-top: 0;
                        }
                        p {
                            color: var(--card-bg, white) === '#1a1a1a' ? '#e8e8e8' : '#333';
                        }
                    </style>
                    <h2>Shadow Component</h2>
                    <p>This component is styled from outside via custom properties.</p>
                `;
            }
        }
        customElements.define('my-card', MyCard);
    </script>
</body>
</html>
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `shadow.html`.
2. Open in a browser that supports Web Components.
3. Observe: the first card has a blue accent and white background. The second card (inside `.dark-context`) has a red accent and dark background.

**Expected Output:** Two cards rendered from the same Web Component. The first uses the default `:root` values; the second uses the overridden values from `.dark-context`, even though the component is encapsulated in Shadow DOM.

**Why This Works:** The custom properties `--card-accent` and `--card-bg` are declared outside the component. When the component's shadow tree uses `var(--card-accent)`, the browser resolves the value from the host element's inherited custom properties. This demonstrates that custom properties pierce the Shadow DOM boundary, allowing external theming without breaking encapsulation.

---

### Real-World Cases

- **Design systems:** Theming Web Components from the page level.
- **Multi-brand platforms:** Different brands set different custom properties on `:root`.
- **Dark mode:** A `[data-theme="dark"]` selector overrides custom properties, affecting all components.
- **Component libraries:** Components expose a set of custom properties as their theming API.

---

## 3. Dependency Chains and Cycles: Building Cascading Variable Graphs and Avoiding Breaking Structural Loops

### Definitions

**Core Definition:** A dependency chain is a sequence in which one custom property references another (e.g., `--btn-bg: var(--brand-color)`). A dependency cycle occurs when two or more custom properties reference each other in a loop (e.g., `--a: var(--b); --b: var(--a);`). Cycles are resolved by marking all declarations in the cycle as invalid at computed-value time.

**Technical Definition:** Variables can refer to other variables in their value. If a dependency cycle is created, all the declarations that directly contribute to the cycle define invalid variables. Cyclic dependencies occur only when multiple data properties on the same element refer to each other in a loop. The resolution process works by building a dependency graph of custom properties and resolving them at computed-value time. If a cycle is detected, all declarations in the cycle become invalid, and the affected properties use their inherited or initial values. Fallback values in `var()` are also considered part of the dependency graph — a cycle can be created through fallback arguments.

**Beginner-Friendly Explanation:** A dependency chain is like a recipe: "the button colour is the brand colour." That is fine. But if you say "the brand colour is the button colour" and "the button colour is the brand colour," you have a loop with no starting point. The browser detects this and marks both as invalid. The fix is to break the loop by giving one of the properties a literal value instead of a reference.

---

### Purposes

- To create layered token systems where one token derives from another.
- To build component tokens that reference global tokens.
- To enable theming where component colours adapt to a brand colour.
- To understand why invalid variable errors occur and how to debug them.
- To design token architectures that avoid circular dependencies.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Valid dependency chain */
:root {
    --brand-color: #3498db;
    --btn-bg: var(--brand-color);
    --btn-text: white;
}

/* INVALID: dependency cycle */
:root {
    --a: var(--b);
    --b: var(--a);
    /* Both --a and --b are invalid */
}

/* INVALID: cycle through fallback */
:root {
    --x: var(--y, var(--x));
    /* --x is invalid */
}

/* Breaking the cycle */
:root {
    --a: var(--b);
    --b: red; /* Literal value breaks the cycle */
}
```

#### Component Breakdown

| Concept | Description | Example |
|---|---|---|
| Dependency chain | One property references another. | `--btn-bg: var(--brand-color)` |
| Direct cycle | A references B, B references A. | `--a: var(--b); --b: var(--a)` |
| Indirect cycle | A → B → C → A. | `--a: var(--b); --b: var(--c); --c: var(--a)` |
| Fallback cycle | Cycle through `var()` fallback argument. | `--x: var(--y, var(--x))` |
| Cycle resolution | All declarations in the cycle become invalid. | Falls back to inherited/initial value |

#### Syntax Rules

1. Custom properties can reference other custom properties via `var()`.
2. If a cycle is detected, all declarations that directly contribute to the cycle become invalid.
3. Cycles can be direct (A → B → A) or indirect (A → B → C → A).
4. Fallback arguments in `var()` are part of the dependency graph and can create cycles.
5. Cycles are resolved at computed-value time, after the cascade.
6. An invalid custom property results in the consuming property using its inherited or initial value.
7. Breaking a cycle requires at least one property in the chain to have a literal value.

#### Constraints and Limitations

- **Silent failure** — cycles do not throw errors; they silently invalidate properties.
- **Debugging difficulty** — cycles can be hard to trace in large token graphs.
- **Fallback participation** — fallbacks are part of the dependency graph, which can create unexpected cycles.
- **No partial resolution** — all declarations in the cycle are invalidated, not just the last one.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dependency Chain and Cycle Detection

**HTML File (`cycles.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dependency Chains and Cycles</title>
    <link rel="stylesheet" href="cycles.css">
</head>
<body>
    <div class="box valid">Valid Chain</div>
    <div class="box cycle">Cycle (Invalid)</div>
    <div class="box broken">Broken Cycle</div>
</body>
</html>
```

**CSS File (`cycles.css`):**

```css
:root {
    /* Valid dependency chain */
    --brand: #3498db;
    --btn-bg: var(--brand);
    --btn-hover: color-mix(in srgb, var(--btn-bg) 80%, black);

    /* INVALID: direct cycle */
    --cycle-a: var(--cycle-b);
    --cycle-b: var(--cycle-a);

    /* Broken cycle: --broken-b has a literal value */
    --broken-a: var(--broken-b);
    --broken-b: #27ae60;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.box {
    padding: 30px;
    border-radius: 12px;
    margin-bottom: 16px;
    font-weight: bold;
    text-align: center;
    color: white;
}

.valid {
    /* Uses the valid chain: --brand → --btn-bg */
    background-color: var(--btn-bg);
}

.cycle {
    /* --cycle-a is invalid, so background-color uses the initial value (transparent) */
    background-color: var(--cycle-a);
    color: #111827;
    border: 2px dashed #e74c3c;
}

.broken {
    /* --broken-a resolves to --broken-b (#27ae60) */
    background-color: var(--broken-a);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `cycles.html` and CSS as `cycles.css`.
2. Open in a browser.
3. Observe:
   - Box 1 (Valid Chain): blue background (`--brand` → `--btn-bg`).
   - Box 2 (Cycle): transparent background (invalid cycle).
   - Box 3 (Broken Cycle): green background (`--broken-b` breaks the cycle).

**Expected Output:** Three boxes demonstrating the outcomes of valid, cyclic, and broken dependency chains.

**Why This Works:** The valid chain resolves `--btn-bg` to `--brand` (blue). The cycle creates two invalid properties, so `--cycle-a` is invalid and `background-color` falls back to transparent. The broken cycle works because `--broken-b` has a literal value, breaking the loop.

---

### Real-World Cases

- **Token systems:** `--color-primary` → `--button-bg` → `--button-hover`.
- **Theming:** Component tokens referencing global tokens that reference primitives.
- **Debugging:** Identifying why a component shows transparent or default styles.
- **Design systems:** Ensuring the token graph is acyclic by design.

---

## 4. The Private Property Pattern: Using Prefix Variations to Isolate Internal Component Variables

### Definitions

**Core Definition:** The private property pattern uses a naming convention — typically a leading underscore after the double dash (e.g., `--_padding`) — to signal that a custom property is intended for internal use within a component and should not be overridden by external styles.

**Technical Definition:** The private property pattern is a naming convention, not a language-enforced privacy mechanism. CSS does not have true private custom properties. The pattern was proposed during the development of CSS custom properties, where a leading underscore was suggested as a way to distinguish custom properties from language-defined names. Today, the `--_` prefix is widely adopted as a signal to other developers: "this property is internal to the component; do not override it." The convention is analogous to the underscore prefix in JavaScript and Python for "private" members, which are also not truly private but are respected by convention. Components expose their public API as custom properties without the underscore prefix (e.g., `--button-bg`), while internal implementation details use the underscore prefix (e.g., `--_button-padding`).

**Beginner-Friendly Explanation:** When you build a component, you want to expose some custom properties for theming (like `--button-bg`) but keep others internal (like `--_button-padding`). CSS does not have real private variables, so the underscore prefix is a signal: "this is my internal stuff, please do not touch it." Other developers see the underscore and know that changing it might break the component. It is a social convention, not a technical enforcement.

---

### Purposes

- To signal which custom properties are part of a component's public API.
- To prevent accidental overrides of internal component variables.
- To improve code readability and maintainability.
- To distinguish between themable tokens and implementation details.
- To follow established conventions in the CSS community.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Public API: no underscore prefix */
.button {
    --button-bg: var(--color-action-primary);
    --button-text: white;
    --button-radius: var(--radius-md);
}

/* Private/internal: underscore prefix */
.button {
    --_button-padding-block: 0.75rem;
    --_button-padding-inline: 1.5rem;
    --_button-transition: 200ms ease-out;
}

.button {
    background-color: var(--button-bg);
    color: var(--button-text);
    border-radius: var(--button-radius);
    padding: var(--_button-padding-block) var(--_button-padding-inline);
    transition: filter var(--_button-transition);
}
```

#### Component Breakdown

| Pattern | Prefix | Meaning | Example |
|---|---|---|---|
| Public | `--name` | Theming API; safe to override. | `--button-bg` |
| Private | `--_name` | Internal implementation; do not override. | `--_button-padding` |

#### Syntax Rules

1. The `--_` prefix is a **naming convention**, not a language feature.
2. CSS does not enforce privacy; any custom property can be overridden.
3. Public custom properties (no underscore) form the component's theming API.
4. Private custom properties (with underscore) are internal implementation details.
5. The convention is analogous to `_private` in JavaScript and Python.
6. Use `@property` with `inherits: false` to prevent private properties from leaking to descendants.
7. Document which properties are public and which are private in the component's API documentation.

#### Constraints and Limitations

- **No enforcement** — any developer can override a private property; the convention relies on respect.
- **No tooling support** — linters and IDE autocomplete do not distinguish private from public custom properties.
- **Team agreement required** — the convention only works if the team agrees to follow it.
- **`@property` can enforce inheritance** — registering a property with `inherits: false` can prevent it from leaking, but does not prevent direct overrides.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Public and Private Properties in a Button Component

**HTML File (`private.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Private Property Pattern</title>
    <link rel="stylesheet" href="private.css">
</head>
<body>
    <button class="btn">Default</button>
    <button class="btn btn--danger">Danger</button>
    <button class="btn btn--large">Large</button>
</body>
</html>
```

**CSS File (`private.css`):**

```css
:root {
    --color-action-primary: #3b82f6;
    --color-danger: #ef4444;
    --radius-md: 8px;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    gap: 16px;
    align-items: center;
}

/* ===== COMPONENT: Button ===== */
.btn {
    /* PUBLIC API: safe to override externally */
    --button-bg: var(--color-action-primary);
    --button-text: white;
    --button-radius: var(--radius-md);

    /* PRIVATE: internal implementation details */
    --_button-padding-block: 0.75rem;
    --_button-padding-inline: 1.5rem;
    --_button-font-size: 1rem;
    --_button-transition: filter 200ms;

    /* Consume public tokens */
    background-color: var(--button-bg);
    color: var(--button-text);
    border-radius: var(--button-radius);

    /* Consume private tokens */
    padding: var(--_button-padding-block) var(--_button-padding-inline);
    font-size: var(--_button-font-size);
    border: none;
    cursor: pointer;
    font-weight: bold;
    transition: var(--_button-transition);
}

.btn:hover {
    filter: brightness(0.9);
}

/* PUBLIC API: Danger variant overrides a public token */
.btn--danger {
    --button-bg: var(--color-danger);
}

/* PRIVATE: Large variant overrides private tokens */
.btn--large {
    --_button-padding-block: 1rem;
    --_button-padding-inline: 2rem;
    --_button-font-size: 1.2rem;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `private.html` and CSS as `private.css`.
2. Open in a browser.
3. Observe: the default button is blue, the danger button is red (via public token override), and the large button is larger (via private token override).

**Expected Output:** Three buttons demonstrating the public/private API split. The danger variant uses a public token, while the large variant uses private tokens — both are valid, but the private tokens are documented as internal.

**Why This Works:** The `.btn` component exposes `--button-bg`, `--button-text`, and `--button-radius` as public theming tokens. It uses `--_button-padding-block`, `--_button-padding-inline`, and `--_button-font-size` as private internal tokens. The `.btn--danger` variant overrides a public token, and the `.btn--large` variant overrides private tokens (which is acceptable within the component's own styles). External consumers should only override public tokens.

---

### Real-World Cases

- **Design systems:** Components document their public custom properties and use `--_` for internal implementation.
- **Component libraries:** Lit, Stencil, and other Web Component libraries encourage the private property pattern.
- **CSS frameworks:** Frameworks use the convention to signal which properties are safe to override.
- **Large codebases:** The convention helps new developers understand which properties are safe to change.

---

## References

- MDN Web Docs — Custom properties (`--*`) - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/--*
- MDN Web Docs — Using CSS custom properties (variables) - https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties
- MDN Web Docs — Registering custom properties in CSS - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Properties_and_values_API/Registering_properties
- W3C — CSS Custom Properties for Cascading Variables Module Level 1 - https://www.w3.org/TR/css-variables-1/
- W3C — CSS Custom Properties for Cascading Variables Module Level 1: Cycles - https://www.w3.org/TR/css-variables-1/#cycles
- Open Web Components — Styling: Styles Piercing Shadow DOM - https://open-wc.org/guides/knowledge/styling/styles-piercing-shadow-dom/
- CSS-Tricks — Breaking CSS Custom Properties out of `:root` Might Be a Good Idea - https://css-tricks.com/breaking-css-custom-properties-out-of-root-might-be-a-good-idea/
- W3C — `[css-variables]` Let's change the syntax - https://lists.w3.org/Archives/Public/www-style/2014Mar/0261.html
- W3C — CSS Shadow Module Level 1 - https://drafts.csswg.org/css-shadow-1/
- Can I Use — CSS Custom Properties - https://caniuse.com/css-variables
- Can I Use — `@property` - https://caniuse.com/mdn-css_at-rules_property