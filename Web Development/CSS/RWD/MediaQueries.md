# CSS Media Queries & User Preference Features — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Media Queries are conditional CSS rules that apply styles based on the characteristics of the user's device, browser, or environment — such as viewport dimensions, screen resolution, input mechanism capabilities, and user-configured accessibility preferences. They form the conditional logic layer of responsive web design, allowing a single stylesheet to adapt its presentation across the full spectrum of devices and user needs.

**Technical Definition:** Media queries are defined in the Media Queries Level 3, 4, and 5 specifications. A media query is composed of an optional media type and any number of media feature expressions, which may be combined using logical operators (`and`, `not`, `only`, and comma-separated lists). Media types define the broad category of device (`all`, `print`, `screen`); media features describe specific characteristics of the user agent, output device, or environment. Media Queries Level 4 introduced range context syntax using mathematical comparison operators (`<`, `>`, `<=`, `>=`), while Level 5 extends the specification with additional user-preference media features such as `prefers-reduced-motion`, `prefers-contrast`, and `prefers-reduced-data`. Media queries are used in the `@media` and `@import` at-rules, the `media` attribute of HTML elements, and programmatically via `window.matchMedia()`.

**Beginner-Friendly Explanation:** A media query is like an "if" statement for CSS. You write a condition — "if the screen is wider than 768 pixels" or "if the user prefers dark mode" — and then you write the styles that should apply when that condition is true. This is how a website can look one way on a phone, another way on a laptop, and yet another way when printed. Media queries are the mechanism that makes responsive design possible, and in recent years they have expanded to cover much more than just screen size — they can detect whether a device supports hovering, how precise its pointer is, and whether the user has requested reduced motion or higher contrast.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Conditional application** | Styles are applied only when the media query condition evaluates to true. |
| **Composable logic** | Multiple conditions can be combined with `and`, `not`, `only`, and comma-separated (OR) lists. |
| **Range syntax** | Level 4 introduced mathematical comparison operators (`<`, `>`, `<=`, `>=`) for viewport dimensions. |
| **Device capability detection** | Features like `pointer`, `hover`, `orientation`, and `resolution` describe input and display characteristics. |
| **User preference awareness** | Features like `prefers-color-scheme`, `prefers-reduced-motion`, and `prefers-contrast` respect user-configured accessibility settings. |
| **Case-insensitive** | Media query syntax is case-insensitive. |
| **Multiple application contexts** | Media queries work in `@media`, `@import`, the `media` HTML attribute, and JavaScript's `matchMedia()`. |

---

### Prerequisites

Before studying CSS Media Queries, you should understand:

- **CSS Syntax** — at-rules, selectors, properties, and the cascade.
- **CSS Box Model and Layout** — how elements are sized and positioned.
- **Viewport concept** — the visible area of a web page in the browser window.
- **Responsive design principles** — fluid layouts and flexible dimensions.

---

### Related Programming Areas

- **Responsive Web Design** — media queries are the conditional layer of responsive layouts.
- **Accessibility** — user-preference media queries (`prefers-reduced-motion`, `prefers-contrast`) are essential for inclusive design.
- **Container Queries** — a newer complement to media queries that respond to container size rather than viewport size.
- **Core Web Vitals** — media queries affect layout stability (CLS) and rendering performance.
- **Progressive Enhancement** — mobile-first design uses `min-width` media queries to layer complexity upward.

---

### Core Concepts / Features

1. The `@media` Rule Syntax: Logical Operators and Comma-Separated Lists
2. Viewport Conditions: Modern Range Syntax vs. Legacy Properties
3. Device Capabilities: Orientation, Resolution, Pointer, and Hover
4. User Preference Media Features: `prefers-color-scheme`, `prefers-reduced-motion`, `prefers-contrast`

---

## 1. The `@media` Rule Syntax: Logical Operators and Comma-Separated Lists

### Definitions

