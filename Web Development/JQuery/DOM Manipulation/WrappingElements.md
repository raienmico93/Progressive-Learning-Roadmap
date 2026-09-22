# jQuery Wrapping Elements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery wrapping methods are a family of DOM manipulation functions that insert a new HTML structure — a "wrapper" — around an element, around all elements as a group, around an element's inner content, or that remove an existing wrapper from the DOM.

**Technical Definition**
Wrapping methods belong to jQuery's **DOM Insertion, Around** category. They operate by creating a new DOM structure (specified as a selector, HTML string, DOM element, or jQuery object) and inserting it into the DOM at a position relative to the matched elements. `.wrap()` creates a **copy** of the wrapper structure for each matched element and wraps each element individually. `.wrapAll()` inserts a single wrapper structure around all matched elements as one group. `.wrapInner()` wraps the **contents** (child nodes, including text) of each matched element, leaving the element itself outside the wrapper. `.unwrap()` is the inverse operation: it removes the **parent** of each matched element, promoting the element (and its siblings) to the parent's position in the DOM tree.

**Beginner-Friendly Explanation**
Sometimes you need to put a box around something on a web page — not just any box, but a specific element that provides styling, structure, or behaviour. jQuery's wrapping methods let you do exactly that. `.wrap()` puts a box around each item individually. `.wrapAll()` puts one big box around all of them together. `.wrapInner()` puts a box around just the contents of an element, leaving the element itself outside. And `.unwrap()` takes a box away, freeing whatever was inside it. Think of it like gift wrapping: you can wrap each gift separately, wrap all the gifts in one big box, wrap just the tissue paper inside a gift, or unwrap a gift to get to what is inside.

### Key Characteristics

- **Structural Manipulation**: Wrapping methods do not change the content of elements; they change the DOM hierarchy by adding or removing parent elements.
- **Copy vs. Move**: `.wrap()` and `.wrapAll()` create copies of the wrapper structure when the wrapper is specified as a selector or jQuery object; `.wrapAll()` moves the matched elements into a single wrapper.
- **Inner vs. Outer**: `.wrapInner()` wraps the contents (inner HTML) of an element, while `.wrap()` wraps the element itself.
- **Inverse Operation**: `.unwrap()` is the inverse of `.wrap()` — it removes the parent element and promotes its children.
- **Chainable**: All methods return a jQuery object, preserving method chaining.
- **Callback Support**: Since jQuery 1.4, all three wrapping methods accept a callback function that returns the wrapper structure to use for each element.
- **Selector Filtering**: `.unwrap()` accepts an optional selector (since jQuery 3.0) to only unwrap parents that match the selector.

### Prerequisites

- Basic understanding of the DOM tree and parent-child relationships.
- Familiarity with jQuery selectors and the `$()` function.
- Awareness of how HTML structure affects CSS styling and layout.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **DOM Manipulation**: Wrapping is a core part of restructuring the DOM dynamically.
- **CSS Styling**: Wrappers often provide styling hooks (classes, IDs) for layout and design.
- **Layout Systems**: Grid and flexbox layouts often require specific wrapper structures that jQuery can create dynamically.
- **Plugin Development**: Plugins frequently wrap elements to create consistent structural contexts.
- **Accessibility**: Wrappers can add semantic meaning or ARIA roles to groups of elements.

### Core Concepts / Features

1. `.wrap()` — Wrap Each Element Individually
2. `.unwrap()` — Remove the Parent Element
3. `.wrapAll()` — Wrap All Elements as a Group
4. `.wrapInner()` — Wrap the Inner Content of Each Element

---

## Core Concept 1: `.wrap()` — Wrap Each Element Individually

### Definitions

**Core Definition**
`.wrap()` wraps an HTML structure around **each element** in the set of matched elements individually, creating a separate wrapper for every element.

