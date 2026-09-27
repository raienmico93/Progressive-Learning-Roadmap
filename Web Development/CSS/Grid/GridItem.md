# CSS Grid Item Placement & Explicit Coordinate Mapping — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Grid Item Placement and Explicit Coordinate Mapping is the system of CSS properties that position grid items within a grid container using explicit line coordinates, named lines, or named areas. It encompasses the shorthand properties `grid-column`, `grid-row`, and `grid-area`, the longhand properties `grid-column-start`, `grid-column-end`, `grid-row-start`, and `grid-row-end`, the `span` keyword for spanning tracks, and the mechanisms for overlapping items at the same grid coordinates.

**Technical Definition:** The CSS Grid Layout Module Level 1 defines a set of grid-placement properties that determine a grid item's size and location within the grid by contributing a line, a span, or nothing (automatic) to its grid placement, thereby specifying the inline-start, block-start, inline-end, and block-end edges of its grid area. The `grid-row-start`, `grid-column-start`, `grid-row-end`, and `grid-column-end` longhands each accept `<grid-line>` values, which can be `auto`, a `<custom-ident>` (named line or named area), an `<integer>` (line index), or a `span` expression. The `grid-row` and `grid-column` shorthands combine start and end values separated by a forward slash (`/`). The `grid-area` shorthand combines all four longhands, accepting one to four values in the order row-start / column-start / row-end / column-end. Grid lines are indexed from 1, respecting the writing mode of the document. Negative integers count backward from the end edge of the explicit grid. Named lines are declared in square brackets within `grid-template-columns` and `grid-template-rows`. Named areas are declared with `grid-template-areas` and automatically generate implicit named lines of the form `<area-name>-start` and `<area-name>-end`. Items can be placed in the same grid area to overlap, with stacking order controlled by `z-index`.

**Beginner-Friendly Explanation:** When you create a grid, the browser automatically numbers the lines between the rows and columns. You can place items by saying "start at line 2 and end at line 4" — that spans two tracks. You can also name your lines (like `[main-start]` and `[main-end]`) to make your code more readable. Or you can draw a map of your layout using named areas (like `"header header" "sidebar main"`) and assign items to those areas by name. If you place two items in the same grid cell, they overlap — and you can control which one is on top with `z-index`. This system gives you precise, readable control over where every item goes.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Line-based coordinate system** | Grid lines are numbered from 1, respecting the writing mode. |
| **Negative line indexing** | Negative integers count backward from the end of the explicit grid. |
| **Span keyword** | `span <integer>` or `span <name>` extends an item across multiple tracks. |
| **Named lines** | Lines can be named in square brackets within track definitions. |
| **Named areas** | `grid-template-areas` creates named areas and implicit named lines. |
| **Overlapping placement** | Multiple items can occupy the same grid area; `z-index` controls stacking. |
| **Shorthand properties** | `grid-column`, `grid-row`, and `grid-area` combine multiple longhands. |

---

### Prerequisites

Before studying grid item placement, you should understand:

- **CSS Grid fundamentals** — grid containers, grid items, tracks, lines, cells, and areas.
- **Grid container properties** — `display: grid`, `grid-template-columns`, `grid-template-rows`, `grid-template-areas`.
- **The CSS Box Model** — content, padding, border, and margin.
- **Stacking contexts and `z-index`** — how elements are layered in the visual order.

---

### Related Programming Areas

- **CSS Flexbox** — shares the `order` and `z-index` concepts for reordering and layering.
- **Responsive Design** — line-based placement and named areas adapt well to media queries.
- **UI Component Design** — dashboard layouts, page shells, and card grids rely on grid placement.
- **Web Accessibility** — visual reordering via grid placement can create DOM-order mismatches.

---

### Core Concepts / Features

1. Shorthand Coordinate Routing: `grid-column` and `grid-row`
2. Line-Based Placement: `grid-column-start`, `grid-column-end`, and Related Longhands
3. Semantic Grid Systems: Named Lines
4. Named Area Registration: `grid-area` and `grid-template-areas`
5. Overlapping Items: Layering at the Same Coordinates

---

## 1. Shorthand Coordinate Routing: Configuring Explicit Item Spans with `grid-column` and `grid-row`

### Definitions

**Core Definition:** The `grid-column` and `grid-row` CSS properties are shorthands that specify a grid item's start and end lines in a single declaration, separated by a forward slash (`/`).

**Technical Definition:** The `grid-row` shorthand sets the `grid-row-start` and `grid-row-end` longhands, while `grid-column` sets `grid-column-start` and `grid-column-end`. Each accepts one or two `<grid-line>` values separated by `/`. If only one value is given, it sets the start line and the end line is set to `auto`. The `<grid-line>` value can be `auto`, a `<custom-ident>` (named line or named area), an `<integer>` (line index), or a `span` expression. The shorthand is defined in the CSS Grid Layout Module Level 1 §8.3 and follows the pattern `grid-row: <grid-line> [ / <grid-line> ]?`.

