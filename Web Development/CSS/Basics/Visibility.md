# CSS Visibility, Rendering Optimization, & Layout Effects: A Comprehensive Research Guide

## Topic Overview

### Definitions

**Core Definition:** This topic encompasses the CSS properties and browser rendering mechanisms that control whether, when, and how elements are displayed, hidden, or optimized for performance, including the interaction between visual hiding, layout suppression, rendering pipeline optimization, and accessibility.

**Technical Definition:** The CSS visibility and rendering optimization model comprises properties that affect the browser's rendering pipeline at different stages: layout (reflow), paint (repaint), and compositing. Properties such as `display`, `visibility`, and `opacity` control element visibility with distinct effects on the box tree, layout flow, and accessibility tree. Performance-oriented properties such as `content-visibility`, `will-change`, and `contain` allow developers to hint at browser optimizations by isolating subtrees from the rest of the document. The accessibility tree, which is derived from the DOM but filtered by CSS and ARIA attributes, determines what assistive technologies can perceive.

**Beginner-Friendly Explanation:** When you build a web page, you often need to hide things (like dropdown menus), show things (like modals), or make things run faster (especially on long pages). CSS gives you several tools for this, but they work very differently. Some make elements completely disappear (as if they were deleted), some make them invisible but still take up space, and some just make them transparent. There are also special properties that tell the browser "hey, this part of the page won't affect anything else, so you can optimize it." Understanding these differences is crucial because they affect not just what you see, but also how fast the page runs and whether people using screen readers can access your content.


## Core Concept 1: Property Behavior Comparison — Element Layout Suppression, Rendering Hiding, and Transparency Modulation

### Definitions

**Core Definition:** `display: none`, `visibility: hidden`, and `opacity: 0` are three CSS mechanisms for hiding elements, each with fundamentally different effects on layout, rendering, interactivity, and accessibility.

**Technical Definition:** `display: none` removes the element and its descendants from the box tree entirely; no boxes are generated, and the document is rendered as if the element did not exist. `visibility: hidden` causes the element's box to be invisible (not drawn) but preserves its space in the layout; the element still participates in the box tree and affects layout as normal. `opacity: 0` makes the element fully transparent while it remains fully present in the box tree, layout, and rendering pipeline; it is still painted (with zero alpha) and remains interactive. `display: none` removes the element from the accessibility tree; `visibility: hidden` also removes the element from the accessibility tree; `opacity: 0` does not remove the element from the accessibility tree, and the element remains focusable and interactive.

**Beginner-Friendly Explanation:** Think of your page as a room with furniture. `display: none` is like removing a chair from the room entirely — there's no trace of it, and everything else shifts to fill the space. `visibility: hidden` is like putting an invisible cover over the chair — it's still there taking up space, but you can't see it, and you can't sit on it. `opacity: 0` is like making the chair completely transparent — it's there, you can sit on it (click it), and it still occupies its space, but you can't see it. Each approach is useful in different situations.

### Purposes

- To completely remove an element from the visual layout and accessibility tree when it should not exist at all (`display: none`)
- To hide an element visually while preserving its space in the layout, preventing layout shifts (`visibility: hidden`)
- To create fade-in/fade-out animations and transparency effects while keeping the element interactive and accessible (`opacity: 0`)
- To toggle UI components on and off without affecting the surrounding layout
- To provide three distinct levels of "hiding" appropriate for different use cases

### Syntax Rules and Structure

**Complete General Syntax:**

```css
/* Layout suppression */
display: none;

/* Rendering hiding */
visibility: visible | hidden | collapse;

/* Transparency modulation */
opacity: <number> | <percentage>;
```

**Component Breakdown:**

| Property | Value | Description |
|----------|-------|-------------|
| `display` | `none` | Element generates no boxes; removed from layout and accessibility tree |
| `visibility` | `visible` | Element is visible (default) |
| `visibility` | `hidden` | Element box is invisible but still affects layout; removed from accessibility tree |
| `visibility` | `collapse` | For table rows/columns, collapses them; for other elements, behaves like `hidden` |
| `opacity` | `<number>` | 0 = fully transparent, 1 = fully opaque; values between are partial transparency |
| `opacity` | `<percentage>` | 0% = fully transparent, 100% = fully opaque |

**Syntax Rules:**

1. `display: none` is a `<display-box>` value. It removes the element and all descendants from the box tree. The document is rendered as if the element did not exist. The element is removed from the accessibility tree and is not focusable.
2. `visibility: hidden` makes the element's box invisible but still affects layout as normal. Descendants can override by setting `visibility: visible`. The element is removed from the accessibility tree.
3. `visibility: collapse` is designed for table rows and columns; for other elements, it behaves like `hidden`.
4. `opacity` accepts values from `0.0` to `1.0` (inclusive), or percentages from `0%` to `100%`. `opacity: 0` makes the element fully transparent; it remains in the layout and accessibility tree.
5. `opacity` creates a new stacking context when its value is less than 1.

**Constraints and Limitations:**

- `display: none` cannot be transitioned or animated in the traditional sense; transitions to/from `none` behave discretely (though `@starting-style` and `transition-behavior: allow-discrete` are changing this).
- `visibility` can be transitioned between `visible` and `hidden`, with a discrete step at the 50% mark.
- `opacity` is animatable and is one of the few properties that can be composited on the GPU without triggering repaint or reflow.
- Elements with `opacity: 0` are still clickable and focusable; this can create accessibility issues if not handled carefully.
- `visibility: hidden` elements are not focusable and are removed from the accessibility tree.
- `display: none` elements are not focusable and are removed from the accessibility tree.

### Multiple Annotated Code Examples

#### Example 1: Comparing All Three Hiding Mechanisms

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Display vs Visibility vs Opacity</title>
  <style>
    /* Step 1: Common styling for all demo boxes */
    .container {
      background-color: #f5f5f5;
      padding: 20px;
      margin-bottom: 20px;
      border: 2px dashed #999;
    }

    .box {
      width: 120px;
      height: 80px;
      background-color: #bbdefb;
      border: 2px solid #1976d2;
      margin: 8px;
      display: inline-block;
      vertical-align: top;
      text-align: center;
      line-height: 80px;
      font-weight: bold;
      transition: opacity 0.3s ease, visibility 0.3s ease;
    }

    /* Step 2: Layout suppression — completely removed */
    .display-none {
      display: none;
    }

    /* Step 3: Rendering hiding — invisible but space preserved */
    .visibility-hidden {
      visibility: hidden;
    }

    /* Step 4: Transparency modulation — fully transparent, still interactive */
    .opacity-zero {
      opacity: 0;
    }

    /* Step 5: Hover effect to demonstrate interactivity */
    .opacity-zero:hover {
      opacity: 1;                     /* Becomes visible on hover */
    }
  </style>
