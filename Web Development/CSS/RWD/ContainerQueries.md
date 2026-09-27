# CSS Container Queries & Component-Isolated Responsiveness — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Container Queries are a conditional styling mechanism that allows styles to be applied to elements based on the size, style, or state of a parent containment context rather than the viewport. An element is registered as a "container" using the `container-type` property, and descendant elements can then be conditionally styled using the `@container` at-rule when the container meets specified conditions.

**Technical Definition:** Container queries are defined in the CSS Containment Module Level 3. The mechanism requires two parts: (1) establishing a containment context on an ancestor element using `container-type` (`size` or `inline-size`), and (2) writing a `@container` conditional group rule that queries that context using a `<container-condition>`. The condition may be a size query (evaluating the container's dimensions) or a style query (evaluating the container's custom property values). Container queries enable components to respond to their local environment rather than the global viewport, and they introduce container-relative length units (`cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax`) that resolve against the query container's dimensions rather than the viewport.

**Beginner-Friendly Explanation:** Normally, responsive design uses media queries, which ask "how wide is the screen?" Container queries ask a different question: "how wide is my parent element?" This means a card component can look one way when placed in a narrow sidebar and another way when placed in a wide main content area — without any JavaScript or knowledge of the viewport size. You register a parent as a "container," then write rules that say "when this container is at least 400px wide, style the children this way." The component becomes self-contained and reusable anywhere.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Container-relative** | Queries evaluate the dimensions or styles of a parent element, not the viewport. |
| **Component isolation** | Components can be dropped into any context and adapt to their local environment. |
| **Two-part setup** | Requires `container-type` on the parent and `@container` rules for the children. |
| **Size and style queries** | Supports both dimensional queries and custom property value queries. |
| **Container-relative units** | New length units (`cqi`, `cqb`, `cqmin`, `cqmax`) resolve against the container. |
| **Logical property support** | Supports logical dimension queries (`inline-size`, `block-size`). |
| **Cascade-aware** | Container queries participate in the cascade like any other conditional rule. |

---

### Prerequisites

Before studying CSS Container Queries, you should understand:

- **CSS Media Queries** — the conditional styling model that container queries extend.
- **CSS Box Model and Layout** — how elements are sized and positioned.
- **CSS Custom Properties** — for style queries.
- **CSS Logical Properties** — `inline-size` and `block-size` concepts.
- **CSS Containment** — the performance isolation model behind container queries.

---

### Related Programming Areas

- **Component-Driven Development** — container queries enable truly modular components.
- **CSS Media Queries** — the viewport-based counterpart to container queries.
- **Design Systems** — components can be shared across contexts without media query duplication.
- **Web Components** — container queries integrate naturally with shadow DOM encapsulation.
- **Responsive Design** — the shift from page-level to component-level responsiveness.

---

### Core Concepts / Features

1. Establishing Containment Contexts: `container-type` and `container-name`
2. The `@container` Rule Syntax: Dimensional and Structural Queries
3. Style Queries: Evaluating Custom Properties of a Parent Container
4. Architectural Shift: From Viewport-Dependent Pages to Component-Level Responsiveness
5. Container-Relative Units: `cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax`

---

## 1. Establishing Containment Contexts: Configuring `container-type` and Registration via `container-name`

### Definitions

**Core Definition:** A containment context is an element that has been registered as a container using the `container-type` property, making it queryable by descendant elements via the `@container` at-rule. The `container-name` property assigns one or more names to the container for targeted querying.

**Technical Definition:** The `container-type` property specifies the type of container context used in a container query. It accepts three values: `normal` (the default, which applies style containment but not size containment), `inline-size` (applies inline-size containment, enabling queries on the inline axis), and `size` (applies both inline-size and block-size containment, enabling queries on both axes). The `container-name` property specifies a list of query container names, which are used to filter which containers are targeted by `@container` rules. The `container` shorthand combines both properties: `container: <name> / <type>`. When `container-type` is `inline-size` or `size`, the element's size becomes independent of its contents (size containment), meaning the container must be explicitly sized by its parent layout.

