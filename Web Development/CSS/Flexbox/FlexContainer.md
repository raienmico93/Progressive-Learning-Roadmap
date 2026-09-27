# Flex Container Properties & Global Space Distribution — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** Flex Container Properties and Global Space Distribution is the set of CSS properties applied to a flex container (the parent element) that govern how flex items (the children) are distributed, aligned, and spaced within the available space. These properties control the flex environment's activation, main axis orientation, line wrapping, and alignment along both the main and cross axes, as well as the gaps between items.

**Technical Definition:** The CSS Flexible Box Layout Module Level 1 defines a set of properties that apply to the flex container itself, as opposed to properties applied to individual flex items. These container-level properties include `display` (to activate the flex formatting context), `flex-direction` (to define the main axis), `flex-wrap` (to control line breaking), `flex-flow` (a shorthand for the latter two), `justify-content` (to distribute space along the main axis), `align-items` (to align items along the cross axis on a single line), `align-content` (to distribute space between lines when wrapping occurs), and `gap` / `row-gap` / `column-gap` (to create gutters between flex items without margin-based spacing). These properties operate within the flex formatting context created by the container and affect the global space distribution across all flex items.

**Beginner-Friendly Explanation:** When you turn a parent element into a flex container using `display: flex`, you gain access to a powerful set of controls for arranging its children. You can decide which direction they flow (row or column), whether they wrap to new lines when they run out of space, how they distribute extra space (packed at the start, centered, spread out), and how they align vertically within their row. You can also add gaps between items without using margins. These container-level properties are what make flexbox so useful for building responsive layouts — they let you describe the overall arrangement of a group of items, and the browser figures out the precise calculations.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Container-level application** | These properties apply to the flex container (parent), not to individual items. |
| **Main axis vs. cross axis** | Properties operate on one of the two axes: `justify-content` on the main axis, `align-items`/`align-content` on the cross axis. |
| **Space distribution** | The core purpose is distributing available (or negative) space among items and lines. |
| **Line-based behaviour** | `flex-wrap` controls whether items stay on one line or break into multiple lines, affecting which alignment properties apply. |
| **Logical direction awareness** | Start/end values adapt to writing modes and `flex-direction`. |
| **Gap as modern spacing** | `gap` replaces margin-based spacing, eliminating edge-margin trimming and calc() complexity. |
| **No inheritance** | Flex container properties do not inherit; each container has its own values. |

---

### Prerequisites

Before studying flex container properties, you should understand:

- **CSS Normal Flow** — how block and inline boxes are laid out by default.
- **The CSS Box Model** — content, padding, border, and margin.
- **The `display` property** — `block`, `inline`, and their formatting contexts.
- **Flexbox fundamentals** — the distinction between flex container and flex items, and the main/cross axis concepts.

---

### Related Programming Areas

- **CSS Grid Layout** — a complementary two-dimensional layout model that shares many alignment properties.
- **Responsive Design** — flex container properties enable fluid, adaptive layouts.
- **UI Component Design** — navigation bars, toolbars, card grids, and form layouts rely on these properties.
- **Internationalization** — logical start/end values in alignment properties support all writing modes.

---

### Core Concepts / Features

1. Activating the Flexbox Environment: `display: flex` vs. `display: inline-flex`
2. Determining Main Axis Orientation: `flex-direction`
3. Wrapping Controls: `flex-wrap` and `flex-flow`
4. Main-Axis Alignment and Negative Space Management: `justify-content`
5. Cross-Axis Single-Line Item Positioning: `align-items`
6. Multi-Line Cross-Axis Row Distribution: `align-content`
7. Grid-Aligned Spatial Distribution: `gap`, `row-gap`, `column-gap`

---

## 1. Activating the Flexbox Environment: `display: flex` vs. `display: inline-flex`

### Definitions

**Core Definition:** The `display: flex` and `display: inline-flex` values activate the flexbox layout model on an element, turning it into a flex container and its direct children into flex items.

**Technical Definition:** A flexbox layout is defined using the `flex` or `inline-flex` values of the `display` property on the parent item. A value of `flex` causes the element to become a block-level flex container, and `inline-flex` an inline-level flex container. These values create a flex formatting context for the element, which is similar to a block formatting context in that floats will not intrude into the container, and the margins on the container will not collapse with those of the items. `inline-flex` makes the container behave like an inline-level element (its outer display type is inline), while its children still participate in the flex layout model (its inner display type is flex).

**Beginner-Friendly Explanation:** Setting `display: flex` on a parent turns it into a flex container. The key difference between `flex` and `inline-flex` is how the container itself behaves in its own context: `flex` makes the container behave like a block-level element (taking up the full width and stacking vertically with other blocks), while `inline-flex` makes the container behave like an inline element (sitting inline with surrounding text or inline elements). The children inside the container behave identically in both cases — they become flex items and are laid out according to flexbox rules.

---

### Purposes

