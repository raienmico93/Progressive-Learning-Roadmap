# jQuery Position Manipulation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Position Manipulation is the set of methods for reading and setting an element's coordinates relative to the document or a positioned ancestor, identifying that ancestor, and controlling the scroll position of scrollable elements.

**Technical Definition:**
jQuery provides five position-related methods. `.offset()` retrieves or sets an element's **border-box** coordinates relative to the **document**. `.position()` retrieves an element's **margin-box** coordinates relative to its **offset parent's padding box**. `.offsetParent()` returns the closest positioned ancestor that defines the coordinate system for `.position()`. `.scrollLeft()` and `.scrollTop()` get or set the horizontal and vertical scroll positions of an element. These methods abstract native `getBoundingClientRect()`, `offsetTop`/`offsetLeft`, and `scrollLeft`/`scrollTop`, normalizing browser inconsistencies and returning unitless numeric values.

**Beginner-Friendly Explanation:**
Position manipulation answers the question "where is this element?" jQuery gives you different ways to ask that question. `.offset()` tells you where an element is on the whole page. `.position()` tells you where it is inside its nearest positioned container. `.offsetParent()` tells you which container that is. And `.scrollTop()` / `.scrollLeft()` tell you how far a scrollable area has been scrolled.

### Key Characteristics

- **Two Coordinate Systems:** `.offset()` is document-relative; `.position()` is offset-parent-relative. Choosing the wrong method produces incorrect coordinates.
- **Box-Model Precision:** `.offset()` measures the **border box** (excluding margins); `.position()` measures the **margin box** relative to the offset parent's **padding box** (excluding the offset parent's margins and borders).
- **Getter/Setter Duality:** `.offset()`, `.scrollTop()`, and `.scrollLeft()` all have getter and setter forms. `.position()` and `.offsetParent()` are getters only.
- **Hidden Element Limitation:** jQuery does not support getting the offset or position coordinates of elements with `display: none`, because they are removed from the rendering tree.
- **Fractional Values:** Coordinates returned by `.offset()` and `.position()` may be fractional; code should not assume integers.
- **Zoom Inconsistency:** Dimensions and coordinates may be incorrect when the page is zoomed; browsers do not expose an API to detect this condition.
- **Scroll Clamping:** Setting a scroll position beyond the scrollable range is clamped by the browser; the getter returns `0` if the element is not scrollable.

### Prerequisites

- Basic knowledge of HTML and CSS positioning (`position: relative`, `absolute`, `fixed`)
- Understanding of the CSS box model (content, padding, border, margin)
- Familiarity with jQuery selectors and chainability
- Awareness of document flow versus positioned elements

### Related Programming Areas

- **CSS Positioning:** The `position` property creates offset parents and defines the coordinate system for `.position()`.
- **Scroll Events:** `.scrollTop()` and `.scrollLeft()` are frequently used inside `scroll` handlers.
- **Drag-and-Drop:** `.offset()` is essential for tracking pointer coordinates relative to the document.
- **Sticky Elements:** Scroll position combined with offset coordinates drives sticky header logic.
- **Performance Optimization:** Position reads trigger layout calculations and must be batched to avoid reflows.

### Core Concepts / Features

1. `.offset()`
2. `.position()`
3. `.offsetParent()`
4. `.scrollLeft()`
5. `.scrollTop()`
6. Position Manipulation Best Practices

> **Note:** The user's list repeats `.scrollLeft()` and `.scrollTop()`. They are the same methods and are covered once each below.

---

## 1. `.offset()`

### Definitions

**Core Definition:**
`.offset()` gets the current document-relative coordinates of the first matched element or sets the document-relative coordinates of every matched element.

**Technical Definition:**
`.offset()` returns an object with `top` and `left` properties representing the element's **border box** position relative to the document (excluding margins). The setter form `.offset(coordinates)` accepts an object with `top` and `left` numeric values (pixels) and repositions every matched element relative to the document. As of jQuery 1.4, the setter also accepts a function returning the coordinates. The method does not support hidden elements (`display: none`).

**Beginner-Friendly Explanation:**
`.offset()` tells you where an element is on the entire page, counting from the top-left corner of the document. If you set `.offset({ top: 100, left: 50 })`, the element moves to that position on the page.

