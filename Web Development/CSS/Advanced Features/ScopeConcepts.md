# CSS Scope Concepts — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Scope Concepts refers to the native scoping mechanisms in CSS that allow authors to limit the reach of style rules to specific DOM subtrees, isolate styles between components, and resolve cascade conflicts based on the proximity of a scoping root to the styled element rather than selector specificity alone. The primary mechanism is the `@scope` at-rule, which defines a scoping root and an optional scoping limit.

**Technical Definition:** The CSS Cascading and Inheritance Level 6 specification introduces the `@scope` rule, which allows authors to define a scope as a subtree or fragment of a document, used by selectors for more targeted matching. A scope is formed by determining a scoping root node and optional scoping limits that prune branches from the scope. The cascade behaviour introduced by scoping is **scope proximity**: declarations with a more proximate scoping root are prioritised, regardless of specificity or source order. Shadow DOM provides a separate encapsulation mechanism using `:host`, `:host-context()`, and `::part()` to allow controlled interaction between the outside page and shadow-encapsulated components.

**Beginner-Friendly Explanation:** Normally, CSS is global — a rule written anywhere can affect any element in the document. This makes it hard to isolate styles for individual components. CSS Scope Concepts give you tools to say "these styles only apply inside this specific part of the page." The `@scope` rule lets you draw a boundary: "start here, stop there." Shadow DOM takes this further with a hard encapsulation barrier, and special pseudo-elements let you reach through that barrier in controlled ways. The result is cleaner, more modular CSS that does not leak or conflict.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Native scoping** | `@scope` provides CSS-only scoping without Shadow DOM. |
| **Donut scope** | Scoping can have both an upper (root) and lower (limit) boundary. |
| **Scope proximity** | Cascade resolves ties based on DOM distance to the scoping root. |
| **Shadow encapsulation** | Shadow DOM creates a hard boundary for style isolation. |
| **Controlled piercing** | `:host`, `:host-context()`, and `::part()` provide controlled access across boundaries. |
| **Zero specificity roots** | The scoping root contributes zero specificity to the cascade (unless wrapped in `:where()`). |
| **Baseline emerging** | `@scope` support is recent; Shadow DOM is widely available. |

---

### Prerequisites

Before studying CSS Scope Concepts, you should understand:

- **CSS Cascade and Specificity** — how declarations compete and win.
- **CSS Selectors** — element, class, ID, attribute, and combinator selectors.
- **CSS Nesting** — the `&` selector and nested rules.
- **Web Components and Shadow DOM** — the encapsulation model for custom elements.
- **CSS Custom Properties** — the primary mechanism for theming across scope boundaries.

---

### Related Programming Areas

- **Web Components** — Shadow DOM encapsulation is the foundation of component-based architecture.
- **Design Systems** — scoping enables component-level style isolation.
- **CSS Architecture** — `@scope` and Shadow DOM provide alternatives to BEM and utility-first CSS.
- **CSS Cascade Layers** — layers and scopes work together to control cascade order.
- **Progressive Enhancement** — `@scope` degrades gracefully in browsers that do not support it.

---

### Core Concepts / Features

1. The `@scope` At-Rule: Scoping Roots and Boundaries
2. Component Boundaries: Preventing Style Leakage
3. Shadow DOM Encapsulation: `:host`, `:host-context()`, and `::part()`
4. Proximity vs. Specificity: How Scoping Alters the Cascade

---

## 1. The `@scope` At-Rule: Implementing Native CSS Scoping

### Definitions

**Core Definition:** The `@scope` at-rule allows authors to apply styles to a specific DOM subtree between a scoping root and an optional scoping limit. It provides native CSS scoping without requiring Shadow DOM.