- To create a block-level flex container that fills the available width and stacks vertically with other blocks.
- To create an inline-level flex container that sits inline with surrounding content.
- To establish a flex formatting context that prevents floats from intruding and margins from collapsing.
- To convert direct children into flex items that participate in the flex layout algorithm.
- To provide a foundation for applying all other flex container properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    display: flex;        /* Block-level flex container */
    display: inline-flex; /* Inline-level flex container */
}
```

#### Component Breakdown

| Value | Outer Display Type | Inner Display Type | Behaviour |
|---|---|---|---|
| `flex` | Block-level | Flex | Container behaves like a block element; children are flex items. |
| `inline-flex` | Inline-level | Flex | Container behaves like an inline element; children are flex items. |

#### Syntax Rules

1. `display: flex` creates a block-level flex container.
2. `display: inline-flex` creates an inline-level flex container.
3. Both values create a flex formatting context.
4. Only direct children of the flex container become flex items.
5. Floats cannot intrude into the flex container.
6. Margins do not collapse across the flex formatting context boundary.

#### Constraints and Limitations

- **Block vs. inline context** — `flex` takes up the full width of its containing block; `inline-flex` shrink-wraps its content.
- **Grandchildren are not flex items** — flex layout applies only to direct children.
- **Anonymous flex items** — raw text inside a flex container becomes an unstyleable anonymous flex item.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing `display: flex` and `display: inline-flex`

**HTML File (`display-flex.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>display: flex vs. inline-flex</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="display-flex.css">
</head>
<body>
    <!-- Block-level flex container: fills width, stacks vertically -->
    <div class="flex-block">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <!-- Inline text to demonstrate inline-flex behaviour -->
    <p>
        Text before
        <!-- Inline-level flex container: sits inline with text -->
        <span class="flex-inline">
            <span class="item">A</span>
            <span class="item">B</span>
            <span class="item">C</span>
        </span>
        text after.
    </p>
</body>
</html>
```

**CSS File (`display-flex.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.8;
    background-color: #f5f5f5;
}

/* Block-level flex container */
.flex-block {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    margin-bottom: 20px;
}

/* Inline-level flex container */
.flex-inline {
    display: inline-flex;
    gap: 5px;
    background-color: #fff3e0;
    border: 2px solid #e65100;
    border-radius: 6px;
    padding: 5px 10px;
    vertical-align: middle;
}

.item {
    background-color: #006064;
    color: white;
    padding: 10px 15px;
    border-radius: 4px;
    font-weight: bold;
    font-size: 0.85rem;
}

.flex-inline .item {
    background-color: #e65100;
    padding: 5px 10px;
    font-size: 0.8rem;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `display-flex.html`.
3. Save the CSS code as `display-flex.css` in the same folder.
4. Open `display-flex.html` in a web browser.
5. Observe that the `.flex-block` container takes up the full width of the page (block-level behaviour) and its items are arranged horizontally.
6. Observe that the `.flex-inline` container sits inline within the paragraph text (inline-level behaviour), shrink-wrapping its items, and the text flows around it.

**Expected Output:** A light blue block-level flex container with three dark teal boxes, followed by a paragraph where a small orange inline-flex container with three small boxes sits inline with the text. The block container fills the width; the inline container is only as wide as its content.

**Why This Works:** The `display: flex` on `.flex-block` creates a block-level flex container that fills its containing block width and stacks vertically with other blocks. The `display: inline-flex` on `.flex-inline` creates an inline-level flex container that participates in the inline formatting context of the paragraph — it sits alongside the text and shrink-wraps its content. In both cases, the children become flex items and are laid out with `gap: 10px` (or `gap: 5px`). The `vertical-align: middle` helps align the inline-flex container with the surrounding text.

---

### Real-World Cases

- **Navigation bars:** `display: flex` on a `<nav>` for a full-width block-level navigation bar.
- **Tags and badges:** `display: inline-flex` for tag components that sit inline with text.
- **Button groups:** `display: inline-flex` for button groups that should shrink-wrap their content and sit inline.
- **Card layouts:** `display: flex` for card containers that fill their grid cell.

---

## 2. Determining Main Axis Orientation: `flex-direction`

### Definitions

**Core Definition:** The `flex-direction` property sets the main axis of a flex container, determining the direction in which flex items are laid out.

**Technical Definition:** The `flex-direction` CSS property sets how flex items are placed in the flex container, defining the main axis and the direction (normal or reversed). Values are `row`, `row-reverse`, `column`, and `column-reverse`. The main axis is defined by `flex-direction`. Should you choose `row` or `row-reverse`, your main axis will run along the row in the inline direction. Choose `column` or `column-reverse` and your main axis will run in the block direction, from the top of the page to the bottom. The cross axis runs perpendicular to the main axis. Using `flex-direction` with values of `row-reverse` or `column-reverse` will create a disconnect between the visual presentation of content and DOM order, which can be problematic for accessibility.

**Beginner-Friendly Explanation:** `flex-direction` tells the browser which way to lay out the flex items. `row` (the default) lays them out horizontally, left to right in English. `row-reverse` also lays them out horizontally but from right to left. `column` lays them out vertically, top to bottom. `column-reverse` lays them out vertically from bottom to top. Changing `flex-direction` also swaps which axis is the "main" axis and which is the "cross" axis, which affects how `justify-content` and `align-items` behave.

---

### Purposes

- To define the primary direction in which flex items are laid out.
- To establish the main axis for alignment and space distribution.
- To enable horizontal or vertical layouts without changing HTML structure.
- To support reverse flow directions for right-to-left or bottom-to-top layouts.
- To provide a coordinate system for `justify-content` and `align-items`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    flex-direction: row | row-reverse | column | column-reverse;
}
```

#### Component Breakdown

| Value | Main Axis Direction | Start Point (LTR) | End Point (LTR) |
|---|---|---|---|
| `row` | Horizontal (inline direction). | Left. | Right. |
| `row-reverse` | Horizontal (reversed). | Right. | Left. |
| `column` | Vertical (block direction). | Top. | Bottom. |
| `column-reverse` | Vertical (reversed). | Bottom. | Top. |

#### Syntax Rules

1. The initial value is `row`.
2. The main axis is defined by `flex-direction`.
3. The cross axis is perpendicular to the main axis.
4. `justify-content` aligns items along the main axis defined by `flex-direction`.
5. `row-reverse` and `column-reverse` reverse the visual flow without reordering the DOM.
6. The direction is affected by the writing mode (`direction: rtl` reverses row flow).

#### Constraints and Limitations

- **Accessibility risk** — reverse values create a disconnect between visual order and DOM order, which can confuse screen reader users and keyboard navigators.
- **Main/cross axis swap** — changing `flex-direction` swaps which axis `justify-content` and `align-items` operate on.
- **No DOM reordering** — reverse values are purely visual; the document tree order is unchanged.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: All Four `flex-direction` Values

**HTML File (`flex-direction.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>flex-direction Values</title>
    <link rel="stylesheet" href="flex-direction.css">
