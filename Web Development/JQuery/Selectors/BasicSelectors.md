# jQuery Basic Selectors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery basic selectors are the foundational string expressions passed to the `$()` function to identify and retrieve sets of DOM elements based on their tag name, ID attribute, class attribute, or a combination thereof.

**Technical Definition**
Basic selectors form the first category in the jQuery selector API. When `$()` is called with a selector string, jQuery parses the string and invokes the appropriate native DOM method — `getElementsByTagName()` for element selectors, `getElementById()` for ID selectors, and `getElementsByClassName()` for class selectors — to build a jQuery object containing the matched elements. The universal selector (`*`) matches every element in the document, and the multiple selector combinator (commas) merges the results of several selectors into one jQuery object.

**Beginner-Friendly Explanation**
Think of jQuery selectors as a way of telling the browser “find me these elements.” If you want all paragraphs, you say `$("p")`. If you want the element with a specific ID, you say `$("#header")`. If you want everything with a certain class, you say `$(".intro")`. These are the basic building blocks of every jQuery operation — you always select something first, then do something with it.

### Key Characteristics

- **CSS-Compatible**: Basic selectors use the same syntax as CSS selectors, making them intuitive for anyone familiar with styling.
- **Native Performance**: jQuery delegates to native browser methods (`getElementById`, `getElementsByTagName`, `getElementsByClassName`) wherever possible, making basic selectors the fastest selector category.
- **Implicit Iteration**: Methods called on the resulting jQuery object automatically apply to every matched element.
- **Composable**: Selectors can be combined (e.g., `"div.myClass"`) for greater precision.
- **Static Collections**: The returned jQuery object represents a snapshot of matched elements at the time of selection.

### Prerequisites

- Basic understanding of HTML structure and attributes (`id`, `class`, `tag names`).
- Familiarity with CSS selectors (element, ID, class).
- jQuery library included in the page via a `<script>` tag or CDN.
- Access to a browser with developer tools for testing.

### Related Programming Areas

- **CSS Styling**: Selectors mirror CSS syntax, so styling knowledge transfers directly.
- **DOM Traversal**: Basic selectors are often the starting point for more complex traversal chains.
- **Event Handling**: Events are typically bound to elements selected with basic selectors.
- **Dynamic Content**: Selectors are used to find and manipulate elements created after page load.

### Core Concepts / Features

1. Element Selectors
2. ID Selectors
3. Class Selectors
4. Universal Selector
5. Multiple Selectors

---

## Core Concept 1: Element Selectors

### Definitions

**Core Definition**
The element selector selects all DOM elements that match a given HTML tag name.

**Technical Definition**
`jQuery("element")` — where `element` is the tag name of a DOM node (e.g., `div`, `p`, `span`) — returns a jQuery object containing every element in the document with that tag name. Internally, jQuery invokes JavaScript's `document.getElementsByTagName()` to retrieve the matching elements.

**Beginner-Friendly Explanation**
If you write `$("p")`, jQuery finds every `<p>` element on the page and puts them all into a single jQuery object. You can then apply methods to that object, and jQuery will automatically apply them to every paragraph.

### Purposes

- To select all instances of a particular HTML tag for batch operations.
- To apply styles, event handlers, or animations to every element of a given type.
- To serve as a broad selection that can be narrowed with traversal methods.
- To count or measure all elements of a specific type on the page.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("element")
```

**Component Breakdown**

- `$` : The jQuery function.
- `("element")` : A string containing the HTML tag name (without angle brackets).
- Returns: A jQuery object containing all matching elements.

**Syntax Rules**

1. The tag name is written **without** angle brackets: `"p"`, not `"<p>"`.
2. Tag names are case-insensitive in HTML documents.
3. If no elements match, the returned jQuery object is empty (`length === 0`).
4. The order of elements in the returned object follows document order.

**Constraints and Limitations**

- Selecting an element that does not exist returns an empty jQuery object, not an error.
- Very common element selectors (e.g., `$("div")`) can be slow on large documents because they scan the entire DOM.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting All Paragraphs**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Selector — Paragraphs</title>
  <!-- Step 1: Include jQuery -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
  <p>Paragraph 3</p>
  <div>Not a paragraph</div>

  <script>
    // Step 2: Wait for the DOM to be ready
    $(function () {
      // Step 3: Select all <p> elements
      var $paragraphs = $("p");

      // Step 4: Log the count and apply a style
      console.log("Paragraph count:", $paragraphs.length); // 3

      $paragraphs.css("background-color", "lightyellow");
    });
  </script>
</body>
</html>
```