**Beginner-Friendly Explanation:** Instead of writing four separate lines to place an item (`grid-column-start`, `grid-column-end`, `grid-row-start`, `grid-row-end`), you can write two: `grid-column: 2 / 4` and `grid-row: 1 / 3`. The value before the slash is the start line, and the value after is the end line. If you only give one value, the item spans one track by default. You can also use `span` to say "span this many tracks from wherever I am."

---

### Purposes

- To reduce repetition when placing grid items with both start and end coordinates.
- To provide a concise syntax for line-based placement.
- To enable readable placement code that mirrors the visual layout.
- To support span-based placement with the `span` keyword.
- To serve as the primary shorthand for grid item placement.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    grid-column: <grid-line> [ / <grid-line> ]?;
    grid-row: <grid-line> [ / <grid-line> ]?;
}

/* <grid-line> values */
<grid-line> = auto | <custom-ident> | [ <integer> && <custom-ident>? ] | [ span && [ <integer> || <custom-ident> ] ]
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<grid-line>` | A line, a span, or `auto`. | `2`, `main-start`, `span 2` |
| `/` | Separator between start and end values. | `2 / 4` |
| `auto` | Default. Auto-placement or default span of 1. | `auto` |
| `<integer>` | Line index (positive or negative, not zero). | `1`, `-1`, `3` |
| `<custom-ident>` | Named line or named area. | `main-start`, `header` |
| `span <integer>` | Span N tracks. | `span 3` |
| `span <name>` | Span to a named line. | `span content-end` |

#### Syntax Rules

1. The value before the slash sets the start longhand; the value after sets the end longhand.
2. If only one value is given, the start longhand is set and the end longhand is set to `auto`.
3. The `<integer>` value cannot be zero.
4. Negative integers count backward from the end of the explicit grid.
5. The `span` keyword can be combined with an integer or a name.
6. If both start and end are `auto`, the item is auto-placed.
7. Named areas implicitly create named lines of the form `<name>-start` and `<name>-end`.

#### Constraints and Limitations

- **Zero is invalid** — `<integer>` values of zero make the declaration invalid.
- **Span cannot be used with both start and end** — the `span` keyword is used for one edge only.
- **Named line ambiguity** — if multiple lines share a name, only lines with that name are counted.
- **Writing mode dependency** — line 1 is on the left in LTR and on the right in RTL.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Shorthand Placement

**HTML File (`shorthand.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>grid-column and grid-row Shorthands</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="shorthand.css">
</head>
<body>
    <!-- Grid container with four items -->
    <div class="grid">
        <div class="item item-1">Item 1 (column 1/3, row 1/2)</div>
        <div class="item item-2">Item 2 (column 3/4, row 1/2)</div>
        <div class="item item-3">Item 3 (column 1/2, row 2/4)</div>
        <div class="item item-4">Item 4 (column 2/4, row 2/3)</div>
    </div>
</body>
</html>
```

**CSS File (`shorthand.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.grid {
    display: grid;
    /* Three columns, two rows */
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: 100px 100px;
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
    font-weight: bold;
    font-size: 0.75rem;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
}

.item-1 {
    /* Start at column line 1, end at column line 3 (spans 2 columns) */
    grid-column: 1 / 3;
    /* Start at row line 1, end at row line 2 (spans 1 row) */
    grid-row: 1 / 2;
}

.item-2 {
    grid-column: 3 / 4;
    grid-row: 1 / 2;
}

.item-3 {
    grid-column: 1 / 2;
    /* Span two rows */
    grid-row: 2 / 4;
}