</head>
<body>
    <h3>row (default)</h3>
    <div class="container row">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>row-reverse</h3>
    <div class="container row-reverse">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>column</h3>
    <div class="container column">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>column-reverse</h3>
    <div class="container column-reverse">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>
</body>
</html>
```

**CSS File (`flex-direction.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

h3 {
    font-size: 0.9rem;
    color: #555;
    margin-bottom: 5px;
    margin-top: 20px;
}

.container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.row { flex-direction: row; }
.row-reverse { flex-direction: row-reverse; }
.column { flex-direction: column; }
.column-reverse { flex-direction: column-reverse; }

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
    min-width: 50px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `flex-direction.html` and CSS as `flex-direction.css`.
2. Open in a browser.
3. Observe the four containers: `row` shows items 1-2-3 left to right; `row-reverse` shows them right to left; `column` shows them top to bottom; `column-reverse` shows them bottom to top.

**Expected Output:** Four containers demonstrating the four `flex-direction` values. Items are laid out horizontally in the first two (in opposite directions) and vertically in the last two (in opposite directions).

**Why This Works:** The `flex-direction` property changes the main axis. `row` and `row-reverse` use the inline axis (horizontal in LTR), while `column` and `column-reverse` use the block axis (vertical). The `gap` property works identically in all directions.

---

### Real-World Cases

- **Horizontal navigation:** `flex-direction: row` for a top navigation bar.
- **Vertical sidebars:** `flex-direction: column` for a sidebar menu.
- **Mobile stacking:** Switching from `row` to `column` at breakpoints for responsive layouts.
- **Chat interfaces:** `column-reverse` for chat messages where the newest message appears at the bottom.

---

## 3. Wrapping Controls: `flex-wrap` and `flex-flow`

### Definitions

**Core Definition:** The `flex-wrap` property controls whether flex items are forced onto a single line or can wrap onto multiple lines. The `flex-flow` shorthand combines `flex-direction` and `flex-wrap`.

**Technical Definition:** The `flex-wrap` property specifies whether flex items are forced onto one line or can wrap onto multiple lines. If wrapping is allowed, it sets the direction that lines are stacked. Values are `nowrap`, `wrap`, and `wrap-reverse`. The `flex-flow` CSS shorthand property specifies the direction of a flex container, as well as its wrapping behavior. It is a shorthand for the `flex-direction` and `flex-wrap` properties. `flex-flow: row wrap` is equivalent to setting `flex-direction: row` and `flex-wrap: wrap`.

**Beginner-Friendly Explanation:** By default, flex items are all squeezed onto one line, even if they overflow. `flex-wrap: wrap` changes this — it allows items to wrap onto multiple lines when they run out of space. This is essential for responsive layouts where items should stack on smaller screens. `wrap-reverse` wraps items in the opposite direction (bottom-to-top instead of top-to-bottom). The `flex-flow` shorthand lets you set both the direction and wrapping in one declaration.

---

### Purposes

