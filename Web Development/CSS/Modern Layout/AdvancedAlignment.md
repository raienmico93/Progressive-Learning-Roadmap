# CSS Advanced Alignment — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Advanced Alignment is the system of properties defined by the CSS Box Alignment Module Level 3 that controls how boxes are aligned and distributed within their containers across all CSS layout models — block layout, flex layout, grid layout, and multi-column layout. It provides a unified alignment vocabulary that works consistently across formatting contexts.

**Technical Definition:** The CSS Box Alignment Module Level 3 defines the features of CSS relating to the alignment of boxes within their containers in the various CSS box layout models: block layout, table layout, flex layout, and grid layout. The module introduces content-distribution properties (`align-content`, `justify-content`, and their `place-content` shorthand) that control alignment of the box's content within its content box, and self-alignment properties (`align-self`, `justify-self`, `align-items`, `justify-items`, and their shorthands) that control alignment of the box within its containing block. The properties operate along two axes: the block axis (cross axis in Flexbox) and the inline axis (main axis in Flexbox). The module also defines the `safe` and `unsafe` overflow alignment keywords and the `gap`, `row-gap`, and `column-gap` properties for distributed spacing.

**Beginner-Friendly Explanation:** Alignment is how you tell the browser where to put things inside a container. Do you want items centred? Pushed to the start? Spread out evenly? CSS has a complete set of alignment properties for this, and they work across Flexbox, Grid, and even normal block layout. The names follow a simple pattern: `justify-*` works on the inline axis (usually horizontal), and `align-*` works on the block axis (usually vertical). The `-content` properties distribute space between items as a group, the `-items` properties set a default alignment for all items, and the `-self` properties let you override the alignment for a single item. There are also shorthand properties (`place-*`) that set both axes at once, and a `gap` property that adds spacing between items without margins.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Unified model** | One alignment system works across Flexbox, Grid, Block, and Multi-column layout. |
| **Two axes** | `justify-*` operates on the inline/main axis; `align-*` operates on the block/cross axis. |
| **Three levels** | `-content` distributes tracks; `-items` sets defaults for all items; `-self` overrides for one item. |
| **Shorthand support** | `place-content`, `place-items`, and `place-self` combine both axes in one declaration. |
| **Overflow control** | `safe` and `unsafe` keywords control behaviour when items overflow the container. |
| **Distributed spacing** | `gap`, `row-gap`, and `column-gap` create gutters without margin hacks. |
| **Layout-mode awareness** | Flexbox ignores `justify-self` and `justify-items` on the main axis; Grid applies all alignment properties. |

---

### Prerequisites

Before studying CSS Advanced Alignment, you should understand:

- **CSS Box Model** — content, padding, border, and margin.
- **CSS Flexbox fundamentals** — flex containers, flex items, main axis, cross axis.
- **CSS Grid fundamentals** — grid containers, grid items, inline axis, block axis.
- **CSS Writing Modes** — how the inline and block axes are oriented.
- **The `gap` property** — basic spacing between layout items.

---

### Related Programming Areas

- **CSS Flexbox** — alignment properties control item placement on the main and cross axes.
- **CSS Grid Layout** — alignment properties control track distribution and item placement.
- **Responsive Design** — alignment enables flexible layouts that adapt to different viewport sizes.
- **UI Component Design** — navigation bars, card layouts, and form controls rely on alignment.
- **Internationalisation** — logical alignment values (`start`, `end`) adapt to writing modes.

---

### Core Concepts / Features

1. Block / Cross Axis Alignment: `align-content`, `align-items`, and `align-self`
2. Inline / Main Axis Alignment: `justify-content`, `justify-items`, and `justify-self`
3. Alignment Shorthands: `place-content`, `place-items`, and `place-self`
4. Flexbox vs. CSS Grid Rules: Why `justify-self` / `justify-items` Are Ignored in Flexbox
5. Safe and Unsafe Alignment: Preventing Data Loss and Layout Overflow
6. Distributed Spacing and Gaps: `gap`, `row-gap`, and `column-gap`

---

## 1. Block / Cross Axis Alignment: `align-content`, `align-items`, and `align-self`

### Definitions

**Core Definition:** Block-axis (or cross-axis in Flexbox) alignment controls how items are positioned vertically in horizontal writing modes. `align-content` distributes space between rows or flex lines, `align-items` sets the default alignment for all items, and `align-self` overrides that default for a single item.