**Expected Output**
- All three paragraphs receive a light yellow background.
- The `<div>` remains unchanged.
- Console output: `Paragraph count: 3`

**Why This Output Occurs**
`$("p")` invokes `document.getElementsByTagName("p")`, which returns all `<p>` elements. The `<div>` is not a paragraph and is therefore excluded. The `.css()` method implicitly iterates over all three paragraphs, applying the background colour to each.

---

**Example 2: Element Selector with `.length` for Counting**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Selector — Counting</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  </ul>

  <script>
    $(function () {
      // Select all <li> elements and count them
      var count = $("li").length;
      console.log("List items:", count); // 3

      // Add a border to every list item
      $("li").css("border", "1px solid gray");
    });
  </script>
</body>
</html>
```

**Expected Output**
- All three list items receive a gray border.
- Console output: `List items: 3`

**Why This Output Occurs**
`$("li")` selects every `<li>` element. The `.length` property returns the number of matched elements without requiring a loop. `.css()` then applies the border to each element.

### Real-World Cases

- **Styling All Links**: `$("a").css("text-decoration", "none")` removes underlines from all anchors.
- **Hiding All Images**: `$("img").hide()` hides every image on the page.
- **Form Reset**: `$("input").val("")` clears all input fields.
- **Counting Elements**: `$("div").length` checks how many `<div>` elements exist.

### References

- Element Selector (“element”) – https://api.jquery.com/element-selector/
- Selecting Elements – https://learn.jquery.com/using-jquery-core/selecting-elements/
- W3Schools — jQuery Selectors – https://www.w3schools.com/jquery/jquery_selectors.asp

---

## Core Concept 2: ID Selectors

### Definitions

**Core Definition**
The ID selector selects the single element that has a specific `id` attribute.

**Technical Definition**
`jQuery("#id")` — where `id` is the value of an element's `id` attribute — returns a jQuery object containing either zero or one element. jQuery uses the native `document.getElementById()` function, which is extremely efficient. Each `id` value must be unique within a document. If multiple elements share the same ID (invalid HTML), jQuery selects only the first match.

**Beginner-Friendly Explanation**
Every HTML element can have a unique name tag called an `id`. The ID selector is like calling someone by their unique name — only one element should answer. If you write `$("#header")`, jQuery finds the one element whose `id` is `"header"`.

### Purposes

- To select a single, unique element for targeted manipulation.
- To perform operations that apply to only one element on the page.
- To efficiently retrieve an element without scanning the entire DOM.
- To serve as a precise starting point for traversal into child elements.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("#id")
```

**Component Breakdown**

- `$` : The jQuery function.
- `("#id")` : A string beginning with `#` followed by the ID value.
- Returns: A jQuery object containing zero or one element.

**Syntax Rules**

1. The ID value must be prefixed with `#`: `$("#header")`.
2. IDs must be unique within a document; duplicates are invalid HTML.
3. If the ID contains special characters (periods, colons, brackets), escape them with backslashes: `$("#myID\\.entry\\[1\\]")`.
4. If no element matches, the jQuery object is empty (`length === 0`).

**Constraints and Limitations**

- Only the first matching element is returned if duplicate IDs exist, but the document is invalid.
- ID selectors cannot select multiple elements by design.
- The ID value is case-sensitive in XHTML but not in HTML in practice.

### Multiple Annotated Complete Code Examples