- To allow flex items to wrap onto multiple lines when they do not fit on one line.
- To prevent items from overflowing or being compressed into illegibility.
- To control the direction in which wrapped lines are stacked.
- To provide a shorthand for setting direction and wrapping together.
- To enable responsive layouts that adapt to different viewport sizes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    flex-wrap: nowrap | wrap | wrap-reverse;
    /* Shorthand: direction and wrap in one declaration */
    flex-flow: <flex-direction> || <flex-wrap>;
}
```

#### Component Breakdown

| Value | Description | Behaviour |
|---|---|---|
| `nowrap` | Default. Items are forced onto one line. | Items may overflow or shrink. |
| `wrap` | Items wrap onto multiple lines. | New lines are added in the cross-axis direction. |
| `wrap-reverse` | Items wrap onto multiple lines in reverse. | Lines are stacked in the reverse cross-axis direction. |

#### Syntax Rules

1. The initial value of `flex-wrap` is `nowrap`.
2. `flex-wrap: wrap` allows items to wrap onto new lines.
3. The direction of wrapping is determined by the cross axis (perpendicular to `flex-direction`).
4. `flex-flow` accepts one or both values (direction and wrap) in any order.
5. `flex-flow: row wrap` is equivalent to `flex-direction: row; flex-wrap: wrap;`.

#### Constraints and Limitations

- **`nowrap` can cause overflow** — items may overflow the container if they cannot shrink enough.
- **Wrapping affects alignment** — when wrapping occurs, `align-content` becomes applicable.
- **`wrap-reverse` reverses line order** — the first line appears at the cross-end side.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `nowrap` vs. `wrap` vs. `wrap-reverse`

**HTML File (`flex-wrap.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>flex-wrap Values</title>
    <link rel="stylesheet" href="flex-wrap.css">
</head>
<body>
    <h3>flex-wrap: nowrap (default) — items squeezed, may overflow</h3>
    <div class="container nowrap">
        <div class="item">Item 1</div>
        <div class="item">Item 2</div>
        <div class="item">Item 3</div>
        <div class="item">Item 4</div>
        <div class="item">Item 5</div>
    </div>

    <h3>flex-wrap: wrap — items wrap to new lines</h3>
    <div class="container wrap">
        <div class="item">Item 1</div>
        <div class="item">Item 2</div>
        <div class="item">Item 3</div>
        <div class="item">Item 4</div>
        <div class="item">Item 5</div>
    </div>

    <h3>flex-flow: row wrap-reverse — lines stacked in reverse</h3>
    <div class="container wrap-reverse">
        <div class="item">Item 1</div>
        <div class="item">Item 2</div>
        <div class="item">Item 3</div>
        <div class="item">Item 4</div>
        <div class="item">Item 5</div>
    </div>
</body>
</html>
```

**CSS File (`flex-wrap.css`):**

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

.container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    max-width: 350px;
}

.nowrap {
    flex-wrap: nowrap;
}

.wrap {
    flex-wrap: wrap;
}

.wrap-reverse {
    flex-flow: row wrap-reverse;
}

.item {
    background-color: #006064;
    color: white;
    padding: 12px;
    border-radius: 6px;
    font-size: 0.8rem;
    min-width: 80px;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `flex-wrap.html` and CSS as `flex-wrap.css`.
2. Open in a browser.
3. Observe the first container (`nowrap`): all five items are squeezed onto one line, overflowing or shrinking.
4. Observe the second container (`wrap`): items wrap onto multiple lines as needed.
5. Observe the third container (`wrap-reverse`): items wrap, but the lines are stacked in reverse order (first line appears at the bottom).

**Expected Output:** Three containers with different wrapping behaviour. In `nowrap`, items overflow the container width. In `wrap`, items wrap naturally. In `wrap-reverse`, the wrapping direction is reversed.

**Why This Works:** `flex-wrap: nowrap` keeps all items on one line, forcing them to shrink or overflow. `flex-wrap: wrap` allows items to break onto new lines when the container runs out of space. `flex-flow: row wrap-reverse` combines `flex-direction: row` and `flex-wrap: wrap-reverse`, causing lines to stack in reverse cross-axis order.

---

### Real-World Cases

- **Responsive card grids:** `flex-wrap: wrap` allows cards to wrap onto new rows on smaller screens.
- **Tag lists:** `flex-wrap: wrap` for tag components that flow onto multiple lines.
- **Navigation menus:** `flex-wrap: wrap` for menus that wrap on narrow screens.
- **Form rows:** `flex-wrap: wrap` for form fields that stack on mobile.

---

## 4. Main-Axis Alignment and Negative Space Management: `justify-content`

### Definitions

**Core Definition:** The `justify-content` property defines how the browser distributes space between and around content items along the main axis of a flex container.

**Technical Definition:** The `justify-content` CSS property defines how the browser distributes space between and around content items along the main axis of a flex container and the inline axis of grid and multicol containers. Values include `flex-start` (items packed toward the main-start side), `flex-end` (items packed toward the main-end side), `center` (items centered along the main axis), `space-between` (first item at the start, last item at the end, equal spacing between), `space-around` (equal spacing around each item, with half-size spaces at the edges), and `space-evenly` (equal spacing between items and at the edges).

**Beginner-Friendly Explanation:** `justify-content` controls how items are spread out along the main axis — the direction they flow. Think of it as controlling the "horizontal" spacing in a row (or "vertical" spacing in a column). `flex-start` packs items at the beginning; `center` centres them; `flex-end` packs them at the end. The space-* values spread them out: `space-between` puts the first item at the start and the last at the end with equal gaps between; `space-around` gives each item equal space on both sides (edges get half); `space-evenly` gives equal space everywhere, including the edges.

---

### Purposes

- To distribute free space along the main axis of a flex container.
- To align items at the start, centre, or end of the main axis.
- To spread items evenly with equal or varying spacing.
- To create balanced layouts without margin hacks.
- To manage negative space (overflow) when items exceed the container.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
}
```

