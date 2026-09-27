# CSS Positioning and Logical Inset Properties — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Positioning and Logical Inset Properties is the system of CSS properties that control the placement of positioned elements using either physical directional offsets (`top`, `right`, `bottom`, `left`) or flow-relative logical offsets (`inset-block-start`, `inset-block-end`, `inset-inline-start`, `inset-inline-end`, and their shorthands). This system governs how inset values are resolved, how conflicting offsets interact, and how automatic sizing behaves when positioned elements are stretched or shrink-wrapped.

**Technical Definition:** Inset properties control positioned elements' location by specifying offsets from the elements' default positions. They comprise physical properties (`top`, `left`, `bottom`, `right`), their flow-relative logical property equivalents (`inset-block-start`, `inset-block-end`, `inset-inline-start`, `inset-inline-end`), and the shorthands `inset-block`, `inset-inline`, and `inset`. Physical properties reference specific physical sides of an element. Logical properties use directional keywords relative to the block and inline axes, where the block axis defines the stacking order of elements in a block layout and the inline axis is perpendicular to it, representing the direction along which inline content flows within a block. The mapping between logical and physical properties depends on the element's `writing-mode`, `direction`, and `text-orientation`. The interpretation of inset properties depends on the value of the `position` property: with `position: absolute`, they represent insets from the containing block; with `position: relative`, they represent insets from the box's default margin edge position; with `position: sticky`, they represent insets from the scroll container edge; and `position: fixed` is similar to absolute, except the element is positioned and sized relative to its fixed positioning containing block, often the viewport.

**Beginner-Friendly Explanation:** When you position an element with `position: absolute` or `position: relative`, you use inset properties to tell the browser how far to push the element from its edges. The traditional way uses physical directions: `top`, `right`, `bottom`, and `left`. But these physical names become confusing when you write in different languages or use vertical writing modes. Logical properties solve this by using "block" and "inline" directions instead. The "block" direction is the way text stacks (usually top-to-bottom in English), and the "inline" direction is the way text flows (usually left-to-right in English). If you switch to a right-to-left language or a vertical writing mode, the logical properties automatically adjust, while the physical ones do not.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Dual property systems** | Physical (`top`, `right`, `bottom`, `left`) and logical (`inset-block-start`, etc.) coexist. |
| **Shorthand properties** | `inset`, `inset-block`, and `inset-inline` provide compact syntax for multiple offsets. |
| **Flow-relative mapping** | Logical properties map to physical properties based on `writing-mode` and `direction`. |
| **Position-dependent interpretation** | The meaning of inset properties depends on the `position` value. |
| **Auto-stretching behaviour** | Setting opposite offsets to `0` stretches an absolutely positioned element to fill its containing block. |
| **Conflict resolution** | When both offsets in an axis are set and `height`/`width` is `auto`, the element stretches; otherwise, one offset is ignored. |
| **Percentage reference** | Percentage insets refer to the size of the containing block. |

---

### Prerequisites

Before studying CSS Positioning and Logical Inset Properties, you should understand:

- **CSS Positioning** — the `position` property and its five values (`static`, `relative`, `absolute`, `fixed`, `sticky`).
- **The CSS Box Model** — content, padding, border, and margin.
- **Containing block concept** — the rectangular box with respect to which an element's dimensions and position are calculated.
- **Writing modes** — `writing-mode` and `direction` and how they affect text flow.
- **CSS Logical Properties** — the general concept of block and inline axes.

---

### Related Programming Areas

- **Internationalization (i18n)** — supporting left-to-right, right-to-left, and vertical writing modes.
- **Responsive Design** — adapting layouts to different viewport sizes and writing directions.
- **Web Accessibility** — ensuring content order and positioning remain logical in all writing modes.
- **UI Component Libraries** — building components that work across languages and cultures.

---

### Core Concepts / Features

1. Physical Directional Offsets: `top`, `right`, `bottom`, `left`
2. Logical Inset Properties: `inset`, `inset-block` (start/end), `inset-inline` (start/end)
3. Interaction Dynamics Between Conflicting Offset Properties
4. Automatic Sizing Interaction: Auto-Stretching vs. Implicit Dimensions

