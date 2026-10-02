# CSS Functional Notations — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS functional notations are named operations written in the form `function-name(arguments)` that compute values at parse time or computed-value time. They allow authors to express dynamic relationships, perform arithmetic, reference external data, and generate colours procedurally — capabilities that static keyword values cannot provide.

**Technical Definition:** In formal CSS terms, functional notations are defined across multiple CSS modules — primarily CSS Values and Units Level 4 (math functions), CSS Grid Layout Level 1 (grid functions), CSS Custom Properties Level 1 (`var()`), CSS Environment Variables Level 1 (`env()`), CSS Color Level 5 (colour functions), and CSS Values and Units Level 5 (`attr()`). Each function has a defined grammar, a set of permitted argument types, and a resolution algorithm that produces a computed value. Functions may be nested, combined with `calc()`, and used anywhere a value of the resulting type is accepted.

**Beginner-Friendly Explanation:** CSS functions are like built-in tools that let you calculate, combine, and transform values directly in your stylesheets. Instead of hard-coding `width: 300px`, you can write `width: calc(100% - 2rem)` to say "full width minus the padding." Instead of picking a colour manually, you can write `color-mix(in oklch, blue, red)` to blend two colours. Functions make CSS smarter, more flexible, and more maintainable.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Composable** | Functions can be nested inside other functions and combined with `calc()`. |
| **Type-checked** | Arguments must match the function's expected data types; mismatches invalidate the declaration. |
| **Context-aware** | Some functions resolve against the element's font, the viewport, a container, or the user's environment. |
| **Runtime-resolved** | Many functions compute at computed-value time, allowing dynamic values based on layout. |
| **Progressive enhancement** | Unsupported functions cause the declaration to be ignored, enabling graceful degradation. |

---

### Prerequisites

- **CSS values and units** — understanding data types (`<length>`, `<number>`, `<percentage>`, `<color>`).
- **CSS custom properties** — the `--*` syntax and `var()` usage.
- **CSS Grid basics** — track sizing, `grid-template-columns`, `grid-template-rows`.
- **CSS colour models** — sRGB, HSL, and modern colour spaces.
- **Basic arithmetic** — operator precedence and unit conversion.

---

### Related Programming Areas

- **Responsive design** — `clamp()`, `min()`, `max()`, viewport units.
- **Design systems** — `var()` for tokens, `color-mix()` for palette generation.
- **Accessibility** — `light-dark()` for colour-scheme-aware theming, `env()` for safe-area insets.
- **CSS Grid** — `minmax()`, `fit-content()`, `repeat()`.
- **Animation** — `calc()` for dynamic timing and positioning.
- **Internationalisation** — logical properties and `ic` units.

---

### Core Concepts / Features

1. Basic Mathematical Functions (`calc()`, `min()`, `max()`, `clamp()`)
2. Grid and Component Layout Functions (`minmax()`, `fit-content()`, `repeat()`)
3. Advanced & Stepped Math Functions (`round()`, `mod()`, `rem()`, `abs()`, `sign()`, trigonometric functions)
4. Custom Properties & Environment Functions (`var()`, `env()`)
5. Data & Attribute Functions (`attr()`, `light-dark()`)
6. Color Functions (legacy and modern spaces, relative colour syntax, `color-mix()`)

---

## 1. Basic Mathematical Functions

### Definitions

**Core Definition:** `calc()`, `min()`, `max()`, and `clamp()` are CSS math functions that perform arithmetic on numeric values, allowing authors to mix units and impose bounds. They enable fluid, responsive values without JavaScript.

**Technical Definition:** The math functions are defined in CSS Values and Units Level 4. `calc()` takes a single `<calc-sum>` expression and evaluates it using standard operator precedence. `min()` and `max()` take a comma-separated list of `<calc-sum>` expressions and return the smallest or largest, respectively. `clamp()` takes exactly three expressions — a minimum, a preferred value, and a maximum — and returns the preferred value constrained between the min and max. Whitespace is required around `+` and `-` operators; `*` and `/` may be written without spaces.

**Beginner-Friendly Explanation:** `calc()` is a calculator for CSS. `min()` and `max()` pick the smaller or larger of several values. `clamp()` keeps a value within a range. Together, they let you create values that adapt: a font size that grows with the screen but never gets too small or too large; a width that fills the container but leaves room for padding.

---

### Purposes