</head>
<body>
  <h2>Original — All three boxes visible</h2>
  <div class="container">
    <div class="box">Box 1</div>
    <div class="box">Box 2</div>
    <div class="box">Box 3</div>
  </div>

  <h2>display: none on Box 2 — removed from layout</h2>
  <div class="container">
    <div class="box">Box 1</div>
    <div class="box display-none">Box 2</div>
    <div class="box">Box 3</div>
  </div>

  <h2>visibility: hidden on Box 2 — space preserved</h2>
  <div class="container">
    <div class="box">Box 1</div>
    <div class="box visibility-hidden">Box 2</div>
    <div class="box">Box 3</div>
  </div>

  <h2>opacity: 0 on Box 2 — invisible but interactive (hover to reveal)</h2>
  <div class="container">
    <div class="box">Box 1</div>
    <div class="box opacity-zero">Box 2</div>
    <div class="box">Box 3</div>
  </div>
</body>
</html>
```

**Expected Output:**

- **Original:** Three blue boxes side by side.
- **With `display: none`:** Only two boxes visible (1 and 3), with Box 3 shifted to the left to fill the gap left by Box 2.
- **With `visibility: hidden`:** Three boxes with Box 2 invisible; Box 3 remains in its original position, leaving a gap where Box 2 is.
- **With `opacity: 0`:** Three boxes with Box 2 fully transparent; Box 3 remains in its original position, leaving a gap. Hovering over the gap reveals Box 2, demonstrating that it remains interactive.

**Explanation of Results:**

- `display: none` removes the element from the box tree, causing subsequent elements to reflow. The element is not focusable and is removed from the accessibility tree.
- `visibility: hidden` removes the element from the rendering tree (it is not painted) but preserves its space in the layout. The element is not focusable and is removed from the accessibility tree.
- `opacity: 0` makes the element fully transparent but it remains in the rendering tree and layout. It is still focusable and interactive, and remains in the accessibility tree.

#### Example 2: JavaScript Toggling and Performance

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Performance of Hiding Mechanisms</title>
  <style>
    .panel {
      width: 300px;
      padding: 20px;
      margin: 10px;
      background-color: #e3f2fd;
      border: 2px solid #1976d2;
      font-size: 1.1em;
    }

    /* Step 1: Use display: none for initial hiding */
    .hidden-display {
      display: none;
    }

    /* Step 2: Use visibility for layout-preserving hiding */
    .hidden-visibility {
      visibility: hidden;
    }

    /* Step 3: Use opacity for animation-friendly hiding */
    .hidden-opacity {
      opacity: 0;
      pointer-events: none;             /* Prevent interaction when hidden */
      transition: opacity 0.3s ease;
    }
  </style>
</head>
<body>
  <h2>Performance Comparison of Hiding Mechanisms</h2>

  <div class="panel" id="displayPanel">
    <strong>display: none</strong> — Element is removed from layout.
  </div>

  <div class="panel" id="visibilityPanel">
    <strong>visibility: hidden</strong> — Element is invisible but space preserved.
  </div>

  <div class="panel" id="opacityPanel">
    <strong>opacity: 0</strong> — Element is transparent but interactive.
  </div>

  <button onclick="toggleDisplay()">Toggle display: none</button>
  <button onclick="toggleVisibility()">Toggle visibility: hidden</button>
  <button onclick="toggleOpacity()">Toggle opacity: 0</button>

  <script>
    // Step 4: Toggle display — triggers reflow
    function toggleDisplay() {
      const el = document.getElementById('displayPanel');
      el.classList.toggle('hidden-display');
    }

    // Step 5: Toggle visibility — triggers repaint only
    function toggleVisibility() {
      const el = document.getElementById('visibilityPanel');
      el.classList.toggle('hidden-visibility');
    }

    // Step 6: Toggle opacity — composited, minimal repaint/reflow
    function toggleOpacity() {
      const el = document.getElementById('opacityPanel');
      el.classList.toggle('hidden-opacity');
    }
  </script>
</body>
</html>
```

**Expected Output:**

Three panels and three buttons. Clicking "Toggle display: none" causes the first panel to disappear and reappear, with the layout shifting as the panel is removed and re-inserted. Clicking "Toggle visibility: hidden" causes the second panel to become invisible and visible while the layout remains stable. Clicking "Toggle opacity: 0" causes the third panel to fade out and fade in via the CSS transition, while the layout remains stable.

**Explanation of Results:**

- Toggling `display` triggers a reflow (layout recalculation) because the element is added to or removed from the box tree.
- Toggling `visibility` triggers a repaint but not a reflow, because the element's space is preserved.
- Toggling `opacity` can be handled by the compositor in modern browsers, requiring neither reflow nor repaint, making it the most performant option for animations.

### Real-World Cases

- **Dropdown Menus:** Using `display: none` to hide dropdown menus when closed is common, but `visibility: hidden` with `opacity` transitions is preferred for smooth animations. `visibility: hidden` is also used to hide content from both visual users and screen readers while preserving layout.
- **Modal Dialogs:** Modals are often shown/hidden using `display: none` toggled via JavaScript, but `opacity` transitions are used for fade-in/fade-out effects. `aria-hidden="true"` is used alongside to hide the rest of the page from screen readers.
- **Tab Interfaces:** Tab panels are typically hidden with `display: none` (removed from layout) or `visibility: hidden` (space preserved), depending on whether the layout should remain stable.
- **Responsive Design:** Hiding desktop navigation on mobile with `display: none` and showing a hamburger menu is a common pattern.

### References

- MDN Web Docs — display - https://developer.mozilla.org/en-US/docs/Web/CSS/display
- MDN Web Docs — visibility - https://developer.mozilla.org/en-US/docs/Web/CSS/visibility
- MDN Web Docs — opacity - https://developer.mozilla.org/en-US/docs/Web/CSS/opacity
- CSS Display Module Level 3 - https://drafts.csswg.org/css-display/
- MDN Web Docs — Element.checkVisibility() - https://developer.mozilla.org/en-US/docs/Web/API/Element/checkVisibility


