# CSS Dynamic Theming — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Dynamic Theming is the discipline of building interfaces whose visual appearance can change at runtime in response to user preferences, application state, or programmatic control — without reloading the page or duplicating stylesheets. It combines CSS custom properties, media queries, data attributes, and JavaScript DOM APIs into a unified theming architecture.

**Technical Definition:** Dynamic theming in CSS is implemented through the interaction of several specifications: the CSS Custom Properties for Cascading Variables Module (which provides the `var()` function and `--`-prefixed custom properties), Media Queries Level 5 (which provides `prefers-color-scheme`), the CSS Color Module Level 5 (which provides the `light-dark()` function and the `color-scheme` property), the DOM Standard (which provides `data-*` attributes and `element.dataset`), and the CSS Object Model (which provides `element.style.setProperty()` and `getComputedStyle()`). A dynamic theming system typically defines design tokens as custom properties on the `:root` element and overrides them within scoped selectors (`[data-theme="dark"]`, `@media (prefers-color-scheme: dark)`) or at runtime via JavaScript.

**Beginner-Friendly Explanation:** Dynamic theming means your website can change its colours, typography, and overall look based on what the user prefers or what your application decides. If the user has dark mode enabled on their phone, your site can automatically switch to a dark theme. If they click a "cyberpunk" theme button, your site can instantly transform. This is possible because CSS custom properties are live — when you change their value, every element that uses them updates immediately, without a page reload.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **System-aware** | Detects and responds to OS-level colour scheme preferences. |
| **State-driven** | Switches themes via data attributes or classes toggled by JavaScript. |
| **Scoped** | Themes can be applied globally or scoped to individual components. |
| **Runtime-modifiable** | Custom property values can be changed live via JavaScript. |
| **Non-destructive** | Swapping themes does not require reloading or re-rendering the DOM. |
| **Progressive** | Provides sensible defaults when JavaScript is unavailable. |
| **Composable** | Component-level overrides do not pollute the global scope. |

---

### Prerequisites

Before studying CSS Dynamic Theming, you should understand:

- **CSS Custom Properties** — declaration, `var()`, and scoping.
- **CSS Media Queries** — the `@media` at-rule and `prefers-color-scheme`.
- **CSS Selectors** — attribute selectors (`[data-theme]`) and the `:root` pseudo-class.
- **JavaScript DOM APIs** — `setAttribute()`, `dataset`, `style.setProperty()`, and `getComputedStyle()`.
- **CSS Cascade and Specificity** — how overrides interact.

---

### Related Programming Areas

- **Design Systems** — dynamic theming is the runtime layer of a token architecture.
- **Accessibility** — respecting `prefers-color-scheme` and `prefers-contrast`.
- **UI Frameworks** — React, Vue, and Svelte integrate theming with reactive state.
- **Progressive Web Apps** — dynamic theming enhances the app-like experience.
- **Web Components** — custom properties pierce Shadow DOM for theming.

---

### Core Concepts / Features

1. System Preference Themes: `@media (prefers-color-scheme: dark)` and `light-dark()`
2. State and Attribute-Based Themes: `data-theme` Attribute Swapping
3. Component-Level Overrides: Scoped Custom Properties
4. Runtime DOM Mutations: `setProperty()` and `getComputedStyle()`

---

## 1. System Preference Themes: Detecting System Preferences Natively

### Definitions

**Core Definition:** System preference themes use the `prefers-color-scheme` media feature and the `light-dark()` colour function to automatically adapt the interface to the user's operating-system-level colour scheme preference (light or dark).

**Technical Definition:** The `prefers-color-scheme` CSS media feature is used to detect if a user has requested light or dark colour themes. A user indicates their preference through an operating system setting (e.g., light or dark mode) or a user agent setting. The `light-dark()` CSS function accepts two colours (or two images) and returns the first value if the used colour scheme is light or if no preference is set, and the second value if the used colour scheme is dark. To enable `light-dark()`, the `color-scheme` property must have a value of `light dark`, usually set on the `:root` pseudo-class. The `color-scheme` property allows an element to indicate which colour schemes it can comfortably render, affecting browser-provided UI such as scrollbars, form controls, and system colours.

