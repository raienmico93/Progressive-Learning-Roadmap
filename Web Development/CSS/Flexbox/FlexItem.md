# Flex Item Properties & Dimensional Calculation Math — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** Flex Item Properties and Dimensional Calculation Math is the set of CSS properties applied to individual flex items (the children of a flex container) that control how each item grows, shrinks, sizes itself initially, aligns on the cross axis, and orders itself visually. These properties — `flex-grow`, `flex-shrink`, `flex-basis`, the `flex` shorthand, `align-self`, and `order` — determine how available or negative space is distributed among items through a precise mathematical algorithm defined by the CSS specification.

**Technical Definition:** The CSS Flexible Box Layout Module Level 1 defines a set of properties that apply to flex items, as opposed to flex container properties. These item-level properties include `flex-grow` (the flex grow factor, a unitless number specifying how much of the container's positive free space is assigned to the item), `flex-shrink` (the flex shrink factor, a unitless number specifying how much the item shrinks relative to other items when negative free space is distributed), `flex-basis` (the initial main size of the item before free space is distributed), the `flex` shorthand (which sets all three), `align-self` (which overrides the container's `align-items` for a single item on the cross axis), and `order` (which controls the visual ordering of items independent of their document order). The dimensional calculation algorithm resolves flex base sizes, determines free space, distributes positive or negative space according to flex factors, and applies minimum/maximum constraints, including the automatic minimum size of flex items (`min-width: auto` / `min-height: auto`).

**Beginner-Friendly Explanation:** Once you have a flex container with items inside, you can control each item individually. Do you want one item to take up twice as much space as the others? Use `flex-grow`. Do you want an item to refuse to shrink when space gets tight? Use `flex-shrink`. Do you want an item to start at a certain size before the browser distributes extra space? Use `flex-basis`. Do you want one item to align differently from the rest? Use `align-self`. Do you want to change the visual order without touching the HTML? Use `order`. These item-level properties give you fine-grained control over each item's behaviour, while the container properties control the overall arrangement.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Item-level application** | These properties apply to flex items (children), not to the flex container (parent). |
| **Three-way sizing model** | `flex-grow`, `flex-shrink`, and `flex-basis` work together to determine an item's final size. |
| **Shorthand presets** | The `flex` shorthand provides convenient keyword presets for common sizing patterns. |
| **Individual alignment override** | `align-self` overrides the container's `align-items` for a single item. |
| **Visual reordering** | `order` changes visual order without changing DOM order, with accessibility implications. |
| **Content-aware minimum** | Flex items have an automatic minimum size (`min-width: auto`) that prevents shrinking below content size unless overridden. |
| **Precise mathematical resolution** | The final size of each item is determined by a multi-step algorithm involving flex base sizes, free space, flex factors, and min/max constraints. |

---

### Prerequisites

Before studying flex item properties, you should understand:

- **CSS Flexbox fundamentals** — flex containers, flex items, main axis, cross axis.
- **Flex container properties** — `display: flex`, `flex-direction`, `flex-wrap`, `justify-content`, `align-items`.
- **The CSS Box Model** — content, padding, border, and margin.
- **Basic arithmetic** — the calculation of proportions and ratios.

---

### Related Programming Areas

- **CSS Grid Layout** — Grid items share some properties with flex items (`order`, `align-self`).
- **Responsive Design** — flex item properties enable fluid layouts that adapt to available space.
- **UI Component Design** — buttons, cards, and form controls rely on flex item sizing.
- **Web Accessibility** — `order` can create DOM-order vs. visual-order mismatches.

---

### Core Concepts / Features

1. Sizing Engines: The 3-Way Flexibility Model (`flex-grow`, `flex-shrink`, `flex-basis`)
2. Shorthand Combination Engineering: `flex` Keyword Presets
3. Overriding Parent Alignment: `align-self`
4. Reordering Visual Source Order: The `order` Property
5. Sizing Interactions: Content-Based Constraints vs. Static Dimensions

---

## 1. Sizing Engines: The 3-Way Flexibility Model (`flex-grow`, `flex-shrink`, and `flex-basis`)

### Definitions

**Core Definition:** The 3-way flexibility model is the mechanism by which `flex-grow`, `flex-shrink`, and `flex-basis` together determine a flex item's initial size and its growth or shrinkage when the container has positive or negative free space.

**Technical Definition:** 
- `flex-grow` sets the flex grow factor, a unitless number that specifies how much of the flex container's positive free space is assigned to the item. It is proportional to the sum of all flex grow factors of items in the same line. 
- `flex-shrink` sets the flex shrink factor, a unitless number that specifies how much the item shrinks relative to other items when negative free space is distributed. The negative space is distributed proportionally to the product of the item's flex shrink factor and its flex base size. 
- `flex-basis` sets the initial main size of the flex item before free space is distributed. It accepts the same values as `width` and `height` (lengths, percentages) plus the `content` keyword. When `flex-basis` is `auto`, it retrieves the value of the main size property (`width` or `height`); if that is also `auto`, the used value is `content`, which sizes the item based on its content.

