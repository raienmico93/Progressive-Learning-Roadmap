# CSS Positioning Mechanics & Layout Shifts — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Positioning Mechanics is the system of CSS properties and algorithms that control how elements are placed within a document, either as part of the normal flow or removed from it. The `position` property is the central mechanism, with five values: `static`, `relative`, `absolute`, `fixed`, and `sticky`. Each value activates a different positioning scheme, governing how an element's position is calculated and how it interacts with surrounding content.

**Technical Definition:** The `position` property determines which positioning scheme is used to calculate the position of a box. Values other than `static` make the box a "positioned box" and cause it to establish an absolute positioning containing block for its descendants. The CSS Positioned Layout Module Level 3 defines four coordinate-based positioning schemes: relative positioning (visual offset without layout impact), sticky positioning (scroll-aware offset within a scroll container), absolute positioning (removal from flow and positioning relative to a containing block), and fixed positioning (absolute positioning relative to the viewport or page area). The `inset` properties (`top`, `right`, `bottom`, `left`) determine the final location of positioned elements. The CSS Anchor Positioning API extends this system by allowing positioned elements to size and position themselves relative to one or more "anchor elements" elsewhere on the page.

**Beginner-Friendly Explanation:** Normally, elements on a webpage follow a natural flow: block elements stack vertically, inline elements flow horizontally. But sometimes you need to pull an element out of that flow — to stick it to the top of the screen when scrolling, to layer a tooltip above a button, or to position a dropdown menu. CSS positioning gives you five tools for this. `static` is the default (everything follows normal flow). `relative` nudges an element from its normal position without disturbing anything else. `absolute` takes an element completely out of the flow and lets you place it anywhere relative to a positioned ancestor. `fixed` pins an element to the viewport so it never scrolls away. `sticky` is a hybrid: it scrolls normally until it hits a threshold, then sticks. The new CSS Anchor Positioning API makes it even easier to position things like tooltips and dropdowns relative to other elements.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Five positioning schemes** | `static`, `relative`, `absolute`, `fixed`, and `sticky`, each with distinct behaviour. |
| **In-flow vs. out-of-flow** | `static` and `relative` remain in normal flow; `absolute` and `fixed` remove elements from flow entirely. |
| **Positioned element** | An element whose computed `position` value is `relative`, `absolute`, `fixed`, or `sticky`. |
| **Containing block dependency** | The positioning reference for `absolute` and `fixed` elements depends on ancestor properties (e.g., `transform`, `filter`, `perspective`). |
| **Stacking context creation** | `relative` (with `z-index` not `auto`), `absolute`, `fixed`, and `sticky` create new stacking contexts. |
| **Layout shift implications** | Removing elements from flow (`absolute`, `fixed`) can cause layout shifts if space is not reserved. |
| **Anchor positioning extension** | The CSS Anchor Positioning API builds on absolute positioning to enable native tooltip and popover placement. |

---

### Prerequisites

Before studying CSS Positioning Mechanics, you should understand:

- **CSS Normal Flow** — how block and inline boxes are laid out by default.
- **The CSS Box Model** — content, padding, border, and margin.
- **The `display` property** — `block`, `inline`, `inline-block`, and their formatting contexts.
- **Containing block concept** — the rectangular box with respect to which an element's dimensions and position are calculated.
- **Stacking contexts and `z-index`** — how elements are layered in the visual order.

---

### Related Programming Areas

- **CSS Layout** — Flexbox and Grid, which override normal flow with their own formatting contexts.
- **Web Accessibility** — ensuring that visually reordered content maintains a logical reading order.
- **Web Performance** — Cumulative Layout Shift (CLS) and the impact of positioned elements on visual stability.
- **UI Component Design** — tooltips, dropdowns, modals, and sticky headers rely heavily on positioning.
- **Responsive Design** — media queries frequently change `position` values for different viewports.

---

### Core Concepts / Features

1. `static`: The Default In-Flow Layout State
2. `relative`: Local Box Offsets Without Layout Impact
3. `absolute`: Out-of-Flow Separation and Containing Block Resolution
4. `fixed`: Viewport-Anchored Positioning and Container Escape Vulnerabilities
5. `sticky`: Hybrid Scroll-Aware Positioning and Parent Height Pitfalls
6. Modern Layout Positioning: CSS Anchor Positioning API

---

## 1. `static`: The Default In-Flow Layout State

### Definitions

**Core Definition:** `position: static` is the default positioning scheme for all elements. In this state, elements are laid out according to the normal flow rules of their parent formatting context, and the `top`, `right`, `bottom`, `left`, and `z-index` properties do not apply.

**Technical Definition:** When `position` computes to `static`, the box is not a positioned box and is laid out according to the rules of its parent formatting context. The inset properties (`top`, `right`, `bottom`, `left`) do not apply. The `z-index` property also does not apply. Static elements participate fully in normal flow and are the baseline against which the effects of other positioning schemes are measured.