### Purposes

- To retrieve an element's position relative to the document for drag-and-drop or overlay positioning.
- To move an element to a specific document coordinate.
- To calculate the distance between two elements on the page.
- To implement "scroll to element" behavior by combining `.offset().top` with `.scrollTop()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
// Getter — document-relative coordinates of first element
var coords = $(selector).offset();
console.log(coords.top, coords.left);

// Setter — move all elements to document coordinates
$(selector).offset({ top: 100, left: 50 });

// Setter with function (jQuery 1.4+)
$(selector).offset(function(index, currentOffset) {
    return { top: currentOffset.top + 10, left: currentOffset.left };
});
```

**Component Breakdown:**

| Form | Returns | Scope |
|------|---------|-------|
| `.offset()` | `{ top, left }` object | First element only. |
| `.offset({top, left})` | jQuery object | All elements (setter). |
| `.offset(fn)` | jQuery object | All elements (computed setter). |

**Syntax Rules:**

1. The getter returns an object with numeric `top` and `left` properties (pixels).
2. The setter accepts numbers without units; do not include `"px"`.
3. The setter affects every element in the collection, moving them all to the same coordinates.
4. For hidden elements (`display: none`), the position is `undefined` and the getter may fail.

**Constraints and Limitations:**

- Does not account for margins set on the `<html>` document element.
- Returned values may be fractional; do not assume integers.
- Does not work reliably when the page is zoomed by the user.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading and Setting Offset Coordinates**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.offset() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { position: absolute; width: 100px; height: 100px; background: #eee; }
    </style>
</head>
<body>
    <div id="box">Box</div>

    <script>
        // Step 1: Get the initial offset
        var initial = $("#box").offset();
        console.log("Initial top: " + initial.top);   // 0
        console.log("Initial left: " + initial.left); // 0

        // Step 2: Set the offset to document coordinates
        $("#box").offset({ top: 100, left: 200 });

        // Step 3: Verify the new position
        var updated = $("#box").offset();
        console.log("Updated top: " + updated.top);   // 100
        console.log("Updated left: " + updated.left); // 200
    </script>
</body>
</html>
```

**Expected Output:**
```
Initial top: 0
Initial left: 0
Updated top: 100
Updated left: 200
```

**Why This Code Produces That Result:**
The `#box` element is absolutely positioned at the document origin initially, so `.offset()` returns `{ top: 0, left: 0 }`. The setter moves the element to `top: 100` and `left: 200` relative to the document. The subsequent getter confirms the new coordinates. This demonstrates the document-relative nature of `.offset()`.

### Real-World Cases

- **Drag-and-Drop:** Tracking pointer coordinates relative to the document to position a dragged element.
- **Scroll-to-Section Navigation:** Computing `$(target).offset().top` and animating `$("html, body").scrollTop()` to that value.
- **Tooltip Placement:** Positioning a tooltip near a target element using its document coordinates.
- **Overlay Positioning:** Placing a modal or overlay at the document coordinates of a triggering element.

---

## 2. `.position()`

### Definitions

**Core Definition:**
`.position()` gets the current coordinates of the first matched element relative to its offset parent.

**Technical Definition:**
`.position()` returns an object with `top` and `left` properties representing the element's **margin box** position relative to the **padding box** of its offset parent (excluding the offset parent's margins and borders). It is a getter-only method. Contrast with `.offset()`, which is document-relative. When positioning a new element near another one within the same containing DOM element, `.position()` is more useful.

**Beginner-Friendly Explanation:**
`.position()` tells you where an element is **inside its positioned container**. If a `<div>` has `position: relative` and contains an absolutely positioned child, `.position()` tells you where that child sits relative to the `<div>`.

### Purposes

- To retrieve an element's coordinates relative to its offset parent for calculations within a positioned container.
- To position a new element near an existing element that shares the same offset parent.
- To determine the internal layout offset of a child within a positioned parent.
- To implement drag-and-drop within a bounded container.

### Syntax Rules and Structure

```javascript
var pos = $(selector).position();
console.log(pos.top, pos.left);
```

**Component Breakdown:**