**Technical Definition**
The `.wrap()` method takes any string or object that could be passed to the `$()` factory function to specify a DOM structure. This structure may be nested several levels deep, but should contain only one inmost element. A **copy** of this structure is wrapped around each element in the set. When a jQuery collection or selector matching more than one element is passed as the wrapper, the first element is used. As of jQuery 1.4, a callback function can be used to generate the wrapper dynamically for each element. The method returns the **original set** of elements for chaining purposes.

**Beginner-Friendly Explanation**
`.wrap()` puts a box around each item separately. If you select three paragraphs and call `.wrap("<div class='box'></div>")`, each paragraph gets its **own** `<div class="box">` around it. This is different from `.wrapAll()`, which would put **one** box around all three paragraphs together.

### Purposes

- To wrap each matched element in its own individual wrapper structure.
- To add a consistent structural container around a group of similar elements for styling or layout.
- To group elements for CSS grid, flexbox, or other layout systems that require wrapper elements.
- To add semantic or functional containers around elements without modifying the elements themselves.
- To prepare elements for plugins or libraries that expect a specific DOM structure.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).wrap(wrappingElement);
$(selector).wrap(function(index) { ... });
```

**Component Breakdown**

- `$(selector)` : The element(s) to be wrapped individually.
- `wrappingElement` : A selector, element, HTML string, or jQuery object specifying the structure to wrap around each matched element. The structure may be nested but should contain only one inmost element. When a collection with multiple elements is passed, only the first is used.
- `function(index)` : A callback function returning the HTML content or jQuery object to wrap around the matched elements. Receives the index position of the element in the set as an argument. `this` refers to the current element.
- Returns: A jQuery object containing the original set of elements (for chaining).

**Syntax Rules**

1. A **copy** of the wrapper structure is created for each matched element.
2. The wrapper structure may be nested several levels deep but should contain only one inmost element.
3. When a jQuery collection or selector matching multiple elements is passed, the first element is used.
4. As of jQuery 1.4, the callback function receives the index and has `this` bound to the current element.
5. The method returns the original set of elements, not the wrapper.

**Constraints and Limitations**

- The wrapper structure should contain only one inmost element; complex structures may produce unexpected results.
- `.wrap()` creates a separate copy of the wrapper for each element; this can be memory-intensive for large sets.
- If the wrapper is an existing DOM element, it is cloned for each matched element (except possibly the last).

### Multiple Annotated Complete Code Examples

**Example 1: Wrapping Each Element Individually**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.wrap() — Individual Wrapping</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .wrapper { border: 2px solid blue; padding: 10px; margin: 5px; }
    .inner { background: lightyellow; padding: 5px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Wrap each .inner div with a new wrapper div
      $(".inner").wrap("<div class='wrapper'></div>");

      // Step 2: Verify each inner div has its own wrapper
      console.log("Wrapper count:", $(".wrapper").length); // 2
      console.log("Inner count:", $(".inner").length); // 2

      // Step 3: Verify structure — each wrapper contains one inner div
      $(".wrapper").each(function (i) {
        console.log("Wrapper " + i + " children:", $(this).children().length); // 1
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Each `.inner` div receives its own blue-bordered wrapper.
- Console output:
```
Wrapper count: 2
Inner count: 2
Wrapper 0 children: 1
Wrapper 1 children: 1
```

**Why This Output Occurs**
`.wrap("<div class='wrapper'></div>")` creates a separate copy of the wrapper div for each `.inner` element. Each wrapper contains exactly one inner div.

---

**Example 2: Wrapping with a Callback Function**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.wrap() — Callback Function</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Wrap each .inner div with a class derived from its text content
      $(".inner").wrap(function () {
        // 'this' is the current DOM element
        return "<div class='" + $(this).text().toLowerCase() + "'></div>";
      });

      // Verify the wrappers
      console.log("Hello wrapper exists:", $(".hello").length); // 1
      console.log("Goodbye wrapper exists:", $(".goodbye").length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Hello wrapper exists: 1
Goodbye wrapper exists: 1
```

