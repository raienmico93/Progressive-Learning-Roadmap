# CSS Intrinsic and Extrinsic Sizing — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Intrinsic and Extrinsic Sizing is the system of CSS properties, keywords, and functions that determine how elements derive their dimensions — either from their own content (intrinsic) or from their surrounding layout context (extrinsic). This system governs how `width`, `height`, and their logical equivalents resolve when set to `auto` or to content-based keywords.

**Technical Definition:** The CSS Intrinsic & Extrinsic Sizing Module Level 3 extends the CSS sizing properties with keywords that represent content-based "intrinsic" sizes and context-based "extrinsic" sizes, allowing CSS to more easily describe boxes that fit their content or fit into a particular layout context. Intrinsic sizing determines sizes based on the contents of an element, without regard for its context. Extrinsic sizing determines sizes based on the context of an element, without regard for its contents. The `min-content` size of a box in each axis is the size it would have if it were a float given an `auto` size in that axis and if its containing block were zero-sized in that axis — in other words, the minimum size it has when sized as "shrink-to-fit". The `max-content` size is the size it would have if its containing block were infinitely-sized in that axis — the maximum size it has when sized as "shrink-to-fit".

**Beginner-Friendly Explanation:** Normally, when you set `width: 300px` on a box, you are telling the browser exactly how wide it should be. That is extrinsic sizing — the size comes from outside the content. But sometimes you want the browser to figure out the width based on what is inside. If you have a box with text, the browser can size it to fit the longest word (`min-content`), to fit the entire paragraph on one line (`max-content`), or to a comfortable size that uses available space but does not get too wide (`fit-content`). These are intrinsic sizes — the size comes from the content itself. This cheat sheet explains how these sizing models work, how they interact with Flexbox and Grid, and how to use them for performance optimisation.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Content vs. context** | Intrinsic sizing depends on content; extrinsic sizing depends on layout context. |
| **Keyword values** | `min-content`, `max-content`, and `fit-content` are content-based intrinsic keywords. |
| **Function form** | `fit-content(<length-percentage>)` clamps a size between minimum and maximum bounds. |
| **Layout-mode awareness** | Individual layout modes like Flexbox and Grid define their own intrinsic sizing rules. |
| **Inline-axis effect** | Intrinsic keywords only have effect in the inline progression direction; they are equivalent to `auto` in the block direction. |
| **Performance integration** | `contain-intrinsic-size` works with `content-visibility` to provide estimated sizes for skipped content. |

---

### Prerequisites

Before studying intrinsic and extrinsic sizing, you should understand:

- **CSS Box Model** — content, padding, border, and margin.
- **CSS `width` and `height`** — how dimensions are declared and resolved.
- **CSS Flexbox and Grid** — layout models that define their own sizing algorithms.
- **CSS Writing Modes** — how the inline and block axes are oriented.
- **The `auto` keyword** — how it resolves differently depending on layout context.

---

### Related Programming Areas

- **CSS Grid Layout** — track sizing functions include `min-content`, `max-content`, and `fit-content()`.
- **CSS Flexbox** — flex items use intrinsic sizes for their flex base size.
- **Web Performance** — `content-visibility` and `contain-intrinsic-size` optimise rendering.
- **Responsive Design** — intrinsic sizing enables layouts that adapt to content without media queries.
- **CSS Containment** — size containment interacts with intrinsic sizing.

---

### Core Concepts / Features

1. Extrinsic Sizing: Strict Dimensions Using Lengths and Percentages
2. Content-Based Keywords: `min-content`, `max-content`, and `fit-content`
3. The `fit-content()` Function: Capping Expansion
4. Available Space and Layout Constraints: Sizing Inside Flexbox and Grid
5. Performance Optimisation: `contain-intrinsic-size` with `content-visibility`

---

## 1. Extrinsic Sizing: Defining Strict Dimensions Using Lengths (px, rem) and Percentages (%)

### Definitions

**Core Definition:** Extrinsic sizing is the model in which an element's dimensions are determined by factors outside its own content — specifically, by explicit length values, percentage values, or the constraints of its containing block.

