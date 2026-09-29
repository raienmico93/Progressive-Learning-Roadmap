# CSS Logical Properties — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Logical Properties are a set of CSS properties and values that define layout and styling based on the **flow of content** (the inline and block axes) rather than the physical dimensions of the viewport (top, bottom, left, right). They allow authors to write a single set of styles that adapt automatically to different writing modes, text directions, and languages.

**Technical Definition:** The CSS Logical Properties and Values Module Level 1 introduces logical properties and values that provide the author with the ability to control layout through logical, rather than physical, direction and dimension mappings. The module defines logical properties and values for the features defined in CSS 2.1, serving as writing-mode-relative equivalents of their corresponding physical properties. The logical-to-physical mappings use the value of the `writing-mode` property, the `direction` property, and the `text-orientation` property to resolve. The block axis defines the stacking order of elements in a block layout, while the inline axis is perpendicular to it and represents the direction along which inline content flows. Properties such as `margin-inline-start`, `padding-block-end`, and `border-start-start-radius` use logical directional keywords that are relative to the content flow along these axes.

**Beginner-Friendly Explanation:** Imagine you are writing a webpage for both English (left-to-right) and Arabic (right-to-left) audiences. With traditional CSS, you would use `margin-left` for English but need `margin-right` for Arabic. Logical properties solve this by using "start" and "end" instead of "left" and "right." The "start" is wherever text begins — left in English, right in Arabic. You write `margin-inline-start` once, and it works correctly in both languages. The same applies to vertical writing modes: what was "top" becomes "block-start," and the browser figures out the physical direction automatically.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Flow-relative** | Properties are defined relative to the inline and block axes, not physical directions. |
| **Writing-mode aware** | Mappings resolve based on `writing-mode`, `direction`, and `text-orientation`. |
| **Single codebase** | One set of styles works across LTR, RTL, and vertical writing modes. |
| **Shorthand support** | Logical shorthands like `margin-inline` and `padding-block` set both edges of an axis. |
| **Sizing equivalents** | `inline-size` and `block-size` replace `width` and `height` logically. |
| **Value-level logic** | `text-align: start`, `float: inline-start`, and `clear: inline-end` are flow-relative values. |
| **Corner mapping** | Logical border radii (`border-start-start-radius`) map to physical corners. |

---

### Prerequisites

Before studying CSS Logical Properties, you should understand:

- **CSS Box Model** — margin, border, padding, and content areas.
- **CSS Writing Modes** — `writing-mode`, `direction`, and `text-orientation`.
- **CSS Positioning** — `top`, `right`, `bottom`, `left`, and the `inset` shorthand.
- **CSS Sizing** — `width`, `height`, `min-width`, `max-width`, and related properties.
- **Basic CSS Syntax** — selectors, properties, values, and the cascade.

---

### Related Programming Areas

- **Internationalization (i18n)** — supporting multiple languages and writing directions.
- **Accessibility** — ensuring layouts work for users with different reading directions.
- **Design Systems** — creating direction-agnostic component libraries.
- **Responsive Design** — logical properties complement fluid and responsive layout techniques.
- **CSS Preprocessors** — PostCSS and Sass can polyfill logical properties for older browsers.

---

### Core Concepts / Features

1. Inline-Axis Properties: `margin-inline`, `padding-inline`, `inset-inline`, `border-inline`
2. Block-Axis Properties: `margin-block`, `padding-block`, `inset-block`, `border-block`
3. Logical Sizing: `inline-size` and `block-size`
4. Writing-Mode-Aware Layouts: LTR, RTL, and Vertical Typography
5. Logical Values & Text Alignment: `text-align: start/end`, Logical Floats, and Clear Values
6. Border Radius Mapping: Logical Equivalents for Corners

---

## 1. Inline-Axis Properties: `margin-inline`, `padding-inline`, `inset-inline`, `border-inline`

### Definitions

**Core Definition:** Inline-axis logical properties control the spacing, positioning, and borders along the **inline dimension** — the direction in which text flows within a line. They map to `left`/`right` in horizontal writing modes and to `top`/`bottom` in vertical writing modes.

**Technical Definition:** The inline axis is perpendicular to the block axis and represents the direction along which inline content (text, inline elements) flows within a block. In left-to-right writing modes, the inline direction is horizontal, left-to-right. In right-to-left languages, it is horizontal, right-to-left. The logical properties `margin-inline-start`, `margin-inline-end`, `padding-inline-start`, `padding-inline-end`, `inset-inline-start`, `inset-inline-end`, and their shorthands (`margin-inline`, `padding-inline`, `inset-inline`) map to the physical `left`/`right` properties in horizontal writing modes. The `border-inline` shorthand sets the border color, style, and width for both inline edges simultaneously, with corresponding longhands for `border-inline-start` and `border-inline-end`.

**Beginner-Friendly Explanation:** The "inline axis" is the direction your text flows. In English, that is left to right. The logical property `margin-inline-start` is the margin on the side where the text begins — the left side in English, the right side in Arabic. The shorthand `margin-inline` sets both the start and end margins in one declaration. Similarly, `padding-inline` sets the padding on both the start and end sides, and `inset-inline` sets both the start and end position offsets. The `border-inline` family controls the borders on the start and end sides. All of these adapt automatically to the writing mode.

---

### Purposes

