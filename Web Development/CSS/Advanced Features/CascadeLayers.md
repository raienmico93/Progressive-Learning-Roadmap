# CSS Cascade Layers — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Cascade Layers is a CSS module that gives authors explicit control over which styles win the cascade by allowing them to group style rules into named layers. Styles in later layers always beat styles in earlier layers, regardless of selector specificity.

**Technical Definition:** The CSS Cascading and Inheritance Level 5 specification introduces cascade layers as a way to explicitly control the cascade order. A cascade layer is a named group of style rules that are sorted together in the cascade. Layers are ordered by the order in which they are first declared. Within each origin (author, user, user agent), the cascade sorts declarations by layer order, then by specificity and order of appearance. Unlayered styles are treated as belonging to an implicit final layer that has the highest priority among normal declarations. When the `!important` flag is used, the priority order among layers is inverted: important declarations in earlier layers take precedence over important declarations in later layers.

**Beginner-Friendly Explanation:** Imagine you have several teams writing CSS for the same website. Without layers, the team with the most specific selectors (or the most `!important` flags) wins — a constant "specificity war." Cascade layers solve this by letting you declare a fixed order of priority: "reset styles come first, then base styles, then components, then utilities." No matter how specific a selector in the reset layer is, it can never override a component style, because the layer order decides. This makes large CSS codebases predictable and maintainable.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Explicit ordering** | Layer order is determined by the order in which layers are first declared. |
| **Specificity override** | Layer order overrides selector specificity for normal declarations. |
| **Unlayered priority** | Unlayered styles have the highest priority among normal declarations. |
| **Inverted `!important`** | For `!important` declarations, the layer priority order is reversed. |
| **Nested layers** | Layers can be nested using dot notation (`framework.base`). |
| **Third-party isolation** | External stylesheets can be imported directly into a layer. |
| **Architectural tool** | Enables ITCSS-style architectures without specificity hacks. |

---

### Prerequisites

Before studying CSS Cascade Layers, you should understand:

- **CSS Cascade and Specificity** — how declarations compete and win.
- **CSS Inheritance** — how properties pass from parent to child.
- **CSS At-Rules** — `@media`, `@supports`, and `@import`.
- **The `!important` Flag** — how it alters cascade priority.
- **CSS Custom Properties** — optional, but commonly used with layered architectures.

---

### Related Programming Areas

- **Design Systems** — cascade layers provide the ordering foundation for token-based systems.
- **CSS Architecture** — ITCSS, CUBE CSS, and other methodologies map naturally to layers.
- **Third-Party Integration** — frameworks and libraries can be isolated in their own layers.
- **Web Components** — layered styles can be scoped to component boundaries.
- **Build Tools** — PostCSS and Vite plugins can automatically wrap CSS modules in layers.

---

### Core Concepts / Features

1. The `@layer` At-Rule: Syntax and Declaration Methods
2. Layer Ordering Mechanics and Unlayered Style Priority
3. Third-Party CSS Isolation
4. Architectural Organization
5. Specificity Interactions and Inverted `!important` Priority

---

## 1. The `@layer` At-Rule: Declaring Explicit Sub-Layers

### Definitions

**Core Definition:** The `@layer` CSS at-rule is used to declare a cascade layer and to define the order of precedence when multiple cascade layers are present.

**Technical Definition:** Cascade layers can be declared in three ways. First, using an `@import` rule with the `layer` keyword or `layer()` function, assigning the contents of the imported file into that layer. Second, using a `@layer` block at-rule, assigning its child style rules into that layer. Third, using a `@layer` statement at-rule, declaring a named layer without assigning any rules. The statement form is particularly useful for establishing layer order upfront. Layer names are period-separated lists of identifiers; nested layers use dot notation (e.g., `framework.base`). Anonymous layers can be created but cannot be referenced subsequently.

**Beginner-Friendly Explanation:** You can create a layer in three ways: by importing a stylesheet into it, by wrapping a block of CSS in `@layer layer-name { ... }`, or by just declaring the layer name with `@layer layer-name;` to set its position in the order. The statement form is the most important for architecture — it lets you declare the entire layer order at the top of your stylesheet, so every subsequent layer block slots into the correct position.

---

### Purposes