**Technical Definition:** Extrinsic sizing determines sizes based on the context of an element, without regard for its contents. When `width` is set to a `<length>` (e.g., `300px`, `20rem`), the element's inline size is fixed at that value regardless of its content. When `width` is set to a `<percentage>`, the size is calculated relative to the containing block's inline size. When `width` is `auto` in a block formatting context, the element fills the available inline space of its containing block. In Flexbox and Grid, `auto` and percentage values are further modified by the layout algorithm, which may stretch, shrink, or distribute space among items.

**Beginner-Friendly Explanation:** Extrinsic sizing means the size comes from outside the element. If you write `width: 500px`, the element is exactly 500px wide, no matter what is inside. If you write `width: 50%`, the element is half the width of its parent. If you write `width: auto` on a normal block element, it fills the available space. These are all extrinsic because the browser is not looking at the content to decide the size — it is looking at the numbers you provided or the container around it.

---

### Purposes

- To specify exact, predictable dimensions for layout-critical elements.
- To create fixed-width sidebars, columns, or containers.
- To size elements relative to their containing block using percentages.
- To provide the baseline against which intrinsic sizing is compared.
- To work in conjunction with `min-width` and `max-width` for bounded fluid sizing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    width: <length> | <percentage> | auto;
    height: <length> | <percentage> | auto;
    min-width: <length> | <percentage>;
    max-width: <length> | <percentage> | none;
}
```

#### Component Breakdown

| Value Type | Example | Behaviour |
|---|---|---|
| Absolute length | `300px`, `20rem` | Fixed size regardless of content or container. |
| Relative length | `50%`, `30vw` | Resolved against containing block or viewport. |
| `auto` | `width: auto` | Context-dependent; fills available space in block layout. |
| `min-width` | `min-width: 200px` | Lower bound; overrides `width` if larger. |
| `max-width` | `max-width: 800px` | Upper bound; overrides `width` if smaller. |

#### Syntax Rules

1. `width` and `height` accept `<length>`, `<percentage>`, and `auto`.
2. Percentage values for `width` resolve against the containing block's inline size.
3. Percentage values for `height` resolve against the containing block's block size, which must be definite.
4. `min-width` overrides `width` if `min-width` is larger.
5. `max-width` overrides `width` if `max-width` is smaller.
6. When `min-width` > `max-width`, `min-width` wins.
7. Negative values are invalid for all sizing properties.

#### Constraints and Limitations

- **Fixed sizes do not adapt** — a fixed pixel width will overflow small containers.
- **Percentage height requires a defined parent height** — `height: 50%` has no effect if the parent's height is `auto`.
- **`auto` behaviour varies** — `width: auto` fills space in block layout but shrink-wraps in Flexbox and Grid.
- **Viewport units** — `vw` and `vh` are extrinsic but relative to the viewport, not the container.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fixed and Percentage Sizing

**HTML File (`extrinsic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Extrinsic Sizing</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="extrinsic.css">
</head>
<body>
    <!-- Container with a fixed width -->
    <div class="container">
        <!-- Fixed-width box -->
        <div class="fixed-box">Fixed: 200px</div>
        <!-- Percentage-width box -->
        <div class="percent-box">Percentage: 50%</div>
        <!-- Auto-width box (fills available space) -->
        <div class="auto-box">Auto: fills available space</div>
    </div>
</body>
</html>
```

**CSS File (`extrinsic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    /* Fixed-width container */
    width: 600px;
    max-width: 100%;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.fixed-box {
    /* Absolute length: always 200px wide */
    width: 200px;
    background-color: #3498db;
    color: white;
    padding: 15px;
    border-radius: 6px;
    margin-block-end: 10px;
}

.percent-box {
    /* Percentage: 50% of the container's width */
    width: 50%;
    background-color: #27ae60;
    color: white;
    padding: 15px;
    border-radius: 6px;
    margin-block-end: 10px;
}