.item-4 {
    /* Span two columns */
    grid-column: 2 / 4;
    grid-row: 2 / 3;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `shorthand.html`.
3. Save the CSS code as `shorthand.css` in the same folder.
4. Open `shorthand.html` in a web browser.
5. Observe the placement: Item 1 spans two columns in the first row, Item 2 is in the third column, Item 3 spans two rows in the first column, and Item 4 spans two columns in the second row.

**Expected Output:** A grid with four items placed at specific coordinates. Item 1 spans two columns, Item 3 spans two rows, and the other items occupy single cells.

**Why This Works:** The `grid-column: 1 / 3` on Item 1 sets `grid-column-start: 1` and `grid-column-end: 3`, spanning two column tracks. The `grid-row: 1 / 2` sets `grid-row-start: 1` and `grid-row-end: 2`, spanning one row. The shorthand reduces four declarations to two per item. Item 3 uses `grid-row: 2 / 4` to span two rows.

---

### Real-World Cases

- **Magazine layouts:** Articles spanning multiple columns with images and pull quotes placed at specific coordinates.
- **Dashboard grids:** Widgets placed at precise line coordinates for asymmetric layouts.
- **Form grids:** Labels and inputs aligned to specific column lines for consistent vertical rhythm.
- **Card grids:** Cards spanning different numbers of columns and rows.

---

## 2. Line-Based Placement: Routing Objects via Specific Line Indices (`grid-column-start`, `grid-column-end`)

### Definitions

**Core Definition:** Line-based placement is the technique of positioning grid items by specifying the grid lines (by number or name) at which the item starts and ends along each axis.

**Technical Definition:** The `grid-row-start`, `grid-column-start`, `grid-row-end`, and `grid-column-end` properties determine a grid item's size and location within the grid by contributing a line, a span, or nothing (automatic) to its grid placement. Each property accepts a `<grid-line>` value. Positive integers count from the start edge of the explicit grid; negative integers count from the end edge. Named lines are declared in square brackets within `grid-template-columns` and `grid-template-rows`. The `span` keyword contributes a grid span to the item's placement such that the corresponding edge is N lines from the opposite edge. If a line name is given with `span`, only lines with that name are counted.

**Beginner-Friendly Explanation:** Every grid has numbered lines. In a 3-column grid, there are 4 column lines (1, 2, 3, 4). You can place an item by saying "start at line 2 and end at line 4" — that spans columns 2 and 3. You can also count from the end: line -1 is the last line. If you name your lines, you can use those names instead of numbers, which makes your code more readable. The `span` keyword lets you say "span this many tracks" instead of specifying an exact end line.

---

### Purposes

- To provide precise control over item placement using line indices.
- To allow items to span multiple tracks by specifying start and end lines.
- To support negative indexing for placement relative to the end of the grid.
- To enable named-line placement for more readable code.
- To provide the underlying longhands for the `grid-column` and `grid-row` shorthands.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    grid-column-start: <grid-line>;
    grid-column-end: <grid-line>;
    grid-row-start: <grid-line>;
    grid-row-end: <grid-line>;
}
```

#### Component Breakdown

| Property | Description | Accepts |
|---|---|---|
| `grid-column-start` | The column line where the item starts. | `<grid-line>` |
| `grid-column-end` | The column line where the item ends. | `<grid-line>` |
| `grid-row-start` | The row line where the item starts. | `<grid-line>` |
| `grid-row-end` | The row line where the item ends. | `<grid-line>` |

#### The `<grid-line>` Value

| Value | Description | Example |
|---|---|---|
| `auto` | Auto-placement or default span of 1. | `auto` |
| `<integer>` | Line index (positive from start, negative from end). | `2`, `-1` |
| `<custom-ident>` | Named line or named area. | `main-start`, `header` |
| `<integer> <custom-ident>` | Nth line with that name. | `2 main` |
| `span <integer>` | Span N tracks from the opposite edge. | `span 2` |
| `span <custom-ident>` | Span to the named line. | `span content-end` |

#### Syntax Rules

1. Positive integers count from the start edge (line 1 is the first line).
2. Negative integers count from the end edge (line -1 is the last line).
3. Zero is invalid as an integer value.
4. Named lines are declared in square brackets in track definitions.
5. The `span` keyword can be combined with an integer or a name.
6. If a named line does not exist, implicit grid lines are assumed to have that name.
7. If both start and end are `auto`, the item is auto-placed with a span of 1.

#### Constraints and Limitations

- **Implicit grid lines** — lines outside the explicit grid cannot be referenced by number but can be referenced by name (implicitly).
- **Named line sets** — if multiple lines share a name, they form a named set indexed by order.
- **Writing mode dependency** — line numbering is relative to the writing mode.
- **Span ambiguity** — `span` with a name searches in the corresponding direction; if not enough named lines exist, implicit lines are assumed.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Line-Based Placement with Positive and Negative Indices

**HTML File (`lines.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Line-Based Placement</title>
    <link rel="stylesheet" href="lines.css">
</head>
<body>
    <div class="grid">
        <!-- Spans from line 1 to line 3 (positive indices) -->
        <div class="item item-1">Lines 1 / 3 (positive)</div>
        <!-- Spans from line -4 to line -2 (negative indices) -->
        <div class="item item-2">Lines -4 / -2 (negative)</div>
        <!-- Uses span keyword -->
        <div class="item item-3">Span 2 from auto</div>
    </div>