**Technical Definition:** The `@scope` CSS at-rule enables you to select elements in specific DOM subtrees, precisely targeting elements without writing overly specific, hard-to-override selectors, and without coupling selectors too tightly to the DOM structure. The at-rule has the syntax `@scope [(<scope-start>)]? [to (<scope-end>)]? { <rule-list> }`. The `<scope-start>` selector list defines the scoping roots (the upper boundary), and the optional `<scope-end>` selector list defines the scoping limits (the lower boundary). The lower boundary is non-inclusive: elements matching the scope-end selector are excluded from the scope, but their descendants are not automatically excluded unless a donut scope is created. The scoping root contributes zero specificity to the cascade; the `&` selector inside `@scope` behaves like `:where(:scope)`.

**Beginner-Friendly Explanation:** The `@scope` rule is like drawing a fence around a part of your HTML. You say "start here" with the scoping root and optionally "stop here" with the scoping limit. Any style rules inside the `@scope` block will only apply to elements within that fenced area. This is useful for components — you can scope a card's styles to only apply inside the card, so they never leak out to other cards or the rest of the page.

---

### Purposes

- To apply styles to a specific DOM subtree without writing overly specific selectors.
- To create a "donut scope" with both an upper boundary (root) and a lower boundary (limit).
- To prevent styles from leaking outside their intended component.
- To reduce coupling between CSS and the DOM structure.
- To provide a CSS-only alternative to Shadow DOM for scoping.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
@scope (<scope-start>) to (<scope-end>) {
    <rule-list>
}

/* Without lower boundary */
@scope (<scope-start>) {
    <rule-list>
}

/* Inline @scope inside <style> */
<parent-element>
    <style>
        @scope {
            <rule-list>
        }
    </style>
</parent-element>
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<scope-start>` | The scoping root selector list (upper boundary). | `@scope (.card)` |
| `to (<scope-end>)` | The scoping limit selector list (lower boundary). | `@scope (.card) to (.slot)` |
| `<rule-list>` | The scoped style rules. | `h2 { color: blue; }` |

#### Syntax Rules

1. The `<scope-start>` selector list defines the elements that act as scoping roots.
2. The `<scope-end>` selector list defines elements that act as scoping limits; their descendants are excluded from the scope.
3. The lower boundary is non-inclusive: the element matching `<scope-end>` is in scope, but its descendants are not.
4. A scope with both boundaries is called a "donut scope" because it has a hole in the middle.
5. The scoping root contributes zero specificity to the cascade.
6. The `&` selector inside `@scope` behaves like `:where(:scope)` (zero specificity).
7. Pseudo-elements cannot be scoping roots or scoping limits.

#### Constraints and Limitations

- **Browser support** — `@scope` is not Baseline widely available; it is supported in Chrome 118+ (with evolving support in Firefox and Safari).
- **Specificity of scoping root** — the root contributes zero specificity, which may be surprising for authors expecting nesting-like behaviour.
- **No nesting inside declarations** — `@scope` is a rule-level at-rule.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `@scope` with a Single Boundary

**HTML File (`scope-basic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basic @scope</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="scope-basic.css">
</head>
<body>
    <div class="card">
        <h2>Card Title</h2>
        <p>Card description text.</p>
    </div>
    <div class="other">
        <h2>Other Heading</h2>
        <p>Other paragraph.</p>
    </div>
</body>
</html>
```

**CSS File (`scope-basic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

/* Scope styles to the .card component */
@scope (.card) {
    h2 {
        color: #006064;
        border-bottom: 2px solid #006064;
        padding-bottom: 8px;
    }
    p {
        color: #555;
        line-height: 1.6;
    }
}

/* Global styles for comparison */
.other h2 {
    color: #e74c3c;
}
.other p {
    color: #333;
}