**Technical Definition:** The `align-content` property sets the distribution of space between and around content items along a flexbox's cross axis, or a grid or block-level element's block axis. It applies to multi-line flex containers and grid containers. The `align-items` property specifies the default `align-self` for all boxes (including anonymous boxes) participating in the box's formatting context. The `align-self` property controls alignment of the box within its containing block along the block/cross axis. Values include `start`, `end`, `center`, `stretch`, `baseline`, `flex-start`, `flex-end`, and `safe`/`unsafe` prefixes.

**Beginner-Friendly Explanation:** `align-content` controls how rows (or flex lines) are distributed vertically within a container. It only works when there are multiple rows. `align-items` sets the default vertical alignment for all items in a container. `align-self` lets you override that default for one specific item. Think of it as: `align-content` moves the whole group of rows, `align-items` moves each item within its row, and `align-self` moves one item individually.

---

### Purposes

- To distribute vertical space between rows or flex lines (`align-content`).
- To set a consistent vertical alignment for all items in a container (`align-items`).
- To override the alignment of a single item (`align-self`).
- To align items by their text baseline for typographic consistency.
- To stretch items to fill the container's cross-axis size.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Content distribution (multi-line containers) */
align-content: normal | start | end | center | space-between | space-around | space-evenly | stretch | safe center | unsafe center;

/* Item alignment (default for all items) */
align-items: normal | stretch | center | start | end | baseline | first baseline | last baseline | safe center | unsafe center;

/* Self alignment (single item) */
align-self: auto | normal | stretch | center | start | end | baseline | first baseline | last baseline | safe center | unsafe center;
```

#### Component Breakdown

| Property | Level | Applies To | Description |
|---|---|---|---|
| `align-content` | Container | Multi-line flex, grid | Distributes space between rows/lines. |
| `align-items` | Container | Flex, grid | Sets default alignment for all items. |
| `align-self` | Item | Flex items, grid items | Overrides `align-items` for one item. |

#### Syntax Rules

1. `align-content` only affects multi-line containers (Flexbox with `flex-wrap: wrap`, or Grid with multiple rows).
2. `align-items` sets the default `align-self` for all items in the container.
3. `align-self: auto` resets to the value of the parent's `align-items`.
4. `baseline` aligns items by their text baseline; `first baseline` and `last baseline` specify which baseline.
5. `stretch` makes items fill the container's cross-axis size (default for `align-items`).
6. `safe` and `unsafe` control overflow behaviour.

#### Constraints and Limitations

- **`align-content` requires multiple lines** — it has no effect on single-line flex containers.
- **`align-self` only applies to items** — it has no effect on the container itself.
- **Baseline alignment** — requires text content; items without text may not align as expected.
- **Stretch requires `auto` cross size** — if the item has an explicit cross size, `stretch` has no effect.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Cross-Axis Alignment in Flexbox

**HTML File (`align-cross.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cross-Axis Alignment</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="align-cross.css">
</head>
<body>
    <h3>align-items: center</h3>
    <div class="container center">
        <div class="item">1</div>
        <div class="item tall">2</div>
        <div class="item">3</div>
    </div>

    <h3>align-self: flex-end on Item 2</h3>
    <div class="container self-end">
        <div class="item">1</div>
        <div class="item tall self">2</div>
        <div class="item">3</div>
    </div>

    <h3>align-content: space-between (wrapped)</h3>
    <div class="container content-between">
        <div class="item">1</div><div class="item">2</div><div class="item">3</div>
        <div class="item">4</div><div class="item">5</div><div class="item">6</div>
    </div>