**Beginner-Friendly Explanation:** To use container queries, you first need to tell the browser which elements are containers. You do this with `container-type: inline-size` (for width-based queries) or `container-type: size` (for width and height queries). You can also give the container a name with `container-name` so you can target it specifically. The `container` shorthand lets you write both in one line: `container: card / inline-size`. Remember: a container cannot be sized by its own contents — it must get its size from its parent layout (like flexbox or grid).

---

### Purposes

- To register an element as a queryable containment context for descendant elements.
- To enable size containment, preventing descendants from affecting the container's dimensions.
- To assign semantic names to containers for targeted `@container` rules.
- To control which axis (inline, block, or both) can be queried.
- To provide the foundational setup required for all container query usage.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    container-type: normal | inline-size | size;
    container-name: <custom-ident>+ | none;
    /* Shorthand */
    container: <name> / <type>;
}

@container [<container-name>] (<container-condition>) {
    /* styles */
}
```

#### Component Breakdown

| Property | Values | Description |
|---|---|---|
| `container-type` | `normal` | Default. Style containment only; no size queries. |
| `container-type` | `inline-size` | Inline-axis containment; queries on inline dimension. |
| `container-type` | `size` | Both-axis containment; queries on inline and block dimensions. |
| `container-name` | `<custom-ident>+` | One or more names for targeted querying. |
| `container` | `<name> / <type>` | Shorthand for both properties. |

#### Syntax Rules

1. `container-type: inline-size` applies inline-size containment; the element's inline size cannot be affected by its contents.
2. `container-type: size` applies both inline-size and block-size containment.
3. `container-type: normal` applies style containment only (no size queries).
4. `container-name` is optional; unnamed containers match any `@container` rule that does not target a specific name.
5. The `container` shorthand sets both `container-name` and `container-type`.
6. A container must be sized by its parent layout (flex, grid, or explicit dimensions); it cannot be sized by its contents when size containment is active.
7. Container names are case-sensitive and cannot be CSS-wide keywords.

#### Constraints and Limitations

- **Size containment side effect** — with `container-type: inline-size` or `size`, the element's size no longer depends on its contents, so it may collapse if not explicitly sized.
- **No self-querying** — a container cannot query its own size to change its own styles; only descendants can be styled.
- **Nested containers** — the nearest ancestor with `container-type` is used unless a name is specified.
- **Performance** — size containment enables rendering optimisations but requires explicit sizing.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Registering a Container with `container-type` and `container-name`

**HTML File (`container-setup.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Setup</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="container-setup.css">
</head>
<body>
    <!-- Wrapper that acts as the container -->
    <div class="card-wrapper">
        <div class="card">
            <h2 class="card-title">Responsive Card</h2>
            <p class="card-text">This card adapts to its container's width.</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`container-setup.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card-wrapper {
    /* Register this element as a container */
    container-type: inline-size;
    container-name: card;
    /* Alternatively: container: card / inline-size; */
    /* The wrapper must have an explicit width from its parent layout */
    width: min(100%, 600px);
    margin: 0 auto;
}