.card, .other {
    background: white;
    padding: 24px;
    border-radius: 12px;
    margin-bottom: 20px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `scope-basic.html`.
3. Save the CSS code as `scope-basic.css` in the same folder.
4. Open `scope-basic.html` in a browser that supports `@scope` (Chrome 118+).
5. Observe that the `.card` h2 is teal with a border, while the `.other` h2 is red.

**Expected Output:** The card's headings and paragraphs are styled with teal colour and a bottom border. The other section's headings remain red. The scoped styles do not leak.

**Why This Works:** The `@scope (.card)` block creates a scope rooted at elements matching `.card`. Only descendants of `.card` are affected by the scoped rules. The `.other` section is outside the scope, so its styles remain unaffected.

---

#### Example 2: Donut Scope with Upper and Lower Boundaries

**HTML File (`scope-donut.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Donut Scope</title>
    <link rel="stylesheet" href="scope-donut.css">
</head>
<body>
    <div class="card">
        <h2>Card Title</h2>
        <p>This paragraph is in the donut.</p>
        <div class="slot">
            <h2>Slot Title</h2>
            <p>This paragraph is in the donut hole.</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`scope-donut.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card {
    background: white;
    padding: 24px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

/* Donut scope: root is .card, limit is .slot */
@scope (.card) to (.slot) {
    h2 {
        color: #006064;
        border-bottom: 2px solid #006064;
        padding-bottom: 8px;
    }
    p {
        color: #555;
        line-height: 1.6;
    }
}

.slot {
    margin-top: 16px;
    padding: 16px;
    background: #f0f0f0;
    border-radius: 8px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `scope-donut.html` and CSS as `scope-donut.css`.
2. Open in a browser that supports `@scope`.
3. Observe: the card's h2 and p are styled teal/grey, but the slot's h2 and p are NOT styled by the scoped rules.

**Expected Output:** The card's heading and paragraph are styled with the scoped rules. The slot's heading and paragraph are excluded because they are inside the donut hole.

**Why This Works:** The `@scope (.card) to (.slot)` creates a scope rooted at `.card` with a lower boundary at `.slot`. The slot element itself is in scope (the lower boundary is non-inclusive for the element itself), but its descendants are excluded. The donut hole is the `.slot` subtree.

---

### Real-World Cases

- **Component-scoped styles:** Scoping a card's styles so they do not leak to other cards.
- **Content slots:** Using donut scope to style a component's wrapper without affecting injected content.
- **Design systems:** Scoping utility styles to specific component subtrees.
- **Progressive enhancement:** Using `@scope` with a fallback for browsers that do not support it.

---

## 2. Component Boundaries: Creating Strict Layout Barriers

### Definitions

**Core Definition:** Component boundaries are the points in the DOM where styles from an outer scope stop affecting an inner component. They prevent parent styles from leaking into child components and vice versa.

**Technical Definition:** CSS scoping establishes boundaries through the `@scope` at-rule's lower boundary (`to (<scope-end>)`) or through Shadow DOM's encapsulation model. The lower boundary is non-inclusive: elements matching the scope-end selector are excluded from the scope, creating a barrier. In Shadow DOM, the shadow boundary is a hard encapsulation barrier: styles from the outside page do not apply inside the shadow tree, and styles from the shadow tree do not leak out. Controlled interaction across the boundary is provided by `:host`, `:host-context()`, `::part()`, and CSS custom properties.

**Beginner-Friendly Explanation:** A component boundary is like a wall between two rooms. Styles from one room cannot affect the other. In CSS scoping, the wall is the lower boundary of the scope — anything inside the "donut hole" is protected. In Shadow DOM, the wall is the shadow boundary — it is a hard barrier. The key is that you can still poke controlled holes in the wall to allow theming and communication.

---

### Purposes

- To prevent parent styles from accidentally affecting child components.
- To prevent child component styles from leaking out to the parent.
- To create predictable, isolated style environments.
- To enable component-based architecture with CSS.
- To provide a foundation for design system components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* @scope lower boundary as a barrier */
@scope (.parent) to (.child-component) {
    /* Styles here apply to .parent, but not inside .child-component */
}

/* Shadow DOM barrier (conceptual) */
/* Outside styles do not apply inside the shadow tree */
/* Inside styles do not leak out */
```

#### Component Breakdown

| Barrier Mechanism | Description | Example |
|---|---|---|
| `@scope` lower boundary | Excludes a subtree from the scope. | `@scope (.card) to (.slot)` |
| Shadow boundary | Hard encapsulation barrier. | `attachShadow({ mode: 'open' })` |
| Custom properties | Pierces boundaries for theming. | `--brand-color: blue;` |

#### Syntax Rules

1. The lower boundary is non-inclusive: the boundary element itself is in scope, but its descendants are not.
2. Multiple lower boundaries can be specified.
3. Shadow DOM boundaries are created via JavaScript (`attachShadow`).
4. Custom properties pierce both scope boundaries and Shadow DOM boundaries.
5. Regular CSS selectors cannot cross Shadow DOM boundaries.
6. `@scope` boundaries do not prevent inheritance of inherited properties (e.g., `color`, `font-family`).

#### Constraints and Limitations

- **Inheritance leakage** — inherited properties still cross boundaries; only style rules are scoped.
- **Custom property piercing** — custom properties are not blocked by scoping boundaries.
- **Browser support** — `@scope` is not universally supported.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Using a Lower Boundary as a Component Barrier

**HTML File (`boundary.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Component Boundary</title>
    <link rel="stylesheet" href="boundary.css">
</head>
<body>
    <div class="layout">
        <div class="sidebar">
            <h2>Sidebar</h2>
            <p>Sidebar content.</p>
        </div>
        <div class="main">
            <h2>Main Content</h2>
            <p>Main content.</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`boundary.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.layout {
    display: flex;
    gap: 20px;
}

/* Scope styles to .sidebar, but stop at .main */
@scope (.sidebar) to (.main) {
    h2 {
        color: #006064;
        border-bottom: 2px solid #006064;
    }
    p {
        color: #555;
    }
}

.sidebar, .main {
    background: white;
    padding: 24px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    flex: 1;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `boundary.html` and CSS as `boundary.css`.
2. Open in a browser that supports `@scope`.
3. Observe that only the sidebar's heading and paragraph are styled.

**Expected Output:** The sidebar's h2 is teal with a bottom border, and its p is grey. The main content's h2 and p are unstyled by the scoped rules.

**Why This Works:** The `@scope (.sidebar) to (.main)` creates a scope rooted at `.sidebar` with a lower boundary at `.main`. The `.main` element and its descendants are excluded from the scope, acting as a barrier.

---

### Real-World Cases

- **Layout systems:** Scoping sidebar styles without affecting main content.
- **Design systems:** Ensuring component styles do not leak between components.
- **CMS content:** Scoping editor styles so they do not affect the surrounding page.
- **Legacy migration:** Wrapping old CSS in a scope to isolate it from new code.

---

## 3. Shadow DOM Encapsulation: Writing Styles That Cross Architectural Boundaries

### Definitions

**Core Definition:** Shadow DOM is a web standard that allows developers to create a separate, encapsulated DOM tree for a custom element. Styles inside the shadow tree are scoped to that tree, and styles outside do not apply inside, except through specific pseudo-elements and custom properties.

**Technical Definition:** The CSS Shadow Module Level 1 defines the `::part()` pseudo-element, which allows an author to style specific, purposely exposed elements in a shadow tree from the outside page's context. The `:host` pseudo-class matches the shadow tree's shadow host, and its functional counterpart `:host()` matches the host only when the host matches the selector passed as an argument. The `:host-context()` pseudo-class matches the shadow host only when the host or its ancestors match the selector passed as an argument. Custom properties pierce the shadow boundary and can be used to pass theme values into the component. The cascade order for styles crossing shadow boundaries is defined so that host-scoped styles (from outside) take precedence over shadow-scoped styles (from inside) for `::part()` styling.

**Beginner-Friendly Explanation:** A Shadow DOM is like a private room inside your house. People outside cannot see or change what is inside, and people inside cannot affect the rest of the house. But you can install a window (`::part`) that lets people outside style specific things, and you can pass notes through the door (custom properties) to change the colours inside. The `:host` pseudo-class lets the component style its own outer container, and `:host-context()` lets it respond to its surroundings.

---

### Purposes

- To create hard encapsulation boundaries for component styles.
- To allow controlled theming from the outside via `::part()`.
- To allow the component to style itself based on its host or context via `:host` and `:host-context()`.
- To pass theme values into the component via custom properties.
- To prevent style leakage in both directions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* :host — from inside the shadow tree */
:host {
    display: block;
    background-color: var(--card-bg, white);
}

/* :host() — with a selector argument */
:host(.featured) {
    border: 2px solid gold;
}

/* :host-context() — based on ancestor context */
:host-context(.dark-mode) {
    background-color: #1a1a1a;
    color: white;
}

/* ::part() — from outside the shadow tree */
my-component::part(header) {
    font-size: 1.5rem;
}

/* Custom properties pierce the boundary */
:root {
    --card-bg: white;
}
```

#### Component Breakdown

| Pseudo-Class / Element | Usage | Description |
|---|---|---|
| `:host` | Inside shadow tree | Matches the shadow host element. |
| `:host(.selector)` | Inside shadow tree | Matches the host only if it matches the selector. |
| `:host-context(.selector)` | Inside shadow tree | Matches the host if it or an ancestor matches the selector. |
| `::part(name)` | Outside shadow tree | Styles an element in the shadow tree that has `part="name"`. |
| Custom properties | Both sides | Pierce the shadow boundary for theming. |

#### Syntax Rules

1. `:host` has the specificity of a pseudo-class; `:host()` adds the specificity of its argument.
2. `:host-context()` has the specificity of a pseudo-class plus the specificity of its argument.
3. `::part()` can only style elements that have been explicitly exposed with a `part` attribute.
4. Custom properties pierce the shadow boundary and inherit into the shadow tree.
5. Regular selectors cannot cross the shadow boundary.
6. `::slotted()` styles elements that are slotted into the shadow tree from the light DOM.

#### Constraints and Limitations

- **`::part()` requires opt-in** — the component author must add `part="name"` to the element.
- **No deep piercing** — `::part()` cannot target descendants of a part; only the part element itself.
- **Custom property inheritance** — custom properties inherit into the shadow tree but cannot be set from inside to affect the outside.
- **Browser support** — Shadow DOM is widely supported; `::part()` is Baseline widely available.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Theming a Web Component with `:host` and `::part()`

**HTML File (`shadow-theming.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Shadow DOM Theming</title>
    <style>
        :root {
            --card-bg: white;
            --card-accent: #3498db;
        }
        .dark-context {
            --card-bg: #1a1a1a;
            --card-accent: #e74c3c;
        }
        /* Style the exposed part from outside */
        my-card::part(header) {
            font-size: 1.5rem;
            text-transform: uppercase;
        }
        body {
            font-family: system-ui, sans-serif;
            padding: 40px;
            background: #f5f5f5;
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
                            background: var(--card-bg, white);
                            border-left: 4px solid var(--card-accent, #ccc);
                            border-radius: 12px;
                            padding: 24px;
                            margin: 16px 0;
                            font-family: system-ui, sans-serif;
                        }
                        :host-context(.dark-context) {
                            color: #e8e8e8;
                        }
                        h2 {
                            color: var(--card-accent, #ccc);
                            margin-top: 0;
                        }
                    </style>
                    <h2 part="header">Shadow Component</h2>
                    <p>This component is themed from outside via custom properties and ::part().</p>
                `;
            }
        }
        customElements.define('my-card', MyCard);
    </script>
</body>
</html>
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `shadow-theming.html`.
2. Open in a browser.
3. Observe: the first card has a blue accent and white background. The second card (inside `.dark-context`) has a red accent and dark background. The `::part(header)` styles apply to both.

**Expected Output:** Two cards from the same Web Component with different theming. The `::part(header)` pseudo-element styles the exposed header element from outside the shadow DOM.

**Why This Works:** The custom properties `--card-bg` and `--card-accent` are set outside the component and inherit into the shadow tree. The `:host` selector reads them inside the component. The `:host-context(.dark-context)` changes the text colour based on the ancestor context. The `::part(header)` styles the exposed header element from the outside page.

---

### Real-World Cases

- **Design systems:** Theming Web Components with custom properties.
- **Multi-brand platforms:** Different brands set different custom properties.
- **Dark mode:** `:host-context(.dark-mode)` for context-aware theming.
- **Component libraries:** Exposing parts for controlled customisation.

---

## 4. Proximity vs. Specificity: How Scoping Alters the Cascade

### Definitions

**Core Definition:** Scope proximity is a cascade criterion that prioritises declarations based on how close the scoping root is to the element being styled, above and beyond selector specificity. It is a new step in the cascade order introduced by `@scope`.

**Technical Definition:** When resolving a selector within an `@scope () {}` at-rule, selectors with the same specificity rely on DOM proximity to decide a winner above what would have otherwise been source order. The cascade now includes scope proximity as a step: declarations with a more proximate scoping root are prioritised, regardless of specificity or source order. In the current specification, the proximity weight is compared from innermost scoping relationship to outermost, with any missing pairs weighted as infinity. The `&` selector inside `@scope` behaves like `:where(:scope)` with zero specificity, so the scoping root does not contribute specificity to the cascade.

**Beginner-Friendly Explanation:** Normally, if two rules have the same specificity, the one that appears later in the CSS wins. With `@scope`, there is a new tiebreaker: proximity. If two rules both match an element, but one comes from a scope whose root is closer to the element (more deeply nested), that one wins — even if it appears earlier in the CSS. This makes CSS work more intuitively: a style defined for a more specific (closer) scope beats a style defined for a broader (further) scope.

---

### Purposes

- To resolve cascade conflicts based on DOM distance rather than source order.
- To make component-scoped styles win over global styles without specificity hacks.
- To create more intuitive cascade behaviour where closer scopes win.
- To reduce the need for `!important` or high-specificity selectors.
- To provide a predictable cascade for component-based architectures.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Scope A: broad scope */
@scope (.container) {
    p {
        color: blue;
    }
}

/* Scope B: narrow scope (more proximate) */
@scope (.card) {
    p {
        color: green; /* Wins because .card is closer to the p */
    }
}
```

#### The Cascade Order (with Scope Proximity)

```
1. Transition declarations (highest)
2. Important user agent declarations
3. Important user declarations
4. Important author declarations
5. Animation declarations
6. Normal author declarations (sorted by):
   a. Cascade layer order
   b. Scope proximity (closer scope wins)
   c. Specificity
   d. Source order
7. Normal user declarations
8. Normal user agent declarations
```

#### Syntax Rules

1. Scope proximity is evaluated after layer order but before specificity.
2. A more proximate scoping root wins over a less proximate one.
3. Proximity is measured by the number of DOM generations between the scoping root and the styled element.
4. The `&` selector inside `@scope` has zero specificity, so the root does not add to specificity.
5. Proximity is compared from innermost scoping relationship to outermost.
6. When scopes are nested, the innermost scope wins.

#### Constraints and Limitations

- **New cascade step** — scope proximity is a relatively new concept; browser implementations may vary.
- **Debugging complexity** — proximity-based cascade resolution is not always visible in DevTools.
- **Interaction with `:where()`** — using `:where()` in scoping roots can drop specificity to zero, making proximity the sole tiebreaker.
- **Browser support** — `@scope` and proximity are not universally supported.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Proximity Wins Over Source Order

**HTML File (`proximity.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scope Proximity</title>
    <link rel="stylesheet" href="proximity.css">
</head>
<body>
    <div class="container">
        <div class="card">
            <p>Which colour wins?</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`proximity.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    background: white;
    padding: 24px;
    border-radius: 12px;
}

.card {
    background: #f0f0f0;
    padding: 24px;
    border-radius: 8px;
    margin-top: 16px;
}

/* Broad scope: .container */
@scope (.container) {
    p {
        color: #3498db; /* Blue */
    }
}

/* Narrow scope: .card (more proximate) */
@scope (.card) {
    p {
        color: #e74c3c; /* Red — wins because .card is closer */
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `proximity.html` and CSS as `proximity.css`.
2. Open in a browser that supports `@scope`.
3. Observe that the paragraph is red, not blue.

**Expected Output:** The paragraph inside the card is red. The `.card` scope is more proximate to the paragraph than the `.container` scope, so it wins — even though the `.container` scope might appear later in the CSS (or vice versa).

**Why This Works:** Both scopes match the paragraph with the same specificity (0,0,1). The cascade resolves the tie by scope proximity: the `.card` scope is closer to the paragraph (fewer DOM generations) than the `.container` scope, so the red declaration wins.

---

#### Example 2: Proximity with `:where()` to Drop Specificity

**HTML File (`proximity-where.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Proximity with :where()</title>
    <link rel="stylesheet" href="proximity-where.css">
</head>
<body>
    <div class="container">
        <div class="card">
            <p>Which colour wins?</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`proximity-where.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    background: white;
    padding: 24px;
    border-radius: 12px;
}

.card {
    background: #f0f0f0;
    padding: 24px;
    border-radius: 8px;
    margin-top: 16px;
}

/* Broad scope with a class selector (specificity 0,1,0) */
@scope (.container) {
    p {
        color: #3498db; /* Blue */
    }
}

/* Narrow scope with :where() — drops specificity to zero */
@scope (:where(.card)) {
    p {
        color: #e74c3c; /* Red — but specificity is now (0,0,1) */
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `proximity-where.html` and CSS as `proximity-where.css`.
2. Open in a browser that supports `@scope`.
3. Observe: the paragraph is red because proximity still wins, even though `:where()` dropped the specificity of the `.card` scope.

**Expected Output:** The paragraph is red. The `:where(.card)` scope has zero specificity, but proximity still favours the closer scope.

**Why This Works:** The `:where(.card)` scoping root has zero specificity. Both scopes now have the same specificity (0,0,1) for the `p` selector. The cascade resolves the tie by scope proximity, and the `.card` scope wins because it is closer.

---

### Real-World Cases

- **Component libraries:** Component-scoped styles naturally win over global styles.
- **Design systems:** Local component overrides beat global defaults.
- **Content scoping:** A style inside a nested scope wins over a broader scope.
- **Debugging:** Understanding why a closer scope wins despite source order.

---

## References

- MDN Web Docs — `@scope` - https://developer.mozilla.org/en-US/docs/Web/CSS/@scope
- CSS-Tricks — `@scope` - https://css-tricks.com/almanac/rules/s/scope/
- W3C — CSS Cascading and Inheritance Level 6 (Editor's Draft) - https://drafts.csswg.org/css-cascade-6/
- W3C — CSS Shadow Module Level 1 - https://drafts.csswg.org/css-shadow-1/
- MDN Web Docs — `:host` - https://developer.mozilla.org/en-US/docs/Web/CSS/:host
- MDN Web Docs — `:host-context()` - https://developer.mozilla.org/en-US/docs/Web/CSS/:host-context
- MDN Web Docs — `::part()` - https://developer.mozilla.org/en-US/docs/Web/CSS/::part
- Frontend Masters — Proximity Scope (`@scope` in CSS) - https://frontendmasters.com/tutorials/chris-coyier/proximity-scope/
- W3C CSSWG — [css-cascade-6] The specificity of a scope rule - https://lists.w3.org/Archives/Public/public-css-archive/2023Feb/0912.html
- Web Platform DX — `@scope` - https://web-platform-dx.github.io/web-features-explorer/features/scope/
- Chrome for Developers — `@scope` - https://developer.chrome.com/docs/css-ui/at-scope