**Beginner-Friendly Explanation:** `static` is the "do nothing special" setting. Every element starts as static. It just follows the normal rules: block elements stack vertically, inline elements flow horizontally. You cannot move a static element with `top`, `left`, or any other offset property. If you want to position something, you must first change its `position` value away from `static`.

---

### Purposes

- To provide the default, predictable layout behaviour for all elements.
- To serve as the baseline for understanding the effects of other positioning schemes.
- To allow elements to participate fully in normal flow without special positioning.
- To maintain document order and accessibility by keeping elements in the natural reading sequence.
- To avoid the unintended side effects (overlap, removal from flow) caused by non-static positioning.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    position: static;
}
```

#### Component Breakdown

| Property | Value | Behaviour |
|---|---|---|
| `position` | `static` | Default. Element is laid out in normal flow. |
| `top` / `right` / `bottom` / `left` | (ignored) | Do not apply to static elements. |
| `z-index` | (ignored) | Does not apply to static elements. |

#### Syntax Rules

1. `static` is the initial value of the `position` property.
2. Inset properties and `z-index` have no effect on static elements.
3. Static elements are laid out according to the rules of their parent formatting context (block or inline).
4. Static positioning is the only scheme that does not create a stacking context or establish a containing block for positioned descendants.

#### Constraints and Limitations

- **No offset control** — you cannot move a static element with `top`, `left`, etc.
- **No `z-index` control** — layering is not possible without changing position.
- **No containing block establishment** — static elements do not serve as containing blocks for absolutely positioned descendants.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Static Positioning in Normal Flow

**HTML File (`static.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Static Positioning</title>
    <link rel="stylesheet" href="static.css">
</head>
<body>
    <!-- Static div: follows normal flow, cannot be offset -->
    <div class="static-box">
        This div has <code>position: static</code> (the default).
        It follows normal flow and cannot be moved with top, left, etc.
    </div>
    <p>
        This paragraph follows naturally after the static div. The static
        element's inset properties are ignored.
    </p>
</body>
</html>
```

**CSS File (`static.css`):**

```css
.static-box {
    /* Explicitly set to static (though it is the default) */
    position: static;
    /* These properties are IGNORED because position is static */
    top: 50px;
    left: 100px;
    /* Visible styling */
    background-color: #e8f5e9;
    border: 2px solid #2e7d32;
    padding: 15px;
    margin-bottom: 20px;
}

body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `static.html` and CSS as `static.css`.
2. Open in a browser.
3. Observe that the `.static-box` div appears in its normal flow position.
4. The `top: 50px` and `left: 100px` have no visible effect.

**Expected Output:** A green-bordered div that sits in the normal flow position. The `top` and `left` values are ignored.

**Why This Works:** Because `position` is `static`, the inset properties (`top`, `left`) do not apply. The element is laid out according to normal flow rules, and its position cannot be offset without changing the `position` value.

---

### Real-World Cases

- **Default behaviour:** Most elements on a page use `position: static` without the author needing to declare it.
- **Debugging:** When an element is not responding to `top` or `left`, the first thing to check is whether `position` has been set to something other than `static`.
- **Baseline layout:** Static positioning is the foundation upon which all other positioning schemes build.

---

## 2. `relative`: Local Box Offsets Without Layout Impact

### Definitions

**Core Definition:** `position: relative` lays out an element according to normal flow, then visually offsets it from its resulting position using the inset properties, without affecting the size or position of any other element.

**Technical Definition:** A relatively positioned box is laid out as for `static`, then offset from the resulting position. This offsetting is a purely visual effect and does not affect the size or position of any other box, except insofar as it increases the scrollable overflow area of its ancestors. The `z-index` property applies to relatively positioned elements and, when not `auto`, creates a new stacking context. Relative positioning does not remove the element from flow; the space it would have occupied remains reserved.

**Beginner-Friendly Explanation:** `relative` is like nudging an element a few pixels from where it would normally be. The element still takes up its original space in the layout — nothing else moves. The offset is purely visual. This is useful for small adjustments, like moving an icon slightly to align with text, or creating a positioning context for absolutely positioned children.

---

### Purposes