- To declare a named cascade layer and assign style rules to it.
- To establish the order of precedence for multiple layers.
- To import external stylesheets directly into a specific layer.
- To nest layers for fine-grained control over sub-ordering.
- To enable predictable cascade behaviour without specificity hacks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Statement at-rule: declare layer order without assigning rules */
@layer reset, base, components, utilities;

/* Block at-rule: create a named layer with rules */
@layer components {
    .card { padding: 1rem; }
}

/* Anonymous layer block */
@layer {
    .box { border: 1px solid; }
}

/* Nested layers using dot notation */
@layer framework.base {
    p { margin-block: 0.75em; }
}

/* Import into a layer */
@import url('normalize.css') layer(reset);
@import url('bootstrap.css') layer(thirdparty.bootstrap);
```

#### Component Breakdown

| Form | Syntax | Purpose |
|---|---|---|
| Statement | `@layer name1, name2;` | Declares layer order only. |
| Block | `@layer name { rules }` | Creates a layer and assigns rules. |
| Anonymous | `@layer { rules }` | Creates an unnamed layer. |
| Nested | `@layer parent.child { rules }` | Creates a sub-layer. |
| Import | `@import url() layer(name);` | Imports a file into a layer. |

#### Syntax Rules

1. Layers are sorted by the order in which they first are declared.
2. Nested layers are grouped within their parent layers after any unlayered rules.
3. The outer layers are sorted first, with any unlayered style rules added to an implicit outer layer which has lower priority than the explicit layers for `!important` declarations, and higher priority for normal declarations.
4. Layer names are case-sensitive.
5. Anonymous layers cannot be referenced by name after creation.
6. The `@layer` statement form is the recommended way to establish order at the top of an entry file.
7. The `@import` rule with a layer must appear before all other rules except `@charset` and `@layer` statements.

#### Constraints and Limitations

- **Order is fixed at first declaration** — once a layer's position is established, it cannot be changed.
- **Unlayered styles win** — styles outside any layer always beat layered styles (for normal declarations).
- **No un-importing** — layers cannot be removed once declared.
- **Browser support** — `@layer` is Baseline widely available since March 2022.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Three Ways to Declare Layers

**HTML File (`layers.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cascade Layers</title>
    <link rel="stylesheet" href="layers.css">
</head>
<body>
    <div class="card">
        <h2>Layered Card</h2>
        <p>This card uses layered styles.</p>
    </div>
</body>
</html>
```

**CSS File (`layers.css`):**

```css
/* 1. Statement at-rule: declare layer order upfront */
@layer reset, base, components, utilities;

/* 2. Block at-rule: assign rules to a layer */
@layer reset {
    *, *::before, *::after {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }
}

@layer base {
    body {
        font-family: system-ui, sans-serif;
        background-color: #f5f5f5;
        padding: 40px;
    }
}

@layer components {
    .card {
        background-color: white;
        border-radius: 12px;
        padding: 30px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
        max-width: 400px;
    }
    .card h2 {
        color: #006064;
        margin-bottom: 8px;
    }
}