**Core Definition:** The `@media` at-rule is the primary mechanism for applying conditional CSS based on media query conditions. It uses logical operators (`and`, `not`, `only`) and comma-separated lists to compose complex conditions.

**Technical Definition:** The `@media` CSS at-rule can be used to apply part of a style sheet based on the result of one or more media queries. The formal syntax is `@media <media-query-list> { <rule-list> }`. Each media query in the list is a `<media-condition>` composed of an optional media type and media feature expressions. The logical operators are: `and` (conjunction, evaluates to true if both operands are true), `not` (negation, evaluates to true if the operand is false), `only` (used to hide the media query from older browsers that do not support media features, preventing them from applying styles incorrectly), and the comma (which acts as a logical OR between media queries in a list). A media query is true if the media type matches the device and all media feature expressions are true. A media query list is true if any of its component media queries are true.

**Beginner-Friendly Explanation:** The `@media` rule works like a conditional block. You write `@media` followed by a condition in parentheses, and then a block of CSS. If the condition is true, the CSS inside the block is applied. You can combine multiple conditions with `and` — for example, "screen AND at least 768px wide." You can negate a condition with `not` — "NOT print." And you can provide multiple alternatives with commas — "screen OR print." The `only` keyword is a legacy safeguard that prevents older browsers from misinterpreting the query.

---

### Purposes

- To conditionally apply CSS rules based on device, viewport, or user-preference conditions.
- To compose complex conditions using logical operators (`and`, `not`, `only`).
- To provide multiple alternative conditions via comma-separated lists (logical OR).
- To ensure older browsers ignore unsupported media features safely (via `only`).
- To target specific media types such as `screen` and `print`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Single condition */
@media <media-type> and (<media-feature>) { ... }

/* Multiple conditions with AND */
@media <media-type> and (<feature-1>) and (<feature-2>) { ... }

/* Negation with NOT */
@media not <media-type> { ... }

/* Multiple alternatives with comma (OR) */
@media <media-type> and (<feature-1>), <media-type> and (<feature-2>) { ... }

/* Only keyword (legacy safeguard) */
@media only <media-type> and (<feature>) { ... }
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `@media` | The at-rule keyword. | `@media` |
| `<media-type>` | The device category: `all`, `print`, `screen`. | `screen` |
| `and` | Logical AND; both operands must be true. | `screen and (min-width: 768px)` |
| `not` | Logical NOT; negates the entire media query. | `not print` |
| `only` | Legacy safeguard; hides the query from old browsers. | `only screen` |
| `,` (comma) | Logical OR; applies if any query in the list is true. | `screen, print` |
| `(<media-feature>)` | A feature expression in parentheses. | `(min-width: 768px)` |

#### Syntax Rules

1. The media type is optional; if omitted, `all` is assumed (except when using `not` or `only`).
2. Media queries are case-insensitive.
3. The `and` operator combines a media type with a media feature, or two media features.
4. The `not` operator negates the entire media query, not just a single feature.
5. The `only` operator is a legacy safeguard and has no effect in modern browsers.
6. A comma-separated list acts as a logical OR: if any query is true, the styles apply.
7. An empty media query list (or one consisting solely of whitespace) is equivalent to `all`.

#### Constraints and Limitations

- **`not` applies to the entire query** — you cannot negate a single feature within a query using `not`.
- **`only` has no modern effect** — it is purely a legacy workaround for browsers that do not support media features.
- **Comma has lower precedence than `and`** — the `and` operator binds more tightly than the comma.
- **Media types are deprecated** — `tty`, `tv`, `projection`, `handheld`, `braille`, `embossed`, `aural`, and `speech` are deprecated; only `all`, `print`, and `screen` remain valid.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Logical Operators and Comma-Separated Lists

