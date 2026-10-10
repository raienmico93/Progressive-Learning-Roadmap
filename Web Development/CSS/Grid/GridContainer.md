# CSS Grid Container Properties & Track Definitions — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Grid Container Properties and Track Definitions is the set of CSS properties applied to a grid container (the parent element) that establish the grid formatting context, define the grid's rows and columns (tracks), map named areas, and manage spacing between tracks. These properties form the structural blueprint of a grid layout, determining where grid items can be placed and how the available space is divided.

**Technical Definition:** The CSS Grid Layout Module Level 2 defines a set of properties that apply to the grid container itself, as opposed to properties applied to individual grid items. These container-level properties include `display` (to activate the grid formatting context), `grid-template-columns` and `grid-template-rows` (to define the explicit grid's track lists), `grid-template-areas` (to define named grid areas using a string-based visual syntax), and `gap` / `row-gap` / `column-gap` (to create gutters between tracks). The track lists accept a combination of sizing functions including absolute lengths (`px`, `rem`), percentages, the fractional unit (`fr`), intrinsic sizing keywords (`min-content`, `max-content`, `auto`), and the `minmax()` and `repeat()` functions. The `grid-template` shorthand combines `grid-template-rows`, `grid-template-columns`, and `grid-template-areas` into a single declaration.

**Beginner-Friendly Explanation:** When you turn an element into a grid container with `display: grid`, you get a powerful set of tools for defining its structure. You decide how many columns and rows it has, how wide or tall each one should be, what to name them, and how much space to put between them. You can use fixed sizes like `200px`, relative sizes like `25%`, flexible sizes like `1fr` (which means "one share of the available space"), or content-based sizes like `auto`. You can also draw a visual map of your layout using `grid-template-areas`, which makes the CSS read like a picture of the final design.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Container-level application** | These properties apply to the grid container (parent), not to individual grid items. |
| **Explicit grid definition** | `grid-template-columns` and `grid-template-rows` define the explicit grid's tracks. |
| **Flexible sizing** | The `fr` unit distributes free space proportionally among tracks. |
| **Visual area mapping** | `grid-template-areas` provides an ASCII-art-style layout definition. |
| **Gap-based spacing** | `gap`, `row-gap`, and `column-gap` create gutters without margin hacks. |
| **Repeat and minmax functions** | `repeat()` and `minmax()` reduce repetition and define flexible ranges. |
| **Implicit grid generation** | Items placed outside the explicit grid generate implicit tracks sized by `grid-auto-columns` and `grid-auto-rows`. |

---

### Prerequisites

Before studying grid container properties, you should understand:

- **CSS Normal Flow** — how block and inline boxes are laid out by default.
- **The CSS Box Model** — content, padding, border, and margin.
- **The `display` property** — `block`, `inline`, and their formatting contexts.
- **Grid fundamentals** — grid containers, grid items, grid lines, tracks, cells, and areas.

---

### Related Programming Areas

- **CSS Flexbox** — one-dimensional layout; often used together with Grid for component internals.
- **Responsive Design** — `minmax()`, `auto-fit`, and `auto-fill` enable fluid grids without media queries.
- **UI Component Design** — page shells, dashboards, and card grids rely on grid container properties.
- **Web Accessibility** — `grid-template-areas` can reorder content visually, creating DOM-order mismatches.

---

### Core Concepts / Features

1. Activating the Grid Environment: `display: grid` vs. `display: inline-grid`
2. Track Configuration Blueprints: `grid-template-columns` and `grid-template-rows`
3. Sizing Systems: Absolute Lengths, Percentages, and the `fr` Unit
4. Visual Layout Mapping: `grid-template-areas` and Empty Cell Structures
5. Spatial Distribution: `gap`, `row-gap`, and `column-gap`

---

## 1. Activating the Grid Environment: `display: grid` vs. `display: inline-grid`

### Definitions

**Core Definition:** The `display: grid` and `display: inline-grid` values activate the CSS Grid layout model on an element, turning it into a grid container and its direct children into grid items.

**Technical Definition:** Applying `display: grid` or `display: inline-grid` to an element establishes a grid formatting context (GFC) and makes the element a grid container. A value of `grid` creates a block-level grid container, while `inline-grid` creates an inline-level grid container. In both cases, the container's direct children become grid items and are laid out according to the grid layout algorithm. The difference is in the container's outer display type: `grid` makes the container behave as a block-level box (filling the available inline space and stacking vertically with other blocks), while `inline-grid` makes it behave as an inline-level box (shrink-wrapping its content and participating in an inline formatting context).

**Beginner-Friendly Explanation:** Setting `display: grid` on a parent turns it into a grid container. The difference between `grid` and `inline-grid` is how the container itself behaves in its own context: `grid` makes the container behave like a block-level element (taking up the full width and stacking vertically with other blocks), while `inline-grid` makes it behave like an inline element (sitting inline with surrounding text or inline elements). The children inside the container behave identically in both cases — they become grid items and are placed into the grid tracks.

---

### Purposes

- To create a block-level grid container that fills the available inline space.
- To create an inline-level grid container that sits inline with surrounding content.
- To establish a grid formatting context for laying out child elements in two dimensions.
- To convert direct children into grid items that participate in the grid layout algorithm.
- To provide a foundation for applying all other grid container properties.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    display: grid;        /* Block-level grid container */
    display: inline-grid; /* Inline-level grid container */
}
```

#### Component Breakdown

| Value | Outer Display Type | Inner Display Type | Behaviour |
|---|---|---|---|
| `grid` | Block-level | Grid | Container behaves like a block element; children are grid items. |
| `inline-grid` | Inline-level | Grid | Container behaves like an inline element; children are grid items. |

#### Syntax Rules

1. `display: grid` creates a block-level grid container.
2. `display: inline-grid` creates an inline-level grid container.
3. Both values create a grid formatting context.
4. Only direct children of the grid container become grid items.
5. Floats cannot intrude into the grid container.
6. Margins do not collapse across the grid formatting context boundary.

#### Constraints and Limitations

- **Block vs. inline context** — `grid` takes up the full width of its containing block; `inline-grid` shrink-wraps its content.
- **Grandchildren are not grid items** — grid layout applies only to direct children.
- **Anonymous grid items** — raw text inside a grid container becomes an unstyleable anonymous grid item.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing `display: grid` and `display: inline-grid`

**HTML File (`display-grid.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>display: grid vs. inline-grid</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="display-grid.css">
</head>
<body>
    <!-- Block-level grid container: fills width, stacks vertically -->
    <div class="grid-block">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>

    <!-- Inline text to demonstrate inline-grid behaviour -->
    <p>
        Text before
        <!-- Inline-level grid container: sits inline with text -->
        <span class="grid-inline">
            <span class="item">A</span>
            <span class="item">B</span>
            <span class="item">C</span>
        </span>
        text after.
    </p>