</body>
</html>
```

**CSS File (`lines.css`):**

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
    /* Five columns, two rows */
    grid-template-columns: repeat(5, 1fr);
    grid-template-rows: 100px 100px;
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
    font-weight: bold;
    font-size: 0.75rem;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
}

.item-1 {
    /* Positive indices: start at line 1, end at line 3 (spans 2 columns) */
    grid-column-start: 1;
    grid-column-end: 3;
    grid-row-start: 1;
    grid-row-end: 2;
}

.item-2 {
    /* Negative indices: start at line -4 (which is line 2), end at line -2 (which is line 4) */
    grid-column-start: -4;
    grid-column-end: -2;
    grid-row-start: 1;
    grid-row-end: 2;
}

.item-3 {
    /* Span keyword: starts at auto (next available), spans 2 columns */
    grid-column-start: span 2;
    grid-row-start: 2;
    grid-row-end: 3;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `lines.html` and CSS as `lines.css`.
2. Open in a browser.
3. Observe the placement: Item 1 spans columns 1–2 (lines 1 to 3). Item 2 uses negative indices to span columns 2–3 (lines -4 to -2). Item 3 spans two columns starting from the auto-placed position.

**Expected Output:** A grid with five columns. Item 1 occupies the first two columns. Item 2 also occupies columns 2–3 (overlapping with Item 1 partially). Item 3 spans two columns in the second row.

**Why This Works:** Positive indices count from the start of the explicit grid. Negative indices count from the end. In a 5-column grid with 6 column lines, line -4 is the same as line 2, and line -2 is the same as line 4. The `span 2` on Item 3 tells the browser to span two tracks from its auto-placed position.

---

#### Example 2: Named Line Placement

**HTML File (`named-lines.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Named Line Placement</title>
    <link rel="stylesheet" href="named-lines.css">
</head>
<body>
    <div class="grid">
        <div class="item item-1">Header</div>
        <div class="item item-2">Sidebar</div>
        <div class="item item-3">Main</div>
    </div>
</body>
</html>
```

**CSS File (`named-lines.css`):**

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
    /* Named lines in square brackets */
    grid-template-columns: [sidebar-start] 150px [sidebar-end main-start] 1fr [main-end];
    grid-template-rows: [header-start] 60px [header-end content-start] 1fr [content-end];
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    min-height: 300px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    font-size: 0.8rem;
    display: flex;
    align-items: center;
    justify-content: center;
}

.item-1 {
    /* Span from header-start to main-end (full width) */
    grid-column: header-start / main-end;
    grid-row: header-start / header-end;
}

.item-2 {
    /* Span from sidebar-start to sidebar-end */
    grid-column: sidebar-start / sidebar-end;
    grid-row: content-start / content-end;
}

.item-3 {
    /* Span from main-start to main-end */
    grid-column: main-start / main-end;
    grid-row: content-start / content-end;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `named-lines.html` and CSS as `named-lines.css`.
2. Open in a browser.
3. Observe the layout: Header spans the full width, Sidebar is on the left, and Main is on the right.

**Expected Output:** A page layout with a header spanning the full width, a sidebar on the left, and main content on the right. The placement uses named lines instead of numbers.

**Why This Works:** The `grid-template-columns` and `grid-template-rows` define named lines in square brackets. The `grid-column` and `grid-row` properties reference those names. The line `[sidebar-end main-start]` gives a single line two names, so it can be referenced by either. Named lines make the CSS more readable and maintainable because the names describe the layout's intent.

---

### Real-World Cases

- **Page layouts:** Header, sidebar, main, and footer placed using named lines like `[header-start]` and `[content-end]`.
- **Responsive layouts:** Named lines remain constant across media queries, so placement code does not need to change.
- **Dashboard grids:** Widgets placed at named lines for consistent alignment.
- **Form grids:** Labels and inputs aligned to named lines like `[label-end input-start]`.

---

## 3. Semantic Grid Systems: Designing Layouts Using Custom Named Lines

### Definitions

**Core Definition:** Named lines are CSS Grid lines that are assigned one or more names using square bracket syntax within `grid-template-columns` and `grid-template-rows`, allowing items to be placed by name instead of by number.

**Technical Definition:** Line names are assigned with `grid-template-rows` and `grid-template-columns` by writing names in square brackets before or after track sizing functions. A line can have multiple names, separated by whitespace within the square brackets. Named grid areas declared with `grid-template-areas` automatically generate implicit named lines of the form `<area-name>-start` and `<area-name>-end`. When multiple lines share a name, they form a named set, and placement by that name counts only lines with that name. If a name is given as a `<custom-ident>`, the placement algorithm first attempts to match it to a named grid area (by looking for `<name>-start` or `<name>-end`), then to a named line.

**Beginner-Friendly Explanation:** Instead of remembering that "the sidebar ends at line 2," you can name that line `[sidebar-end main-start]` and refer to it by name. This makes your CSS self-documenting: anyone reading the code can understand the layout intent. You can give a line multiple names — for example, one line can be both `[sidebar-end]` and `[main-start]` because it marks the boundary between the two. Named lines are especially useful for responsive layouts, because the names stay the same even when the grid structure changes.

---

### Purposes

- To make grid placement code more readable and self-documenting.
- To provide semantic names for grid lines that describe the layout's intent.
- To allow a single line to serve multiple purposes with multiple names.
- To simplify responsive layouts where line numbers change but names remain constant.
- To combine line-based placement with named areas for a flexible, readable system.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
.container {
    grid-template-columns: [<name>] <track-size> [<name> <name>] <track-size> [<name>];
    grid-template-rows: [<name>] <track-size> [<name>] <track-size> [<name>];
}

.item {
    grid-column: <name> / <name>;
    grid-row: <name> / <name>;
}
```