**Why This Output Occurs**
The callback function is called once for each `.inner` element. It reads the element's text content, converts it to lowercase, and returns an HTML string that becomes the wrapper. This produces unique wrappers for each element.

### Real-World Cases

- **CSS Grid Wrappers**: `$(".grid-item").wrap("<div class='grid-cell'></div>")` to create grid cells around items.
- **Link Wrapping**: `$("img.thumbnail").wrap("<a href='#' class='lightbox'></a>")` to make images clickable.
- **Form Field Groups**: `$("input[type=text]").wrap("<div class='form-group'></div>")` to create Bootstrap-style form groups.
- **Table Cell Wrapping**: `$("td").wrapInner("<div class='cell-inner'></div>")` to add inner containers for styling.

### References

- jQuery API — .wrap() – https://api.jquery.com/wrap/
- jQuery API — DOM Insertion, Around – https://api.jquery.com/category/manipulation/dom-insertion-around/
- W3Schools — jQuery wrap() Method – https://www.w3schools.com/jquery/html_wrap.asp

---

## Core Concept 2: `.unwrap()` — Remove the Parent Element

### Definitions

**Core Definition**
`.unwrap()` removes the **parent** of each matched element from the DOM, leaving the matched element (and its siblings) in the parent's place.

**Technical Definition**
The `.unwrap()` method removes the element's parent and returns the unwrapped content. It is effectively the **inverse** of the `.wrap()` method. The matched elements (and their siblings, if any) replace their parents within the DOM structure. As of jQuery 3.0, `.unwrap()` accepts an optional selector argument: if an element's parent does not match the selector, the element won't be unwrapped. The method was added in jQuery 1.4 and extended with the selector argument in jQuery 3.0.

**Beginner-Friendly Explanation**
`.unwrap()` is the "undo" button for wrapping. If an element is inside a wrapper, `.unwrap()` removes the wrapper and promotes the element (and anything else inside the wrapper) up one level in the DOM. It is like taking a gift out of its box and throwing the box away, leaving the gift where the box used to be.

### Purposes

- To remove an unnecessary or unwanted parent element from the DOM.
- To undo a previous `.wrap()` operation.
- To simplify the DOM structure by promoting children to a higher level.
- To remove wrapper elements that are no longer needed for layout or styling.
- To conditionally unwrap elements based on the parent's selector (jQuery 3.0+).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).unwrap();
$(selector).unwrap(parentSelector);    // jQuery 3.0+
```

**Component Breakdown**

- `$(selector)` : The element(s) whose parent should be removed.
- `.unwrap()` : Removes the parent of each matched element. Takes no arguments.
- `parentSelector` (Optional, jQuery 3.0+) : A selector to check the parent element against. If the parent does not match, the element is not unwrapped.
- Returns: A jQuery object containing the unwrapped (original) set of elements.

**Syntax Rules**

1. Removes the **parent** of each matched element, not the element itself.
2. The matched elements (and their siblings) replace their parents in the DOM.
3. The `<body>` element is never removed.
4. Added in jQuery 1.4; selector argument added in jQuery 3.0.
5. Returns the original set of matched elements (now at a higher level).

**Constraints and Limitations**

- If the matched element has no parent (e.g., it is the root element), nothing happens.
- The parent is removed entirely, along with any other children it may have had (which are promoted, not removed).
- Cannot unwrap more than one level at a time; call `.unwrap()` multiple times for nested wrappers.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Unwrapping**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.unwrap() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .wrapper { border: 2px solid red; padding: 10px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="wrapper">
      <p class="target">Hello</p>
    </div>
    <div class="wrapper">
      <p class="target">Goodbye</p>
    </div>
  </div>

  <script>
    $(function () {
      // Step 1: Verify initial structure
      console.log("Wrappers before:", $(".wrapper").length); // 2

      // Step 2: Unwrap the paragraphs (removes their parents)
      $(".target").unwrap();

      // Step 3: Verify the wrappers are gone
      console.log("Wrappers after:", $(".wrapper").length); // 0
      console.log("Targets still exist:", $(".target").length); // 2
      console.log("Container children:", $(".container").children().length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Wrappers before: 2
Wrappers after: 0
Targets still exist: 2
Container children: 2
```