- To set spacing, positioning, and borders along the inline axis in a writing-mode-aware way.
- To eliminate the need for separate LTR and RTL stylesheets.
- To provide shorthands that set both inline edges simultaneously.
- To support vertical writing modes where the inline axis is vertical.
- To simplify maintenance of directional styles in multilingual projects.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Longhand properties */
selector {
    margin-inline-start: <length> | <percentage> | auto;
    margin-inline-end: <length> | <percentage> | auto;
    padding-inline-start: <length> | <percentage>;
    padding-inline-end: <length> | <percentage>;
    inset-inline-start: <length> | <percentage> | auto;
    inset-inline-end: <length> | <percentage> | auto;
    border-inline-start: <border-width> <border-style> <border-color>;
    border-inline-end: <border-width> <border-style> <border-color>;
}

/* Shorthand properties */
selector {
    margin-inline: <start> <end>?;
    padding-inline: <start> <end>?;
    inset-inline: <start> <end>?;
    border-inline: <width> <style> <color>;
    border-inline-color: <color>{1,2};
    border-inline-style: <style>{1,2};
    border-inline-width: <width>{1,2};
}
```

#### Component Breakdown

| Logical Property | Physical Mapping (LTR, horizontal-tb) | Physical Mapping (RTL, horizontal-tb) |
|---|---|---|
| `margin-inline-start` | `margin-left` | `margin-right` |
| `margin-inline-end` | `margin-right` | `margin-left` |
| `margin-inline` | `margin-left` & `margin-right` | `margin-right` & `margin-left` |
| `padding-inline-start` | `padding-left` | `padding-right` |
| `padding-inline-end` | `padding-right` | `padding-left` |
| `inset-inline-start` | `left` | `right` |
| `inset-inline-end` | `right` | `left` |
| `border-inline-start` | `border-left` | `border-right` |
| `border-inline-end` | `border-right` | `border-left` |

#### Syntax Rules

1. `margin-inline` accepts one or two values: one value sets both start and end; two values set start and end respectively.
2. `padding-inline` follows the same one-or-two-value pattern.
3. `inset-inline` accepts one or two values for the inline-start and inline-end offsets.
4. `border-inline` is a shorthand for `border-inline-start` and `border-inline-end`.
5. `border-inline-color`, `border-inline-style`, and `border-inline-width` accept one or two values for start and end.
6. All inline properties resolve against the containing block's inline dimension for percentages.
7. These properties are supported in all modern browsers (Baseline widely available).

#### Constraints and Limitations

- **Percentage resolution** — percentages for inline margins and padding resolve against the containing block's inline size, which may differ from the physical width in vertical writing modes.
- **Legacy browser support** — Internet Explorer does not support logical properties; use a PostCSS polyfill for legacy support.
- **Physical/logical mixing** — mixing physical and logical properties on the same element can lead to unexpected results if the cascade order is not carefully managed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline Spacing with `margin-inline` and `padding-inline`

**HTML File (`inline-spacing.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inline-Axis Logical Properties</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="inline-spacing.css">
</head>
<body>
    <!-- Card with logical inline spacing -->
    <div class="card">
        <h2>Logical Inline Spacing</h2>
        <p>
            This card uses <code>margin-inline: auto</code> to centre itself
            and <code>padding-inline: 2rem</code> for horizontal padding.
            Switch the <code>direction</code> to <code>rtl</code> in DevTools
            to see the spacing adapt automatically.
        </p>
    </div>