## Core Concept 2: Visual vs. Layout Effects — Repaint vs. Reflow Performance Costs

### Definitions

**Core Definition:** Reflow (layout) is the browser process of recalculating the positions and dimensions of elements on a page; repaint is the process of redrawing elements' visual appearance without changing their layout.

**Technical Definition:** Reflow (also called layout) occurs when changes affect the geometry of elements — their size, position, or the document structure. The browser must recalculate the layout tree, which can cascade from a single element to its ancestors, descendants, and siblings. Repaint occurs when changes affect only the visual appearance of an element (e.g., `background-color`, `color`, `box-shadow`) without affecting its geometry. Repaint is less expensive than reflow because it does not require layout recalculation. The browser accumulates reflow and repaint operations and performs them asynchronously, but certain operations (e.g., reading `offsetHeight`) force synchronous layout.

**Beginner-Friendly Explanation:** Imagine you're rearranging furniture in a room. Reflow is like moving the furniture around — you have to recalculate where everything goes, and moving one piece might require moving others. Repaint is like repainting the walls — the furniture stays in place, you just change the color. Reflow is much more work for the browser, so it is slower. The key to performance is to minimize reflow by batching changes and using properties that don't affect layout.

### Purposes

- To understand the performance implications of different CSS changes so developers can make informed optimization decisions
- To minimize browser work by preferring repaint-only properties (e.g., `color`, `background-color`) over reflow-triggering properties (e.g., `width`, `height`, `top`, `left`)
- To use compositor-friendly properties (`transform`, `opacity`) for animations that don't trigger reflow or repaint
- To avoid layout thrashing (forced synchronous layout) in JavaScript by batching DOM reads and writes
- To optimize rendering performance for complex, dynamic web applications

### Syntax Rules and Structure

**Reflow-Triggering Operations (High Cost):**

- Modifying CSS styles that affect geometry: `width`, `height`, `padding`, `margin`, `border`, `font-size`, `top`, `left`, `right`, `bottom`, `display`, `position`, `float`
- Adding, removing, or modifying DOM nodes
- Resizing the window or changing the default font
- Reading layout properties in JavaScript: `offsetHeight`, `offsetWidth`, `offsetTop`, `scrollTop`, `getComputedStyle()`
- Activating CSS pseudo-classes (e.g., `:hover`) that change layout

**Repaint-Triggering Operations (Medium Cost):**

- Modifying visual-only properties: `color`, `background-color`, `background-image`, `border-color`, `box-shadow`, `outline`, `visibility`
- Changing `visibility` from `visible` to `hidden` (triggers repaint, not reflow)

**Compositor-Only Operations (Low Cost):**

- `transform` (translate, scale, rotate)
- `opacity`
- `filter` (in some cases)

These can be handled by the GPU compositor without triggering reflow or repaint on the main thread.

**Syntax Rules:**

1. Reflow is triggered by any change that affects the layout tree. The browser processes reflows in batches, but forced synchronous layout (layout thrashing) occurs when JavaScript reads a layout property immediately after writing to the DOM.
2. Repaint is triggered by visual changes that do not affect geometry. Repaint may be scoped to a specific region or the entire viewport.
3. Properties in the compositor-only category can be animated smoothly at 60fps because they bypass the main thread.
4. `will-change` can promote an element to its own compositor layer, making `transform` and `opacity` animations even more efficient.

**Constraints and Limitations:**

- `transform` and `opacity` are compositor-friendly only when the element has its own compositor layer; otherwise, they still trigger paint.
- Forcing an element into a compositor layer (via `will-change` or `transform: translateZ(0)`) consumes memory and should not be overused.
- Reading layout properties in a loop that also modifies the DOM causes layout thrashing, which is extremely expensive.
- `display: none` toggling triggers reflow; `visibility` toggling triggers repaint; `opacity` toggling can be compositor-only.

### Multiple Annotated Code Examples

#### Example 1: Reflow vs. Repaint vs. Compositor

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Reflow, Repaint, and Compositor</title>
  <style>
    .demo-box {
      width: 200px;
      height: 100px;
      background-color: #bbdefb;
      border: 2px solid #1976d2;
      margin: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.1em;
      transition: all 0.3s ease;
    }

    /* Step 1: Reflow — changing width triggers layout */
    .reflow-demo:hover {
      width: 300px;                    /* Reflow: geometry changes */
    }

    /* Step 2: Repaint — changing background triggers paint only */
    .repaint-demo:hover {
      background-color: #ffcdd2;       /* Repaint: visual only */
    }

    /* Step 3: Compositor — transform and opacity bypass layout/paint */
    .compositor-demo:hover {
      transform: translateX(50px);     /* Compositor: no reflow/repaint */
      opacity: 0.5;                    /* Compositor: no reflow/repaint */
    }
  </style>
</head>
<body>
  <h2>Reflow vs Repaint vs Compositor</h2>

  <div class="demo-box reflow-demo">
    Hover me — <strong>Reflow</strong> (width changes)
  </div>

  <div class="demo-box repaint-demo">
    Hover me — <strong>Repaint</strong> (background changes)
  </div>

  <div class="demo-box compositor-demo">
    Hover me — <strong>Compositor</strong> (transform + opacity)
  </div>
</body>
</html>
```

**Expected Output:**

Three blue boxes. Hovering the first box causes it to expand from 200px to 300px, pushing content to the right (reflow). Hovering the second box changes its background color to red without changing its size or position (repaint). Hovering the third box causes it to slide to the right and become semi-transparent (compositor-only).

**Explanation of Results:**

- The first box changes `width`, which alters its geometry. The browser must recalculate the layout for the box and any affected siblings or ancestors (reflow).
- The second box changes `background-color`, which is a visual-only property. The browser only needs to repaint the box's background (repaint).
- The third box changes `transform` and `opacity`, which can be handled by the GPU compositor. The browser does not need to recalculate layout or repaint the element (compositor-only).

#### Example 2: Layout Thrashing Demonstration

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Layout Thrashing</title>
  <style>
    .box {
      width: 100px;
      height: 100px;
      background-color: #c8e6c9;
      margin: 5px;
      display: inline-block;
    }
  </style>
</head>
<body>
  <h2>Layout Thrashing vs. Batched Writes</h2>

  <div id="container">
    <div class="box"></div>
    <div class="box"></div>
    <div class="box"></div>
  </div>

  <button onclick="badLayout()">Bad: Interleaved Read/Write</button>
  <button onclick="goodLayout()">Good: Batched Writes</button>

  <script>
    // Step 1: BAD — interleaving reads and writes forces synchronous layout
    function badLayout() {
      const boxes = document.querySelectorAll('.box');
      boxes.forEach(box => {
        box.style.width = (box.offsetWidth + 10) + 'px';  // Read then write — forces reflow
      });
    }

    // Step 2: GOOD — batching writes avoids forced synchronous layout
    function goodLayout() {
      const boxes = document.querySelectorAll('.box');
      const widths = [];
      // Read all widths first
      boxes.forEach(box => {
        widths.push(box.offsetWidth);
      });
      // Then write all widths
      boxes.forEach((box, i) => {
        box.style.width = (widths[i] + 10) + 'px';
      });
    }
  </script>
</body>
</html>
```