**Beginner-Friendly Explanation:** When a user enables dark mode on their phone or computer, `prefers-color-scheme: dark` detects that preference. You can then override your colour custom properties inside the media query. Alternatively, the newer `light-dark()` function lets you write a single declaration that automatically picks the right colour for the current scheme — no media query needed. You just need to set `color-scheme: light dark` on `:root` first.

---

### Purposes

- To automatically respect the user's operating system colour scheme.
- To provide a dark-mode experience without requiring a manual toggle.
- To reduce code duplication by using `light-dark()` instead of media queries.
- To ensure browser-provided UI (scrollbars, form controls) matches the theme.
- To provide a baseline theme that works even when JavaScript is disabled.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Approach 1: prefers-color-scheme media query */
:root {
    --bg: #ffffff;
    --text: #1a1a1a;
}

@media (prefers-color-scheme: dark) {
    :root {
        --bg: #1a1a1a;
        --text: #e8e8e8;
    }
}

/* Approach 2: light-dark() function */
:root {
    color-scheme: light dark;
}

body {
    background-color: light-dark(#ffffff, #1a1a1a);
    color: light-dark(#1a1a1a, #e8e8e8);
}
```

#### Component Breakdown

| Feature / Property | Description | Values |
|---|---|---|
| `prefers-color-scheme` | Media feature detecting user preference. | `light`, `dark`, `no-preference` |
| `light-dark()` | Function returning a value based on active colour scheme. | Two `<color>` or `<image>` values |
| `color-scheme` | Indicates which schemes the element supports. | `normal`, `light`, `dark`, `light dark`, `only light` |

#### Syntax Rules

1. `prefers-color-scheme` is a media feature used inside `@media` queries.
2. The values are `light`, `dark`, and `no-preference` (the latter is deprecated; use `light` as default).
3. `light-dark()` requires `color-scheme: light dark` on the same element or an ancestor.
4. `light-dark()` returns the first value for light or unknown schemes, the second for dark.
5. `color-scheme: light dark` tells the browser both schemes are supported.
6. `color-scheme: only light` forces light mode, disabling browser auto-darkening.
7. The `color-scheme` property affects browser-provided UI (scrollbars, form controls, system colours).

#### Constraints and Limitations

- **No JavaScript required** — works even if scripts are blocked.
- **`light-dark()` browser support** — Baseline 2024 (Chrome 123+, Firefox 120+, Safari 17.5+).
- **`prefers-color-scheme` support** — Baseline widely available since January 2020.
- **`color-scheme` required** — `light-dark()` does nothing without `color-scheme: light dark`.
- **No forced theme** — `prefers-color-scheme` reflects the OS setting; users cannot override it from within the page without JavaScript.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Automatic Dark Mode with `prefers-color-scheme`

**HTML File (`system-theme.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Declares colour scheme support for browser UI -->
    <meta name="color-scheme" content="light dark">
    <title>System Preference Theme</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="system-theme.css">
</head>
<body>
    <div class="card">
        <h2>Automatic Dark Mode</h2>
        <p>This card adapts to your system colour scheme automatically.</p>
    </div>
</body>
</html>
```

**CSS File (`system-theme.css`):**

```css
/* Light theme defaults */
:root {
    --color-bg: #f9fafb;
    --color-surface: #ffffff;
    --color-text-primary: #111827;
    --color-text-secondary: #4b5563;
    --color-border: #e5e7eb;

    /* Tell the browser both schemes are supported */
    color-scheme: light dark;
}

/* Dark theme overrides */
@media (prefers-color-scheme: dark) {
    :root {
        --color-bg: #111827;
        --color-surface: #1f2937;
        --color-text-primary: #f9fafb;
        --color-text-secondary: #d1d5db;
        --color-border: #374151;
    }
}

body {
    font-family: system-ui, sans-serif;
    background-color: var(--color-bg);
    color: var(--color-text-primary);
    margin: 0;
    padding: 40px;
    transition: background-color 300ms, color 300ms;
}

.card {
    background-color: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: 12px;
    padding: 30px;
    max-width: 400px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card h2 {
    color: var(--color-text-primary);
    margin-top: 0;
}

.card p {
    color: var(--color-text-secondary);
    margin-bottom: 0;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `system-theme.html`.
3. Save the CSS code as `system-theme.css` in the same folder.
4. Open `system-theme.html` in a web browser.
5. Toggle dark mode in your operating system settings. The card should switch between light and dark themes automatically.

**Expected Output:** A card that automatically adapts to the system colour scheme. In light mode, it has a white background and dark text; in dark mode, it has a dark background and light text. The `color-scheme: light dark` declaration also ensures that scrollbars and form controls match the theme.

**Why This Works:** The `:root` selector defines the light-theme custom properties. The `@media (prefers-color-scheme: dark)` block overrides them when the user's system is in dark mode. Because all component styles reference the custom properties via `var()`, they update automatically when the properties change.

---

#### Example 2: `light-dark()` with `color-scheme`

**HTML File (`light-dark.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>light-dark() Function</title>
    <link rel="stylesheet" href="light-dark.css">
</head>
<body>
    <div class="card">
        <h2>light-dark() Demo</h2>
        <p>This card uses the light-dark() function for its colours.</p>
    </div>
</body>
</html>
```

**CSS File (`light-dark.css`):**

```css
:root {
    /* Required for light-dark() to work */
    color-scheme: light dark;
}

body {
    font-family: system-ui, sans-serif;
    /* Single declaration handles both themes */
    background-color: light-dark(#f9fafb, #111827);
    color: light-dark(#111827, #f9fafb);
    margin: 0;
    padding: 40px;
    transition: background-color 300ms, color 300ms;
}

.card {
    background-color: light-dark(#ffffff, #1f2937);
    border: 1px solid light-dark(#e5e7eb, #374151);
    color: light-dark(#111827, #f9fafb);
    border-radius: 12px;
    padding: 30px;
    max-width: 400px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card h2 {
    color: light-dark(#111827, #f9fafb);
    margin-top: 0;
}

.card p {
    color: light-dark(#4b5563, #d1d5db);
    margin-bottom: 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `light-dark.html` and CSS as `light-dark.css`.
2. Open in a browser that supports `light-dark()` (Chrome 123+, Firefox 120+, Safari 17.5+).
3. Toggle your system dark mode. The card switches automatically.

**Expected Output:** The same automatic dark-mode behaviour as the previous example, but with a single declaration per property instead of a media query and separate custom property overrides.

**Why This Works:** The `light-dark()` function takes two values: the first for light mode, the second for dark mode. The browser checks the computed `color-scheme` value and returns the appropriate colour. This eliminates the need for a `@media (prefers-color-scheme)` block and separate custom property overrides.

---

### Real-World Cases

- **Operating system integration:** Websites that automatically match the user's light/dark preference.
- **Accessibility:** Respecting user preferences for reduced eye strain.
- **Browser UI consistency:** `color-scheme` ensures scrollbars and form controls match the theme.
- **Baseline theming:** Providing a system-aware theme without JavaScript or user interaction.

---

## 2. State and Attribute-Based Themes: Toggling Global Layouts via `data-theme`

### Definitions

**Core Definition:** State and attribute-based theming uses `data-*` attributes (or classes) on the root or body element to apply a named theme. JavaScript toggles the attribute, and CSS selectors like `[data-theme="cyberpunk"]` apply the corresponding token overrides.

**Technical Definition:** The `data-*` global attributes form a class of attributes called custom data attributes, which allow proprietary information to be exchanged between HTML and its DOM representation. When a theme is changed, JavaScript updates `document.documentElement.dataset.theme` (which sets the `data-theme` attribute on the `<html>` element). CSS then uses attribute selectors such as `:root[data-theme="dark"]` to apply theme-specific custom property overrides. This approach is more flexible than class-based theming because data attributes express semantic state rather than presentational styling.

**Beginner-Friendly Explanation:** You put a label on the `<html>` element that says which theme is active — for example, `data-theme="cyberpunk"`. In your CSS, you write different sets of custom property values for each theme: `:root[data-theme="light"]`, `:root[data-theme="dark"]`, `:root[data-theme="cyberpunk"]`. When the user clicks a theme button, JavaScript changes the label, and the browser instantly applies the new theme's values. Because all your components reference the custom properties, they all update at once.

---

### Purposes

- To allow users to manually select a theme from multiple options.
- To support themes beyond light and dark (e.g., high contrast, brand variants).
- To persist the user's theme choice across sessions via `localStorage`.
- To provide a clean separation between theme state (HTML attribute) and theme styling (CSS).
- To enable programmatic theme switching via JavaScript frameworks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<!-- The theme attribute on the html element -->
<html data-theme="light">
```

```css
/* Default theme tokens */
:root {
    --color-bg: #ffffff;
    --color-text: #1a1a1a;
}

/* Dark theme overrides */
:root[data-theme="dark"] {
    --color-bg: #1a1a1a;
    --color-text: #e8e8e8;
}

/* Cyberpunk theme overrides */
:root[data-theme="cyberpunk"] {
    --color-bg: #0a0a2e;
    --color-text: #00ffcc;
    --color-accent: #ff00aa;
}
```

```javascript
// JavaScript to toggle the theme
document.documentElement.setAttribute('data-theme', 'cyberpunk');
// Or using dataset
document.documentElement.dataset.theme = 'cyberpunk';
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `data-theme` attribute | Applied to `<html>` or `<body>`. | `data-theme="dark"` |
| Attribute selector | CSS selector matching the theme. | `:root[data-theme="dark"]` |
| `element.dataset.theme` | JavaScript accessor for the attribute. | `document.documentElement.dataset.theme = 'dark'` |
| `localStorage` | Persists the theme choice. | `localStorage.setItem('theme', 'dark')` |

#### Syntax Rules

1. The `data-theme` attribute should be placed on the `<html>` element for global theming.
2. Attribute values are case-insensitive in HTML but case-sensitive in CSS selectors when using `[data-theme="dark"]`.
3. The `:root[data-theme="..."]` selector has higher specificity than `:root` alone, ensuring overrides win.
4. Multiple themes can coexist — each with its own selector block.
5. `element.dataset.theme` is the JavaScript equivalent of `setAttribute('data-theme', value)`.
6. Persist the theme choice in `localStorage` and restore it on page load to prevent flash of incorrect theme (FOUC).
7. Use an inline script in `<head>` to apply the stored theme before the page renders.

#### Constraints and Limitations

- **Flash of incorrect theme** — if the theme is applied after the page renders, users may see a flash of the default theme. Mitigate with an inline script in `<head>`.
- **Specificity management** — deeply nested attribute selectors can become complex; keep theme blocks at the `:root` level.
- **JavaScript dependency** — manual theme switching requires JavaScript; provide a sensible default for no-JS environments.
- **No CSS-only persistence** — `localStorage` persistence requires JavaScript.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Theme Switcher with Persistence

**HTML File (`multi-theme.html`):**

```html
<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multi-Theme Switcher</title>
    <link rel="stylesheet" href="multi-theme.css">
    <!-- Inline script to apply stored theme before render (prevents FOUC) -->
    <script>
        (function() {
            const stored = localStorage.getItem('theme');
            if (stored) {
                document.documentElement.setAttribute('data-theme', stored);
            }
        })();
    </script>
</head>
<body>
    <div class="theme-switcher">
        <button data-theme-set="light">Light</button>
        <button data-theme-set="dark">Dark</button>
        <button data-theme-set="cyberpunk">Cyberpunk</button>
    </div>

    <div class="card">
        <h2>Multi-Theme Card</h2>
        <p>Click a theme button to switch.</p>
    </div>

    <script>
        const buttons = document.querySelectorAll('[data-theme-set]');
        buttons.forEach(btn => {
            btn.addEventListener('click', () => {
                const theme = btn.dataset.themeSet;
                document.documentElement.setAttribute('data-theme', theme);
                localStorage.setItem('theme', theme);
            });
        });
    </script>
</body>
</html>
```

**CSS File (`multi-theme.css`):**

```css
/* Light theme (default) */
:root {
    --color-bg: #f9fafb;
    --color-surface: #ffffff;
    --color-text-primary: #111827;
    --color-text-secondary: #4b5563;
    --color-accent: #3b82f6;
    --color-border: #e5e7eb;
}

/* Dark theme */
:root[data-theme="dark"] {
    --color-bg: #111827;
    --color-surface: #1f2937;
    --color-text-primary: #f9fafb;
    --color-text-secondary: #d1d5db;
    --color-accent: #60a5fa;
    --color-border: #374151;
}

/* Cyberpunk theme */
:root[data-theme="cyberpunk"] {
    --color-bg: #0a0a2e;
    --color-surface: #12123e;
    --color-text-primary: #00ffcc;
    --color-text-secondary: #7a7aff;
    --color-accent: #ff00aa;
    --color-border: #ff00aa;
}

body {
    font-family: system-ui, sans-serif;
    background-color: var(--color-bg);
    color: var(--color-text-primary);
    margin: 0;
    padding: 40px;
    transition: background-color 300ms, color 300ms;
}

.theme-switcher {
    display: flex;
    gap: 10px;
    margin-bottom: 30px;
}

.theme-switcher button {
    padding: 10px 20px;
    border: 2px solid var(--color-border);
    background-color: var(--color-surface);
    color: var(--color-text-primary);
    border-radius: 8px;
    cursor: pointer;
    font-weight: bold;
    transition: border-color 200ms, background-color 200ms;
}

.theme-switcher button:hover {
    border-color: var(--color-accent);
}

.card {
    background-color: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: 12px;
    padding: 30px;
    max-width: 400px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card h2 {
    color: var(--color-text-primary);
    margin-top: 0;
}

.card p {
    color: var(--color-text-secondary);
    margin-bottom: 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `multi-theme.html` and CSS as `multi-theme.css`.
2. Open in a browser.
3. Click the theme buttons to switch between light, dark, and cyberpunk themes.
4. Reload the page — the last selected theme persists via `localStorage`.

**Expected Output:** A page with a theme switcher and a card. Clicking each button instantly applies the corresponding theme. The theme persists across page reloads.

**Why This Works:** The `data-theme` attribute on `<html>` drives the CSS attribute selectors. Each theme block overrides the custom properties. The inline script in `<head>` applies the stored theme before rendering, preventing a flash of the default theme. The buttons use `data-theme-set` attributes and a single event listener to update the theme.

---

### Real-World Cases

- **SaaS applications:** User-selectable themes (light, dark, high contrast, brand variants).
- **E-commerce:** Seasonal or campaign-specific themes.
- **Documentation sites:** Themes for different reading preferences.
- **Multi-brand platforms:** Each brand maps to a `data-theme` value.

---

## 3. Component-Level Overrides: Injecting Targeted Variant Custom Properties Inside Specific Containers

### Definitions

**Core Definition:** Component-level overrides involve setting custom properties on a specific container (or component instance) so that only that component and its descendants receive the overridden values, without polluting the global scope or affecting other instances.

**Technical Definition:** Because CSS custom properties inherit, setting a custom property on a component's root element scopes the override to that component's subtree. This is the recommended pattern for component customisation: components expose a set of custom properties as their "public API," and consumers override those properties on the component instance. The key architectural rule is to keep custom properties as configurable values and use CSS property declarations for state handling (`:hover`, `:active`, `[data-disabled]`), so that higher layers can safely override the custom property values without interfering with how states resolve. This pattern is used by Bulma, Lit, and the WordPress Gutenberg design system.

**Beginner-Friendly Explanation:** Imagine a button component that has a custom property called `--button-bg`. Normally, it is blue. If you want one specific button to be red, you set `--button-bg: red` on that button element (or its parent container). Only that button changes — every other button stays blue. This is component-level overrides: scoping a variable change to a single instance without affecting the global theme.

---

### Purposes

- To customise individual component instances without affecting others.
- To expose a controlled "public API" for component styling.
- To prevent global theme pollution from component-specific changes.
- To enable composition where a parent component overrides a child's tokens.
- To support variant-specific styling within a single page.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Component defines its custom properties */
.button {
    --button-bg: var(--color-action-primary, #3b82f6);
    --button-text: var(--color-action-text, white);

    background-color: var(--button-bg);
    color: var(--button-text);
}

/* Consumer overrides the custom property on a specific instance */
.button.danger {
    --button-bg: #ef4444;
    --button-text: white;
}

/* Or on a parent container */
.dialog--danger .button {
    --button-bg: #ef4444;
}
```

#### Component Breakdown

| Pattern | Description | Example |
|---|---|---|
| Component property definition | Component declares its tokens with fallbacks. | `--button-bg: var(--color-action-primary, #3b82f6)` |
| Instance-level override | Override on the specific component element. | `.button.danger { --button-bg: #ef4444; }` |
| Parent-level override | Override on an ancestor container. | `.dialog--danger .button { --button-bg: #ef4444; }` |
| Fallback chain | Component provides defaults if consumer does not override. | `var(--button-bg, var(--color-action-primary, #3b82f6))` |

#### Syntax Rules

1. Components should define their custom properties with fallbacks so they work without consumer overrides.
2. Overrides are applied by setting the custom property on the component element or an ancestor.
3. Because custom properties inherit, an override on a parent affects all descendant components that read that property.
4. State handling (`:hover`, `:active`, `[data-disabled]`) should use CSS property declarations, not custom property reassignment.
5. Component tokens should follow a consistent naming convention (e.g., `--{component}-{property}-{state}`).
6. Use `@property` to register component tokens with types and initial values for safer overrides.
7. Custom properties pierce Shadow DOM boundaries, making them the preferred theming API for Web Components.

#### Constraints and Limitations

- **Inheritance leakage** — custom properties inherit to all descendants; a broad override may affect unintended components.
- **Specificity management** — overrides must have sufficient specificity to win over the component's default declarations.
- **State handling complexity** — reassigning custom properties inside state selectors can break when higher layers override them; keep state handling in CSS property declarations.
- **No CSS-only isolation** — custom properties are not scoped to a component; they always inherit.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Button Component with Variant Overrides

**HTML File (`component-override.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Component-Level Overrides</title>
    <link rel="stylesheet" href="component-override.css">
</head>
<body>
    <div class="button-row">
        <!-- Default button -->
        <button class="btn">Default</button>
        <!-- Danger variant via class -->
        <button class="btn btn--danger">Danger</button>
        <!-- Warning variant via parent -->
        <div class="alert--warning">
            <button class="btn">Warning Context</button>
        </div>
    </div>
</body>
</html>
```

**CSS File (`component-override.css`):**

```css
:root {
    /* Global design tokens */
    --color-action-primary: #3b82f6;
    --color-action-text: #ffffff;
    --color-danger: #ef4444;
    --color-warning: #f59e0b;
    --radius-md: 8px;
    --space-3: 0.75rem;
    --space-6: 1.5rem;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.button-row {
    display: flex;
    gap: 16px;
    align-items: center;
}

/* ===== COMPONENT: Button ===== */
.btn {
    /* Component tokens (configurable) */
    --btn-bg: var(--color-action-primary);
    --btn-text: var(--color-action-text);
    --btn-radius: var(--radius-md);

    /* Consume the tokens */
    background-color: var(--btn-bg);
    color: var(--btn-text);
    border: none;
    padding: var(--space-3) var(--space-6);
    border-radius: var(--btn-radius);
    font-weight: bold;
    cursor: pointer;
    transition: filter 200ms;
}

.btn:hover {
    filter: brightness(0.9);
}

/* Variant: Danger (overrides component tokens) */
.btn--danger {
    --btn-bg: var(--color-danger);
}

/* Context: Warning alert (overrides via parent) */
.alert--warning .btn {
    --btn-bg: var(--color-warning);
    --btn-text: #1a1a1a;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `component-override.html` and CSS as `component-override.css`.
2. Open in a browser.
3. Observe: the default button is blue, the danger button is red, and the button inside the warning alert is amber.

**Expected Output:** Three buttons with different colours, all using the same `.btn` component class. The colour changes are driven by component-level custom property overrides.

**Why This Works:** The `.btn` component defines `--btn-bg` and `--btn-text` with global token fallbacks. The `.btn--danger` class overrides `--btn-bg` on that specific button. The `.alert--warning .btn` selector overrides `--btn-bg` on any `.btn` inside a warning alert. Because custom properties inherit, the override on the alert container flows down to the button. No global tokens were modified.

---

### Real-World Cases

- **Design systems:** Components expose `--button-*`, `--card-*`, `--input-*` tokens as their public API.
- **Web Components:** Custom properties pierce Shadow DOM for theming.
- **Composition:** A dialog component overrides button tokens for a destructive action context.
- **Variant styling:** Success, warning, danger, and info variants of the same component.

---

## 4. Runtime DOM Mutations: Reading and Updating Active Variable Values Live

### Definitions

**Core Definition:** Runtime DOM mutations use JavaScript's `element.style.setProperty()` to change custom property values live and `getComputedStyle().getPropertyValue()` to read the current computed values, enabling dynamic, programmatic theming and component adaptation.

**Technical Definition:** The `style.setProperty()` method on an element's inline style object allows setting a CSS property (including custom properties) to a new value. When the property is a custom property and the element is `:root` (`document.documentElement`), the change cascades to all elements that reference that variable. The `getComputedStyle()` method returns the resolved computed value of a property, which for custom properties means the value after all cascade and inheritance rules have been applied. The `removeProperty()` method removes an inline override, restoring the stylesheet default.

**Beginner-Friendly Explanation:** You can change a theme colour from JavaScript by calling `document.documentElement.style.setProperty('--primary', '#ff0000')`. This writes an inline style on the `<html>` element that overrides the stylesheet value. Every element that uses `var(--primary)` updates instantly. You can also read the current value with `getComputedStyle(document.documentElement).getPropertyValue('--primary')`. This is how you build dynamic colour pickers, live theme editors, and adaptive UI that responds to user input.

---

### Purposes

- To change theme colours live without reloading or re-rendering.
- To read the current computed value of a custom property.
- To build dynamic UI (colour pickers, sliders) that updates the theme in real time.
- To scope changes to a specific element or subtree.
- To integrate with JavaScript frameworks (React, Vue, Svelte) for reactive theming.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
// Set a custom property on :root (global)
document.documentElement.style.setProperty('--primary', '#ff5733');

// Set a custom property on a specific element (scoped)
element.style.setProperty('--primary', '#ff5733');

// Read a computed custom property value
const value = getComputedStyle(document.documentElement)
    .getPropertyValue('--primary')
    .trim();

// Remove an inline override
document.documentElement.style.removeProperty('--primary');
```

#### Component Breakdown

| Method / API | Description | Returns |
|---|---|---|
| `style.setProperty()` | Sets a CSS property on the element's inline style. | `undefined` |
| `style.removeProperty()` | Removes an inline property, restoring the stylesheet value. | `undefined` |
| `getComputedStyle()` | Returns the computed style object for an element. | `CSSStyleDeclaration` |
| `getPropertyValue()` | Returns the computed value of the given property. | `string` |
| `element.dataset` | Accesses `data-*` attributes. | `DOMStringMap` |

#### Syntax Rules

1. `setProperty()` on `document.documentElement` sets a global override via inline style.
2. Inline styles have higher specificity than stylesheet rules (except `!important`).
3. `getComputedStyle()` returns the resolved value after cascade, inheritance, and calculation.
4. Always `.trim()` the result of `getPropertyValue()` to remove leading whitespace.
5. Custom property names in `setProperty()` must include the `--` prefix.
6. `removeProperty()` restores the stylesheet value; use it to revert temporary overrides.
7. Scoped overrides on a specific element affect only that element's subtree.

#### Constraints and Limitations

- **Inline style specificity** — `setProperty()` creates an inline style that wins over stylesheet rules. Use `removeProperty()` to revert.
- **No transition on custom properties by default** — unregistered custom properties do not animate; use `@property` to register them.
- **Performance** — setting a custom property on `:root` triggers style recalculation for all elements that use it; use scoped overrides where possible.
- **Framework integration** — frameworks like Vue and Svelte provide reactive bindings (`v-bind`, `useCssVar`) that avoid manual DOM manipulation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Live Colour Picker with `setProperty()` and `getComputedStyle()`

**HTML File (`runtime-theme.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Runtime Theme Mutations</title>
    <link rel="stylesheet" href="runtime-theme.css">
</head>
<body>
    <div class="controls">
        <label>
            Primary Colour:
            <input type="color" id="color-picker" value="#3b82f6">
        </label>
        <button id="reset-btn">Reset</button>
    </div>

    <div class="card">
        <h2>Live Theme Editor</h2>
        <p>Change the colour above to see the theme update in real time.</p>
        <button class="btn">Primary Action</button>
    </div>

    <script>
        const picker = document.getElementById('color-picker');
        const resetBtn = document.getElementById('reset-btn');
        const root = document.documentElement;

        // Read the initial computed value
        const initialColor = getComputedStyle(root)
            .getPropertyValue('--color-primary')
            .trim();
        console.log('Initial --color-primary:', initialColor);

        // Update the theme on colour change
        picker.addEventListener('input', (e) => {
            root.style.setProperty('--color-primary', e.target.value);
            const current = getComputedStyle(root)
                .getPropertyValue('--color-primary')
                .trim();
            console.log('Updated --color-primary:', current);
        });

        // Reset to the stylesheet value
        resetBtn.addEventListener('click', () => {
            root.style.removeProperty('--color-primary');
            picker.value = '#3b82f6';
            console.log('Reset --color-primary:', getComputedStyle(root)
                .getPropertyValue('--color-primary').trim());
        });
    </script>
</body>
</html>
```

**CSS File (`runtime-theme.css`):**

```css
:root {
    --color-primary: #3b82f6;
    --color-bg: #f9fafb;
    --color-surface: #ffffff;
    --color-text: #111827;
}

body {
    font-family: system-ui, sans-serif;
    background-color: var(--color-bg);
    color: var(--color-text);
    margin: 0;
    padding: 40px;
}

.controls {
    display: flex;
    gap: 20px;
    align-items: center;
    margin-bottom: 30px;
}

.controls label {
    display: flex;
    align-items: center;
    gap: 10px;
    font-weight: bold;
}

.controls input[type="color"] {
    width: 50px;
    height: 40px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}

.controls button {
    padding: 10px 20px;
    background-color: var(--color-primary);
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-weight: bold;
}

.card {
    background-color: var(--color-surface);
    border: 1px solid #e5e7eb;
    border-radius: 12px;
    padding: 30px;
    max-width: 400px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card h2 {
    margin-top: 0;
}

.btn {
    background-color: var(--color-primary);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 8px;
    cursor: pointer;
    font-weight: bold;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `runtime-theme.html` and CSS as `runtime-theme.css`.
2. Open in a browser.
3. Open DevTools → Console to see the logged values.
4. Use the colour picker to change the primary colour. The button and the reset button update in real time.
5. Click "Reset" to restore the original colour.

**Expected Output:** A page with a live colour picker. Changing the colour updates the primary colour token, which instantly updates the button and the reset button. The console logs the computed values before, during, and after the change.

**Why This Works:** The `setProperty()` call writes an inline style on the `<html>` element, overriding the stylesheet value. Because every element that uses `var(--color-primary)` reads the computed value, they all update instantly. The `getComputedStyle()` call reads the resolved value, confirming the change. The `removeProperty()` call reverts the inline override, restoring the stylesheet default.

---

### Real-World Cases

- **Theme editors:** Live colour pickers that let users customise the interface.
- **Data visualisation:** Changing chart colours based on data values.
- **Adaptive UI:** Adjusting theme colours based on ambient light sensors or time of day.
- **Framework integration:** React, Vue, and Svelte use these APIs under the hood for reactive theming.

---

## References

- MDN Web Docs — `prefers-color-scheme` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-color-scheme
- MDN Web Docs — `light-dark()` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/color_value/light-dark
- MDN Web Docs — `color-scheme` - https://developer.mozilla.org/en-US/docs/Web/CSS/color-scheme
- MDN Web Docs — Using CSS custom properties - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties
- MDN Web Docs — `CSSStyleDeclaration.setProperty()` - https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty
- MDN Web Docs — `Window.getComputedStyle()` - https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle
- W3C — CSS Color Module Level 5 - https://www.w3.org/TR/css-color-5/
- W3C — CSS Custom Properties for Cascading Variables Module Level 1 - https://www.w3.org/TR/css-variables-1/
- web.dev — CSS colour-scheme dependent colours with `light-dark()` - https://web.dev/articles/light-dark
- web.dev — `prefers-color-scheme`: Hello darkness, my old friend - https://web.dev/articles/prefers-color-scheme
- CoreUI — How to implement dark mode in React - https://coreui.io/answers/how-to-implement-dark-mode-in-react/
- Virtocommerce — Managing Application Themes with `useTheme` - https://docs.virtocommerce.org/platform/developer-guide/1.0/custom-apps-development/vc-shell/Essentials/Usage-Guides/managing-themes-with-usetheme/
- Vue School — How to Update `:root` CSS Variables with JavaScript in Vue - https://vueschool.io/articles/vuejs-tutorials/how-to-update-root-css-variable-with-javascript/
- WordPress Gutenberg — UI Guidelines: Custom Properties and Disabled State - https://github.com/WordPress/gutenberg/pull/75912/files
- Can I Use — `prefers-color-scheme` - https://caniuse.com/prefers-color-scheme
- Can I Use — `light-dark()` - https://caniuse.com/mdn-css_types_color_light-dark