.auto-box {
    /* Auto: fills the container's available width */
    width: auto;
    background-color: #e67e22;
    color: white;
    padding: 15px;
    border-radius: 6px;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `extrinsic.html`.
3. Save the CSS code as `extrinsic.css` in the same folder.
4. Open `extrinsic.html` in a web browser.
5. Observe the three boxes: the fixed box is exactly 200px wide, the percentage box is 50% of the container (300px), and the auto box fills the remaining space.

**Expected Output:** A 600px container with a 200px fixed box, a 300px percentage box, and a full-width auto box. The sizes are determined by explicit values, not by the content inside.

**Why This Works:** The `.fixed-box` has `width: 200px`, so it is always 200px regardless of content. The `.percent-box` has `width: 50%`, which resolves to 300px (50% of 600px). The `.auto-box` has `width: auto`, which in block layout fills the containing block's inline size. All three are extrinsic because the sizes come from the declared values or the container, not from the content.

---

### Real-World Cases

- **Fixed sidebars:** `width: 250px` for a navigation sidebar that should not change size.
- **Content containers:** `max-width: 1200px` for a readable article width.
- **Responsive images:** `width: 100%; max-width: 100%` for images that fill their container.
- **Grid columns:** `grid-template-columns: 200px 1fr` for a fixed-sidebar layout.

---

## 2. Content-Based Keywords: Detailed Behavior of `min-content`, `max-content`, and `fit-content`

### Definitions

**Core Definition:** `min-content`, `max-content`, and `fit-content` are CSS keyword values that size an element based on its content. `min-content` is the smallest size without overflow, `max-content` is the size to fit all content on one line, and `fit-content` uses available space within those bounds.

**Technical Definition:** The `min-content` size of a box in each axis is the size it would have if it were a float given an `auto` size in that axis and if its containing block were zero-sized in that axis. The `max-content` size is the size it would have if its containing block were infinitely-sized in that axis. The `fit-content` keyword represents the formula `min(max-content, max(min-content, stretch))`, where `stretch` is the available space. These keywords only have effect in the inline progression direction; they are equivalent to `auto` when set on the block axis of horizontal elements or on the inline axis of vertical elements.

**Beginner-Friendly Explanation:** `min-content` sizes an element to the smallest width that does not overflow — usually the width of the longest word. `max-content` sizes it to the width needed to fit all the content on one line without wrapping. `fit-content` is the smart one: it uses the available space, but never goes smaller than `min-content` or larger than `max-content`. These keywords let the content decide the size, which is useful for buttons, tags, labels, and grid columns that should size to their content.

---

### Purposes

- To size elements based on their content rather than explicit dimensions.
- To create grids and flex layouts that adapt to content width.
- To prevent overflow by using `min-content` as a lower bound.
- To prevent excessive line length by using `max-content` as an upper bound.
- To provide content-driven sizing that works without media queries.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    width: min-content | max-content | fit-content;
    height: min-content | max-content | fit-content;
    min-width: min-content | max-content | fit-content;
    max-width: min-content | max-content | fit-content | none;
    inline-size: min-content | max-content | fit-content;
    block-size: min-content | max-content | fit-content;
}
```

#### Component Breakdown

| Keyword | Definition | Typical Result |
|---|---|---|
| `min-content` | Smallest size without overflow. | Width of the longest word. |
| `max-content` | Size to fit all content on one line. | Full width of the text. |
| `fit-content` | `min(max-content, max(min-content, stretch))`. | Uses available space within bounds. |

#### Syntax Rules

1. `min-content` and `max-content` can be used with `width`, `height`, `min-width`, `max-width`, and their logical equivalents.
2. `fit-content` (keyword) is distinct from `fit-content()` (function); the keyword takes no argument.
3. These keywords only have effect in the inline progression direction.
4. In the block direction, they are equivalent to `auto`.
5. `min-content` is bound below by the content's minimum size (e.g., the longest word).
6. `max-content` is bound above by the content's full unwrapped size.
7. All three are supported in all modern browsers.

#### Constraints and Limitations

- **Content dependency** — intrinsic sizes depend on the content, which may change dynamically.
- **Performance** — intrinsic sizing requires content measurement, which can be expensive in large layouts.
- **Block-axis limitation** — these keywords have no effect in the block direction.
- **Replaced elements** — for images with intrinsic aspect ratios but no intrinsic size, special rules apply (default 300×150 fallback).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing `min-content`, `max-content`, and `fit-content`

**HTML File (`keywords.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Content-Based Keywords</title>
    <link rel="stylesheet" href="keywords.css">
