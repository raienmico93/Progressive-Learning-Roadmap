# CSS Numeric Values — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS numeric values are the fundamental data types used to express quantities, measurements, and mathematical relationships within CSS property declarations. They encompass integers, real numbers, percentages, dimensions (numbers with units), ratios, angles, durations, and calculated expressions. These values form the quantitative backbone of CSS, governing everything from element sizing and positioning to animation timing and colour manipulation.

**Technical Definition:** In formal CSS terms, numeric values are defined in the CSS Values and Units Module (Levels 3 and 4) as a set of typed data types including `<integer>`, `<number>`, `<dimension>`, `<percentage>`, `<ratio>`, and the math functions (`calc()`, `min()`, `max()`, `clamp()`) that operate upon them. Each data type has a precise literal grammar, a defined computation model, and rules for interpolation, addition, and type compatibility. The CSS type system is strict: operations that mix incompatible types (such as adding a `<length>` to an `<angle>`) are invalid and cause the entire declaration to be ignored.

**Beginner-Friendly Explanation:** Think of CSS numeric values as the numbers, measurements, and formulas you use to describe how a web page should look. Some values are plain numbers like `3` or `1.5` (used for things like `line-height` or `opacity`). Some are measurements with units like `16px` or `2em`. Some are percentages like `50%`. Some are ratios like `16 / 9` for aspect ratios. And some are calculated expressions like `calc(100% - 20px)` that let you mix different units in a single formula. The key thing to understand is that CSS is strict about which types are allowed for which properties — you cannot use a time value like `2s` for a `width` property, and you cannot add a pixel value to a degree value.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Strict typing** | Each property defines which numeric data types it accepts; mismatched types cause the declaration to be invalid. |
| **Decimal notation only** | CSS numeric literals are written in decimal notation; hexadecimal, octal, and binary literals are not supported for numeric values. |
| **Unit suffix** | Dimensions attach a unit identifier directly to the number with no whitespace (e.g., `10px`, `2em`, `90deg`). |
| **Unitless zero exception** | For `<length>` and `<angle>` types, a literal `0` may omit its unit; this exception does not apply to `<time>`, `<frequency>`, or other dimension types. |
| **Percentage relativity** | Every property that accepts a percentage defines the reference quantity to which that percentage resolves (often a parent dimension, but sometimes the element's own property or the formatting context). |
| **Calculation-aware** | Math functions can combine values of compatible types, performing type checking at parse time and resolving to a single canonical unit. |
| **Infinite precision (theoretical)** | CSS theoretically supports infinite precision and range; implementations impose practical finite limits. |

---

### Prerequisites

Before studying CSS Numeric Values, you should understand:

- **Basic CSS syntax** — selectors, properties, values, declarations, and the cascade.
- **The CSS box model** — how `width`, `height`, `margin`, and `padding` interact.
- **Inheritance and the computed value** — how values propagate and resolve.
- **Basic arithmetic** — addition, subtraction, multiplication, division, and operator precedence.

---

### Related Programming Areas

- **CSS Layout** — sizing, positioning, flexbox, and grid all rely heavily on numeric values.
- **CSS Animations and Transitions** — `<time>` values control durations and delays.
- **CSS Transforms** — `<angle>` values drive rotations and skews.
- **Responsive Design** — percentages, viewport units, and `calc()` enable fluid layouts.
- **Colour Manipulation** — numeric values appear in `rgb()`, `hsl()`, and relative colour functions.
- **Accessibility** — respecting user font-size preferences via relative units.

---

### Core Concepts / Features

1. Integers and Decimals
2. Percentages
3. Ratios and Dimensions (Angles, Time)
4. Calculated Values

---

## 1. Integers and Decimals

### Definitions

**Core Definition:** Integers and decimals are the foundational numeric primitives in CSS — raw numbers without units. They form the basis from which all other numeric types (dimensions, percentages) are constructed.

**Technical Definition:** An `<integer>` is denoted by one or more decimal digits `0` through `9`, optionally preceded by a `-` or `+` sign. A `<number>` (also called a real number) is either an integer or zero or more decimal digits followed by a dot (`.`) followed by one or more decimal digits, optionally followed by an exponent composed of `e` or `E` and an integer. Both correspond to the `<number-token>` production in the CSS Syntax Module. The value `<zero>` represents a literal number with the value 0; expressions that merely evaluate to 0 (e.g., `calc(0)`) do not match `<zero>` — only literal number tokens do.

**Beginner-Friendly Explanation:** Integers are whole numbers like `3`, `-12`, or `0`. Decimals are numbers with a fractional part like `1.5`, `0.75`, or `-2.25`. These are the raw numbers of CSS — they have no units attached. You use them for properties that expect a plain number, such as `line-height: 1.6`, `opacity: 0.5`, or `z-index: 10`. The important thing to remember is that a number like `5` is not the same as `5px` — CSS treats them as completely different types, and most properties only accept one or the other.

---

### Purposes

- To provide unitless quantities for properties such as `line-height`, `opacity`, `z-index`, `flex-grow`, and `font-weight` (in the numeric range 1–1000).
- To serve as the raw numeric component from which `<dimension>` and `<percentage>` values are constructed.
- To enable relative scaling relationships without tying values to specific measurement units.
- To express ratios and multipliers in mathematical calculations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Integer literal */
<integer>  /* e.g., 0, 42, -17, +8 */

/* Number literal (integer or real) */
<number>   /* e.g., 0, 1.5, -3.14, 2.5e3, 0.5 */
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| Digits `0–9` | The decimal digits that form the number. | `1024` |
| Sign `+` or `-` | Optional; immediately precedes the first digit with no space. | `-55`, `+3` |
| Decimal point `.` | Separates integer and fractional parts; followed by one or more digits. | `1.5` |
| Exponent `e` or `E` | Indicates scientific notation; followed by an integer exponent. | `2.5e3` (2500) |

#### Syntax Rules

1. Integer values consist of **one or more decimal digits**; leading zeros are permitted but redundant (`007` is valid).
2. A `<number>` may be an integer or a real number; the decimal point must be followed by at least one digit (`1.` is invalid; `1.0` is valid).
3. Signs must immediately precede the first digit with **no whitespace**.
4. Scientific notation uses `e` or `E` followed by an integer (positive or negative). e.g., `1e3` = 1000, `1e-3` = 0.001.
5. A literal `0` is a valid `<zero>` and may be used where a `<length>` or `<angle>` is expected (the **unitless zero exception**). This exception does **not** apply to `<time>`, `<frequency>`, or `<resolution>` values.

#### Constraints and Limitations

- **Range limitations** — CSS theoretically supports infinite precision, but implementations impose finite bounds. Values outside a property's allowed range cause the declaration to be ignored.
- **Integer truncation** — When an `<integer>` is expected and a `<number>` is provided via `calc()`, the result is rounded to the nearest integer; exactly 0.5 rounds toward positive infinity (`calc(1.5)` → `2`; `calc(-1.5)` → `-1`).
- **No unit arithmetic in plain values** — You cannot write `10px + 5` outside of a math function; raw arithmetic is only permitted inside `calc()` and related functions.
- **Unitless zero scope** — The ability to omit units after zero applies **only to lengths and angles**, not to times or other dimensions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Unitless Values in Practice

**HTML File (`unitless.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Unitless Numeric Values</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="unitless.css">
</head>
<body>
    <!-- Paragraph with unitless line-height and opacity -->
    <p class="intro">
        This paragraph uses a unitless <code>line-height</code> of 1.7, which means
        the line spacing is 1.7 times the font size. It also has a slightly
        reduced opacity of 0.9.
    </p>
    <!-- Box with a unitless z-index and flex-grow -->
    <div class="container">
        <div class="box box-1">Box 1 (flex-grow: 2)</div>
        <div class="box box-2">Box 2 (flex-grow: 1)</div>
    </div>
    <!-- Element with unitless opacity -->
    <div class="faded">This element has opacity: 0.6</div>
</body>
</html>
```

**CSS File (`unitless.css`):**

```css
/* Body base styles */
body {
    font-family: system-ui, sans-serif;
    margin: 20px;
}

/* Unitless line-height: relative to the element's font size */
.intro {
    font-size: 16px;
    /* 1.7 means 1.7 × 16px = 27.2px line spacing */
    line-height: 1.7;
    /* Opacity is always unitless; 0 is transparent, 1 is opaque */
    opacity: 0.9;
    margin-bottom: 20px;
}

/* Flex container to demonstrate unitless flex-grow */
.container {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

.box {
    padding: 12px;
    background-color: #3498db;
    color: white;
    border-radius: 4px;
}

/* flex-grow: 2 means this box takes twice the available space */
.box-1 {
    flex-grow: 2;
}

/* flex-grow: 1 means this box takes one share of the available space */
.box-2 {
    flex-grow: 1;
}

/* Opacity with a decimal value */
.faded {
    opacity: 0.6;
    background-color: #e74c3c;
    color: white;
    padding: 12px;
    border-radius: 4px;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML as `unitless.html` and the CSS as `unitless.css` in the same folder.
3. Open `unitless.html` in a web browser.
4. Observe the paragraph's comfortable line spacing and slightly faded appearance.
5. Observe that Box 1 is wider than Box 2 due to `flex-grow: 2` versus `flex-grow: 1`.
6. Observe the red box's semi-transparent appearance from `opacity: 0.6`.

**Expected Output:** A page with a readable paragraph (line-height 1.7, opacity 0.9), a flex row where Box 1 is twice as wide as Box 2, and a red box that appears slightly see-through.

**Why This Works:** Each property accepts a `<number>` (not a `<dimension>`). `line-height: 1.7` is interpreted as a multiplier of the font size. `opacity: 0.9` and `opacity: 0.6` are direct decimal values between 0 and 1. `flex-grow: 2` and `flex-grow: 1` are unitless ratios that determine how available space is distributed. No units are involved because these properties are defined to accept bare numbers.

---

#### Example 2: The Unitless Zero Exception and Its Limits

**HTML File (`zero.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Unitless Zero Exception</title>
    <link rel="stylesheet" href="zero.css">
</head>
<body>
    <!-- Length properties accept unitless zero -->
    <div class="zero-length">
        This box has <code>margin: 0</code> and <code>padding: 0</code> — the units
        are omitted because zero is zero regardless of the length unit.
    </div>
    <!-- Time property requires a unit even for zero -->
    <div class="zero-time">
        This box has a transition duration specified as <code>0s</code>. Writing
        just <code>0</code> would be invalid for a time value.
    </div>
</body>
</html>
```

**CSS File (`zero.css`):**

```css
/* Body styling */
body {
    font-family: system-ui, sans-serif;
    background-color: #f0f0f0;
    padding: 20px;
}

/* Length zero: units may be omitted */
.zero-length {
    /* margin: 0 is valid — unitless zero is allowed for lengths */
    margin: 0;
    /* padding: 0 is valid — unitless zero is allowed for lengths */
    padding: 0;
    background-color: #2ecc71;
    color: white;
    /* border-width: 0 is valid — unitless zero is allowed for lengths */
    border: 0 solid black;
}

/* Time zero: units MUST be present */
.zero-time {
    background-color: #e67e22;
    color: white;
    padding: 12px;
    margin-top: 16px;
    /* transition-duration: 0 would be INVALID for a <time> value */
    transition-duration: 0s;
    /* transition-delay: 0 is also invalid; 0ms or 0s is required */
    transition-delay: 0s;
    /* Hover effect to demonstrate the transition */
    transition-property: background-color;
}

.zero-time:hover {
    background-color: #d35400;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `zero.html` and the CSS as `zero.css`.
2. Open `zero.html` in a browser.
3. Observe that the green box has no margin or padding (the zero values take effect).
4. Hover over the orange box and observe the smooth background-colour transition — this works because `0s` is a valid `<time>` value.
5. If you changed `transition-duration: 0s` to `transition-duration: 0`, the transition would break entirely because unitless zero is invalid for time values.

**Expected Output:** A green box flush against the container edges (zero margin and padding) and an orange box that smoothly changes colour on hover.

**Why This Works:** The CSS specification permits the unit to be omitted after a zero **length** value. The green box's `margin: 0`, `padding: 0`, and `border: 0` are all length properties, so the unitless zero is valid. The orange box's `transition-duration` is a `<time>` property, so the unit is mandatory — `0s` is correct, `0` would be invalid.

---

### Real-World Cases

- **Flexbox layouts:** Using unitless `flex-grow`, `flex-shrink`, and `flex-basis` values to create responsive columns.
- **Typography systems:** Using unitless `line-height` (e.g., `1.5` or `1.6`) so that line spacing scales proportionally with font size.
- **Overlay effects:** Using unitless `opacity` values between 0 and 1 for modal overlays, image hover effects, and disabled states.
- **Z-index stacking:** Using unitless integers to control the stacking order of positioned elements.

---

## 2. Percentages

### Definitions

**Core Definition:** A percentage in CSS is a numeric value followed by a percent sign (`%`) that represents a fraction of another reference value. The reference value is defined individually by each property that accepts percentages.

**Technical Definition:** The `<percentage>` type is denoted by a `<number>` immediately followed by a `%` sign, corresponding to the `<percentage-token>` production in the CSS Syntax Module. Percentage values are always relative to another quantity; each property that allows percentages defines the quantity to which the percentage refers. This reference quantity may be a value of another property for the same element, a property value for an ancestor element, a measurement of the formatting context (e.g., the width of a containing block), or something else entirely. Unless otherwise specified (as in `font-size`, where percentages compute to `<length>`), the computed value of a percentage is the specified percentage itself.

**Beginner-Friendly Explanation:** A percentage is a "relative" value — it means "this much out of 100 parts of something else." For example, `width: 50%` means "half the width of the parent container." `font-size: 150%` means "one and a half times the parent's font size." The tricky part is that "something else" (the reference value) is different for every property. For `width`, it is the parent's width. For `font-size`, it is the parent's font size. For `line-height`, it is the element's own font size. CSS resolves these references dynamically at layout time, which is why percentages are so powerful for responsive design.

---

### Purposes

- To create fluid, responsive layouts that adapt to their parent container or viewport.
- To express proportions and scaling relationships without hard-coding absolute measurements.
- To enable typographic scaling relative to parent font sizes.
- To define positioning offsets relative to containing blocks.
- To support dynamic sizing in grid and flexbox layouts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
<percentage>  /* e.g., 50%, 100%, -25%, 0%, 12.5% */
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<number>` | The numeric part of the percentage. | `50` in `50%` |
| `%` | The percent sign; must immediately follow the number with no whitespace. | `%` in `50%` |
| Optional sign | `+` or `-` may precede the number; negative percentages are not valid for all properties. | `-10%` |

#### Syntax Rules

1. The `%` sign must **immediately follow** the number with no space: `50%` is valid; `50 %` is invalid.
2. Negative percentages are permitted syntactically but may be invalid for certain properties (e.g., `width: -50%` is invalid).
3. The reference quantity for a percentage is **defined by the property**, not by the percentage itself.
4. For inherited properties, only the **computed value** is inherited; if a percentage computes to a length (as in `font-size`), the inherited value is the computed length, not the percentage.
5. Percentages can be combined with dimensions inside `calc()` when the property grammar permits a combined type such as `<length-percentage>`.

#### Constraints and Limitations

- **Property-specific reference** — You cannot assume that `50%` means the same thing in every property. The reference may be a parent dimension, the element's own dimension, or something else.
- **Circular dependency** — If a percentage width depends on the parent's width, and the parent's width depends on the child's width (e.g., in a shrink-to-fit context), the browser must resolve the dependency, which can lead to unexpected results.
- **No percentage for all properties** — Only properties whose grammar explicitly includes `<percentage>` (or a compatible combined type) accept percentage values.
- **Inheritance of computed values** — When a percentage is inherited, the child receives the computed value (often a resolved length), not the original percentage expression.
- **Range restrictions** — Some properties restrict percentages to non-negative values or to a specific range (e.g., `0%` to `100%`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Percentages in Layout and Typography

**HTML File (`percentages.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Percentage Values in CSS</title>
    <link rel="stylesheet" href="percentages.css">
</head>
<body>
    <!-- Container with a defined width to serve as the percentage reference -->
    <div class="container">
        <!-- Box with 50% width and 20% left margin -->
        <div class="box box-1">
            width: 50%, margin-left: 20%
        </div>
        <!-- Box with 30% width and 60% left margin -->
        <div class="box box-2">
            width: 30%, margin-left: 60%
        </div>
    </div>

    <!-- Typography example: font-size percentages relative to the parent -->
    <div class="text-container">
        <p>Full-size text (18px base)</p>
        <p><span class="half">50% (9px)</span></p>
        <p><span class="double">200% (36px)</span></p>
    </div>
</body>
</html>
```

**CSS File (`percentages.css`):**

```css
/* Body base styles */
body {
    font-family: system-ui, sans-serif;
    margin: 20px;
}

/* Container with a fixed width — this is the reference for child percentages */
.container {
    background-color: navy;
    padding: 10px;
    /* Width is set so the 50% and 30% child widths have a definite reference */
    width: 600px;
    margin-bottom: 30px;
}

/* Box 1: 50% of the container's width */
.box-1 {
    /* Width resolves to 50% of 600px = 300px */
    width: 50%;
    /* Margin-left resolves to 20% of 600px = 120px */
    margin-left: 20%;
    background-color: chartreuse;
}

/* Box 2: 30% of the container's width */
.box-2 {
    /* Width resolves to 30% of 600px = 180px */
    width: 30%;
    /* Margin-left resolves to 60% of 600px = 360px */
    margin-left: 60%;
    background-color: pink;
}

/* Shared box styling */
.box {
    padding: 8px;
    margin-bottom: 8px;
    color: #000;
    font-weight: 500;
}

/* Text container with a base font size */
.text-container {
    /* This 18px is the reference for child font-size percentages */
    font-size: 18px;
    line-height: 1.5;
}

/* 50% of the parent font size (18px) = 9px */
.half {
    font-size: 50%;
    /* Font-size percentages compute to a length, so the inherited value is 9px */
}

/* 200% of the parent font size (18px) = 36px */
.double {
    font-size: 200%;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `percentages.html` and the CSS as `percentages.css`.
2. Open `percentages.html` in a browser.
3. Observe the navy container with two boxes: Box 1 (chartreuse) is 300px wide with a 120px left margin; Box 2 (pink) is 180px wide with a 360px left margin.
4. Observe the three text lines: the second is half the size of the first, and the third is double the size of the first.

**Expected Output:** A navy container containing a bright green box that takes up half the width and a pink box that takes up a third of the width, each offset by different percentage margins. Below, three lines of text at 18px, 9px, and 36px respectively.

**Why This Works:** The `.container` has a fixed `width: 600px`, which becomes the reference value for all percentage-based properties on its children. `width: 50%` on `.box-1` resolves to 300px; `margin-left: 20%` resolves to 120px. For the typography example, `.text-container` has `font-size: 18px`, which becomes the reference for child `font-size` percentages. `50%` resolves to 9px, and `200%` resolves to 36px. This demonstrates that the reference value is property-specific: width percentages refer to parent width, while font-size percentages refer to parent font-size.

---

#### Example 2: The Computed Value Inheritance Rule

**HTML File (`inheritance.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Percentage Inheritance and Computed Values</title>
    <link rel="stylesheet" href="inheritance.css">
</head>
<body>
    <!-- Parent with a percentage font-size -->
    <div class="parent">
        Parent: font-size 150% of 16px = 24px
        <!-- Child inherits the computed value (24px), not the percentage -->
        <div class="child">
            Child: inherited font-size (24px), then applied 50% of that = 12px
        </div>
    </div>
</body>
</html>
```

**CSS File (`inheritance.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    font-size: 16px;  /* Root reference for the parent */
    margin: 20px;
}

/* Parent: font-size is 150% of the body's 16px = 24px */
.parent {
    font-size: 150%;
    background-color: #ecf0f1;
    padding: 16px;
    border-left: 4px solid #3498db;
}

/* Child: font-size is 50% of the inherited computed value (24px) = 12px.
   IMPORTANT: The child does NOT inherit "150%" — it inherits the computed
   value 24px, and then the 50% is applied to that. */
.child {
    font-size: 50%;
    background-color: #d5dbdb;
    padding: 12px;
    margin-top: 8px;
    border-left: 3px solid #e74c3c;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `inheritance.html` and the CSS as `inheritance.css`.
2. Open `inheritance.html` in a browser.
3. Observe that the parent text is noticeably larger than the body default (24px vs 16px).
4. Observe that the child text is smaller than the parent (12px) — it inherited the **computed** value of 24px, then applied its own 50% to that.

**Expected Output:** A light grey parent box with text at 24px, containing a darker grey child box with text at 12px.

**Why This Works:** When `font-size: 150%` is applied to `.parent`, the browser computes this to 24px (since the body's font-size is 16px). The child element inherits the **computed value** (24px), not the specified percentage (150%). When the child applies `font-size: 50%`, that 50% is relative to the inherited 24px, resulting in 12px. This is a critical behaviour to understand: **percentage values for font-size compute to lengths, and only computed values are inherited.**

---

### Real-World Cases

- **Responsive grid layouts:** Using `width: 33.333%` for three-column layouts that adapt to the container width.
- **Fluid typography:** Using `font-size: 120%` on headings to scale relative to the parent text.
- **Centering with transforms:** Using `left: 50%` combined with `transform: translateX(-50%)` for horizontal centering.
- **Padding and margin systems:** Using percentage-based spacing that scales with the containing block's width.
- **Progress bars:** Using `width: 75%` to visually represent completion status.

---

## 3. Ratios and Dimensions (Angles, Time)

### Definitions

**Core Definition:** Ratios express proportional relationships between two quantities, while dimensions are numbers with units attached. In CSS, `<ratio>` represents a width-to-height relationship (used primarily in `aspect-ratio`), and `<angle>` and `<time>` are dimension types used for transforms, gradients, animations, and transitions.

**Technical Definition:** A `<dimension>` is a `<number>` immediately followed by a unit identifier (an `<ident>`), corresponding to the `<dimension-token>` production. Like keywords, unit identifiers are ASCII case-insensitive. CSS uses dimensions to specify distances (`<length>`), durations (`<time>`), angles (`<angle>`), frequencies (`<frequency>`), and resolutions (`<resolution>`). A `<ratio>` is a special value type representing a proportion, written as `<number> / <number>` or as a single `<number>`. The `<angle>` type includes `deg`, `grad`, `rad`, and `turn` units; the `<time>` type includes `s` and `ms` units. All units within each compatible group are mutually convertible, with a canonical unit defined for serialisation.

**Beginner-Friendly Explanation:** A ratio is like saying "this is twice as wide as it is tall" — written in CSS as `2 / 1` or just `2`. Dimensions are measurements with a unit, like `90deg` (an angle), `0.3s` (a duration), or `16px` (a length). Angles are used for rotations and gradients; times are used for animations and transitions. Think of a ratio as a recipe for proportions, and a dimension as a measurement with a label attached.

---

### Purposes

- To define aspect ratios for responsive boxes that maintain their shape as they resize.
- To control rotation and skew angles in CSS transforms.
- To define gradient directions.
- To specify animation and transition durations and delays.
- To express timing functions and delays in milliseconds for precise control.
- To enable mathematical relationships between width and height without JavaScript.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Ratio */
<ratio>            /* e.g., 1 / 1, 16 / 9, 0.5, 4 / 3 */

/* Angle */
<angle>            /* e.g., 90deg, 1.5708rad, 100grad, 0.25turn */

/* Time */
<time>             /* e.g., 0.3s, 300ms, 1.5s */
```

#### Component Breakdown

| Type | Units | Description | Full Circle / Conversion |
|---|---|---|---|
| `<ratio>` | (none) | `width / height` or a single number | `16 / 9`, `1`, `0.5` |
| `<angle>` — `deg` | Degrees | 360 degrees in a full circle | `90deg` |
| `<angle>` — `grad` | Gradians | 400 gradians in a full circle | `100grad` |
| `<angle>` — `rad` | Radians | 2π radians in a full circle | ≈ `1.5708rad` |
| `<angle>` — `turn` | Turns | 1 turn in a full circle | `0.25turn` |
| `<time>` — `s` | Seconds | Base unit of time | `1.5s` |
| `<time>` — `ms` | Milliseconds | 1000 ms = 1 s | `300ms` |

#### Syntax Rules

1. **Ratio:** Written as `<number> / <number>` or as a single `<number>` (which is equivalent to `<number> / 1`). The slash may be surrounded by optional whitespace.
2. **Angle:** A `<number>` immediately followed by the unit identifier. No space between the number and unit: `90 deg` is invalid. A unitless `0` is permitted for legacy angle uses but is not recommended.
3. **Time:** A `<number>` immediately followed by `s` or `ms`. **Unitless zero is invalid for `<time>`** — `0` does not work; `0s` or `0ms` is required.
4. All units within a compatible group are mutually convertible: `90deg = 100grad = 0.25turn ≈ 1.5708rad`. The canonical unit for angles is `deg`; for times it is `s`.
5. Angle values are interpreted as **bearing angles**: `0deg` is "up" or "north", with larger angles rotating clockwise (so `90deg` is "right" or "east").
6. Time values may be negative for certain properties (e.g., `animation-delay`), but must still carry a unit.

#### Constraints and Limitations

- **Ratio limitations:** The `aspect-ratio` property requires at least one dimension to be `auto` for the ratio to have an effect. If both `width` and `height` are explicitly set, the ratio is ignored.
- **Angle legacy exception:** Some legacy properties accept a bare `0` to mean `0deg`; this is not universal and will not apply to future uses of `<angle>`.
- **Time unitless zero prohibition:** Unlike lengths and angles, `<time>` values **never** permit a unitless zero. `transition-duration: 0` is invalid; `transition-duration: 0s` is required.
- **Case sensitivity:** Unit identifiers are case-insensitive (`MS`, `Ms`, `mS`, `ms` all work), but lowercase is conventional and recommended.
- **Negative ratios:** The `<ratio>` type must be non-negative; negative ratios are invalid.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Aspect Ratio for Responsive Media

**HTML File (`ratio.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aspect Ratio Example</title>
    <link rel="stylesheet" href="ratio.css">
</head>
<body>
    <!-- 16:9 video-style box -->
    <div class="video-box">
        <p>16:9 Aspect Ratio</p>
    </div>
    <!-- 1:1 square box -->
    <div class="square-box">
        <p>1:1 Aspect Ratio</p>
    </div>
    <!-- 4:3 classic TV-style box -->
    <div class="classic-box">
        <p>4:3 Aspect Ratio</p>
    </div>
</body>
</html>
```

**CSS File (`ratio.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    display: flex;
    gap: 20px;
    padding: 20px;
    flex-wrap: wrap;
}

/* 16:9 video box — width is set, height is auto (derived from ratio) */
.video-box {
    /* Width is 100% of the flex item's available space */
    width: 300px;
    /* aspect-ratio forces height = width / (16/9) = 168.75px */
    aspect-ratio: 16 / 9;
    background-color: #2c3e50;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
}

/* 1:1 square box */
.square-box {
    width: 200px;
    /* aspect-ratio: 1 means height = width */
    aspect-ratio: 1;
    background-color: #27ae60;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
}

/* 4:3 classic TV box */
.classic-box {
    width: 240px;
    /* aspect-ratio: 4 / 3 means height = width / (4/3) = 180px */
    aspect-ratio: 4 / 3;
    background-color: #8e44ad;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `ratio.html` and the CSS as `ratio.css`.
2. Open `ratio.html` in a browser.
3. Observe the three boxes: a dark blue 16:9 box, a green square, and a purple 4:3 box.
4. Resize the browser window and observe that the boxes maintain their proportions.

**Expected Output:** Three coloured boxes with different aspect ratios displayed side by side. The 16:9 box is wider than it is tall, the square is perfectly square, and the 4:3 box is slightly wider than tall.

**Why This Works:** Each box has a `width` set and `height: auto` (the default). The `aspect-ratio` property derives the height from the width: for the 16:9 box, height = 300px / (16/9) = 168.75px. Because the ratio is specified, the box maintains its shape regardless of how the width changes. This is exactly how video embeds maintain their letterbox shape.

---

#### Example 2: Angles and Time in Animation

**HTML File (`angles-time.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Angles and Time Values</title>
    <link rel="stylesheet" href="angles-time.css">
</head>
<body>
    <!-- Rotating box using deg -->
    <div class="rotate-deg">45deg</div>
    <!-- Rotating box using rad -->
    <div class="rotate-rad">0.7854rad</div>
    <!-- Rotating box using turn -->
    <div class="rotate-turn">0.125turn</div>
    <!-- Animated box using s and ms -->
    <div class="animated-box">Animated</div>
</body>
</html>
```

**CSS File (`angles-time.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    display: flex;
    gap: 30px;
    padding: 40px;
    flex-wrap: wrap;
    align-items: center;
}

/* Shared box styling */
div {
    width: 100px;
    height: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    border-radius: 8px;
    font-size: 14px;
}

/* Rotate 45 degrees clockwise */
.rotate-deg {
    background-color: #e74c3c;
    /* 45deg is the most common angle unit */
    transform: rotate(45deg);
}

/* Rotate by the equivalent of 45 degrees in radians */
.rotate-rad {
    background-color: #3498db;
    /* 0.7854rad ≈ 45deg (π/4 radians) */
    transform: rotate(0.7854rad);
}

/* Rotate by one-eighth of a full turn (45 degrees) */
.rotate-turn {
    background-color: #9b59b6;
    /* 0.125turn = 45deg (1/8 of a full turn) */
    transform: rotate(0.125turn);
}

/* Animated box demonstrating s and ms */
.animated-box {
    background-color: #e67e22;
    /* animation: name duration timing-function delay iteration-count */
    animation: pulse 1.5s ease-in-out 0.3s infinite;
}

/* Keyframes for the pulse animation */
@keyframes pulse {
    0% {
        transform: scale(1);
        background-color: #e67e22;
    }
    50% {
        /* 0.5s = 500ms — both units work identically */
        transform: scale(1.2);
        background-color: #d35400;
    }
    100% {
        transform: scale(1);
        background-color: #e67e22;
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `angles-time.html` and the CSS as `angles-time.css`.
2. Open `angles-time.html` in a browser.
3. Observe the three rotated boxes — all are rotated by the same visual amount (45 degrees), despite using different units.
4. Observe the orange box pulsing continuously — it scales up and down in a 1.5-second cycle, with a 0.3-second delay before each cycle.

**Expected Output:** Three identically rotated boxes in red, blue, and purple, all tilted 45 degrees clockwise. An orange box that continuously pulses (scales up and down) with a smooth ease-in-out motion.

**Why This Works:** The three rotation boxes use equivalent angle values in different units: `45deg`, `0.7854rad` (≈ π/4), and `0.125turn` (1/8 of a full circle). Because all angle units are compatible and convertible, the browser renders all three rotations identically. The animation uses `1.5s` for the duration and `0.3s` for the delay — both are `<time>` values that require units. The `0.5s` inside the keyframes is equivalent to `500ms`, demonstrating that both time units work interchangeably.

---

### Real-World Cases

- **Video players:** Using `aspect-ratio: 16 / 9` for responsive video embeds that maintain their shape.
- **Card components:** Using `aspect-ratio: 1` for square product thumbnails.
- **Loading spinners:** Using `transform: rotate(360deg)` with `animation: spin 1s linear infinite`.
- **UI transitions:** Using `transition-duration: 200ms` for snappy hover effects.
- **Gradient directions:** Using `linear-gradient(135deg, ...)` for diagonal colour transitions.

---

## 4. Calculated Values

### Definitions

**Core Definition:** Calculated values in CSS are expressions evaluated by math functions — primarily `calc()` — that combine multiple values (potentially with different units) into a single result. They enable runtime arithmetic within CSS property values.

**Technical Definition:** The `calc()` function accepts a single mathematical expression as its argument. Operands within the expression may be literal values, other math functions, or expressions such as `var()` that evaluate to a valid argument type. Standard mathematical precedence rules apply: multiplication and division bind more tightly than addition and subtraction. Parentheses can override precedence. Components of a calculation can mix different units, provided the types are compatible (e.g., length with length, angle with angle). Type checking occurs at parse time: if the expression's resolved type does not match the target property's expected type, the declaration is invalid and must be ignored.

**Beginner-Friendly Explanation:** `calc()` is like a built-in calculator for CSS. Instead of writing a fixed value like `300px`, you can write `calc(100% - 20px)`, which means "take the full width and subtract 20 pixels." This is incredibly useful when you need to mix different units — like percentages and pixels — in a single value. You can also use it for simpler things like `calc(2 * 1.5em)` or `calc(100vw / 3)`. The key rules are: you must put spaces around `+` and `-`, you can only add or subtract values of the same type (both lengths, both angles, etc.), and the final result must match what the property expects.

---

### Purposes

- To mix units that cannot normally be combined, such as percentages and absolute lengths.
- To perform runtime arithmetic based on viewport dimensions or container sizes.
- To express values more readably than manually computed decimals (e.g., `calc(100% / 3)` instead of `33.333%`).
- To create fluid layouts that respond to dynamic conditions.
- To compute values that depend on custom properties (`var()`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Basic calc() */
calc(<expression>)

/* Expression with mixed units */
calc(100% - 80px)
calc(100vw / 3 - 2rem)

/* Nested calc() */
calc(calc(2 + 3) * 4)

/* With custom properties */
calc(var(--base-size) * 2)

/* Comparison functions */
min(<value1>, <value2>, ...)
max(<value1>, <value2>, ...)
clamp(<min>, <preferred>, <max>)
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `calc()` | The primary math function; evaluates a single expression. | `calc(100% - 20px)` |
| Operands | Values in the expression: literal numbers, dimensions, percentages, `var()`, or nested functions. | `100%`, `20px`, `var(--gap)` |
| Operators | `+` (addition), `-` (subtraction), `*` (multiplication), `/` (division). | `+`, `-`, `*`, `/` |
| Parentheses | Override default operator precedence. | `(2 + 3) * 4` |
| `min()` | Returns the smallest of the provided values. | `min(50vw, 400px)` |
| `max()` | Returns the largest of the provided values. | `max(200px, 30vw)` |
| `clamp()` | Constrains a preferred value between a minimum and maximum. | `clamp(16px, 4vw, 32px)` |

#### Syntax Rules

1. **Spaces around `+` and `-` are mandatory.** `calc(100%-20px)` is invalid; `calc(100% - 20px)` is valid.
2. **`*` and `/` do not require spaces**, but spaces are conventionally used for readability.
3. **Type compatibility:** Both operands of `+` and `-` must be of the same data type. Adding a `<length>` to an `<angle>` is invalid. Multiplication and division require one operand to be a `<number>`.
4. **Unitless zero prohibition inside calc():** `calc(0 + 5px)` is invalid because the literal `0` inside `calc()` is treated as a `<number>`, not a `<length>`. You must write `calc(0px + 5px)`.
5. **Result type must match the property:** `margin: calc(1px + 2px)` is valid; `margin: calc(1 + 2)` is invalid because it resolves to a unitless number, not a length.
6. **calc() cannot replace partial units:** `calc(100 / 4)%` is invalid; `calc(100% / 4)` is valid.
7. **Standard operator precedence applies:** `calc(2 + 3 * 4)` = 14, not 20. Parentheses can override: `calc((2 + 3) * 4)` = 20.
8. **Nested calc() is equivalent to parentheses:** `calc(calc(2 + 3) * 4)` is the same as `calc((2 + 3) * 4)`.

#### Constraints and Limitations

- **Strict type checking:** The expression's resolved type must match the property's expected type. Mismatches cause the entire declaration to be ignored.
- **No unitless zero for lengths inside calc():** Unlike plain property values, `calc(0 + 5px)` is invalid; you must use `calc(0px + 5px)`.
- **Division by zero:** Dividing by zero produces an invalid value (the declaration is ignored).
- **Infinite/NaN handling:** Operations that produce infinity or NaN result in the declaration being ignored.
- **Performance considerations:** Complex calc() expressions are evaluated at computed-value time; extremely nested expressions may impact performance, though this is rarely significant in practice.
- **Browser support for newer functions:** `min()`, `max()`, and `clamp()` have slightly less historical support than `calc()` itself, though they are now widely available.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mixing Percentages and Pixels with calc()

**HTML File (`calc-mix.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>calc() Unit Mixing</title>
    <link rel="stylesheet" href="calc-mix.css">
</head>
<body>
    <!-- Full-width section with calc() width -->
    <section class="full-width-section">
        <p>This section uses <code>calc(100% / 3 - 2em - 2px)</code> to take up
        one-third of the available space, minus its own padding and border.</p>
    </section>

    <!-- Three-column layout using calc() -->
    <div class="three-columns">
        <div class="column">Column 1</div>
        <div class="column">Column 2</div>
        <div class="column">Column 3</div>
    </div>
</body>
</html>
```

**CSS File (`calc-mix.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

/* Section with calc() mixing percentage, em, and px */
.full-width-section {
    /* Start with 100% / 3, then subtract the element's own
       em-based and px-based spacing */
    width: calc(100% / 3 - 2em - 2px);
    /* Padding uses em (relative to font-size) */
    padding: 1em;
    /* Border uses px (absolute) */
    border: solid 1px #333;
    /* box-sizing ensures padding and border are included in width */
    box-sizing: border-box;
    background-color: #e8f4fd;
    margin-bottom: 20px;
}

/* Three-column flex layout */
.three-columns {
    display: flex;
    gap: 10px;
}

.column {
    /* Each column takes one-third of the space minus the gap */
    width: calc(33.333% - 7px);
    /* Alternative: use calc(100% / 3 - 7px) for clarity */
    padding: 12px;
    background-color: #3498db;
    color: white;
    border-radius: 4px;
    text-align: center;
    box-sizing: border-box;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `calc-mix.html` and the CSS as `calc-mix.css`.
2. Open `calc-mix.html` in a browser.
3. Resize the browser window and observe that the section and columns adapt proportionally.
4. Notice that the section's width changes dynamically as the viewport changes — it is always one-third of the body width minus the specified offsets.

**Expected Output:** A light blue section that occupies roughly one-third of the page width (adjusting responsively), and a row of three blue columns that fill the page width with small gaps between them.

**Why This Works:** `calc(100% / 3 - 2em - 2px)` mixes three units: a percentage (100% of the parent width), `em` (2 × the font size), and `px` (2 pixels). CSS allows this because addition and subtraction require compatible types — and percentage-of-length and length are compatible (both resolve to lengths). The `100% / 3` performs division first (precedence), then the subtractions occur left to right. The result is a single length value that the `width` property accepts.

---

#### Example 2: Responsive Font-Size with clamp()

**HTML File (`clamp.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Typography with clamp()</title>
    <link rel="stylesheet" href="clamp.css">
</head>
<body>
    <!-- Heading with responsive font size -->
    <h1 class="responsive-heading">Responsive Heading</h1>
    <!-- Paragraph with responsive font size -->
    <p class="responsive-text">
        This text uses <code>clamp(16px, 4vw, 24px)</code>, meaning the font size
        will never go below 16px, never exceed 24px, and will scale fluidly
        with the viewport width in between.
    </p>
    <!-- Box with responsive padding -->
    <div class="responsive-box">
        This box uses <code>padding: clamp(10px, 3vw, 30px)</code> for
        responsive internal spacing.
    </div>
</body>
</html>
```

**CSS File (`clamp.css`):**

```css
/* Body base */
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #fafafa;
}

/* Responsive heading: font-size scales with viewport but is bounded */
.responsive-heading {
    /* clamp(min, preferred, max):
       - Never smaller than 24px
       - Ideally 5vw (5% of viewport width)
       - Never larger than 48px */
    font-size: clamp(24px, 5vw, 48px);
    color: #2c3e50;
    margin-bottom: 16px;
}

/* Responsive body text */
.responsive-text {
    /* Never smaller than 16px, ideally 4vw, never larger than 24px */
    font-size: clamp(16px, 4vw, 24px);
    line-height: 1.6;
    color: #34495e;
    margin-bottom: 20px;
}

/* Responsive padding box */
.responsive-box {
    /* Padding scales from 10px to 30px based on viewport width */
    padding: clamp(10px, 3vw, 30px);
    background-color: #3498db;
    color: white;
    border-radius: 8px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `clamp.html` and the CSS as `clamp.css`.
2. Open `clamp.html` in a browser.
3. Resize the browser window from very narrow to very wide.
4. Observe that the heading and paragraph text grow and shrink smoothly with the viewport, but never exceed their bounds.
5. Observe that the box's padding also scales smoothly.

**Expected Output:** A heading that is 24px on narrow screens, grows to 5% of the viewport width as the screen widens, and caps at 48px on very wide screens. Body text follows a similar pattern between 16px and 24px. The blue box's padding scales between 10px and 30px.

**Why This Works:** `clamp(min, preferred, max)` is a comparison function that ensures the value never goes below `min` or above `max`. The `preferred` value is a `calc()`-style expression (here, `5vw`) that provides the fluid scaling. On narrow screens, `5vw` might be less than 24px, so `clamp()` returns 24px. On wide screens, `5vw` might exceed 48px, so `clamp()` returns 48px. In between, the preferred value is used directly. This provides a much more concise and readable alternative to media queries for fluid typography.

---

### Real-World Cases

- **Fluid hero sections:** Using `height: calc(100vh - 80px)` to fill the viewport minus a fixed header height.
- **Responsive typography:** Using `font-size: clamp(1rem, 2.5vw, 2rem)` for headings that scale with the viewport.
- **Centering with known offsets:** Using `left: calc(50% - 100px)` to centre an element with a fixed half-width.
- **Grid gaps:** Using `gap: calc(1rem + 1vw)` for responsive spacing between grid items.
- **Custom property arithmetic:** Using `--spacing: calc(var(--base) * 2)` to build up spacing scales.

---

## References

- W3C — CSS Values and Units Module Level 3 - https://www.w3.org/TR/css-values-3/
- W3C — CSS Values and Units Module Level 4 - https://www.w3.org/TR/css-values-4/
- MDN Web Docs — Numeric Data Types - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Values_and_units/Numeric_data_types
- MDN Web Docs — `<integer>` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/integer
- MDN Web Docs — `<number>` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/number
- MDN Web Docs — `<percentage>` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/percentage
- MDN Web Docs — `<ratio>` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/ratio
- MDN Web Docs — `<angle>` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/angle
- MDN Web Docs — `<time>` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/time
- MDN Web Docs — `calc()` - https://developer.mozilla.org/en-US/docs/Web/CSS/calc
- MDN Web Docs — `aspect-ratio` - https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio
- MDN Web Docs — CSS Typed Arithmetic - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Values_and_units/Using_typed_arithmetic
- W3C — CSS Syntax Module Level 3 - https://www.w3.org/TR/css-syntax-3/
- Chrome for Developers — CSS Layout Gets Smarter with calc() - https://developer.chrome.com/blog/calc/