**Expected Output:**

Three green boxes in a row. Clicking the "Bad" button causes a noticeable delay (if the boxes are numerous) because the browser forces a synchronous layout for each box. Clicking the "Good" button is faster because all reads are done first, followed by all writes.

**Explanation of Results:**

- In `badLayout()`, each iteration reads `offsetWidth` and immediately writes `style.width`. This forces the browser to recalculate layout synchronously before each read, resulting in multiple reflows (layout thrashing).
- In `goodLayout()`, all `offsetWidth` values are read first, then all writes are performed. The browser can batch the writes and perform a single reflow.

### Real-World Cases

- **Animation Performance:** Using `transform` and `opacity` for animations instead of `top`/`left` or `width`/`height` ensures smooth 60fps animations without jank.
- **Scroll Performance:** Avoiding layout-triggering changes during scroll events prevents scroll jank. Using `will-change: transform` on elements that will move during scroll can help.
- **Form Validation:** Dynamically adding/removing error messages with `display: none` triggers reflow; using `visibility: hidden` or `opacity` with `max-height` transitions can be smoother.
- **Virtual Scrolling:** Libraries like React Virtualized avoid rendering all items at once, reducing reflow cost.

### References

- MDN Web Docs — CSS performance optimization - https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/CSS
- web.dev — Rendering performance - https://web.dev/articles/rendering-performance
- MDN Web Docs — will-change - https://developer.mozilla.org/en-US/docs/Web/CSS/will-change
- Google Developers — Minimizing browser reflow - https://developers.google.com/speed/docs/insights/browser-reflow


## Core Concept 3: Next-Gen Rendering Properties — `content-visibility` and `will-change`

### Definitions

**Core Definition:** `content-visibility` and `will-change` are CSS properties that allow developers to hint at browser rendering optimizations, respectively by skipping rendering of off-screen content and by pre-announcing upcoming visual changes.

**Technical Definition:** `content-visibility` controls whether an element renders its contents at all, and applies a strong set of CSS containment rules. The `auto` value turns on layout, style, and paint containment and allows the user agent to skip rendering work for off-screen content; the element's children are not laid out or painted until the element approaches the viewport. `will-change` provides a hint to the browser about which properties are expected to change, allowing the browser to set up optimizations (typically by creating a new compositor layer) before the change actually occurs.

**Beginner-Friendly Explanation:** `content-visibility` is like telling the browser "don't bother drawing this section of the page until the user scrolls near it." On long pages, this can dramatically speed up initial load because the browser skips work on parts the user can't see yet. `will-change` is like telling the browser "I'm about to animate this element, so get ready." The browser can prepare a special fast path (a compositor layer) so the animation runs smoothly.

### Purposes

- To reduce initial page rendering time by skipping layout and paint work for off-screen content (`content-visibility: auto`)
- To hint the browser about upcoming visual changes so it can pre-optimize (`will-change`)
- To improve Interaction to Next Paint (INP) scores by reducing the work done during page load and after interactions
- To enable smooth animations by promoting elements to their own compositor layers
- To provide a declarative, CSS-only alternative to JavaScript-based virtual scrolling for simple cases

### Syntax Rules and Structure

**Complete General Syntax:**

```css
/* content-visibility */
content-visibility: visible | auto | hidden;

/* will-change */
will-change: auto | scroll-position | contents | <custom-ident>;
```

**Component Breakdown:**