#### Component Breakdown

| Value | Description | Space at Edges? |
|---|---|---|
| `flex-start` | Items packed toward the main-start side. | None (all space at the end). |
| `flex-end` | Items packed toward the main-end side. | None (all space at the start). |
| `center` | Items centered along the main axis. | Equal space at both ends. |
| `space-between` | First item at start, last at end, equal gaps between. | None at edges. |
| `space-around` | Equal space around each item. | Half-size spaces at edges. |
| `space-evenly` | Equal space between all items and at edges. | Full-size spaces at edges. |

#### Syntax Rules

1. `justify-content` operates on the **main axis** (defined by `flex-direction`).
2. The initial value is `flex-start` (or `normal`, which behaves as `flex-start`).
3. `space-between`, `space-around`, and `space-evenly` distribute free space.
4. If there is no free space (items overflow), `justify-content` has no effect on overflow positioning unless using `safe`/`unsafe` keywords.
5. Logical values (`start`, `end`) are preferred over physical values (`left`, `right`) for internationalization.

#### Constraints and Limitations

- **No effect on a single item** — with one item, `space-between` behaves like `flex-start`, and `space-around`/`space-evenly` center the item.
- **Overflow behaviour** — when items overflow, `justify-content` does not prevent overflow; use `safe`/`unsafe` to control alignment behaviour.
- **Axis dependency** — changing `flex-direction` changes which axis `justify-content` operates on.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `justify-content` Distribution Values

**HTML File (`justify.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>justify-content Values</title>
    <link rel="stylesheet" href="justify.css">
</head>
<body>
    <h3>flex-start</h3>
    <div class="container flex-start">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>center</h3>
    <div class="container center">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>space-between</h3>
    <div class="container space-between">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>space-around</h3>
    <div class="container space-around">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>space-evenly</h3>
    <div class="container space-evenly">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>
</body>
</html>
```

**CSS File (`justify.css`):**

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
    margin-top: 15px;
}

.container {
    display: flex;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    min-height: 50px;
}

.flex-start { justify-content: flex-start; }
.center { justify-content: center; }
.space-between { justify-content: space-between; }
.space-around { justify-content: space-around; }
.space-evenly { justify-content: space-evenly; }