- To mix units (e.g., percentages and pixels) in a single value.
- To create fluid typography that scales with the viewport but remains bounded.
- To compute layout dimensions dynamically based on other properties.
- To enforce safe minimums and maximums on otherwise fluid values.
- To perform arithmetic with custom properties (`var()`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
calc(<calc-sum>)
min(<calc-sum>#)
max(<calc-sum>#)
clamp(<calc-sum>#{3})

<calc-sum> = <calc-product> [ [ '+' | '-' ] <calc-product> ]*
<calc-product> = <calc-value> [ [ '*' | '/' ] <calc-value> ]*
<calc-value> = <number> | <dimension> | <percentage> | <calc-constant> | ( <calc-sum> )
```

#### Component Breakdown

| Function | Arguments | Description |
|---|---|---|
| `calc()` | One expression | Evaluates the expression to a single value. |
| `min()` | Two or more expressions | Returns the smallest of the arguments. |
| `max()` | Two or more expressions | Returns the largest of the arguments. |
| `clamp()` | Three expressions: min, preferred, max | Returns `preferred` clamped between `min` and `max`. |

#### Syntax Rules

1. Whitespace is **mandatory** around `+` and `-`: `calc(100% - 20px)` is valid; `calc(100%-20px)` is invalid.
2. `*` and `/` may be written without spaces.
3. Arguments inside `min()`, `max()`, and `clamp()` are full math expressions — no need to nest `calc()`.
4. `clamp(MIN, VAL, MAX)` returns `max(MIN, min(VAL, MAX))`. If MIN > MAX, the MIN wins.
5. Math functions resolve to a type (length, number, etc.) based on their contents and can be used anywhere that type is accepted.
6. At least 32 terms and 32 levels of nesting must be supported.

#### Constraints and Limitations

- **Type compatibility** — arguments to `+` and `-` must be of the same type; adding a `<length>` to an `<angle>` is invalid.
- **Unitless zero inside calc()** — `calc(0 + 5px)` is invalid because `0` is a `<number>`; `calc(0px + 5px)` is required.
- **Division by zero** — produces an invalid value (NaN); the declaration is ignored.
- **Browser support** — `calc()`, `min()`, `max()`, and `clamp()` are widely supported; `clamp()` has slightly less historical support.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fluid Typography with `clamp()`

**HTML File (`clamp-typography.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fluid Typography with clamp()</title>
    <link rel="stylesheet" href="clamp-typography.css">
</head>
<body>
    <h1 class="fluid-heading">Fluid Heading</h1>
    <p class="fluid-body">
        This paragraph uses clamp() to scale the font size with the viewport.
        It never drops below 1rem (16px) and never exceeds 1.25rem (20px).
    </p>
</body>
</html>
```

**CSS File (`clamp-typography.css`):**

```css
/* Base body styling */
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 5vw;
    background-color: #fafafa;
}

/* Fluid heading: min 2rem, preferred 5vw, max 4rem */
.fluid-heading {
    /* clamp(MIN, VAL, MAX) = max(MIN, min(VAL, MAX)) */
    font-size: clamp(2rem, 5vw, 4rem);
    color: #2c3e50;
    line-height: 1.2;
    margin-bottom: 3vh;
}

/* Fluid body: min 1rem, preferred 2.5vw, max 1.25rem */
.fluid-body {
    font-size: clamp(1rem, 2.5vw, 1.25rem);
    line-height: 1.7;
    color: #34495e;
    max-width: 65ch;
}
```

**Step-by-Step Setup Guide:**

1. Save the files as `clamp-typography.html` and `clamp-typography.css`.
2. Open the HTML file in a browser.
3. Resize the window from narrow to wide.
4. Observe the heading grows from 2rem to 4rem but never smaller or larger.

**Expected Output:** A heading that is 2rem on narrow screens, scales with 5vw in the middle range, and caps at 4rem on wide screens. Body text scales between 1rem and 1.25rem.

**Why This Works:** `clamp()` evaluates the preferred value (5vw) and constrains it between the minimum (2rem) and maximum (4rem). On narrow screens, 5vw is below 2rem, so the minimum is returned. On wide screens, 5vw exceeds 4rem, so the maximum is returned. In between, the preferred value is used directly.

---

#### Example 2: `calc()` for Responsive Layout

**HTML File (`calc-layout.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>calc() Layout</title>
    <link rel="stylesheet" href="calc-layout.css">
</head>
<body>
    <div class="container">
        <div class="sidebar">Sidebar</div>
        <div class="main">Main Content</div>
    </div>
</body>
</html>
```

**CSS File (`calc-layout.css`):**

```css
/* Reset */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: system-ui, sans-serif;
    background-color: #f5f5f5;
}

/* Flex container */
.container {
    display: flex;
    min-height: 100vh;
}

/* Sidebar: fixed 250px width */
.sidebar {
    width: 250px;
    background-color: #2c3e50;
    color: white;
    padding: 2rem;
}

/* Main content: fills remaining space minus gap */
.main {
    /* calc() subtracts the sidebar width and the gap from 100% */
    width: calc(100% - 250px - 2rem);
    padding: 2rem;
    background-color: white;
    margin-left: 2rem;
}
```

**Step-by-Step Setup Guide:**

1. Save the files as `calc-layout.html` and `calc-layout.css`.
2. Open the HTML file in a browser.
3. Resize the window; the sidebar stays fixed at 250px, and the main content adjusts.

**Expected Output:** A two-column layout with a fixed-width sidebar and a fluid main content area separated by a 2rem gap.

**Why This Works:** `calc(100% - 250px - 2rem)` mixes a percentage (100% of the parent width), an absolute length (250px), and a relative length (2rem). The browser resolves the percentage against the container width at layout time, producing a single length value for the `width` property.

---

### Real-World Cases

- **Responsive hero sections:** `height: calc(100vh - 80px)` to fill the viewport minus a fixed header.
- **Fluid typography:** `font-size: clamp(1rem, 2.5vw, 1.5rem)` for headings that scale with the viewport.
- **Centering with offsets:** `left: calc(50% - 100px)` to centre a fixed-width element.
- **Grid gaps:** `gap: calc(1rem + 1vw)` for responsive spacing.

---

## 2. Grid and Component Layout Functions

### Definitions

**Core Definition:** `minmax()`, `fit-content()`, and `repeat()` are CSS Grid functions that define track sizing behaviour, enabling flexible, responsive grid layouts without media queries.

**Technical Definition:** `minmax()` defines a size range for a grid track, accepting a minimum and maximum value. `fit-content()` clamps a size to an available space using the formula `min(maximum size, max(minimum size, argument))`. `repeat()` represents a repeated fragment of a track list, allowing compact specification of recurring patterns. These functions are defined in CSS Grid Layout Level 1 and Level 2.

**Beginner-Friendly Explanation:** These functions are the building blocks of CSS Grid. `minmax(200px, 1fr)` means "at least 200px, but grow to fill available space." `fit-content(300px)` means "shrink to fit your content, but don't exceed 300px." `repeat(3, 1fr)` means "create three equal columns." Together, they make grid layouts that adapt naturally to any screen size.

---

### Purposes

- To define flexible track sizes with minimum and maximum constraints.
- To create responsive grids that adapt to available space without media queries.
- To avoid repetition in track lists with recurring patterns.
- To clamp content to a readable width while allowing it to grow with the container.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
minmax(<min>, <max>)
fit-content(<length-percentage>)
repeat(<count>, <track-list>)
repeat(auto-fill, <track-list>)
repeat(auto-fit, <track-list>)
```

#### Component Breakdown

| Function | Arguments | Description |
|---|---|---|
| `minmax()` | `<min>`, `<max>` | Defines a size range. `max` < `min` → `max` ignored. |
| `fit-content()` | `<length-percentage>` | Clamps to `min(max-content, max(min-content, argument))`. |
| `repeat()` | `<count>` or `auto-fill`/`auto-fit`, `<track-list>` | Repeats the track list. |

#### Syntax Rules

1. `minmax()` accepts `<length>`, `<percentage>`, `<flex>`, `max-content`, `min-content`, and `auto` for both arguments.
2. If `max` < `min` in `minmax()`, the `max` is ignored and `minmax(min, max)` is treated as `min`.
3. `repeat()` with `auto-fill` creates as many tracks as fit without overflowing; `auto-fit` collapses empty tracks.
4. `fit-content()` accepts a single `<length>` or `<percentage>` argument.
5. `repeat()` can appear multiple times in a track list, but only once per `auto-repeat`.

#### Constraints and Limitations

- `auto-fill` and `auto-fit` cannot be combined with an explicit `<integer>` count.
- `minmax()` cannot be nested inside `minmax()`.
- `fit-content()` is not a valid track size inside `minmax()`.
- `repeat()` with `auto-fill`/`auto-fit` cannot contain `subgrid`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Responsive Grid with `auto-fit` and `minmax()`

**HTML File (`grid-auto.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Grid</title>
    <link rel="stylesheet" href="grid-auto.css">
</head>
<body>
    <div class="grid">
        <div class="card">Card 1</div>
        <div class="card">Card 2</div>
        <div class="card">Card 3</div>
        <div class="card">Card 4</div>
        <div class="card">Card 5</div>
        <div class="card">Card 6</div>
    </div>
</body>
</html>
```

**CSS File (`grid-auto.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    padding: 2rem;
    background-color: #f5f5f5;
}

/* Responsive grid using repeat() and minmax() */
.grid {
    display: grid;
    /* Create as many columns as fit, each between 250px and 1fr */
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1.5rem;
}

/* Card styling */
.card {
    background-color: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    text-align: center;
    font-weight: 600;
    color: #2c3e50;
}
```

**Step-by-Step Setup Guide:**

1. Save the files as `grid-auto.html` and `grid-auto.css`.
2. Open in a browser.
3. Resize the window: columns appear and disappear automatically.

**Expected Output:** A grid that shows 1 column on narrow screens, 2 on medium, and 3 or more on wide screens, with cards stretching to fill the space.

**Why This Works:** `auto-fit` creates as many 250px-minimum columns as will fit. Each column grows to fill available space via `1fr`. When the container is narrow, only one column fits; on wider screens, more columns appear. Empty tracks collapse, so the layout remains balanced.

---

### Real-World Cases

- **Dashboard layouts:** `repeat(auto-fit, minmax(300px, 1fr))` for responsive widget grids.
- **Image galleries:** `repeat(auto-fill, minmax(200px, 1fr))` for photo grids that adapt to any screen.
- **Form layouts:** `grid-template-columns: repeat(2, minmax(200px, 1fr))` for two-column forms.
- **Content areas:** `fit-content(65ch)` to constrain article text to a readable measure.

---

## 3. Advanced & Stepped Math Functions

### Definitions

**Core Definition:** CSS provides a suite of advanced math functions beyond `calc()`: `round()` for rounding, `mod()` and `rem()` for modular arithmetic, `abs()` and `sign()` for absolute value and sign, and trigonometric functions (`sin()`, `cos()`, `tan()`, `asin()`, `acos()`, `atan()`, `atan2()`) for geometry and animation.

**Technical Definition:** These functions are defined in CSS Values and Units Level 4. `round(<strategy>, A, B)` rounds `A` to the nearest multiple of `B` using the specified strategy (`nearest`, `up`, `down`, `to-zero`). `mod(A, B)` returns the modulus (result has the sign of the divisor); `rem(A, B)` returns the remainder (result has the sign of the dividend). `abs(A)` returns the absolute value; `sign(A)` returns −1, 0, or 1. Trigonometric functions accept `<number>` or `<angle>` arguments and return numbers; inverse functions accept numbers and return angles.

**Beginner-Friendly Explanation:** These are the advanced mathematical tools. `round()` snaps a value to a grid (e.g., round a width to the nearest 10px). `mod()` and `rem()` are useful for cyclic patterns (e.g., calculating a position within a repeating cycle). `abs()` gives the distance from zero. `sign()` tells you if a number is positive, negative, or zero. Trig functions let you compute positions on a circle, useful for circular layouts and animations.

---

### Purposes

- To snap values to a grid or increment for consistent spacing.
- To compute cyclic or modular values for repeating patterns.
- To normalise values using absolute value and sign.
- To compute positions, rotations, and paths using trigonometry.
- To create circular layouts and animations without JavaScript.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
round(<rounding-strategy>?, A, B)
mod(A, B)
rem(A, B)
abs(A)
sign(A)
sin(A)
cos(A)
tan(A)
asin(A)
acos(A)
atan(A)
atan2(A, B)
```

#### Component Breakdown

| Function | Arguments | Returns |
|---|---|---|
| `round()` | optional strategy, value, interval | Rounded value |
| `mod()` | dividend, divisor | Modulus (sign of divisor) |
| `rem()` | dividend, divisor | Remainder (sign of dividend) |
| `abs()` | value | Absolute value |
| `sign()` | value | −1, 0, or 1 |
| `sin()`, `cos()`, `tan()` | number or angle | number (−1 to 1 for sin/cos) |
| `asin()`, `acos()`, `atan()` | number | angle |
| `atan2()` | Y, X | angle |

#### Syntax Rules

1. `round()` strategies: `nearest` (default), `up`, `down`, `to-zero`.
2. If `B` is 0 in `round()`, `mod()`, or `rem()`, the result is NaN.
3. `sin(45deg)`, `sin(.125turn)`, and `sin(π/4)` all return approximately 0.707.
4. Inverse trig functions return angles normalized to specific ranges: `asin()` → [−90deg, 90deg], `acos()` → [0deg, 180deg], `atan()` → [−90deg, 90deg].
5. `atan2(Y, X)` returns the angle between the positive X-axis and point (X, Y).

#### Constraints and Limitations

- **Browser support** — `round()`, `mod()`, and `rem()` are Baseline 2024 (May 2024); trigonometric functions have similar support.
- **NaN handling** — division by zero and infinite arguments produce NaN, invalidating the declaration.
- **Unit compatibility** — `mod()` and `rem()` require both arguments to have the same type.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `round()` for Consistent Spacing

**HTML File (`round-spacing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>round() Spacing</title>
    <link rel="stylesheet" href="round-spacing.css">
</head>
<body>
    <div class="box">Width rounded to nearest 50px</div>
    <div class="box-2">Width rounded up to nearest 50px</div>
</body>
</html>
```

**CSS File (`round-spacing.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    padding: 2rem;
    background-color: #f5f5f5;
}

/* Box 1: width rounded to nearest 50px */
.box {
    /* round(nearest, 137px, 50px) = 150px */
    width: round(nearest, 137px, 50px);
    background-color: #3498db;
    color: white;
    padding: 1rem;
    margin-bottom: 1rem;
    border-radius: 6px;
}

/* Box 2: width rounded up to nearest 50px */
.box-2 {
    /* round(up, 137px, 50px) = 150px */
    width: round(up, 137px, 50px);
    background-color: #e74c3c;
    color: white;
    padding: 1rem;
    border-radius: 6px;
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser.
2. Observe both boxes have a width of 150px.

**Expected Output:** Two boxes, both 150px wide, demonstrating that `round(nearest, 137, 50)` and `round(up, 137, 50)` both produce 150.

**Why This Works:** `round(nearest, 137, 50)` finds the multiple of 50 closest to 137, which is 150 (since 137 is closer to 150 than to 100). `round(up, 137, 50)` finds the smallest multiple of 50 that is ≥ 137, which is 150.

---

#### Example 2: Circular Layout with `sin()` and `cos()`

**HTML File (`trig-circle.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Circular Layout with Trig</title>
    <link rel="stylesheet" href="trig-circle.css">
</head>
<body>
    <div class="circle">
        <div class="dot" style="--angle: 0deg;">1</div>
        <div class="dot" style="--angle: 60deg;">2</div>
        <div class="dot" style="--angle: 120deg;">3</div>
        <div class="dot" style="--angle: 180deg;">4</div>
        <div class="dot" style="--angle: 240deg;">5</div>
        <div class="dot" style="--angle: 300deg;">6</div>
    </div>
</body>
</html>
```

**CSS File (`trig-circle.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background-color: #f5f5f5;
}

/* Circle container */
.circle {
    position: relative;
    width: 300px;
    height: 300px;
    border-radius: 50%;
    border: 2px dashed #bdc3c7;
}

/* Dots positioned using trigonometry */
.dot {
    position: absolute;
    width: 40px;
    height: 40px;
    background-color: #3498db;
    color: white;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    /* Position using sin() and cos() */
    left: calc(50% + 130px * cos(var(--angle)) - 20px);
    top: calc(50% + 130px * sin(var(--angle)) - 20px);
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser.
2. Observe six dots evenly spaced around the circle.

**Expected Output:** Six numbered dots arranged in a perfect circle.

**Why This Works:** Each dot has a `--angle` custom property. `cos()` and `sin()` convert the angle to X and Y offsets. `left: calc(50% + 130px * cos(var(--angle)) - 20px)` positions the dot horizontally, and `top` positions it vertically. The `- 20px` centres the 40px dot.

---

### Real-World Cases

- **Consistent spacing systems:** Using `round()` to snap values to a spacing scale.
- **Loading spinners:** Using `sin()` and `cos()` for circular motion paths.
- **Progress indicators:** Using `mod()` to cycle through states.
- **Responsive sizing:** Using `abs()` to ensure positive dimensions.

---

## 4. Custom Properties & Environment Functions

### Definitions

**Core Definition:** `var()` inserts the value of a custom property (a CSS variable) into a property value, with an optional fallback. `env()` inserts a user-agent-defined environment variable, such as safe-area insets on notched devices.

**Technical Definition:** `var(--name, fallback)` is defined in CSS Custom Properties Level 1. It substitutes the computed value of the referenced custom property. If the property is invalid or not defined, the fallback (if provided) is used; otherwise, the declaration is invalid. `env(<name>, fallback)` is defined in CSS Environment Variables Level 1. It inserts the value of a browser- or OS-provided variable, primarily `safe-area-inset-*` for device notches and rounded corners.

**Beginner-Friendly Explanation:** `var()` lets you define reusable values once and use them everywhere. `env()` gives you access to special browser-provided values, like the size of the notch on an iPhone screen, so your content does not get hidden behind it.

---

### Purposes

- To define reusable design tokens (colours, spacing, typography).
- To enable dynamic theming by changing custom property values.
- To provide fallback values for robustness.
- To respect device safe areas (notches, rounded corners).
- To create context-aware components that adapt to their environment.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
var(<custom-property-name>, <declaration-value>?)
env(<custom-ident>, <declaration-value>?)
```

#### Component Breakdown

| Function | First Argument | Second Argument |
|---|---|---|
| `var()` | Custom property name (e.g., `--primary-color`) | Fallback value (optional) |
| `env()` | Environment variable name (e.g., `safe-area-inset-top`) | Fallback value (optional) |

#### Syntax Rules

1. `var()` cannot be used in property names, selectors, or anywhere other than property values.
2. The fallback in `var()` can itself be a `var()` function, allowing chained fallbacks.
3. `env()` is globally scoped to the document, unlike `var()` which is element-scoped.
4. `safe-area-inset-*` variables are the only universally supported environment variables.
5. `env()` values update dynamically (e.g., on device rotation).

#### Constraints and Limitations

- **`var()` cycles** — circular references invalidate all involved custom properties.
- **`var()` in shorthand** — using `var()` in shorthand properties can reset longhands.
- **`env()` support** — `safe-area-inset-*` is widely supported; other environment variables are experimental.
- **Fallback complexity** — fallbacks containing commas require care because the first comma ends the property name.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `var()` for Design Tokens

**HTML File (`var-tokens.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Design Tokens with var()</title>
    <link rel="stylesheet" href="var-tokens.css">
</head>
<body>
    <div class="card">
        <h2>Card Title</h2>
        <p>Card content using design tokens.</p>
    </div>
    <div class="card card--alt">
        <h2>Alternate Card</h2>
        <p>This card uses different token values.</p>
    </div>
</body>
</html>
```

**CSS File (`var-tokens.css`):**

```css
/* Root design tokens */
:root {
    --color-primary: #3498db;
    --color-text: #2c3e50;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 2rem;
    --radius: 8px;
}

/* Alternate theme */
.card--alt {
    --color-primary: #e74c3c;
    --color-text: #c0392b;
}

/* Body styling */
body {
    font-family: system-ui, sans-serif;
    background-color: #f5f5f5;
    padding: var(--spacing-lg);
}

/* Card component using tokens */
.card {
    background: white;
    padding: var(--spacing-lg);
    margin-bottom: var(--spacing-md);
    border-radius: var(--radius);
    border-left: 4px solid var(--color-primary);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card h2 {
    color: var(--color-text);
    margin-bottom: var(--spacing-sm);
}

.card p {
    color: var(--color-text);
    opacity: 0.8;
    margin: 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser.
2. Observe the first card is blue; the second is red.

**Expected Output:** Two cards with different accent colours, both using the same CSS component code. The alternate card changes only the token values.

**Why This Works:** The `:root` selector defines custom properties. The `.card` component references them via `var()`. The `.card--alt` class overrides the tokens, so the same component renders differently without changing any component CSS.

---

#### Example 2: `env()` for Safe Areas

**HTML File (`env-safe.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
    <title>Safe Area with env()</title>
    <link rel="stylesheet" href="env-safe.css">
</head>
<body>
    <header class="app-header">App Header</header>
    <main class="app-content">
        <p>This content respects the device safe areas.</p>
    </main>
    <footer class="app-footer">App Footer</footer>
</body>
</html>
```

**CSS File (`env-safe.css`):**

```css
/* Reset */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: system-ui, sans-serif;
    background-color: #f5f5f5;
}

/* Header: respects top safe area */
.app-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background-color: #2c3e50;
    color: white;
    text-align: center;
    /* Add safe-area-inset-top to normal padding */
    padding-top: calc(1rem + env(safe-area-inset-top, 0px));
    padding-bottom: 1rem;
    z-index: 10;
}

/* Content: respects left and right safe areas */
.app-content {
    padding: 5rem 1rem 5rem 1rem;
    /* Add safe-area-inset-left and -right */
    padding-left: calc(1rem + env(safe-area-inset-left, 0px));
    padding-right: calc(1rem + env(safe-area-inset-right, 0px));
    max-width: 700px;
    margin: 0 auto;
}

/* Footer: respects bottom safe area */
.app-footer {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: #2c3e50;
    color: white;
    text-align: center;
    padding: 1rem;
    padding-bottom: calc(1rem + env(safe-area-inset-bottom, 0px));
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser (use a device with a notch or emulate one).
2. Observe the header, content, and footer respect the device's safe areas.

**Expected Output:** Content that does not get hidden behind notches or rounded corners.

**Why This Works:** `env(safe-area-inset-top, 0px)` returns the size of the top safe area (or 0px if not defined). Adding it to normal padding ensures content clears the notch. The fallback `0px` ensures the layout works on devices without notches.

---

### Real-World Cases

- **Design systems:** Using `var()` for colours, spacing, and typography tokens.
- **Theming:** Switching between light and dark mode by changing token values.
- **Mobile apps (PWA):** Using `env(safe-area-inset-*)` for notched devices.
- **White-labeling:** Overriding `var()` values per brand or client.

---

## 5. Data & Attribute Functions

### Definitions

**Core Definition:** `attr()` retrieves the value of an HTML attribute for use in a CSS property. `light-dark()` returns one of two values based on the current `color-scheme`.

**Technical Definition:** `attr(<attr-name>, <type-or-unit>?, <fallback>?)` is defined in CSS Values and Units Level 5. It reads an attribute from the selected element (or its originating element for pseudo-elements) and converts it to the specified type. `light-dark(<color>, <color>)` is defined in CSS Color Level 5. It returns the first colour when the used `color-scheme` is light or unknown, and the second when it is dark.

**Beginner-Friendly Explanation:** `attr()` lets you use HTML attribute values in your CSS. For example, you could store a colour in a `data-color` attribute and use it in your styles. `light-dark()` lets you define two colours — one for light mode and one for dark mode — and automatically picks the right one based on the user's system preference.

---

### Purposes

- To extract HTML attribute values for use in CSS (content, dimensions, colours).
- To reduce duplication between HTML and CSS.
- To create automatic light/dark mode theming.
- To provide accessible colour contrast in both colour schemes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
attr(<attr-name>, <type-or-unit>?, <fallback>?)
light-dark(<color>, <color>)
light-dark(<image>, <image>)
```

#### Component Breakdown

| Function | Arguments | Description |
|---|---|---|
| `attr()` | attribute name, optional type/unit, optional fallback | Reads an attribute value. |
| `light-dark()` | two colours or two images | Returns based on `color-scheme`. |

#### Syntax Rules

1. `attr()` can be used on any property, but support for properties other than `content` is experimental.
2. The `type-or-unit` parameter specifies how the attribute value is parsed (e.g., `color`, `length`, `angle`).
3. If the attribute is missing or invalid, the fallback is used.
4. `light-dark()` requires `color-scheme: light dark` on the root element to work correctly.
5. `light-dark()` returns the first value when the scheme is light or unknown, and the second when dark.

#### Constraints and Limitations

- **`attr()` browser support** — full typed support is experimental; only string values in `content` are widely supported.
- **Security considerations** — `attr()` can expose attribute values; avoid using it with sensitive data.
- **`light-dark()` support** — Baseline 2024; requires `color-scheme` to be set.
- **Image form** — mixing an image and a colour in `light-dark()` is a parse error.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `attr()` for Dynamic Content

**HTML File (`attr-content.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>attr() for Content</title>
    <link rel="stylesheet" href="attr-content.css">
</head>
<body>
    <a href="https://example.com" data-label="Example Website">Visit</a>
    <div class="tooltip" data-tooltip="This is a tooltip">Hover me</div>
</body>
</html>
```

**CSS File (`attr-content.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    padding: 2rem;
    background-color: #f5f5f5;
}

/* Link with attr() content */
a::after {
    /* Append the data-label attribute value after the link text */
    content: " (" attr(data-label) ")";
    color: #7f8c8d;
    font-size: 0.9em;
}

/* Tooltip using attr() */
.tooltip {
    position: relative;
    display: inline-block;
    padding: 0.5rem 1rem;
    background-color: #3498db;
    color: white;
    border-radius: 4px;
    cursor: help;
    margin-top: 2rem;
}

.tooltip::after {
    /* Use the data-tooltip attribute as the tooltip content */
    content: attr(data-tooltip);
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    background-color: #2c3e50;
    color: white;
    padding: 0.5rem;
    border-radius: 4px;
    white-space: nowrap;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.2s;
}

.tooltip:hover::after {
    opacity: 1;
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser.
2. Observe the link shows the label from `data-label`.
3. Hover over the tooltip element to see the content from `data-tooltip`.

**Expected Output:** A link with its label appended, and a tooltip that appears on hover.

**Why This Works:** `content: attr(data-label)` reads the `data-label` attribute and inserts its value as generated content. This keeps the content in HTML where it belongs, while the presentation is controlled by CSS.

---

### Real-World Cases

- **Tooltips:** Using `attr(data-tooltip)` for hover tooltips.
- **Accessible labels:** Using `attr(aria-label)` to surface accessible names visually.
- **Dynamic theming:** Using `light-dark()` for automatic colour-scheme switching.
- **Print styles:** Using `attr(href)` to show URLs in printed documents.

---

## 6. Color Functions

### Definitions

**Core Definition:** CSS colour functions generate colours in various colour spaces. Legacy functions include `rgb()`, `rgba()`, `hsl()`, and `hsla()`. Modern device-independent functions include `lab()`, `lch()`, `oklab()`, and `oklch()`. `color-mix()` blends colours, and relative colour syntax derives colours from a base colour.

**Technical Definition:** CSS Color Level 4 and Level 5 define these functions. `rgb()` and `hsl()` operate in sRGB. `lab()` and `lch()` use the CIELAB colour space; `oklab()` and `oklch()` use the Oklab perceptual colour space, which prevents hue shifts during brightness modifications. Relative colour syntax (`rgb(from <color> r g b)`) allows channel-level manipulation of an origin colour. `color-mix(in <space>, <color> <percentage>, <color> <percentage>)` interpolates between two colours in a specified space.

**Beginner-Friendly Explanation:** Colour functions let you define colours in different ways. `rgb(255, 0, 0)` is red in the classic way. `oklch(0.7 0.15 30)` is red in a modern perceptual space that keeps colours looking consistent when you change brightness. `color-mix()` blends two colours. Relative colour syntax lets you say "take this blue, make it 20% lighter, and shift the hue by 30 degrees."

---

### Purposes

- To define colours in various colour spaces for different design needs.
- To create perceptually uniform colour palettes using OKLCH.
- To blend colours natively with `color-mix()`.
- To generate entire themes from a single base colour using relative colour syntax.
- To maintain hue consistency during brightness changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Legacy */
rgb(R, G, B)
rgba(R, G, B, A)
hsl(H, S, L)
hsla(H, S, L, A)

/* Modern device-independent */
lab(L, a, b)
lch(L, C, H)
oklab(L, a, b)
oklch(L, C, H)

/* Relative colour syntax */
rgb(from <color> r g b)
hsl(from <color> h s l)
oklch(from <color> l c h)

/* Colour mixing */
color-mix(in <space>, <color> <percentage>, <color> <percentage>)
```

#### Component Breakdown

| Function | Space | Description |
|---|---|---|
| `rgb()` | sRGB | Red, green, blue (0–255). |
| `hsl()` | sRGB | Hue, saturation, lightness. |
| `lab()` | CIELAB | Lightness, a-axis, b-axis. |
| `lch()` | CIELAB | Lightness, chroma, hue. |
| `oklab()` | Oklab | Perceptually uniform Lab. |
| `oklch()` | Oklab | Perceptually uniform LCH. |
| `color-mix()` | Any | Blends two colours. |
| Relative syntax | Any | Derives from a base colour. |

#### Syntax Rules

1. `rgb()` and `hsl()` are the most widely supported; `lab()`, `lch()`, `oklab()`, `oklch()` are Baseline 2023.
2. `oklch()` and `oklab()` prevent hue shifts when lightness changes, unlike HSL.
3. `color-mix()` requires a colour space (`in oklab`, `in lch`, etc.) and two colours with optional percentages.
4. Relative colour syntax uses `from <origin-color>` followed by channel values.
5. Alpha is specified with a slash (`/`): `oklch(0.7 0.15 30 / 0.5)`.

#### Constraints and Limitations

- **Gamut** — `lab()` and `lch()` can produce colours outside the sRGB gamut.
- **Browser support** — modern functions are Baseline 2023; relative colour syntax is newer.
- **Hue interpolation** — `color-mix()` hue interpolation can be `shorter`, `longer`, `increasing`, or `decreasing`.
- **Fallback** — always provide a fallback for older browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `oklch()` for Perceptual Colour

**HTML File (`oklch.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>oklch() Colour</title>
    <link rel="stylesheet" href="oklch.css">
</head>
<body>
    <div class="swatch original">Original</div>
    <div class="swatch lighter">Lighter (oklch)</div>
    <div class="swatch hsl-lighter">Lighter (hsl)</div>
</body>
</html>
```

**CSS File (`oklch.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    display: flex;
    gap: 1rem;
    padding: 2rem;
    background-color: #f5f5f5;
}

/* Shared swatch styling */
.swatch {
    width: 150px;
    height: 150px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    border-radius: 12px;
    text-align: center;
    padding: 1rem;
}

/* Original blue in oklch */
.original {
    background-color: oklch(0.6 0.15 250);
}

/* Lighter version using oklch — hue stays consistent */
.lighter {
    background-color: oklch(0.8 0.15 250);
}

/* Lighter version using hsl — hue may shift */
.hsl-lighter {
    background-color: hsl(220, 70%, 70%);
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser.
2. Compare the three swatches.

**Expected Output:** The oklch lighter swatch appears as a consistent lighter version of the original blue, while the HSL lighter swatch may appear slightly different in hue.

**Why This Works:** `oklch()` uses a perceptually uniform colour space. Increasing lightness (`L`) from 0.6 to 0.8 while keeping chroma (`C`) and hue (`H`) constant produces a lighter version of the same colour without hue shift. HSL does not guarantee hue consistency when lightness changes.

---

#### Example 2: `color-mix()` and Relative Colour Syntax

**HTML File (`color-mix.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>color-mix() and Relative Colors</title>
    <link rel="stylesheet" href="color-mix.css">
</head>
<body>
    <div class="base">Base Colour</div>
    <div class="mixed">color-mix(in oklch, base, white 30%)</div>
    <div class="relative">Relative: oklch(from base l c h + 180)</div>
</body>
</html>
```

**CSS File (`color-mix.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    display: flex;
    gap: 1rem;
    padding: 2rem;
    background-color: #f5f5f5;
    flex-wrap: wrap;
}

/* Shared styling */
div {
    width: 200px;
    height: 150px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    border-radius: 12px;
    text-align: center;
    padding: 1rem;
}

/* Base colour */
.base {
    background-color: oklch(0.6 0.15 250);
}

/* Mixed with white */
.mixed {
    /* Mix 70% of the base colour with 30% white in oklch space */
    background-color: color-mix(in oklch, oklch(0.6 0.15 250) 70%, white 30%);
}

/* Relative colour: shift hue by 180 degrees */
.relative {
    /* Derive from the base colour, add 180 to the hue channel */
    background-color: oklch(from oklch(0.6 0.15 250) l c calc(h + 180));
}
```

**Step-by-Step Setup Guide:**

1. Save the files and open in a browser.
2. Observe the three swatches: base blue, lighter mixed blue, and a complementary colour.

**Expected Output:** Three swatches: the original blue, a lighter blue-white blend, and a complementary (roughly yellow/orange) colour.

**Why This Works:** `color-mix(in oklch, ...)` blends the base colour with white in the perceptually uniform oklch space, producing a smooth, natural lightening. Relative colour syntax (`oklch(from ... l c calc(h + 180))`) takes the base colour's lightness and chroma, and rotates the hue by 180 degrees, producing the complementary colour.

---

### Real-World Cases

- **Design systems:** Using `oklch()` for perceptually uniform colour scales.
- **Theming:** Using relative colour syntax to generate hover states, borders, and shadows from a single base colour.
- **Palette generation:** Using `color-mix()` to create tints and shades without manual colour picking.
- **Dark mode:** Using `light-dark()` and `color-mix()` together for accessible dark-mode colours.

---

## References

- W3C — CSS Values and Units Module Level 4 - https://www.w3.org/TR/css-values-4/
- W3C — CSS Grid Layout Module Level 1 - https://www.w3.org/TR/css-grid-1/
- W3C — CSS Grid Layout Module Level 2 - https://www.w3.org/TR/css-grid-2/
- W3C — CSS Custom Properties for Cascading Variables Module Level 1 - https://www.w3.org/TR/css-variables-1/
- W3C — CSS Environment Variables Module Level 1 - https://drafts.csswg.org/css-env-1/
- W3C — CSS Color Module Level 5 - https://www.w3.org/TR/css-color-5/
- MDN Web Docs — CSS Functions - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Functions
- MDN Web Docs — `calc()` - https://developer.mozilla.org/en-US/docs/Web/CSS/calc
- MDN Web Docs — `minmax()` - https://developer.mozilla.org/en-US/docs/Web/CSS/minmax
- MDN Web Docs — `fit-content()` - https://developer.mozilla.org/en-US/docs/Web/CSS/fit-content_function
- MDN Web Docs — `repeat()` - https://developer.mozilla.org/en-US/docs/Web/CSS/repeat
- MDN Web Docs — `round()` - https://developer.mozilla.org/en-US/docs/Web/CSS/round
- MDN Web Docs — `mod()` - https://developer.mozilla.org/en-US/docs/Web/CSS/mod
- MDN Web Docs — `rem()` - https://developer.mozilla.org/en-US/docs/Web/CSS/rem
- MDN Web Docs — `var()` - https://developer.mozilla.org/en-US/docs/Web/CSS/var
- MDN Web Docs — `env()` - https://developer.mozilla.org/en-US/docs/Web/CSS/env
- MDN Web Docs — `attr()` - https://developer.mozilla.org/en-US/docs/Web/CSS/attr
- MDN Web Docs — `light-dark()` - https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/light-dark
- MDN Web Docs — `color-mix()` - https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix
- MDN Web Docs — Using relative colours - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_colors/Relative_colors
- CSS-Tricks — `env()` - https://css-tricks.com/almanac/functions/e/env/