# CSS Design Tokens — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** Design Tokens are named entities that store design decisions — such as colours, typography, spacing, and motion values — in a platform-agnostic format. They serve as a single source of truth between design tools and development code, enabling consistent theming and scalable UI architecture.

**Technical Definition:** Design tokens are a methodology for expressing design decisions in a platform-agnostic way so that they can be shared across different disciplines, tools, and technologies. The W3C Design Tokens Community Group (DTCG) Format Module 2025.10 defines a vendor-neutral JSON file format for exchanging design tokens, using `$`-prefixed reserved properties (`$value`, `$type`, `$description`, `$deprecated`) and curly-brace alias syntax for token references. In CSS, design tokens are typically implemented as CSS Custom Properties (variables prefixed with `--`) that can be consumed via the `var()` function, with registration via `@property` for type-safe, animatable tokens.

**Beginner-Friendly Explanation:** A design token is like a labelled ingredient in a recipe book. Instead of writing "#3498db" everywhere in your CSS, you write "--color-primary". If you later decide to change the brand colour, you change it in one place, and every element that uses that token updates automatically. Tokens make design decisions reusable, consistent, and easy to change — whether you are switching to dark mode, rebranding, or scaling to a new platform.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Single source of truth** | One definition drives all platforms (web, iOS, Android). |
| **Three-tier hierarchy** | Primitives → Semantic → Component tokens. |
| **Platform-agnostic** | Stored in W3C DTCG JSON format, compiled to CSS, Swift, Android XML, etc. |
| **Themeable** | Swapping semantic tokens enables dark mode and multi-brand support. |
| **Type-safe** | `@property` adds syntax checking, initial values, and inheritance control. |
| **Computable** | `calc()`, `clamp()`, `min()`, and `max()` enable dynamic scaling. |
| **Tooling-friendly** | Style Dictionary, Tokens Studio, and Figma plugins automate the pipeline. |

---

### Prerequisites

- **CSS Custom Properties** — declaration, `var()`, and `@property`.
- **CSS Functions** — `calc()`, `clamp()`, `min()`, `max()`.
- **JSON Basics** — for DTCG token files.
- **Build Tooling** — Style Dictionary or similar token transformation pipelines.
- **Design Tool Awareness** — Figma Variables and Styles.

---

### Related Programming Areas

- **Design Systems** — tokens are the foundation of any scalable design system.
- **Theming** — dark mode, multi-brand, and accessibility variants.
- **CSS Architecture** — Feature-Sliced Design, ITCSS, and other methodologies.
- **Build Pipelines** — Style Dictionary, Tokens Studio, and Figma plugins.
- **Component Libraries** — components consume semantic tokens, not raw values.

---

### Core Concepts / Features

1. Primitive vs. Semantic Tokens
2. Core Token Systems: Colours, Typography, Spacing, Radii, Shadows, and Breakpoints
3. Token Math and Processing: `calc()`, `clamp()`, `min()`, and `max()`
4. Abstracting Frameworks: Figma to CSS Custom Properties

---

## 1. Primitive vs. Semantic Tokens: Structuring Scale-Based Choices vs. Context-Based Roles

### Definitions

**Core Definition:** Primitive tokens are raw, context-free design values (e.g., `blue-500: #3b82f6`). Semantic tokens are purpose-based references that give primitives contextual meaning (e.g., `color-primary: {blue-500}`). The two-layer system allows themes to swap semantic tokens while primitives remain constant.

**Technical Definition:** Primitives define the raw design palette with no semantic meaning attached. They answer "what options exist?" and form the foundation that semantic tokens reference. Semantic tokens assign meaning to primitives. They answer "what does this value mean?" and are the tokens that components should consume. A third optional layer — component tokens — answers "where is this used?" (e.g., `button-background: {color-primary}`). Most production systems operate well with just primitives and semantic tokens; component tokens multiply the token count significantly — a system with 200 semantic tokens might balloon to 2000+ with component tokens.

**Beginner-Friendly Explanation:** Think of primitive tokens as the paint swatches in a hardware store — they have names like "sky blue" and "forest green" but no opinion about where they should be used. Semantic tokens are the room labels — "living room wall," "kitchen cabinet" — that reference the swatches and give them purpose. When you repaint the living room, you change the semantic token, and every wall that references it updates. The paint swatches themselves never change.

---