/* 3. Unlayered style: always wins over layered normal declarations */
.card {
    border-left: 4px solid #e74c3c;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `layers.html`.
3. Save the CSS code as `layers.css` in the same folder.
4. Open `layers.html` in a modern browser.
5. Observe that the card has a red left border (from the unlayered rule) even though the `.card` rule in the `components` layer has the same specificity.

**Expected Output:** A card with a red left border, white background, and teal heading. The unlayered `.card` rule overrides the layered `.card` rule regardless of specificity.

**Why This Works:** The statement `@layer reset, base, components, utilities;` establishes the order. The `components` layer contains the card styling. The unlayered `.card { border-left: ... }` is outside any layer, so it has the highest priority among normal declarations. This demonstrates that unlayered styles always win.

---

### Real-World Cases

- **Design systems:** Declaring `@layer reset, tokens, base, components, utilities;` at the top of the entry stylesheet.
- **Framework integration:** `@import url('bootstrap.css') layer(vendor);` to isolate framework styles.
- **Legacy migration:** Wrapping old CSS in a layer and placing it early in the order.
- **Component libraries:** Using nested layers like `components.button` and `components.card`.

---

## 2. Layer Ordering Mechanics: Precedence and Unlayered Styles

### Definitions

**Core Definition:** Layer ordering determines which layer's declarations win when multiple layers define the same property for the same element. Unlayered styles occupy a special position: they always beat layered normal declarations.

**Technical Definition:** Cascade layers are sorted by the order in which they first are declared. Unlayered styles are added to an implicit outer layer which has higher priority than explicit layers for normal declarations, and lower priority than explicit layers for `!important` declarations. This means that unlayered normal styles override all layered normal styles, regardless of specificity. When layers are nested, the outer layer order determines the primary sort, and nested layers are sorted within their parent.

**Beginner-Friendly Explanation:** Think of layers as a stack of transparent sheets. The sheet you place last is on top. But there is a special unlayered sheet that is always on top of all the layered sheets — unless you use `!important`, in which case the order flips and the unlayered sheet goes to the bottom. This is why unlayered styles are so powerful, and why you should use them sparingly.

---

### Purposes

- To establish a predictable order of precedence across style groups.
- To ensure that later layers override earlier layers regardless of specificity.
- To understand why unlayered styles always win.
- To design layer hierarchies that avoid conflicts.
- To debug cascade issues by identifying which layer is winning.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Establish order upfront */
@layer reset, base, components, utilities;

/* Later layers win */
@layer utilities {
    .text-center { text-align: center; }
}

/* Unlayered styles win over layered normal declarations */
.card { border: 1px solid red; }
```

#### Precedence Order (Normal Declarations)

| Priority | Source | Example |
|---|---|---|
| 1 (lowest) | `reset` layer | Browser reset styles |
| 2 | `base` layer | Element defaults |
| 3 | `components` layer | Component styles |
| 4 | `utilities` layer | Utility classes |
| 5 (highest) | Unlayered styles | Global escape hatches |

#### Precedence Order (`!important` Declarations)

| Priority | Source |
|---|---|
| 1 (lowest) | Unlayered `!important` |
| 2 | `utilities` layer `!important` |
| 3 | `components` layer `!important` |
| 4 | `base` layer `!important` |
| 5 (highest) | `reset` layer `!important` |

#### Syntax Rules

1. The first declared layer has the lowest priority; the last declared layer has the highest priority.
2. Unlayered normal declarations beat all layered normal declarations.
3. Unlayered `!important` declarations are beaten by all layered `!important` declarations.
4. Within a layer, the normal cascade rules apply (specificity, then order).
5. Nested layers are sorted within their parent layer.
6. The declaration order of layers is fixed at first declaration.
7. The priority of layers is inverted when the `!important` flag is used.

#### Constraints and Limitations

- **Order is immutable** — once declared, a layer's position cannot be changed.
- **Unlayered styles are a double-edged sword** — they win easily but can make debugging difficult.
- **No visual inspection** — browser DevTools show layer order but not always clearly.
- **Nested layer complexity** — deeply nested layers can be hard to reason about.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Layer Precedence Demonstration

**HTML File (`precedence.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Layer Precedence</title>
    <link rel="stylesheet" href="precedence.css">
</head>
<body>
    <div class="box">Layer Precedence</div>
</body>
</html>
```

**CSS File (`precedence.css`):**

```css
/* Declare layer order: base, components, utilities */
@layer base, components, utilities;

@layer base {
    .box {
        background-color: #3498db;
        padding: 40px;
        color: white;
        font-weight: bold;
        text-align: center;
    }
}

@layer components {
    .box {
        background-color: #e74c3c; /* Overrides base */
        border-radius: 16px;
    }
}

@layer utilities {
    .box {
        background-color: #27ae60; /* Overrides components */
    }
}

/* Unlayered: overrides all layers for normal declarations */
.box {
    background-color: #9b59b6;
    border: 4px solid #2c3e50;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `precedence.html` and CSS as `precedence.css`.
2. Open in a browser.
3. Observe that the box has a purple background (unlayered rule) with a dark border.

**Expected Output:** A purple box with a dark border, despite the `utilities` layer also setting `background-color`. The unlayered rule wins because unlayered normal declarations have the highest priority.

**Why This Works:** The `@layer base, components, utilities;` statement establishes the order. The `utilities` layer would normally win over `components` and `base`. But the unlayered `.box` rule beats all layered normal declarations, so the purple background wins.

---

### Real-World Cases

- **Design system architecture:** `@layer reset, tokens, base, layout, components, utilities, overrides;`
- **Framework isolation:** Importing Bootstrap into a `vendor` layer so custom styles always win.
- **Legacy code:** Wrapping old CSS in an early layer and placing new CSS in later layers.
- **Debugging:** Using layer order to determine why a style is not applying.

---

## 3. Third-Party CSS Isolation: Wrapping Framework Styles

### Definitions

**Core Definition:** Third-party CSS isolation uses `@import ... layer(name)` to import external stylesheets (frameworks, libraries, widgets) into a low-priority layer, ensuring that your own styles always override them without specificity hacks.

**Technical Definition:** The `@import` rule can accept a `layer` keyword or `layer()` function, assigning the imported stylesheet's contents into the specified layer. This is the primary mechanism for integrating third-party CSS while maintaining control over the cascade. Imported layers can be nested (e.g., `layer(vendor.bootstrap)`), and if the imported stylesheet itself contains layers, those layers become nested within the import layer. This prevents naming conflicts and ensures that your custom styles, placed in a later layer or left unlayered, always win.

**Beginner-Friendly Explanation:** If you use Bootstrap or another framework, you usually have to fight its specificity. With cascade layers, you just import it into a `vendor` layer at the top of your layer order. Now every style you write — in a later layer or unlayered — automatically beats Bootstrap without any `!important` or deep selectors. It is the cleanest way to integrate third-party CSS.

---

### Purposes

- To isolate third-party CSS in a low-priority layer.
- To ensure custom styles override framework styles without specificity hacks.
- To prevent naming conflicts between external and internal layers.
- To enable clean integration of multiple libraries.
- To maintain a predictable cascade across all stylesheets.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Import into a named layer */
@import url('bootstrap.css') layer(vendor);

/* Import into a nested layer */
@import url('bootstrap.css') layer(vendor.bootstrap);

/* Import multiple files into the same layer */
@import url('headings.css') layer(default);
@import url('links.css') layer(default);

/* Import into an anonymous layer */
@import url('some-library.css') layer();
```

#### Component Breakdown

| Syntax | Description |
|---|---|
| `layer(name)` | Imports into a named layer. |
| `layer(parent.child)` | Imports into a nested layer. |
| `layer()` | Imports into an anonymous layer. |
| `layer` keyword | Same as `layer()` for anonymous. |

#### Syntax Rules

1. The `@import` rule with `layer()` must appear before all other rules except `@charset` and `@layer` statements.
2. Imported layers are created if they do not exist.
3. If the imported file contains layers, they become nested within the import layer.
4. Multiple files can be imported into the same layer.
5. Use `layer(vendor)` or `layer(thirdparty)` as a naming convention.
6. The imported layer should be placed early in your layer order (e.g., `@layer reset, vendor, base, components, utilities;`).

#### Constraints and Limitations

- **Order dependency** — the import must come before other rules.
- **No `<link>` equivalent** — there is no HTML `<link>` attribute for importing into a layer; use `@import` in CSS.
- **Performance** — `@import` can cause additional HTTP requests; use build tools to inline when possible.
- **Framework internals** — if a framework uses its own layers, they nest inside your import layer.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Isolating a Framework in a Vendor Layer

**HTML File (`vendor.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vendor Layer Isolation</title>
    <link rel="stylesheet" href="vendor.css">
</head>
<body>
    <button class="btn">Framework Button</button>
    <button class="btn btn-primary">Custom Primary Button</button>
</body>
</html>
```

**CSS File (`vendor.css`):**

```css
/* 1. Establish layer order with vendor early */
@layer vendor, base, components, utilities;

/* 2. Import framework CSS into the vendor layer */
@import url('https://cdn.example.com/framework.css') layer(vendor);

/* 3. Your base styles */
@layer base {
    body {
        font-family: system-ui, sans-serif;
        padding: 40px;
    }
}

/* 4. Your component styles override the framework */
@layer components {
    .btn {
        background-color: #006064;
        color: white;
        border: none;
        padding: 12px 24px;
        border-radius: 8px;
        cursor: pointer;
        font-weight: bold;
    }
    .btn-primary {
        background-color: #3498db;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `vendor.html` and CSS as `vendor.css`.
2. Open in a browser (the framework CSS will be fetched from the CDN).
3. Observe that the buttons use your custom styles, not the framework's, despite the framework being loaded.

**Expected Output:** Both buttons use the teal and blue backgrounds from your `components` layer, overriding the framework's styles in the `vendor` layer.

**Why This Works:** The `@import url(...) layer(vendor);` places the framework CSS in the `vendor` layer. Because `vendor` is declared first in the layer order, it has the lowest priority. Your `components` layer comes later, so it wins regardless of specificity. This isolates the framework cleanly.

---

### Real-World Cases

- **Bootstrap integration:** `@import url('bootstrap.css') layer(vendor);`
- **Tailwind integration:** Importing Tailwind's preflight into a `reset` layer.
- **Widget libraries:** Importing third-party widget styles into a `widgets` layer.
- **Multi-framework projects:** Using separate nested layers for each framework.

---

## 4. Architectural Organization: Designing Robust Style Hierarchies

### Definitions

**Core Definition:** Architectural organization with cascade layers is the practice of structuring a stylesheet into a fixed hierarchy of layers — typically `reset`, `base`, `tokens`, `components`, `utilities`, and `overrides` — to permanently eliminate specificity conflicts.

**Technical Definition:** The recommended layer order is `@layer reset, base, tokens, components, utilities, overrides;`. Each layer serves a distinct purpose: `reset` normalizes browser defaults, `base` sets element-level defaults, `tokens` defines design tokens, `components` contains component-scoped styles, `utilities` holds utility classes, and `overrides` is for page-specific or one-off overrides. Because layer order determines precedence, styles in `overrides` always beat styles in `components`, regardless of selector specificity. This architecture maps directly to the ITCSS (Inverted Triangle CSS) methodology.

**Beginner-Friendly Explanation:** Think of your stylesheet as a pyramid. At the bottom is the reset (the broadest, least specific styles). Then base, then tokens, then components, then utilities, and at the top, overrides (the narrowest, most specific styles). Because layers are ordered, a style in the overrides layer always wins over a style in the components layer — you never have to write `!important` or deeply nested selectors again.

---

### Purposes

- To establish a predictable, maintainable cascade order.
- To eliminate specificity wars between teams or components.
- To separate concerns (reset, base, tokens, components, utilities).
- To enable large teams to work on the same codebase without conflicts.
- To map directly to established CSS architectures like ITCSS.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Declare the full layer order at the top of the entry file */
@layer reset, base, tokens, components, utilities, overrides;

@layer reset {
    *, *::before, *::after {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }
}

@layer base {
    body {
        font-family: system-ui, sans-serif;
        line-height: 1.6;
    }
}

@layer tokens {
    :root {
        --color-primary: #3498db;
        --space-4: 1rem;
        --radius-md: 8px;
    }
}

@layer components {
    .card {
        background-color: white;
        border-radius: var(--radius-md);
        padding: var(--space-4);
    }
}

@layer utilities {
    .sr-only {
        position: absolute;
        width: 1px;
        height: 1px;
        overflow: hidden;
    }
}

@layer overrides {
    .hero-banner .card {
        padding: 3rem;
    }
}
```

#### Component Breakdown

| Layer | Purpose | Example |
|---|---|---|
| `reset` | Normalize browser defaults. | `*, *::before { box-sizing: border-box; }` |
| `base` | Element-level defaults. | `body { font-family: var(--font-sans); }` |
| `tokens` | Design tokens / custom properties. | `:root { --color-primary: oklch(...); }` |
| `components` | Component-scoped styles. | `.card { border-radius: var(--radius-md); }` |
| `utilities` | Utility classes. | `.sr-only { position: absolute; }` |
| `overrides` | Page-specific overrides. | `.hero-banner .card { padding: 3rem; }` |

#### Syntax Rules

1. Declare all layers in a single `@layer` statement at the top of the entry CSS file.
2. Later layers beat earlier layers regardless of selector specificity.
3. Assign third-party CSS to early layers (`reset` or `base`) for clean overrides.
4. Never use `!important` to fight specificity — restructure layers instead.
5. Unlayered CSS beats all layers — keep it minimal.
6. Tokens can be placed in their own layer or in `:root` unlayered.
7. Tailwind v4 uses layers internally; place custom layers around it accordingly.

#### Constraints and Limitations

- **Layer order is fixed** — you cannot change the order after the first declaration.
- **Unlayered styles win** — keep unlayered CSS to an absolute minimum.
- **Team adoption** — the architecture only works if the whole team follows it.
- **Tooling support** — some build tools and frameworks may not yet fully support layers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Complete Layer Architecture

**HTML File (`architecture.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Layer Architecture</title>
    <link rel="stylesheet" href="architecture.css">
</head>
<body>
    <div class="card">
        <h2>Architecture Card</h2>
        <p>This card uses a full layer architecture.</p>
        <button class="btn btn-primary">Action</button>
    </div>
</body>
</html>
```

**CSS File (`architecture.css`):**

```css
/* 1. Layer order */
@layer reset, base, tokens, components, utilities, overrides;

/* 2. Reset */
@layer reset {
    *, *::before, *::after {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }
}

/* 3. Base */
@layer base {
    body {
        font-family: system-ui, sans-serif;
        background-color: #f5f5f5;
        padding: 40px;
    }
}

/* 4. Tokens */
@layer tokens {
    :root {
        --color-primary: #3498db;
        --color-surface: #ffffff;
        --color-text: #111827;
        --space-4: 1rem;
        --space-6: 1.5rem;
        --radius-lg: 12px;
    }
}

/* 5. Components */
@layer components {
    .card {
        background-color: var(--color-surface);
        border-radius: var(--radius-lg);
        padding: var(--space-6);
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
        max-width: 400px;
    }
    .card h2 {
        color: var(--color-text);
        margin-bottom: var(--space-4);
    }
    .btn {
        background-color: var(--color-primary);
        color: white;
        border: none;
        padding: var(--space-4) var(--space-6);
        border-radius: 8px;
        cursor: pointer;
        font-weight: bold;
    }
}

/* 6. Utilities */
@layer utilities {
    .text-center {
        text-align: center;
    }
}

/* 7. Overrides */
@layer overrides {
    .card {
        border-left: 4px solid #e74c3c;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `architecture.html` and CSS as `architecture.css`.
2. Open in a browser.
3. Observe the card with a red left border, teal button, and clean spacing — all driven by the layer architecture.

**Expected Output:** A card with a red left border (from `overrides`), a teal button (from `components`), and consistent spacing (from `tokens`). No `!important` is used anywhere.

**Why This Works:** The layer order `reset, base, tokens, components, utilities, overrides` establishes a clear hierarchy. The `overrides` layer comes last, so its `.card` rule wins over the `components` layer's `.card` rule — regardless of specificity. The tokens layer provides custom properties that all other layers consume.

---

### Real-World Cases

- **Design systems:** A shared layer architecture across multiple applications.
- **Large teams:** Multiple teams working on the same codebase without conflicts.
- **Legacy migration:** Wrapping old CSS in early layers and placing new CSS in later layers.
- **Tailwind CSS v4:** Tailwind v4 uses layers internally; custom layers are placed around them.

---

## 5. Specificity Interactions: How Layering Alters the Classic Cascade and Inverted `!important` Priority

### Definitions

**Core Definition:** Cascade layers fundamentally alter the classic cascade by making layer order the primary sorting criterion for normal declarations, overriding selector specificity. For `!important` declarations, the priority order among layers is inverted: important declarations in earlier layers take precedence over important declarations in later layers.

**Technical Definition:** Within each origin (author, user, user agent), the cascade sorts declarations by layer order, then by specificity and order of appearance. For normal declarations, later layers beat earlier layers, and unlayered styles beat all layered styles. For `!important` declarations, the order is reversed: important declarations in earlier layers beat important declarations in later layers, and important declarations in layers beat important declarations outside of layers. This inversion ensures that important declarations in the `reset` layer (the earliest layer) have the highest priority, which is counterintuitive but designed to protect user and accessibility styles.

**Beginner-Friendly Explanation:** Normally, if you want a style to win, you write a more specific selector. With layers, you do not need to — the layer order decides. But `!important` flips everything upside down. In the normal case, `utilities` (the last layer) wins. But if both `reset` and `utilities` have `!important` declarations, `reset` wins because its position is earlier in the layer order. This is designed to give the "lowest" layer the highest important priority, which can be confusing but is intentional.

---

### Purposes

- To understand why layer order overrides specificity for normal declarations.
- To understand the inverted priority of `!important` across layers.
- To debug cascade issues involving `!important` and layers.
- To design layer architectures that avoid `!important` entirely.
- To leverage the inversion for accessibility and user-preference styles.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Normal declarations: later layers win */
@layer reset, base, components, utilities;

@layer reset {
    .card { color: red; } /* Low priority */
}

@layer utilities {
    .card { color: blue; } /* High priority — wins */
}

/* !important declarations: earlier layers win */
@layer reset {
    .card { color: red !important; } /* High priority — wins */
}

@layer utilities {
    .card { color: blue !important; } /* Low priority */
}
```

#### Component Breakdown

| Declaration Type | Priority Order |
|---|---|
| Normal layered | Last layer wins; unlayered beats all. |
| Normal unlayered | Highest priority among normal. |
| `!important` layered | First layer wins; last layer loses. |
| `!important` unlayered | Lowest priority among `!important`. |

#### The Full Cascade Order (Author Origin)

```
1. Transitions (highest)
2. Important user agent
3. Important user
4. Important author (layered — first layer wins)
5. Important author (unlayered — lowest important priority)
6. Normal author (unlayered — highest normal priority)
7. Normal author (layered — last layer wins)
8. Normal user
9. Normal user agent
```

#### Syntax Rules

1. Layer order overrides specificity for normal declarations.
2. Unlayered normal declarations beat all layered normal declarations.
3. For `!important` declarations, the layer priority is inverted.
4. Important declarations in layers beat important declarations outside layers.
5. The first declared layer gets the highest `!important` priority.
6. The `!important` flag is not part of specificity but interacts with it.
7. All important declarations beat all normal declarations.

#### Constraints and Limitations

- **Inversion is counterintuitive** — the earliest layer wins for `!important`, which can be confusing.
- **Debugging difficulty** — DevTools may not clearly show layer order.
- **No `!important` in `@keyframes`** — the `!important` flag is invalid inside `@keyframes`.
- **Transitions beat everything** — CSS transitions have the highest precedence, above `!important`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inverted `!important` Priority

**HTML File (`important.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inverted !important</title>
    <link rel="stylesheet" href="important.css">
</head>
<body>
    <div class="box">Inverted !important Priority</div>
</body>
</html>
```

**CSS File (`important.css`):**

```css
@layer reset, base, components, utilities;

@layer reset {
    .box {
        background-color: #e74c3c !important; /* Wins — earliest layer */
        padding: 40px;
        color: white;
        font-weight: bold;
        text-align: center;
    }
}

@layer base {
    .box {
        background-color: #3498db;
    }
}

@layer components {
    .box {
        background-color: #27ae60;
    }
}

@layer utilities {
    .box {
        background-color: #9b59b6 !important; /* Loses — latest layer */
    }
}

/* Unlayered !important: lowest priority among !important */
.box {
    background-color: #f1c40f !important;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `important.html` and CSS as `important.css`.
2. Open in a browser.
3. Observe that the box is red (from the `reset` layer's `!important` declaration), not purple or yellow.

**Expected Output:** A red box with white text, despite the `utilities` layer and the unlayered rule both using `!important`. The `reset` layer wins because `!important` inverts the layer priority order.

**Why This Works:** For `!important` declarations, the layer priority is inverted. The `reset` layer is the first declared, so it has the highest `!important` priority. The `utilities` layer is the last declared, so it has the lowest `!important` priority. The unlayered `!important` declaration is even lower. This is the inverted priority behaviour.

---

### Real-World Cases

- **Accessibility themes:** Placing high-contrast `!important` styles in an early `reset` layer so they cannot be overridden.
- **User preferences:** Using `!important` in a `user` layer to override author styles.
- **Legacy code:** Understanding why old `!important` declarations behave unexpectedly in layered systems.
- **Debugging:** Tracing why a style with `!important` is not applying — it may be in the wrong layer.

---

## References

- W3C — CSS Cascading and Inheritance Level 5 (Cascade Layers) - https://www.w3.org/TR/2021/WD-css-cascade-5-20210827/
- MDN Web Docs — `@layer` CSS at-rule - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@layer
- MDN Web Docs — `!important` - https://developer.mozilla.org/en-US/docs/Web/CSS/important
- MDN Web Docs — Cascade layers - https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_layers
- MDN Web Docs — `@import` - https://developer.mozilla.org/en-US/docs/Web/CSS/@import
- CSS-Tricks — Cascade Layers - https://css-tricks.com/css-cascade-layers/
- Smashing Magazine — A Complete Guide to CSS Cascade Layers - https://www.smashingmagazine.com/2022/01/introduction-css-cascade-layers/
- web.dev — Cascade layers - https://web.dev/learn/css/cascade-layers
- W3C — CSS Cascading and Inheritance Level 5 (Editor's Draft) - https://drafts.csswg.org/css-cascade-5/
- Can I Use — CSS Cascade Layers - https://caniuse.com/css-cascade-layers