| Property | Value | Description |
|----------|-------|-------------|
| `content-visibility` | `visible` | No effect; contents render normally |
| `content-visibility` | `auto` | Turns on layout, style, and paint containment; skips rendering off-screen content until needed |
| `content-visibility` | `hidden` | Turns on layout, style, and paint containment; skips rendering contents entirely (similar to `display: none` but preserves the element's box) |
| `will-change` | `auto` | No hint; browser uses default optimizations |
| `will-change` | `scroll-position` | Indicates the element's scroll position will change |
| `will-change` | `contents` | Indicates the element's contents will change |
| `will-change` | `<custom-ident>` | Property name(s) that will change (e.g., `transform`, `opacity`) |

**Syntax Rules:**

1. `content-visibility: auto` applies `contain: layout style paint` and `contain: size` when the element is off-screen. The element's contents remain in the DOM and accessibility tree. When the element approaches the viewport, the browser removes `size` containment and renders the contents.
2. `content-visibility: hidden` applies `contain: layout style paint` and skips rendering the element's contents entirely. The element's box is preserved, but its children are not rendered.
3. `will-change` accepts one or more comma-separated property names or keywords.
4. `will-change: transform` creates a new compositor layer for the element, making `transform` animations compositor-only.
5. `will-change` should be applied shortly before the change occurs and removed after the change is complete, to avoid excessive memory usage from too many compositor layers.
6. `content-visibility: auto` requires `contain-intrinsic-size` (or its longhands) to provide an estimated size for off-screen elements, preventing scrollbar jumps.

**Constraints and Limitations:**

- `content-visibility: auto` can cause anchor links to jump incorrectly if the target section hasn't been rendered yet. Browsers have implemented `content-visibility: auto` support for anchor links, but behavior varies.
- Find-in-page (Ctrl+F) may not search un-rendered content in some browsers.
- Screen readers may skip contained sections in some browsers, though the specification requires off-screen content to remain in the accessibility tree.
- `will-change` overuse creates memory pressure from too many compositor layers; it should be used only on elements that will actually change and removed after the change.
- `will-change: transform` can cause blurry text on some devices due to rasterization changes.

### Multiple Annotated Code Examples

#### Example 1: `content-visibility: auto` on a Long Page

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>content-visibility: auto</title>
  <style>
    .section {
      /* Step 1: Apply content-visibility: auto to off-screen sections */
      content-visibility: auto;

      /* Step 2: Provide an estimated size to prevent scrollbar jumping */
      contain-intrinsic-size: auto 500px;

      padding: 40px;
      margin: 20px;
      background-color: #e3f2fd;
      border: 2px solid #1976d2;
    }

    .section h2 {
      color: #1565c0;
    }

    .section p {
      line-height: 1.6;
      color: #333;
    }
  </style>
</head>
<body>
  <h1>Long Page with content-visibility: auto</h1>

  <!-- Step 3: First section is visible immediately -->
  <div class="section">
    <h2>Section 1 — Visible on load</h2>
    <p>This section is above the fold and renders immediately. It is fully interactive and accessible.</p>
  </div>

  <!-- Step 4: Subsequent sections are off-screen and skipped -->
  <div class="section">
    <h2>Section 2 — Off-screen, rendering skipped</h2>
    <p>This section is below the fold. The browser skips its layout and paint until the user scrolls near it.</p>
  </div>

  <div class="section">
    <h2>Section 3 — Off-screen, rendering skipped</h2>
    <p>This section is also below the fold. Its rendering work is deferred until needed.</p>
  </div>

  <div class="section">
    <h2>Section 4 — Off-screen, rendering skipped</h2>
    <p>This section is also below the fold. Its rendering work is deferred until needed.</p>
  </div>

  <div class="section">
    <h2>Section 5 — Off-screen, rendering skipped</h2>
    <p>This section is also below the fold. Its rendering work is deferred until needed.</p>
  </div>
</body>
</html>
```

**Expected Output:**

A long page with five blue-bordered sections. On initial load, only Section 1 is rendered. Sections 2–5 are not laid out or painted, reducing initial rendering work. As the user scrolls, each section is rendered just before it enters the viewport. The `contain-intrinsic-size: auto 500px` provides an estimated height for un-rendered sections, preventing scrollbar jumping.

**Explanation of Results:**

- `content-visibility: auto` tells the browser to skip rendering for off-screen sections.
- `contain-intrinsic-size: auto 500px` provides an estimated height of 500px for each un-rendered section. The `auto` keyword means the browser remembers the actual size once the section has been rendered.
- The combination of skipped rendering and size estimation results in faster initial load and stable scrolling.

#### Example 2: `will-change` for Animation Optimization

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>will-change for Smooth Animation</title>
  <style>
    .animated-box {
      width: 100px;
      height: 100px;
      background-color: #c8e6c9;
      border: 2px solid #388e3c;
      margin: 20px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.2em;

      /* Step 1: Hint that transform will change */
      will-change: transform;

      /* Step 2: Define the animation */
      animation: slide 2s ease-in-out infinite alternate;
    }

    @keyframes slide {
      from {
        transform: translateX(0);
      }
      to {
        transform: translateX(300px);
      }
    }

    /* Step 3: Another box without will-change for comparison */
    .no-will-change {
      will-change: auto;                /* No hint */
    }
  </style>
</head>
<body>
  <h2>will-change: transform — Smooth Animation</h2>
  <div class="animated-box">Smooth</div>

  <h2>No will-change — May be less smooth</h2>
  <div class="animated-box no-will-change">Default</div>
</body>
</html>
```

**Expected Output:**

Two boxes animating from left to right. The first box uses `will-change: transform` and animates smoothly on the compositor thread. The second box does not use `will-change` and may be less smooth because the browser must paint each frame on the main thread.

**Explanation of Results:**

- `will-change: transform` promotes the element to its own compositor layer before the animation starts. The `transform` animation can then be handled by the GPU compositor without triggering reflow or repaint on the main thread.
- Without `will-change`, the browser may not create a compositor layer, forcing the animation to be painted on the main thread, which can result in jank on complex pages.

### Real-World Cases

- **Long Articles and Blogs:** `content-visibility: auto` on article sections can significantly reduce initial load time and improve INP.
- **Infinite Scroll Feeds:** Applying `content-visibility: auto` to feed items below the fold reduces rendering work as new items are appended.
- **Carousels and Sliders:** `will-change: transform` on carousel slides ensures smooth sliding animations.
- **Modal and Dropdown Animations:** `will-change: transform, opacity` on elements that will animate in/out improves animation smoothness.
- **Scroll-Driven Animations:** `will-change: transform` on elements that move during scroll prevents scroll jank.

### References

- MDN Web Docs — content-visibility - https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility
- MDN Web Docs — will-change - https://developer.mozilla.org/en-US/docs/Web/CSS/will-change
- web.dev — content-visibility: the new CSS property that boosts your rendering performance - https://web.dev/articles/content-visibility
- web.dev — CSS content-visibility is now Baseline - https://web.dev/blog/css-content-visibility-baseline
- CSS Containment Module Level 2 — content-visibility - https://drafts.csswg.org/css-contain-2/#content-visibility


## Core Concept 4: Layout Containment Rules — `contain` Property for Paint and Style Boundaries

### Definitions

**Core Definition:** The `contain` property allows developers to indicate that an element and its contents are independent of the rest of the document tree, enabling the browser to limit layout, style, paint, and size calculations to a subtree rather than the entire page.

**Technical Definition:** The `contain` property accepts one or more of the keywords `size`, `layout`, `style`, and `paint`, or the shorthand values `strict` (equivalent to `size layout paint`) and `content` (equivalent to `layout paint`). `layout` containment makes the element act as a containing block for absolutely and fixed positioned descendants, establishes a new stacking context and block formatting context, and ensures that nothing outside the element affects its internal layout. `paint` containment clips descendants to the element's bounds and ensures they do not display outside. `style` containment ensures that counters and other style effects do not escape the element. `size` containment allows the element to be sized without examining its descendants.

**Beginner-Friendly Explanation:** The `contain` property is like telling the browser "this box is a self-contained unit — nothing inside it affects anything outside, and nothing outside affects anything inside." This allows the browser to skip work when rendering the rest of the page, because it knows that changes inside the box won't cause changes elsewhere. It is especially useful for pages with many independent widgets, like dashboards or card grids.

### Purposes

- To isolate a DOM subtree so that its internal layout, style, and paint operations do not affect the rest of the page
- To reduce the scope of reflow and repaint when changes occur inside a contained element
- To improve rendering performance on pages with many independent widgets
- To provide a foundation for `content-visibility: auto`, which automatically applies containment
- To enable predictable containment of counters, floats, and absolutely positioned elements

### Syntax Rules and Structure

**Complete General Syntax:**

```css
/* Keyword values */
contain: none;
contain: strict;          /* Equivalent to size layout paint */
contain: content;         /* Equivalent to layout paint */

/* Individual keywords */
contain: size;
contain: layout;
contain: style;
contain: paint;

/* Multiple keywords */
contain: size paint;
contain: size layout paint;
contain: layout style paint;

/* Global values */
contain: inherit;
contain: initial;
contain: revert;
contain: unset;
```

**Component Breakdown:**

| Keyword | Description |
|---------|-------------|
| `none` | No containment applied |
| `strict` | All containment rules except `style` (equivalent to `size layout paint`) |
| `content` | All containment rules except `size` and `style` (equivalent to `layout paint`) |
| `size` | Element can be sized without examining its descendants' sizes |
| `layout` | Nothing outside the element may affect its internal layout and vice versa |
| `style` | Style effects (counters, etc.) do not escape the containing element |
| `paint` | Descendants do not display outside the element's bounds |

**Syntax Rules:**

1. `contain` accepts one or more keywords in any order.
2. `contain: layout` makes the element a containing block for absolutely and fixed positioned descendants, creates a new stacking context, and establishes a new block formatting context.
3. `contain: paint` clips descendants to the element's border-box and ensures they do not display outside.
4. `contain: style` ensures that counters and other style effects do not escape the element.
5. `contain: size` allows the element to be sized without examining its descendants; the element's size must be determined by its own properties (width, height, etc.) or by the containing block.
6. `contain: strict` applies all containment except `style`; `contain: content` applies `layout` and `paint`.
7. Applying `layout`, `paint`, `strict`, or `content` creates a new containing block, stacking context, and block formatting context.

**Constraints and Limitations:**

- `contain: size` requires that the element's size be determinable without its contents; if the element relies on its children for sizing, the size will collapse to zero.
- `contain: paint` clips overflow, which may be undesirable if content is meant to overflow (e.g., tooltips, dropdowns).
- Containment is not a substitute for proper semantic structure; it is a performance optimization.
- `contain: style` is not supported in all browsers and has limited use cases.
- The `contain` property does not inherit.

### Multiple Annotated Code Examples

#### Example 1: `contain: layout paint` on Independent Widgets

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>contain: layout paint</title>
  <style>
    /* Step 1: Apply containment to each widget */
    .widget {
      contain: layout paint;            /* Isolate layout and paint */
      width: 280px;
      padding: 20px;
      margin: 10px;
      background-color: #e3f2fd;
      border: 2px solid #1976d2;
      display: inline-block;
      vertical-align: top;
    }

    .widget h3 {
      margin-top: 0;
      color: #1565c0;
    }

    .widget .content {
      background-color: #fff;
      padding: 10px;
      border: 1px solid #90caf9;
    }

    /* Step 2: An element that overflows its container */
    .overflow-demo {
      width: 200px;
      height: 60px;
      background-color: #ffcdd2;
      border: 2px solid #c62828;
      padding: 10px;
      overflow: visible;                /* Content will be clipped by contain: paint */
    }
  </style>
</head>
<body>
  <h2>contain: layout paint on Independent Widgets</h2>

  <div class="widget">
    <h3>Widget 1</h3>
    <div class="content">
      <p>This widget is isolated with <code>contain: layout paint</code>.</p>
      <p>Changes inside this widget will not affect the layout of other widgets.</p>
    </div>
  </div>

  <div class="widget">
    <h3>Widget 2</h3>
    <div class="content">
      <div class="overflow-demo">
        This content overflows but is clipped by paint containment.
      </div>
    </div>
  </div>

  <div class="widget">
    <h3>Widget 3</h3>
    <div class="content">
      <p>Each widget is independent. The browser can optimize rendering by
      limiting its work to the contained subtree.</p>
    </div>
  </div>
</body>
</html>
```

**Expected Output:**

Three blue-bordered widgets arranged side by side. The second widget contains a red-bordered box with overflowing content. Because of `contain: paint`, the overflowing content is clipped to the widget's bounds, and the overflow does not affect other widgets. Changes inside any widget (e.g., resizing its content) will not trigger reflow of the other widgets.

**Explanation of Results:**

- `contain: layout` isolates the widget's internal layout so that it does not affect the layout of other elements, and vice versa.
- `contain: paint` clips the widget's descendants to its bounds, so the overflowing red box's content is clipped.
- The browser can optimize rendering by limiting layout, style, and paint work to each contained widget.

#### Example 2: `contain: size` for Fixed-Size Elements

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>contain: size</title>
  <style>
    .size-contained {
      contain: size;                    /* Size determined without children */
      width: 200px;
      height: 100px;
      background-color: #c8e6c9;
      border: 2px solid #388e3c;
      padding: 10px;
      margin: 10px;
      overflow: hidden;                 /* Needed if content overflows */
    }

    .no-size-contained {
      width: 200px;
      height: 100px;
      background-color: #ffcdd2;
      border: 2px solid #c62828;
      padding: 10px;
      margin: 10px;
      overflow: hidden;
    }

    .content {
      width: 300px;                     /* Wider than the container */
      height: 150px;                    /* Taller than the container */
      background-color: #fff;
      border: 1px solid #999;
    }
  </style>
</head>
<body>
  <h2>contain: size vs. No contain: size</h2>

  <div class="size-contained">
    <div class="content">Content wider and taller than container</div>
  </div>

  <div class="no-size-contained">
    <div class="content">Content wider and taller than container</div>
  </div>
</body>
</html>
```

**Expected Output:**

Two boxes with the same declared dimensions (200px × 100px). The first box uses `contain: size`, so its size is determined by its own width/height declarations, not by its children. The second box does not use `contain: size`, but since it has explicit `width` and `height`, it also ignores its children's size. Visually, both boxes appear identical.

**Explanation of Results:**

- `contain: size` tells the browser that the element's size can be determined without examining its descendants. The element's declared width and height are used directly.
- Without `contain: size`, the element's declared width and height still override its children's size, but the browser may still need to examine children for other layout calculations.
- In this example, the visual result is the same, but the browser can skip examining the children for sizing purposes with `contain: size`.

### Real-World Cases

- **Dashboard Widgets:** Each widget can be contained with `contain: layout paint` to isolate its internal changes from other widgets.
- **Card Grids:** Applying `contain: content` to each card allows the browser to optimize rendering of individual cards.
- **Infinite Scroll Lists:** `contain: layout paint` on list items reduces the scope of reflow when items are added or removed.
- **Third-Party Embeds:** Containing embedded widgets (e.g., social media embeds, maps) prevents their styles and scripts from affecting the host page's layout.

### References

- MDN Web Docs — contain - https://developer.mozilla.org/en-US/docs/Web/CSS/contain
- CSS Containment Module Level 1 - https://www.w3.org/TR/css-contain-1/
- CSS Containment Module Level 2 - https://drafts.csswg.org/css-contain-2/
- web.dev — CSS Containment in Chrome 52 - https://developer.chrome.com/blog/css-containment


## Core Concept 5: Accessibility Implications — DOM Presence, Screen Reader Visibility, and Keyboard Navigation

### Definitions

**Core Definition:** The accessibility implications of visibility and rendering properties concern how CSS and ARIA attributes affect the accessibility tree, screen reader behavior, keyboard navigation, and focus management.

**Technical Definition:** The accessibility tree is a subset of the DOM tree that assistive technologies (screen readers, switch devices, etc.) use to understand and navigate web content. Elements with `display: none` or `visibility: hidden` are removed from the accessibility tree. Elements with `opacity: 0` remain in the accessibility tree. `display: contents` removes the element from the box tree, and in current browser implementations, also from the accessibility tree (though descendants remain). The `aria-hidden="true"` attribute removes an element and its descendants from the accessibility tree but does not affect focusability; combining it with focusable elements creates a "focus trap" where keyboard users can focus on elements that screen readers cannot perceive.

**Beginner-Friendly Explanation:** Screen readers rely on a special tree (the accessibility tree) to understand your page. Some CSS properties affect this tree in ways that can make content disappear for screen reader users even if it's visible on screen, or vice versa. For example, `display: none` removes content from both sight and screen readers, while `opacity: 0` makes content invisible but still accessible. `aria-hidden="true"` hides content from screen readers but does not stop keyboard users from tabbing to it, creating a confusing situation. Understanding these interactions is essential for building accessible websites.

### Purposes

- To understand how CSS visibility properties affect screen reader access and keyboard navigation
- To avoid creating "focus traps" where keyboard users can focus on elements that screen readers cannot perceive
- To ensure that content hidden visually is also hidden from assistive technologies when appropriate
- To ensure that content hidden from assistive technologies is also hidden from keyboard navigation
- To manage the accessibility tree correctly when using `display: contents` for layout purposes

### Syntax Rules and Structure

**Accessibility Tree Effects:**

| CSS/ARIA | Accessibility Tree | Focusable | Keyboard Navigable |
|----------|-------------------|-----------|-------------------|
| `display: none` | Removed | No | No |
| `visibility: hidden` | Removed | No | No |
| `opacity: 0` | Present | Yes | Yes |
| `display: contents` | Removed (current browsers) | Depends | Depends |
| `aria-hidden="true"` | Removed | Yes (if natively focusable) | Yes |
| `aria-hidden="true"` + `tabindex="-1"` | Removed | No | No |
| `inert` attribute | Removed | No | No |

**Syntax Rules:**

1. `display: none` and `visibility: hidden` both remove the element from the accessibility tree and make it non-focusable.
2. `opacity: 0` does not remove the element from the accessibility tree; it remains focusable and interactive.
3. `display: contents` removes the element's box and, in current browser implementations, removes the element from the accessibility tree. Descendants remain in the accessibility tree.
4. `aria-hidden="true"` removes the element and its descendants from the accessibility tree but does not affect focusability. It should never be used on focusable elements or their ancestors.
5. The `inert` attribute removes the element and its descendants from the accessibility tree and makes them non-focusable, non-clickable, and non-editable. It is the preferred solution for hiding content from both assistive technologies and keyboard navigation.
6. `visibility: hidden` can be overridden by descendants with `visibility: visible`, but the element itself remains hidden from the accessibility tree.

**Constraints and Limitations:**

- `aria-hidden="true"` on a focusable element creates a "focus trap" where keyboard users can focus on something that screen readers cannot announce.
- `display: contents` is "fundamentally broken" in current browsers regarding accessibility; elements with this value are removed from the accessibility tree even though the specification says they should remain.
- `opacity: 0` elements are still focusable and interactive, which can be confusing if they are visually hidden.
- `visibility: hidden` elements are not focusable, but descendants with `visibility: visible` become focusable again.
- Screen readers may handle `display: contents` differently across browsers, making it unreliable for accessibility-sensitive content.

### Multiple Annotated Code Examples

#### Example 1: Focus Trap with `aria-hidden` and Focusable Elements

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>aria-hidden Focus Trap</title>
  <style>
    .modal {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: white;
      padding: 30px;
      border: 3px solid #1976d2;
      box-shadow: 0 0 20px rgba(0,0,0,0.3);
      z-index: 100;
    }

    .overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.5);
      z-index: 50;
    }

    /* Step 1: This class is applied to the rest of the page
       when the modal is open */
    .aria-hidden-page {
      aria-hidden: true;                /* Hides from screen readers */
    }

    /* Step 2: This class removes focusability */
    .non-focusable {
      /* No focusable elements inside */
    }
  </style>