**Beginner-Friendly Explanation:** Imagine you and your friends are sharing a pizza. 
- `flex-basis` is how many slices you start with. 
- `flex-grow` is how many extra slices you get if there are leftovers — higher numbers mean more extra slices. 
- `flex-shrink` is how many slices you give up if there are too few slices to go around — higher numbers mean you give up more. 
- The browser does the math for you: it first gives everyone their `flex-basis` slices, then distributes leftover slices according to `flex-grow`, or takes back slices according to `flex-shrink`.

---

### Purposes

- To specify the initial main size of a flex item before free space distribution.
- To control how much an item grows relative to its siblings when positive free space exists.
- To control how much an item shrinks relative to its siblings when negative free space exists.
- To enable proportional sizing of items without explicit pixel widths.
- To provide the foundational values that the `flex` shorthand expands into.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    flex-grow: <number>;
    flex-shrink: <number>;
    flex-basis: <length> | <percentage> | auto | content;
}
```

#### Component Breakdown

| Property | Initial Value | Description | Accepts |
|---|---|---|---|
| `flex-grow` | `0` | Flex grow factor. Negative values are invalid. | Unitless number. |
| `flex-shrink` | `1` | Flex shrink factor. Negative values are invalid. | Unitless number. |
| `flex-basis` | `auto` | Initial main size. | Length, percentage, `auto`, `content`. |

#### Syntax Rules

1. `flex-grow` and `flex-shrink` accept only unitless numbers; negative values are invalid.
2. `flex-basis` accepts the same values as `width` and `height`, plus `content`.
3. When `flex-basis` is `auto`, it resolves to the item's `width` (in row direction) or `height` (in column direction).
4. If `width`/`height` is also `auto`, `flex-basis: auto` resolves to `content` (the item's intrinsic size).
5. `flex-basis` percentages resolve against the flex container's main size.
6. The initial value of `flex-grow` is `0` (items do not grow by default).
7. The initial value of `flex-shrink` is `1` (items can shrink by default).
8. The initial value of `flex-basis` is `auto`.

#### The Mathematical Algorithm

The flex sizing algorithm proceeds in the following steps:

1. **Determine flex base size:** For each item, the flex base size is determined by `flex-basis`. If `flex-basis` is `auto`, the main size property (`width`/`height`) is used; if that is `auto`, the content size is used.
2. **Determine hypothetical main size:** The flex base size is clamped by `min-width`/`min-height` and `max-width`/`max-height`.
3. **Calculate free space:** The container's inner main size minus the sum of all items' hypothetical main sizes (and margins, if any).
4. **Distribute positive free space (if free space > 0):** Each item receives a share of the free space proportional to its `flex-grow` value divided by the sum of all `flex-grow` values.
5. **Distribute negative free space (if free space < 0):** Each item's shrinkage is proportional to the product of its `flex-shrink` factor and its flex base size, divided by the sum of these products across all items.
6. **Apply min/max constraints:** If any item hits a minimum or maximum constraint, the distribution is repeated with the constrained item frozen.

#### Constraints and Limitations

- **Automatic minimum size:** Flex items have an automatic minimum size of `min-width: auto` (in row direction), which prevents them from shrinking below their content size unless overridden with `min-width: 0` or `overflow` other than `visible`.
- **Content-based minimum:** The `min-content` size acts as a lower bound for shrinking.
- **Percentage basis ambiguity:** Percentage values for `flex-basis` resolve against the container's main size, which may itself be indefinite in some cases.
- **Browser differences:** Older browsers may handle `flex-basis: content` differently.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Flex Grow Distribution

**HTML File (`flex-grow.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flex Grow Distribution</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="flex-grow.css">
</head>
<body>
    <!-- Flex container with items of different flex-grow values -->
    <div class="container">
        <div class="item grow-1">flex-grow: 1</div>
        <div class="item grow-2">flex-grow: 2</div>
        <div class="item grow-1">flex-grow: 1</div>
    </div>
</body>
</html>
```

**CSS File (`flex-grow.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    /* Container width: 600px - 30px padding = 570px content */
}

.item {
    /* flex-basis: 0 so all space is distributed by flex-grow */
    flex-basis: 0;
    background-color: #006064;
    color: white;
    padding: 20px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
}

.grow-1 { flex-grow: 1; }
.grow-2 { flex-grow: 2; }
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `flex-grow.html`.
3. Save the CSS code as `flex-grow.css` in the same folder.
4. Open `flex-grow.html` in a web browser.
5. Observe the widths of the three items. The container's content width is approximately 570px (600px minus 30px padding). With `gap: 10px`, the available space for distribution is 570 − 20 = 550px. The sum of `flex-grow` values is 1 + 2 + 1 = 4. Item 1 gets 1/4 × 550 = 137.5px, Item 2 gets 2/4 × 550 = 275px, Item 3 gets 1/4 × 550 = 137.5px.