| Form | Returns | Scope |
|------|---------|-------|
| `.position()` | `{ top, left }` object | First element only. |

**Syntax Rules:**

1. Returns the element's margin-box position relative to the offset parent's padding box.
2. The offset parent is determined by `.offsetParent()`.
3. Returns `undefined` for hidden elements (`display: none`).
4. Does not account for margins on the `<html>` document element.

**Constraints and Limitations:**

- Getter only; cannot set position with this method (use `.css("top", ...)` or `.offset()`).
- Values may be fractional.
- Depends on the element being positioned (`position: relative`, `absolute`, or `fixed`); otherwise, the position may be relative to the initial containing block.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Position Relative to Offset Parent**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.position() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #parent { position: relative; margin: 50px; padding: 20px; border: 1px solid #ccc; }
        #child { position: absolute; top: 10px; left: 20px; }
    </style>
</head>
<body>
    <div id="parent">
        Parent
        <div id="child">Child</div>
    </div>

    <script>
        // Step 1: Get position relative to the offset parent
        var pos = $("#child").position();
        console.log("Position top: " + pos.top);   // 10
        console.log("Position left: " + pos.left); // 20

        // Step 2: Contrast with offset (document-relative)
        var off = $("#child").offset();
        console.log("Offset top: " + off.top);   // includes parent margin/padding
        console.log("Offset left: " + off.left); // includes parent margin/padding
    </script>
</body>
</html>
```

**Expected Output:**
```
Position top: 10
Position left: 20
Offset top: [document-relative value > 10]
Offset left: [document-relative value > 20]
```

**Why This Code Produces That Result:**
The `#parent` is positioned (`position: relative`), so it becomes the offset parent for `#child`. The child is absolutely positioned at `top: 10` and `left: 20` relative to the parent's padding box. `.position()` returns exactly those values. `.offset()` returns larger values because it includes the parent's margin, padding, and border offsets relative to the document.

### Real-World Cases

- **Drag-and-Drop Within a Container:** Tracking an item's position inside a positioned list or canvas.
- **Tooltip Placement Within a Modal:** Positioning a tooltip relative to a field inside a modal container.
- **Resizable Panels:** Calculating the relative position of a resize handle within a panel.
- **Sortable Lists:** Determining the position of a dragged list item relative to the list container.

---

## 3. `.offsetParent()`

### Definitions

**Core Definition:**
`.offsetParent()` returns a jQuery object containing the closest positioned ancestor of each element in the set.

**Technical Definition:**
`.offsetParent()` traverses the DOM tree upward from each matched element and returns the nearest ancestor that is **positioned** (`position: relative`, `absolute`, `fixed`, or `sticky`, depending on browser behavior). This is the element that defines the coordinate system for `.position()`. If no positioned ancestor exists, the offset parent is typically the `<body>` or `<html>` element, depending on the browser and the element's own positioning.

**Beginner-Friendly Explanation:**
`.offsetParent()` tells you which container `.position()` is measuring against. It is the nearest parent that has a `position` other than `static`. This helps you understand where the coordinates from `.position()` are coming from.

### Purposes

- To identify the element that defines the coordinate system for `.position()`.
- To calculate offsets relative to the correct container.
- To debug why `.position()` returns unexpected values.
- To traverse to the positioning context for layout calculations.

### Syntax Rules and Structure

```javascript
var $parent = $(selector).offsetParent();
console.log($parent.prop("tagName"));
```

**Component Breakdown:**

| Form | Returns | Scope |
|------|---------|-------|
| `.offsetParent()` | jQuery object | Closest positioned ancestor for each element. |

**Syntax Rules:**

1. Returns a jQuery object wrapping the closest positioned ancestor.
2. For hidden elements, the offset parent behavior may differ.
3. If no positioned ancestor exists, returns `body` or `html` depending on the browser.

**Constraints and Limitations:**

- The definition of "positioned" can vary slightly across browsers for `position: fixed` and `position: sticky`.
- Getter only.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Identifying the Offset Parent**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.offsetParent() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #outer { position: relative; padding: 20px; border: 1px solid #333; }
        #middle { padding: 20px; }
        #inner { position: absolute; top: 10px; left: 10px; }
    </style>