</head>
<body>
  <h2>aria-hidden Focus Trap Demonstration</h2>

  <!-- Step 3: Page content — hidden from screen readers but still focusable -->
  <div class="aria-hidden-page">
    <p>This content is hidden from screen readers.</p>
    <a href="#">Link 1 — still focusable by keyboard!</a>
    <button>Button 1 — still focusable by keyboard!</button>
  </div>

  <!-- Step 4: Modal dialog -->
  <div class="overlay"></div>
  <div class="modal">
    <h3>Modal Dialog</h3>
    <p>This modal is the only content that should be accessible.</p>
    <button>Close</button>
  </div>

  <!-- Step 5: Demonstration of the trap -->
  <p><strong>Warning:</strong> With <code>aria-hidden="true"</code> on the page content
  but no focus management, keyboard users can still Tab to "Link 1" and "Button 1,"
  even though screen readers cannot announce them.</p>
</body>
</html>
```

**Expected Output:**

A modal dialog centered on the screen with a dark overlay. The page content behind the modal is hidden from screen readers (via `aria-hidden="true"`), but keyboard users can still Tab to the links and buttons in the page content. This creates a focus trap: the keyboard user can focus on elements that screen readers cannot perceive.

**Explanation of Results:**

- `aria-hidden="true"` removes the page content from the accessibility tree, so screen readers do not announce it.
- However, `aria-hidden` does not affect focusability. The links and buttons remain in the tab order, so keyboard users can focus on them.
- This creates an inconsistent experience: keyboard users can interact with content that screen reader users cannot perceive.
- The correct solution is to use the `inert` attribute (which removes both accessibility and focusability) or to manually set `tabindex="-1"` on all focusable elements.

#### Example 2: `display: contents` Accessibility Bug

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>display: contents Accessibility</title>
  <style>
    /* Step 1: Apply display: contents to a semantic element */
    .contents-demo {
      display: contents;                /* Removes the box and accessibility node */
    }

    .contents-demo h3 {
      color: #1565c0;
      font-size: 1.3em;
    }

    .contents-demo p {
      color: #333;
      line-height: 1.6;
    }

    /* Step 2: Compare with a normal semantic element */
    .normal-demo {
      /* No display: contents — element remains in accessibility tree */
      border: 1px solid #ccc;
      padding: 10px;
      margin: 10px 0;
    }

    .normal-demo h3 {
      color: #1565c0;
      font-size: 1.3em;
    }

    .normal-demo p {
      color: #333;
      line-height: 1.6;
    }
  </style>
</head>
<body>
  <h2>display: contents — Accessibility Impact</h2>

  <div class="normal-demo">
    <h3>Normal Article</h3>
    <p>This article element is in the accessibility tree. Screen readers can
    navigate to it as a landmark region.</p>
  </div>

  <article class="contents-demo">
    <h3>Article with display: contents</h3>
    <p>This article element has <code>display: contents</code>. Its box is
    removed, and in current browsers, it is also removed from the accessibility
    tree. Screen readers cannot navigate to the article landmark, though the
    heading and paragraph inside are still accessible.</p>
  </article>
</body>
</html>
```