- To visually offset an element from its normal position without disturbing surrounding layout.
- To create a positioning context (containing block) for absolutely positioned descendants.
- To enable `z-index` layering on an element while keeping it in normal flow.
- To make small alignment adjustments without affecting the document flow.
- To serve as a subtle animation or hover effect mechanism.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    position: relative;
    top: <length> | <percentage> | auto;
    right: <length> | <percentage> | auto;
    bottom: <length> | <percentage> | auto;
    left: <length> | <percentage> | auto;
    z-index: <integer> | auto;
}
```

#### Component Breakdown

| Property | Description | Effect |
|---|---|---|
| `top` | Moves the element down from its normal top edge. | Positive values move down; negative values move up. |
| `bottom` | Moves the element up from its normal bottom edge. | Positive values move up; negative values move down. |
| `left` | Moves the element right from its normal left edge. | Positive values move right; negative values move left. |
| `right` | Moves the element left from its normal right edge. | Positive values move left; negative values move right. |
| `z-index` | Controls stacking order. | Creates a stacking context when not `auto`. |

#### Syntax Rules

1. The element remains in normal flow; its original space is preserved.
2. Offsets are relative to the element's own normal position.
3. If both `top` and `bottom` are specified, `top` wins for relatively positioned elements.
4. If both `left` and `right` are specified, `left` wins in left-to-right writing modes.
5. `z-index` applies and creates a stacking context when not `auto`.

#### Constraints and Limitations

- **No layout impact** — surrounding elements do not move; the element only shifts visually.
- **Overflow can increase** — the offset can increase the scrollable overflow area of ancestors.
- **Not removed from flow** — the element still occupies its original space, which can leave gaps.
- **Percentage offsets** — percentages for `top` and `bottom` refer to the containing block's height; for `left` and `right`, the containing block's width.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Relative Offset and Stacking Context

**HTML File (`relative.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Relative Positioning</title>
    <link rel="stylesheet" href="relative.css">
</head>
<body>
    <div class="container">
        <!-- Box 1: normal flow -->
        <div class="box box-1">Box 1 (static)</div>
        <!-- Box 2: relatively positioned, offset -->
        <div class="box box-2">Box 2 (relative, offset 20px down and 40px right)</div>
        <!-- Box 3: normal flow -->
        <div class="box box-3">Box 3 (static)</div>
    </div>
</body>
</html>
```

**CSS File (`relative.css`):**

```css
.container {
    padding: 20px;
    background-color: #f5f5f5;
}

.box {
    padding: 15px;
    margin-bottom: 10px;
    border-radius: 6px;
    color: white;
    font-weight: bold;
}

.box-1 {
    background-color: #3498db;
}

.box-2 {
    background-color: #e74c3c;
    /* Relatively positioned: offset from its normal position */
    position: relative;
    top: 20px;   /* Move down 20px from normal top edge */
    left: 40px;  /* Move right 40px from normal left edge */
    /* z-index creates a stacking context */
    z-index: 1;
}

.box-3 {
    background-color: #27ae60;
    /* This box is NOT moved, but it will be covered by Box 2's offset */
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `relative.html` and CSS as `relative.css`.
2. Open in a browser.
3. Observe that Box 1 and Box 3 remain in their normal positions.
4. Observe that Box 2 is visually shifted down and right, overlapping Box 3. Its original space is still reserved in the layout.

**Expected Output:** Three coloured boxes. Box 1 (blue) is at the top. Box 2 (red) is shifted down and right, overlapping Box 3. Box 3 (green) is in its normal position, partially covered by Box 2.

**Why This Works:** The `top: 20px` and `left: 40px` on `.box-2` offset it visually from its normal position. The space it would have occupied remains reserved, so Box 3 still appears where it would have been. The `z-index: 1` ensures Box 2 renders above Box 3.

---

### Real-World Cases

- **Icon alignment:** Nudging an icon a few pixels to align with adjacent text.
- **Dropdown menus:** Using `position: relative` on a parent to create a containing block for an absolutely positioned dropdown.
- **Hover effects:** Slightly shifting a card or button on hover for a tactile feel.
- **Tooltips:** A relatively positioned trigger element serves as the containing block for an absolutely positioned tooltip.

---

## 3. `absolute`: Out-of-Flow Separation and Containing Block Resolution

### Definitions

**Core Definition:** `position: absolute` removes an element from normal flow and positions it relative to its nearest positioned ancestor (or the initial containing block if none exists).

**Technical Definition:** An absolutely positioned box is taken out of flow such that it has no impact on the size or position of its siblings and ancestors, and does not participate in its parent's formatting context. The box is positioned and sized solely in reference to its absolute positioning containing block, as modified by the box's inset properties. The containing block is formed by the padding edge of the nearest ancestor whose `position` is `relative`, `absolute`, `fixed`, or `sticky`. If no positioned ancestor exists, the initial containing block (the viewport) is used. The `z-index` property applies and creates a stacking context when not `auto`.

**Beginner-Friendly Explanation:** `absolute` takes an element completely out of the layout. It no longer affects or is affected by surrounding elements — it sits on its own layer. You then position it using `top`, `left`, etc., relative to the nearest ancestor that has a position other than `static`. If no such ancestor exists, it positions relative to the page itself. This is how dropdowns, modals, and tooltips are positioned.

---

### Purposes

- To remove an element from normal flow so it does not affect surrounding layout.
- To position an element precisely relative to a specific ancestor.
- To create layered UI components like dropdowns, tooltips, and modals.
- To overlay content on top of other content using `z-index`.
- To enable precise placement without disturbing the document flow.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    position: absolute;
    top: <length> | <percentage> | auto;
    right: <length> | <percentage> | auto;
    bottom: <length> | <percentage> | auto;
    left: <length> | <percentage> | auto;
    z-index: <integer> | auto;
}
```

#### Component Breakdown

| Concept | Description |
|---|---|
| **Containing block** | The padding edge of the nearest positioned ancestor, or the initial containing block (viewport) if none exists. |
| **Removal from flow** | The element no longer participates in the parent's formatting context. No space is reserved for it. |
| **Positioned ancestor** | An ancestor with `position: relative`, `absolute`, `fixed`, or `sticky`. |
| **Initial containing block** | The viewport-sized rectangle used when no positioned ancestor exists. |

#### Syntax Rules

1. The element is removed from normal flow and does not affect siblings or ancestors.
2. The containing block is the nearest positioned ancestor's padding edge, or the initial containing block.
3. If both `top` and `bottom` are specified, and `height` is `auto`, the element is stretched.
4. If both `left` and `right` are specified, and `width` is `auto`, the element is stretched.
5. Margins do not collapse for absolutely positioned elements.
6. `z-index` applies and creates a stacking context when not `auto`.

#### Constraints and Limitations

- **No space reserved** — surrounding content does not account for the absolutely positioned element.
- **Containing block dependency** — if no positioned ancestor exists, the element positions relative to the viewport.
- **Layout shift risk** — removing an element from flow can cause CLS if space is not reserved elsewhere.
- **Accessibility** — absolutely positioned elements can be visually reordered relative to their DOM position.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Absolute Positioning Relative to a Positioned Ancestor

**HTML File (`absolute.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Absolute Positioning</title>
    <link rel="stylesheet" href="absolute.css">