</head>
<body>
    <div id="outer">
        <div id="middle">
            <div id="inner">Inner</div>
        </div>
    </div>

    <script>
        // Step 1: Find the offset parent of #inner
        var $parent = $("#inner").offsetParent();
        console.log("Offset parent ID: " + $parent.attr("id")); // outer

        // Step 2: Verify with .position()
        var pos = $("#inner").position();
        console.log("Position top: " + pos.top);   // 10
        console.log("Position left: " + pos.left); // 10
    </script>
</body>
</html>
```

**Expected Output:**
```
Offset parent ID: outer
Position top: 10
Position left: 10
```

**Why This Code Produces That Result:**
`#outer` has `position: relative`, making it the closest positioned ancestor of `#inner`. `#middle` is not positioned, so it is skipped. `.offsetParent()` returns `#outer`, and `.position()` measures `#inner`'s coordinates relative to `#outer`'s padding box.

### Real-World Cases

- **Debugging Layout Issues:** Identifying why `.position()` returns unexpected values.
- **Dynamic Positioning:** Finding the correct container to position a new element within.
- **Plugin Development:** Determining the coordinate system for widget placement.

---

## 4. `.scrollLeft()`

### Definitions

**Core Definition:**
`.scrollLeft()` gets the current horizontal scroll position of the first matched element or sets the horizontal scroll position of every matched element.

**Technical Definition:**
`.scrollLeft()` returns the number of pixels that the content of an element is scrolled to the left (i.e., the number of pixels hidden from view to the left of the scrollable area). If the element is not scrollable or the scroll bar is at the far left, it returns `0`. The setter form `.scrollLeft(value)` sets the horizontal scroll position for every matched element. The method works on scrollable elements and on `window`/`document`. When called on a hidden element, `.scrollLeft()` does not work.

**Beginner-Friendly Explanation:**
`.scrollLeft()` tells you how far a scrollable area has been scrolled to the right. If you set `.scrollLeft(100)`, the content moves 100 pixels to the left, revealing content that was off-screen.

### Purposes

- To read the horizontal scroll position of a container for lazy loading or infinite scrolling.
- To programmatically scroll a container to a specific horizontal position.
- To synchronize scroll positions between multiple elements.
- To implement horizontal carousels or sliders.

### Syntax Rules and Structure

```javascript
// Getter — horizontal scroll position of first element
var left = $(selector).scrollLeft();

// Setter — horizontal scroll position of all elements
$(selector).scrollLeft(200);
```

**Component Breakdown:**

| Form | Returns | Scope |
|------|---------|-------|
| `.scrollLeft()` | Integer (pixels) | First element only. |
| `.scrollLeft(value)` | jQuery object | All elements (setter). |

**Syntax Rules:**

1. Returns `0` if the element is not scrollable or is at the far left.
2. The setter accepts a number (pixels).
3. Does not work on hidden elements (`display: none`).
4. Works on `window` and `document` for page-level horizontal scrolling.

**Constraints and Limitations:**