**Expected Output:**

Two visually similar sections. The first uses a normal `<article>` element with a border, and screen readers can navigate to it as an article landmark. The second uses `<article>` with `display: contents`, so its border is not rendered (the children are hoisted up), and screen readers cannot navigate to the article landmark because the element has been removed from the accessibility tree.

**Explanation of Results:**

- `display: contents` removes the element's box from the layout, causing its children to be laid out as if they were direct children of the parent.
- In current browser implementations, the element is also removed from the accessibility tree. The heading and paragraph inside remain in the accessibility tree, but the `<article>` landmark itself is lost.
- This is considered a bug in browser implementations; the specification says the element should remain in the accessibility tree, but browsers do not yet implement this correctly.

### Real-World Cases

- **Modal Dialogs:** When a modal is open, the rest of the page should be hidden from both screen readers and keyboard navigation. Using `inert` on the background content is the recommended approach; using `aria-hidden` alone creates a focus trap.
- **Responsive Navigation:** Hamburger menus that are hidden with `display: none` are also removed from the accessibility tree and keyboard navigation. When shown, they are accessible again. This is correct behavior.
- **`display: contents` in Grid Layouts:** Using `display: contents` on wrapper elements in grid layouts can improve layout flexibility but may break accessibility if the wrapper is a semantic landmark. Developers should be cautious and test with screen readers.
- **`opacity: 0` for Animations:** Elements that fade in using `opacity` remain in the accessibility tree and focusable throughout the animation, which may be undesirable if the element is initially hidden and should not be interactive.