</head>
<body>
    <!-- Positioned ancestor: establishes the containing block -->
    <div class="card">
        <h2>Card Title</h2>
        <p>This card has a badge positioned absolutely in its top-right corner.</p>
        <!-- Absolutely positioned badge -->
        <span class="badge">New</span>
    </div>

    <p>
        The badge is removed from normal flow and positioned relative to the
        card. The card itself is <code>position: relative</code>, making it
        the containing block for the badge.
    </p>
</body>
</html>
```

**CSS File (`absolute.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}

.card {
    /* Establish the containing block for absolutely positioned children */
    position: relative;
    background-color: #fff;
    border: 2px solid #ddd;
    border-radius: 8px;
    padding: 20px;
    margin-bottom: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.badge {
    /* Remove from flow and position relative to the card */
    position: absolute;
    top: 10px;
    right: 10px;
    background-color: #e74c3c;
    color: white;
    padding: 4px 10px;
    border-radius: 12px;
    font-size: 0.8rem;
    font-weight: bold;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `absolute.html` and CSS as `absolute.css`.
2. Open in a browser.
3. Observe that the badge appears in the top-right corner of the card, regardless of the card's content.
4. Remove `position: relative` from `.card` and reload — the badge will position relative to the viewport instead.

**Expected Output:** A card with a red "New" badge positioned in its top-right corner. The badge does not affect the layout of the card's title or paragraph.

**Why This Works:** The `position: relative` on `.card` establishes it as the containing block for `.badge`. The `position: absolute` on `.badge` removes it from flow and positions it relative to the card's padding edge using `top: 10px` and `right: 10px`.

---

### Real-World Cases

- **Dropdown menus:** A navigation item is `position: relative`; the dropdown is `position: absolute` and positioned below it.
- **Modal dialogs:** A full-screen overlay is `position: absolute` or `fixed`; the dialog content is centered within it.
- **Image captions:** A caption is absolutely positioned over an image without affecting the image's layout.
- **Tooltips:** A tooltip is absolutely positioned relative to its trigger element.

---

## 4. `fixed`: Viewport-Anchored Positioning and Container Escape Vulnerabilities

### Definitions

**Core Definition:** `position: fixed` removes an element from normal flow and positions it relative to the viewport (or the page area in paged media), so that it remains in the same position even when the document is scrolled.

**Technical Definition:** Fixed positioning is the same as absolute positioning, except the box is positioned and sized relative to a fixed positioning containing block — usually the viewport in continuous media, or the page area in paged media. The box's position is fixed with respect to this reference rectangle: when attached to the viewport, it does not move when the document is scrolled. However, if an ancestor has a `transform`, `perspective`, or `filter` value other than `none`, that ancestor becomes the containing block for fixed-positioned descendants. This is known as the "container escape vulnerability" — fixed elements can be trapped inside a transformed ancestor, breaking their viewport anchoring.

**Beginner-Friendly Explanation:** `fixed` is like pinning an element to your screen. No matter how much you scroll, the element stays in the same place — like a sticky navigation bar or a "back to top" button. The catch is that if any ancestor has a CSS transform, filter, or perspective, the fixed element will be positioned relative to that ancestor instead of the viewport. This is a common source of bugs.

---

### Purposes

- To keep an element visible at all times regardless of scroll position.
- To create persistent UI elements like navigation bars, headers, and footers.
- To position floating action buttons and "back to top" controls.
- To overlay content that must remain in a fixed screen position.
- To anchor elements to the viewport for full-screen overlays and modals.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    position: fixed;
    top: <length> | <percentage> | auto;
    right: <length> | <percentage> | auto;
    bottom: <length> | <percentage> | auto;
    left: <length> | <percentage> | auto;
    z-index: <integer> | auto;
}
```

#### Component Breakdown

| Concept | Description |
|---|---|
| **Fixed positioning containing block** | The viewport (continuous media) or page area (paged media). |
| **Container escape** | If an ancestor has `transform`, `perspective`, `filter`, or `will-change: transform`, that ancestor becomes the containing block. |
| **Removal from flow** | The element no longer participates in normal flow. |
| **Stacking context** | `fixed` always creates a new stacking context. |

#### Syntax Rules

1. The element is removed from normal flow and positioned relative to the viewport.
2. The containing block is the viewport unless an ancestor has `transform`, `perspective`, or `filter` other than `none`.
3. `fixed` always creates a new stacking context.
4. The element does not move when the page is scrolled.
5. In paged media, the element is replicated on every page.

#### Constraints and Limitations

- **Container escape vulnerability** — transformed ancestors break viewport anchoring.
- **CORS and security** — fixed-position overlays have been exploited for phishing attacks when CSS injection is possible.
- **Layout shift risk** — removing an element from flow can cause CLS if space is not reserved.
- **Accessibility** — fixed elements can obscure content and interfere with screen readers if not managed carefully.
- **Print media** — fixed elements may appear on every printed page, which is often undesirable.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fixed Header and Container Escape Demonstration

**HTML File (`fixed.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fixed Positioning</title>
    <link rel="stylesheet" href="fixed.css">
</head>
<body>
    <!-- Fixed header: stays at the top of the viewport -->
    <header class="fixed-header">
        Fixed Header — stays at the top when scrolling
    </header>

    <main class="content">
        <p>Scroll down to see the fixed header stay in place.</p>
        <!-- Tall content to enable scrolling -->
        <div class="spacer"></div>

        <!-- Transformed ancestor: creates container escape -->
        <div class="transformed-parent">
            <div class="fixed-child">
                Fixed child inside a transformed parent — it escapes to the parent, not the viewport!
            </div>
        </div>

        <div class="spacer"></div>
    </main>
</body>
</html>
```

**CSS File (`fixed.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
    line-height: 1.6;
}