</head>
<body>
    <h3>min-content (longest word)</h3>
    <div class="box min-content-box">
        This is a long sentence with a verylongwordinit.
    </div>

    <h3>max-content (all on one line)</h3>
    <div class="box max-content-box">
        This is a long sentence with a verylongwordinit.
    </div>

    <h3>fit-content (available space, clamped)</h3>
    <div class="box fit-content-box">
        This is a long sentence with a verylongwordinit.
    </div>
</body>
</html>
```

**CSS File (`keywords.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

h3 {
    font-size: 0.85rem;
    color: #555;
    margin-bottom: 5px;
    margin-top: 20px;
}

.box {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.9rem;
}

.min-content-box {
    /* Smallest width without overflow (longest word) */
    width: min-content;
    background-color: #e74c3c;
}

.max-content-box {
    /* Width to fit all content on one line */
    width: max-content;
    background-color: #3498db;
}

.fit-content-box {
    /* Available space, clamped between min-content and max-content */
    width: fit-content;
    background-color: #27ae60;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `keywords.html` and CSS as `keywords.css`.
2. Open in a browser.
3. Observe the `min-content` box: it is only as wide as the longest word ("verylongwordinit").
4. Observe the `max-content` box: it is wide enough to fit the entire sentence on one line, possibly overflowing the container.
5. Observe the `fit-content` box: it uses the available space, but is clamped between the min-content and max-content sizes.

**Expected Output:** Three boxes with different widths based on the same text content. The red `min-content` box is narrow, the blue `max-content` box is wide (possibly overflowing), and the green `fit-content` box is sized to the available space.

**Why This Works:** The `min-content` keyword sizes the box to the smallest width without overflow — the longest word. The `max-content` keyword sizes it to fit all content on one line. The `fit-content` keyword uses the available space but clamps to `min(max-content, max(min-content, stretch))`. This demonstrates the three content-based sizing models in action.

---

### Real-World Cases

- **Buttons and tags:** `width: fit-content` for buttons that should be as wide as their label but not wider.
- **Grid columns:** `grid-template-columns: min-content 1fr` for label-value layouts where labels size to content.
- **Card titles:** `max-content` for titles that should not wrap.
- **Breadcrumb navigation:** `min-content` for breadcrumb segments that should not break.

---

## 3. The `fit-content()` Function: Utilizing `fit-content(percentage/length)` to Cap Expansion

### Definitions

**Core Definition:** The `fit-content()` CSS function clamps a given size to an available size according to the formula `min(maximum size, max(minimum size, argument))`. It is used primarily in CSS Grid track sizing and as a laid-out box size for `width`, `height`, and their logical equivalents.

**Technical Definition:** The `fit-content()` function clamps a given size to an available size according to the formula `min(maximum size, max(minimum size, argument))`. It is distinct from the `fit-content` keyword, which takes no argument and sizes a box based on its content within the available space. Only `fit-content()` is valid in grid track sizing properties such as `grid-template-columns`. In grid track sizing, the maximum size is defined by `max-content` and the minimum size by `auto`, which is calculated similar to `auto` (i.e., `minmax(auto, max-content)`), except that the track size is clamped at the argument if it is greater than the auto minimum. The `fit-content()` function can also be used as a laid-out box size for `width`, `height`, `min-width`, `min-height`, `max-width`, and `max-height`.

**Beginner-Friendly Explanation:** The `fit-content()` function is like `fit-content` but with a cap. You write `fit-content(300px)` and the browser sizes the element to fit its content, but never wider than 300px. If the content is small, the element is small. If the content is large, the element grows up to 300px and then wraps or overflows. This is useful for grid columns that should size to content but not exceed a certain width, or for boxes that should be no wider than a comfortable reading measure.

---

### Purposes

- To cap the expansion of content-sized elements at a maximum width.
- To create grid tracks that size to content but do not exceed a specified limit.
- To provide a middle ground between `max-content` (unbounded) and a fixed length.
- To improve readability by limiting line length in content-sized boxes.
- To work in conjunction with `min-content` and `auto` for bounded content sizing.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    width: fit-content(<length-percentage>);
    height: fit-content(<length-percentage>);
    grid-template-columns: fit-content(<length-percentage>);
    grid-template-rows: fit-content(<length-percentage>);
}
```

#### Component Breakdown

| Parameter | Description | Example |
|---|---|---|
| `<length>` | Absolute or relative length. | `fit-content(200px)` |
| `<percentage>` | Percentage of the containing block. | `fit-content(40%)` |
| Formula | `min(max-content, max(auto, argument))`. | — |

#### Syntax Rules

1. `fit-content()` accepts a `<length>` or `<percentage>` argument.
2. The formula is `min(maximum size, max(minimum size, argument))`.
3. In grid track sizing, the maximum is `max-content` and the minimum is `auto`.
4. In box sizing (`width`, `height`), the maximum and minimum refer to the content size.
5. `fit-content()` is valid in `grid-template-columns` and `grid-template-rows`.
6. `fit-content()` is valid for `width`, `height`, `min-width`, `min-height`, `max-width`, and `max-height`.
7. The keyword `fit-content` (no parentheses) is distinct and takes no argument.

#### Constraints and Limitations

- **Argument is required** — `fit-content()` must have a length or percentage argument.
- **Not interchangeable with the keyword** — `fit-content` (keyword) and `fit-content()` (function) behave differently.
- **Percentage resolution** — percentage arguments resolve against the containing block's inline size.
- **Browser support** — `fit-content()` is supported in all modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Grid Columns with `fit-content()`

**HTML File (`fit-content-function.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>fit-content() Function</title>
    <link rel="stylesheet" href="fit-content-function.css">