**Why This Output Occurs**
`.unwrap()` removes the `.wrapper` divs (the parents of the `.target` paragraphs). The paragraphs are promoted to become direct children of `.container`. The wrappers are removed entirely.

---

**Example 2: Unwrapping with a Selector Filter (jQuery 3.0+)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.unwrap() — Selector Filter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="wrapper special">
      <p class="target">Special paragraph</p>
    </div>
    <div class="wrapper">
      <p class="target">Normal paragraph</p>
    </div>
  </div>

  <script>
    $(function () {
      // Unwrap only if the parent matches .special
      $(".target").unwrap(".special");

      // Verify
      console.log("Special wrapper:", $(".special").length); // 0 (removed)
      console.log("Normal wrapper:", $(".wrapper").length); // 1 (preserved)
      console.log("Targets:", $(".target").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Special wrapper: 0
Normal wrapper: 1
Targets: 2
```

**Why This Output Occurs**
`.unwrap(".special")` only removes the parent if it matches the `.special` selector. The wrapper with class `special` is removed, but the normal wrapper is preserved because its parent does not match the selector.

### Real-World Cases

- **Removing Redundant Wrappers**: `$(".content").unwrap()` to remove a wrapper div added by a template engine.
- **Undoing a Wrap Operation**: `$(".wrapped").unwrap()` to revert a previous `.wrap()` call.
- **Simplifying DOM After AJAX**: Removing wrapper divs that were only needed during loading.
- **Conditional Unwrapping**: `$(".item").unwrap(".legacy-wrapper")` to remove only legacy wrappers.

### References

- jQuery API — .unwrap() – https://api.jquery.com/unwrap/
- jQuery API — DOM Removal – https://api.jquery.com/category/manipulation/dom-removal/
- W3Schools — jQuery unwrap() Method – https://www.w3schools.com/jquery/html_unwrap.asp

---

## Core Concept 3: `.wrapAll()` — Wrap All Elements as a Group

### Definitions

**Core Definition**
`.wrapAll()` wraps a **single** HTML structure around **all** elements in the set of matched elements, treating them as one group.

**Technical Definition**
The `.wrapAll()` function can take any string or object that could be passed to the `$()` function to specify a DOM structure. This structure may be nested several levels deep, but should contain only one inmost element. The structure will be wrapped around **all** of the elements in the set of matched elements, as a single group. The matched elements are **moved** into the new wrapper, not cloned. As of jQuery 1.4, a callback function can be used; the callback is called once for the first element in the set (prior to jQuery 3.0, it was incorrectly called for every element).

**Beginner-Friendly Explanation**
`.wrapAll()` puts **one** box around **all** the selected items together. If you select three paragraphs and call `.wrapAll("<div class='box'></div>")`, all three paragraphs go into a **single** `<div class="box">`. This is different from `.wrap()`, which would create three separate boxes.

### Purposes

- To wrap a group of elements in a single container for unified styling or layout.
- To group related elements (e.g., a list of items) into a container without wrapping each individually.
- To move a set of elements into a new parent element as a group.
- To create structural containers for CSS grid, flexbox, or other layout systems.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).wrapAll(wrappingElement);
$(selector).wrapAll(function(index) { ... });
```

**Component Breakdown**

- `$(selector)` : The elements to be wrapped as a group.
- `wrappingElement` : A selector, element, HTML string, or jQuery object specifying the structure to wrap around all matched elements. Should contain only one inmost element.
- `function(index)` : A callback function returning the HTML content or jQuery object to wrap around all the matched elements. `this` refers to the first element in the set. Prior to jQuery 3.0, the callback was incorrectly called for every element and received the index.
- Returns: A jQuery object containing the original set of elements.

**Syntax Rules**

1. All matched elements are **moved** into a single wrapper structure.
2. The wrapper structure should contain only one inmost element.
3. If the matched elements are not contiguous in the DOM, `.wrapAll()` moves them to be together inside the wrapper.
4. As of jQuery 1.4, a callback function can be used; it is called once for the first element.
5. Returns the original set of elements for chaining.

**Constraints and Limitations**

- If the matched elements are scattered throughout the DOM, `.wrapAll()` moves them, which may change their order or position.
- The wrapper structure should contain only one inmost element.

### Multiple Annotated Complete Code Examples

**Example 1: Wrapping All Elements as a Group**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.wrapAll() — Group Wrapping</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .wrapper { border: 3px solid green; padding: 10px; }
    .inner { background: lightyellow; margin: 5px; padding: 5px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Wrap all .inner divs in a single wrapper
      $(".inner").wrapAll("<div class='wrapper'></div>");

      // Step 2: Verify there is only ONE wrapper
      console.log("Wrapper count:", $(".wrapper").length); // 1
      console.log("Wrapper children:", $(".wrapper").children().length); // 2

      // Step 3: Compare with .wrap() — which would create 2 wrappers
    });
  </script>
</body>
</html>
```

**Expected Output**
- Both `.inner` divs are wrapped in a **single** green-bordered wrapper.
- Console output:
```
Wrapper count: 1
Wrapper children: 2
```

**Why This Output Occurs**
`.wrapAll("<div class='wrapper'></div>")` creates one wrapper div and moves both `.inner` divs into it. Unlike `.wrap()`, which would create two separate wrappers, `.wrapAll()` creates only one.

---

**Example 2: Non-Contiguous Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.wrapAll() — Non-Contiguous Elements</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <p class="target">First</p>
    <div>Unrelated div</div>
    <p class="target">Second</p>
  </div>

  <script>
    $(function () {
      // WrapAll moves non-contiguous elements together
      $(".target").wrapAll("<div class='wrapper'></div>");

      console.log("Wrapper HTML:", $(".wrapper").html());
      // Both paragraphs are now together inside the wrapper
      console.log("Targets in wrapper:", $(".wrapper .target").length); // 2

      // The unrelated div is now a sibling of the wrapper, not between the paragraphs
      console.log("Container HTML:", $(".container").html());
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Wrapper HTML: <p class="target">First</p><p class="target">Second</p>
Targets in wrapper: 2
Container HTML: <div class="wrapper"><p class="target">First</p><p class="target">Second</p></div><div>Unrelated div</div>
```

**Why This Output Occurs**
`.wrapAll()` moves both `.target` paragraphs into the new wrapper, even though they were not contiguous. The unrelated `<div>` is left outside the wrapper, now appearing after it.

### Real-World Cases

- **List Grouping**: `$("li").wrapAll("<ul class='grouped-list'></ul>")` to group list items into a new list.
- **Footnotes**: `$("span.footnote").wrapAll("<ol id='notes'></ol>")` to collect footnotes into an ordered list.
- **Form Sections**: `$("fieldset").wrapAll("<div class='form-section'></div>")` to group fieldsets.
- **Card Grouping**: `$(".card").wrapAll("<div class='card-grid'></div>")` to create a grid container for cards.

### References

- jQuery API — .wrapAll() – https://api.jquery.com/wrapAll/
- jQuery API — DOM Insertion, Around – https://api.jquery.com/category/manipulation/dom-insertion-around/
- W3Schools — jQuery wrapAll() Method – https://www.w3schools.com/jquery/html_wrapall.asp

---

## Core Concept 4: `.wrapInner()` — Wrap the Inner Content of Each Element

### Definitions

**Core Definition**
`.wrapInner()` wraps an HTML structure around the **content** (inner HTML) of each element in the set of matched elements, leaving the element itself outside the wrapper.

**Technical Definition**
The `.wrapInner()` function can take any string or object that could be passed to the `$()` factory function to specify a DOM structure. This structure may be nested several levels deep, but should contain only one inmost element. The structure will be wrapped around the **content** of each of the elements in the set of matched elements. When passing a selector string, the expected input is well-formed HTML with correctly closed tags. As of jQuery 1.4, a callback function can be used to generate the wrapper dynamically for each element.

**Beginner-Friendly Explanation**
`.wrapInner()` puts a box around the **contents** of an element, not around the element itself. If you have a `<p>` with the text "Hello", `.wrapInner("<b></b>")` will wrap the text in a `<b>` tag, producing `<p><b>Hello</b></p>`. The `<p>` element stays outside the wrapper; only its contents go inside.

### Purposes

- To wrap the contents of each matched element in a new structure.
- To add a consistent inner container around text or child elements for styling.
- To apply typographic or layout wrappers without affecting the outer element.
- To prepare content for CSS transitions, animations, or grid layouts that require inner containers.
- To add semantic meaning (e.g., wrapping text in `<strong>` or `<em>`) to element contents.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).wrapInner(wrappingElement);
$(selector).wrapInner(function(index) { ... });
```

**Component Breakdown**

- `$(selector)` : The element(s) whose contents should be wrapped.
- `wrappingElement` : An HTML snippet, selector expression, jQuery object, or DOM element specifying the structure to wrap around the content of the matched elements. Should contain only one inmost element.
- `function(index)` : A callback function which generates a structure to wrap around the content. Receives the index position of the element in the set. `this` refers to the current element.
- Returns: A jQuery object containing the original set of elements.

**Syntax Rules**

1. Wraps the **contents** (child nodes, including text) of each matched element.
2. The element itself remains outside the wrapper.
3. The wrapper structure should contain only one inmost element.
4. As of jQuery 1.4, a callback function can be used; it is called once for each matched element.
5. When passing a selector string, the expected input is well-formed HTML with correctly closed tags.

**Constraints and Limitations**

- The wrapper structure should contain only one inmost element.
- Wrapping text nodes may affect event handlers attached to those text nodes (rare, but possible).
- The element's own attributes and event handlers are not affected; only its contents are wrapped.

### Multiple Annotated Complete Code Examples

**Example 1: Wrapping Inner Content**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.wrapInner() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .inner-wrap { background: lightblue; padding: 5px; }
  </style>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Step 1: Wrap the contents of each .inner div
      $(".inner").wrapInner("<div class='inner-wrap'></div>");

      // Step 2: Verify the structure
      console.log("Inner wrap count:", $(".inner-wrap").length); // 2
      console.log("Inner div children:", $(".inner").first().children().length); // 1

      // Step 3: Verify the text is inside the wrapper
      console.log("Inner wrap text:", $(".inner-wrap").first().text()); // "Hello"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The text content of each `.inner` div is wrapped in a light blue `.inner-wrap` div.
- Console output:
```
Inner wrap count: 2
Inner div children: 1
Inner wrap text: Hello
```

**Why This Output Occurs**
`.wrapInner("<div class='inner-wrap'></div>")` wraps the contents of each `.inner` div in a new `.inner-wrap` div. The `.inner` div itself remains outside the wrapper, with the wrapper as its only child.

---

**Example 2: Wrapping with a Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.wrapInner() — Callback Function</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="container">
    <div class="inner">Hello</div>
    <div class="inner">Goodbye</div>
  </div>

  <script>
    $(function () {
      // Wrap contents with a class derived from the text
      $(".inner").wrapInner(function () {
        return "<div class='" + $(this).text().toLowerCase() + "'></div>";
      });

      console.log("Hello wrapper:", $(".hello").length); // 1
      console.log("Goodbye wrapper:", $(".goodbye").length); // 1
      console.log("Hello wrapper text:", $(".hello").text()); // "Hello"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Hello wrapper: 1
Goodbye wrapper: 1
Hello wrapper text: Hello
```

**Why This Output Occurs**
The callback function is called once for each `.inner` element. It reads the element's text, converts it to lowercase, and returns an HTML string that becomes the inner wrapper. Each `.inner` div gets a unique inner wrapper.

### Real-World Cases

- **Typographic Styling**: `$("p").wrapInner("<span class='text'></span>")` to apply text-specific styling.
- **Animation**: `$(".card").wrapInner("<div class='card-inner'></div>")` to create an inner container for flip animations.
- **Grid Layouts**: `$(".grid-item").wrapInner("<div class='grid-cell'></div>")` to add inner containers for grid alignment.
- **Accessibility**: `$("h1").wrapInner("<span class='visually-hidden'></span>")` to add screen-reader text.

### References

- jQuery API — .wrapInner() – https://api.jquery.com/wrapInner/
- jQuery API — DOM Insertion, Around – https://api.jquery.com/category/manipulation/dom-insertion-around/
- W3Schools — jQuery wrapInner() Method – https://www.w3schools.com/jquery/html_wrapinner.asp

---

## Summary Table: jQuery Wrapping Methods at a Glance

| Method | What It Wraps | Wrapper Count | Element Position | Inverse | Added |
|---|---|---|---|---|---|
| `.wrap()` | Each element individually | One per element | Inside wrapper | `.unwrap()` | 1.0 |
| `.unwrap()` | Removes the parent | N/A | Promoted to parent's level | `.wrap()` | 1.4 |
| `.wrapAll()` | All elements as a group | One for the entire set | Moved inside single wrapper | N/A | 1.2 |
| `.wrapInner()` | The contents of each element | One per element | Outside wrapper (contents inside) | N/A | 1.2 |

### Key Behaviour Notes

| Behaviour | `.wrap()` | `.wrapAll()` | `.wrapInner()` |
|---|---|---|---|
| Creates copies of wrapper | Yes (one per element) | No (one wrapper) | Yes (one per element) |
| Moves matched elements | No (elements stay, wrappers added) | Yes (elements moved into wrapper) | No (contents wrapped in place) |
| Returns | Original set | Original set | Original set |
| Callback receives | Index and `this` = element | Index (once for first) and `this` = first element | Index and `this` = element |
| Selector argument | Yes (first match used) | Yes | Yes |

---

## General References

- jQuery API — DOM Insertion, Around – https://api.jquery.com/category/manipulation/dom-insertion-around/
- jQuery API — .wrap() – https://api.jquery.com/wrap/
- jQuery API — .unwrap() – https://api.jquery.com/unwrap/
- jQuery API — .wrapAll() – https://api.jquery.com/wrapAll/
- jQuery API — .wrapInner() – https://api.jquery.com/wrapInner/
- jQuery API — DOM Removal – https://api.jquery.com/category/manipulation/dom-removal/
- W3Schools — jQuery wrap() Method – https://www.w3schools.com/jquery/html_wrap.asp
- W3Schools — jQuery unwrap() Method – https://www.w3schools.com/jquery/html_unwrap.asp
- W3Schools — jQuery wrapAll() Method – https://www.w3schools.com/jquery/html_wrapall.asp
- W3Schools — jQuery wrapInner() Method – https://www.w3schools.com/jquery/html_wrapinner.asp
- O'Reilly — jQuery Pocket Reference – https://www.oreilly.com/library/view/jquery-pocket-reference/9781449397227/
- O'Reilly — Learning jQuery 3 (Fifth Edition) – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/
- Stack Overflow — Difference between jQuery wrap and wrapAll – https://stackoverflow.com/questions/11849876/
- Stack Overflow — jQuery wrap vs wrapAll vs wrapInner – https://stackoverflow.com/questions/5034184/
- Stack Overflow — How to use .wrapAll() in jQuery? – https://stackoverflow.com/questions/1571722/
- Stack Overflow — jQuery unwrap() method – https://stackoverflow.com/questions/1436502/
- jQuery Bug Tracker — index of callback function in .wrap is always 0 – https://bugs.jquery.com/ticket/10181/