**Example 1: Basic ID Selection**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ID Selector — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="notMe">This is not the target</div>
  <div id="myDiv">This is the target</div>

  <script>
    $(function () {
      // Step 1: Select the element with id="myDiv"
      var $target = $("#myDiv");

      // Step 2: Verify and style
      console.log("Length:", $target.length);       // 1
      console.log("Text:", $target.text());         // "This is the target"

      $target.css("border", "3px solid red");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The second `<div>` receives a red border.
- Console output:
```
Length: 1
Text: This is the target
```

**Why This Output Occurs**
`$("#myDiv")` calls `document.getElementById("myDiv")`, which returns the single matching element. The other `<div>` with `id="notMe"` is not selected. The `.css()` method applies the border to the matched element.

---

**Example 2: Escaping Special Characters in IDs**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ID Selector — Escaping</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="myID.entry[0]">Entry 0</div>
  <div id="myID.entry[1]">Entry 1</div>
  <div id="myID.entry[2]">Entry 2</div>

  <script>
    $(function () {
      // The ID contains periods and brackets — escape them with backslashes
      $("#myID\\.entry\\[1\\]").css("border", "3px solid red");
    });
  </script>
</body>
</html>
```

**Expected Output**
- Only the second `<div>` (id `"myID.entry[1]"`) receives a red border.

**Why This Output Occurs**
Characters like `.`, `[`, and `]` have special meaning in CSS selectors. Backslashes tell jQuery to treat them as literal characters. Without escaping, the selector would be misinterpreted and fail to match.

### Real-World Cases

- **Navigation Highlighting**: `$("#current-page").addClass("active")` highlights the active navigation link.
- **Form Field Focus**: `$("#email").focus()` focuses the email input.
- **Modal Control**: `$("#modal").fadeIn()` displays a specific modal dialog.
- **Content Updates**: `$("#result").html(data)` injects data into a unique container.

### References

- ID Selector (“#id”) – https://api.jquery.com/id-selector/
- How do I select an element by an ID that has characters used in CSS notation? – https://learn.jquery.com/using-jquery-core/faq/how-do-i-select-an-element-by-an-id-that-has-characters-used-in-css-notation/
- W3Schools — jQuery #id Selector – https://www.w3schools.com/jquery/sel_id.asp

---

## Core Concept 3: Class Selectors

### Definitions

**Core Definition**
The class selector selects all elements that have a specified class in their `class` attribute.

**Technical Definition**
`jQuery(".class")` — where `class` is the class name — returns a jQuery object containing every element whose `class` attribute includes that class name. An element may have multiple classes; only one needs to match. jQuery uses the native `getElementsByClassName()` function when supported by the browser.

**Beginner-Friendly Explanation**
Classes are like labels that can be shared by many elements. The class selector finds every element that carries a particular label. If you write `$(".intro")`, jQuery finds every element with `class="intro"` — even if those elements are different types (paragraphs, divs, spans, etc.).

### Purposes

- To select all elements sharing a common class for batch styling or behaviour.
- To apply consistent styling to groups of related elements.
- To attach event handlers to multiple elements at once.
- To filter or manipulate subsets of elements based on semantic grouping.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(".class")
$(".class1.class2")   // elements with BOTH classes
```

**Component Breakdown**

- `$` : The jQuery function.
- `(".class")` : A string beginning with `.` followed by the class name.
- `(".class1.class2")` : Combined class selector matching elements with both classes.
- Returns: A jQuery object containing all matching elements.

**Syntax Rules**

1. The class name must be prefixed with a dot: `$(".intro")`.
2. An element can have multiple classes; the selector matches if **any** of them match (unless chained).
3. Chaining class selectors (`.class1.class2`) requires the element to have **both** classes.
4. Class names are case-sensitive in modern browsers.
5. The order of classes in the HTML attribute does not matter.

**Constraints and Limitations**

- Class selectors return **all** matching elements, not just the first.
- If no elements match, the jQuery object is empty.
- Very common class names can lead to large result sets, which may impact performance.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Class Selection**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Class Selector — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="notMe">Not selected</div>
  <div class="myClass">Selected div</div>
  <span class="myClass">Selected span</span>

  <script>
    $(function () {
      // Step 1: Select all elements with class "myClass"
      var $items = $(".myClass");

      // Step 2: Verify and style
      console.log("Count:", $items.length); // 2

      $items.css("border", "3px solid red");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `<div>` and `<span>` with `class="myClass"` both receive a red border.
- The `<div class="notMe">` remains unchanged.
- Console output: `Count: 2`

**Why This Output Occurs**
`$(".myClass")` calls `document.getElementsByClassName("myClass")`, which returns both elements that have that class. The selector matches regardless of the element's tag name, so both the `<div>` and `<span>` are selected.

---

**Example 2: Combined Class Selectors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Class Selector — Combined</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="myclass">Only myclass</div>
  <div class="myclass otherclass">Both classes</div>
  <span class="myclass otherclass">Both classes (span)</span>

  <script>
    $(function () {
      // Select only elements that have BOTH "myclass" and "otherclass"
      $(".myclass.otherclass").css("border", "3px solid blue");
    });
  </script>
</body>
</html>
```

**Expected Output**
- Only the `<div>` and `<span>` that have **both** `myclass` and `otherclass` receive a blue border.
- The `<div>` with only `myclass` remains unchanged.

**Why This Output Occurs**
The combined selector `.myclass.otherclass` requires both classes to be present. The first `<div>` has only `myclass`, so it does not match. The other two elements have both classes and are selected.

### Real-World Cases

- **Highlighting Errors**: `$(".error").addClass("highlight")` highlights all error messages.
- **Tab Interfaces**: `$(".tab-content").hide()` hides all tab panels before showing one.
- **Card Layouts**: `$(".card").css("border-radius", "8px")` styles all card elements.
- **Form Validation**: `$(".required").addClass("mandatory")` marks all required fields.

### References

- Class Selector (“.class”) – https://api.jquery.com/class-selector/
- Selecting Elements by Class Name – https://learn.jquery.com/using-jquery-core/selecting-elements/
- W3Schools — jQuery .class Selector – https://www.w3schools.com/jquery/sel_class.asp

---

## Core Concept 4: Universal Selector

### Definitions

**Core Definition**
The universal selector, written as `*`, selects every element in the document.

**Technical Definition**
`jQuery("*")` returns a jQuery object containing every element node in the DOM, including `<html>`, `<head>`, `<body>`, and all descendants. The jQuery documentation explicitly cautions that the universal selector is “extremely slow, except when used by itself.” When combined with other selectors, it should be used sparingly and with performance in mind.

**Beginner-Friendly Explanation**
The universal selector means “everything.” `$("*")` finds every single element on the page. It is powerful but slow, so it should be used only when you truly need to target all elements, such as for a global reset or a diagnostic count.

### Purposes

- To select every element in the document for global operations.
- To count the total number of elements on a page for diagnostic purposes.
- To apply a universal style reset across all elements.
- To serve as a context for `.find()` when scoped to a container.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("*")                    // all elements in the document
$("#container").find("*") // all elements within a specific container
```

**Component Breakdown**

- `$("*")` : Selects all elements.
- `$("#container").find("*")` : Scoped selection — only elements inside `#container`.

**Syntax Rules**

1. The universal selector is written as an asterisk enclosed in quotes: `"*"`.
2. It can be used alone (`$("*")`) or as part of a compound selector (`$("div *")`).
3. When used alone, it is relatively fast; when combined with other selectors, performance degrades significantly.
4. It includes elements in `<head>`, such as `<script>` and `<link>`, unless scoped.

**Constraints and Limitations**

- **Performance**: The universal selector is extremely slow, especially on large documents with many elements.
- **Inclusiveness**: It selects elements the developer may not expect, such as `<head>`, `<script>`, and `<style>`.
- **Best Practice**: Prefer scoping with `.find("*")` within a container or using more specific selectors.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting All Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Universal Selector — All Elements</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>DIV</div>
  <span>SPAN</span>
  <p>P <button>Button</button></p>

  <script>
    $(function () {
      // Step 1: Select all elements and count them
      var elementCount = $("*").length;
      console.log("Total elements:", elementCount);

      // Step 2: Apply a border to all elements for visual demonstration
      $("*").css("border", "1px solid red");

      // Step 3: Add a heading with the count
      $("body").prepend("<h3>" + elementCount + " elements found</h3>");
    });
  </script>
</body>
</html>
```

**Expected Output**
- Every element on the page receives a red border.
- A heading appears at the top showing the total element count.

**Why This Output Occurs**
`$("*")` selects every element node in the document, including `<html>`, `<head>`, `<body>`, and all their descendants. The `.css()` method applies the border to all of them. The count is prepended to the body for visibility.

---

**Example 2: Scoped Universal Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Universal Selector — Scoped</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="test">
    <div>Inner DIV</div>
    <span>Inner SPAN</span>
    <p>Inner P <button>Button</button></p>
  </div>
  <div>Outside DIV</div>

  <script>
    $(function () {
      // Step 1: Find all elements only within #test
      var count = $("#test").find("*").length;
      console.log("Elements inside #test:", count);

      // Step 2: Style only those elements
      $("#test").find("*").css("border", "2px solid green");

      // Step 3: Prepend the count inside #test
      $("#test").prepend("<h3>" + count + " elements found</h3>");
    });
  </script>
</body>
</html>
```

**Expected Output**
- Only elements inside `#test` receive a green border.
- The outside `<div>` remains unchanged.
- A heading appears inside `#test` showing the count.

**Why This Output Occurs**
`$("#test").find("*")` scopes the universal selector to the `#test` container. Only elements inside that container are selected and styled. The outside `<div>` is excluded, demonstrating the performance and precision advantage of scoping.

### Real-World Cases

- **Global CSS Reset**: `$("*").css("box-sizing", "border-box")` applies a universal box-sizing rule.
- **Diagnostic Counting**: `$("*").length` counts all elements for performance analysis.
- **Debug Outlines**: `$("*").css("outline", "1px solid red")` adds outlines to every element during layout debugging.
- **Scoped Styling**: `$(".widget").find("*").css("font-family", "Arial")` applies a font to all descendants of a widget.

### References

- All Selector (“*”) – https://api.jquery.com/all-selector/
- Scaler Topics — jQuery Selectors – https://www.scaler.com/topics/jquery/jquery-selectors/

---

## Core Concept 5: Multiple Selectors

### Definitions

**Core Definition**
The multiple selector combinator allows several selectors to be combined with commas, returning the combined results of all specified selectors in a single jQuery object.

**Technical Definition**
`jQuery("selector1, selector2, selectorN")` returns a jQuery object containing the union of all elements matched by each individual selector. The order of elements in the returned object follows document order, which may differ from the order in which the selectors were specified. An alternative to this combinator is the `.add()` method.

**Beginner-Friendly Explanation**
The multiple selector is like giving a shopping list with several items: “Find all paragraphs, all divs, and all spans.” Instead of running three separate queries, you write one selector with commas, and jQuery returns everything in one go.

### Purposes

- To select disparate elements with a single query for efficient batch operations.
- To apply the same styling or behaviour to elements of different types or classes.
- To reduce the number of DOM queries by combining selections.
- To simplify code when multiple element groups require identical treatment.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$("selector1, selector2, selectorN")
```

**Component Breakdown**

- `"selector1, selector2, selectorN"` : A string containing two or more valid selectors separated by commas.
- Each selector can be an element, ID, class, or any other valid jQuery selector.
- Returns: A jQuery object containing the combined results.

**Syntax Rules**

1. Selectors are separated by commas: `$("h1, h2, h3")`.
2. Whitespace after commas is optional but recommended for readability.
3. The order of elements in the result is document order, not selector order.
4. Any valid selector can be combined, including ID, class, and attribute selectors.
5. The `.add()` method can be used as an alternative to the comma combinator.

**Constraints and Limitations**

- Combining many selectors into one string can reduce readability.
- The result set may contain duplicates if the same element matches multiple selectors; jQuery removes duplicates.
- Performance depends on the individual selectors; inefficient selectors slow the combined query.

### Multiple Annotated Complete Code Examples

**Example 1: Combining Element and Class Selectors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Multiple Selectors — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div>DIV element</div>
  <p class="myClass">Paragraph with myClass</p>
  <p class="notMyClass">Paragraph without myClass</p>
  <span>SPAN element</span>

  <script>
    $(function () {
      // Step 1: Combine div, .myClass, and span selectors
      var $combined = $("div, .myClass, span");

      // Step 2: Log the count and apply a style
      console.log("Combined count:", $combined.length); // 3

      $combined.css("border", "2px solid purple");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `<div>`, the `<p class="myClass">`, and the `<span>` receive a purple border.
- The `<p class="notMyClass">` remains unchanged.
- Console output: `Combined count: 3`

**Why This Output Occurs**
The multiple selector `"div, .myClass, span"` combines three separate queries: all `<div>` elements, all elements with class `myClass`, and all `<span>` elements. The results are merged into one jQuery object. The `<p class="notMyClass">` matches none of the selectors and is excluded.

---

**Example 2: Combining ID and Element Selectors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Multiple Selectors — ID and Element</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <h1 id="title">Page Title</h1>
  <p>First paragraph</p>
  <p>Second paragraph</p>
  <div>Some div</div>

  <script>
    $(function () {
      // Combine ID selector and element selector
      var $combined = $("#title, p");

      console.log("Count:", $combined.length); // 3

      $combined.css("color", "darkgreen");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `<h1>` and both `<p>` elements turn dark green.
- The `<div>` remains unchanged.
- Console output: `Count: 3`

**Why This Output Occurs**
The selector `"#title, p"` combines the ID selector `#title` (matching the `<h1>`) and the element selector `p` (matching both paragraphs). The results are merged into a single jQuery object with three elements. The `<div>` is not matched by either selector.

### Real-World Cases

- **Navigation Styling**: `$("nav a, footer a").css("text-decoration", "none")` removes underlines from all links in navigation and footer.
- **Heading Reset**: `$("h1, h2, h3, h4, h5, h6").css("margin", "0")` resets margins for all heading levels.
- **Form Elements**: `$("input, select, textarea").addClass("form-control")` styles all form controls consistently.
- **Error Highlighting**: `$(".error, .invalid, .required").addClass("highlight")` highlights all validation-related elements.

### References

- Multiple Selector (“selector1, selector2, selectorN”) – https://api.jquery.com/multiple-selector/
- Basic | jQuery API Documentation – https://api.jquery.com/category/selectors/basic-css-selectors/
- W3Schools — jQuery Selectors – https://www.w3schools.com/jquery/jquery_selectors.asp

---

## Summary Table: jQuery Basic Selectors at a Glance

| Selector | Syntax | Example | Returns | Performance |
|---|---|---|---|---|
| Element | `$("element")` | `$("p")` | All `<p>` elements | Fast |
| ID | `$("#id")` | `$("#header")` | Zero or one element | Fastest |
| Class | `$(".class")` | `$(".intro")` | All elements with class | Fast |
| Universal | `$("*")` | `$("*")` | All elements | Slow (caution) |
| Multiple | `$("sel1, sel2")` | `$("h1, h2")` | Combined results | Depends on selectors |

---

## General References

- jQuery API — Basic CSS Selectors – https://api.jquery.com/category/selectors/basic-css-selectors/
- jQuery API — Element Selector – https://api.jquery.com/element-selector/
- jQuery API — ID Selector – https://api.jquery.com/id-selector/
- jQuery API — Class Selector – https://api.jquery.com/class-selector/
- jQuery API — All Selector – https://api.jquery.com/all-selector/
- jQuery API — Multiple Selector – https://api.jquery.com/multiple-selector/
- Learn jQuery — Selecting Elements – https://learn.jquery.com/using-jquery-core/selecting-elements/
- W3Schools — jQuery Selectors – https://www.w3schools.com/jquery/jquery_selectors.asp
- MDN Web Docs — CSS Selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors
- jQuery Official Website – https://jquery.com/