.item {
    background-color: #006064;
    color: white;
    padding: 12px 18px;
    border-radius: 6px;
    font-weight: bold;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `justify.html` and CSS as `justify.css`.
2. Open in a browser.
3. Compare the five containers: `flex-start` packs items left; `center` centres them; `space-between` puts item 1 at the left, item 3 at the right, item 2 in the middle; `space-around` adds half-space at the edges; `space-evenly` adds equal space at the edges and between items.

**Expected Output:** Five containers showing different main-axis distributions. The items change position based on the `justify-content` value, while the container size remains constant.

**Why This Works:** `justify-content` distributes free space along the main axis. `flex-start` and `flex-end` pack items at one end; `center` centres them; `space-between` puts the first item at the start and the last at the end, distributing remaining space equally between items; `space-around` gives each item equal space on both sides (edges get half); `space-evenly` gives equal space everywhere.

---

### Real-World Cases

- **Navigation bars:** `justify-content: space-between` for a logo on the left and menu on the right.
- **Centering content:** `justify-content: center` for horizontally centering a hero section.
- **Evenly spaced buttons:** `justify-content: space-evenly` for a row of evenly spaced action buttons.
- **Card grids:** `justify-content: space-between` for cards that spread across the container.

---

## 5. Cross-Axis Single-Line Item Positioning: `align-items`

### Definitions

**Core Definition:** The `align-items` property sets the alignment of flex items along the cross axis of a single-line flex container.

**Technical Definition:** The CSS `align-items` property sets the `align-self` value on all direct children as a group. In Flexbox, it controls the alignment of items on the cross axis. Values include `stretch` (default; items stretch to fill the container's cross size), `flex-start` (items packed at the cross-start edge), `flex-end` (items packed at the cross-end edge), `center` (items centred on the cross axis), and `baseline` (items aligned by their text baseline). The `baseline` value aligns items based on their text baseline, which is useful for aligning text of different sizes.

**Beginner-Friendly Explanation:** `align-items` controls how items are aligned vertically within their line. If you have a row of items with different heights, `align-items: flex-start` puts them all at the top, `center` centres them vertically, `flex-end` puts them at the bottom, and `stretch` (the default) makes them all the same height. The `baseline` value aligns the text baselines of items, which is useful when you have items with different font sizes and want their text to line up.

---

### Purposes

- To align items along the cross axis within a single flex line.
- To control vertical alignment in a row layout (or horizontal alignment in a column layout).
- To stretch items to fill the cross-axis size of the container.
- To align items by their text baseline for typographic consistency.
- To provide a group-level alignment that individual items can override with `align-self`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    align-items: stretch | flex-start | flex-end | center | baseline;
}
```

#### Component Breakdown

| Value | Description | Behaviour |
|---|---|---|
| `stretch` | Default. Items stretch to fill the cross size. | Items expand to the container's cross size. |
| `flex-start` | Items packed at the cross-start edge. | Items align at the top (for row) or left (for column). |
| `flex-end` | Items packed at the cross-end edge. | Items align at the bottom (for row) or right (for column). |
| `center` | Items centred on the cross axis. | Items are vertically centred (for row). |
| `baseline` | Items aligned by their text baseline. | Text baselines line up across items. |

#### Syntax Rules

1. `align-items` operates on the **cross axis** (perpendicular to `flex-direction`).
2. The initial value is `stretch`.
3. `align-items` applies to all direct children as a group.
4. Individual items can override with `align-self`.
5. `baseline` aligns items by their first baseline (or last baseline if specified).
6. `align-items` has no effect on multi-line containers for line distribution; use `align-content` for that.

#### Constraints and Limitations

- **`baseline` requires text content** — items without text may not align as expected.
- **`stretch` is default** — if you do not set a cross-size on items, they stretch.
- **Cross-axis dependency** — changing `flex-direction` changes which axis `align-items` operates on.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `align-items` Alignment Values

**HTML File (`align-items.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>align-items Values</title>
    <link rel="stylesheet" href="align-items.css">
</head>
<body>
    <h3>align-items: flex-start</h3>
    <div class="container flex-start">
        <div class="item small">Small</div>
        <div class="item medium">Medium</div>
        <div class="item large">Large</div>
    </div>

    <h3>align-items: center</h3>
    <div class="container center">
        <div class="item small">Small</div>
        <div class="item medium">Medium</div>
        <div class="item large">Large</div>
    </div>

    <h3>align-items: flex-end</h3>
    <div class="container flex-end">
        <div class="item small">Small</div>
        <div class="item medium">Medium</div>
        <div class="item large">Large</div>
    </div>

    <h3>align-items: baseline</h3>
    <div class="container baseline">
        <div class="item small">Small</div>
        <div class="item medium">Medium</div>
        <div class="item large">Large</div>
    </div>
</body>
</html>
```

**CSS File (`align-items.css`):**

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
    margin-top: 15px;
}

.container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    min-height: 100px;
}

.flex-start { align-items: flex-start; }
.center { align-items: center; }
.flex-end { align-items: flex-end; }
.baseline { align-items: baseline; }

.item {
    background-color: #006064;
    color: white;
    padding: 10px 15px;
    border-radius: 6px;
    font-weight: bold;
}

.small { font-size: 0.75rem; }
.medium { font-size: 1rem; }
.large { font-size: 1.5rem; }
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `align-items.html` and CSS as `align-items.css`.
2. Open in a browser.
3. Observe the four containers: `flex-start` aligns items at the top; `center` centres them vertically; `flex-end` aligns them at the bottom; `baseline` aligns their text baselines (the bottom of the "S", "M", "L" letters line up).

**Expected Output:** Four containers showing different cross-axis alignments. The items have different font sizes, making the baseline alignment especially visible in the fourth container.

**Why This Works:** `align-items: flex-start` packs items at the cross-start edge (top for row). `center` centres them. `flex-end` packs them at the bottom. `baseline` aligns the text baselines of the items, so the bottoms of the letters line up regardless of font size. The `min-height: 100px` on the container provides visible space for the alignment to be observed.

---

### Real-World Cases

- **Navigation bars:** `align-items: center` to vertically centre nav links.
- **Card layouts:** `align-items: stretch` (default) to make cards equal height.
- **Form rows:** `align-items: baseline` to align labels and inputs by text baseline.
- **Icon + text combinations:** `align-items: center` to vertically centre an icon with text.

---

## 6. Multi-Line Cross-Axis Row Distribution: `align-content`

### Definitions

**Core Definition:** The `align-content` property sets the distribution of space between and around flex lines along the cross axis when there are multiple lines (i.e., when wrapping occurs).

**Technical Definition:** The `align-content` property sets the distribution of space between and around content items along a flexbox's cross axis, or a grid or block-level element's block axis. It has no effect on single-line flex containers (i.e., ones with `flex-wrap: nowrap`). Values include `flex-start` (lines packed to the cross-start), `flex-end` (lines packed to the cross-end), `center` (lines centered), `space-between` (first line at start, last at end, equal spacing between), `space-around` (equal space around each line, half at edges), `stretch` (default; lines stretch to fill remaining space), and `space-evenly`.

**Beginner-Friendly Explanation:** `align-content` is like `justify-content` but for the cross axis and for multiple lines. If your flex items wrap onto multiple rows, `align-content` controls how those rows are distributed vertically. `stretch` (the default) spreads them out to fill the container. `center` groups them in the middle. `space-between` puts the first row at the top, the last at the bottom, and equal space between. This property only works when wrapping is enabled (`flex-wrap: wrap` or `wrap-reverse`).

---

### Purposes

- To distribute space between and around multiple flex lines along the cross axis.
- To control the vertical positioning of rows in a wrapping flex container.
- To stretch lines to fill the container's cross size.
- To group lines at the start, centre, or end of the cross axis.
- To create balanced multi-row layouts without margin hacks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    align-content: flex-start | flex-end | center | space-between | space-around | space-evenly | stretch;
}
```

#### Component Breakdown

| Value | Description | Space at Cross Edges? |
|---|---|---|---|
| `stretch` | Default. Lines stretch to fill remaining cross space. | No — lines expand. |
| `flex-start` | Lines packed to the cross-start. | None (all space at the end). |
| `flex-end` | Lines packed to the cross-end. | None (all space at the start). |
| `center` | Lines centred on the cross axis. | Equal space at both ends. |
| `space-between` | First line at start, last at end, equal gaps between. | None at edges. |
| `space-around` | Equal space around each line. | Half-size spaces at edges. |
| `space-evenly` | Equal space between all lines and at edges. | Full-size spaces at edges. |

#### Syntax Rules

1. `align-content` operates on the **cross axis** and distributes **flex lines**.
2. It has **no effect** on single-line flex containers (`flex-wrap: nowrap`).
3. The initial value is `stretch`.
4. `align-content` applies to the flex container, not individual items.
5. It works in conjunction with `flex-wrap: wrap` or `wrap-reverse`.

#### Constraints and Limitations

- **Requires wrapping** — `align-content` has no effect without `flex-wrap: wrap`.
- **Not for single items** — it distributes lines, not items; use `align-items` for item alignment.
- **Axis dependency** — changing `flex-direction` changes which axis `align-content` operates on.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `align-content` with Wrapping

**HTML File (`align-content.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>align-content Values</title>
    <link rel="stylesheet" href="align-content.css">
</head>
<body>
    <h3>align-content: flex-start</h3>
    <div class="container flex-start">
        <div class="item">1</div><div class="item">2</div><div class="item">3</div>
        <div class="item">4</div><div class="item">5</div><div class="item">6</div>
    </div>

    <h3>align-content: center</h3>
    <div class="container center">
        <div class="item">1</div><div class="item">2</div><div class="item">3</div>
        <div class="item">4</div><div class="item">5</div><div class="item">6</div>
    </div>

    <h3>align-content: space-between</h3>
    <div class="container space-between">
        <div class="item">1</div><div class="item">2</div><div class="item">3</div>
        <div class="item">4</div><div class="item">5</div><div class="item">6</div>
    </div>
</body>
</html>
```

**CSS File (`align-content.css`):**

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
    margin-top: 15px;
}

.container {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    height: 200px;
    align-content: flex-start; /* Default for demonstration */
}

.flex-start { align-content: flex-start; }
.center { align-content: center; }
.space-between { align-content: space-between; }

.item {
    background-color: #006064;
    color: white;
    padding: 15px 20px;
    border-radius: 6px;
    font-weight: bold;
    font-size: 0.9rem;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `align-content.html` and CSS as `align-content.css`.
2. Open in a browser.
3. Observe the three containers: `flex-start` packs the wrapped lines at the top; `center` groups them in the middle; `space-between` puts the first line at the top, the last at the bottom, and equal space between.

**Expected Output:** Three containers with six items each, wrapping onto multiple lines. The lines are distributed differently based on the `align-content` value. The `height: 200px` provides extra vertical space for the distribution to be visible.

**Why This Works:** With `flex-wrap: wrap`, the six items wrap onto multiple lines. The `align-content` property then distributes the free space along the cross axis (vertical, since `flex-direction` is `row`). `flex-start` packs lines at the top; `center` centres them; `space-between` spreads them with the first at the top and last at the bottom.

---

### Real-World Cases

- **Card grids:** `align-content: space-between` for rows of cards that spread across the container height.
- **Wrapping navigation:** `align-content: center` for menus that wrap and should be vertically centred.
- **Dashboard widgets:** `align-content: flex-start` for widgets that should stack from the top.
- **Multi-row form layouts:** `align-content: space-around` for balanced vertical distribution of form rows.

---

## 7. Grid-Aligned Spatial Distribution: `gap`, `row-gap`, `column-gap`

### Definitions

**Core Definition:** The `gap` property (and its longhands `row-gap` and `column-gap`) sets the size of gutters (spacing) between flex items, rows, and columns, replacing the need for margin-based spacing calculations.

**Technical Definition:** The CSS `gap` property defines the gaps (gutters) between rows and columns. It is a shorthand for `row-gap` and `column-gap`. For flex containers, `gap` applies to both row and column gaps. The `gap` property is specified using the `row-gap` value and optionally the `column-gap` value. If `column-gap` is omitted, it uses the same value as `row-gap`. The `gap` property applies to multi-column elements, flex containers, and grid containers. Gutters effect a minimum spacing between items; the actual spacing may be larger if there is free space distributed by alignment properties.

**Beginner-Friendly Explanation:** `gap` is a simple way to add space between flex items without using margins. Instead of writing `.item { margin-right: 10px; }` and then having to remove the margin from the last item, you just write `gap: 10px` on the container. This creates equal spacing between all items. `row-gap` sets the spacing between rows (when wrapping), and `column-gap` sets the spacing between columns. `gap: 10px 20px` sets row-gap to 10px and column-gap to 20px.

---

### Purposes

- To create consistent spacing between flex items without margin-based hacks.
- To eliminate the need for `:last-child` margin resets and `calc()` calculations.
- To set separate spacing for rows and columns in wrapping layouts.
- To provide a unified spacing mechanism across flexbox, grid, and multi-column layouts.
- To simplify responsive layouts where spacing needs to change at breakpoints.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    gap: <row-gap> <column-gap>;
    /* Longhands */
    row-gap: <length> | <percentage>;
    column-gap: <length> | <percentage>;
}
```

#### Component Breakdown

| Property | Description | Applies To |
|---|---|---|
| `gap` | Shorthand for `row-gap` and `column-gap`. | Flex, Grid, Multi-column. |
| `row-gap` | Spacing between rows (flex lines). | Flex, Grid. |
| `column-gap` | Spacing between columns (items within a row). | Flex, Grid. |

#### Syntax Rules

1. `gap: 10px` sets both `row-gap` and `column-gap` to 10px.
2. `gap: 10px 20px` sets `row-gap: 10px` and `column-gap: 20px`.
3. `row-gap` and `column-gap` accept `<length>` and `<percentage>` values.
4. `gap` applies to flex containers, grid containers, and multi-column elements.
5. Gutters create a minimum spacing; alignment properties may add additional space.
6. `gap` does not apply to items that are not flex/grid/multi-column items.

#### Constraints and Limitations

- **Percentage gaps** — percentage values for `gap` refer to the content box size, which can be confusing.
- **Browser support** — `gap` in flexbox is supported in all modern browsers (Chrome 84+, Firefox 63+, Safari 14.1+).
- **No gap on non-flex/grid** — `gap` has no effect on regular block or inline layouts.
- **Gap vs. margin** — `gap` creates space between items, not around the outer edges; margins can still be used for outer spacing.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `gap` vs. Margin-Based Spacing

**HTML File (`gap.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>gap vs. Margin Spacing</title>
    <link rel="stylesheet" href="gap.css">
</head>
<body>
    <h3>Without gap — using margin-right</h3>
    <div class="container margin-spacing">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>With gap — clean and consistent</h3>
    <div class="container gap-spacing">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>gap with row-gap and column-gap</h3>
    <div class="container gap-row-column">
        <div class="item">1</div><div class="item">2</div><div class="item">3</div>
        <div class="item">4</div><div class="item">5</div><div class="item">6</div>
    </div>
</body>
</html>
```

**CSS File (`gap.css`):**

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
    margin-top: 15px;
}

.container {
    display: flex;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

/* Margin-based spacing (legacy) */
.margin-spacing .item {
    margin-right: 10px;
}

.margin-spacing .item:last-child {
    margin-right: 0; /* Must reset last item */
}

/* Gap-based spacing (modern) */
.gap-spacing {
    gap: 10px; /* One line, no resets needed */
}

/* Separate row and column gaps */
.gap-row-column {
    flex-wrap: wrap;
    gap: 15px 10px; /* 15px row-gap, 10px column-gap */
}

.gap-row-column .item {
    width: 100px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px 20px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `gap.html` and CSS as `gap.css`.
2. Open in a browser.
3. Compare the first two containers: they look identical, but the margin-based one requires a `:last-child` reset, while the gap-based one does not.
4. Observe the third container: it wraps and uses `gap: 15px 10px` for different row and column spacing.

**Expected Output:** Three containers. The first two look identical (three items with 10px spacing). The third wraps its items and has 15px between rows and 10px between columns.

**Why This Works:** The `gap: 10px` on `.gap-spacing` creates 10px gutters between all flex items without any margin hacks. The `gap: 15px 10px` on `.gap-row-column` sets `row-gap: 15px` and `column-gap: 10px`, demonstrating separate control over row and column spacing. This is far cleaner than the margin-based approach, which requires resetting the last item's margin.

---

### Real-World Cases

- **Card grids:** `gap: 20px` for equal spacing between cards.
- **Button groups:** `gap: 8px` for consistent spacing between buttons.
- **Form rows:** `gap: 15px` for spacing between form fields.
- **Tag lists:** `gap: 6px` for tightly spaced tags.
- **Responsive layouts:** Changing `gap` at breakpoints with a single declaration.

---

## References

- MDN Web Docs — Flex container - https://developer.mozilla.org/en-US/docs/Glossary/Flex_Container
- MDN Web Docs — `flex-direction` - https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction
- MDN Web Docs — Basic concepts of flexbox - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Basic_concepts
- MDN Web Docs — `justify-content` - https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content
- MDN Web Docs — `align-items` - https://developer.mozilla.org/en-US/docs/Web/CSS/align-items
- MDN Web Docs — `align-content` - https://developer.mozilla.org/en-US/docs/Web/CSS/align-content
- MDN Web Docs — `gap` - https://developer.mozilla.org/en-US/docs/Web/CSS/gap
- W3C — CSS Flexible Box Layout Module Level 1 - https://www.w3.org/TR/css-flexbox-1/
- CSS-Tricks — A Complete Guide to CSS Flexbox - https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- Can I Use — CSS Flexbox - https://caniuse.com/flexbox
- Can I Use — `gap` in Flexbox - https://caniuse.com/flex-gap