</body>
</html>
```

**CSS File (`align-cross.css`):**

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
    min-height: 120px;
}

.center {
    align-items: center;
}

.self-end {
    align-items: start;
}

.self {
    align-self: flex-end;
}

.content-between {
    flex-wrap: wrap;
    align-content: space-between;
    height: 200px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
    min-width: 50px;
}

.tall {
    padding: 30px 15px;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `align-cross.html`.
3. Save the CSS code as `align-cross.css` in the same folder.
4. Open `align-cross.html` in a web browser.
5. Observe the first container: all items are vertically centred because of `align-items: center`.
6. Observe the second container: Item 2 is pushed to the bottom because of `align-self: flex-end`.
7. Observe the third container: the wrapped rows are distributed with equal space between them.

**Expected Output:** Three containers demonstrating `align-items`, `align-self`, and `align-content`. The first centres all items, the second overrides one item, and the third distributes rows evenly.

**Why This Works:** `align-items: center` sets the default cross-axis alignment for all flex items. `align-self: flex-end` overrides this default for Item 2, pushing it to the bottom of its flex line. `align-content: space-between` distributes the wrapped flex lines with equal spacing between them. All three properties operate on the block/cross axis.

---

### Real-World Cases

- **Navigation bars:** `align-items: center` to vertically centre logo and links.
- **Card layouts:** `align-items: stretch` (default) to make cards equal height.
- **Form rows:** `align-items: baseline` to align labels and inputs by text baseline.
- **Dashboard widgets:** `align-content: space-between` to distribute widget rows evenly.

---

## 2. Inline / Main Axis Alignment: `justify-content`, `justify-items`, and `justify-self`

### Definitions

**Core Definition:** Inline-axis (or main-axis in Flexbox) alignment controls how items are positioned horizontally in horizontal writing modes. `justify-content` distributes space between items along the inline axis, `justify-items` sets the default inline alignment for all items, and `justify-self` overrides that default for a single item.

**Technical Definition:** The `justify-content` property defines how the browser distributes space between and around content items along the main axis of a flex container and the inline axis of grid and multicol containers. The `justify-items` property defines the default `justify-self` for all items of the box, giving them all a default way of justifying each box along the appropriate axis. The `justify-self` property sets the way a box is justified inside its alignment container along the appropriate axis. In Flexbox, `justify-items` and `justify-self` are ignored on the main axis because Flexbox deals with items as a group, not individually.

**Beginner-Friendly Explanation:** `justify-content` is the horizontal equivalent of `align-content` — it spreads items out along the main axis. `justify-items` sets the default horizontal alignment for all items inside their grid cells. `justify-self` lets you override that for one item. The key difference from the `align-*` family is the axis: `justify-*` works on the inline/main axis, `align-*` works on the block/cross axis.

---

### Purposes

- To distribute horizontal space between items along the inline axis (`justify-content`).
- To set a consistent horizontal alignment for all grid items (`justify-items`).
- To override the horizontal alignment of a single grid item (`justify-self`).
- To centre, start-align, or end-align items along the main axis.
- To spread items evenly with `space-between`, `space-around`, or `space-evenly`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Content distribution */
justify-content: normal | start | end | center | space-between | space-around | space-evenly | stretch | safe center | unsafe center;

/* Item alignment (Grid only; ignored in Flexbox main axis) */
justify-items: normal | stretch | center | start | end | baseline | safe center | unsafe center;

/* Self alignment (Grid only; ignored in Flexbox main axis) */
justify-self: auto | normal | stretch | center | start | end | baseline | safe center | unsafe center;
```

#### Component Breakdown

| Property | Level | Applies To | Description |
|---|---|---|---|
| `justify-content` | Container | Flex, grid, multicol | Distributes space along main/inline axis. |
| `justify-items` | Container | Grid | Sets default inline alignment for items. |
| `justify-self` | Item | Grid items | Overrides `justify-items` for one item. |

#### Syntax Rules

1. `justify-content` works in Flexbox (main axis) and Grid (inline axis).
2. `justify-items` and `justify-self` work in Grid but are ignored in Flexbox on the main axis.
3. `justify-content: space-between` puts the first item at the start and the last at the end.
4. `justify-content: space-around` gives each item equal space on both sides.
5. `justify-content: space-evenly` gives equal space everywhere.
6. `stretch` is the default for `justify-items` in Grid when items have `auto` width.

#### Constraints and Limitations

- **Flexbox ignores `justify-self` and `justify-items`** — Flexbox treats items as a group on the main axis.
- **`justify-content` has no effect with one item** — `space-between` behaves like `flex-start` with a single item.
- **Overflow behaviour** — when items overflow, `justify-content` does not prevent overflow; use `safe`/`unsafe` to control alignment behaviour.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Main-Axis Alignment in Flexbox and Grid