**Expected Output:** Three items with widths proportional to their `flex-grow` values. The middle item (flex-grow: 2) is twice as wide as the other two (flex-grow: 1).

**Why This Works:** With `flex-basis: 0`, all available space is distributed according to `flex-grow`. The free space (550px) is divided by the sum of grow factors (4), giving 137.5px per unit. Item 2 receives 2 units (275px), and Items 1 and 3 receive 1 unit each (137.5px).

---

#### Example 2: Flex Shrink Distribution

**HTML File (`flex-shrink.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flex Shrink Distribution</title>
    <link rel="stylesheet" href="flex-shrink.css">
</head>
<body>
    <div class="container">
        <div class="item shrink-1">flex-shrink: 1</div>
        <div class="item shrink-2">flex-shrink: 2</div>
        <div class="item shrink-1">flex-shrink: 1</div>
    </div>
</body>
</html>
```

**CSS File (`flex-shrink.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    width: 400px; /* Narrow container to force shrinking */
}

.item {
    /* Each item wants 200px, but the container is too narrow */
    flex-basis: 200px;
    background-color: #006064;
    color: white;
    padding: 20px;
    border-radius: 6px;
    font-weight: bold;
    text-align: center;
    font-size: 0.8rem;
}

.shrink-1 { flex-shrink: 1; }
.shrink-2 { flex-shrink: 2; }
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `flex-shrink.html` and CSS as `flex-shrink.css`.
2. Open in a browser.
3. Observe that the three items each want 200px (flex-basis), for a total of 600px plus 20px gap = 620px, but the container is only 400px wide. The negative free space is 620 − 400 = 220px.
4. The shrink distribution uses the product of `flex-shrink` and `flex-basis`: Item 1: 1 × 200 = 200; Item 2: 2 × 200 = 400; Item 3: 1 × 200 = 200. Sum = 800. Item 1 shrinks by (200/800) × 220 = 55px → 145px. Item 2 shrinks by (400/800) × 220 = 110px → 90px. Item 3 shrinks by (200/800) × 220 = 55px → 145px.

**Expected Output:** Three items with different widths. The middle item (flex-shrink: 2) is narrower than the other two, demonstrating that it shrinks more aggressively.

**Why This Works:** When negative free space is distributed, the shrink amount is proportional to the product of `flex-shrink` and `flex-basis` (not just `flex-shrink` alone). This ensures that larger items shrink more than smaller ones, even if they have the same `flex-shrink` value.

---

### Real-World Cases

- **Navigation bars:** `flex-grow: 1` on nav items to distribute them evenly.
- **Sidebar layouts:** `flex-basis: 250px; flex-shrink: 0` for a fixed-width sidebar that does not shrink.
- **Card grids:** `flex: 1 1 300px` for cards that grow to fill space but have a 300px starting point.
- **Form layouts:** `flex-grow: 1` on an input field to fill remaining space next to a fixed-width button.

---

## 2. Shorthand Combination Engineering: The `flex` Property Keyword Presets

### Definitions

**Core Definition:** The `flex` shorthand property sets `flex-grow`, `flex-shrink`, and `flex-basis` in a single declaration, with keyword presets (`initial`, `auto`, `none`) that expand to specific combinations of these three values.

**Technical Definition:** The `flex` CSS shorthand property sets how a flex item will grow or shrink to fit the space available in its flex container. It is a shorthand for `flex-grow`, `flex-shrink`, and `flex-basis`. The one-value syntax accepts a valid `flex-grow` value (expanding to `<flex-grow> 1 0%`), a valid `flex-basis` value (expanding to `1 1 <flex-basis>`), or the keyword `none`. The two-value syntax accepts `flex-grow` and either `flex-shrink` (expanding to `<flex-grow> <flex-shrink> 0%`) or `flex-basis` (expanding to `<flex-grow> 1 <flex-basis>`). The three-value syntax is `<flex-grow> <flex-shrink> <flex-basis>`. The keyword presets are: `initial` (equivalent to `0 1 auto`), `auto` (equivalent to `1 1 auto`), and `none` (equivalent to `0 0 auto`).

**Beginner-Friendly Explanation:** The `flex` shorthand lets you set all three sizing properties in one line. Instead of writing `flex-grow: 1; flex-shrink: 1; flex-basis: 0;`, you can just write `flex: 1`. The keyword presets are shortcuts for common patterns: `flex: none` means "don't grow, don't shrink, use my width"; `flex: auto` means "grow and shrink based on my content size"; `flex: initial` means "don't grow, but can shrink, based on my width." The most common value you will see in real code is `flex: 1`, which means "grow to fill available space, starting from zero."

---

### Purposes

- To provide a concise syntax for setting all three flex sizing properties at once.
- To offer keyword presets for the most common sizing patterns.
- To reduce the risk of forgetting to set one of the three properties.
- To improve code readability by using semantic keywords.
- To ensure consistent sizing behaviour across a set of flex items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    /* One-value syntax */
    flex: <flex-grow>;
    flex: <flex-basis>;
    flex: none;

    /* Two-value syntax */
    flex: <flex-grow> <flex-shrink>;
    flex: <flex-grow> <flex-basis>;

    /* Three-value syntax */
    flex: <flex-grow> <flex-shrink> <flex-basis>;
}
```

#### Component Breakdown

| Shorthand | Expansion | Meaning | Common Use Case |
|---|---|---|---|
| `flex: initial` | `0 1 auto` | Don't grow, can shrink, size by width. | Default behaviour. |
| `flex: auto` | `1 1 auto` | Grow and shrink, size by content. | Items that absorb free space proportionally to their content size. |
| `flex: none` | `0 0 auto` | Don't grow, don't shrink, use width. | Fixed-size items. |
| `flex: 1` | `1 1 0%` | Grow and shrink, start from zero. | Equal-width columns. |
| `flex: 2` | `2 1 0%` | Grow twice as much as `flex: 1`. | Asymmetric columns. |
| `flex: 1 1 auto` | `1 1 auto` | Explicit version of `flex: auto`. | — |

#### Syntax Rules

1. A single unitless number is interpreted as `flex-grow`, with `flex-shrink: 1` and `flex-basis: 0%`.
2. A single length or percentage is interpreted as `flex-basis`, with `flex-grow: 1` and `flex-shrink: 1`.
3. Two values: if the second is a number, it is `flex-shrink` (basis becomes `0%`); if it is a length/percentage, it is `flex-basis` (shrink becomes `1`).
4. Three values: `flex-grow`, `flex-shrink`, `flex-basis` in that order.
5. The keyword `none` expands to `0 0 auto`.
6. The keyword `auto` expands to `1 1 auto`.
7. The keyword `initial` (or no value) expands to `0 1 auto`.

#### Constraints and Limitations

- **`flex: 1` vs `flex: auto`:** `flex: 1` sets `flex-basis: 0%`, so all items with `flex: 1` become equal width regardless of content. `flex: auto` sets `flex-basis: auto`, so items grow based on their content size.
- **Overriding individual properties:** If you set `flex: 1` and then `flex-basis: 200px`, the `flex-basis` declaration overrides the shorthand's basis.
- **Browser defaults:** The initial value of `flex` is `0 1 auto` (the same as `flex: initial`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing `flex: 1`, `flex: auto`, `flex: none`, and `flex: initial`

**HTML File (`flex-presets.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flex Shorthand Presets</title>
    <link rel="stylesheet" href="flex-presets.css">
</head>
<body>
    <h3>flex: 1 — all items equal width</h3>
    <div class="container">
        <div class="item flex-1">Short</div>
        <div class="item flex-1">Much longer content here</div>
        <div class="item flex-1">Medium text</div>
    </div>

    <h3>flex: auto — items grow based on content</h3>
    <div class="container">
        <div class="item flex-auto">Short</div>
        <div class="item flex-auto">Much longer content here</div>
        <div class="item flex-auto">Medium text</div>
    </div>

    <h3>flex: none — items use their natural width</h3>
    <div class="container">
        <div class="item flex-none">Short</div>
        <div class="item flex-none">Much longer content here</div>
        <div class="item flex-none">Medium text</div>
    </div>
</body>
</html>
```

**CSS File (`flex-presets.css`):**

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
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.8rem;
    text-align: center;
}

.flex-1 { flex: 1; }
.flex-auto { flex: auto; }
.flex-none { flex: none; }
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `flex-presets.html` and CSS as `flex-presets.css`.
2. Open in a browser.
3. Observe the first container: all three items are equal width because `flex: 1` sets `flex-basis: 0%`, so content size is ignored during distribution.
4. Observe the second container: the item with more text is wider because `flex: auto` uses `flex-basis: auto`, which considers content size before growing.
5. Observe the third container: each item is only as wide as its content because `flex: none` prevents growth and shrinkage.

**Expected Output:** Three containers showing dramatically different sizing behaviour. In the first, all items are equal width. In the second, the longest item is widest. In the third, items are sized to their content.

**Why This Works:** `flex: 1` sets `flex-basis: 0%`, so all items start from zero and grow equally. `flex: auto` sets `flex-basis: auto`, so items start from their content size and then grow. `flex: none` sets `flex-grow: 0` and `flex-shrink: 0`, so items stay at their content width.

---

### Real-World Cases

- **Equal-width columns:** `flex: 1` for a row of equally sized columns.
- **Content-sized items:** `flex: auto` for items that should be sized proportionally to their content.
- **Fixed buttons:** `flex: none` for buttons that should not stretch or shrink.
- **Sidebar + main content:** `flex: none` on the sidebar and `flex: 1` on the main content.

---

## 3. Overriding Parent Alignment: Individual Cross-Axis Alignment Modifications via `align-self`

### Definitions

**Core Definition:** The `align-self` property allows an individual flex item to override the cross-axis alignment set by the container's `align-items` property.

**Technical Definition:** The CSS `align-self` property overrides a grid or flex item's `align-items` value. In flexbox, it aligns the item on the cross axis. The property accepts the same values as `align-items` (`auto`, `flex-start`, `flex-end`, `center`, `baseline`, `stretch`) plus the `auto` keyword, which resets the item to the value of the parent's `align-items` property. The initial value is `auto`. The property applies to flex items, grid items, and absolutely positioned boxes. It is not inherited.

**Beginner-Friendly Explanation:** The flex container's `align-items` property sets the default cross-axis alignment for all items. But sometimes you want one item to align differently. That is what `align-self` is for. For example, if your container has `align-items: center` but you want one item to stick to the top, you set `align-self: flex-start` on that item. The `auto` value means "inherit from the parent's `align-items`."

---

### Purposes

- To override the container's cross-axis alignment for a single flex item.
- To create visual variety in a flex row without affecting other items.
- To align a specific item to the baseline for typographic alignment.
- To stretch a single item to fill the container's cross size.
- To provide item-level control over cross-axis positioning.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

#### Component Breakdown

| Value | Description | Behaviour |
|---|---|---|
| `auto` | Inherits from the parent's `align-items`. | Default. |
| `flex-start` | Aligns to the cross-start edge. | Top (for row) or left (for column). |
| `flex-end` | Aligns to the cross-end edge. | Bottom (for row) or right (for column). |
| `center` | Centres on the cross axis. | Vertically centred (for row). |
| `baseline` | Aligns by text baseline. | Baselines line up across items. |
| `stretch` | Stretches to fill the cross size. | Items expand to container's cross size. |

#### Syntax Rules

1. `align-self` applies to individual flex items (and grid items).
2. The initial value is `auto`, which resolves to the parent's `align-items` value.
3. It accepts the same values as `align-items` plus `auto`.
4. It is not inherited.
5. If the item's cross-axis size is `auto`, `align-self` may be ignored (e.g., `stretch` requires `auto` cross size).

#### Constraints and Limitations

- **Only affects the cross axis** — `align-self` has no effect on main-axis alignment (use `justify-content` on the container or `margin` on the item).
- **`stretch` requires `auto` cross size** — if the item has an explicit cross size, `stretch` has no effect.
- **`baseline` requires text** — items without text may not align as expected.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Individual Alignment Override

**HTML File (`align-self.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>align-self Override</title>
    <link rel="stylesheet" href="align-self.css">
</head>
<body>
    <div class="container">
        <div class="item">Default (align-items: center)</div>
        <div class="item align-top">align-self: flex-start</div>
        <div class="item align-bottom">align-self: flex-end</div>
        <div class="item">Default again</div>
    </div>
</body>
</html>
```

**CSS File (`align-self.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    /* Container default: centre items on the cross axis */
    align-items: center;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
    min-height: 150px;
}

.item {
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.8rem;
    font-weight: bold;
    flex: 1;
    text-align: center;
}

.align-top {
    /* Override: align this item to the top */
    align-self: flex-start;
}

.align-bottom {
    /* Override: align this item to the bottom */
    align-self: flex-end;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `align-self.html` and CSS as `align-self.css`.
2. Open in a browser.
3. Observe that the first and fourth items are vertically centred (following `align-items: center`), the second item is aligned to the top, and the third item is aligned to the bottom.

**Expected Output:** A flex container with four items. The second item is at the top, the third is at the bottom, and the other two are centred.

**Why This Works:** The container's `align-items: center` sets the default alignment for all items. The `.align-top` and `.align-bottom` classes use `align-self` to override this default for those specific items, demonstrating individual control over cross-axis alignment.

---

### Real-World Cases

- **Card layouts:** `align-self: flex-start` on a card with a "featured" badge that should stick to the top.
- **Form rows:** `align-self: baseline` on a label to align it with an input's text baseline.
- **Navigation bars:** `align-self: center` on a logo while other items are `flex-start`.
- **Dashboard widgets:** `align-self: stretch` on a widget that should fill the available height.

---

## 4. Reordering Visual Source Order Independent of DOM Rendering Nodes Using the `order` Property

### Definitions

**Core Definition:** The `order` property controls the visual ordering of flex items (and grid items) within their container, independent of their document source order.

**Technical Definition:** The `order` CSS property sets the order to lay out an item in a flex or grid container. Items in a container are sorted by ascending `order` value and then by their source code order. Items not given an explicit `order` value are assigned the default value of `0`. The property accepts an `<integer>` value (positive, negative, or zero). It is defined in the CSS Display module and impacts only grid and flex items. When `order` is set on an element whose parent's `display` property is not creating a flex or grid container, the property has no effect. Using `order` creates a disconnect between visual presentation and DOM order, which can adversely affect users navigating with assistive technology.

**Beginner-Friendly Explanation:** `order` lets you change the visual position of a flex item without moving it in the HTML. By default, all items have `order: 0` and are displayed in the order they appear in the HTML. If you give one item `order: -1`, it moves to the front. If you give another `order: 1`, it moves to the end. This is useful for responsive layouts — for example, moving a sidebar below the main content on mobile without changing the HTML. However, be careful: screen readers and keyboard navigation follow the DOM order, not the visual order, so reordering can create a confusing experience for some users.

---

### Purposes

- To change the visual order of flex items without modifying the HTML structure.
- To enable responsive layouts where items reorder at different breakpoints.
- To pull specific items to the front or push them to the back of a layout.
- To create visual hierarchies that differ from the source order.
- To provide a CSS-only solution for reordering content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    order: <integer>;
}
```

#### Component Breakdown

| Value | Description | Behaviour |
|---|---|---|
| `<integer>` | Positive, negative, or zero. | Lower values appear first. |
| `0` | Default for all items. | Items with the same `order` follow source order. |

#### Syntax Rules

1. The initial value is `0`.
2. Items are sorted by ascending `order` value.
3. Items with the same `order` value follow their source code order.
4. `order` applies only to flex and grid items.
5. Negative values are allowed (e.g., `order: -1` moves an item to the front).
6. `order` does not affect the DOM order or tab order — only the visual order.

#### Constraints and Limitations

- **Accessibility risk:** Using `order` creates a disconnect between visual presentation and DOM order. This adversely affects low vision users navigating with assistive technology such as a screen reader.
- **Tab order mismatch:** Keyboard navigation follows DOM order, not visual order, so reordered items may receive focus in an unexpected sequence.
- **Screen reader confusion:** Screen readers read content in DOM order, which may not match the visual layout.
- **No effect on non-flex/grid:** `order` has no effect on elements whose parent is not a flex or grid container.
- **Not for semantic reordering:** Authors must not use `order` as a substitute for correct source ordering, as that can ruin the accessibility of the document.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reordering Flex Items

**HTML File (`order.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Order Property</title>
    <link rel="stylesheet" href="order.css">
</head>
<body>
    <div class="container">
        <!-- DOM order: 1, 2, 3, 4 -->
        <div class="item order-0">Order 0 (default)</div>
        <div class="item order-1">Order 1</div>
        <div class="item order-minus">Order -1</div>
        <div class="item order-2">Order 2</div>
    </div>
    <p>
        DOM order is: Order 0, Order 1, Order -1, Order 2.
        Visual order is: Order -1, Order 0, Order 1, Order 2.
    </p>
</body>
</html>
```

**CSS File (`order.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 10px;
    background-color: #e0f7fa;
    border: 2px solid #006064;
    border-radius: 8px;
    padding: 15px;
}

.item {
    padding: 15px;
    border-radius: 6px;
    color: white;
    font-weight: bold;
    font-size: 0.8rem;
    text-align: center;
    flex: 1;
}

.order-0 {
    background-color: #3498db;
    /* order: 0 is the default */
}

.order-1 {
    background-color: #27ae60;
    order: 1;
}

.order-minus {
    background-color: #e74c3c;
    order: -1; /* Moves to the front */
}

.order-2 {
    background-color: #9b59b6;
    order: 2;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `order.html` and CSS as `order.css`.
2. Open in a browser.
3. Observe the visual order: the red box (order: -1) appears first, then the blue box (order: 0), then the green box (order: 1), and finally the purple box (order: 2).
4. Note that the DOM order (as shown in the HTML) is different from the visual order.

**Expected Output:** Four coloured boxes displayed in visual order: red, blue, green, purple. The DOM order is blue, green, red, purple — demonstrating the disconnect.

**Why This Works:** The `order` property sorts items by ascending integer value. The red box with `order: -1` comes before all items with `order: 0`, which come before `order: 1` and `order: 2`. The visual order is determined by `order`, not by the source order in the HTML. This creates the accessibility concern that the visual order diverges from the DOM order.

---

### Real-World Cases

- **Responsive layouts:** Moving a sidebar below the main content on mobile using `order`, while keeping the sidebar first in the DOM for desktop.
- **Dashboard widgets:** Reordering widgets based on user preference without changing the HTML.
- **E-commerce product listings:** Moving "featured" products to the front visually while keeping the DOM order logical.
- **⚠️ Accessibility caution:** Screen readers follow DOM order, so visual reordering can cause confusion. Use `order` sparingly and ensure the DOM order still makes logical sense.

---

## 5. Sizing Interactions: Content-Based Constraints vs. Static Dimensions (`width`/`height`) on Flex Calculations

### Definitions

**Core Definition:** Sizing interactions describe how `flex-basis`, `width`/`height`, and content-based constraints (such as `min-content` and `max-content`) interact to determine a flex item's final size.

**Technical Definition:** The `flex-basis` property sets the initial main size of a flex item. When `flex-basis` is `auto`, it retrieves the value of the main size property (`width` in row direction, `height` in column direction); if that value is itself `auto`, the used value is `content`, which sizes the item based on its content. A key difference between `flex-basis` and `width` is that `flex-basis` is bound below by `min-content`: if you specify a `flex-basis` smaller than the item's `min-content` size, the item will still be at least `min-content` sized. In contrast, `width` can size a box arbitrarily small, even below its content size. Additionally, flex items have an automatic minimum size (`min-width: auto` / `min-height: auto`) that prevents them from shrinking below their content size unless overridden with `min-width: 0` or `overflow` other than `visible`.

**Beginner-Friendly Explanation:** When you set `flex-basis: 200px` on an item, you are saying "start at 200px." But if the item's content is wider than 200px (for example, a long unbreakable word), the item cannot actually be smaller than that word — it will grow to fit the content. This is called the "min-content" constraint. On the other hand, `width: 200px` can be overridden by content in some cases, but it is not bound by min-content in the same way. Also, flex items have an automatic minimum size that prevents them from shrinking below their content. This is why you sometimes see `min-width: 0` in flexbox code — it disables that automatic minimum, allowing items to shrink below their content size.

---

### Purposes

- To understand how `flex-basis` and `width`/`height` interact to determine item size.
- To explain why flex items refuse to shrink below their content size by default.
- To provide the `min-width: 0` workaround for allowing items to shrink further.
- To clarify the difference between `flex-basis: auto` and `flex-basis: content`.
- To inform authors about the automatic minimum size of flex items.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    flex-basis: auto | content | <length> | <percentage>;
    width: <length> | <percentage> | auto;
    min-width: 0 | auto | <length>;
    max-width: none | <length> | <percentage>;
}
```

#### Component Breakdown

| Property | Value | Behaviour |
|---|---|---|
| `flex-basis: auto` | (default) | Uses `width` (or `height` in column direction); if `width` is `auto`, uses content size. |
| `flex-basis: content` | — | Sizes based on content, ignoring `width`/`height`. |
| `flex-basis: <length>` | 200px | Initial main size is 200px, but bound below by `min-content`. |
| `min-width: auto` | (default for flex items) | Prevents shrinking below content size. |
| `min-width: 0` | — | Allows shrinking below content size. |
| `width: <length>` | 200px | Main size is 200px, but may be overridden by `flex-basis`. |

#### Syntax Rules

1. If `flex-basis` is set to a value other than `auto` or `content`, it takes precedence over `width` (in row direction) or `height` (in column direction).
2. `flex-basis: auto` resolves to the value of the main size property (`width`/`height`).
3. If the main size property is also `auto`, `flex-basis: auto` resolves to `content` (the item's intrinsic size).
4. `flex-basis` is bound below by the item's `min-content` size; it cannot shrink below this unless `min-width`/`min-height` is explicitly set to a smaller value.
5. Flex items have `min-width: auto` (row direction) or `min-height: auto` (column direction) as their initial value, which prevents shrinking below content size.
6. Setting `min-width: 0` or `min-height: 0` (or using `overflow` other than `visible`) allows items to shrink below their content size.

#### Constraints and Limitations

- **Content-based minimum:** The `min-content` size acts as a hard lower bound for `flex-basis` unless overridden.
- **`min-width: auto` is the default:** This is a common source of confusion — flex items do not shrink below their content size by default.
- **`flex-basis: auto` vs `content`:** `auto` looks at `width` first, then content; `content` always looks at content.
- **Browser differences:** Older browsers may not support `flex-basis: content`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: The `min-width: auto` Problem and Solution

**HTML File (`min-width.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flex min-width: auto</title>
    <link rel="stylesheet" href="min-width.css">
</head>
<body>
    <h3>Without min-width: 0 — item refuses to shrink</h3>
    <div class="container">
        <div class="item no-override">
            This item has a long unbreakable word:
            Supercalifragilisticexpialidocious
        </div>
        <div class="item">Normal item</div>
    </div>

    <h3>With min-width: 0 — item shrinks and text wraps</h3>
    <div class="container">
        <div class="item with-override">
            This item has a long unbreakable word:
            Supercalifragilisticexpialidocious
        </div>
        <div class="item">Normal item</div>
    </div>
</body>
</html>
```

**CSS File (`min-width.css`):**

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
    width: 400px;
}

.item {
    flex: 1;
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.8rem;
    word-break: break-all; /* Allow breaking long words */
}

.no-override {
    /* Default: min-width: auto prevents shrinking below content */
}

.with-override {
    /* Override: allow shrinking below content */
    min-width: 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `min-width.html` and CSS as `min-width.css`.
2. Open in a browser.
3. Observe the first container: the item with the long word refuses to shrink because `min-width: auto` prevents it from going below its content size. The container overflows.
4. Observe the second container: the item shrinks and the long word breaks across lines because `min-width: 0` allows it to shrink below its content size.

**Expected Output:** In the first container, the long-word item takes up more than half the container width, causing overflow. In the second container, both items are equal width and the long word wraps.

**Why This Works:** By default, flex items have `min-width: auto`, which means they cannot be smaller than their content's minimum size (the `min-content` size). The long unbreakable word sets a high `min-content` size, preventing shrinking. Setting `min-width: 0` overrides this default, allowing the item to shrink below its content size, and `word-break: break-all` allows the word to break across lines.

---

#### Example 2: `flex-basis` vs `width` with Content Constraints

**HTML File (`basis-vs-width.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>flex-basis vs width</title>
    <link rel="stylesheet" href="basis-vs-width.css">
</head>
<body>
    <h3>flex-basis: 100px with long content</h3>
    <div class="container">
        <div class="item basis-small">
            This content is much wider than 100px
        </div>
        <div class="item">Other item</div>
    </div>

    <h3>width: 100px with long content</h3>
    <div class="container">
        <div class="item width-small">
            This content is much wider than 100px
        </div>
        <div class="item">Other item</div>
    </div>
</body>
</html>
```

**CSS File (`basis-vs-width.css`):**

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
    width: 500px;
}

.item {
    flex: 1;
    background-color: #006064;
    color: white;
    padding: 15px;
    border-radius: 6px;
    font-size: 0.8rem;
}

.basis-small {
    /* flex-basis: 100px, but bound below by min-content */
    flex-basis: 100px;
}

.width-small {
    /* width: 100px, but flex-basis: auto resolves to width */
    width: 100px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `basis-vs-width.html` and CSS as `basis-vs-width.css`.
2. Open in a browser.
3. Observe that both containers look similar — the item with long content is wider than 100px in both cases because the content forces it to be larger.
4. The key difference is subtle: `flex-basis` is explicitly bound by `min-content`, while `width` can be overridden by `flex-basis` in some cases.

**Expected Output:** Both containers show the long-content item wider than 100px due to the content's `min-content` size. The difference between `flex-basis` and `width` is not visually apparent in this example because both are constrained by content.

**Why This Works:** Both `flex-basis: 100px` and `width: 100px` are lower bounds that can be exceeded by content. The `min-content` size of the long text forces the item to be wider than 100px. The key difference is that `flex-basis` is explicitly defined to be bound below by `min-content`, while `width` can, in some edge cases, be overridden by `flex-basis` when both are set.

---

### Real-World Cases

- **Truncated text in flex items:** Using `min-width: 0` and `text-overflow: ellipsis` to truncate long text in a flex item.
- **Responsive sidebars:** Using `flex-basis: auto` with a `width` to define the starting size, then letting `flex-grow` fill remaining space.
- **Equal-height cards:** Using `min-width: 0` to allow cards to shrink evenly without content forcing uneven widths.
- **Form inputs:** Using `min-width: 0` on flex inputs to prevent long placeholder text from breaking the layout.

---

## References

- MDN Web Docs — `flex` - https://developer.mozilla.org/en-US/docs/Web/CSS/flex
- MDN Web Docs — `flex-grow` - https://developer.mozilla.org/en-US/docs/Web/CSS/flex-grow
- MDN Web Docs — `flex-shrink` - https://developer.mozilla.org/en-US/docs/Web/CSS/flex-shrink
- MDN Web Docs — `flex-basis` - https://developer.mozilla.org/en-US/docs/Web/CSS/flex-basis
- MDN Web Docs — `align-self` - https://developer.mozilla.org/en-US/docs/Web/CSS/align-self
- MDN Web Docs — `order` - https://developer.mozilla.org/en-US/docs/Web/CSS/order
- W3C — CSS Flexible Box Layout Module Level 1 - https://www.w3.org/TR/css-flexbox-1/
- W3C — CSS Flexible Box Layout Module Level 1 (Editor's Draft) - https://drafts.csswg.org/css-flexbox-1/
- CSS-Tricks — A Complete Guide to CSS Flexbox - https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- MDN Web Docs — Controlling ratios of flex items along the main axis - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Controlling_flex_item_ratios
- web.dev — Reordering content - https://web.dev/articles/flexbox-order
- Tink — Flexbox and the keyboard navigation disconnect - https://tink.uk/flexbox-the-keyboard-navigation-disconnect/
- Adrian Roselli — Source Order Matters - https://adrianroselli.com/2015/09/source-order-matters.html