.fixed-header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    background-color: #2c3e50;
    color: white;
    padding: 15px;
    text-align: center;
    font-weight: bold;
    z-index: 100;
    box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}

.content {
    padding: 80px 20px 20px;
    max-width: 700px;
    margin: 0 auto;
}

.spacer {
    height: 300px;
    background: linear-gradient(#f5f5f5, #ddd);
    margin: 20px 0;
    border-radius: 8px;
}

.transformed-parent {
    /* This transform makes the parent the containing block for fixed children */
    transform: translateX(0);
    background-color: #fff3e0;
    border: 2px dashed #e65100;
    padding: 20px;
    margin: 20px 0;
    position: relative;
}

.fixed-child {
    /* Fixed positioning is trapped inside the transformed parent */
    position: fixed;
    bottom: 20px;
    right: 20px;
    background-color: #e74c3c;
    color: white;
    padding: 10px 15px;
    border-radius: 6px;
    font-size: 0.85rem;
    max-width: 300px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `fixed.html` and CSS as `fixed.css`.
2. Open in a browser.
3. Scroll down — the `.fixed-header` stays at the top of the viewport.
4. Observe the `.fixed-child`: because `.transformed-parent` has `transform: translateX(0)`, the fixed child is positioned relative to that parent, not the viewport. It appears at the bottom-right of the parent, not the screen.

**Expected Output:** A dark header fixed at the top of the screen that never moves when scrolling. The red fixed child appears at the bottom-right of the transformed parent, demonstrating the container escape vulnerability.

**Why This Works:** The `transform: translateX(0)` on `.transformed-parent` creates a containing block for fixed-positioned descendants. The `.fixed-child` is therefore positioned relative to the parent's padding edge, not the viewport. This is the container escape vulnerability — a fixed element loses its viewport anchoring when an ancestor has a transform, filter, or perspective.

---

### Real-World Cases

- **Sticky navigation bars:** A header that remains visible at the top of the screen while scrolling.
- **Floating action buttons:** A "chat" or "back to top" button fixed to the bottom-right corner.
- **Cookie consent banners:** A fixed banner at the bottom of the viewport.
- **Modal overlays:** A full-screen fixed overlay with centered dialog content.

---

## 5. `sticky`: Hybrid Scroll-Aware Positioning and Parent Height Pitfalls

### Definitions

**Core Definition:** `position: sticky` is a hybrid positioning scheme that behaves like `relative` until a specified scroll threshold is reached, at which point it behaves like `fixed` within its containing block. It remains in normal flow and does not affect the position of other elements.

**Technical Definition:** Sticky positioning is identical to relative positioning, except that its offsets are automatically adjusted in reference to the nearest ancestor scroll container's scrollport (as modified by the inset properties) in whichever axes the inset properties are not both `auto`, to try to keep the box in view within its containing block as the user scrolls. The element sticks to the nearest scrolling ancestor, which is the nearest ancestor with `overflow` set to `hidden`, `scroll`, `auto`, or `overlay`. A sticky element only sticks within its parent's bounds: if the parent is not taller than the sticky element, there is no room for it to stick.

**Beginner-Friendly Explanation:** `sticky` is a combination of `relative` and `fixed`. An element scrolls normally until it reaches a threshold you set (like `top: 0`), then it sticks in place while the rest of the page scrolls. It only sticks within its parent container — once the parent scrolls out of view, the sticky element goes with it. The most common pitfall is that sticky does not work if any ancestor has `overflow: hidden`, `scroll`, or `auto`, or if the parent is not taller than the sticky element.

---

### Purposes

- To create scroll-aware headers, sidebars, and table headers.
- To keep important information visible while scrolling through long content.
- To provide a native CSS alternative to JavaScript scroll listeners.
- To maintain context (e.g., a section title) while scrolling within a section.
- To enable sticky table headers that remain visible as the table body scrolls.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    position: sticky;
    top: <length> | <percentage> | auto;
    right: <length> | <percentage> | auto;
    bottom: <length> | <percentage> | auto;
    left: <length> | <percentage> | auto;
    z-index: <integer> | auto;
}
```

#### Component Breakdown

| Concept | Description |
|---|---|
| **Scroll container** | The nearest ancestor with `overflow` other than `visible` (e.g., `hidden`, `scroll`, `auto`, `overlay`). |
| **Sticky constraint** | The element sticks within its containing block (parent). |
| **Inset threshold** | At least one inset property (e.g., `top: 0`) must be set for sticky to work. |
| **Parent height requirement** | The parent must be taller than the sticky element for sticking to occur. |
| **Overflow restriction** | An ancestor with `overflow: hidden`, `scroll`, or `auto` breaks sticky. |

#### Syntax Rules

1. At least one inset property (`top`, `right`, `bottom`, `left`) must be set to a non-`auto` value for sticky to take effect.
2. The element sticks within its containing block (parent); it does not escape the parent.
3. An ancestor with `overflow: hidden`, `scroll`, or `auto` breaks sticky positioning.
4. The parent must be taller than the sticky element for sticking to occur.
5. `sticky` always creates a new stacking context.
6. Multiple sticky elements in the same container are offset independently and may overlap.

#### Constraints and Limitations

- **Parent height pitfall** — if the parent is not taller than the sticky element, sticky does not work.
- **Overflow ancestor pitfall** — `overflow: hidden`, `scroll`, or `auto` on any ancestor breaks sticky. Use `overflow: clip` instead.
- **No inset set** — without `top`, `bottom`, `left`, or `right`, sticky behaves like relative.
- **Browser inconsistencies** — some browsers handle sticky differently in table elements and flex containers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Sticky Table Header and Parent Height Demonstration

**HTML File (`sticky.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sticky Positioning</title>
    <link rel="stylesheet" href="sticky.css">
</head>
<body>
    <!-- Scrollable container with sticky table header -->
    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th class="sticky-header">Name</th>
                    <th class="sticky-header">Role</th>
                    <th class="sticky-header">Location</th>
                </tr>
            </thead>
            <tbody>
                <tr><td>Alice</td><td>Engineer</td><td>New York</td></tr>
                <tr><td>Bob</td><td>Designer</td><td>London</td></tr>
                <tr><td>Carol</td><td>Manager</td><td>Tokyo</td></tr>
                <tr><td>Dave</td><td>Analyst</td><td>Berlin</td></tr>
                <tr><td>Eve</td><td>Developer</td><td>Sydney</td></tr>
                <tr><td>Frank</td><td>Architect</td><td>Toronto</td></tr>
                <tr><td>Grace</td><td>Writer</td><td>Paris</td></tr>
                <tr><td>Henry</td><td>Editor</td><td>Chicago</td></tr>
                <tr><td>Ivy</td><td>Researcher</td><td>Seoul</td></tr>
                <tr><td>Jack</td><td>Consultant</td><td>Madrid</td></tr>
            </tbody>
        </table>
    </div>

    <!-- Demonstration of parent height pitfall -->
    <div class="short-parent">
        <div class="sticky-that-wont-work">
            This sticky element won't work because the parent is too short.
        </div>
    </div>
</body>
</html>
```

**CSS File (`sticky.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}