### Purposes

- To separate raw design values from their contextual usage.
- To enable theming (dark mode, multi-brand) by swapping the semantic layer.
- To provide a maintainable, scalable token architecture.
- To create a shared vocabulary between designers and developers.
- To prevent hardcoded values from leaking into component styles.

---

### Syntax Rules and Structure

#### Complete General Syntax (DTCG JSON)

```json
{
  "primitive": {
    "color": {
      "blue": {
        "100": { "$value": "#dbeafe", "$type": "color" },
        "500": { "$value": "#3b82f6", "$type": "color" },
        "900": { "$value": "#1e3a8a", "$type": "color" }
      },
      "gray": {
        "50":  { "$value": "#f9fafb", "$type": "color" },
        "900": { "$value": "#111827", "$type": "color" }
      }
    }
  },
  "semantic": {
    "color": {
      "text": {
        "primary":   { "$value": "{primitive.color.gray.900}", "$type": "color" },
        "secondary": { "$value": "{primitive.color.gray.600}", "$type": "color" }
      },
      "background": {
        "default": { "$value": "{primitive.color.gray.50}", "$type": "color" }
      },
      "action": {
        "primary": { "$value": "{primitive.color.blue.500}", "$type": "color" }
      }
    }
  }
}
```

#### Compiled CSS Custom Properties

```css
:root {
  /* Primitives */
  --color-blue-100: #dbeafe;
  --color-blue-500: #3b82f6;
  --color-blue-900: #1e3a8a;
  --color-gray-50:  #f9fafb;
  --color-gray-900: #111827;

  /* Semantic tokens reference primitives */
  --color-text-primary:   var(--color-gray-900);
  --color-text-secondary: var(--color-gray-600);
  --color-bg-default:     var(--color-gray-50);
  --color-action-primary: var(--color-blue-500);
}

/* Dark theme overrides only the semantic layer */
[data-theme="dark"] {
  --color-text-primary:   var(--color-gray-50);
  --color-text-secondary: var(--color-gray-300);
  --color-bg-default:     var(--color-gray-900);
}
```

#### Component Breakdown

| Layer | Also Called | Purpose | Example |
|---|---|---|---|
| Primitive | Core, Base, Global | Raw values, context-free | `blue-500: #3b82f6` |
| Semantic | Alias, Purpose, Role | Meaning-based references | `color-primary: {blue-500}` |
| Component | Specific, Local | Component-scoped tokens | `button-bg: {color-primary}` |

#### Syntax Rules

1. Primitive token names describe **what the value is** (appearance-based: `blue-500`, `gray-50`).
2. Semantic token names describe **what the value means** (purpose-based: `color-text-primary`, `color-action-primary`).
3. Semantic tokens **always reference primitives** via DTCG alias syntax `{path.to.token}` or CSS `var()`.
4. Component tokens are optional and should only be created for complex, stateful components.
5. Theming works by **swapping the semantic layer** while primitives stay constant.
6. Always use semantic tokens in component styles. If a semantic token does not exist for your use case, create one rather than reaching for a primitive or hardcoded value.

#### Constraints and Limitations

- **Token count explosion** — component tokens can multiply token count by 10×; use them sparingly.
- **Naming drift** — inconsistent naming conventions across teams cause confusion; establish a convention early.
- **Primitive leakage** — components that reference primitives directly break theming.
- **No CSS-spec enforcement** — the three-tier hierarchy is a codebase convention, not a CSS-spec-enforced structure.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Two-Layer Token System with Dark Mode