#### Component Breakdown

| Syntax | Description | Example |
|---|---|---|
| `[name]` | A single line name. | `[sidebar-start]` |
| `[name1 name2]` | A line with multiple names. | `[sidebar-end main-start]` |
| `<name>-start` | Implicit line from a named area. | `header-start` |
| `<name>-end` | Implicit line from a named area. | `header-end` |

#### Syntax Rules

1. Line names are written in square brackets before or after track sizing functions.
2. Multiple names on one line are separated by whitespace.
3. A line name cannot be `span` or `auto`.
4. Named areas automatically create `<name>-start` and `<name>-end` lines.
5. If multiple lines share a name, they form a named set.
6. Named lines can be mixed with numeric indices in placement properties.
7. Named lines are not affected by `grid-auto-flow` or implicit grid generation.

#### Constraints and Limitations

- **Naming conflicts** — if a named area and a named line share a name, the area's lines take precedence.
- **Implicit named lines** — `<area-name>-start` and `<area-name>-end` lines are generated automatically but do not appear in the `grid-template-*` value.
- **Multiple names** — a line can have many names, but only the first is used for serialization.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Names on a Single Line

**HTML File (`multi-names.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multiple Line Names</title>
    <link rel="stylesheet" href="multi-names.css">
</head>
<body>
    <div class="grid">
        <div class="item item-1">Sidebar (sidebar-start to sidebar-end)</div>
        <div class="item item-2">Main (main-start to main-end)</div>
    </div>
</body>
</html>
```

**CSS File (`multi-names.css`):**

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
    /* A single line serves as both sidebar-end and main-start */
    grid-template-columns: [sidebar-start] 150px [sidebar-end main-start] 1fr [main-end];
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 20px;
    border-radius: 6px;
    font-weight: bold;
    font-size: 0.8rem;
    text-align: center;
}

.item-1 {
    /* Referenced by its first name */
    grid-column: sidebar-start / sidebar-end;
}