.table-container {
    /* Scroll container: enables sticky within this container */
    max-height: 300px;
    overflow-y: auto;
    border: 1px solid #ddd;
    border-radius: 8px;
    margin-bottom: 20px;
}

table {
    width: 100%;
    border-collapse: collapse;
}

.sticky-header {
    /* Sticky table header: sticks to the top of the scroll container */
    position: sticky;
    top: 0;
    background-color: #2c3e50;
    color: white;
    padding: 12px;
    text-align: left;
    z-index: 1;
}

td {
    padding: 12px;
    border-bottom: 1px solid #eee;
}

tbody tr:hover {
    background-color: #f5f5f5;
}

/* Parent height pitfall demonstration */
.short-parent {
    /* Parent is only as tall as its content — no room to stick */
    background-color: #fff3e0;
    border: 2px dashed #e65100;
    padding: 10px;
    margin-top: 20px;
}

.sticky-that-wont-work {
    position: sticky;
    top: 0;
    background-color: #e74c3c;
    color: white;
    padding: 10px;
    border-radius: 4px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `sticky.html` and CSS as `sticky.css`.
2. Open in a browser.
3. Scroll within the `.table-container` — the table header sticks to the top of the container.
4. Observe the `.short-parent` demonstration: the sticky element does not stick because the parent is not taller than the element.

**Expected Output:** A scrollable table with a dark header that remains visible as you scroll through the rows. The short-parent demonstration shows a red sticky element that does not stick because the parent has no extra height.

**Why This Works:** The `.sticky-header` has `position: sticky` and `top: 0`, so it sticks to the top of its nearest scrolling ancestor (`.table-container`). The `.sticky-that-wont-work` element fails because its parent (`.short-parent`) is not taller than the sticky element itself — there is no room for it to stick within the parent's bounds.

---

### Real-World Cases

- **Sticky table headers:** Keeping column labels visible while scrolling through long tables.
- **Sticky sidebars:** A sidebar that follows the user as they scroll through an article.
- **Section headers:** A section title that sticks to the top while scrolling through that section.
- **Sticky navigation:** A navigation bar that becomes sticky after scrolling past the hero section.

---

## 6. Modern Layout Positioning: CSS Anchor Positioning API

### Definitions

**Core Definition:** The CSS Anchor Positioning API is a native CSS mechanism that allows a positioned element to size and position itself relative to one or more "anchor elements" elsewhere on the page, without requiring JavaScript.

**Technical Definition:** The CSS Anchor Positioning specification defines anchor positioning, where a positioned element can size and position itself relative to one or more "anchor elements" elsewhere on the page. The API introduces the `anchor-name` property to designate an element as an anchor, the `position-anchor` property to associate a positioned element with an anchor, and the `anchor()` function to reference the anchor's edges (e.g., `top: anchor(bottom)`). Additional features include `position-area` (a 3×3 grid for positioning around the anchor), `position-try-fallbacks` (alternative positions when the primary position overflows), and `anchor-scope` (limiting anchor name visibility to a subtree). Anchor positioning builds on absolute positioning: the positioned element must have `position: absolute`. As of January 2026, with Firefox 147's release, anchor positioning is supported across all major browsers.

**Beginner-Friendly Explanation:** Anchor positioning is a new CSS feature that makes it easy to position things like tooltips, dropdown menus, and popovers relative to other elements. Instead of calculating positions with JavaScript, you just tell CSS: "This tooltip should be anchored to that button." You name the button as an anchor, then use the `anchor()` function to say where the tooltip should appear relative to the button — above it, below it, centered, etc. The browser handles all the calculations, and it even has fallbacks for when the preferred position does not fit on screen.

---

### Purposes

- To natively position tooltips, popovers, and dropdowns relative to trigger elements.
- To eliminate JavaScript-based positioning calculations for floating UI elements.
- To provide automatic fallback positioning when the preferred position overflows the viewport.
- To enable declarative positioning of complex UI components.
- To simplify responsive positioning of elements relative to other elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Step 1: Designate an anchor */
.anchor-element {
    anchor-name: --my-anchor;
}

/* Step 2: Associate a positioned element with the anchor */
.positioned-element {
    position: absolute;
    position-anchor: --my-anchor;
    /* Step 3: Use anchor() to position relative to the anchor */
    top: anchor(bottom);
    left: anchor(right);
}

/* Alternative: explicit anchor reference */
.positioned-element {
    position: absolute;
    top: anchor(--my-anchor bottom);
    left: anchor(--my-anchor right);
}

/* Position area: 3×3 grid around the anchor */
.positioned-element {
    position: absolute;
    position-anchor: --my-anchor;
    position-area: top; /* Places element above the anchor */
}

/* Fallback positions */
.positioned-element {
    position: absolute;
    position-anchor: --my-anchor;
    position-area: top;
    position-try-fallbacks: bottom, left, right;
}
```

#### Component Breakdown

| Property / Function | Description |
|---|---|
| `anchor-name` | Assigns a unique name (prefixed with `--`) to an element, making it an anchor. |
| `position-anchor` | Associates a positioned element with a named anchor. |
| `anchor()` | References the position of an anchor's edge (e.g., `anchor(top)`, `anchor(bottom)`). |
| `position-area` | Places the positioned element on a 3×3 grid surrounding the anchor. |
| `position-try-fallbacks` | Specifies alternative positions when the primary position overflows. |
| `anchor-scope` | Limits the scope of anchor names to a particular subtree. |
| `anchor-center` | Centers the positioned element relative to the anchor. |

#### Syntax Rules

1. The anchor element must have `anchor-name` set to a unique identifier prefixed with `--`.
2. The positioned element must have `position: absolute` (or `fixed`) for anchor positioning to work.
3. Use `position-anchor` for implicit anchoring, or `anchor()` with an explicit anchor name.
4. The `position-area` property uses a 3×3 grid: `top`, `bottom`, `left`, `right`, `top left`, `top right`, etc.
5. Fallback positions are specified with `position-try-fallbacks`.
6. `anchor-scope` limits anchor name visibility to a subtree to prevent naming conflicts.

#### Constraints and Limitations

- **Requires `position: absolute`** — anchor positioning builds on absolute positioning.
- **Browser support** — fully supported in Chrome 125+, Edge 125+, Safari 26+, and Firefox 147+. A polyfill is available for older browsers.
- **Transform-aware positioning** — anchor positioning initially did not follow transforms; this was fixed in later browser versions.
- **Complexity** — the API introduces many new properties and functions, which can be overwhelming for simple use cases.
- **Accessibility** — popovers and tooltips positioned with anchor positioning should still follow accessibility best practices for focus management and ARIA.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Anchor-Positioned Tooltip

**HTML File (`anchor.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Anchor Positioning</title>
    <link rel="stylesheet" href="anchor.css">
</head>
<body>
    <!-- The anchor element: a button -->
    <button class="anchor-button" popovertarget="my-tooltip">
        Hover or click me
    </button>

    <!-- The positioned element: a tooltip -->
    <div class="tooltip" popover id="my-tooltip">
        I am a tooltip anchored to the button above!
    </div>
</body>
</html>
```

**CSS File (`anchor.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    margin: 0;
    background-color: #f5f5f5;
}