- Setting a value beyond the maximum scroll range is clamped by the browser.
- The getter may return different values across browsers for `window`; use `$(window).scrollLeft()` for consistency.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading and Setting Horizontal Scroll**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.scrollLeft() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #scroller { width: 200px; overflow-x: scroll; white-space: nowrap; border: 1px solid #ccc; }
        .item { display: inline-block; width: 100px; height: 50px; background: #eee; margin: 5px; }
    </style>
</head>
<body>
    <div id="scroller">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>

    <script>
        // Step 1: Initial scroll position
        console.log("Initial: " + $("#scroller").scrollLeft()); // 0

        // Step 2: Set scroll position
        $("#scroller").scrollLeft(150);
        console.log("After set: " + $("#scroller").scrollLeft()); // 150
    </script>
</body>
</html>
```

**Expected Output:**
```
Initial: 0
After set: 150
```

**Why This Code Produces That Result:**
The container has `overflow-x: scroll` and content wider than the container, making it horizontally scrollable. The initial scroll position is `0`. Setting `.scrollLeft(150)` scrolls the content 150 pixels to the left, and the getter confirms the new position.

### Real-World Cases

- **Horizontal Carousels:** Scrolling to a specific slide or item.
- **Infinite Scrolling (Horizontal):** Detecting when the user has scrolled near the right edge to load more content.
- **Synchronized Scroll Panes:** Keeping two horizontally scrollable panes in sync.

---

## 5. `.scrollTop()`

### Definitions

**Core Definition:**
`.scrollTop()` gets the current vertical scroll position of the first matched element or sets the vertical scroll position of every matched element.

**Technical Definition:**
`.scrollTop()` returns the number of pixels that the content of an element is scrolled upward (i.e., the number of pixels hidden from view above the scrollable area). If the element is not scrollable or the scroll bar is at the very top, it returns `0`. The setter form `.scrollTop(value)` sets the vertical scroll position. The method works on scrollable elements and on `window`/`document`. Setting `scrollTop` positions the vertical scroll of each matched element.

**Beginner-Friendly Explanation:**
`.scrollTop()` tells you how far down a scrollable area has been scrolled. If you set `.scrollTop(0)`, it scrolls back to the top. If you set `.scrollTop(500)`, it scrolls down 500 pixels.

### Purposes

- To read the vertical scroll position for sticky headers, scroll animations, or lazy loading.
- To programmatically scroll to the top or to a specific vertical position.
- To implement "scroll to element" behavior using `.offset().top`.
- To synchronize scroll positions between multiple elements.

### Syntax Rules and Structure

```javascript
// Getter — vertical scroll position of first element
var top = $(selector).scrollTop();

// Setter — vertical scroll position of all elements
$(selector).scrollTop(300);

// Scroll window to top
$(window).scrollTop(0);
```

**Component Breakdown:**

| Form | Returns | Scope |
|------|---------|-------|
| `.scrollTop()` | Number (pixels) | First element only. |
| `.scrollTop(value)` | jQuery object | All elements (setter). |

**Syntax Rules:**

1. Returns `0` if the element is not scrollable or is at the top.
2. The setter accepts a number (pixels).
3. Works on `window` and `document` for page-level vertical scrolling.
4. Setting a value beyond the scrollable range is clamped.

**Constraints and Limitations:**

- Values may be fractional on high-DPI displays.
- Does not work on hidden elements.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading and Setting Vertical Scroll**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.scrollTop() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #scroller { height: 100px; overflow-y: scroll; border: 1px solid #ccc; }
        .item { height: 60px; background: #eee; margin: 5px; }
    </style>
</head>
<body>
    <div id="scroller">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>

    <script>
        // Step 1: Initial scroll position
        console.log("Initial: " + $("#scroller").scrollTop()); // 0

        // Step 2: Set scroll position
        $("#scroller").scrollTop(100);
        console.log("After set: " + $("#scroller").scrollTop()); // 100

        // Step 3: Scroll window to top
        $(window).scrollTop(0);
        console.log("Window top: " + $(window).scrollTop()); // 0
    </script>
</body>
</html>
```

**Expected Output:**
```
Initial: 0
After set: 100
Window top: 0
```

**Why This Code Produces That Result:**
The container has `overflow-y: scroll` and content taller than the container. Setting `.scrollTop(100)` scrolls the content down by 100 pixels. The window scroll is set to `0`, returning the page to the top.

### Real-World Cases

- **Sticky Headers:** Detecting when `$(window).scrollTop()` exceeds a threshold to fix a header to the top.
- **Scroll-to-Top Buttons:** Smoothly animating `scrollTop` to `0`.
- **Scroll-to-Element:** Combining `$(target).offset().top` with `$("html, body").scrollTop(...)`.
- **Lazy Loading:** Loading content when the scroll position approaches the bottom of a container.

---

## 6. Position Manipulation Best Practices

### Definitions

**Core Definition:**
Position manipulation best practices are guidelines for reading and setting element coordinates and scroll positions in a performant, accurate, and maintainable manner.

**Technical Definition:**
Best practices encompass choosing the correct coordinate system (`.offset()` vs. `.position()`), caching jQuery objects and coordinate values to avoid repeated layout calculations, batching reads and writes to prevent forced synchronous layouts (reflows), throttling scroll handlers, and accounting for edge cases such as hidden elements, fractional values, and browser zoom.

**Beginner-Friendly Explanation:**
These are the rules that keep your position code fast and accurate. They help you avoid measuring the same thing repeatedly, accidentally slowing down the page, or getting wrong numbers when elements are hidden.

### Purposes

- To choose the correct method for the coordinate system needed.
- To minimize layout thrashing and reflow costs.
- To handle edge cases (hidden elements, zoom, fractional values) correctly.
- To write position code that is easy to understand and maintain.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Use `.offset()` for document coordinates | `.position()` for page-relative placement | `.offset()` |
| 2 | Use `.position()` for container-relative coordinates | `.offset()` minus parent offset manually | `.position()` |
| 3 | Cache jQuery objects | `$(this).offset()` repeatedly | `var $el = $(this); $el.offset();` |
| 4 | Batch reads and writes | Read offset → write CSS → read offset | All reads, then all writes |
| 5 | Throttle scroll handlers | `.on("scroll", handler)` on every pixel | Throttle with `requestAnimationFrame` or a timer |
| 6 | Check for hidden elements | Assume `.offset()` works on `display: none` | Show, measure, hide, or use `visibility: hidden` |
| 7 | Use `window`/`document` consistently | Mixing native `pageYOffset` and jQuery | Use `$(window).scrollTop()` |

**Syntax Rules:**

1. Use `.offset()` when the position must be relative to the whole document.
2. Use `.position()` when the position must be relative to a positioned container.
3. Cache coordinate values when they are used more than once in a function.
4. Separate read operations from write operations to avoid forced reflows.
5. Throttle scroll and resize handlers that read scroll positions.

**Constraints and Limitations:**

- No best practice eliminates the browser zoom limitation.
- Hidden elements (`display: none`) cannot be measured; a temporary show-measure-hide pattern triggers reflows.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Position Best Practices**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Position Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #target { position: absolute; top: 100px; left: 200px; width: 50px; height: 50px; background: blue; }
    </style>
</head>
<body>
    <div id="target">Target</div>

    <script>
        // ✅ Cache the jQuery object
        var $target = $("#target");

        // ✅ Batch reads before writes
        var offset = $target.offset();        // read
        var pos = $target.position();          // read
        var scrollTop = $(window).scrollTop(); // read

        // ✅ Write after all reads
        $target.css("border", "2px solid red");

        console.log("Offset top: " + offset.top);
        console.log("Position top: " + pos.top);
        console.log("Scroll top: " + scrollTop);

        // ✅ Throttled scroll handler (conceptual pattern)
        var ticking = false;
        $(window).on("scroll", function() {
            if (!ticking) {
                requestAnimationFrame(function() {
                    var st = $(window).scrollTop();
                    // Do work with st
                    ticking = false;
                });
                ticking = true;
            }
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
Offset top: 100
Position top: 100
Scroll top: 0
```

**Why This Code Produces That Result:**
The jQuery object is cached once. All coordinate reads are performed before any writes, avoiding layout thrashing. The scroll handler is throttled with `requestAnimationFrame`, limiting the frequency of scroll position reads. This demonstrates the core performance best practices for position manipulation.

### Real-World Cases

- **Sticky Headers:** Caching the header's offset and throttling the scroll handler.
- **Drag-and-Drop:** Caching the container's offset parent and batching position reads during drag.
- **Infinite Scroll:** Throttling `.scrollTop()` reads to check scroll thresholds.
- **Scroll Animations:** Caching target offsets and using `requestAnimationFrame` for smooth animations.

---

## References

- jQuery API Documentation — `.offset()` - https://api.jquery.com/offset/
- jQuery API Documentation — `.position()` - https://api.jquery.com/position/
- jQuery API Documentation — `.offsetParent()` - https://api.jquery.com/offsetParent/
- jQuery API Documentation — `.scrollLeft()` - https://api.jquery.com/scrollLeft/
- jQuery API Documentation — `.scrollTop()` - https://api.jquery.com/scrollTop/
- jQuery API Documentation — Offset Category - https://api.jquery.com/category/offset/
- jQuery API Documentation — CSS Category - https://api.jquery.com/category/css/
- Stack Overflow — jQuery offset performance and caching - https://stackoverflow.com/revisions/7c9bdc8a-b407-43b8-9b90-0b59d4ef9755/view-source