.item-2 {
    /* Referenced by its second name */
    grid-column: main-start / main-end;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `multi-names.html` and CSS as `multi-names.css`.
2. Open in a browser.
3. Observe that the sidebar and main content are placed side by side, with the shared line serving both as `sidebar-end` and `main-start`.

**Expected Output:** A two-column layout with a fixed-width sidebar and a flexible main content area. The shared line between them is referenced by different names in different rules.

**Why This Works:** The line `[sidebar-end main-start]` has two names. Item 1 references it as `sidebar-end`, and Item 2 references it as `main-start`. This is a common pattern for adjacent regions in a layout, where the boundary line belongs to both. The names make the code more readable than using line numbers.

---

### Real-World Cases

- **Page layouts:** `[header-start]`, `[header-end content-start]`, `[content-end footer-start]`, `[footer-end]`.
- **Responsive grids:** Names remain constant across media queries, so placement code does not change.
- **Dashboard layouts:** `[stats-start]`, `[stats-end chart-start]`, `[chart-end]`.
- **Form layouts:** `[label-start]`, `[label-end input-start]`, `[input-end]`.

---

## 4. Named Area Registration: Assigning Grid Items to Zones Declared via `grid-area` Properties

### Definitions

**Core Definition:** Named grid areas are rectangular regions of a grid that are assigned names using the `grid-template-areas` property. Grid items are placed into these areas using the `grid-area` property.

**Technical Definition:** The `grid-template-areas` property specifies named grid areas, establishing the cells in the grid and assigning them names. The property accepts a sequence of strings, each representing a row in the grid, with each word in a string representing a cell in that row. Named areas must form a rectangular shape; if they do not, the declaration is invalid. A period (`.`) represents a null cell token, creating an empty cell. The `grid-area` property is a shorthand for `grid-row-start`, `grid-column-start`, `grid-row-end`, and `grid-column-end`. When used with a `<custom-ident>`, it places the item into the named area. Named areas automatically generate implicit named lines of the form `<area-name>-start` and `<area-name>-end`.

**Beginner-Friendly Explanation:** `grid-template-areas` lets you draw your layout directly in CSS using text. Each string is a row, and each word is a cell. You use the same word for cells that belong to the same area, and a period for empty cells. Then you assign items to those areas with `grid-area: header`, for example. This is often called "ASCII art" grid layout because the CSS looks like a picture of the final design. It is the most readable way to define a grid layout, and it automatically creates named lines that you can also use.

---

### Purposes

- To provide a visual, readable definition of the grid layout.
- To name logical regions of the grid for easy item placement.
- To create rectangular areas that items can span.
- To define empty cells using the period (`.`) syntax.
- To make layout code self-documenting and maintainable.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
.container {
    display: grid;
    grid-template-areas:
        "<row-1-cell-1> <row-1-cell-2> ..."
        "<row-2-cell-1> <row-2-cell-2> ..."
        ...;
    grid-template-columns: <track-list>;
    grid-template-rows: <track-list>;
}

.item {
    grid-area: <area-name>;
}
```

#### Component Breakdown

| Element | Description | Example |
|---|---|---|
| String | Represents a row in the grid. | `"header header header"` |
| Word | Represents a cell in the row. | `header`, `main`, `.` |
| Period (`.`) | Represents an empty cell. | `". main main"` |
| `grid-area` | Places an item into a named area. | `grid-area: header;` |

#### Syntax Rules

1. Each string in `grid-template-areas` represents a row.
2. Each word in a string represents a cell in that row.
3. All strings must have the same number of columns.
4. Named areas must form a rectangular shape.
5. A period (`.`) represents an empty cell.
6. `grid-area: <name>` places an item into the named area.
7. Named areas create implicit named lines `<name>-start` and `<name>-end`.

#### Constraints and Limitations

- **Rectangular areas only** — grid areas must be rectangular; L-shaped or irregular areas are not possible.
- **Naming conflicts** — area names must be unique within the grid.
- **Period syntax** — one or more periods can be used for empty cells.
- **Accessibility** — visual reordering via grid areas can create DOM-order vs. visual-order mismatches.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Page Layout with Named Areas

**HTML File (`areas.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Named Grid Areas</title>
    <link rel="stylesheet" href="areas.css">
</head>
<body>
    <div class="page">
        <header class="header">Header</header>
        <aside class="sidebar">Sidebar</aside>
        <main class="main">Main Content</main>
        <footer class="footer">Footer</footer>
    </div>
</body>
</html>
```

**CSS File (`areas.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.page {
    display: grid;
    /* Visual layout map using named areas */
    grid-template-areas:
        "header header header"
        "sidebar main main"
        "footer footer footer";
    grid-template-columns: 150px 1fr 1fr;
    grid-template-rows: auto 1fr auto;
    gap: 10px;
    min-height: 400px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.header {
    grid-area: header;
    background-color: #2c3e50;
    color: white;
    padding: 20px;
    border-radius: 6px;
    text-align: center;
}

.sidebar {
    grid-area: sidebar;
    background-color: #3498db;
    color: white;
    padding: 20px;
    border-radius: 6px;
}

.main {
    grid-area: main;
    background-color: #27ae60;
    color: white;
    padding: 20px;
    border-radius: 6px;
}

.footer {
    grid-area: footer;
    background-color: #2c3e50;
    color: white;
    padding: 20px;
    border-radius: 6px;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `areas.html` and CSS as `areas.css`.
2. Open in a browser.
3. Observe the layout: header spans the full width, sidebar and main are side by side, and footer spans the full width.

**Expected Output:** A page layout with a dark header across the top, a blue sidebar on the left, green main content on the right, and a dark footer across the bottom.

**Why This Works:** The `grid-template-areas` property defines a visual diagram of the layout. The `grid-area` property on each item places it into the corresponding named area. The CSS reads like a picture of the final design, making the layout structure immediately understandable.

---

#### Example 2: Empty Cells with Periods

**HTML File (`empty-areas.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid Areas with Empty Cells</title>
    <link rel="stylesheet" href="empty-areas.css">
</head>
<body>
    <div class="layout">
        <div class="hero">Hero</div>
        <div class="content">Content</div>
        <div class="aside">Aside</div>
    </div>
</body>
</html>
```

**CSS File (`empty-areas.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.layout {
    display: grid;
    /* Periods represent empty cells */
    grid-template-areas:
        "hero hero hero"
        ". content aside"
        ". content .";
    grid-template-columns: 1fr 2fr 1fr;
    grid-template-rows: auto 1fr 1fr;
    gap: 10px;
    min-height: 400px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.hero {
    grid-area: hero;
    background-color: #2c3e50;
    color: white;
    padding: 20px;
    border-radius: 6px;
    text-align: center;
}

.content {
    grid-area: content;
    background-color: #27ae60;
    color: white;
    padding: 20px;
    border-radius: 6px;
}

.aside {
    grid-area: aside;
    background-color: #3498db;
    color: white;
    padding: 20px;
    border-radius: 6px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `empty-areas.html` and CSS as `empty-areas.css`.
2. Open in a browser.
3. Observe the layout: the hero spans all three columns, the content spans two rows in the middle column, the aside is in the third column of the second row, and the third row has empty cells on the left and right.

**Expected Output:** A layout with a full-width hero at the top, a tall content block in the middle, and an aside block to its right. The bottom-left and bottom-right cells are empty.

**Why This Works:** The periods (`.`) in the `grid-template-areas` strings represent null cell tokens, creating empty cells. The named areas (`hero`, `content`, `aside`) form rectangular shapes. The content area spans two rows because it appears in two consecutive strings at the same position.

---

### Real-World Cases

- **Page layouts:** Header, sidebar, main, and footer defined with named areas.
- **Dashboard layouts:** Widgets placed into named areas like "stats", "chart", "activity".
- **Magazine layouts:** Articles and images placed into named editorial areas.
- **Responsive layouts:** Redefining `grid-template-areas` at breakpoints to reorganise content.

---

## 5. Overlapping Items: Layering Multiple Grid Items on the Same Coordinates

### Definitions

**Core Definition:** Overlapping grid items occur when two or more grid items are placed into the same grid area or share overlapping grid coordinates. The stacking order of overlapping items is controlled by the `z-index` property.

**Technical Definition:** Grid items can be placed into the same grid area by assigning them the same `grid-area` value or the same `grid-column` / `grid-row` coordinates. When multiple items occupy the same grid cell or area, they are painted in the order determined by their `z-index` values. The `z-index` property applies to grid items without requiring `position` to be set (unlike in normal flow). Grid items with a `z-index` value other than `auto` create a stacking context. The order of items in the DOM also affects painting order when `z-index` is `auto`: later items are painted on top of earlier ones.

**Beginner-Friendly Explanation:** Normally, grid items are placed into separate cells. But you can place multiple items into the same cell to make them overlap. This is useful for layering effects — like placing a caption on top of an image, or creating a design where a coloured block overlaps a photograph. To control which item appears on top, you use `z-index`. Higher numbers are on top. Unlike regular positioning, you do not need to set `position: relative` on grid items for `z-index` to work — it just works.

---

### Purposes

- To create layered visual effects without absolute positioning.
- To place captions, labels, or overlays on top of images or backgrounds.
- To design complex layouts where items intentionally overlap.
- To control stacking order of overlapping grid items with `z-index`.
- To enable creative layouts that break out of the rigid grid structure.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Place two items in the same area */
.item-1 {
    grid-area: 1 / 1 / 2 / 2; /* row-start / col-start / row-end / col-end */
    z-index: 1;
}

.item-2 {
    grid-area: 1 / 1 / 2 / 2;
    z-index: 2; /* This item appears on top */
}

/* Alternative: use named area */
.container {
    grid-template-areas: "stack";
}

.item-1 { grid-area: stack; z-index: 1; }
.item-2 { grid-area: stack; z-index: 2; }
```

#### Component Breakdown

| Property | Description | Effect |
|---|---|---|
| `grid-area` | Places item into the same area. | Overlaps items. |
| `z-index` | Controls stacking order. | Higher values on top. |
| `position` | Not required for grid items. | `z-index` works without it. |

#### Syntax Rules

1. Grid items can overlap by sharing the same `grid-area` or line coordinates.
2. `z-index` applies to grid items without requiring `position` to be set.
3. Items with `z-index: auto` are painted in DOM order (later on top).
4. A `z-index` other than `auto` creates a stacking context.
5. Negative `z-index` values place the item below the grid container's background.
6. The `z-index` property works on grid items even if they are not positioned.

#### Constraints and Limitations

- **Accessibility** — overlapping items can obscure content; ensure text remains readable.
- **Stacking context** — a `z-index` value other than `auto` creates a stacking context, which can affect child elements.
- **DOM order** — with `z-index: auto`, later items are painted on top, which may not match visual expectations.
- **Pointer events** — overlapping items may block pointer events on items below.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Image with Overlay Caption

**HTML File (`overlap.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Overlapping Grid Items</title>
    <link rel="stylesheet" href="overlap.css">
</head>
<body>
    <div class="card">
        <!-- Image and caption overlap in the same grid area -->
        <img class="card-img" src="https://via.placeholder.com/400x250" alt="Placeholder">
        <div class="card-caption">Overlay Caption</div>
    </div>
</body>
</html>
```

**CSS File (`overlap.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card {
    display: grid;
    /* Single grid area for stacking */
    grid-template-areas: "stack";
    max-width: 400px;
    border-radius: 10px;
    overflow: hidden;
}

.card-img {
    /* Place image in the stack area */
    grid-area: stack;
    width: 100%;
    height: 250px;
    object-fit: cover;
    /* z-index: auto (default) — painted first */
}

.card-caption {
    /* Place caption in the same stack area */
    grid-area: stack;
    /* Align caption to the bottom of the cell */
    align-self: end;
    background-color: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 15px;
    font-size: 1.1rem;
    font-weight: bold;
    text-align: center;
    /* Higher z-index ensures caption is on top */
    z-index: 1;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `overlap.html` and CSS as `overlap.css`.
2. Open in a browser.
3. Observe that the caption is displayed on top of the image, at the bottom of the card.

**Expected Output:** A card with an image and a dark semi-transparent caption bar overlaid at the bottom. The caption is on top of the image because it has `z-index: 1` while the image has `z-index: auto`.

**Why This Works:** Both the image and caption are placed into the `stack` grid area, so they overlap. The caption has `z-index: 1`, which places it on top of the image (which has no `z-index` set, so it defaults to `auto`). The `align-self: end` positions the caption at the bottom of the grid cell. This creates a layered effect without absolute positioning.

---

#### Example 2: Three Overlapping Items with Explicit Z-Index

**HTML File (`overlap-three.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Three Overlapping Items</title>
    <link rel="stylesheet" href="overlap-three.css">
</head>
<body>
    <div class="stack-container">
        <div class="layer layer-1">Layer 1 (z-index: 1)</div>
        <div class="layer layer-2">Layer 2 (z-index: 2)</div>
        <div class="layer layer-3">Layer 3 (z-index: 3)</div>
    </div>
</body>
</html>
```

**CSS File (`overlap-three.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.stack-container {
    display: grid;
    grid-template-areas: "stack";
    width: 300px;
    height: 200px;
    border: 2px solid #006064;
    border-radius: 8px;
    overflow: hidden;
}

.layer {
    /* All layers occupy the same grid area */
    grid-area: stack;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    color: white;
    font-size: 0.9rem;
}

.layer-1 {
    background-color: #e74c3c;
    z-index: 1;
    /* Offset to show overlap */
    transform: translate(20px, 20px);
}

.layer-2 {
    background-color: #3498db;
    z-index: 2;
    transform: translate(10px, 10px);
}

.layer-3 {
    background-color: #27ae60;
    z-index: 3;
    /* No offset — on top */
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `overlap-three.html` and CSS as `overlap-three.css`.
2. Open in a browser.
3. Observe that the green layer (z-index: 3) is on top, the blue layer (z-index: 2) is in the middle, and the red layer (z-index: 1) is at the bottom. The offsets make the layering visible.

**Expected Output:** Three coloured squares stacked on top of each other. The green square is fully visible on top. The blue square is partially visible behind it. The red square is at the bottom, partially visible behind the blue.

**Why This Works:** All three layers occupy the same `stack` grid area. The `z-index` values (1, 2, 3) control the stacking order. Layer 3 (z-index: 3) is painted last and appears on top. The `transform: translate()` offsets shift each layer slightly, making the overlap visible. This demonstrates how `z-index` controls layering within a grid area.

---

### Real-World Cases

- **Hero sections:** Text overlaid on a background image using the same grid area.
- **Card badges:** "New" or "Sale" badges layered on top of product images.
- **Image galleries:** Captions or icons overlaid on images.
- **Dashboard widgets:** Charts with overlaid labels or indicators.
- **⚠️ Accessibility caution:** Ensure overlaid text has sufficient contrast and does not obscure critical content.

---

## References

- MDN Web Docs — Grid layout using line-based placement - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_using_line-based_placement
- MDN Web Docs — Layout using named grid lines - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_using_named_grid_lines
- MDN Web Docs — Grid template areas - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_template_areas
- MDN Web Docs — `grid-column` - https://developer.mozilla.org/en-US/docs/Web/CSS/grid-column
- MDN Web Docs — `grid-row` - https://developer.mozilla.org/en-US/docs/Web/CSS/grid-row
- MDN Web Docs — `grid-area` - https://developer.mozilla.org/en-US/docs/Web/CSS/grid-area
- W3C — CSS Grid Layout Module Level 1: Placement Shorthands - https://www.w3.org/TR/css-grid-1/#placement-shorthands
- W3C — CSS Grid Layout Module Level 1: Line-based Placement - https://www.w3.org/TR/css-grid-1/#line-placement
- W3C — CSS Grid Layout Module Level 1: Named Lines - https://www.w3.org/TR/css-grid-1/#named-lines
- W3C — CSS Grid Layout Module Level 1: Grid Template Areas - https://www.w3.org/TR/css-grid-1/#grid-template-areas-property
- CSS-Tricks — `grid-column` - https://css-tricks.com/almanac/properties/g/grid-column/
- CSS-Tricks — Simple Named Grid Areas - https://css-tricks.com/simple-named-grid-areas/
- Stack Overflow — CSS Grid: Creating Overlapping Grid Items with Different Z-Indexes - https://stackoverflow.com/q/78549190
- CSS-Tricks — `z-index` - https://css-tricks.com/almanac/properties/z/z-index/