---

## 1. Physical Directional Offsets: `top`, `right`, `bottom`, `left`

### Definitions

**Core Definition:** Physical directional offsets are the four CSS properties `top`, `right`, `bottom`, and `left` that specify offsets from the corresponding physical edges of a positioned element's containing block.

**Technical Definition:** The `top`, `right`, `bottom`, and `left` properties specify offsets from the top, right, bottom, and left edges of the containing block, respectively, for absolutely positioned elements. For relatively positioned elements, they specify offsets from the corresponding edges of the box's own default position. For sticky positioned elements, they specify offsets from the corresponding edges of the nearest scrollport. These properties apply only to positioned elements (those with `position` other than `static`). Percentages refer to the height of the containing block for `top` and `bottom`, and the width of the containing block for `left` and `right`.

**Beginner-Friendly Explanation:** These are the four classic properties you use to position an element. If you set `top: 20px` on an absolutely positioned element, it will be 20 pixels down from the top of its containing block. `left: 30px` moves it 30 pixels from the left edge. They are called "physical" because they always refer to the actual top, right, bottom, and left sides of the screen or containing block, regardless of the language direction.

---

### Purposes

- To specify precise physical offsets for positioned elements.
- To provide the foundation for absolute, relative, fixed, and sticky positioning.
- To enable pixel-perfect control over element placement.
- To serve as the underlying properties that logical properties map to.
- To support legacy code and older browsers that do not support logical properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    position: absolute | relative | fixed | sticky;
    top: <length> | <percentage> | auto;
    right: <length> | <percentage> | auto;
    bottom: <length> | <percentage> | auto;
    left: <length> | <percentage> | auto;
}
```

#### Component Breakdown

| Property | Description | Percentage Reference |
|---|---|---|
| `top` | Offset from the top edge. | Height of containing block. |
| `right` | Offset from the right edge. | Width of containing block. |
| `bottom` | Offset from the bottom edge. | Height of containing block. |
| `left` | Offset from the left edge. | Width of containing block. |

#### Syntax Rules

1. Apply only to positioned elements (`position` other than `static`).
2. Initial value is `auto` for all four properties.
3. Percentages refer to the size of the containing block.
4. Negative values are allowed.
5. For relatively positioned elements, offsets are from the element's own normal position.
6. For absolutely positioned elements, offsets are from the containing block's edges.

#### Constraints and Limitations

- **Writing-mode agnostic** — physical properties do not adapt to `writing-mode` or `direction`.
- **No shorthand for logical pairs** — physical properties do not have built-in logical equivalents beyond the individual logical properties.
- **Over-specification** — setting all four offsets without `auto` values can over-constrain the element.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Physical Offsets with Absolute Positioning

**HTML File (`physical.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Physical Offsets</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="physical.css">
</head>
<body>
    <!-- Positioned ancestor: establishes the containing block -->
    <div class="container">
        <!-- Absolutely positioned element with physical offsets -->
        <div class="box physical-box">
            Positioned with<br>top: 20px, left: 40px
        </div>
    </div>
</body>
</html>
```

**CSS File (`physical.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    /* Establish the containing block for absolutely positioned children */
    position: relative;
    width: 400px;
    height: 300px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
}