</body>
</html>
```

**CSS File (`display-grid.css`):**

```css
/* Block-level grid container */
.grid-block {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    margin-bottom: 20px;
}

/* Inline-level grid container */
.grid-inline {
    display: inline-grid;
    grid-template-columns: repeat(3, auto);
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
    text-align: center;
}

.grid-inline .item {
    background-color: #e65100;
    padding: 5px 10px;
    font-size: 0.8rem;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `display-grid.html`.
3. Save the CSS code as `display-grid.css` in the same folder.
4. Open `display-grid.html` in a web browser.
5. Observe that the `.grid-block` container takes up the full width of the page (block-level behaviour) and its items are arranged in three equal columns.
6. Observe that the `.grid-inline` container sits inline within the paragraph text (inline-level behaviour), shrink-wrapping its items, and the text flows around it.

**Expected Output:** A light blue block-level grid container with three dark teal boxes in a row, followed by a paragraph where a small orange inline-grid container with three small boxes sits inline with the text. The block container fills the width; the inline container is only as wide as its content.

**Why This Works:** The `display: grid` on `.grid-block` creates a block-level grid container that fills its containing block width and stacks vertically with other blocks. The `display: inline-grid` on `.grid-inline` creates an inline-level grid container that participates in the inline formatting context of the paragraph — it sits alongside the text and shrink-wraps its content. In both cases, the children become grid items and are laid out using `grid-template-columns` and `gap`. The `vertical-align: middle` helps align the inline-grid container with the surrounding text.

---

### Real-World Cases

- **Page shells:** `display: grid` on the `<body>` or a wrapper for a full-width page layout.
- **Inline widgets:** `display: inline-grid` for small components like rating stars or calendar icons that sit inline with text.
- **Card components:** `display: grid` for card internals that need a two-dimensional structure.
- **Tag groups:** `display: inline-grid` for tags that should sit inline while maintaining a grid structure.

---

## 2. Track Configuration Blueprints: `grid-template-columns` and `grid-template-rows`

### Definitions

**Core Definition:** `grid-template-columns` and `grid-template-rows` define the column and row tracks of a grid's explicit grid. Each track is assigned a sizing function that determines its width or height.

**Technical Definition:** The `grid-template-columns` CSS property defines the line names and track sizing functions of the grid columns. The `grid-template-rows` CSS property defines the line names and track sizing functions of the grid rows. Values are space-separated lists of track sizing functions, optionally interspersed with line names in square brackets. The `none` keyword indicates that there is no explicit grid; any rows or columns will be implicitly generated. The `subgrid` value (CSS Grid Level 2) indicates that the grid will adopt the spanned portion of its parent grid in that axis. The `masonry` value (CSS Grid Level 3, experimental) enables a masonry-style layout.

**Beginner-Friendly Explanation:** These two properties are how you tell the browser how many rows and columns your grid has and how big each one should be. For example, `grid-template-columns: 200px 1fr 1fr` creates three columns: the first is 200px wide, and the other two share the remaining space equally. You can also name your lines by putting names in square brackets, which makes it easier to place items later. If you do not define enough rows or columns for all your items, the browser creates implicit tracks automatically.

---

### Purposes

- To define the number and size of the grid's columns and rows.
- To provide named lines for more readable item placement.
- To establish the explicit grid that items are placed into.
- To control the overall structure of the grid layout.
- To enable responsive track definitions using `fr`, `minmax()`, and `repeat()`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
.container {
    grid-template-columns: none | <track-list> | subgrid <line-name-list>? | masonry;
    grid-template-rows: none | <track-list> | subgrid <line-name-list>? | masonry;
}

/* Track list examples */
grid-template-columns: 100px 1fr 1fr;
grid-template-columns: [full-start] 1fr [content-start] 2fr [content-end] 1fr [full-end];
grid-template-columns: repeat(3, 1fr);
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
grid-template-columns: minmax(100px, 1fr) 2fr;
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<length>` | Fixed track size. | `100px`, `5rem` |
| `<percentage>` | Relative to the container's content box. | `25%` |
| `<flex>` | Fractional unit of free space. | `1fr`, `2fr` |
| `auto` | Sized by content, then stretched. | `auto` |
| `min-content` | Smallest size without overflow. | `min-content` |
| `max-content` | Size to fit content on one line. | `max-content` |
| `minmax(min, max)` | Minimum and maximum size. | `minmax(100px, 1fr)` |
| `repeat(n, track)` | Repeats a track pattern. | `repeat(3, 1fr)` |
| `[name]` | Named line. | `[col-start]` |
| `none` | No explicit tracks. | `none` |
| `subgrid` | Adopts parent grid tracks (Level 2). | `subgrid` |

#### Syntax Rules

1. Track lists are space-separated.
2. Line names are enclosed in square brackets and can appear before or after tracks.
3. The `repeat()` function repeats a track pattern a specified number of times.
4. `repeat(auto-fill, ...)` and `repeat(auto-fit, ...)` create responsive track counts.
5. `minmax()` accepts a minimum and maximum size.
6. The `fr` unit distributes free space proportionally.
7. `subgrid` allows a nested grid to adopt its parent's tracks (CSS Grid Level 2).
8. `none` resets the explicit grid to empty.

#### Constraints and Limitations

- **`subgrid` browser support** — supported in Firefox, Safari 16+, and Chrome 117+.
- **`masonry` is experimental** — not widely supported and subject to change.
- **`fr` unit ambiguity** — `fr` distributes free space, but a track with content larger than its share may overflow.
- **Percentage tracks** — percentages refer to the container's content box in the corresponding dimension.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Track Definitions

**HTML File (`tracks.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid Track Definitions</title>
    <link rel="stylesheet" href="tracks.css">
</head>
<body>
    <h3>Fixed + Flexible: 150px 1fr 1fr</h3>
    <div class="grid grid-1">
        <div class="item">150px</div>
        <div class="item">1fr</div>
        <div class="item">1fr</div>
    </div>

    <h3>Named Lines: [side] 200px [main] 1fr [end]</h3>
    <div class="grid grid-2">
        <div class="item">Sidebar (200px)</div>
        <div class="item">Main (1fr)</div>
    </div>

    <h3>Repeat: repeat(4, 1fr)</h3>
    <div class="grid grid-3">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>

    <h3>Minmax: minmax(150px, 1fr) 2fr</h3>
    <div class="grid grid-4">
        <div class="item">minmax(150px, 1fr)</div>
        <div class="item">2fr</div>
    </div>
</body>
</html>
```

**CSS File (`tracks.css`):**

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
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.grid-1 {
    grid-template-columns: 150px 1fr 1fr;
}

.grid-2 {
    /* Named lines in square brackets */
    grid-template-columns: [side] 200px [main] 1fr [end];
}

.grid-3 {
    /* Repeat function for four equal columns */
    grid-template-columns: repeat(4, 1fr);
}

.grid-4 {
    /* Minmax: at least 150px, at most 1fr */
    grid-template-columns: minmax(150px, 1fr) 2fr;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-weight: bold;
    font-size: 0.8rem;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `tracks.html` and CSS as `tracks.css`.
2. Open in a browser.
3. Observe the four grids demonstrating different track definition techniques.
4. Notice that `grid-1` uses a fixed + flexible combination, `grid-2` uses named lines, `grid-3` uses `repeat()`, and `grid-4` uses `minmax()`.

**Expected Output:** Four grids showing different track definitions. The first has a fixed 150px column and two flexible columns. The second has a 200px sidebar and a flexible main column. The third has four equal columns. The fourth has a flexible column clamped to a minimum of 150px and a second column twice as wide.

**Why This Works:** The `grid-template-columns` property accepts a space-separated list of track sizing functions. Named lines (in square brackets) provide readable references for item placement. `repeat()` reduces repetition. `minmax()` defines a size range. The `fr` unit distributes free space proportionally among flexible tracks.

---

#### Example 2: Responsive Track Definitions with `auto-fit` and `minmax()`

**HTML File (`responsive-tracks.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Grid Tracks</title>
    <link rel="stylesheet" href="responsive-tracks.css">
</head>
<body>
    <div class="grid">
        <div class="item">Card 1</div>
        <div class="item">Card 2</div>
        <div class="item">Card 3</div>
        <div class="item">Card 4</div>
        <div class="item">Card 5</div>
        <div class="item">Card 6</div>
    </div>
</body>
</html>
```

**CSS File (`responsive-tracks.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 900px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.grid {
    display: grid;
    /* Responsive: as many columns as fit, each at least 200px */
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 30px 15px;
    border-radius: 8px;
    font-weight: bold;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `responsive-tracks.html` and CSS as `responsive-tracks.css`.
2. Open in a browser at full width. Observe four columns (900px ÷ 200px = 4.5, so 4 columns).
3. Narrow the browser window. Observe the grid automatically reduces to three columns, then two, then one.
4. There are no media queries — the responsive behaviour is driven entirely by `repeat(auto-fit, minmax(200px, 1fr))`.

**Expected Output:** A responsive grid of six cards. At wide widths, four cards appear per row. As the window narrows, the grid automatically reduces to three, two, and finally one column.

**Why This Works:** `repeat(auto-fit, minmax(200px, 1fr))` tells the browser to create as many columns as will fit, each at least 200px wide, and to distribute remaining space equally (`1fr`). When the container is too narrow to fit another 200px column, the browser reduces the column count. This is the classic "responsive grid without media queries" pattern.

---

### Real-World Cases

- **Page layouts:** `grid-template-columns: 250px 1fr` for a fixed sidebar and flexible main content.
- **Dashboard grids:** `repeat(auto-fit, minmax(300px, 1fr))` for responsive widget layouts.
- **Form layouts:** `grid-template-columns: max-content 1fr` for labels and inputs.
- **Article layouts:** `grid-template-columns: repeat(12, 1fr)` for a 12-column grid system.

---

## 3. Sizing Systems: Combining Absolute Lengths (px, rem), Percentages, and the Fractional (fr) Unit

### Definitions

**Core Definition:** Grid track sizing systems are the categories of values that can be used to define the size of grid tracks: absolute lengths (px, rem), percentages, flexible units (fr), and intrinsic sizing keywords (auto, min-content, max-content).

**Technical Definition:** The track sizing algorithm accepts several categories of sizing functions. Absolute lengths (`px`, `rem`, `em`, `cm`, `mm`, `in`, `pt`, `pc`) define fixed track sizes that do not change with the container size. Percentages define track sizes relative to the grid container's content box in the corresponding dimension. The `<flex>` unit (`fr`) represents a fraction of the free space in the grid container. Intrinsic sizing keywords (`auto`, `min-content`, `max-content`) size tracks based on their content. The `minmax()` function combines a minimum and maximum size. When mixing units, the browser resolves them in a specific order: fixed sizes are resolved first, then intrinsic sizes, then flexible sizes.

**Beginner-Friendly Explanation:** You can mix and match different sizing units in the same grid. For example, `grid-template-columns: 200px 25% 1fr` creates a fixed 200px column, a column that is 25% of the container, and a column that takes the remaining space. The `fr` unit is the most flexible — it distributes whatever space is left after fixed and percentage tracks are accounted for. You can also use `auto`, which sizes to content but can stretch to fill space. The `minmax()` function lets you set a minimum and maximum, like "at least 200px, but no more than 1fr."

---

### Purposes

- To combine fixed, relative, and flexible sizing in a single grid.
- To distribute available space proportionally using the `fr` unit.
- To size tracks based on content using `auto`, `min-content`, and `max-content`.
- To set minimum and maximum bounds on track sizes with `minmax()`.
- To create responsive layouts that adapt to different container sizes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
.container {
    grid-template-columns: <track-size> <track-size> ...;
    grid-template-rows: <track-size> <track-size> ...;
}

/* Track size types */
<track-size> = <length> | <percentage> | <flex> | min-content | max-content | auto | minmax(<min>, <max>) | fit-content(<length>)
```

#### Component Breakdown

| Unit | Type | Description | Example |
|---|---|---|---|
| `px` | Absolute length | Fixed pixels. | `200px` |
| `rem` | Absolute length | Relative to root font size. | `10rem` |
| `em` | Absolute length | Relative to element font size. | `5em` |
| `%` | Percentage | Relative to container size. | `25%` |
| `fr` | Flex | Fraction of free space. | `1fr`, `2fr` |
| `auto` | Intrinsic | Content-based, stretchable. | `auto` |
| `min-content` | Intrinsic | Smallest size without overflow. | `min-content` |
| `max-content` | Intrinsic | Size to fit content on one line. | `max-content` |
| `minmax()` | Function | Min and max bounds. | `minmax(100px, 1fr)` |
| `fit-content()` | Function | `max(min-content, min(max-content, length))`. | `fit-content(200px)` |

#### The `fr` Unit Resolution

The `fr` unit is resolved using the following formula:

```
W = (n / T) * R
```

Where:
- `W` = the track's width
- `n` = the track's `fr` value
- `T` = the total number of `fr` units in the grid
- `R` = the remaining free space after fixed and intrinsic tracks are resolved

For example, if a grid has `grid-template-columns: 200px 1fr 2fr` in a 800px container with no gaps:
- Fixed track: 200px
- Remaining space: 800 − 200 = 600px
- Total `fr` units: 1 + 2 = 3
- First `fr` track: (1/3) × 600 = 200px
- Second `fr` track: (2/3) × 600 = 400px

#### Syntax Rules

1. Fixed sizes (`px`, `rem`, `em`) are resolved first.
2. Percentages are resolved against the container's content box.
3. The `fr` unit distributes the remaining free space after fixed and intrinsic tracks.
4. `auto` tracks are sized by content but can stretch to fill free space.
5. `minmax()` accepts any valid track size for both arguments.
6. `fit-content()` clamps the track between `min-content` and `max-content`, with an optional upper bound.
7. Negative values are not allowed for track sizes.

#### Constraints and Limitations

- **`fr` unit overflow** — if a track's content is larger than its `fr` share, the track may overflow.
- **Percentage ambiguity** — percentages refer to the container's content box, which may be affected by padding and gaps.
- **`auto` vs `fr`** — `auto` sizes to content first, then stretches; `fr` distributes free space from zero.
- **`minmax()` with `auto`** — `minmax(auto, 1fr)` is a common pattern for content-aware flexible tracks.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mixing Sizing Units

**HTML File (`sizing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid Sizing Systems</title>
    <link rel="stylesheet" href="sizing.css">
</head>
<body>
    <h3>Mixing units: 150px 20% 1fr 2fr</h3>
    <div class="grid">
        <div class="item">150px</div>
        <div class="item">20%</div>
        <div class="item">1fr</div>
        <div class="item">2fr</div>
    </div>
</body>
</html>
```

**CSS File (`sizing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

h3 {
    font-size: 0.85rem;
    color: #555;
    margin-bottom: 5px;
}

.grid {
    display: grid;
    /* Mix fixed, percentage, and flexible units */
    grid-template-columns: 150px 20% 1fr 2fr;
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
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `sizing.html` and CSS as `sizing.css`.
2. Open in a browser with a wide viewport.
3. Observe the track sizes: the first column is 150px, the second is 20% of the container width, and the third and fourth share the remaining space in a 1:2 ratio.
4. Resize the browser window and observe that the 150px column stays fixed, the 20% column scales with the container, and the `fr` columns absorb the remaining space.

**Expected Output:** A grid with four columns of different sizing types. The fixed column remains 150px wide, the percentage column changes with the container, and the `fr` columns distribute the remaining space in a 1:2 ratio.

**Why This Works:** The track sizing algorithm resolves fixed sizes first (150px), then percentages (20% of the container's content width), then flexible sizes (`1fr` and `2fr`). The remaining space after fixed and percentage tracks is divided by the total `fr` units (3), giving the `1fr` track one share and the `2fr` track two shares.

---

#### Example 2: `auto` vs `fr` Track Behaviour

**HTML File (`auto-vs-fr.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>auto vs fr Tracks</title>
    <link rel="stylesheet" href="auto-vs-fr.css">
</head>
<body>
    <h3>auto tracks: size to content, then stretch</h3>
    <div class="grid auto-grid">
        <div class="item">Short</div>
        <div class="item">Much longer content here</div>
        <div class="item">Medium text</div>
    </div>

    <h3>1fr tracks: all equal, ignoring content</h3>
    <div class="grid fr-grid">
        <div class="item">Short</div>
        <div class="item">Much longer content here</div>
        <div class="item">Medium text</div>
    </div>
</body>
</html>
```

**CSS File (`auto-vs-fr.css`):**

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
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.auto-grid {
    /* auto: content-based sizing, then stretch */
    grid-template-columns: auto auto auto;
}

.fr-grid {
    /* 1fr: all equal, ignoring content */
    grid-template-columns: 1fr 1fr 1fr;
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

1. Save the HTML as `auto-vs-fr.html` and CSS as `auto-vs-fr.css`.
2. Open in a browser.
3. Observe the first grid (`auto`): the column with the longest content is wider because `auto` sizes to content.
4. Observe the second grid (`1fr`): all three columns are equal width because `1fr` distributes space equally, ignoring content.

**Expected Output:** Two grids with the same content but different sizing. In the `auto` grid, the columns are sized based on their content. In the `fr` grid, the columns are all equal width.

**Why This Works:** `auto` tracks are sized by their content's intrinsic size first, then stretched to fill any remaining space. `1fr` tracks distribute free space equally, ignoring content size. This is why `auto` is useful for content-aware layouts and `fr` is useful for equal-width layouts.

---

### Real-World Cases

- **Content-aware layouts:** `grid-template-columns: auto 1fr` for labels and values where labels size to content.
- **Equal-width dashboards:** `repeat(3, 1fr)` for three equal dashboard columns.
- **Responsive grids:** `repeat(auto-fit, minmax(250px, 1fr))` for fluid card grids.
- **Asymmetric layouts:** `1fr 2fr` for a sidebar that is half the width of the main content.

---

## 4. Visual Layout Mapping: Declarative Layout Design Using `grid-template-areas` and Empty Cell Structures

### Definitions

**Core Definition:** `grid-template-areas` is a CSS property that defines named grid areas using a string-based syntax, allowing authors to create a visual ASCII-art map of the layout. Empty cells are represented by a period (`.`).

**Technical Definition:** The `grid-template-areas` CSS property specifies named grid areas, establishing the cells in the grid and assigning them names. The property accepts a sequence of strings, each representing a row in the grid, with each word in a string representing a cell in that row. Named areas must form a rectangular shape; if they do not, the declaration is invalid and the property is ignored. A period (`.`) represents a null cell token, creating an empty cell. All the strings must have the same number of columns. The `grid-area` property on grid items references these named areas to place items.

**Beginner-Friendly Explanation:** `grid-template-areas` lets you draw your layout directly in CSS using text. Each string is a row, and each word is a cell. You use the same word for cells that belong to the same area, and a period for empty cells. This makes the CSS read like a picture of the final layout. For example:

```css
grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
```

This creates a header spanning all three columns, a sidebar and main content in the middle row, and a footer spanning all three columns. You then place items by name with `grid-area: header`, and so on.

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
7. `grid-area` can also be used with line numbers: `grid-area: 1 / 1 / 3 / 3`.

#### Constraints and Limitations

- **Rectangular areas only** — grid areas must be rectangular; L-shaped or irregular areas are not possible.
- **Naming conflicts** — area names must be unique within the grid.
- **Period syntax** — one or more periods can be used for empty cells; they do not need to match the cell name.
- **Accessibility** — visual reordering via grid areas can create DOM-order vs. visual-order mismatches.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Named Grid Areas for a Page Layout

**HTML File (`grid-areas.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Named Grid Areas</title>
    <link rel="stylesheet" href="grid-areas.css">
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

**CSS File (`grid-areas.css`):**

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

1. Save the HTML as `grid-areas.html` and CSS as `grid-areas.css`.
2. Open in a browser.
3. Observe the layout: header spans the full width, sidebar and main content are side by side, and footer spans the full width.
4. Note that the CSS `grid-template-areas` block visually depicts the layout.

**Expected Output:** A page layout with a dark header across the top, a blue sidebar on the left, green main content on the right, and a dark footer across the bottom. Each area is named in the CSS, making the layout structure immediately readable.

**Why This Works:** The `grid-template-areas` property defines a visual diagram of the layout using strings. Each string represents a row, and each word represents a named area. The `grid-area` property on each item places it into the corresponding named area. The grid container's `grid-template-columns` and `grid-template-rows` define the track sizes.

---

#### Example 2: Empty Cells with Periods

**HTML File (`empty-cells.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid Template Areas with Empty Cells</title>
    <link rel="stylesheet" href="empty-cells.css">
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

**CSS File (`empty-cells.css`):**

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
    /* The period (.) represents an empty cell */
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

1. Save the HTML as `empty-cells.html` and CSS as `empty-cells.css`.
2. Open in a browser.
3. Observe the layout: the hero spans all three columns, the content spans two rows in the middle column, the aside is in the third column of the second row, and the third row has empty cells on the left and right.

**Expected Output:** A layout with a full-width hero at the top, a tall content block in the middle, and an aside block to its right. The bottom-left and bottom-right cells are empty, as defined by the periods in the `grid-template-areas`.

**Why This Works:** The periods (`.`) in the `grid-template-areas` strings represent null cell tokens, creating empty cells. The named areas (`hero`, `content`, `aside`) form rectangular shapes, which is required for the property to be valid. The content area spans two rows because it appears in two consecutive strings at the same position.

---

### Real-World Cases

- **Page layouts:** Header, sidebar, main, and footer defined with named areas.
- **Dashboard layouts:** Widgets placed into named areas like "stats", "chart", "activity".
- **Magazine layouts:** Articles and images placed into named editorial areas.
- **Responsive layouts:** Redefining `grid-template-areas` at breakpoints to reorganise content.

---

## 5. Spatial Distribution: Managing Gutters Cleanly via `gap`, `row-gap`, and `column-gap` Properties

### Definitions

**Core Definition:** The `gap` property (and its longhands `row-gap` and `column-gap`) sets the size of gutters (spacing) between grid tracks, replacing the need for margin-based spacing calculations.

**Technical Definition:** The CSS `gap` property defines the gaps (gutters) between rows and columns. It is a shorthand for `row-gap` and `column-gap`. For grid containers, `gap` applies to both row and column gaps. The `gap` property is specified using the `row-gap` value and optionally the `column-gap` value. If `column-gap` is omitted, it uses the same value as `row-gap`. The `gap` property applies to multi-column elements, flex containers, and grid containers. Gutters effect a minimum spacing between tracks; the actual spacing may be larger if there is free space distributed by alignment properties. Importantly, gutters are not part of the track sizing calculation — the free space available for `fr` units is the container's inner size minus the sum of all gutters.

**Beginner-Friendly Explanation:** `gap` is a simple way to add space between grid tracks without using margins. Instead of adding margin to every grid item and then having to remove it from the edges, you just write `gap: 10px` on the container. This creates equal spacing between all rows and columns. `row-gap` sets the spacing between rows, and `column-gap` sets the spacing between columns. `gap: 10px 20px` sets row-gap to 10px and column-gap to 20px. The best part is that gaps do not affect the track sizing calculation — the browser subtracts the total gap space before distributing `fr` units.

---

### Purposes

- To create consistent spacing between grid tracks without margin hacks.
- To eliminate the need for `:last-child` margin resets and `calc()` calculations.
- To set separate spacing for rows and columns.
- To provide a unified spacing mechanism across grid, flexbox, and multi-column layouts.
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
| `gap` | Shorthand for `row-gap` and `column-gap`. | Grid, Flex, Multi-column. |
| `row-gap` | Spacing between rows (grid tracks). | Grid, Flex. |
| `column-gap` | Spacing between columns (grid tracks). | Grid, Flex, Multi-column. |

#### Syntax Rules

1. `gap: 10px` sets both `row-gap` and `column-gap` to 10px.
2. `gap: 10px 20px` sets `row-gap: 10px` and `column-gap: 20px`.
3. `row-gap` and `column-gap` accept `<length>` and `<percentage>` values.
4. `gap` applies to grid containers, flex containers, and multi-column elements.
5. Gutters are not part of the track sizing calculation — they reduce the free space available for `fr` units.
6. Gutters create a minimum spacing; alignment properties may add additional space.

#### Constraints and Limitations

- **Percentage gaps** — percentage values for `gap` refer to the content box size, which can be confusing.
- **Gap in the implicit grid** — gaps apply between implicit tracks as well as explicit tracks.
- **No gap on non-grid/flex** — `gap` has no effect on regular block or inline layouts.
- **Gap vs. margin** — `gap` creates space between tracks, not around the outer edges; padding can be used for outer spacing.

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
    <title>Grid Gap</title>
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
    /* Equal row and column gaps */
    gap: 15px;
}

.separate-gap {
    /* 20px row-gap, 10px column-gap */
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

#### Example 2: Gap Interaction with `fr` Units

**HTML File (`gap-fr.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gap and fr Interaction</title>
    <link rel="stylesheet" href="gap-fr.css">
</head>
<body>
    <h3>Three 1fr columns with 20px gap in a 620px container</h3>
    <div class="grid">
        <div class="item">1fr</div>
        <div class="item">1fr</div>
        <div class="item">1fr</div>
    </div>
    <p>
        Container width: 620px. Gaps: 2 × 20px = 40px. Remaining free space: 580px.
        Each 1fr track: 580 ÷ 3 ≈ 193.33px.
    </p>
</body>
</html>
```

**CSS File (`gap-fr.css`):**

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
}

.grid {
    display: grid;
    /* Three equal columns */
    grid-template-columns: repeat(3, 1fr);
    /* 20px gap between all tracks */
    gap: 20px;
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
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `gap-fr.html` and CSS as `gap-fr.css`.
2. Open in a browser.
3. Inspect the grid in DevTools and check the computed widths of the three columns — they should each be approximately 193.33px.
4. The calculation: container content width (620px) − total gap (40px) = 580px. 580 ÷ 3 = 193.33px.

**Expected Output:** Three equal-width columns with 20px gaps between them. The columns share the remaining space after the gaps are subtracted.

**Why This Works:** The `fr` unit distributes free space, not the total container size. Free space is calculated by subtracting the total gap space from the container's inner size. In this case, the container's content box is 620px, the total gap is 40px (two 20px gaps), so the remaining free space is 580px, divided equally among the three `1fr` tracks. This is a critical detail for understanding grid sizing.

---

### Real-World Cases

- **Card grids:** `gap: 20px` for consistent spacing between cards.
- **Page layouts:** `gap: 0 20px` for column gaps without row gaps.
- **Dashboard grids:** `gap: 16px 24px` for different row and column spacing.
- **Responsive layouts:** Changing `gap` at breakpoints with a single declaration.

---

## References

- MDN Web Docs — `display` - https://developer.mozilla.org/en-US/docs/Web/CSS/display
- MDN Web Docs — `grid-template-columns` - https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns
- MDN Web Docs — `grid-template-rows` - https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-rows
- MDN Web Docs — `grid-template-areas` - https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-areas
- MDN Web Docs — `gap` - https://developer.mozilla.org/en-US/docs/Web/CSS/gap
- MDN Web Docs — Basic concepts of grid layout - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout/Basic_concepts
- MDN Web Docs — Grid template areas - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout/Grid_template_areas
- W3C — CSS Grid Layout Module Level 2 - https://www.w3.org/TR/css-grid-2/
- W3C — CSS Grid Layout Module Level 1 - https://www.w3.org/TR/css-grid-1/
- CSS-Tricks — A Complete Guide to CSS Grid - https://css-tricks.com/snippets/css/complete-guide-grid/
- Can I Use — CSS Grid Layout - https://caniuse.com/css-grid
- Can I Use — `gap` in Grid - https://caniuse.com/mdn-css_properties_gap_grid_context