</head>
<body>
    <div class="grid">
        <div class="item">Short</div>
        <div class="item">This is a much longer piece of content that will be capped at 200px.</div>
        <div class="item">Medium</div>
    </div>
</body>
</html>
```

**CSS File (`fit-content-function.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.grid {
    display: grid;
    /* Each column sizes to content but is capped at 200px */
    grid-template-columns: fit-content(200px) fit-content(200px) fit-content(200px);
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.8rem;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `fit-content-function.html` and CSS as `fit-content-function.css`.
2. Open in a browser.
3. Observe that the first and third columns are sized to their short content, while the second column is capped at 200px (the long content wraps within that limit).

**Expected Output:** A three-column grid where each column sizes to its content but never exceeds 200px. The long content in the middle column wraps to fit within the 200px cap.

**Why This Works:** The `fit-content(200px)` in `grid-template-columns` uses the formula `min(max-content, max(auto, 200px))`. For short content, `auto` (the minimum) is small, so the track sizes to content. For the long content, `max-content` would be very wide, but the `200px` argument caps it. The track grows to 200px and the content wraps.

---

### Real-World Cases

- **Grid layouts:** `grid-template-columns: fit-content(200px) 1fr` for sidebars that size to content but do not exceed 200px.
- **Card titles:** `width: fit-content(300px)` for titles that should not exceed a readable line length.
- **Form labels:** `fit-content(150px)` for labels that size to their text but cap at 150px.
- **Data grids:** `fit-content(120px)` for columns that size to content but keep a consistent maximum.

---

## 4. Available Space & Layout Constraints: How Sizing Changes Inside Flexbox or Grid Tracks

### Definitions

**Core Definition:** Available space and layout constraints describe how sizing properties resolve differently depending on the layout mode. In Flexbox and Grid, the container's sizing algorithm overrides or modifies the element's intrinsic and extrinsic sizes.

**Technical Definition:** Intrinsic sizing determines sizes based on the contents of a box, without regard for the context in which it is placed. Individual layout modes, such as Flexbox or Grid, can define their own intrinsic sizing rules. In Flexbox, a flex item's `flex-basis` determines its initial main size, and `flex-grow` and `flex-shrink` modify it based on available space. In Grid, track sizing functions (`min-content`, `max-content`, `auto`, `fit-content()`, `<flex>`) determine track sizes through a multi-step algorithm that first resolves intrinsic sizes, then distributes free space. The `min-content` and `max-content` contributions of grid items determine the intrinsic track sizes, and the `fr` unit distributes remaining space after fixed and intrinsic tracks are resolved. Available space can alternatively be either a `min-content` constraint or a `max-content` constraint.

**Beginner-Friendly Explanation:** When you use `width: 100px` on a normal block element, it is exactly 100px wide. But inside a Flexbox or Grid container, things change. In Flexbox, the `flex-basis` property can override `width`, and `flex-grow` can make the item larger. In Grid, the track sizing algorithm looks at the content of all items in a column and decides how wide the column should be. The `min-content` and `max-content` keywords are used by the algorithm to figure out the smallest and largest sizes the content needs. This is why the same element can have different sizes in different layout contexts.

---

### Purposes

- To understand why sizing behaves differently in Flexbox and Grid compared to normal flow.
- To use `min-content` and `max-content` as grid track sizing functions.
- To predict how `flex-basis`, `flex-grow`, and `flex-shrink` interact with intrinsic sizes.
- To debug unexpected sizing in nested layout contexts.
- To leverage layout-mode-specific sizing for responsive designs.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Flexbox sizing */
.flex-item {
    flex: <grow> <shrink> <basis>;
    /* basis can be a length, percentage, auto, min-content, max-content, or fit-content */
}

/* Grid track sizing */
.grid {
    grid-template-columns: min-content max-content auto fit-content(200px) 1fr;
}
```

#### Component Breakdown

| Layout Mode | Sizing Property | Intrinsic Keyword Support |
|---|---|---|
| Block | `width`, `height` | `min-content`, `max-content`, `fit-content` |
| Flexbox | `flex-basis` | `min-content`, `max-content`, `fit-content`, `auto` |
| Grid | `grid-template-columns` / `grid-template-rows` | `min-content`, `max-content`, `auto`, `fit-content()`, `<flex>` |

#### Syntax Rules

1. In Flexbox, `flex-basis: auto` uses the item's `width` or `height` as the initial size.
2. In Flexbox, `flex-basis: content` sizes the item based on its content.
3. In Grid, `min-content` as a track sizing function sizes the track to the smallest content contribution.
4. In Grid, `max-content` sizes the track to the largest content contribution.
5. In Grid, `auto` as a track sizing function behaves like `minmax(auto, max-content)`.
6. In Grid, `fit-content(<length>)` clamps the track between `auto` and the argument.
7. In Grid, `<flex>` (`fr`) distributes remaining free space after fixed and intrinsic tracks.

#### Constraints and Limitations

- **Context-dependent** — the same keyword can behave differently in Flexbox vs. Grid.
- **Intrinsic contribution** — grid track sizing considers the intrinsic contributions of all items in a track.
- **Flexbox flex base size** — the flex base size is determined by `flex-basis`, which may override `width`.
- **Percentage resolution** — percentages in grid tracks resolve against the grid container's content box.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `min-content` and `max-content` in Grid Tracks

**HTML File (`grid-sizing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid Track Sizing</title>
    <link rel="stylesheet" href="grid-sizing.css">
</head>
<body>
    <div class="grid">
        <div class="item">Short</div>
        <div class="item">A much longer piece of content</div>
        <div class="item">Medium text</div>
    </div>
</body>
</html>
```

**CSS File (`grid-sizing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.grid {
    display: grid;
    /* Three columns with different sizing functions */
    grid-template-columns: min-content max-content auto;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.8rem;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `grid-sizing.html` and CSS as `grid-sizing.css`.
2. Open in a browser.
3. Observe the three columns: the first (`min-content`) is only as wide as its longest word, the second (`max-content`) is as wide as its full content, and the third (`auto`) sizes to content and then stretches to fill remaining space.

**Expected Output:** A three-column grid where the first column is narrow (min-content), the second is wide (max-content), and the third fills the remaining space (auto).

**Why This Works:** The `min-content` track sizing function sizes the first column to the smallest contribution from its items (the longest word). The `max-content` function sizes the second column to the largest contribution (the full text). The `auto` function sizes the third column to its content and then stretches to fill remaining space. This demonstrates how intrinsic keywords are used in the Grid track sizing algorithm.

---

### Real-World Cases

- **Label-value layouts:** `grid-template-columns: min-content 1fr` for labels that size to content and values that fill space.
- **Dashboard grids:** `grid-template-columns: repeat(3, minmax(min-content, 1fr))` for columns that are at least content-sized but can grow.
- **Flexbox toolbars:** `flex: 0 0 auto` for items that should size to content, and `flex: 1` for items that should fill remaining space.
- **Card grids:** `grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))` for responsive card layouts.

---

## 5. Performance Optimisation: `contain-intrinsic-size` Used Alongside `content-visibility` to Prevent Browser Rendering Lag

### Definitions

**Core Definition:** `contain-intrinsic-size` is a CSS property that provides an estimated size for an element whose contents are being skipped by `content-visibility`, preventing the browser from having to render the content to determine its size.

**Technical Definition:** The `contain-intrinsic-size` property allows the browser to estimate the size of an element when its contents are not being rendered due to `content-visibility: auto` or `content-visibility: hidden`. By default, an element with `content-visibility: auto` that is off-screen will skip rendering its contents, which means the browser does not know its size. This can cause the scrollbar to jump when the element is scrolled into view and its real size is computed. The `contain-intrinsic-size` property provides a placeholder size, and the `auto` keyword allows the browser to remember the last rendered size and use it as the placeholder on subsequent skips. The `contain-intrinsic-size` gives the browser an estimated height so the scrollbar does not jump when the section eventually renders. Using `content-visibility: auto` with `contain-intrinsic-size` can cut LCP by 30–50% on content-heavy pages because the browser focuses all its rendering budget on the above-fold content first.

**Beginner-Friendly Explanation:** When you have a long page with many sections, rendering all of them at once is slow. The `content-visibility: auto` property tells the browser: "Skip rendering this section until it is about to be scrolled into view." This speeds up initial page load dramatically. But there is a catch: if the browser skips rendering a section, it does not know how tall it is, which can cause the scrollbar to jump. The `contain-intrinsic-size` property solves this by giving the browser an estimated size. You write `contain-intrinsic-size: auto 500px` and the browser uses 500px as the placeholder height until the section is rendered, then remembers the real height for next time.

---

### Purposes

- To improve page rendering performance by skipping off-screen content.
- To prevent scrollbar jumping when skipped content is rendered.
- To provide the browser with a size estimate for content-visibility-skipped elements.
- To reduce Largest Contentful Paint (LCP) by focusing rendering resources on above-fold content.
- To enable efficient long-page layouts with many sections.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    content-visibility: visible | auto | hidden;
    contain-intrinsic-size: <length> | auto <length>;
    contain-intrinsic-size: auto <length>{1,2};
}
```

#### Component Breakdown

| Property | Value | Description |
|---|---|---|
| `content-visibility` | `visible` | Default; content is rendered normally. |
| `content-visibility` | `auto` | Content is skipped when off-screen; rendered when needed. |
| `content-visibility` | `hidden` | Content is always skipped but remains in the layout. |
| `contain-intrinsic-size` | `<length>` | Fixed placeholder size. |
| `contain-intrinsic-size` | `auto <length>` | Remember the last rendered size; use the length as fallback. |

#### Syntax Rules

1. `content-visibility: auto` requires `contain-intrinsic-size` to prevent scrollbar jumping.
2. `contain-intrinsic-size` accepts one or two values (width and height).
3. `contain-intrinsic-size: auto 500px` uses 500px as the fallback height and remembers the last rendered height.
4. The `auto` keyword remembers the last rendered size and uses it on subsequent skips.
5. `content-visibility: hidden` always skips content but keeps it in the layout.
6. These properties are supported in all modern browsers (Chrome 85+, Firefox 125+, Safari 18+).

#### Constraints and Limitations

- **Estimated size accuracy** — the placeholder size should be a reasonable estimate; too small causes jumps, too large wastes space.
- **Layout dependencies** — `contain-intrinsic-size` provides a size, but the content's actual size may differ.
- **Browser support** — `content-visibility` is supported in modern browsers but not Internet Explorer.
- **Accessibility** — content that is skipped with `content-visibility: hidden` is not accessible to screen readers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Optimising a Long Page with `content-visibility` and `contain-intrinsic-size`

**HTML File (`content-visibility.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Content Visibility Optimisation</title>
    <link rel="stylesheet" href="content-visibility.css">
</head>
<body>
    <!-- Above-fold section: always rendered -->
    <section class="hero">
        <h1>Above the Fold</h1>
        <p>This section is always rendered.</p>
    </section>

    <!-- Below-fold sections: skipped until scrolled into view -->
    <section class="below-fold">
        <h2>Section 1</h2>
        <p>This section is skipped until it is about to be scrolled into view.</p>
    </section>

    <section class="below-fold">
        <h2>Section 2</h2>
        <p>This section is also skipped until needed.</p>
    </section>

    <section class="below-fold">
        <h2>Section 3</h2>
        <p>And this one too.</p>
    </section>
</body>
</html>
```

**CSS File (`content-visibility.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.hero {
    background-color: #006064;
    color: white;
    padding: 60px 20px;
    border-radius: 10px;
    margin-bottom: 20px;
    text-align: center;
}

.below-fold {
    /* Skip rendering when off-screen */
    content-visibility: auto;
    /* Provide a placeholder size to prevent scrollbar jumping */
    contain-intrinsic-size: auto 400px;
    background-color: white;
    padding: 30px;
    border-radius: 10px;
    margin-bottom: 20px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `content-visibility.html` and CSS as `content-visibility.css`.
2. Open in a browser.
3. Open DevTools → Performance and record a page load. Observe that the below-fold sections are not rendered initially.
4. Scroll down and observe that the sections render as they come into view.
5. The `contain-intrinsic-size: auto 400px` provides a placeholder height so the scrollbar does not jump.

**Expected Output:** A page with a hero section and three below-fold sections. The below-fold sections are skipped during initial rendering, improving load performance. As you scroll, they render smoothly without scrollbar jumps.

**Why This Works:** The `content-visibility: auto` on `.below-fold` tells the browser to skip rendering these sections when they are off-screen. The `contain-intrinsic-size: auto 400px` provides a placeholder height of 400px, preventing the scrollbar from jumping. The `auto` keyword tells the browser to remember the actual rendered height and use it as the placeholder on subsequent skips. This dramatically reduces the initial rendering work, improving LCP and overall page performance.

---

### Real-World Cases

- **Long article pages:** Skipping below-fold sections to improve initial load performance.
- **E-commerce category pages:** Skipping off-screen product cards to reduce rendering time.
- **Documentation sites:** Skipping large code examples until scrolled into view.
- **Dashboard applications:** Skipping off-screen widgets to focus rendering on visible content.

---

## References

- W3C — CSS Intrinsic & Extrinsic Sizing Module Level 3 - https://www.w3.org/TR/css-sizing-3/
- W3C — CSS Intrinsic & Extrinsic Sizing Module Level 3 (Editor's Draft) - https://drafts.csswg.org/css-sizing-3/
- MDN Web Docs — `fit-content()` CSS function - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/fit-content_function
- MDN Web Docs — Intrinsic size - https://developer.mozilla.org/en-US/docs/Glossary/Intrinsic_Size
- MDN Web Docs — `content-visibility` - https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility
- MDN Web Docs — `contain-intrinsic-size` - https://developer.mozilla.org/en-US/docs/Web/CSS/contain-intrinsic-size
- CSS-Tricks — `fit-content` and `fit-content()` - https://css-tricks.com/fit-content-and-fit-content/
- web.dev — content-visibility: the new CSS property that boosts your rendering performance - https://web.dev/articles/content-visibility
- Chrome for Developers — Animate to height: auto; (and other intrinsic sizing keywords) - https://developer.chrome.com/blog/animate-to-height-auto
- QuirksMode — `fit-content` and `fit-content()` - https://www.quirksmode.org/blog/archives/2021/04/fitcontent_and.html
- CSS Working Group — Intrinsic contribution of `fit-content()` with cyclic percentages - https://lists.w3.org/Archives/Public/public-css-archive/2025Jun/0042.html
- CSS Working Group — `contain-intrinsic-size: auto` with `content-visibility: auto` - https://lists.w3.org/Archives/Public/public-css-archive/2022Jul/0459.html
- Can I Use — CSS `content-visibility` - https://caniuse.com/css-content-visibility
- Can I Use — `fit-content()` - https://caniuse.com/mdn-css_properties_width_fit-content_function