</body>
</html>
```

**CSS File (`inline-spacing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.card {
    /* Centre the card along the inline axis */
    margin-inline: auto;
    /* Padding on both inline edges */
    padding-inline: 2rem;
    padding-block: 1.5rem;
    /* Fixed inline size with a maximum */
    max-inline-size: 600px;
    background-color: white;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    /* Logical border on the start edge */
    border-inline-start: 4px solid #006064;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `inline-spacing.html`.
3. Save the CSS code as `inline-spacing.css` in the same folder.
4. Open `inline-spacing.html` in a web browser.
5. Observe the card centred with padding on both inline edges and a border on the start edge.
6. In DevTools, change the `<html>` element's `dir` attribute to `rtl` and observe that the border moves to the right side automatically.

**Expected Output:** A centred card with equal inline padding and a coloured border on the inline-start edge. When the direction changes to RTL, the border moves to the right side and the padding remains correct.

**Why This Works:** The `margin-inline: auto` centres the card along the inline axis, which resolves to `margin-left: auto; margin-right: auto` in LTR and `margin-right: auto; margin-left: auto` in RTL. The `padding-inline: 2rem` sets equal padding on both inline edges. The `border-inline-start` places a border on the start edge, which is the left in LTR and the right in RTL. All mappings adapt automatically to the writing direction.

---

### Real-World Cases

- **Multilingual navigation bars:** Using `margin-inline-start` for spacing between a logo and navigation links so the spacing works in both LTR and RTL.
- **Card components:** Using `padding-inline` for consistent horizontal padding regardless of writing direction.
- **Tooltips and popovers:** Using `inset-inline` to position floating elements relative to their trigger.
- **Form layouts:** Using `border-inline-start` for accent borders on form fields that adapt to RTL.

---

## 2. Block-Axis Properties: `margin-block`, `padding-block`, `inset-block`, `border-block`

### Definitions

**Core Definition:** Block-axis logical properties control the spacing, positioning, and borders along the **block dimension** — the direction in which block-level elements stack. They map to `top`/`bottom` in horizontal writing modes and to `left`/`right` in vertical writing modes.

**Technical Definition:** The block axis refers to the axis that defines the stacking order of elements in a block layout — essentially the direction along which blocks of content (paragraphs, headings, divs) are laid out. In left-to-right and right-to-left languages, the block direction is the vertical direction of the content flow, going from top to bottom. The block-start and block-end directions represent the start edge and end edge of content along the block axis, with block-start being the equivalent of `top` and block-end being the equivalent of `bottom` in horizontal writing modes. The logical properties `margin-block-start`, `margin-block-end`, `padding-block-start`, `padding-block-end`, `inset-block-start`, `inset-block-end`, and their shorthands map to the physical `top`/`bottom` properties. The `border-block` shorthand sets the border for both block edges.

**Beginner-Friendly Explanation:** The "block axis" is the direction your paragraphs stack — usually top to bottom. The logical property `margin-block-start` is the margin at the top (the start of the block flow). The shorthand `margin-block` sets both the top and bottom margins. `padding-block` sets the top and bottom padding. `inset-block` sets the top and bottom position offsets. And `border-block` controls the top and bottom borders. In vertical writing modes, these all adapt: block-start becomes the right or left edge instead of the top.

---

### Purposes

- To set spacing, positioning, and borders along the block axis in a writing-mode-aware way.
- To provide shorthands that set both block edges simultaneously.
- To support vertical writing modes where the block axis is horizontal.
- To simplify vertical rhythm management in multilingual projects.
- To complement inline-axis properties for complete logical control.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Longhand properties */
selector {
    margin-block-start: <length> | <percentage> | auto;
    margin-block-end: <length> | <percentage> | auto;
    padding-block-start: <length> | <percentage>;
    padding-block-end: <length> | <percentage>;
    inset-block-start: <length> | <percentage> | auto;
    inset-block-end: <length> | <percentage> | auto;
}

/* Shorthand properties */
selector {
    margin-block: <start> <end>?;
    padding-block: <start> <end>?;
    inset-block: <start> <end>?;
    border-block: <width> <style> <color>;
    border-block-color: <color>{1,2};
    border-block-style: <style>{1,2};
    border-block-width: <width>{1,2};
}
```

#### Component Breakdown

| Logical Property | Physical Mapping (LTR, horizontal-tb) | Physical Mapping (vertical-rl) |
|---|---|---|
| `margin-block-start` | `margin-top` | `margin-right` |
| `margin-block-end` | `margin-bottom` | `margin-left` |
| `margin-block` | `margin-top` & `margin-bottom` | `margin-right` & `margin-left` |
| `padding-block-start` | `padding-top` | `padding-right` |
| `padding-block-end` | `padding-bottom` | `padding-left` |
| `inset-block-start` | `top` | `right` |
| `inset-block-end` | `bottom` | `left` |
| `border-block-start` | `border-top` | `border-right` |
| `border-block-end` | `border-bottom` | `border-left` |

#### Syntax Rules

1. `margin-block` accepts one or two values: one sets both start and end; two set start and end respectively.
2. `padding-block` follows the same one-or-two-value pattern.
3. `inset-block` accepts one or two values for the block-start and block-end offsets.
4. `border-block` is a shorthand for `border-block-start` and `border-block-end`.
5. `border-block-color`, `border-block-style`, and `border-block-width` accept one or two values.
6. Percentage values for block margins and padding resolve against the containing block's inline size (not block size) in most cases.
7. These properties are supported in all modern browsers.

#### Constraints and Limitations

- **Percentage resolution quirk** — percentage block margins and padding resolve against the containing block's inline size, not its block size, which can be counterintuitive.
- **Legacy browser support** — not supported in Internet Explorer.
- **Margin collapsing** — logical block margins collapse in the same way as physical margins in a block formatting context.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Vertical Rhythm with `margin-block` and `padding-block`

**HTML File (`block-spacing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Block-Axis Logical Properties</title>
    <link rel="stylesheet" href="block-spacing.css">
</head>
<body>
    <article class="article">
        <h1>Block-Axis Spacing</h1>
        <p>
            This article uses <code>margin-block</code> for vertical rhythm
            and <code>padding-block</code> for internal spacing. The
            <code>border-block-start</code> adds a decorative top border.
        </p>
        <p>
            Change the <code>writing-mode</code> to <code>vertical-rl</code>
            in DevTools to see the block direction become horizontal.
        </p>
    </article>
</body>
</html>
```

**CSS File (`block-spacing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.article {
    max-inline-size: 600px;
    margin-inline: auto;
    /* Block margins for vertical rhythm */
    margin-block: 2rem;
    /* Block padding for internal spacing */
    padding-block: 1.5rem;
    padding-inline: 2rem;
    background-color: white;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    /* Decorative border on the block-start edge */
    border-block-start: 4px solid #e67e22;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `block-spacing.html` and CSS as `block-spacing.css`.
2. Open in a browser. Observe the article with vertical margins and padding, and a border on the top edge.
3. In DevTools, set `writing-mode: vertical-rl` on the `.article` element. Observe that the border moves to the right edge (the new block-start), and the block margins and padding now apply horizontally.

**Expected Output:** An article with top and bottom margins, internal vertical padding, and a decorative top border. In vertical writing mode, the same properties apply to the right and left edges.

**Why This Works:** The `margin-block: 2rem` sets `margin-top: 2rem; margin-bottom: 2rem` in horizontal writing mode. The `padding-block: 1.5rem` sets `padding-top: 1.5rem; padding-bottom: 1.5rem`. The `border-block-start` applies a border to the block-start edge (top in horizontal, right in vertical-rl). All mappings resolve based on the writing mode.

---

### Real-World Cases

- **Article layouts:** Using `margin-block` for consistent vertical spacing between paragraphs and headings.
- **Card internals:** Using `padding-block` for equal top and bottom padding inside cards.
- **Vertical navigation:** Using `inset-block` for positioning elements in vertical writing modes.
- **Accordion components:** Using `border-block` for separator lines between accordion items.

---

## 3. Logical Sizing: `inline-size` and `block-size`

### Definitions

**Core Definition:** `inline-size` and `block-size` are the logical equivalents of `width` and `height`, sizing elements along the inline and block axes respectively.

**Technical Definition:** The logical mappings for `width` and `height` are `inline-size` (which sets the length in the inline dimension) and `block-size` (which sets the length in the block dimension). In English (horizontal-tb), replacing `width` with `inline-size` and `height` with `block-size` produces the same layout. In a vertical writing mode, the same properties follow the rotated text direction as if the entire block were rotated. The properties `min-inline-size`, `min-block-size`, `max-inline-size`, and `max-block-size` are the logical equivalents of `min-width`, `min-height`, `max-width`, and `max-height`.

**Beginner-Friendly Explanation:** `inline-size` is the logical version of `width`, and `block-size` is the logical version of `height`. In English, they behave exactly the same. But in vertical writing mode, `inline-size` controls the vertical size (because text flows vertically) and `block-size` controls the horizontal size. Using logical sizing means your layout adapts automatically when the writing mode changes, without any additional CSS.

---

### Purposes

- To size elements along the inline and block axes in a writing-mode-aware way.
- To replace `width`/`height` and `min-width`/`max-width` with logical equivalents.
- To support vertical writing modes without rewriting sizing code.
- To provide a consistent sizing vocabulary across writing modes.
- To complement logical spacing and positioning properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    inline-size: <length> | <percentage> | auto | min-content | max-content | fit-content(<length>);
    block-size: <length> | <percentage> | auto | min-content | max-content | fit-content(<length>);
    min-inline-size: <length> | <percentage> | auto | min-content | max-content | fit-content(<length>);
    min-block-size: <length> | <percentage> | auto | min-content | max-content | fit-content(<length>);
    max-inline-size: <length> | <percentage> | none | min-content | max-content | fit-content(<length>);
    max-block-size: <length> | <percentage> | none | min-content | max-content | fit-content(<length>);
}
```

#### Component Breakdown

| Logical Property | Physical Equivalent (LTR, horizontal-tb) | Physical Equivalent (vertical-rl) |
|---|---|---|
| `inline-size` | `width` | `height` |
| `block-size` | `height` | `width` |
| `min-inline-size` | `min-width` | `min-height` |
| `min-block-size` | `min-height` | `min-width` |
| `max-inline-size` | `max-width` | `max-height` |
| `max-block-size` | `max-height` | `max-width` |

#### Syntax Rules

1. `inline-size` and `block-size` accept the same values as `width` and `height`.
2. `min-inline-size` and `min-block-size` accept the same values as `min-width` and `min-height`.
3. `max-inline-size` and `max-block-size` accept the same values as `max-width` and `max-height`.
4. Percentage values resolve against the containing block's corresponding logical dimension.
5. `resize: inline` allows resizing in the inline dimension; `resize: block` allows resizing in the block dimension.
6. These properties are supported in all modern browsers.

#### Constraints and Limitations

- **Browser support** — logical sizing properties are supported in all modern browsers but not in Internet Explorer.
- **Percentage resolution** — percentages resolve against the containing block's logical dimension, which may differ from the physical dimension in vertical writing modes.
- **Fixed vs. logical** — using fixed `width`/`height` locks the layout to physical dimensions; logical sizing adapts to the writing mode.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Logical Sizing Across Writing Modes

**HTML File (`logical-sizing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logical Sizing</title>
    <link rel="stylesheet" href="logical-sizing.css">
</head>
<body>
    <h2>Horizontal Writing Mode</h2>
    <div class="box horizontal">
        <p>inline-size: 300px, block-size: 150px</p>
    </div>

    <h2>Vertical Writing Mode</h2>
    <div class="box vertical">
        <p>inline-size: 300px, block-size: 150px</p>
    </div>
</body>
</html>
```

**CSS File (`logical-sizing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.box {
    /* Logical sizing properties */
    inline-size: 300px;
    block-size: 150px;
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 8px;
    margin-block: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
}

.horizontal {
    /* Default horizontal writing mode */
}

.vertical {
    /* Vertical writing mode */
    writing-mode: vertical-rl;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `logical-sizing.html` and CSS as `logical-sizing.css`.
2. Open in a browser.
3. Observe the horizontal box: 300px wide, 150px tall.
4. Observe the vertical box: because of `writing-mode: vertical-rl`, the `inline-size` (300px) now controls the vertical dimension and the `block-size` (150px) controls the horizontal dimension. The box appears rotated.

**Expected Output:** Two boxes with the same CSS sizing declarations but different physical appearances due to the writing mode. The horizontal box is wider than it is tall; the vertical box is taller than it is wide.

**Why This Works:** The `inline-size: 300px` sets the size along the inline axis. In horizontal writing mode, the inline axis is horizontal, so the box is 300px wide. In vertical-rl writing mode, the inline axis is vertical, so the box is 300px tall. The `block-size: 150px` correspondingly becomes the horizontal size. This demonstrates how logical sizing adapts to the writing mode.

---

### Real-World Cases

- **Multilingual layouts:** Using `inline-size` and `block-size` for components that must work in both horizontal and vertical writing modes.
- **Responsive containers:** Using `max-inline-size` for content containers that should not exceed a comfortable reading width.
- **Vertical sidebars:** Using `block-size` for sidebars in vertical writing modes.
- **Icon containers:** Using `inline-size` and `block-size` for square icons that adapt to writing mode.

---

## 4. Writing-Mode-Aware Layouts: Handling LTR, RTL, and Vertical Typography Layouts Seamlessly

### Definitions

**Core Definition:** Writing-mode-aware layouts are layouts that adapt automatically to the document's writing mode — left-to-right (LTR), right-to-left (RTL), or vertical typography — without requiring separate stylesheets or conditional logic. Logical properties enable this by defining styles in terms of the content flow rather than physical directions.

**Technical Definition:** The CSS Logical Properties and Values Module defines properties that indirectly set certain other CSS properties (including `width`, `margin`, `float`, `text-align`, and `page-break`) based on the writing mode: left-to-right, right-to-left, or top-to-bottom. The logical-to-physical mappings use the value of the `writing-mode` property, the `direction` property, and the `text-orientation` property to resolve. The `writing-mode` property can be `horizontal-tb` (horizontal, top-to-bottom), `vertical-rl` (vertical, right-to-left), or `vertical-lr` (vertical, left-to-right). The `direction` property can be `ltr` or `rtl`. Together, these properties determine the orientation of the block and inline axes, and therefore the physical mapping of all logical properties.

**Beginner-Friendly Explanation:** A writing-mode-aware layout is one that automatically adjusts when you change the language or writing direction. In English (LTR), the inline axis goes left-to-right and the block axis goes top-to-bottom. In Arabic (RTL), the inline axis goes right-to-left but the block axis stays top-to-bottom. In Japanese vertical writing, the inline axis goes top-to-bottom and the block axis goes right-to-left. Logical properties let you write one set of styles that works in all these modes. You use `margin-inline-start` instead of `margin-left`, and the browser figures out which physical direction that means.

---

### Purposes

- To create layouts that work seamlessly across LTR, RTL, and vertical writing modes.
- To eliminate the need for separate LTR and RTL stylesheets.
- To support internationalisation without duplicating CSS.
- To simplify maintenance of multilingual design systems.
- To future-proof layouts for emerging writing modes and languages.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Logical properties adapt automatically */
selector {
    margin-inline: 1rem;
    padding-block: 2rem;
    inline-size: 100%;
    text-align: start;
    float: inline-start;
    border-inline-start: 4px solid;
}

/* Writing mode changes the physical mapping */
selector {
    writing-mode: horizontal-tb | vertical-rl | vertical-lr;
    direction: ltr | rtl;
}
```

#### Component Breakdown

| Writing Mode | Direction | Inline Axis | Block Axis | `margin-inline-start` maps to |
|---|---|---|---|---|
| `horizontal-tb` | `ltr` | Left → Right | Top → Bottom | `margin-left` |
| `horizontal-tb` | `rtl` | Right → Left | Top → Bottom | `margin-right` |
| `vertical-rl` | `ltr` | Top → Bottom | Right → Left | `margin-top` |
| `vertical-lr` | `ltr` | Top → Bottom | Left → Right | `margin-top` |

#### Syntax Rules

1. The `writing-mode` property determines the orientation of the inline and block axes.
2. The `direction` property determines the inline direction (LTR or RTL) in horizontal writing modes.
3. Logical properties resolve based on both `writing-mode` and `direction`.
4. Changing the writing mode changes all logical-to-physical mappings.
5. Logical properties work in all writing modes without modification.
6. Physical properties (`margin-left`, `width`, etc.) do not adapt to writing mode.

#### Constraints and Limitations

- **Browser support** — logical properties are supported in all modern browsers but not in Internet Explorer.
- **Mixed content** — when different parts of a page use different writing modes, each formatting context resolves its own mappings.
- **Debugging complexity** — logical properties can be harder to debug because the computed physical values depend on the writing mode.
- **Third-party libraries** — some CSS frameworks and component libraries may not fully support logical properties.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Card That Works in LTR, RTL, and Vertical Writing Modes

**HTML File (`writing-mode-card.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Writing-Mode-Aware Card</title>
    <link rel="stylesheet" href="writing-mode-card.css">
</head>
<body>
    <div class="container ltr">
        <div class="card">
            <div class="card-icon">★</div>
            <div class="card-content">
                <h3>LTR Card</h3>
                <p>Left-to-right horizontal writing mode.</p>
            </div>
        </div>
    </div>

    <div class="container rtl" dir="rtl">
        <div class="card">
            <div class="card-icon">★</div>
            <div class="card-content">
                <h3>RTL Card</h3>
                <p>Right-to-left horizontal writing mode.</p>
            </div>
        </div>
    </div>

    <div class="container vertical">
        <div class="card">
            <div class="card-icon">★</div>
            <div class="card-content">
                <h3>Vertical Card</h3>
                <p>Vertical right-to-left writing mode.</p>
            </div>
        </div>
    </div>
</body>
</html>
```

**CSS File (`writing-mode-card.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.container {
    margin-block-end: 20px;
}

.rtl {
    direction: rtl;
}

.vertical {
    writing-mode: vertical-rl;
}

.card {
    display: flex;
    align-items: center;
    /* Logical gap between icon and content */
    gap: 1rem;
    /* Logical padding */
    padding-inline: 1.5rem;
    padding-block: 1rem;
    /* Logical border on the start edge */
    border-inline-start: 4px solid #006064;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    /* Logical inline size */
    max-inline-size: 400px;
}

.card-icon {
    font-size: 2rem;
    color: #e67e22;
    /* Prevent the icon from shrinking */
    flex-shrink: 0;
}

.card-content {
    /* Allow content to shrink */
    min-inline-size: 0;
}

.card-content h3 {
    margin: 0 0 0.25rem;
}

.card-content p {
    margin: 0;
    color: #555;
    font-size: 0.9rem;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `writing-mode-card.html` and CSS as `writing-mode-card.css`.
2. Open in a browser.
3. Observe the LTR card: the icon is on the left, the border is on the left, and the text flows left-to-right.
4. Observe the RTL card: the icon is on the right, the border is on the right, and the text flows right-to-left.
5. Observe the vertical card: the icon is at the top, the border is on the top edge, and the text flows vertically.

**Expected Output:** Three identical card components that adapt their layout automatically to the writing mode and direction. The icon, border, padding, and text alignment all shift correctly.

**Why This Works:** The `.card` uses only logical properties: `padding-inline`, `padding-block`, `border-inline-start`, `max-inline-size`, and `gap`. These properties resolve differently depending on the writing mode and direction. In LTR, `border-inline-start` is the left border. In RTL, it is the right border. In vertical-rl, it is the top border. The `flex` layout and `gap` also adapt to the writing mode because flexbox respects the writing mode's inline axis.

---

### Real-World Cases

- **International e-commerce sites:** Product cards that work in English, Arabic, and Japanese without separate stylesheets.
- **Content management systems:** Components that adapt to the language direction of the content they display.
- **Design systems:** A single component library that supports all writing modes.
- **News sites:** Article layouts that reflow correctly for RTL languages and vertical scripts.

---

## 5. Logical Values & Text Alignment: `text-align: start/end`, Logical Floats, and Clear Values

### Definitions

**Core Definition:** Logical values are flow-relative keyword values that can be used with properties like `text-align`, `float`, and `clear`. They replace physical keywords (`left`, `right`) with logical keywords (`start`, `end`, `inline-start`, `inline-end`) that adapt to the writing mode and direction.

**Technical Definition:** The CSS Logical Properties and Values Module defines `start` and `end` values for the `text-align` property, and `inline-start` and `inline-end` values for the `float` and `clear` properties. The `text-align` property accepts the `start` and `end` keywords where `left` and `right` are allowed; `start` is the same as `left` if direction is left-to-right and `right` if direction is right-to-left, and `end` is the opposite. The `float` and `clear` properties accept `inline-start` and `inline-end` as mappings for `left` and `right`. These values are 1-dimensional logical directions, consistent with the 1-dimensional nature of text alignment and float positioning.

**Beginner-Friendly Explanation:** `text-align: start` means "align text to the start of the line" — left in English, right in Arabic. `text-align: end` means "align to the end" — right in English, left in Arabic. Similarly, `float: inline-start` floats an element to the start side of its container, and `clear: inline-end` clears floats on the end side. These logical values make your CSS work correctly in any language without conditional logic.

---

### Purposes

- To align text to the start or end of the inline direction without knowing the language direction.
- To float elements to the start or end side of their container in a writing-mode-aware way.
- To clear floats on the start or end side regardless of the writing mode.
- To eliminate the need for separate LTR and RTL alignment rules.
- To provide a consistent logical vocabulary for directional values.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    text-align: start | end | left | right | center | justify | match-parent;
    float: inline-start | inline-end | left | right | none;
    clear: inline-start | inline-end | left | right | both | none;
}
```

#### Component Breakdown

| Property | Logical Value | Physical Mapping (LTR) | Physical Mapping (RTL) |
|---|---|---|---|
| `text-align` | `start` | `left` | `right` |
| `text-align` | `end` | `right` | `left` |
| `float` | `inline-start` | `left` | `right` |
| `float` | `inline-end` | `right` | `left` |
| `clear` | `inline-start` | `left` | `right` |
| `clear` | `inline-end` | `right` | `left` |

#### Syntax Rules

1. `text-align: start` aligns text to the start of the inline direction.
2. `text-align: end` aligns text to the end of the inline direction.
3. `float: inline-start` floats the element to the inline-start side.
4. `float: inline-end` floats the element to the inline-end side.
5. `clear: inline-start` clears floats on the inline-start side.
6. `clear: inline-end` clears floats on the inline-end side.
7. These logical values are supported in all modern browsers.

#### Constraints and Limitations

- **`text-align: start` is the initial value** — in CSS Text Level 3, the initial value of `text-align` is `start`, though browser default styles may set it differently.
- **Float and clear logical values** — these are 1-dimensional and do not support the full 2-dimensional logical syntax.
- **Legacy browser support** — older browsers may not support `inline-start` and `inline-end` for `float` and `clear`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Logical Text Alignment and Floats

**HTML File (`logical-values.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logical Values</title>
    <link rel="stylesheet" href="logical-values.css">
</head>
<body>
    <div class="article">
        <h2>Logical Text Alignment</h2>
        <p class="align-start">
            This paragraph is aligned to the <code>start</code> of the inline
            direction. In English, that is left. In Arabic, it would be right.
        </p>
        <p class="align-end">
            This paragraph is aligned to the <code>end</code> of the inline
            direction. In English, that is right. In Arabic, it would be left.
        </p>

        <h2>Logical Floats</h2>
        <div class="float-demo">
            <div class="float-box">Float inline-start</div>
            <p>
                This text wraps around a box that is floated with
                <code>float: inline-start</code>. The box sits on the start
                side of the container, which is the left in English and the
                right in Arabic.
            </p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`logical-values.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.article {
    max-inline-size: 600px;
    margin-inline: auto;
    background-color: white;
    padding: 20px;
    border-radius: 10px;
}

.align-start {
    /* Align to the start of the inline direction */
    text-align: start;
    background-color: #e0f7fa;
    padding: 10px;
    border-radius: 6px;
}

.align-end {
    /* Align to the end of the inline direction */
    text-align: end;
    background-color: #fff3e0;
    padding: 10px;
    border-radius: 6px;
}

.float-demo {
    /* Clear the float after the container */
    overflow: hidden;
}

.float-box {
    /* Float to the inline-start side */
    float: inline-start;
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    margin-inline-end: 15px;
    margin-block-end: 10px;
    font-weight: bold;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `logical-values.html` and CSS as `logical-values.css`.
2. Open in a browser.
3. Observe the first paragraph aligned to the left (start in LTR).
4. Observe the second paragraph aligned to the right (end in LTR).
5. Observe the floated box on the left side of the container, with text wrapping around it.
6. In DevTools, set `direction: rtl` on the `.article` element. Observe that the alignment and float swap sides automatically.

**Expected Output:** Two paragraphs aligned to start and end respectively, and a floated box on the start side with text wrapping around it. When the direction changes to RTL, all alignment and float positions swap automatically.

**Why This Works:** The `text-align: start` and `text-align: end` values resolve based on the `direction` property. In LTR, `start` is left and `end` is right. In RTL, they swap. The `float: inline-start` floats the box to the start side of the inline direction, which is left in LTR and right in RTL. The `margin-inline-end` provides spacing on the opposite side of the float.

---

### Real-World Cases

- **Multilingual navigation:** Using `text-align: start` for navigation links so they align correctly in both LTR and RTL.
- **Pull quotes:** Using `float: inline-start` for pull quotes that sit on the start side of the text.
- **Clearfix patterns:** Using `clear: inline-start` or `clear: inline-end` for clearing floats in a writing-mode-aware way.
- **Table cells:** Using `text-align: start` for table headers and cells that adapt to the language direction.

---

## 6. Border Radius Mapping: Logical Equivalents for Corners (`border-start-start-radius`)

### Definitions

**Core Definition:** Logical border radius properties define rounded corners using logical directions (block-start, block-end, inline-start, inline-end) instead of physical corners (top-left, top-right, bottom-left, bottom-right). They map to the physical corner radii based on the writing mode and direction.

**Technical Definition:** The logical border radius properties are `border-start-start-radius`, `border-start-end-radius`, `border-end-start-radius`, and `border-end-end-radius`. The `border-start-start-radius` property defines the radius of the corner between the block-start and inline-start sides of the element, regardless of the writing mode. The `border-start-end-radius` defines the corner between block-start and inline-end. The `border-end-start-radius` defines the corner between block-end and inline-start. The `border-end-end-radius` defines the corner between block-end and inline-end. In horizontal-tb with LTR direction, `border-start-start-radius` maps to `border-top-left-radius`, `border-start-end-radius` maps to `border-top-right-radius`, `border-end-start-radius` maps to `border-bottom-left-radius`, and `border-end-end-radius` maps to `border-bottom-right-radius`.

**Beginner-Friendly Explanation:** Logical border radii let you round corners based on the flow of text instead of physical positions. `border-start-start-radius` is the corner where the block-start edge meets the inline-start edge — the top-left corner in English, the top-right in Arabic, and the top-right in vertical writing. The four logical radius properties cover all four corners, and they adapt automatically to the writing mode. This is useful for components that need consistent corner rounding across languages.

---

### Purposes

- To define rounded corners in a writing-mode-aware way.
- To support RTL and vertical writing modes without separate corner rules.
- To provide a logical alternative to `border-top-left-radius` and its physical counterparts.
- To maintain consistent corner rounding across multilingual layouts.
- To complement logical spacing and sizing properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    border-start-start-radius: <length> | <percentage>;
    border-start-end-radius: <length> | <percentage>;
    border-end-start-radius: <length> | <percentage>;
    border-end-end-radius: <length> | <percentage>;
}
```

#### Component Breakdown

| Logical Property | Physical Mapping (LTR, horizontal-tb) | Physical Mapping (RTL, horizontal-tb) | Physical Mapping (vertical-rl) |
|---|---|---|---|
| `border-start-start-radius` | `border-top-left-radius` | `border-top-right-radius` | `border-top-right-radius` |
| `border-start-end-radius` | `border-top-right-radius` | `border-top-left-radius` | `border-bottom-right-radius` |
| `border-end-start-radius` | `border-bottom-left-radius` | `border-bottom-right-radius` | `border-top-left-radius` |
| `border-end-end-radius` | `border-bottom-right-radius` | `border-bottom-left-radius` | `border-bottom-left-radius` |

#### Syntax Rules

1. The first "start" or "end" refers to the block axis; the second refers to the inline axis.
2. `border-start-start-radius` is the corner between block-start and inline-start.
3. `border-start-end-radius` is the corner between block-start and inline-end.
4. `border-end-start-radius` is the corner between block-end and inline-start.
5. `border-end-end-radius` is the corner between block-end and inline-end.
6. Each accepts one or two values (horizontal radius and optional vertical radius for elliptical corners).
7. These properties are supported in all modern browsers.

#### Constraints and Limitations

- **Writing-mode complexity** — the physical mapping in vertical writing modes can be counterintuitive; the first value of a two-value syntax applies to the horizontal axis in horizontal-tb but may apply differently in vertical modes.
- **Legacy browser support** — not supported in Internet Explorer.
- **Two-value syntax** — the interpretation of two values in logical border radii can be confusing in vertical writing modes.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Logical Border Radius on a Card

**HTML File (`logical-radius.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logical Border Radius</title>
    <link rel="stylesheet" href="logical-radius.css">
</head>
<body>
    <div class="card ltr">
        <h3>LTR Card</h3>
        <p>Rounded corners using logical border radius properties.</p>
    </div>

    <div class="card rtl" dir="rtl">
        <h3>RTL Card</h3>
        <p>Rounded corners using logical border radius properties.</p>
    </div>

    <div class="card vertical">
        <h3>Vertical Card</h3>
        <p>Rounded corners using logical border radius properties.</p>
    </div>
</body>
</html>
```

**CSS File (`logical-radius.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.card {
    /* Logical border radii */
    border-start-start-radius: 20px;
    border-start-end-radius: 4px;
    border-end-start-radius: 4px;
    border-end-end-radius: 20px;
    /* Visible styling */
    background-color: white;
    padding: 20px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    inline-size: 200px;
    border-inline-start: 4px solid #006064;
}

.rtl {
    direction: rtl;
}

.vertical {
    writing-mode: vertical-rl;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `logical-radius.html` and CSS as `logical-radius.css`.
2. Open in a browser.
3. Observe the LTR card: the top-left corner has a 20px radius, the top-right has 4px, the bottom-left has 4px, and the bottom-right has 20px.
4. Observe the RTL card: the corners are mirrored — the top-right has 20px, the top-left has 4px, etc.
5. Observe the vertical card: the corners are mapped according to the vertical writing mode.

**Expected Output:** Three cards with different corner rounding patterns that adapt to the writing mode and direction. The LTR and RTL cards are mirror images of each other. The vertical card has its corners mapped according to the vertical writing mode.

**Why This Works:** The `border-start-start-radius: 20px` rounds the corner between the block-start and inline-start edges. In LTR horizontal writing, that is the top-left corner. In RTL, it is the top-right corner. In vertical-rl, it is the top-right corner. The other logical radius properties map correspondingly. This allows a single set of corner rounding rules to work across all writing modes.

---

### Real-World Cases

- **Multilingual cards:** Cards with asymmetric corner rounding that adapt to the language direction.
- **Chat bubbles:** Chat bubbles with a "tail" corner that should be on the start side of the message.
- **Design systems:** Components with corner rounding that must work in RTL and vertical writing modes.
- **Buttons and badges:** Buttons with rounded corners that adapt to the writing direction.

---

## References

- MDN Web Docs — Logical properties - https://developer.mozilla.org/en-US/docs/Glossary/Logical_properties
- MDN Web Docs — Logical properties for margins, borders, and padding - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values/Margins_borders_padding
- MDN Web Docs — Logical properties for floating and positioning - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values/Floating_and_positioning
- MDN Web Docs — Logical properties for sizing - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values/Sizing
- MDN Web Docs — `text-align` - https://developer.mozilla.org/en-US/docs/Web/CSS/text-align
- MDN Web Docs — `clear` - https://developer.mozilla.org/en-US/docs/Web/CSS/clear
- MDN Web Docs — `border-start-start-radius` - https://developer.mozilla.org/en-US/docs/Web/CSS/border-start-start-radius
- W3C — CSS Logical Properties and Values Module Level 1 - https://www.w3.org/TR/css-logical-1/
- W3C — CSS Logical Properties and Values Module Level 1 (Editor's Draft) - https://drafts.csswg.org/css-logical/
- CSS-Tricks — CSS Logical Properties - https://css-tricks.com/css-logical-properties/
- CSS-Tricks — Logical Properties for Useful Shorthands - https://css-tricks.com/logical-properties-for-useful-shorthands/
- Smashing Magazine — Understanding Logical Properties and Values - https://www.smashingmagazine.com/2018/03/understanding-logical-properties-values/
- 30 Seconds of Code — Logical and Physical CSS Properties Equivalents - https://github.com/Chalarangelo/30-seconds-of-code/blob/master/content/snippets/css/s/logical-physical-properties-map.md
- Can I Use — CSS Logical Properties - https://caniuse.com/css-logical-props
- Can I Use — `border-start-start-radius` - https://caniuse.com/mdn-css_properties_border-start-start-radius