/* Step 1: Designate the anchor */
.anchor-button {
    anchor-name: --my-anchor;
    padding: 12px 24px;
    font-size: 1rem;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}

/* Step 2: Position the tooltip relative to the anchor */
.tooltip {
    position: absolute;
    position-anchor: --my-anchor;
    /* Position the bottom of the tooltip at the top of the anchor */
    bottom: anchor(top);
    /* Center the tooltip horizontally relative to the anchor */
    justify-self: anchor-center;
    background-color: #2c3e50;
    color: white;
    padding: 8px 16px;
    border-radius: 6px;
    font-size: 0.9rem;
    /* Fallback positions if the preferred position overflows */
    position-try-fallbacks: bottom, left, right;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `anchor.html` and CSS as `anchor.css`.
2. Open in a browser that supports anchor positioning (Chrome 125+, Safari 26+, Firefox 147+).
3. Click the button — the tooltip appears above the button, centered.
4. Resize the window to see the fallback positions in action.

**Expected Output:** A blue button with a dark tooltip positioned above it, centered horizontally. If there is not enough space above, the tooltip moves below, left, or right.

**Why This Works:** The `anchor-name: --my-anchor` on the button designates it as an anchor. The `.tooltip` has `position: absolute` and `position-anchor: --my-anchor`, associating it with the button. The `bottom: anchor(top)` places the tooltip's bottom edge at the anchor's top edge. The `justify-self: anchor-center` centers it horizontally. The `position-try-fallbacks` provide alternatives when the preferred position overflows.

---

### Real-World Cases

- **Tooltips:** Positioning a tooltip relative to a button or icon without JavaScript.
- **Dropdown menus:** Positioning a menu below a navigation item, with fallback positions for viewport edges.
- **Popovers:** Native HTML popovers positioned relative to their trigger elements.
- **Floating labels:** Form field labels that float above the input when focused.

---

## References

- MDN Web Docs — `position` - https://developer.mozilla.org/en-US/docs/Web/CSS/position
- W3C — CSS Positioned Layout Module Level 3 - https://www.w3.org/TR/css-position-3/
- W3C — CSS Anchor Positioning Specification - https://drafts.csswg.org/css-anchor-position/
- Chrome for Developers — CSS Anchor Positioning API - https://developer.chrome.com/docs/css-ui/anchor-positioning-api
- MDN Web Docs — CSS Anchor Positioning - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning
- CSS-Tricks — `position-area` - https://css-tricks.com/almanac/properties/p/position-area/
- CSS-Tricks — `anchor-scope` - https://css-tricks.com/almanac/properties/a/anchor-scope/
- web.dev — Cumulative Layout Shift (CLS) - https://web.dev/articles/cls
- Oddbird — CSS Anchor Positioning Polyfill - https://github.com/oddbird/css-anchor-positioning
- W3C — CSS Positioned Layout Module Level 3 (Editor's Draft) - https://drafts.csswg.org/css-position/