**HTML File (`tokens.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Design Tokens</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="tokens.css">
</head>
<body>
    <!-- Toggle button for dark mode -->
    <button class="theme-toggle" onclick="document.documentElement.toggleAttribute('data-theme-dark')">
        Toggle Dark Mode
    </button>

    <div class="card">
        <h2>Design Token Card</h2>
        <p>This card uses semantic tokens for all colours.</p>
        <button class="btn">Primary Action</button>
    </div>
</body>
</html>
```

**CSS File (`tokens.css`):**

```css
/* ===== PRIMITIVE TOKENS ===== */
:root {
  /* Raw colour palette — never referenced directly in components */
  --color-blue-100: #dbeafe;
  --color-blue-500: #3b82f6;
  --color-blue-900: #1e3a8a;
  --color-gray-50:  #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-300: #d1d5db;
  --color-gray-600: #4b5563;
  --color-gray-900: #111827;
  --color-white:    #ffffff;
}

/* ===== SEMANTIC TOKENS (LIGHT THEME) ===== */
:root {
  --color-text-primary:   var(--color-gray-900);
  --color-text-secondary: var(--color-gray-600);
  --color-bg-default:     var(--color-gray-50);
  --color-bg-surface:     var(--color-white);
  --color-border-default: var(--color-gray-300);
  --color-action-primary: var(--color-blue-500);
  --color-action-text:    var(--color-white);
}

/* ===== SEMANTIC TOKENS (DARK THEME) ===== */
[data-theme-dark] {
  --color-text-primary:   var(--color-gray-50);
  --color-text-secondary: var(--color-gray-300);
  --color-bg-default:     var(--color-gray-900);
  --color-bg-surface:     var(--color-gray-800, #1f2937);
  --color-border-default: var(--color-gray-600);
  --color-action-primary: var(--color-blue-500);
  --color-action-text:    var(--color-white);
}

/* ===== COMPONENT STYLES (consume semantic tokens only) ===== */
body {
    font-family: system-ui, sans-serif;
    background-color: var(--color-bg-default);
    color: var(--color-text-primary);
    transition: background-color 300ms, color 300ms;
    padding: 40px;
}

.card {
    background-color: var(--color-bg-surface);
    border: 1px solid var(--color-border-default);
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
}

.btn {
    background-color: var(--color-action-primary);
    color: var(--color-action-text);
    border: none;
    padding: 12px 24px;
    border-radius: 8px;
    cursor: pointer;
    font-weight: bold;
}

.theme-toggle {
    padding: 10px 20px;
    margin-bottom: 20px;
    background-color: var(--color-action-primary);
    color: var(--color-action-text);
    border: none;
    border-radius: 8px;
    cursor: pointer;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `tokens.html`.
3. Save the CSS code as `tokens.css` in the same folder.
4. Open `tokens.html` in a web browser.
5. Click "Toggle Dark Mode" to see the semantic tokens swap while primitives stay constant.

**Expected Output:** A card that switches between light and dark themes. The primitives (`--color-blue-500`, `--color-gray-900`) never change; only the semantic tokens (`--color-text-primary`, `--color-bg-default`) are reassigned.

**Why This Works:** The primitive tokens define the raw palette. The semantic tokens in `:root` map them to light-theme roles. The `[data-theme-dark]` selector overrides only the semantic tokens, remapping them to dark-theme roles. Components reference only semantic tokens, so they adapt automatically.

---

### Real-World Cases

- **Multi-brand platforms:** Swapping semantic tokens for different brands while sharing primitives.
- **Dark mode:** Overriding semantic colour tokens in a `[data-theme="dark"]` selector.
- **Accessibility themes:** High-contrast semantic tokens for users with low vision.
- **Component libraries:** Components consume semantic tokens, ensuring consistent theming.

---

## 2. Core Token Systems: Architecting Standardized Scales

### Definitions

**Core Definition:** Core token systems are the organized collections of tokens for each design property — colours, typography, spacing, border radii, elevation shadows, and layout breakpoints — each following a consistent scale and naming convention.

**Technical Definition:** A token system typically organizes tokens into categories (colour, typography, spacing, radius, shadow, breakpoint), each with its own scale. The colour system uses a neutral ramp plus accent ramps plus semantic bases. Spacing typically follows a consistent increment grid (4pt or 8pt). Typography uses a modular scale ratio (1.25× is common). Elevation shadows are defined as a set of layered box-shadows. Breakpoints define the viewport widths at which layout changes occur. Each category is exposed as CSS custom properties with a consistent naming convention.

**Beginner-Friendly Explanation:** A token system is like a well-organized toolbox. Colours have their own drawer, typography has another, spacing has another. Each drawer follows a consistent scale — colours might go from 50 (lightest) to 900 (darkest), spacing might go from 4px to 64px in consistent steps. This organization makes it easy to find the right token and ensures consistency across the entire design system.

---

### Purposes

- To provide a consistent scale for each design property.
- To enable predictable, maintainable design decisions.
- To create a shared vocabulary between designers and developers.
- To support theming and multi-brand scenarios.
- To simplify the process of adding new components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
:root {
    /* ---- Colour Tokens ---- */
    --color-neutral-50:  #f9fafb;
    --color-neutral-100: #f3f4f6;
    --color-neutral-500: #6b7280;
    --color-neutral-900: #111827;
    --color-brand-500:   #3b82f6;
    --color-brand-600:   #2563eb;
    --color-danger-500:  #ef4444;
    --color-success-500: #22c55e;

    /* ---- Spacing Scale (8pt grid) ---- */
    --space-1: 0.25rem;  /* 4px */
    --space-2: 0.5rem;   /* 8px */
    --space-3: 0.75rem;  /* 12px */
    --space-4: 1rem;     /* 16px */
    --space-6: 1.5rem;   /* 24px */
    --space-8: 2rem;     /* 32px */
    --space-12: 3rem;    /* 48px */
    --space-16: 4rem;    /* 64px */

    /* ---- Typography Scale (1.25× ratio) ---- */
    --font-size-xs:   0.75rem;   /* 12px */
    --font-size-sm:   0.875rem;  /* 14px */
    --font-size-base: 1rem;      /* 16px */
    --font-size-lg:   1.25rem;   /* 20px */
    --font-size-xl:   1.5625rem; /* 25px */
    --font-size-2xl:  1.953rem;  /* 31px */
    --font-size-3xl:  2.441rem;  /* 39px */

    --font-weight-regular: 400;
    --font-weight-medium:  500;
    --font-weight-bold:    700;

    --font-family-sans: system-ui, -apple-system, sans-serif;
    --font-family-mono: "Fira Code", "Courier New", monospace;

    /* ---- Border Radius Tokens ---- */
    --radius-sm:   4px;
    --radius-md:   8px;
    --radius-lg:   12px;
    --radius-xl:   20px;
    --radius-full: 9999px;

    /* ---- Elevation Shadows ---- */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
    --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
    --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.15);

    /* ---- Breakpoints (as custom properties for reference) ---- */
    --breakpoint-sm: 40em;   /* 640px */
    --breakpoint-md: 48em;   /* 768px */
    --breakpoint-lg: 64em;   /* 1024px */
    --breakpoint-xl: 80em;   /* 1280px */
}
```

#### Component Breakdown

| Category | Scale | Naming Pattern | Example |
|---|---|---|---|
| Colour | 50–900 (10 steps per hue) | `--color-{hue}-{weight}` | `--color-brand-500` |
| Spacing | 4px–64px (8pt grid) | `--space-{n}` | `--space-4` |
| Typography | 1.25× modular ratio | `--font-size-{size}` | `--font-size-lg` |
| Radius | 4px–9999px | `--radius-{size}` | `--radius-md` |
| Shadow | 4 elevation levels | `--shadow-{size}` | `--shadow-lg` |
| Breakpoint | 640px–1280px | `--breakpoint-{size}` | `--breakpoint-lg` |

#### Syntax Rules

1. Colour scales typically use 50–900 with 100-point increments (50, 100, 200, … 900).
2. Spacing scales should follow a consistent increment (4pt or 8pt grid).
3. Typography scales use a modular ratio — 1.25× (major third) is common.
4. Radius tokens should cover the full range from subtle (`4px`) to pill (`9999px`).
5. Shadow tokens should provide 3–5 elevation levels.
6. Breakpoints are declared as custom properties for reference but used in media queries as literal values.
7. All tokens use the `--` prefix and kebab-case naming.

#### Constraints and Limitations

- **Token proliferation** — too many tokens (especially component tokens) can become unmanageable.
- **Scale rigidity** — an overly rigid scale may not accommodate all design needs.
- **Breakpoint limitations** — custom properties cannot be used inside `@media` conditions; breakpoints must be literal values in media queries.
- **Naming consistency** — inconsistent naming across categories causes confusion.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Using Spacing and Radius Tokens in a Card

**HTML File (`core-tokens.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Core Token Systems</title>
    <link rel="stylesheet" href="core-tokens.css">
</head>
<body>
    <div class="card">
        <h2>Token-Driven Card</h2>
        <p>Spacing, radius, shadow, and typography all use tokens.</p>
        <button class="btn">Action</button>
    </div>
</body>
</html>
```

**CSS File (`core-tokens.css`):**

```css
:root {
    /* Colour tokens */
    --color-brand-500: #3b82f6;
    --color-brand-600: #2563eb;
    --color-neutral-50: #f9fafb;
    --color-neutral-100: #f3f4f6;
    --color-neutral-300: #d1d5db;
    --color-neutral-600: #4b5563;
    --color-neutral-900: #111827;
    --color-white: #ffffff;

    /* Spacing tokens */
    --space-1: 0.25rem;
    --space-2: 0.5rem;
    --space-3: 0.75rem;
    --space-4: 1rem;
    --space-6: 1.5rem;
    --space-8: 2rem;
    --space-12: 3rem;

    /* Typography tokens */
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-xl: 1.5625rem;
    --font-weight-bold: 700;

    /* Radius tokens */
    --radius-md: 8px;
    --radius-lg: 12px;

    /* Shadow tokens */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
}

body {
    font-family: system-ui, sans-serif;
    background-color: var(--color-neutral-50);
    padding: var(--space-12);
}

.card {
    background-color: var(--color-white);
    border-radius: var(--radius-lg);
    padding: var(--space-8);
    box-shadow: var(--shadow-lg);
    max-width: 400px;
    border: 1px solid var(--color-neutral-100);
}

.card h2 {
    font-size: var(--font-size-xl);
    font-weight: var(--font-weight-bold);
    color: var(--color-neutral-900);
    margin-top: 0;
    margin-bottom: var(--space-4);
}

.card p {
    font-size: var(--font-size-base);
    color: var(--color-neutral-600);
    margin-bottom: var(--space-6);
}

.btn {
    background-color: var(--color-brand-500);
    color: var(--color-white);
    border: none;
    padding: var(--space-3) var(--space-6);
    border-radius: var(--radius-md);
    font-size: var(--font-size-sm);
    font-weight: var(--font-weight-bold);
    cursor: pointer;
    transition: background-color 200ms;
}

.btn:hover {
    background-color: var(--color-brand-600);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `core-tokens.html` and CSS as `core-tokens.css`.
2. Open in a browser.
3. Observe the card using tokens for every visual property.

**Expected Output:** A card with consistent spacing, typography, radius, and shadow — all driven by tokens.

**Why This Works:** Every visual property in the card uses a token. The spacing tokens provide consistent increments. The colour tokens ensure consistency. The radius and shadow tokens define the card's visual style. Changing a token value updates the card and every other component that uses it.

---

### Real-World Cases

- **Design systems:** A shared token set across multiple applications.
- **Multi-brand platforms:** Different semantic mappings for different brands.
- **Theming:** Dark mode, high contrast, and accessibility variants.
- **Component libraries:** Components that consume tokens for consistent styling.

---

## 3. Token Math & Processing: Scaling Tokens Dynamically

### Definitions

**Core Definition:** Token math is the use of CSS math functions — `calc()`, `clamp()`, `min()`, and `max()` — to compute dynamic token values that scale with the viewport, container, or other tokens.

**Technical Definition:** Token math enables fluid, responsive design by computing values at runtime rather than hardcoding them. The `calc()` function performs arithmetic with mixed units (e.g., `calc(var(--space-4) * 2)`). The `clamp(min, preferred, max)` function constrains a value to a range, enabling fluid typography that scales linearly between viewport sizes without breakpoints. The preferred value is typically a `calc()` expression mixing a fixed unit with a viewport unit. The `min()` and `max()` functions provide single-bound constraints. These functions can be used with custom properties to create dynamic token systems.

**Beginner-Friendly Explanation:** Token math lets your tokens adapt to the screen size. Instead of defining a font size as "24px," you can define it as `clamp(1.25rem, 1rem + 1vw, 2rem)`, which means "at least 1.25rem, at most 2rem, and otherwise scale with the viewport." This creates smooth, fluid scaling without media queries. You can also use `calc()` to derive tokens from other tokens — for example, a "large spacing" token that is always twice the "base spacing" token.

---

### Purposes

- To create fluid tokens that scale with the viewport.
- To derive tokens from other tokens (e.g., `--space-lg: calc(var(--space-base) * 2)`).
- To eliminate the need for per-breakpoint overrides.
- To ensure consistent scaling across typography, spacing, and layout.
- To build dynamic, responsive design systems with fewer tokens.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
:root {
    /* calc() arithmetic */
    --space-base: 1rem;
    --space-lg: calc(var(--space-base) * 1.5);
    --space-xl: calc(var(--space-base) * 2);

    /* clamp() for fluid typography */
    --font-size-base: clamp(1rem, 0.9rem + 0.5vw, 1.25rem);
    --font-size-h1: clamp(2rem, 1.5rem + 2.5vw, 4rem);

    /* min() and max() */
    --container-width: min(100% - 2rem, 1200px);
    --sidebar-width: max(200px, 20vw);
}
```

#### Component Breakdown

| Function | Description | Example |
|---|---|---|
| `calc()` | Arithmetic with mixed units. | `calc(var(--space-4) * 2)` |
| `clamp(min, preferred, max)` | Constrains a value to a range. | `clamp(1rem, 0.9rem + 0.5vw, 1.25rem)` |
| `min(a, b, …)` | Returns the smallest value. | `min(100% - 2rem, 1200px)` |
| `max(a, b, …)` | Returns the largest value. | `max(200px, 20vw)` |

#### Fluid Typography Formula

The preferred value in `clamp()` is computed using linear interpolation:

```
preferred = calc(Vmin + (Vmax - Vmin) * ((100vw - BPmin) / (BPmax - BPmin)))
```

Where:
- `Vmin` = minimum font size (at `BPmin`)
- `Vmax` = maximum font size (at `BPmax`)
- `BPmin` = minimum viewport width (e.g., 360px)
- `BPmax` = maximum viewport width (e.g., 1440px)

#### Syntax Rules

1. `calc()` requires spaces around `+` and `-` operators.
2. `clamp()` takes exactly three arguments: minimum, preferred, maximum.
3. The preferred value in `clamp()` is typically a `calc()` expression.
4. `min()` and `max()` accept two or more comma-separated values.
5. Token math can reference other tokens via `var()`.
6. Fluid typography should use `rem` units for the minimum and maximum to respect user font-size preferences.
7. `calc()` can be used with `clamp()`, `min()`, and `max()` in any combination.

#### Constraints and Limitations

- **Performance** — excessive `calc()` and `clamp()` usage can increase style calculation time.
- **Debugging complexity** — computed values are not visible in the source; use DevTools to inspect.
- **Browser support** — `clamp()` is Baseline widely available; `min()` and `max()` are also well-supported.
- **Nested functions** — deeply nested `calc()` expressions become hard to read; consider intermediate tokens.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fluid Typography and Spacing Tokens

**HTML File (`token-math.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Token Math</title>
    <link rel="stylesheet" href="token-math.css">
</head>
<body>
    <div class="container">
        <h1>Fluid Typography</h1>
        <p>This heading and paragraph scale fluidly with the viewport using clamp() tokens.</p>
        <div class="card">A card with fluid padding.</div>
    </div>
</body>
</html>
```

**CSS File (`token-math.css`):**

```css
:root {
    /* Base spacing unit */
    --space-unit: 0.25rem;

    /* Derived spacing tokens using calc() */
    --space-1: calc(var(--space-unit) * 1);   /* 4px */
    --space-2: calc(var(--space-unit) * 2);   /* 8px */
    --space-4: calc(var(--space-unit) * 4);   /* 16px */
    --space-6: calc(var(--space-unit) * 6);   /* 24px */
    --space-8: calc(var(--space-unit) * 8);   /* 32px */
    --space-12: calc(var(--space-unit) * 12); /* 48px */

    /* Fluid spacing tokens using clamp() */
    --space-fluid-sm: clamp(var(--space-2), 1vw, var(--space-4));
    --space-fluid-md: clamp(var(--space-4), 2vw, var(--space-8));
    --space-fluid-lg: clamp(var(--space-6), 3vw, var(--space-12));

    /* Fluid typography tokens */
    --font-size-body: clamp(1rem, 0.9rem + 0.5vw, 1.25rem);
    --font-size-h1: clamp(2rem, 1.5rem + 2.5vw, 4rem);
    --font-size-h2: clamp(1.5rem, 1.25rem + 1.25vw, 2.5rem);

    /* Fluid container width */
    --container-max: min(100% - var(--space-8), 1200px);
}

body {
    font-family: system-ui, sans-serif;
    background-color: #f5f5f5;
    padding: var(--space-fluid-md);
}

.container {
    max-width: var(--container-max);
    margin: 0 auto;
}

h1 {
    font-size: var(--font-size-h1);
    margin-bottom: var(--space-fluid-md);
}

p {
    font-size: var(--font-size-body);
    line-height: 1.6;
    margin-bottom: var(--space-fluid-lg);
}

.card {
    background-color: white;
    border-radius: 12px;
    padding: var(--space-fluid-md);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    font-size: var(--font-size-body);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `token-math.html` and CSS as `token-math.css`.
2. Open in a browser.
3. Resize the browser window. Observe the typography and spacing scale smoothly.

**Expected Output:** A page with fluid typography and spacing that scales smoothly between mobile and desktop sizes without any media queries.

**Why This Works:** The `calc()` function derives spacing tokens from a base unit. The `clamp()` function creates fluid tokens that scale between a minimum and maximum based on the viewport. The `min()` function constrains the container width. All tokens are computed at runtime, creating a fully fluid, responsive design system.

---

### Real-World Cases

- **Fluid typography systems:** Type scales that scale smoothly from mobile to desktop.
- **Responsive spacing:** Padding and margin that adapt to the viewport.
- **Container widths:** `min(100% - 2rem, 1200px)` for content containers.
- **Sidebar widths:** `max(200px, 20vw)` for sidebars that never get too narrow.

---

## 4. Abstracting Frameworks: Mapping Figma Design Token JSON Exports into CSS Custom Properties

### Definitions

**Core Definition:** Abstracting frameworks is the process of exporting design tokens from Figma (or another design tool) as DTCG JSON, transforming them through a build pipeline (e.g., Style Dictionary), and generating CSS custom properties that can be imported into a web project.

**Technical Definition:** Figma Variables (colours, numbers, strings, booleans) can be exported to W3C DTCG JSON format, then transformed via Style Dictionary into platform outputs (CSS custom properties, Tailwind theme, iOS/Android tokens). The W3C DTCG Format Module 2025.10 standardises the JSON structure using `$value`, `$type`, and `$description` properties, with alias syntax `{path.to.token}` for token references. Figma plugins such as Variables Exporter and Tokens Exporter generate `:root` CSS custom properties, with aliases automatically resolved to `var(--referenced-name)` and multiple modes exported as `[data-theme="mode-name"]` selectors.

**Beginner-Friendly Explanation:** You design your colour palette, spacing scale, and typography in Figma. Instead of manually copying values into CSS, you export them as a JSON file. A build tool reads that JSON and generates a CSS file full of custom properties. You import that CSS file into your project, and all your tokens are available. When the design changes in Figma, you re-export and re-run the build — the CSS updates automatically.

---

### Purposes

- To maintain a single source of truth between design and code.
- To automate the design-to-development handoff.
- To ensure consistency across platforms (web, iOS, Android).
- To enable rapid theming and multi-brand support.
- To reduce manual errors in token transcription.

---

### Syntax Rules and Structure

#### Complete General Syntax (Pipeline)

```
Figma Variables/Styles
        ↓  (export)
DTCG JSON (tokens.json)
        ↓  (Style Dictionary transform)
CSS Custom Properties (tokens.css)
        ↓  (import)
Web Project
```

#### Step 1: Export Figma Variables to DTCG JSON

```json
{
  "color": {
    "brand": {
      "500": {
        "$value": "#3b82f6",
        "$type": "color",
        "$description": "Primary brand colour"
      }
    },
    "surface": {
      "default": {
        "$value": "{color.neutral.50}",
        "$type": "color"
      }
    }
  },
  "spacing": {
    "4": {
      "$value": "16px",
      "$type": "dimension"
    }
  }
}
```

#### Step 2: Style Dictionary Configuration

```javascript
// sd.config.js
export default {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'dist/',
      files: [{
        destination: 'tokens.css',
        format: 'css/variables',
        options: {
          outputReferences: true // Preserve var() aliases
        }
      }]
    }
  }
};
```

#### Step 3: Generated CSS Output

```css
:root {
  --color-brand-500: #3b82f6;
  --color-surface-default: var(--color-neutral-50);
  --spacing-4: 16px;
}
```

#### Syntax Rules

1. Figma Variables must be exported in W3C DTCG JSON format.
2. Style Dictionary v4+ supports the DTCG format natively.
3. The `outputReferences: true` option preserves alias references as `var()` calls.
4. Multiple Figma modes (light/dark) export as separate selectors: `[data-theme="dark"]`.
5. Figma plugins such as Variables Exporter automate the JSON export.
6. The generated CSS file should be imported before any component styles.
7. Tokens should be namespaced (e.g., `--ds-color-primary`) to avoid collisions.

#### Constraints and Limitations

- **Build tooling required** — a Node.js pipeline with Style Dictionary is necessary.
- **Figma API limitations** — some token types (e.g., gradients) may not export cleanly.
- **Mode mapping** — Figma modes must be mapped to CSS selectors (e.g., `[data-theme="dark"]`).
- **Version drift** — tokens must be re-exported and rebuilt when designs change.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Figma to CSS Pipeline

**Step 1: Export from Figma** (using Variables Exporter plugin)

The plugin generates a `tokens.json` file in DTCG format:

```json
{
  "color": {
    "brand": {
      "500": { "$value": "#3b82f6", "$type": "color" }
    },
    "text": {
      "primary": { "$value": "{color.neutral.900}", "$type": "color" }
    }
  },
  "spacing": {
    "4": { "$value": "16px", "$type": "dimension" }
  }
}
```

**Step 2: Style Dictionary Build**

```bash
npx style-dictionary build
```

**Step 3: Generated CSS (`dist/tokens.css`)**

```css
:root {
  --color-brand-500: #3b82f6;
  --color-text-primary: var(--color-neutral-900);
  --spacing-4: 16px;
}

[data-theme="dark"] {
  --color-text-primary: var(--color-neutral-50);
}
```

**Step 4: Import into Project**

```css
/* main.css */
@import 'dist/tokens.css';

.card {
  background: var(--color-surface-default);
  padding: var(--spacing-4);
  color: var(--color-text-primary);
}
```

**Step-by-Step Setup Guide:**

1. Install the Variables Exporter Figma plugin.
2. Export variables as DTCG JSON.
3. Install Style Dictionary: `npm install style-dictionary`.
4. Create `sd.config.js` with the CSS platform configuration.
5. Run `npx style-dictionary build` to generate `tokens.css`.
6. Import `tokens.css` into your project.

**Expected Output:** A CSS file full of custom properties that match your Figma design tokens, with aliases preserved as `var()` references and dark mode exported as a `[data-theme="dark"]` selector.

**Why This Works:** The plugin exports Figma variables in the standard DTCG JSON format. Style Dictionary reads the JSON, resolves aliases, and generates CSS custom properties. The `outputReferences: true` option ensures that aliases like `{color.neutral.900}` become `var(--color-neutral-900)` rather than being resolved to the raw hex value. This preserves the token hierarchy in the generated CSS.

---

### Real-World Cases

- **Enterprise design systems:** A single token source drives web, iOS, and Android.
- **Multi-brand platforms:** Different Figma modes generate different CSS themes.
- **Design-to-dev handoff:** Designers update Figma; developers re-run the build.
- **Component libraries:** Components consume generated tokens for consistent styling.

---

## References

- W3C Design Tokens Format Module 2025.10 - https://www.w3.org/community/reports/design-tokens/CG-FINAL-format-20251028/
- Design Tokens Community Group — Specification - https://www.designtokens.org/
- MDN Web Docs — Using CSS custom properties - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties
- MDN Web Docs — `@property` - https://developer.mozilla.org/en-US/docs/Web/CSS/@property
- MDN Web Docs — `clamp()` - https://developer.mozilla.org/en-US/docs/Web/CSS/clamp
- Style Dictionary — Official Documentation - https://styledictionary.com/
- Figma — Variables Exporter Plugin - https://www.figma.com/community/plugin/1609647189314085608/variables-exporter
- Design Tokens and Theming Architecture (sujeet.pro) - https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/design-tokens-and-theming/README.md
- Feature-Sliced Design — Design Tokens: The Foundation of Your UI Arch - https://feature-sliced.design/kr/blog/design-tokens-architecture
- Design System Skills — Design Tokens Structure - https://github.com/dylantarre/design-system-skills/blob/main/skills/tokens/design-tokens-structure/SKILL.md
- Open Library — Design Token Guide - https://docs.openlibrary.org/
- Compound Design System — Naming Approach - https://compound.thephoenixgroup.com/
- zeroheight — Exporting Design Tokens from Figma - https://zeroheight.com/
- Can I Use — CSS Custom Properties - https://caniuse.com/css-variables