**HTML File (`justify-main.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Main-Axis Alignment</title>
    <link rel="stylesheet" href="justify-main.css">
</head>
<body>
    <h3>Flexbox: justify-content: space-between</h3>
    <div class="flex-container between">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>Grid: justify-items: center</h3>
    <div class="grid-container center-items">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>

    <h3>Grid: justify-self: end on Item 2</h3>
    <div class="grid-container self-end">
        <div class="item">1</div>
        <div class="item self">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>
</body>
</html>
```

**CSS File (`justify-main.css`):**

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

.flex-container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.between {
    justify-content: space-between;
}

.grid-container {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.center-items {
    justify-items: center;
}

.self-end {
    justify-items: start;
}

.self {
    justify-self: end;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
    min-width: 60px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `justify-main.html` and CSS as `justify-main.css`.
2. Open in a browser.
3. Observe the Flexbox container: Item 1 is at the start, Item 3 at the end, and Item 2 is centred between them.
4. Observe the first Grid container: all items are centred within their grid cells.
5. Observe the second Grid container: Item 2 is pushed to the end of its cell because of `justify-self: end`.

**Expected Output:** Three containers demonstrating main/inline-axis alignment. The Flexbox container spreads items apart. The Grid containers show centred items and an individual override.

**Why This Works:** `justify-content: space-between` distributes free space along the main axis, pushing the first item to the start and the last to the end. `justify-items: center` sets the default inline alignment for all grid items. `justify-self: end` overrides this default for Item 2, pushing it to the end of its grid cell.

---

### Real-World Cases

- **Navigation bars:** `justify-content: space-between` for logo on the left and menu on the right.
- **Centred content:** `justify-content: center` for horizontally centring a hero section.
- **Grid galleries:** `justify-items: center` for centred gallery items.
- **Card grids:** `justify-self: end` for right-aligning a single card in a grid.

---

## 3. Alignment Shorthands: `place-content`, `place-items`, and `place-self`

### Definitions

**Core Definition:** The `place-content`, `place-items`, and `place-self` shorthand properties combine the `align-*` and `justify-*` properties for their respective levels into a single declaration, reducing code repetition.

**Technical Definition:** The `place-content` CSS shorthand property allows you to align content along both the block and inline directions at once (i.e., the `align-content` and `justify-content` properties) in a relevant layout system such as Grid or Flexbox. The `place-items` shorthand sets both `align-items` and `justify-items` in a single declaration. The `place-self` shorthand sets both `align-self` and `justify-self` for an individual item. The first value is assigned to the `align-*` property, and the second value is assigned to the `justify-*` property. If only one value is provided, it is used for both.

**Beginner-Friendly Explanation:** Instead of writing two lines to set both axes, you can write one. `place-content: center` is the same as `align-content: center; justify-content: center`. `place-items: center start` is the same as `align-items: center; justify-items: start`. These shorthands save time and make your CSS more readable.

---

### Purposes

- To set alignment on both axes with a single declaration.
- To reduce code repetition in alignment-heavy layouts.
- To improve readability of alignment declarations.
- To provide a consistent shorthand API across all three alignment levels.
- To simplify centring and distribution patterns.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* place-content: <align-content> <justify-content>? */
place-content: center;
place-content: space-between center;

/* place-items: <align-items> <justify-items>? */
place-items: center;
place-items: center start;

/* place-self: <align-self> <justify-self>? */
place-self: center;
place-self: end start;
```

#### Component Breakdown

| Shorthand | Longhands | First Value | Second Value |
|---|---|---|---|
| `place-content` | `align-content` + `justify-content` | `align-content` | `justify-content` |
| `place-items` | `align-items` + `justify-items` | `align-items` | `justify-items` |
| `place-self` | `align-self` + `justify-self` | `align-self` | `justify-self` |

#### Syntax Rules

1. If only one value is provided, it is used for both axes.
2. The first value always maps to the `align-*` property.
3. The second value always maps to the `justify-*` property.
4. If the second value is not present, the first value is used for both, provided it is valid for both.
5. If the value is invalid for one property, the whole declaration is invalid.
6. These shorthands work in Flexbox and Grid.

#### Constraints and Limitations

- **`place-items` in Flexbox** — `justify-items` is ignored in Flexbox main axis, so `place-items` only affects cross-axis alignment in Flexbox.
- **`place-self` in Flexbox** — `justify-self` is ignored, so `place-self` only affects cross-axis alignment in Flexbox.
- **Browser support** — all three shorthands are widely supported in modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Centring with `place-content`, `place-items`, and `place-self`

**HTML File (`place.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Place Shorthands</title>
    <link rel="stylesheet" href="place.css">
</head>
<body>
    <h3>place-content: center (Grid)</h3>
    <div class="grid place-content">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <h3>place-items: center start (Grid)</h3>
    <div class="grid place-items">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>

    <h3>place-self: end start on Item 2 (Grid)</h3>
    <div class="grid place-self">
        <div class="item">1</div>
        <div class="item self">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>
</body>
</html>
```

**CSS File (`place.css`):**

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

.grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    min-height: 100px;
}

.place-content {
    place-content: center;
}

.place-items {
    place-items: center start;
}

.place-self {
    place-items: start;
}

.self {
    place-self: end start;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
    min-width: 60px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `place.html` and CSS as `place.css`.
2. Open in a browser.
3. Observe the first grid: the entire grid content is centred within the container because of `place-content: center`.
4. Observe the second grid: items are centred vertically and start-aligned horizontally because of `place-items: center start`.
5. Observe the third grid: Item 2 is bottom-aligned and start-aligned because of `place-self: end start`.

**Expected Output:** Three grids demonstrating the shorthand properties. The first centres the whole grid content, the second sets default item alignment, and the third overrides a single item.

**Why This Works:** `place-content: center` sets both `align-content: center` and `justify-content: center`, centring the grid tracks within the container. `place-items: center start` sets `align-items: center` and `justify-items: start`. `place-self: end start` sets `align-self: end` and `justify-self: start` for the individual item.

---

### Real-World Cases

- **Centring modals:** `place-items: center` on a full-screen overlay grid.
- **Dashboard grids:** `place-content: center` for centred dashboard widgets.
- **Card layouts:** `place-self: end` for pinning a button to the bottom of a card.
- **Form layouts:** `place-items: center start` for centred, start-aligned form fields.

---

## 4. Flexbox vs. CSS Grid Rules: Why `justify-self` / `justify-items` Are Ignored in Flexbox

### Definitions

**Core Definition:** In Flexbox, `justify-self` and `justify-items` are ignored on the main axis because Flexbox treats all items as a single group on that axis. Grid, by contrast, applies all alignment properties because each grid item occupies its own grid area.

**Technical Definition:** In Flexbox layouts, `justify-self` is ignored because Flexbox deals with flex items as a group on the main axis. The `*-self` properties only work if the child is all alone in that axis; when there are multiple boxes to be aligned, the concept does not apply. In Grid layouts, `justify-self` aligns the item within its grid area on the inline axis. The Grid specification explicitly defines `justify-self` for grid items, while the Flexbox specification explicitly excludes it.

**Beginner-Friendly Explanation:** In a Flexbox row, items are treated as one group. You can move the group as a whole with `justify-content`, but you cannot move one item independently because there is no "cell" for it to be aligned within. In Grid, every item has its own cell, so `justify-self` can move it within that cell. This is a fundamental difference between the two layout models.

---

### Purposes

- To understand why alignment properties behave differently in Flexbox and Grid.
- To choose the right layout model for the alignment task.
- To debug alignment issues caused by using the wrong property in the wrong layout mode.
- To apply the correct alignment properties for each layout model.
- To design components that work correctly in both Flexbox and Grid contexts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Flexbox: justify-self/justify-items are IGNORED on main axis */
.flex-container {
    display: flex;
    justify-content: center; /* Works: moves the group */
}
.flex-item {
    justify-self: center; /* IGNORED */
}

/* Grid: justify-self/justify-items WORK */
.grid-container {
    display: grid;
    justify-items: center; /* Sets default for all items */
}
.grid-item {
    justify-self: center; /* Works: aligns item in its cell */
}
```

#### Component Breakdown

| Layout Mode | `justify-content` | `justify-items` | `justify-self` | `align-content` | `align-items` | `align-self` |
|---|---|---|---|---|---|---|
| Flexbox | ✅ Main axis | ❌ Ignored | ❌ Ignored | ✅ Cross axis (multi-line) | ✅ Cross axis | ✅ Cross axis |
| Grid | ✅ Inline axis | ✅ Inline axis | ✅ Inline axis | ✅ Block axis | ✅ Block axis | ✅ Block axis |

#### Syntax Rules

1. In Flexbox, `justify-self` and `justify-items` are ignored on the main axis.
2. In Flexbox, `align-self` works on the cross axis because each flex line has a cross size.
3. In Grid, all alignment properties work because each item has its own grid area.
4. To align a single Flexbox item on the main axis, use `margin: auto` on that item.
5. To align a single Grid item, use `justify-self` or `align-self`.

#### Constraints and Limitations

- **Flexbox group behaviour** — items cannot be individually aligned on the main axis.
- **Grid item independence** — each grid item is independent and can be aligned individually.
- **`margin: auto` workaround** — in Flexbox, `margin: auto` can absorb free space and effectively align an item.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `justify-self` Works in Grid but Not in Flexbox

**HTML File (`flex-vs-grid.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox vs Grid Alignment</title>
    <link rel="stylesheet" href="flex-vs-grid.css">
</head>
<body>
    <h3>Flexbox: justify-self is IGNORED</h3>
    <div class="flex-container">
        <div class="item">1</div>
        <div class="item justify-self">2 (justify-self: end)</div>
        <div class="item">3</div>
    </div>

    <h3>Grid: justify-self WORKS</h3>
    <div class="grid-container">
        <div class="item">1</div>
        <div class="item justify-self">2 (justify-self: end)</div>
        <div class="item">3</div>
    </div>
</body>
</html>
```

**CSS File (`flex-vs-grid.css`):**

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

.flex-container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.justify-self {
    justify-self: end;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `flex-vs-grid.html` and CSS as `flex-vs-grid.css`.
2. Open in a browser.
3. Observe the Flexbox container: Item 2 is **not** pushed to the end. The `justify-self: end` is ignored.
4. Observe the Grid container: Item 2 **is** pushed to the end of its cell because `justify-self: end` works in Grid.

**Expected Output:** Two containers with the same HTML. In the Flexbox container, Item 2 stays in its normal position. In the Grid container, Item 2 is pushed to the end of its cell.

**Why This Works:** Flexbox ignores `justify-self` on the main axis because items are treated as a group. Grid applies `justify-self` because each grid item has its own grid area. This is the fundamental difference between the two layout models.

---

### Real-World Cases

- **Navigation bars:** Use `justify-content` in Flexbox to distribute items, not `justify-self`.
- **Card grids:** Use `justify-self` in Grid to align individual cards within their cells.
- **Form layouts:** Use `margin: auto` in Flexbox to push a button to the end of a row.
- **Dashboard layouts:** Use Grid with `justify-self` for individual widget alignment.

---

## 5. Safe and Unsafe Alignment: Preventing Data Loss and Layout Overflow

### Definitions

**Core Definition:** The `safe` and `unsafe` keywords control what happens when an aligned item overflows its container. `safe` alignment changes the alignment mode to prevent data loss; `unsafe` alignment honours the specified alignment even if it causes overflow.

**Technical Definition:** The `<overflow-position>` value defines whether the alignment mode should be overridden to ensure the content is visible (`safe`) or if the alignment mode must be adhered to (`unsafe`). "Unsafe" alignment honors the specified alignment mode in overflow situations, even if it causes data loss, while "safe" alignment changes the alignment mode in overflow situations in an attempt to avoid data loss. If the overflow alignment isn't explicitly specified, the default overflow alignment is a blend of `safe` and `unsafe`. The default behaviour ("unsafe") doesn't prevent content from overflowing off the "start" side, and this cannot be scrolled to, and so is not accessible.

**Beginner-Friendly Explanation:** When you centre an item that is wider than its container, it overflows on both sides. The left side (or start side) is cut off and cannot be scrolled to. The `safe` keyword fixes this by changing the alignment to `start` when overflow occurs, so the content remains visible. The `unsafe` keyword keeps the centred alignment, even if it means the start side is cut off. The default is a blend of both, but `safe` is the accessible choice.

---

### Purposes

- To prevent data loss when aligned items overflow their container.
- To ensure content remains accessible and scrollable.
- To control whether alignment is honoured or overridden during overflow.
- To improve accessibility for users with zoomed-in views.
- To provide a predictable overflow behaviour for alignment properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    align-content: safe center | unsafe center;
    justify-content: safe center | unsafe center;
    align-items: safe center | unsafe center;
    justify-items: safe center | unsafe center;
    align-self: safe center | unsafe center;
    justify-self: safe center | unsafe center;
}
```

#### Component Breakdown

| Keyword | Behaviour | Accessibility |
|---|---|---|
| `safe` | Changes alignment to `start` on overflow. | ✅ Content remains visible. |
| `unsafe` | Honours alignment; content may be cut off. | ❌ Content may be inaccessible. |
| (omitted) | Blend of `safe` and `unsafe` (default). | ⚠️ Depends on browser. |

#### Syntax Rules

1. `safe` and `unsafe` are used as prefixes to positional alignment values (e.g., `safe center`).
2. `safe` changes the alignment to `start` when the item overflows.
3. `unsafe` honours the alignment regardless of overflow.
4. The default overflow alignment is a blend of `safe` and `unsafe`.
5. These keywords apply to all alignment properties: `align-content`, `justify-content`, `align-items`, `justify-items`, `align-self`, `justify-self`.
6. The `<overflow-position>` keywords are marked as at-risk in the specification.

#### Constraints and Limitations

- **Browser support** — `safe` and `unsafe` are supported in modern browsers but not Internet Explorer.
- **At-risk feature** — the `<overflow-position>` keywords are marked as "at-risk" and may be dropped during the CR period.
- **Default behaviour** — the default blend of `safe` and `unsafe` may vary between browsers.
- **Scroll safety** — safe alignment with scroll containers has additional complexity.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe vs. Unsafe Centring

**HTML File (`safe-unsafe.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Safe vs Unsafe Alignment</title>
    <link rel="stylesheet" href="safe-unsafe.css">
</head>
<body>
    <h3>unsafe center: content overflows start side</h3>
    <div class="container unsafe">
        <div class="item wide">This is a very wide item that overflows</div>
    </div>

    <h3>safe center: content remains visible</h3>
    <div class="container safe">
        <div class="item wide">This is a very wide item that overflows</div>
    </div>
</body>
</html>
```

**CSS File (`safe-unsafe.css`):**

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
    justify-content: center;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    width: 300px;
    overflow-x: auto;
}

.unsafe {
    justify-content: unsafe center;
}

.safe {
    justify-content: safe center;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
}

.wide {
    width: 500px; /* Wider than the container */
    flex-shrink: 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `safe-unsafe.html` and CSS as `safe-unsafe.css`.
2. Open in a browser.
3. Observe the first container: the wide item is centred, but the start side is cut off and cannot be scrolled to.
4. Observe the second container: the wide item is aligned to the start because `safe` changed the alignment, so the content is visible and scrollable.

**Expected Output:** Two containers with the same wide item. The `unsafe` container centres the item, cutting off the start side. The `safe` container aligns the item to the start, making it fully visible.

**Why This Works:** `unsafe center` honours the centred alignment even when it causes overflow. `safe center` detects the overflow and changes the alignment to `start`, ensuring the content remains visible and accessible. This is the key accessibility benefit of the `safe` keyword.

---

### Real-World Cases

- **Zoomed-in views:** `safe center` ensures content is visible when users zoom in.
- **Narrow containers:** `safe` alignment prevents content from being cut off in narrow viewports.
- **Accessibility compliance:** `safe` alignment helps meet WCAG requirements for content visibility.
- **Responsive layouts:** `safe` alignment adapts gracefully when content is wider than the container.

---

## 6. Distributed Spacing and Gaps: `gap`, `row-gap`, and `column-gap`

### Definitions

**Core Definition:** The `gap` property and its longhands `row-gap` and `column-gap` create consistent spacing (gutters) between flex items, grid tracks, and multi-column columns, without requiring margin hacks.

**Technical Definition:** The `gap` property is a shorthand that sets `row-gap` and `column-gap` in one declaration. If `column-gap` is omitted, it is set to the same value as `row-gap`. The `gap` property applies to multi-column containers, flex containers, and grid containers. In grid layouts, `gap` defines the space between rows and columns. In flexbox, `column-gap` defines the space between items along the main axis, and `row-gap` defines the space between flex lines. Note that the `gap` property is only one component of the visible "gutter" or "alley" created between boxes; margins, padding, or distributed alignment may increase the visible separation beyond what is specified in `gap`. Legacy aliases `grid-row-gap`, `grid-column-gap`, and `grid-gap` must be supported as aliases of the standard properties.

**Beginner-Friendly Explanation:** `gap` is a simple way to add space between items in a Flexbox or Grid container without using margins. Instead of adding `margin-right` to every item and then removing it from the last one, you just write `gap: 10px` on the container. `row-gap` sets the spacing between rows, and `column-gap` sets the spacing between columns. `gap: 10px 20px` sets `row-gap` to 10px and `column-gap` to 20px. The best part is that gaps do not affect the track sizing calculation — the browser subtracts the total gap space before distributing `fr` units.

---

### Purposes

- To create consistent spacing between flex items and grid tracks.
- To eliminate the need for margin-based spacing hacks.
- To set separate spacing for rows and columns.
- To provide a unified spacing mechanism across flexbox, grid, and multi-column layouts.
- To simplify responsive layouts where spacing changes at breakpoints.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    gap: <row-gap> <column-gap>?;
    row-gap: <length> | <percentage>;
    column-gap: <length> | <percentage>;
}
```

#### Component Breakdown

| Property | Description | Applies To |
|---|---|---|
| `gap` | Shorthand for `row-gap` and `column-gap`. | Flex, Grid, Multi-column. |
| `row-gap` | Spacing between rows (flex lines or grid rows). | Flex, Grid. |
| `column-gap` | Spacing between columns (flex items or grid columns). | Flex, Grid, Multi-column. |

#### Syntax Rules

1. `gap: 10px` sets both `row-gap` and `column-gap` to 10px.
2. `gap: 10px 20px` sets `row-gap: 10px` and `column-gap: 20px`.
3. `row-gap` and `column-gap` accept `<length>` and `<percentage>` values.
4. `gap` applies to flex containers, grid containers, and multi-column elements.
5. Gutters are not part of the track sizing calculation — they reduce the free space available for `fr` units.
6. Gutters create a minimum spacing; alignment properties may add additional space.
7. Legacy `grid-row-gap`, `grid-column-gap`, and `grid-gap` are aliases of the standard properties.

#### Constraints and Limitations

- **Percentage gaps** — percentage values for `gap` refer to the content box size, which can be confusing.
- **Gap in the implicit grid** — gaps apply between implicit tracks as well as explicit tracks.
- **No gap on non-flex/grid** — `gap` has no effect on regular block or inline layouts.
- **Gap vs. margin** — `gap` creates space between items, not around the outer edges; padding can be used for outer spacing.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Uniform vs. Separate Row and Column Gaps

**HTML File (`gap.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gap Property</title>
    <link rel="stylesheet" href="gap.css">
</head>
<body>
    <h3>Uniform gap: gap: 15px</h3>
    <div class="grid uniform-gap">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
        <div class="item">5</div>
        <div class="item">6</div>
    </div>

    <h3>Separate gaps: gap: 20px 10px</h3>
    <div class="grid separate-gap">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
        <div class="item">5</div>
        <div class="item">6</div>
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
    margin-top: 20px;
}

.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.uniform-gap {
    gap: 15px;
}

.separate-gap {
    gap: 20px 10px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `gap.html` and CSS as `gap.css`.
2. Open in a browser.
3. Observe the first grid: the gaps between rows and columns are equal (15px).
4. Observe the second grid: the gaps between rows (20px) are larger than the gaps between columns (10px).

**Expected Output:** Two grids with six items each. The first has uniform 15px gaps. The second has 20px row gaps and 10px column gaps.

**Why This Works:** `gap: 15px` sets both `row-gap` and `column-gap` to 15px. `gap: 20px 10px` sets `row-gap: 20px` and `column-gap: 10px`. Gutters are applied between tracks (not around the edges), and they are subtracted from the container's inner size before `fr` units are distributed.

---

### Real-World Cases

- **Card grids:** `gap: 20px` for consistent spacing between cards.
- **Page layouts:** `gap: 0 20px` for column gaps without row gaps.
- **Dashboard grids:** `gap: 16px 24px` for different row and column spacing.
- **Responsive layouts:** Changing `gap` at breakpoints with a single declaration.

---

## References

- W3C — CSS Box Alignment Module Level 3 - https://www.w3.org/TR/css-align-3/
- MDN Web Docs — `align-content` - https://developer.mozilla.org/en-US/docs/Web/CSS/align-content
- MDN Web Docs — `align-items` - https://developer.mozilla.org/en-US/docs/Web/CSS/align-items
- MDN Web Docs — `align-self` - https://developer.mozilla.org/en-US/docs/Web/CSS/align-self
- MDN Web Docs — `justify-content` - https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content
- MDN Web Docs — `justify-items` - https://developer.mozilla.org/en-US/docs/Web/CSS/justify-items
- MDN Web Docs — `justify-self` - https://developer.mozilla.org/en-US/docs/Web/CSS/justify-self
- MDN Web Docs — `place-content` - https://developer.mozilla.org/en-US/docs/Web/CSS/place-content
- MDN Web Docs — `place-items` - https://developer.mozilla.org/en-US/docs/Web/CSS/place-items
- MDN Web Docs — `place-self` - https://developer.mozilla.org/en-US/docs/Web/CSS/place-self
- MDN Web Docs — `<overflow-position>` - https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-position
- MDN Web Docs — Box alignment in flexbox - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_alignment/Box_alignment_in_flexbox
- MDN Web Docs — Box alignment in grid layout - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_alignment/Box_alignment_in_grid_layout
- MDN Web Docs — `gap` - https://developer.mozilla.org/en-US/docs/Web/CSS/gap
- CSS-Tricks — A Complete Guide to Flexbox - https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- CSS-Tricks — A Complete Guide to CSS Grid - https://css-tricks.com/snippets/css/complete-guide-grid/
- Can I Use — CSS Box Alignment - https://caniuse.com/css-box-alignment