.card {
    background-color: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card-title {
    margin: 0 0 10px;
}

.card-text {
    margin: 0;
    color: #555;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `container-setup.html`.
3. Save the CSS code as `container-setup.css` in the same folder.
4. Open `container-setup.html` in a modern browser.
5. The card renders normally. Inspect the `.card-wrapper` in DevTools — the Layout panel shows it as a container.

**Expected Output:** A white card centred on the page. The `.card-wrapper` is registered as a container with the name `card` and type `inline-size`.

**Why This Works:** The `container-type: inline-size` on `.card-wrapper` establishes a containment context. The `container-name: card` assigns it a queryable name. The wrapper's width is set with `min(100%, 600px)`, giving it an explicit size from its parent layout. This is the required first step before any `@container` rules can be written.

---

### Real-World Cases

- **Card components:** Registering a card wrapper as a container so the card's internal layout can adapt to its placement.
- **Sidebar widgets:** Containers in sidebars that query their narrow width and stack content vertically.
- **Dashboard tiles:** Each tile registered as a container so its content adapts to the tile's size.
- **Navigation components:** A nav wrapper registered as a container so the nav links adapt to the available width.

---

## 2. The `@container` Rule Syntax: Querying Parents Based on Dimensional Layout Thresholds or Structural Properties

### Definitions

**Core Definition:** The `@container` at-rule is a conditional group rule that applies styles to descendants of a containment context when the container meets specified conditions. Conditions can query the container's dimensions (`width`, `height`, `inline-size`, `block-size`, `aspect-ratio`, `orientation`) or its custom property values.

**Technical Definition:** The `@container` at-rule has the formal syntax `@container [<container-name>]? <container-condition># { <stylesheet> }`. The `<container-condition>` is composed of one or more `<size-query>` or `<style-query>` expressions connected by logical operators (`and`, `or`, `not`). Size queries use comparison operators (`>`, `<`, `=`, `>=`, `<=`) or the legacy colon syntax (`min-width`, `max-width`, etc.). The range syntax allows expressions like `(400px <= width <= 800px)`. Logical properties (`inline-size`, `block-size`) are preferred over physical ones (`width`, `height`) for internationalisation. The `@container` rule applies styles only to elements inside a matching container; it never styles the container itself.

**Beginner-Friendly Explanation:** Once you have a container, you can write `@container` rules that apply styles when the container meets a condition. For example: `@container card (min-width: 400px) { .card-title { font-size: 1.5rem; } }`. This says: "If the container named `card` is at least 400px wide, make the card title bigger." You can use range syntax too: `@container (width > 400px)`. The styles inside the rule apply to the descendants of the container, not the container itself.

---

### Purposes

- To apply conditional styles to descendants based on the container's dimensions.
- To use range syntax for more expressive and readable conditions.
- To combine multiple conditions with logical operators.
- To target specific named containers for precise scoping.
- To support logical property queries for internationalisation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Unnamed container query */
@container (<size-query>) { ... }

/* Named container query */
@container <container-name> (<size-query>) { ... }

/* Multiple conditions */
@container (<condition-1>) and (<condition-2>) { ... }
@container (<condition-1>) or (<condition-2>) { ... }
@container not (<condition>) { ... }

/* Range syntax */
@container (width > 400px) { ... }
@container (400px <= width <= 800px) { ... }

/* Logical properties */
@container (inline-size > 30em) { ... }
```

#### Component Breakdown

| Condition Type | Syntax | Example |
|---|---|---|
| Size query (colon) | `(min-width: <length>)` | `(min-width: 400px)` |
| Size query (range) | `(width >= <length>)` | `(width >= 400px)` |
| Bounded range | `(<min> <= width <= <max>)` | `(400px <= width <= 800px)` |
| Logical size | `(inline-size > <length>)` | `(inline-size > 30em)` |
| Orientation | `(orientation: landscape)` | `(orientation: portrait)` |
| Aspect ratio | `(aspect-ratio > 16/9)` | `(aspect-ratio: 16/9)` |

#### Syntax Rules

1. The container name is optional; if omitted, the nearest ancestor with a containment context is used.
2. Size queries use either the colon syntax (`min-width`, `max-width`) or the range syntax (`>`, `<`, `>=`, `<=`).
3. Multiple conditions can be combined with `and`, `or`, and `not`.
4. Logical properties (`inline-size`, `block-size`) are preferred over physical (`width`, `height`).
5. A container cannot style itself; only its descendants are affected.
6. The `@container` rule follows the same cascade rules as any conditional rule.
7. If no matching container is found, the rule does not apply.

#### Constraints and Limitations

- **No self-querying** — a container cannot change its own styles based on its own size.
- **Size containment required** — the container must have `container-type: inline-size` or `size` for dimension queries.
- **Nearest ancestor** — unnamed queries use the nearest containment context, which may not be the intended one in nested containers.
- **Browser support** — container queries are Baseline widely available (Chrome 105+, Firefox 110+, Safari 16+).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dimension-Based Container Queries

**HTML File (`container-query.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Query Dimensions</title>
    <link rel="stylesheet" href="container-query.css">
</head>
<body>
    <div class="wrapper">
        <div class="card">
            <div class="card-layout">
                <div class="card-image">Image</div>
                <div class="card-content">
                    <h2>Card Title</h2>
                    <p>Card description text.</p>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

**CSS File (`container-query.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.wrapper {
    /* Container setup */
    container-type: inline-size;
    container-name: card;
    max-width: 900px;
    margin: 0 auto;
}

.card-layout {
    /* Default: stacked layout */
    display: flex;
    flex-direction: column;
    background-color: white;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card-image {
    background-color: #3498db;
    color: white;
    padding: 40px;
    text-align: center;
    font-weight: bold;
}

.card-content {
    padding: 20px;
}

/* When the container is at least 500px wide, switch to horizontal layout */
@container card (min-width: 500px) {
    .card-layout {
        flex-direction: row;
    }
    .card-image {
        flex: 0 0 200px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `container-query.html` and CSS as `container-query.css`.
2. Open in a browser at full width. The card renders with a horizontal layout (image on the left).
3. Narrow the browser window until the wrapper is below 500px. The card switches to a stacked layout (image on top).

**Expected Output:** A card that switches between horizontal and stacked layouts based on the width of its container, not the viewport.

**Why This Works:** The `@container card (min-width: 500px)` rule evaluates the `.wrapper` container's width. When the container is at least 500px, `flex-direction: row` is applied to `.card-layout`. When the container is narrower, the default `flex-direction: column` applies. Because the query targets the container's inline-size, the card adapts to its local context regardless of the viewport size.

---

### Real-World Cases

- **Product cards in sidebars:** Cards that stack vertically in narrow sidebars and switch to horizontal in wide main areas.
- **Form layouts:** Form rows that stack on narrow containers and align inline on wide containers.
- **Navigation menus:** Menus that switch from a horizontal bar to a vertical stack based on container width.
- **Dashboard tiles:** Tiles that adjust their content layout based on their allocated grid space.

---

## 3. Style Queries: Evaluating Custom Properties of a Parent Container

### Definitions

**Core Definition:** Style queries are a type of container query that evaluates the computed values of a container's CSS custom properties (and, in some implementations, standard CSS properties). They allow styles to be applied to descendants based on the container's style state rather than its dimensions.

**Technical Definition:** Style queries are written using the `style()` function within a `@container` rule: `@container style(<property>: <value>) { ... }`. Currently, the primary supported use case is querying custom properties, as standard CSS property querying has limited and inconsistent browser support. The `style()` function supports the colon syntax for specific values and, from Chrome 142, the range syntax for numeric comparisons: `@container style(--lightness < 50%)`. Style queries enable conditional styling based on design tokens, theme variables, or component configuration flags passed down through custom properties. A container with `container-type: normal` (the default) can be queried by style queries without size containment side effects.

**Beginner-Friendly Explanation:** Style queries let you say: "If the container has a custom property set to a certain value, apply these styles." For example, if a parent has `--theme: dark`, you can write `@container style(--theme: dark) { ... }` to apply dark-mode styles. This is useful for theming components or creating variants without adding extra classes. Style queries are newer and currently work best with custom properties.

---

### Purposes

- To apply styles based on the container's custom property values.
- To enable component theming and variant configuration via custom properties.
- To support conditional styling without additional classes or media queries.
- To use range syntax for numeric custom property comparisons.
- To complement size queries with style-based conditional logic.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Specific value query */
@container style(<property>: <value>) { ... }

/* Range syntax (Chrome 142+) */
@container style(<property> <comparison> <value>) { ... }

/* Named container with style query */
@container <name> style(<property>: <value>) { ... }
```

#### Component Breakdown

| Syntax | Description | Example |
|---|---|---|
| `style(--prop: value)` | Queries the exact value of a custom property. | `style(--theme: dark)` |
| `style(--prop >= value)` | Range comparison on a numeric custom property. | `style(--lightness >= 50%)` |
| `style(--prop < value)` | Less-than comparison. | `style(--spacing < 1rem)` |

#### Syntax Rules

1. Style queries use the `style()` function inside `@container`.
2. The primary supported feature is custom properties (`--*`).
3. Range syntax on style queries requires Chrome 142+.
4. Standard CSS property querying has limited browser support.
5. Style queries work with `container-type: normal` (the default) without size containment.
6. The `style()` function can be combined with size queries using logical operators.

#### Constraints and Limitations

- **Limited property support** — currently, only custom properties are reliably queryable.
- **Browser support** — style queries are newer; range syntax requires Chrome 142+.
- **No standard property querying** — querying standard properties like `background-color` is not consistently supported.
- **Inheritance** — custom properties inherit through the DOM, so a style query on a parent affects all descendants that inherit the value.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Theming with Style Queries

**HTML File (`style-query.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Style Queries</title>
    <link rel="stylesheet" href="style-query.css">
</head>
<body>
    <!-- Light theme container -->
    <div class="theme-container light">
        <div class="card">
            <h2>Light Theme Card</h2>
            <p>This card uses the light theme styles.</p>
        </div>
    </div>

    <!-- Dark theme container -->
    <div class="theme-container dark">
        <div class="card">
            <h2>Dark Theme Card</h2>
            <p>This card uses the dark theme styles.</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`style-query.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}

.theme-container {
    /* Style queries work with the default container-type: normal */
    flex: 1;
    min-width: 250px;
}

.light {
    --theme: light;
}

.dark {
    --theme: dark;
}

.card {
    padding: 20px;
    border-radius: 10px;
    background-color: white;
    color: #1a1a1a;
}

/* When the container has --theme: dark */
@container style(--theme: dark) {
    .card {
        background-color: #1a1a1a;
        color: #e8e8e8;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `style-query.html` and CSS as `style-query.css`.
2. Open in a modern browser that supports style queries (Chrome 111+ for basic style queries).
3. Observe that the light container's card is white with dark text, and the dark container's card is dark with light text.

**Expected Output:** Two cards side by side. The left card (light theme) has a white background and dark text. The right card (dark theme) has a dark background and light text. Both cards use identical HTML — only the custom property value differs.

**Why This Works:** The `.light` and `.dark` classes set the `--theme` custom property to `light` and `dark` respectively. The `@container style(--theme: dark)` rule applies dark styles to any `.card` inside a container where `--theme` is `dark`. Because style queries work with `container-type: normal` (the default), no explicit container setup is required for style queries. This demonstrates how style queries enable conditional theming without extra classes or media queries.

---

### Real-World Cases

- **Theming systems:** Switching component styles based on `--theme` custom properties.
- **Component variants:** Applying different styles based on `--variant: compact` or `--variant: spacious`.
- **Density modes:** Adjusting spacing based on `--density: compact` or `--density: comfortable`.
- **Brand customisation:** Applying brand-specific styles based on `--brand: acme`.

---

## 4. Architectural Shift: Transitioning from Viewport-Dependent Pages to Modular, Independent Component-Level Responsiveness

### Definitions

**Core Definition:** The architectural shift enabled by container queries is the transition from page-level responsive design (where components respond to the viewport) to component-level responsive design (where components respond to their own container). This inverts the traditional top-down responsive workflow into a bottom-up, component-driven model.

**Technical Definition:** Traditional responsive design uses media queries that query the viewport, meaning all components on a page respond to the same global breakpoints. Container queries decouple component behaviour from the viewport, allowing each component to define its own responsive logic based on the size of its containing element. This enables true component reusability: a card component can be dropped into a sidebar, a main content area, or a dashboard grid, and it will adapt to each context without modification. The architectural implications include: simplified component design (bottom-up rather than top-down), reduced reliance on JavaScript for layout management, and elimination of complex media query chains. Netflix's engineering team reported that container queries revolutionised their component design approach, enabling components to be responsive without knowledge of the viewport.

**Beginner-Friendly Explanation:** Before container queries, responsive design was page-centric. You would write media queries for the whole page, and every component would respond to the same breakpoints. With container queries, each component becomes self-contained. A card component can define "if my container is narrow, stack the content; if wide, show it side by side." You can use that card anywhere — in a sidebar, a modal, a grid — and it will adapt correctly. This is a fundamental shift in how you think about responsive design: you design components that are inherently responsive, not pages that happen to contain responsive elements.

---

### Purposes

- To enable true component-level responsiveness independent of the viewport.
- To simplify component design by making components self-contained.
- To reduce reliance on JavaScript for layout adaptation.
- To improve component reusability across different contexts.
- To align responsive design with component-driven development workflows.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Component defines its own responsive logic */
.component-container {
    container-type: inline-size;
}

@container (min-width: 400px) {
    .component-child {
        /* Component-specific responsive styles */
    }
}
```

#### Component Breakdown

| Aspect | Viewport-Based (Media Queries) | Container-Based (Container Queries) |
|---|---|---|
| Query target | Viewport dimensions | Container dimensions |
| Responsive scope | Page-level | Component-level |
| Reusability | Limited (depends on viewport) | High (adapts to any container) |
| JavaScript dependency | Often needed for complex layouts | Minimal |
| Breakpoint logic | Global breakpoints | Component-defined thresholds |

#### Syntax Rules

1. Each component defines its own `container-type` on a wrapper element.
2. The component's internal responsive logic is expressed with `@container` rules.
3. The component does not need to know the viewport size.
4. The same component can be placed in different containers and adapt accordingly.
5. Container queries can be nested: a container can contain another container.

#### Constraints and Limitations

- **Container setup required** — each component needs a wrapper with `container-type`.
- **Size containment side effects** — the wrapper must be sized by its parent layout.
- **Browser support** — container queries are Baseline widely available but not supported in Internet Explorer.
- **Nesting complexity** — deeply nested containers require careful naming to target the correct context.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reusable Card Component with Container Queries

**HTML File (`component.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Component-Level Responsiveness</title>
    <link rel="stylesheet" href="component.css">
</head>
<body>
    <!-- Narrow sidebar context -->
    <aside class="sidebar">
        <div class="card-container">
            <div class="card">
                <div class="card-image">Image</div>
                <div class="card-body">
                    <h3>Sidebar Card</h3>
                    <p>This card adapts to the narrow sidebar.</p>
                </div>
            </div>
        </div>
    </aside>

    <!-- Wide main content context -->
    <main class="main-content">
        <div class="card-container">
            <div class="card">
                <div class="card-image">Image</div>
                <div class="card-body">
                    <h3>Main Content Card</h3>
                    <p>This card adapts to the wide main area.</p>
                </div>
            </div>
        </div>
    </main>
</body>
</html>
```

**CSS File (`component.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
    display: flex;
    gap: 20px;
}

.sidebar {
    flex: 0 0 250px;
}

.main-content {
    flex: 1;
}

.card-container {
    /* Each card container is a queryable container */
    container-type: inline-size;
    container-name: card;
}

.card {
    display: flex;
    flex-direction: column;
    background-color: white;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card-image {
    background-color: #3498db;
    color: white;
    padding: 30px;
    text-align: center;
    font-weight: bold;
}

.card-body {
    padding: 15px;
}

/* When the card container is at least 400px, switch to horizontal */
@container card (min-width: 400px) {
    .card {
        flex-direction: row;
    }
    .card-image {
        flex: 0 0 150px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `component.html` and CSS as `component.css`.
2. Open in a browser.
3. Observe that the sidebar card (narrow container) uses a stacked layout, while the main content card (wide container) uses a horizontal layout.
4. The same HTML and CSS is used for both cards — only the container width differs.

**Expected Output:** Two cards with identical HTML. The sidebar card is stacked (image on top, text below) because its container is narrow. The main content card is horizontal (image on the left, text on the right) because its container is wide.

**Why This Works:** The `.card-container` wrapper has `container-type: inline-size` and `container-name: card`. The `@container card (min-width: 400px)` rule applies `flex-direction: row` when the container is wide enough. Because the sidebar container is narrower than 400px, the card stays stacked. Because the main content container is wider, the card switches to horizontal. The same component adapts to both contexts without any JavaScript or viewport knowledge.

---

### Real-World Cases

- **Design systems:** Components that ship with their own responsive logic and work in any context.
- **Netflix-scale applications:** Reducing JavaScript layout management by moving responsive logic into CSS.
- **CMS-driven content:** Components that adapt to the unpredictable widths of CMS content areas.
- **Web Components:** Self-contained components with encapsulated responsive behaviour.

---

## 5. Container-Relative Units: Typography and Spacing Orchestration Using `cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax`

### Definitions

**Core Definition:** Container query length units are CSS length units that resolve against the dimensions of the nearest query container. They enable typography and spacing to scale fluidly with the container's size rather than the viewport's size.

**Technical Definition:** Container query length units are defined in the CSS Containment Module Level 3. The units are: `cqw` (1% of the query container's width), `cqh` (1% of the query container's height), `cqi` (1% of the query container's inline size), `cqb` (1% of the query container's block size), `cqmin` (the smaller of `cqi` and `cqb`), and `cqmax` (the larger of `cqi` and `cqb`). These units resolve against the dimensions of the nearest ancestor with a containment context. They are analogous to viewport units (`vw`, `vh`, etc.) but are container-relative rather than viewport-relative. The logical units (`cqi`, `cqb`) are preferred over the physical units (`cqw`, `cqh`) for internationalisation, as they respect the writing mode.

**Beginner-Friendly Explanation:** Container query units work like viewport units, but they measure the container instead of the screen. `1cqi` is 1% of the container's inline size (usually its width). If your container is 500px wide, `50cqi` equals 250px. This lets you size text and spacing proportionally to the container. For example, `font-size: 5cqi` makes the text 5% of the container width, so it scales smoothly as the container resizes. The logical units (`cqi`, `cqb`) are better because they adapt to writing modes.

---

### Purposes

- To size typography and spacing relative to the container rather than the viewport.
- To create fluid scaling within components that adapts to their local context.
- To provide logical units (`cqi`, `cqb`) that respect writing modes.
- To complement container queries with proportional sizing inside components.
- To reduce the need for manual calculations when sizing elements within containers.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    font-size: <number>cqi;
    padding: <number>cqb;
    width: <number>cqw;
    height: <number>cqh;
    gap: <number>cqmin;
}
```

#### Component Breakdown

| Unit | Resolves Against | Description |
|---|---|---|
| `cqw` | Container width | 1% of the query container's width. |
| `cqh` | Container height | 1% of the query container's height. |
| `cqi` | Container inline size | 1% of the query container's inline size (logical). |
| `cqb` | Container block size | 1% of the query container's block size (logical). |
| `cqmin` | Smaller of `cqi` and `cqb` | The smaller container dimension. |
| `cqmax` | Larger of `cqi` and `cqb` | The larger container dimension. |

#### Syntax Rules

1. Container query units resolve against the nearest ancestor with a containment context.
2. If no container is found, the units resolve against the small viewport size.
3. `cqi` and `cqb` are logical units that respect the writing mode.
4. `cqw` and `cqh` are physical units (width and height).
5. `cqmin` and `cqmax` are the smaller and larger of `cqi` and `cqb`.
6. These units can be used anywhere a `<length>` value is accepted.
7. They work in conjunction with `@container` rules but do not require them.

#### Constraints and Limitations

- **Container required** — a containment context must exist for the units to resolve correctly.
- **No fallback in older browsers** — browsers that do not support container units will ignore declarations using them.
- **Performance** — container-relative sizing can cause additional layout calculations.
- **Browser support** — container query units are Baseline widely available (Chrome 105+, Firefox 110+, Safari 16+).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fluid Typography and Spacing with Container Units

**HTML File (`units.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Query Units</title>
    <link rel="stylesheet" href="units.css">
</head>
<body>
    <div class="card-container">
        <div class="card">
            <h2 class="card-title">Fluid Typography</h2>
            <p class="card-text">
                This text scales with the container, not the viewport.
                Resize the browser to see the effect.
            </p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`units.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card-container {
    /* Establish the containment context */
    container-type: inline-size;
    max-width: 700px;
    margin: 0 auto;
}

.card {
    background-color: white;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    /* Padding scales with the container's inline size */
    padding: 5cqi;
}

.card-title {
    /* Font size is 8% of the container's inline size */
    font-size: clamp(1.25rem, 8cqi, 2.5rem);
    margin: 0 0 2cqi;
}

.card-text {
    /* Font size is 4% of the container's inline size */
    font-size: clamp(0.875rem, 4cqi, 1.125rem);
    line-height: 1.6;
    margin: 0;
    color: #555;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `units.html` and CSS as `units.css`.
2. Open in a browser at full width. Observe the card's typography and padding.
3. Narrow the browser window. Observe that the typography and padding scale down proportionally with the container.
4. The `clamp()` function ensures the text never becomes too small or too large.

**Expected Output:** A card whose title, body text, and padding scale proportionally with the container's width. The `clamp()` function bounds the scaling between a minimum and maximum size.

**Why This Works:** The `cqi` unit resolves against the `.card-container`'s inline size. `padding: 5cqi` means the padding is 5% of the container's width. `font-size: clamp(1.25rem, 8cqi, 2.5rem)` means the title font is at least `1.25rem`, at most `2.5rem`, and otherwise 8% of the container's width. As the container resizes, the typography and spacing scale smoothly, creating a fully fluid component.

---

### Real-World Cases

- **Card components:** Typography and padding that scale with the card's container size.
- **Dashboard tiles:** Spacing and font sizes that adapt to the tile's allocated space.
- **Hero sections:** Headlines that scale with the hero container rather than the viewport.
- **Sidebar widgets:** Compact typography and spacing that scale with the sidebar's width.

---

## References

- MDN Web Docs — CSS containment - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment
- MDN Web Docs — `@container` - https://developer.mozilla.org/en-US/docs/Web/CSS/@container
- MDN Web Docs — `container-type` - https://developer.mozilla.org/en-US/docs/Web/CSS/container-type
- MDN Web Docs — `container-name` - https://developer.mozilla.org/en-US/docs/Web/CSS/container-name
- MDN Web Docs — `container` (shorthand) - https://developer.mozilla.org/en-US/docs/Web/CSS/container
- MDN Web Docs — Using container size and style queries - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_size_and_style_queries
- CSS-Tricks — CSS Container Queries - https://css-tricks.com/css-container-queries/
- CSS-Tricks — The Range Syntax Has Come to Container Style Queries and if() - https://css-tricks.com/the-range-syntax-has-come-to-container-style-queries-and-if/
- CSS-Tricks — Container Units Should Be Pretty Handy - https://css-tricks.com/container-units-should-be-pretty-handy/
- web.dev — Container queries - https://web.dev/learn/css/container-queries
- Chrome for Developers — Container Queries Case Study - https://developer.chrome.com/blog/css-ui-ecommerce-cq
- W3C — CSS Containment Module Level 3 - https://www.w3.org/TR/css-contain-3/
- Can I Use — CSS Container Queries - https://caniuse.com/css-container-queries
- Can I Use — Container Query Units - https://caniuse.com/css-container-query-units