.physical-box {
    /* Remove from flow and position relative to the container */
    position: absolute;
    /* Physical offsets: 20px from top, 40px from left */
    top: 20px;
    left: 40px;
    /* Visible styling */
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.9rem;
    line-height: 1.4;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `physical.html`.
3. Save the CSS code as `physical.css` in the same folder.
4. Open `physical.html` in a web browser.
5. Observe the dark teal box positioned 20 pixels from the top and 40 pixels from the left of the light blue container.

**Expected Output:** A light blue container with a dark teal box positioned 20px from the top and 40px from the left. The box is removed from normal flow and does not affect the container's size.

**Why This Works:** The `position: relative` on `.container` establishes it as the containing block. The `position: absolute` on `.physical-box` removes it from flow. The `top: 20px` and `left: 40px` offset the box from the container's padding edge.

---

### Real-World Cases

- **Legacy codebases:** Many existing websites use physical offsets extensively.
- **Simple positioning:** When writing mode is not a concern, physical offsets are direct and intuitive.
- **Cross-browser compatibility:** Physical properties have universal browser support.

---

## 2. Logical Inset Properties: `inset`, `inset-block` (start/end), and `inset-inline` (start/end)

### Definitions

**Core Definition:** Logical inset properties are flow-relative alternatives to physical offsets that use the block and inline axes to determine positioning, adapting automatically to the element's writing mode and direction.

**Technical Definition:** Logical inset properties include `inset-block-start`, `inset-block-end`, `inset-inline-start`, and `inset-inline-end`, along with the shorthands `inset-block` (sets block-start and block-end) and `inset-inline` (sets inline-start and inline-end). The `inset` shorthand sets all four physical properties. Logical properties use directional keywords relative to the block and inline axes: the block axis defines the stacking order of elements in a block layout, and the inline axis represents the direction along which inline content flows. The mapping depends on the element's `writing-mode`, `direction`, and `text-orientation`. In a standard horizontal writing mode with `direction: ltr`, `inset-block-start` maps to `top`, `inset-block-end` maps to `bottom`, `inset-inline-start` maps to `left`, and `inset-inline-end` maps to `right`.

**Beginner-Friendly Explanation:** Logical properties are like physical properties, but they think in terms of "start" and "end" of text flow rather than "top" and "left." In English (left-to-right, horizontal), `inset-inline-start` means the same as `left`, and `inset-block-start` means the same as `top`. But if you switch to Arabic (right-to-left), `inset-inline-start` becomes `right` and `inset-inline-end` becomes `left`. If you switch to Japanese vertical writing, the block and inline axes swap entirely. The shorthand `inset: 10px` sets all four physical offsets, while `inset-block: 10px` sets only the block-axis offsets and `inset-inline: 10px` sets only the inline-axis offsets.

---

### Purposes

- To support internationalization by adapting to different writing modes and text directions.
- To reduce the need for separate stylesheets for left-to-right and right-to-left languages.
- To provide a more semantic way of expressing positioning intent.
- To simplify responsive and multilingual layout code.
- To future-proof CSS for emerging writing modes and text orientations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Longhand logical properties */
selector {
    position: absolute;
    inset-block-start: <length> | <percentage> | auto;
    inset-block-end: <length> | <percentage> | auto;
    inset-inline-start: <length> | <percentage> | auto;
    inset-inline-end: <length> | <percentage> | auto;
}

/* Axis shorthands */
selector {
    inset-block: <block-start> <block-end>;  /* sets block-start and block-end */
    inset-inline: <inline-start> <inline-end>; /* sets inline-start and inline-end */
}

/* All-in-one shorthand */
selector {
    inset: <top> <right> <bottom> <left>; /* follows margin shorthand syntax */
}
```

#### Component Breakdown

| Property | Logical Axis | Physical Equivalent (LTR, horizontal) |
|---|---|---|
| `inset-block-start` | Block start | `top` |
| `inset-block-end` | Block end | `bottom` |
| `inset-inline-start` | Inline start | `left` |
| `inset-inline-end` | Inline end | `right` |
| `inset-block` | Block axis shorthand | `top` + `bottom` |
| `inset-inline` | Inline axis shorthand | `left` + `right` |
| `inset` | All four sides | `top` + `right` + `bottom` + `left` |

#### Syntax Rules

1. Logical properties apply only to positioned elements.
2. The `inset-block` shorthand sets `inset-block-start` and `inset-block-end`.
3. The `inset-inline` shorthand sets `inset-inline-start` and `inset-inline-end`.
4. The `inset` shorthand sets the four physical properties (`top`, `right`, `bottom`, `left`) following the margin shorthand syntax.
5. Percentage values refer to the size of the containing block.
6. The mapping depends on `writing-mode`, `direction`, and `text-orientation`.

#### Constraints and Limitations

- **Browser support** — logical properties are supported in all modern browsers, but older browsers (IE11) lack support.
- **Mixing physical and logical** — mixing physical and logical properties in the same declaration can lead to unexpected results.
- **Specificity conflicts** — when both physical and logical properties are set, the physical property wins if it is declared later.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Logical vs. Physical Offsets Across Writing Modes

**HTML File (`logical.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logical Inset Properties</title>
    <link rel="stylesheet" href="logical.css">
</head>
<body>
    <h1>Horizontal LTR</h1>
    <!-- Container with logical positioning -->
    <div class="container ltr-container">
        <div class="box logical-box">
            Logical: inset-block-start 20px, inset-inline-start 40px
        </div>
    </div>

    <h1>Vertical RL</h1>
    <!-- Container with vertical writing mode -->
    <div class="container vertical-container">
        <div class="box logical-box">
            Logical: inset-block-start 20px, inset-inline-start 40px
        </div>
    </div>
</body>
</html>
```

**CSS File (`logical.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

h1 {
    font-size: 1rem;
    color: #333;
    margin-top: 20px;
}

.container {
    position: relative;
    width: 300px;
    height: 250px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    margin-bottom: 20px;
}

.vertical-container {
    /* Vertical writing mode: block axis becomes horizontal */
    writing-mode: vertical-rl;
}

.logical-box {
    position: absolute;
    /* Logical offsets: block-start and inline-start */
    inset-block-start: 20px;
    inset-inline-start: 40px;
    background-color: #006064;
    color: white;
    padding: 12px;
    border-radius: 6px;
    font-size: 0.85rem;
    line-height: 1.4;
    max-width: 180px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `logical.html` and CSS as `logical.css`.
2. Open in a browser.
3. Observe the first container: the logical box is positioned 20px from the top and 40px from the left (same as physical `top` and `left` in horizontal LTR mode).
4. Observe the second container: because of `writing-mode: vertical-rl`, the block axis is now horizontal (right-to-left), and the inline axis is vertical (top-to-bottom). The logical box is positioned 20px from the right edge and 40px from the top edge.

**Expected Output:** Two containers with dark teal boxes. In the first (horizontal), the box is at the top-left. In the second (vertical-rl), the box is at the top-right because the block-start maps to the right edge in vertical-rl writing mode.

**Why This Works:** The logical properties `inset-block-start` and `inset-inline-start` map to different physical edges depending on the `writing-mode`. In horizontal LTR, block-start = top and inline-start = left. In vertical-rl, block-start = right and inline-start = top. This demonstrates the power of logical properties for internationalization.

---

### Real-World Cases

- **Multilingual websites:** Using logical properties to support both LTR (English) and RTL (Arabic, Hebrew) layouts with a single stylesheet.
- **Vertical text layouts:** Using logical properties for Japanese, Chinese, and Mongolian vertical writing modes.
- **Design systems:** Adopting logical properties as the standard for all positioning to ensure global readiness.

---

## 3. Interaction Dynamics Between Conflicting Offset Properties

### Definitions

**Core Definition:** Conflicting offset properties occur when both offsets in a given axis (e.g., `top` and `bottom`, or `left` and `right`) are set to non-`auto` values. The CSS specification provides a set of equations to resolve these conflicts based on the `position` value and the element's dimensions.

**Technical Definition:** For absolutely positioned elements, the used values of the vertical dimensions must satisfy the constraint: `top + margin-top + border-top-width + padding-top + height + padding-bottom + border-bottom-width + margin-bottom + bottom = height of containing block`. When both `top` and `bottom` are specified and `height` is `auto`, the element's height is determined by solving the equation, effectively stretching the element to fill the available space. If `height` is not `auto`, one of the offsets is ignored. Per the CSS Positioned Layout Module Level 3, if only one inset property in a given axis is `auto`, it is set to zero. When both are `auto`, the element is positioned at its static position.

**Beginner-Friendly Explanation:** If you set both `top` and `bottom` on an absolutely positioned element, the browser has to figure out what to do. The rule is: if the element has a fixed height, one of the offsets (usually `top`) wins, and the other is ignored. If the element's height is `auto`, the element stretches to fill the space between the two offsets. This is how you can make an element fill the entire height of its container: set `top: 0` and `bottom: 0` with no height.

---

### Purposes

- To enable an element to stretch to fill available space between two offsets.
- To provide precise control over element dimensions through offset constraints.
- To resolve ambiguous positioning scenarios predictably.
- To support centering and stretching layouts without explicit dimensions.
- To provide a foundation for responsive layouts that adapt to container size.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Stretching: both offsets set, height auto */
selector {
    position: absolute;
    top: 0;
    bottom: 0;
    height: auto; /* Default */
    /* Element stretches to fill the container's height */
}

/* Constrained: both offsets set, height explicit */
selector {
    position: absolute;
    top: 20px;
    bottom: 20px;
    height: 100px; /* Element is 100px tall; one offset is ignored */
}
```

#### Component Breakdown

| Scenario | `height` | `top` | `bottom` | Result |
|---|---|---|---|---|
| Both offsets set | `auto` | Set | Set | Element stretches to fill available space. |
| Both offsets set | Fixed | Set | Set | Element uses fixed height; `top` wins (in LTR). |
| One offset set | `auto` | Set | `auto` | Element is positioned from `top`; height is content-driven. |
| One offset set | `auto` | `auto` | Set | Element is positioned from `bottom`; height is content-driven. |
| Both offsets `auto` | `auto` | `auto` | `auto` | Element is positioned at its static position; height is content-driven. |

#### Syntax Rules

1. If both `top` and `bottom` are set and `height` is `auto`, the element stretches to fill the space between them.
2. If both are set and `height` is not `auto`, one offset is ignored (`top` wins in LTR, `bottom` in some RTL cases).
3. For horizontal axes, `left` wins over `right` in LTR; `right` wins in RTL.
4. If only one offset is set, the element is positioned from that edge and sized by its content.
5. If neither offset is set, the element uses its static position.
6. The same logic applies to the horizontal axis with `left` and `right`.

#### Constraints and Limitations

- **Over-constrained elements** — setting all four offsets with a fixed width and height can cause the element to overflow its containing block.
- **Writing-mode dependency** — which offset "wins" when over-constrained depends on the writing mode and direction.
- **Margin interaction** — margins are part of the constraint equation; `auto` margins can absorb extra space and prevent stretching.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Stretching vs. Constrained Offsets

**HTML File (`conflicts.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Conflicting Offsets</title>
    <link rel="stylesheet" href="conflicts.css">
</head>
<body>
    <!-- Container 1: stretching element -->
    <div class="container">
        <div class="box stretch-box">
            top: 0, bottom: 0, height: auto
        </div>
    </div>

    <!-- Container 2: constrained element -->
    <div class="container">
        <div class="box constrained-box">
            top: 20px, bottom: 20px, height: 80px
        </div>
    </div>
</body>
</html>
```

**CSS File (`conflicts.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    position: relative;
    width: 400px;
    height: 200px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    margin-bottom: 20px;
}

.box {
    position: absolute;
    background-color: #006064;
    color: white;
    padding: 10px;
    border-radius: 6px;
    font-size: 0.85rem;
    left: 20px;
    right: 20px;
}

.stretch-box {
    /* Both offsets set, height auto → element stretches */
    top: 0;
    bottom: 0;
    /* height: auto is the default */
}

.constrained-box {
    /* Both offsets set, height fixed → element is 80px tall,
       top wins, bottom is ignored */
    top: 20px;
    bottom: 20px;
    height: 80px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `conflicts.html` and CSS as `conflicts.css`.
2. Open in a browser.
3. Observe the first container: the box stretches to fill the entire height (from top to bottom).
4. Observe the second container: the box is 80px tall, positioned 20px from the top. The `bottom: 20px` is ignored because the height is fixed.

**Expected Output:** The first box fills the full height of the container. The second box is a fixed 80px tall, positioned 20px from the top, with the bottom offset having no effect.

**Why This Works:** The CSS constraint equation resolves the conflict. When `height` is `auto`, both offsets are used, and the height is computed to satisfy the equation (stretching). When `height` is fixed, the equation cannot be satisfied with both offsets, so one offset (`top`) wins, and the other (`bottom`) is ignored.

---

### Real-World Cases

- **Full-height panels:** Using `top: 0; bottom: 0` on a sidebar to make it fill the viewport height.
- **Centered modals:** Using `top: 0; bottom: 0; left: 0; right: 0` with `margin: auto` to center a modal both horizontally and vertically.
- **Sticky footers:** Using `bottom: 0` with `height: auto` to pin a footer to the bottom of a container.

---

## 4. Automatic Sizing Interaction: Auto-Stretching vs. Implicit Dimensions

### Definitions

**Core Definition:** Automatic sizing interaction describes how an absolutely positioned element's dimensions are determined when `width` or `height` is `auto`, particularly when opposite offsets are set and the element either stretches to fill the containing block or sizes to its content.

**Technical Definition:** For absolutely positioned elements, if `width` is `auto` and only one of `left` or `right` is set, the width is determined by the intrinsic width of the element's content (shrink-to-fit). If both `left` and `right` are set and `width` is `auto`, the element stretches to fill the space between them. The same logic applies to `height` with `top` and `bottom`. If `width` is `auto` and neither `left` nor `right` is set, the element is sized to its content and positioned at its static position. The CSS Working Group has discussed the interaction of auto-stretching with self-alignment, resolving that stretch alignment should allow the size to stretch when the normal alignment case allows stretching.

**Beginner-Friendly Explanation:** When you set `position: absolute` on an element and do not give it a width or height, the browser has to decide how big it should be. If you set `left: 0; right: 0`, the element stretches to fill the width of its container. If you only set `left: 20px`, the element shrinks to fit its content. The same applies to height with `top` and `bottom`. This is why you can create a full-width overlay by setting `left: 0; right: 0` without setting a width.

---

### Purposes

- To enable elements to stretch to fill available space without explicit dimensions.
- To allow elements to shrink-wrap their content when only one offset is set.
- To provide automatic sizing behaviour that adapts to container size.
- To support responsive layouts that do not rely on fixed pixel dimensions.
- To simplify centering and stretching without JavaScript or calc().

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Auto-stretching: both offsets set, width auto */
selector {
    position: absolute;
    left: 0;
    right: 0;
    width: auto; /* Default */
    /* Element stretches to fill container width */
}

/* Shrink-to-fit: one offset set, width auto */
selector {
    position: absolute;
    left: 20px;
    width: auto;
    /* Element width is determined by its content */
}
```

#### Component Breakdown

| Scenario | `width` | `left` | `right` | Result |
|---|---|---|---|---|
| Both offsets set | `auto` | Set | Set | Element stretches to fill available width. |
| One offset set | `auto` | Set | `auto` | Element shrink-wraps its content. |
| One offset set | `auto` | `auto` | Set | Element shrink-wraps its content and is positioned from the right. |
| Neither offset set | `auto` | `auto` | `auto` | Element shrink-wraps and uses its static position. |
| Both offsets set | Fixed | Set | Set | Element uses fixed width; `left` wins. |

#### Syntax Rules

1. If `width` is `auto` and both `left` and `right` are set, the element stretches to fill the containing block width.
2. If `width` is `auto` and only one of `left` or `right` is set, the element shrink-wraps its content.
3. If `width` is `auto` and neither `left` nor `right` is set, the element shrink-wraps and uses its static position.
4. The same logic applies to `height` with `top` and `bottom`.
5. `auto` margins can absorb extra space and prevent stretching.

#### Constraints and Limitations

- **Shrink-to-fit is implementation-dependent** — the exact algorithm for shrink-to-fit is not precisely defined in CSS 2.1.
- **Percentage sizing** — percentages for width/height refer to the containing block's dimensions.
- **Min/max constraints** — `min-width`, `max-width`, `min-height`, and `max-height` can override stretching behaviour.
- **Writing-mode interaction** — the physical axis (horizontal/vertical) determines which offsets affect which dimension.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Auto-Stretching and Shrink-to-Fit

**HTML File (`sizing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Auto-Stretching and Shrink-to-Fit</title>
    <link rel="stylesheet" href="sizing.css">
</head>
<body>
    <!-- Container 1: stretching element -->
    <div class="container">
        <div class="box stretch-width">
            left: 0, right: 0 → stretches to full width
        </div>
    </div>

    <!-- Container 2: shrink-to-fit element -->
    <div class="container">
        <div class="box shrink-width">
            left: 20px only → shrinks to content width
        </div>
    </div>
</body>
</html>
```

**CSS File (`sizing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    position: relative;
    width: 400px;
    height: 120px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    margin-bottom: 20px;
}

.box {
    position: absolute;
    background-color: #006064;
    color: white;
    padding: 12px;
    border-radius: 6px;
    font-size: 0.85rem;
    line-height: 1.4;
    top: 20px;
}

.stretch-width {
    /* Both left and right set → element stretches */
    left: 0;
    right: 0;
    /* width: auto is the default */
}

.shrink-width {
    /* Only left set → element shrinks to content */
    left: 20px;
    /* right is auto */
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `sizing.html` and CSS as `sizing.css`.
2. Open in a browser.
3. Observe the first container: the box stretches across the full width of the container.
4. Observe the second container: the box is only as wide as its text content, positioned 20px from the left.

**Expected Output:** The first box fills the entire width of the container. The second box is narrow, just wide enough to fit its text, positioned 20px from the left edge.

**Why This Works:** The `.stretch-width` box has both `left: 0` and `right: 0` with `width: auto`, so the CSS constraint equation forces it to stretch to fill the available width. The `.shrink-width` box has only `left: 20px` set; with `right: auto`, the width is determined by the element's intrinsic content size (shrink-to-fit).

---

### Real-World Cases

- **Full-width overlays:** Using `left: 0; right: 0` to create a full-width overlay without setting `width: 100%`.
- **Tooltips and labels:** Using shrink-to-fit for tooltips that should be only as wide as their text.
- **Responsive sidebars:** Using `top: 0; bottom: 0` to make a sidebar fill the viewport height.
- **Centered modals:** Using `top: 0; bottom: 0; left: 0; right: 0` with `margin: auto` and explicit dimensions to center a modal.

---

## References

- MDN Web Docs — Inset properties - https://developer.mozilla.org/en-US/docs/Glossary/Inset_properties
- MDN Web Docs — Logical properties for floating and positioning - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values/Floating_and_positioning
- W3C — CSS Positioned Layout Module Level 3 - https://www.w3.org/TR/css-position-3/
- W3C — CSS Logical Properties and Values Level 1 - https://www.w3.org/TR/css-logical-1/
- W3C — CSS 2.1 Specification: Visual Formatting Model - https://www.w3.org/TR/CSS2/visuren.html
- W3C — CSS 2.1 Specification: Calculating Heights and Margins - https://www.w3.org/TR/CSS2/visudet.html#abs-non-replaced-height
- CSS Working Group — Absolute positioning auto-size behaviour discussion - https://lists.w3.org/Archives/Public/public-css-archive/2025May/0433.html
- CSS-Tricks — Logical Properties for Useful Shorthands - https://css-tricks.com/logical-properties-for-useful-shorthands/
- CSS-Tricks — `inset` - https://css-tricks.com/almanac/properties/i/inset/
- Stack Overflow — Constraint equation for absolutely positioned elements - https://stackoverflow.com/revisions/58a0fa38-5ccd-4b15-9407-070d1266f844/view-source