**HTML File (`operators.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Media Query Operators</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="operators.css">
</head>
<body>
    <div class="box">Resize the window to see the styles change.</div>
</body>
</html>
```

**CSS File (`operators.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.box {
    padding: 30px;
    border-radius: 8px;
    text-align: center;
    font-weight: bold;
    background-color: #3498db;
    color: white;
}

/* AND: screen AND at least 768px */
@media screen and (min-width: 768px) {
    .box {
        background-color: #27ae60;
        content: "AND: screen and (min-width: 768px)";
    }
}

/* NOT: not print (applies on screen, not on print) */
@media not print {
    .box {
        border: 4px solid #2c3e50;
    }
}

/* Comma (OR): screen OR print */
@media screen, print {
    .box {
        font-size: 1.2rem;
    }
}

/* ONLY: legacy safeguard */
@media only screen and (min-width: 1024px) {
    .box {
        background-color: #e67e22;
    }
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `operators.html`.
3. Save the CSS code as `operators.css` in the same folder.
4. Open `operators.html` in a web browser.
5. Resize the window: below 768px the box is blue; above 768px it turns green; above 1024px it turns orange.
6. The `not print` rule adds a border on screen but not when printed.

**Expected Output:** A coloured box that changes appearance based on viewport width. The `and` operator combines screen type with width; the comma acts as OR; `not print` applies the border on screen; `only` is a legacy safeguard.

**Why This Works:** Each media query evaluates independently. The `and` requires both the media type and the width condition to be true. The comma-separated list applies if any query is true. The `not print` negates the print media type, so the border applies on screen. The `only` keyword is ignored by modern browsers but prevents older browsers from applying the query incorrectly.

---

### Real-World Cases

- **Print stylesheets:** `@media print { ... }` to optimise layouts for printing.
- **Mobile-first breakpoints:** `@media screen and (min-width: 768px) { ... }` for progressive enhancement.
- **Multi-target stylesheets:** `@media screen, print { ... }` for styles shared across screen and print.
- **Legacy safeguards:** `@media only screen { ... }` for codebases that need to support very old browsers.

---

## 2. Viewport Conditions: Modern Range Syntax vs. Legacy Properties

### Definitions

**Core Definition:** Viewport conditions are media features that test the dimensions of the viewport (`width`, `height`) and its aspect ratio. Media Queries Level 4 introduced a modern range syntax using mathematical comparison operators, which is more concise and readable than the legacy `min-width`/`max-width` prefix syntax.

**Technical Definition:** The `width`, `height`, `aspect-ratio`, and `orientation` media features are range features, meaning they accept `min-` and `max-` prefixes in the legacy syntax, or can be written using range context syntax with comparison operators. The legacy syntax `(min-width: 40rem)` is equivalent to `(width >= 40rem)`. The range syntax is exclusive to Media Queries Level 4 and is supported in all modern browsers. The range context allows expressions such as `(400px <= width <= 1000px)` to test whether the viewport width falls within a bounded range. The `device-width`, `device-height`, and `device-aspect-ratio` features are deprecated and should not be used in new code.

**Beginner-Friendly Explanation:** Traditionally, you would write `@media (min-width: 768px)` to apply styles when the screen is at least 768px wide. The new range syntax lets you write `@media (width >= 768px)` instead, which reads more like a mathematical comparison. You can also write a range: `@media (400px <= width <= 1000px)` applies only when the viewport width is between 400px and 1000px. The new syntax is shorter, easier to read, and avoids the confusion of remembering whether `min-width` means "at least" or "at most."

---

### Purposes

- To apply styles conditionally based on the viewport's width and height.
- To define breakpoints using either legacy or modern range syntax.
- To test whether the viewport falls within a specific range of dimensions.
- To write more concise and readable media queries with comparison operators.
- To target specific viewport aspect ratios.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Legacy syntax: min-width / max-width */
@media (min-width: 40rem) { ... }
@media (max-width: 80rem) { ... }
@media (min-width: 40rem) and (max-width: 80rem) { ... }

/* Modern range syntax */
@media (width >= 40rem) { ... }
@media (width <= 80rem) { ... }
@media (40rem <= width <= 80rem) { ... }

/* Height and aspect-ratio */
@media (height >= 600px) { ... }
@media (aspect-ratio >= 16/9) { ... }
```

#### Component Breakdown

| Syntax | Description | Equivalence |
|---|---|---|
| `(min-width: 40rem)` | Legacy: width is at least 40rem. | `(width >= 40rem)` |
| `(max-width: 80rem)` | Legacy: width is at most 80rem. | `(width <= 80rem)` |
| `(width >= 40rem)` | Range: width is at least 40rem. | `(min-width: 40rem)` |
| `(width <= 80rem)` | Range: width is at most 80rem. | `(max-width: 80rem)` |
| `(40rem <= width <= 80rem)` | Range: width is between 40rem and 80rem. | `(min-width: 40rem) and (max-width: 80rem)` |
| `(width > 40rem)` | Range: width is strictly greater than 40rem. | No legacy equivalent. |
| `(width < 40rem)` | Range: width is strictly less than 40rem. | No legacy equivalent. |

#### Syntax Rules

1. The range syntax uses `<`, `>`, `<=`, and `>=` comparison operators.
2. The value can be on the left or right side of the operator: `(40rem <= width)` and `(width >= 40rem)` are equivalent.
3. Range syntax can express bounded ranges: `(400px <= width <= 1000px)`.
4. The legacy `min-` and `max-` prefixes are still valid and widely supported.
5. Range syntax is supported in all modern browsers (Chrome 104+, Firefox 102+, Safari 16.4+).
6. `device-width` and `device-height` are deprecated; use `width` and `height`.

#### Constraints and Limitations

- **Browser support** — range syntax is supported in all modern browsers but not in Internet Explorer.
- **Email clients** — many email clients do not support range syntax; use legacy `min-width`/`max-width` for email templates.
- **Mixing syntax** — you can mix legacy and range syntax in the same stylesheet, but consistency is recommended.
- **`min-`/`max-` are inclusive** — `(min-width: 400px)` includes 400px; range syntax `(width >= 400px)` also includes 400px.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Legacy vs. Range Syntax Side by Side

**HTML File (`range.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Range Syntax</title>
    <link rel="stylesheet" href="range.css">
</head>
<body>
    <div class="box">Resize the window to see the styles change.</div>
</body>
</html>
```

**CSS File (`range.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.box {
    padding: 30px;
    border-radius: 8px;
    text-align: center;
    font-weight: bold;
    background-color: #3498db;
    color: white;
}

/* Legacy syntax */
@media (min-width: 600px) {
    .box {
        background-color: #27ae60;
    }
}

/* Range syntax equivalent */
@media (width >= 600px) {
    .box {
        /* Same effect, but written with a comparison operator */
        font-size: 1.2rem;
    }
}

/* Bounded range: 800px to 1200px */
@media (800px <= width <= 1200px) {
    .box {
        background-color: #e67e22;
    }
}

/* Strictly greater than 1200px */
@media (width > 1200px) {
    .box {
        background-color: #9b59b6;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `range.html` and CSS as `range.css`.
2. Open in a browser.
3. Resize the window: below 600px the box is blue; above 600px it turns green and text grows; between 800px and 1200px it turns orange; above 1200px it turns purple.

**Expected Output:** A box that changes colour and font size at different viewport widths, demonstrating the range syntax with `>=`, `<=`, and bounded ranges.

**Why This Works:** The `(width >= 600px)` query is equivalent to `(min-width: 600px)`. The bounded range `(800px <= width <= 1200px)` is equivalent to `(min-width: 800px) and (max-width: 1200px)` but shorter and more readable. The `(width > 1200px)` query uses a strict comparison that has no legacy equivalent.

---

### Real-World Cases

- **Modern responsive frameworks:** Tailwind CSS v4 uses range syntax (`@media (width >= 40rem)`) for its breakpoints.
- **Complex breakpoint logic:** Bounded ranges for targeting specific device categories (e.g., tablets between 768px and 1024px).
- **Email templates:** Legacy `min-width`/`max-width` remains necessary for broad email client support.
- **Height-based layouts:** `@media (height >= 600px)` for layouts that depend on vertical space.

---

## 3. Device Capabilities: Orientation, Resolution, Pointer, and Hover

### Definitions

**Core Definition:** Device capability media features describe the physical and interactive characteristics of the output device and input mechanisms — including the screen's orientation and pixel density, and the precision and hover capability of the primary pointing device.

**Technical Definition:** The `orientation` media feature is `portrait` when the viewport height is greater than or equal to its width, and `landscape` otherwise. The `resolution` media feature describes the pixel density of the output device, accepting values in `dpi` (dots per inch), `dpcm` (dots per centimetre), or `dppx` (dots per pixel). The `pointer` media feature queries the accuracy of the primary pointing device, with values `none`, `coarse`, and `fine`. The `hover` media feature queries the primary pointing device's ability to hover over elements, with values `none` and `hover`. The `any-pointer` and `any-hover` variants query the capabilities of any available pointing device, not just the primary one. These features are defined in Media Queries Level 4.

**Beginner-Friendly Explanation:** These media features let you adapt your design to how the device is being used. `orientation` tells you whether the device is held vertically (portrait) or horizontally (landscape). `resolution` tells you how sharp the screen is — a high-resolution screen needs higher-quality images. `pointer` tells you whether the user is using a precise input device like a mouse (`fine`) or an imprecise one like a finger (`coarse`). `hover` tells you whether the device can hover over elements — a mouse can, but a touchscreen cannot. This lets you design touch-friendly interfaces for touch devices and mouse-optimised interfaces for desktops.

---

### Purposes

- To adapt layouts based on whether the device is in portrait or landscape orientation.
- To serve higher-resolution images to high-DPI screens.
- To design touch-friendly interfaces for devices with coarse pointers.
- To avoid hover-dependent interactions on devices that cannot hover.
- To query the capabilities of any available pointing device, not just the primary one.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Orientation */
@media (orientation: portrait) { ... }
@media (orientation: landscape) { ... }

/* Resolution */
@media (min-resolution: 2dppx) { ... }
@media (resolution >= 300dpi) { ... }

/* Pointer accuracy */
@media (pointer: coarse) { ... }
@media (pointer: fine) { ... }
@media (pointer: none) { ... }

/* Hover capability */
@media (hover: hover) { ... }
@media (hover: none) { ... }

/* Combined: mouse-like device */
@media (hover: hover) and (pointer: fine) { ... }

/* Any available pointing device */
@media (any-pointer: coarse) { ... }
@media (any-hover: hover) { ... }
```

#### Component Breakdown

| Feature | Values | Description |
|---|---|---|
| `orientation` | `portrait`, `landscape` | Viewport orientation. |
| `resolution` | `<resolution>` (`dpi`, `dpcm`, `dppx`) | Pixel density of the output device. |
| `pointer` | `none`, `coarse`, `fine` | Accuracy of the primary pointing device. |
| `hover` | `none`, `hover` | Hover capability of the primary pointing device. |
| `any-pointer` | `none`, `coarse`, `fine` | Accuracy of any available pointing device. |
| `any-hover` | `none`, `hover` | Hover capability of any available pointing device. |

#### Syntax Rules

1. `orientation` is `portrait` when height ≥ width; otherwise `landscape`.
2. `resolution` accepts `dpi`, `dpcm`, and `dppx` units.
3. `pointer: coarse` indicates an imprecise input device (e.g., finger).
4. `pointer: fine` indicates a precise input device (e.g., mouse, stylus).
5. `hover: hover` indicates the primary device can hover easily.
6. `hover: none` indicates the primary device cannot hover or hovering is inconvenient.
7. `any-pointer` and `any-hover` query all available pointing devices, not just the primary one.
8. These features are discrete (they match or do not match); they do not have `min-`/`max-` prefixes except `resolution`.

#### Constraints and Limitations

- **Not a device detector** — `pointer` and `hover` describe capabilities, not device types. A tablet with a connected mouse may report `pointer: fine`.
- **Primary vs. any** — `pointer` and `hover` reflect the primary pointing device; `any-pointer` and `any-hover` reflect any available device.
- **Accessibility override** — user agents may report `hover: none` even on capable devices if the user has difficulty hovering.
- **`orientation` is viewport-relative** — it changes when the user resizes the browser window, not just when the device rotates.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Touch-Friendly and Hover-Aware Design

**HTML File (`capabilities.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Device Capabilities</title>
    <link rel="stylesheet" href="capabilities.css">
</head>
<body>
    <button class="btn">Hover or tap me</button>
    <p class="hint">The button changes size and behaviour based on your input device.</p>
</body>
</html>
```

**CSS File (`capabilities.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    text-align: center;
}

.btn {
    padding: 12px 24px;
    font-size: 1rem;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: background-color 0.2s;
}

/* Hover-capable devices: hover effect */
@media (hover: hover) and (pointer: fine) {
    .btn:hover {
        background-color: #2c3e50;
        transform: scale(1.05);
    }
}

/* Touch devices: larger tap target */
@media (pointer: coarse) {
    .btn {
        padding: 20px 40px;
        font-size: 1.2rem;
        min-width: 48px;
        min-height: 48px;
    }
}

/* High-DPI screens: sharper borders */
@media (min-resolution: 2dppx) {
    .btn {
        border: 2px solid rgba(255, 255, 255, 0.5);
    }
}

/* Landscape orientation: wider layout */
@media (orientation: landscape) {
    body {
        padding: 20px 60px;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `capabilities.html` and CSS as `capabilities.css`.
2. Open in a browser.
3. On a desktop with a mouse: hover over the button — it darkens and scales.
4. On a touch device: the button is larger with more padding for easier tapping.
5. On a high-DPI screen: the button has a visible border.

**Expected Output:** A button that adapts to the input device. On mouse-equipped devices, it has a hover effect. On touch devices, it has larger tap targets. On high-DPI screens, it has a border.

**Why This Works:** The `(hover: hover) and (pointer: fine)` query targets devices with a precise, hovering input device (typically a mouse). The `(pointer: coarse)` query targets touch devices with imprecise input. The `(min-resolution: 2dppx)` query targets high-DPI screens. Each query applies styles only when its condition matches, creating a device-appropriate experience.

---

### Real-World Cases

- **Touch-friendly navigation:** Larger tap targets for mobile navigation menus on `pointer: coarse` devices.
- **Hover menus:** Hover-activated dropdowns on `hover: hover` devices, with tap-based alternatives on `hover: none` devices.
- **High-DPI images:** Serving higher-resolution images on `min-resolution: 2dppx` screens.
- **Orientation layouts:** Switching from a stacked layout in portrait to a side-by-side layout in landscape.

---

## 4. User Preference Media Features: Designing for Accessibility and Device States

### Definitions

**Core Definition:** User preference media features are media queries that detect the user's operating system or browser accessibility settings — including colour scheme preference, motion sensitivity, contrast needs, and data-saving preferences — allowing websites to adapt their presentation to the user's expressed needs.

**Technical Definition:** Media Queries Level 5 defines a set of user-preference media features that reflect user-configured settings: `prefers-color-scheme` (values: `light`, `dark`) detects whether the user prefers a light or dark colour theme; `prefers-reduced-motion` (values: `no-preference`, `reduce`) detects whether the user has requested minimised non-essential motion; `prefers-contrast` (values: `no-preference`, `more`, `less`, `custom`) detects whether the user has requested higher or lower contrast; `prefers-reduced-transparency` (values: `no-preference`, `reduce`) detects whether the user prefers reduced transparency; and `prefers-reduced-data` (values: `no-preference`, `reduce`) detects whether the user prefers reduced data usage. These features are designed to respect user autonomy and improve accessibility.

**Beginner-Friendly Explanation:** These media queries let your website respect the user's personal settings. If the user has enabled dark mode on their phone, `prefers-color-scheme: dark` lets your site serve a dark theme. If the user has motion sensitivity, `prefers-reduced-motion: reduce` lets you disable animations. If the user needs higher contrast, `prefers-contrast: more` lets you adjust colours accordingly. These features are about accessibility and respect — they let users control their experience rather than forcing a one-size-fits-all design.

---

### Purposes

- To serve a dark colour scheme when the user prefers dark mode.
- To disable or reduce animations when the user has motion sensitivity.
- To increase contrast when the user requires higher contrast for readability.
- To reduce data usage when the user is on a constrained connection.
- To respect user autonomy and improve accessibility for all users.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Dark mode */
@media (prefers-color-scheme: dark) { ... }
@media (prefers-color-scheme: light) { ... }

/* Reduced motion */
@media (prefers-reduced-motion: reduce) { ... }
@media (prefers-reduced-motion: no-preference) { ... }

/* Contrast */
@media (prefers-contrast: more) { ... }
@media (prefers-contrast: less) { ... }

/* Reduced transparency */
@media (prefers-reduced-transparency: reduce) { ... }

/* Reduced data */
@media (prefers-reduced-data: reduce) { ... }
```

#### Component Breakdown

| Feature | Values | Description |
|---|---|---|
| `prefers-color-scheme` | `light`, `dark` | User's preferred colour theme. |
| `prefers-reduced-motion` | `no-preference`, `reduce` | User's motion sensitivity preference. |
| `prefers-contrast` | `no-preference`, `more`, `less`, `custom` | User's contrast preference. |
| `prefers-reduced-transparency` | `no-preference`, `reduce` | User's transparency preference. |
| `prefers-reduced-data` | `no-preference`, `reduce` | User's data-saving preference. |

#### Syntax Rules

1. `prefers-color-scheme` accepts `light` and `dark`; the default is usually `light` but varies by browser.
2. `prefers-reduced-motion: reduce` applies when the user has requested reduced motion.
3. `prefers-contrast: more` applies when the user has requested higher contrast.
4. `prefers-reduced-transparency: reduce` applies when the user prefers reduced transparency.
5. `prefers-reduced-data: reduce` applies when the user prefers reduced data usage.
6. All these features are boolean or discrete; they match when the specified condition is true.

#### Constraints and Limitations

- **Browser and OS support** — support for these features varies; `prefers-color-scheme` and `prefers-reduced-motion` are widely supported; `prefers-reduced-data` and `prefers-reduced-transparency` have more limited support.
- **Default values** — the default value for each feature is not guaranteed and may vary by browser.
- **Not a guarantee** — these features reflect user preferences but do not guarantee the user's actual needs.
- **Always provide a baseline** — define light-mode and no-motion styles as the default, then override with preferences.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dark Mode, Reduced Motion, and High Contrast

**HTML File (`preferences.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User Preference Media Queries</title>
    <link rel="stylesheet" href="preferences.css">
</head>
<body>
    <div class="card">
        <h1>Respect User Preferences</h1>
        <p>This card adapts to your colour scheme, motion preference, and contrast needs.</p>
        <button class="btn">Animated Button</button>
    </div>
</body>
</html>
```

**CSS File (`preferences.css`):**

```css
/* Default: light mode, normal motion, normal contrast */
:root {
    --color-text: #1a1a1a;
    --color-background: #ffffff;
    --color-primary: #0066cc;
    --color-card: #f8f8f8;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: var(--color-background);
    color: var(--color-text);
    transition: background-color 0.3s, color 0.3s;
}

.card {
    background-color: var(--color-card);
    padding: 30px;
    border-radius: 10px;
    max-width: 500px;
    margin: 0 auto;
}

.btn {
    padding: 12px 24px;
    background-color: var(--color-primary);
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 1rem;
    transition: transform 0.2s, background-color 0.2s;
}

.btn:hover {
    transform: scale(1.05);
    background-color: #0055aa;
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
    :root {
        --color-text: #e8e8e8;
        --color-background: #1a1a1a;
        --color-primary: #66aaff;
        --color-card: #2a2a2a;
    }
}

/* Reduced motion: disable animations */
@media (prefers-reduced-motion: reduce) {
    body, .btn {
        transition-duration: 0.001ms;
    }
    .btn:hover {
        transform: none;
    }
}

/* Higher contrast */
@media (prefers-contrast: more) {
    :root {
        --color-text: #000000;
        --color-background: #ffffff;
        --color-primary: #0000cc;
        --color-card: #ffffff;
    }
    .card {
        border: 2px solid #000000;
    }
    .btn {
        border: 2px solid #000000;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `preferences.html` and CSS as `preferences.css`.
2. Open in a browser.
3. Enable dark mode in your operating system settings — the page switches to a dark theme.
4. Enable "Reduce Motion" in your OS accessibility settings — the button no longer animates on hover.
5. Enable "High Contrast" — the page uses higher-contrast colours and borders.

**Expected Output:** A card that adapts to the user's colour scheme (light/dark), motion preference (animated/static), and contrast needs (normal/high contrast). The default is light mode with normal motion and contrast.

**Why This Works:** The `:root` custom properties define the default light-mode theme. The `prefers-color-scheme: dark` query overrides these properties with dark-mode values. The `prefers-reduced-motion: reduce` query disables transitions and transforms. The `prefers-contrast: more` query increases contrast and adds borders. Each preference is handled independently, and the default (no media query) provides the baseline experience.

---

### Real-World Cases

- **Dark mode support:** Serving a dark theme when the user's OS is set to dark mode.
- **Accessibility compliance:** Respecting `prefers-reduced-motion` to prevent vestibular triggers in users with motion sensitivity.
- **High-contrast mode:** Increasing contrast and adding borders for users with low vision.
- **Data-saving mode:** Serving lower-resolution images or fewer web fonts when `prefers-reduced-data: reduce` is set.
- **Transparency reduction:** Using solid backgrounds when `prefers-reduced-transparency: reduce` is set.

---

## References

- MDN Web Docs — Using media queries - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries
- MDN Web Docs — `@media` - https://developer.mozilla.org/en-US/docs/Web/CSS/@media
- W3C — Media Queries Level 3 - https://www.w3.org/TR/css3-mediaqueries/
- W3C — Media Queries Level 4 - https://www.w3.org/TR/mediaqueries-4/
- W3C — Media Queries Level 5 (Editor's Draft) - https://drafts.csswg.org/mediaqueries-5/
- CSS-Tricks — The New CSS Media Query Range Syntax - https://css-tricks.com/the-new-css-media-query-range-syntax/
- CSS-Tricks — A Complete Guide to CSS Media Queries - https://css-tricks.com/a-complete-guide-to-css-media-queries/
- CSS-Tricks — `prefers-reduced-motion` - https://css-tricks.com/almanac/rules/m/media/prefers-reduced-motion/
- Frontend Masters — How much do you really know about media queries? - https://frontendmasters.com/blog/learn-media-queries/
- web.dev — New syntax for range media queries - https://web.dev/articles/media-query-range-syntax
- web.dev — prefers-color-scheme: Hello darkness, my old friend - https://web.dev/articles/prefers-color-scheme
- Web Platform DX — Media query range syntax - https://web-platform-dx.github.io/web-features-explorer/features/media-query-range-syntax/
- Can I Use — Media Queries: Range Syntax - https://caniuse.com/media-query-range-syntax
- Can I Use — `prefers-reduced-motion` - https://caniuse.com/prefers-reduced-motion
- Can I Use — `prefers-color-scheme` - https://caniuse.com/prefers-color-scheme