### References

- MDN Web Docs — ARIA: aria-hidden attribute - https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden
- MDN Web Docs — display (display: contents accessibility warning) - https://developer.mozilla.org/en-US/docs/Web/CSS/display
- W3C — Element with aria-hidden has no content in sequential focus navigation - https://www.w3.org/WAI/standards-guidelines/act/rules/6cfa84/
- W3C CSS Working Group — display: contents Accessibility Warning - https://lists.w3.org/Archives/Public/www-style/2019May/0001.html
- MDN Web Docs — inert attribute - https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert


## Consolidated References

- MDN Web Docs — display - https://developer.mozilla.org/en-US/docs/Web/CSS/display
- MDN Web Docs — visibility - https://developer.mozilla.org/en-US/docs/Web/CSS/visibility
- MDN Web Docs — opacity - https://developer.mozilla.org/en-US/docs/Web/CSS/opacity
- MDN Web Docs — content-visibility - https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility
- MDN Web Docs — will-change - https://developer.mozilla.org/en-US/docs/Web/CSS/will-change
- MDN Web Docs — contain - https://developer.mozilla.org/en-US/docs/Web/CSS/contain
- MDN Web Docs — ARIA: aria-hidden attribute - https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden
- MDN Web Docs — Element.checkVisibility() - https://developer.mozilla.org/en-US/docs/Web/API/Element/checkVisibility
- MDN Web Docs — CSS performance optimization - https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/CSS
- CSS Display Module Level 3 - https://drafts.csswg.org/css-display/
- CSS Containment Module Level 1 - https://www.w3.org/TR/css-contain-1/
- CSS Containment Module Level 2 - https://drafts.csswg.org/css-contain-2/
- web.dev — content-visibility: the new CSS property that boosts your rendering performance - https://web.dev/articles/content-visibility
- web.dev — CSS content-visibility is now Baseline - https://web.dev/blog/css-content-visibility-baseline
- web.dev — Rendering performance - https://web.dev/articles/rendering-performance
- Chrome for Developers — CSS Containment in Chrome 52 - https://developer.chrome.com/blog/css-containment
- W3C — Element with aria-hidden has no content in sequential focus navigation - https://www.w3.org/WAI/standards-guidelines/act/rules/6cfa84/
- W3C CSS Working Group — display: contents Accessibility Warning - https://lists.w3.org/Archives/Public/www-style/2019May/0001.html
- Google Developers — Minimizing browser reflow - https://developers.google.com/speed/docs/insights/browser-reflow