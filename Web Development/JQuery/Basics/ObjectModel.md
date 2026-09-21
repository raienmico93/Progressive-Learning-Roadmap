# jQuery Object Model: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The jQuery Object Model is a JavaScript-based abstraction layer that wraps native DOM elements inside a specialised container object, exposing a unified API for DOM traversal, manipulation, event handling, and animation across all supported browsers.

**Technical Definition**
The jQuery Object Model is a monadic design pattern in which the `jQuery` factory function (aliased as `$`) returns an array-like object containing zero or more native DOM element references. This object inherits from `jQuery.prototype` (also exposed as `$.fn`), granting every instance access to a large library of chainable methods. Methods that mutate state return the same jQuery object (or a new one), enabling fluent interfaces, while getter methods return the underlying value.

**Beginner-Friendly Explanation**
Imagine you have a box of LEGO bricks (the native DOM elements). Handling each brick individually is tedious. The jQuery Object Model is like putting those bricks into a smart toolbox that knows how to paint, move, or hide all the bricks inside it at once. The `$` function is the act of putting bricks into that toolbox, and every tool (method) you use automatically applies to everything inside.

### Key Characteristics

- **Array-Like Structure**: A jQuery object has a `length` property and numeric indices (0, 1, 2, …), allowing array-style access to wrapped DOM elements.
- **Implicit Iteration**: Most methods automatically loop through every element in the collection, applying the operation to each one.
- **Method Chaining**: Methods that return a jQuery object can be chained together in a single statement, producing concise, fluent code.
- **Cross-Browser Compatibility**: Native DOM inconsistencies are normalised, so the same jQuery code works in all supported browsers.
- **Non-Destructive**: Most traversal methods return a new jQuery object rather than mutating the original, preserving the original selection for later use via `.end()`.
- **Extensible**: Plugins can be added by extending `$.fn`, making new methods available to every jQuery object.

### Prerequisites

- Basic understanding of HTML and CSS selectors.
- Familiarity with JavaScript fundamentals: variables, functions, objects, and arrays.
- Knowledge of the Document Object Model (DOM) — how HTML elements are represented as a tree of objects in the browser.
- A text editor and a web browser with a developer console for running examples.
- jQuery library included in the page via a `<script>` tag or CDN.

### Related Programming Areas

- **DOM Manipulation**: Creating, reading, updating, and deleting HTML elements.
- **Event Handling**: Attaching and removing event listeners using jQuery's normalised event system.
- **AJAX**: Asynchronous HTTP requests via `$.ajax()`, `$.get()`, and `$.post()`.
- **Animation and Effects**: Built-in methods like `.fadeIn()`, `.slideUp()`, and `.animate()`.
- **Plugin Development**: Extending `$.fn` to create reusable UI components.
- **Utility Functions**: Helpers such as `$.each()`, `$.map()`, and `$.extend()`.

### Core Concepts / Features

1. `$` Function
2. jQuery Object
3. Element Collections
4. Implicit Iteration
5. Method Chaining
6. Native DOM Element versus jQuery Object
7. Converting Between Native DOM and jQuery Objects

---

## Core Concept 1: `$` Function

### Definitions

**Core Definition**
The `$` function is the primary entry point into the jQuery library. It is an alias for the `jQuery` function and is used to select DOM elements, create new elements, or wrap existing DOM references in a jQuery object.

**Technical Definition**
`jQuery()` (or `$()`) is a factory function defined at the global scope. It accepts one or more arguments — a CSS selector string, a DOM element, an array of DOM elements, an HTML string, a plain object, an existing jQuery object, or a function — and returns a new jQuery object. The function is a property of the `window` object (or the global object in non-browser environments) and is also exposed as a property of the jQuery namespace itself: `jQuery === $`.

**Beginner-Friendly Explanation**
Think of `$` as a universal remote control. You point it at something on the page (by describing it with CSS-like syntax), and it hands you back a controller that can operate everything you pointed at. It is called “dollar” simply because `$` is a valid and short JavaScript variable name.

### Purposes

- To select DOM elements by CSS selector and return a jQuery object.
- To wrap an existing native DOM element or array of elements into a jQuery object.
- To create new DOM elements from an HTML string.
- To execute a function when the DOM is fully loaded (when a function is passed as the argument).
- To clone an existing jQuery object.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: Selection by CSS Selector**
```javascript
$(selector)
$(selector, context)
```
- `selector` (String): A CSS selector expression (e.g., `"div"`, `"#id"`, `".class"`).
- `context` (Element, Document, jQuery, or selector): Optional. A DOM element, document, or jQuery object to use as the search context. Internally equivalent to `$(context).find(selector)`.

**Syntax 2: Wrapping a DOM Element**
```javascript
$(element)
```
- `element` (Element): A native DOM element to wrap in a jQuery object.

**Syntax 3: Wrapping an Array of DOM Elements**
```javascript
$(elementArray)
```
- `elementArray` (Array): An array containing DOM elements to wrap. Elements are copied in array order, not necessarily DOM order.

**Syntax 4: Creating from HTML String**
```javascript
$(htmlString)
```
- `htmlString` (String): A string containing HTML markup (e.g., `"<div>Hello</div>"`). jQuery examines the string to determine if it looks like HTML (contains `<tag …>`); if so, it creates the corresponding DOM elements.

**Syntax 5: DOM-Ready Callback**
```javascript
$(function() { /* code */ })
```
- `function`: A function to execute once the DOM is fully parsed and ready. Equivalent to `$(document).ready(function)`.

**Syntax 6: Empty Set**
```javascript
$()
```
- No arguments. Returns an empty jQuery set (`length === 0`). Added in jQuery 1.4; prior versions returned a set containing the document node.

**Syntax Rules**

1. `$` and `jQuery` are interchangeable; both refer to the same function.
2. When a jQuery object is passed as an argument, a clone is created that references the same DOM elements.
3. If a string argument resembles HTML, jQuery creates elements rather than performing a DOM query.
4. Selector context is implemented via `.find()`, so `$("span", this)` is equivalent to `$(this).find("span")`.
5. If no elements match a selector, the returned jQuery object is empty (`length === 0`), not `null` or `undefined`.

**Constraints and Limitations**

- In `noConflict()` mode, the `$` alias may be released to avoid conflicts with other libraries; use `jQuery` instead.
- Passing a plain object (not a DOM element or array) wraps that object in a jQuery container but does not grant it DOM behaviour.
- HTML strings must contain valid markup; malformed HTML may be parsed inconsistently across browsers.

### Multiple Annotated Complete Code Examples

**Example 1: Selecting Elements by CSS Selector**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>$ Function — Selector Example</title>
  <!-- Step 1: Include jQuery from the official CDN -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p class="intro">Paragraph 1</p>
  <p class="intro">Paragraph 2</p>
  <p>Paragraph 3</p>

  <script>
    // Step 2: Wait until the DOM is ready
    $(function () {
      // Step 3: Select all elements with class "intro"
      var $intros = $(".intro");

      // Step 4: Log the number of matched elements
      console.log("Matched elements:", $intros.length);

      // Step 5: Log the text content of each matched element
      $intros.each(function (index) {
        console.log(index + ": " + $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Matched elements: 2
0: Paragraph 1
1: Paragraph 2
```

**Why This Output Occurs**
The selector `".intro"` matches only the two paragraphs that carry the `intro` class. The third paragraph is excluded. The `$intros.length` property returns `2`, and the `.each()` callback logs each matched element's index and text content.

---

**Example 2: Wrapping a Native DOM Element**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>$ Function — DOM Element Example</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" style="width:100px;height:100px;background:red;"></div>

  <script>
    $(function () {
      // Step 1: Get a native DOM element using vanilla JavaScript
      var nativeElement = document.getElementById("box");

      // Step 2: Wrap the native element in a jQuery object
      var $box = $(nativeElement);

      // Step 3: Use a jQuery method that is not available on native elements
      $box.css("background-color", "blue");

      console.log("jQuery object length:", $box.length);
      console.log("Is same element:", $box[0] === nativeElement); // true
    });
  </script>
</body>
</html>
```

**Expected Output**
- The red square turns blue.
- Console output:
```
jQuery object length: 1
Is same element: true
```

**Why This Output Occurs**
`document.getElementById("box")` returns a raw DOM element. Wrapping it with `$(nativeElement)` creates a jQuery object containing that element. The `.css()` method, available only on jQuery objects, changes the background colour. The `$box[0]` array-style access returns the original native element, confirming identity.

---

**Example 3: Creating Elements from an HTML String**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>$ Function — HTML String Example</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="container"></div>

  <script>
    $(function () {
      // Step 1: Create a new <p> element from an HTML string
      var $newParagraph = $("<p>Dynamically created</p>");

      // Step 2: Append it to the container
      $("#container").append($newParagraph);

      // Step 3: Verify the container's HTML
      console.log($("#container").html());
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
<p>Dynamically created</p>
```

**Why This Output Occurs**
Because the argument to `$()` begins with `<`, jQuery treats it as an HTML string and constructs the corresponding DOM element. `.append()` then inserts that element into the `#container` div. The final `.html()` call confirms the resulting markup.

### Real-World Cases

- **Form Validation**: `$("input[type=email]")` selects all email fields for client-side validation.
- **Dynamic Content Loading**: `$("<li>New item</li>")` creates list items before appending them to a `<ul>`.
- **Event Delegation Setup**: `$(document)` wraps the document to attach delegated event handlers.
- **Plugin Initialisation**: `$(".carousel").carousel()` wraps all carousel elements and initialises a plugin on each.

### References

- jQuery API — jQuery() – https://api.jquery.com/jQuery/
- MDN Web Docs — Document Object Model (DOM) – https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model
- Smashing Magazine — The Mystery Of The jQuery Object – https://www.smashingmagazine.com/2014/05/mystery-jquery-object-syntax-basic-introduction/

---

## Core Concept 2: jQuery Object

### Definitions

**Core Definition**
A jQuery object is a JavaScript object created by the `$` function that contains a collection of zero or more DOM elements and exposes the entire jQuery API as its methods.

**Technical Definition**
A jQuery object is an instance of the `jQuery` constructor. It is an array-like object with a `length` property and numeric indices from `0` to `length - 1`, each referencing a native DOM element. The object’s prototype chain includes `jQuery.prototype` (aliased as `$.fn`), where all jQuery methods are defined. When a jQuery method is called, it is executed with `this` bound to the jQuery object itself, and most methods return either the same jQuery object or a new jQuery object containing a different set of elements.

**Beginner-Friendly Explanation**
A jQuery object is like a shopping bag that holds one or more HTML elements. The bag itself has special tools attached to it — tools that can style, move, or delete everything inside the bag at once. You get a bag whenever you use `$()`, and the bag always knows how many items it contains and what they are.

### Purposes

- To provide a consistent, cross-browser wrapper around native DOM elements.
- To enable implicit iteration so that methods apply to all elements in the collection.
- To support method chaining by returning jQuery objects from setter methods.
- To expose a rich API of traversal, manipulation, and event methods.
- To allow plugins to extend the behaviour of all jQuery objects.

### Syntax Rules and Structure

**Complete General Syntax**

A jQuery object is not created with `new`; it is always created through the `$` / `jQuery` factory function:

```javascript
var $collection = $(selectorOrElementOrHtml);
```

**Component Breakdown**

- `var $collection` : A variable conventionally prefixed with `$` to indicate it holds a jQuery object.
- `$(…)` : The factory call that produces the jQuery object.
- `selectorOrElementOrHtml` : The argument that determines what the object contains.

**Syntax Rules**

1. jQuery objects are always returned by the `$` function; you never use `new jQuery()` directly (the constructor is internal).
2. A jQuery object always has a `.length` property indicating how many elements it contains.
3. Accessing elements by numeric index (`$collection[0]`) returns a native DOM element, not a jQuery object.
4. Calling a jQuery method on an empty jQuery object is safe; it simply does nothing.
5. jQuery objects are **not** arrays; they are array-like objects. Methods like `Array.prototype.forEach` do not work directly on them without conversion.

**Constraints and Limitations**

- A jQuery object cannot be used where a native DOM element is expected (e.g., passing it to `document.appendChild()`) without conversion.
- Two jQuery objects that wrap the same DOM element are not `===` equal, even though their contents are equivalent.
- jQuery objects created from the same selector at different times are independent clones.

### Multiple Annotated Complete Code Examples

**Example 1: Inspecting a jQuery Object**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>jQuery Object — Inspection</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Alpha</li>
    <li>Beta</li>
    <li>Gamma</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Create a jQuery object from a selector
      var $items = $("li");

      // Step 2: Inspect its array-like properties
      console.log("Length:", $items.length);        // 3
      console.log("First element:", $items[0].textContent); // "Alpha"

      // Step 3: Confirm it is not a true Array
      console.log("Is Array:", Array.isArray($items)); // false

      // Step 4: Use a jQuery method on the object
      $items.addClass("highlight");
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Length: 3
First element: Alpha
Is Array: false
```

**Why This Output Occurs**
`$("li")` selects all three list items. The `.length` property returns `3`. `$items[0]` returns the first native DOM element, whose `textContent` is `"Alpha"`. `Array.isArray()` returns `false` because jQuery objects are array-like, not true arrays. The `.addClass()` method implicitly iterates over all three elements, adding the `highlight` class to each.

---

**Example 2: jQuery Object is a Collection, Not an Array**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>jQuery Object — Array-Like</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="item">One</div>
  <div class="item">Two</div>

  <script>
    $(function () {
      var $divs = $(".item");

      // Step 1: Array-like access works
      console.log($divs[0].textContent); // "One"

      // Step 2: Native array methods do NOT work directly
      // $divs.forEach(...) would throw a TypeError

      // Step 3: jQuery provides .each() for iteration
      $divs.each(function (index) {
        console.log("Index " + index + ": " + $(this).text());
      });

      // Step 4: Convert to a true array using .get()
      var nativeArray = $divs.get();
      console.log("Native array length:", nativeArray.length); // 2
      console.log("Is Array:", Array.isArray(nativeArray));    // true
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
One
Index 0: One
Index 1: Two
Native array length: 2
Is Array: true
```

**Why This Output Occurs**
jQuery objects support numeric indexing but lack array methods like `forEach`. The `.each()` method provides iteration. Calling `.get()` with no arguments returns a true JavaScript array of the underlying DOM elements.

### Real-World Cases

- **Storing Selections**: `var $buttons = $("button")` caches a jQuery object for repeated use.
- **Passing Selections to Functions**: Functions can accept a jQuery object as a parameter and call jQuery methods on it.
- **Plugin Authoring**: Plugins receive a jQuery object as `this` inside their methods.
- **Conditional Logic**: Checking `$selection.length > 0` determines whether a selector matched anything.

### References

- Learn jQuery — The jQuery Object – https://learn.jquery.com/using-jquery-core/jquery-object/
- jQuery API — Types: jQuery – https://api.jquery.com/Types/#jQuery
- MDN Web Docs — Array-like objects – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections#array-like_objects

---

## Core Concept 3: Element Collections

### Definitions

**Core Definition**
An element collection is the set of native DOM elements contained within a jQuery object. It is the “payload” that jQuery methods operate on.

**Technical Definition**
When the jQuery function is invoked, it searches the DOM (or accepts provided elements) and stores the matched DOM elements in the newly created jQuery object. These elements are stored at numeric indices `0` through `length - 1` of the jQuery object. The collection is static: it represents the elements matched at the time the jQuery object was created, though methods that traverse or filter the DOM can produce new collections.

**Beginner-Friendly Explanation**
If the jQuery object is a shopping bag, the element collection is the actual items inside the bag. When you write `$("div")`, jQuery finds every `<div>` on the page and puts them into the bag. You can look at the bag’s label (`.length`) to see how many items there are, and you can pull out individual items by their position.

### Purposes

- To group multiple DOM elements under a single jQuery object for batch operations.
- To enable implicit iteration across all matched elements.
- To provide a stable reference to a set of elements for later manipulation.
- To allow filtering and traversal methods to produce new collections from the original.
- To support array-style access and `.get()` conversion.

### Syntax Rules and Structure

**Complete General Syntax**

Element collections are not created directly; they are the result of calling `$()`:

```javascript
var $collection = $(selector);
var elementCount = $collection.length;
var firstElement = $collection[0];       // native DOM element
var lastElement = $collection.get(-1);   // native DOM element
```

**Component Breakdown**

- `$collection.length` : Number of elements in the collection.
- `$collection[index]` : Native DOM element at the given zero-based index.
- `$collection.get(index)` : Same as above, but supports negative indices (counted from the end).
- `$collection.eq(index)` : Returns a **new jQuery object** containing the element at the given index (unlike `.get()`, which returns the native element).

**Syntax Rules**

1. Collections are **static**: adding a new element to the DOM after selection does not automatically add it to an existing collection.
2. The order of elements in the collection is generally DOM order, except when constructed from an array argument, in which case array order is preserved.
3. If no elements match, the collection is empty (`length === 0`). All jQuery methods safely do nothing on empty collections.
4. Negative indices in `.get()` count from the end: `.get(-1)` returns the last element.
5. `.eq()` returns a jQuery object; `.get()` and `[index]` return native DOM elements.

**Constraints and Limitations**

- Collections are not live NodeLists; they do not update automatically as the DOM changes.
- Accessing an out-of-bounds index via `.get()` returns `undefined`; via `[index]` it also returns `undefined`.
- The `.length` property is read-only; assigning to it does not truncate the collection.

### Multiple Annotated Complete Code Examples

**Example 1: Working with Collection Length and Indices**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Collections — Length and Indices</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul id="list">
    <li>First</li>
    <li>Second</li>
    <li>Third</li>
  </ul>

  <script>
    $(function () {
      var $items = $("#list li");

      // Step 1: Check collection size
      console.log("Count:", $items.length); // 3

      // Step 2: Access elements by index
      console.log("Index 0:", $items[0].textContent);   // "First"
      console.log("Index 1:", $items[1].textContent);   // "Second"
      console.log("Index 2:", $items[2].textContent);   // "Third"

      // Step 3: Negative index with .get()
      console.log("Last:", $items.get(-1).textContent); // "Third"

      // Step 4: Out-of-bounds returns undefined
      console.log("Index 99:", $items[99]);              // undefined
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Count: 3
Index 0: First
Index 1: Second
Index 2: Third
Last: Third
Index 99: undefined
```

**Why This Output Occurs**
The selector `"#list li"` matches exactly three list items. `$items.length` returns `3`. Array-style indexing returns native DOM elements whose `textContent` properties hold the expected text. `.get(-1)` counts backward from the end, returning the third element. Index `99` is out of bounds, so `undefined` is returned.

---

**Example 2: The Difference Between `.get()` and `.eq()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Element Collections — get() vs eq()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p class="note">Note 1</p>
  <p class="note">Note 2</p>
  <p class="note">Note 3</p>

  <script>
    $(function () {
      var $notes = $(".note");

      // Step 1: .get(1) returns a native DOM element
      var nativeEl = $notes.get(1);
      console.log("get(1) type:", nativeEl.constructor.name); // "HTMLParagraphElement"
      console.log("Can use .css()?", typeof nativeEl.css);    // "undefined"

      // Step 2: .eq(1) returns a jQuery object
      var $jqEl = $notes.eq(1);
      console.log("eq(1) type:", $jqEl.constructor.name);     // "jQuery"
      console.log("Can use .css()?", typeof $jqEl.css);       // "function"

      // Step 3: Both refer to the same DOM element
      console.log("Same element:", nativeEl === $jqEl[0]);    // true
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
get(1) type: HTMLParagraphElement
Can use .css()? undefined
eq(1) type: jQuery
Can use .css()? function
Same element: true
```

**Why This Output Occurs**
`.get(1)` extracts the native DOM element at index `1`, which is an `HTMLParagraphElement` and has no jQuery methods. `.eq(1)` returns a new jQuery object containing that same element, so jQuery methods are available. The final comparison confirms that both refer to the identical DOM node.

### Real-World Cases

- **Batch Styling**: `$(".card").addClass("visible")` applies a class to every card element in the collection.
- **Form Serialisation**: `$("form input").serialize()` reads values from all input elements in the collection.
- **Event Binding**: `$("button").on("click", handler)` attaches the same handler to every button in the collection.
- **Filtering**: `$("li").filter(".active")` produces a new collection containing only active list items.

### References

- Learn jQuery — The jQuery Object – https://learn.jquery.com/using-jquery-core/jquery-object/
- jQuery API — .get() – https://api.jquery.com/get/
- jQuery API — .eq() – https://api.jquery.com/eq/

---

## Core Concept 4: Implicit Iteration

### Definitions

**Core Definition**
Implicit iteration is jQuery’s automatic behaviour of looping through every element in a collection and applying a method’s operation to each one, without the developer writing an explicit loop.

**Technical Definition**
Most jQuery methods that are designed to modify elements (setters) internally iterate over the `0` to `length - 1` indices of the jQuery object and invoke the corresponding native DOM operation on each element. This behaviour is called “implicit” because the iteration is hidden from the developer. Methods that retrieve values (getters) typically operate only on the first element in the collection and return a single value.

**Beginner-Friendly Explanation**
Imagine you have a stack of sticky notes and you want to write the same message on all of them. Instead of writing on each note one by one, you use a stamp that marks every note in the stack at once. Implicit iteration is jQuery’s stamp — when you tell it to add a class or change a style, it automatically applies that change to every element you selected.

### Purposes

- To eliminate the need for manual `for` loops when applying operations to multiple elements.
- To reduce boilerplate code and improve readability.
- To ensure consistent behaviour across all matched elements.
- To allow developers to focus on *what* to do rather than *how* to loop.
- To work in harmony with method chaining, as setter methods return the jQuery object.

### Syntax Rules and Structure

**Complete General Syntax**

Implicit iteration requires no special syntax — it is the default behaviour of setter methods:

```javascript
$(selector).method(argument);
```

**Component Breakdown**

- `$(selector)` : Creates a jQuery object containing all matched elements.
- `.method(argument)` : A setter method that applies the argument to each element in the collection.
- The iteration happens internally; the developer does not write a loop.

**Syntax Rules**

1. Setter methods (those that accept an argument to modify elements) implicitly iterate over the entire collection.
2. Getter methods (those called without arguments to retrieve a value) return the value for the **first element only**, with the exception of `.text()`, which returns concatenated text from all elements.
3. Not all jQuery methods support implicit iteration. Methods that return a single value or a non-jQuery object (e.g., `.width()` without arguments) do not iterate.
4. Traversal methods such as `.children()` and `.parent()` also act on each element in the collection, producing a new collection of their results.
5. If the collection is empty, implicit iteration simply does nothing; no error is thrown.

**Constraints and Limitations**

- Implicit iteration applies the **same** operation to every element. If different elements require different operations, an explicit `.each()` loop is necessary.
- Getters do not iterate; if you need a value from every element, use `.map()` or `.each()`.
- The callback context inside `.each()` is the current DOM element, but inside implicit iteration there is no per-element callback unless the method accepts one.

### Multiple Annotated Complete Code Examples

**Example 1: Implicit Iteration with a Setter**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Implicit Iteration — Setter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background: yellow; font-weight: bold; }
  </style>
</head>
<body>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
  <p>Paragraph 3</p>

  <script>
    $(function () {
      // Step 1: Select all paragraphs
      var $paragraphs = $("p");
      console.log("Before:", $paragraphs.length); // 3

      // Step 2: Apply a setter method — implicit iteration occurs
      $paragraphs.addClass("highlight");

      // Step 3: Verify that all paragraphs received the class
      $paragraphs.each(function (index) {
        console.log("P" + index + " classes:", $(this).attr("class"));
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- All three paragraphs have a yellow background and bold text.
- Console output:
```
Before: 3
P0 classes: highlight
P1 classes: highlight
P2 classes: highlight
```

**Why This Output Occurs**
`.addClass("highlight")` is a setter method. jQuery internally loops over all three elements in the `$paragraphs` collection and adds the class to each one. The developer did not write a loop — the iteration is implicit.

---

**Example 2: Implicit Iteration Versus Explicit `.each()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Implicit Iteration — Versus .each()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="box">Box A</div>
  <div class="box">Box B</div>

  <script>
    $(function () {
      // --- Implicit iteration (preferred for uniform operations) ---
      $(".box").css("border", "2px solid red");

      // --- Explicit iteration (needed when logic varies per element) ---
      $(".box").each(function (index) {
        // Different text for each box
        $(this).text("Updated Box " + (index + 1));
      });

      // Step 3: Log results
      $(".box").each(function (i) {
        console.log("Box " + i + ": " + $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Box 0: Updated Box 1
Box 1: Updated Box 2
```

**Why This Output Occurs**
The first operation (`.css()`) uses implicit iteration to apply the same border to both boxes. The second operation requires different text for each box, so explicit `.each()` is used. The callback receives the index, allowing per-element customisation.

### Real-World Cases

- **Applying CSS Classes**: `$(".error").addClass("highlight")` highlights all error messages.
- **Disabling Form Fields**: `$("input").prop("disabled", true)` disables every input.
- **Setting Attributes**: `$("a").attr("target", "_blank")` makes all links open in new tabs.
- **Removing Elements**: `$(".temp").remove()` removes all temporary elements at once.
- **Adding Event Listeners**: `$("button").on("click", handleClick)` binds the same handler to every button.

### References

- jQuery API — .each() – https://api.jquery.com/each/
- Learn jQuery — Iterating over jQuery and non-jQuery Objects – https://learn.jquery.com/using-jquery-core/iterating/
- Learn jQuery — Working with Selections – https://learn.jquery.com/using-jquery-core/working-with-selections/

---

## Core Concept 5: Method Chaining

### Definitions

**Core Definition**
Method chaining is a programming technique in which multiple method calls are connected in a single expression, each call operating on the result of the previous call.

**Technical Definition**
In jQuery, method chaining is enabled by the convention that most setter and traversal methods return a jQuery object. Because each method returns a jQuery object, the next method can be called immediately on that returned object. This creates a fluent interface. The `.end()` method allows the chain to “rewind” to a previous jQuery object in the chain, effectively restoring an earlier selection.

**Beginner-Friendly Explanation**
Think of method chaining like a sentence: “Find the heading, change its text, then add a class, then fade it in.” Instead of writing three separate statements, you write one continuous line of code that reads naturally from left to right. Each method “hands off” the result to the next method.

### Purposes

- To write concise, readable code by eliminating intermediate variables.
- To express a sequence of DOM operations as a single fluent statement.
- To maintain a clear connection between related operations on the same selection.
- To reduce the risk of inconsistent variable names and stale references.
- To allow `.end()` to recover earlier selections within a chain.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector)
  .method1()
  .method2(argument)
  .method3();
```

**Component Breakdown**

- `$(selector)` : The initial jQuery object.
- `.method1()` : A method that returns a jQuery object (usually a setter or traversal method).
- `.method2(argument)` : The next method in the chain, called on the result of `method1`.
- `.method3()` : The final method; if it returns a jQuery object, the chain can continue.

**Syntax Rules**

1. Only methods that return a jQuery object can be chained. Getter methods (e.g., `.html()` without arguments, `.val()`) break the chain because they return strings, numbers, or native elements.
2. `.end()` returns the jQuery object from the previous step in the chain, allowing recovery of an earlier selection.
3. Chaining can span multiple lines for readability; each `.method()` on a new line is valid JavaScript.
4. Excessive chaining can reduce readability and make debugging harder; break long chains when clarity suffers.

**Constraints and Limitations**

- A chain cannot continue after a getter method unless the getter result is re-wrapped in `$()`.
- `.end()` only works if the chain has previous selections; calling it at the start of a chain returns an empty jQuery object.
- Some methods (e.g., `.animate()` without a callback) return a jQuery object; others (e.g., `.get()`) do not.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Method Chaining**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Method Chaining — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background: yellow; }
    .bold { font-weight: bold; }
  </style>
</head>
<body>
  <h1>Hello World</h1>

  <script>
    $(function () {
      // Step 1: Chain three methods in a single statement
      $("h1")
        .text("Chained!")          // setter: returns jQuery object
        .addClass("highlight")     // setter: returns jQuery object
        .addClass("bold");         // setter: returns jQuery object

      // Step 2: Verify the result
      console.log("Text:", $("h1").text());
      console.log("Classes:", $("h1").attr("class"));
    });
  </script>
</body>
</html>
```

**Expected Output**
- The heading text becomes “Chained!” with a yellow background and bold font.
- Console output:
```
Text: Chained!
Classes: highlight bold
```

**Why This Output Occurs**
`.text("Chained!")` sets the text and returns the same jQuery object. `.addClass("highlight")` adds the class and returns the object again. `.addClass("bold")` adds the second class. All three operations occur on the same `<h1>` element in sequence.

---

**Example 2: Chaining with `.end()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Method Chaining — .end()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="content">
    <h3>Heading 1</h3>
    <h3>Heading 2</h3>
    <h3>Heading 3</h3>
  </div>

  <script>
    $(function () {
      // Step 1: Chain selection, traversal, and .end()
      $("#content")
        .find("h3")          // new selection: all h3 elements
        .eq(2)               // new selection: the third h3
        .text("Third!")      // set text on third h3
        .end()               // revert to all h3 elements
        .eq(0)               // new selection: the first h3
        .text("First!");     // set text on first h3

      // Step 2: Log all h3 texts
      $("#content h3").each(function (i) {
        console.log("H3 " + i + ": " + $(this).text());
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
H3 0: First!
H3 1: Heading 2
H3 2: Third!
```

**Why This Output Occurs**
`.find("h3")` changes the selection from the `#content` div to its `h3` children. `.eq(2)` reduces the selection to the third heading, and `.text("Third!")` changes its text. `.end()` rewinds the selection back to all three `h3` elements. `.eq(0)` then selects the first heading, and `.text("First!")` changes its text. The second heading is untouched.

---

**Example 3: Chaining Broken by a Getter**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Method Chaining — Broken by Getter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p>Hello</p>

  <script>
    $(function () {
      // This chain works: .text("New") is a setter
      $("p").text("New").addClass("done");
      console.log("Works:", $("p").text()); // "New"

      // This chain FAILS: .text() without arguments is a getter
      // $("p").text().addClass("fail"); // TypeError: addClass is not a function

      // Correct approach: wrap the getter result or use a separate statement
      var text = $("p").text();          // getter returns a string
      $("p").addClass("success");        // start a new chain

      console.log("Text value:", text);  // "New"
      console.log("Classes:", $("p").attr("class")); // "done success"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Works: New
Text value: New
Classes: done success
```

**Why This Output Occurs**
`.text("New")` is a setter and returns a jQuery object, so `.addClass("done")` can be chained. `.text()` without arguments is a getter that returns a string; calling `.addClass()` on a string throws a `TypeError`. The correct approach is to retrieve the value and then start a new chain.

### Real-World Cases

- **Form Styling**: `$("input").val("").addClass("cleared").focus()` clears, styles, and focuses inputs in one statement.
- **Animation Sequences**: `$(".panel").slideUp().delay(200).fadeIn()` chains animations.
- **DOM Construction**: `$("<div>").addClass("card").append($("<p>").text("Hello")).appendTo("body")` builds and inserts an element.
- **Event Setup**: `$("button").addClass("btn").on("click", handler)` styles and binds in one chain.

### References

- Learn jQuery — Working with Selections – https://learn.jquery.com/using-jquery-core/working-with-selections/
- jQuery API — .end() – https://api.jquery.com/end/
- jQuery API — Traversing – https://api.jquery.com/category/traversing/

---

## Core Concept 6: Native DOM Element versus jQuery Object

### Definitions

**Core Definition**
A native DOM element is a browser-provided JavaScript object representing an HTML element, while a jQuery object is a wrapper created by jQuery that contains one or more native DOM elements and adds jQuery’s API on top.

**Technical Definition**
A native DOM element is an instance of a class such as `HTMLDivElement`, `HTMLParagraphElement`, or `HTMLInputElement`. It exposes browser-native properties (`textContent`, `innerHTML`, `className`, `style`) and methods (`appendChild()`, `setAttribute()`, `addEventListener()`). A jQuery object is an instance of `jQuery` that holds references to these native elements at numeric indices. It does **not** inherit from `HTMLElement`; therefore, native properties and methods are not directly available on the jQuery object itself.

**Beginner-Friendly Explanation**
A native DOM element is like a raw piece of wood — it has its own natural properties (grain, colour, hardness) but no built-in tools for shaping it. A jQuery object is like putting that wood into a carpenter’s workshop — the workshop has saws, sanders, and drills (jQuery methods) that can shape the wood in ways the raw wood alone cannot.

### Purposes

- To understand when to use jQuery’s convenience methods versus native DOM APIs.
- To avoid errors caused by calling native methods on jQuery objects or vice versa.
- To enable interoperability with third-party libraries that expect native DOM elements.
- To optimise performance by using native methods when jQuery’s abstraction is unnecessary.
- To correctly convert between the two representations.

### Syntax Rules and Structure

**Comparison of Native DOM and jQuery Object**

| Aspect | Native DOM Element | jQuery Object |
|---|---|---|
| Obtained via | `document.getElementById()`, `document.querySelector()` | `$()`, `$(selector)` |
| Type | `HTMLDivElement`, etc. | `jQuery` |
| Properties | `textContent`, `innerHTML`, `className`, `style`, `value` | `.length`, `[index]` |
| Methods | `appendChild()`, `setAttribute()`, `addEventListener()` | `.css()`, `.addClass()`, `.on()`, `.append()` |
| Array-like | No | Yes (numeric indices, `.length`) |
| Implicit iteration | No | Yes |
| Chaining | No | Yes |

**Syntax Rules**

1. A native DOM element cannot call jQuery methods (e.g., `element.css()` is `undefined`).
2. A jQuery object cannot call native DOM methods directly (e.g., `$element.appendChild()` is `undefined`).
3. A jQuery object can access its underlying native elements via `[index]` or `.get(index)`.
4. A native DOM element can be wrapped in a jQuery object via `$(element)`.

**Constraints and Limitations**

- jQuery objects do **not** extend native DOM elements; they wrap them.
- Passing a jQuery object to a native method that expects a DOM element will fail.
- Native DOM collections (e.g., `NodeList` from `querySelectorAll`) are not jQuery objects and must be converted with `$()`.

### Multiple Annotated Complete Code Examples

**Example 1: Native DOM Properties vs jQuery Methods**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Native DOM vs jQuery Object</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="demo">Original text</div>

  <script>
    $(function () {
      // Step 1: Obtain a native DOM element
      var nativeDiv = document.getElementById("demo");
      console.log("Native type:", nativeDiv.constructor.name); // "HTMLDivElement"

      // Step 2: Native properties and methods work
      nativeDiv.textContent = "Changed by native DOM";
      console.log("Native textContent:", nativeDiv.textContent);

      // Step 3: Obtain a jQuery object wrapping the same element
      var $div = $("#demo");
      console.log("jQuery type:", $div.constructor.name); // "jQuery"

      // Step 4: jQuery methods work
      $div.css("color", "red");
      console.log("jQuery text:", $div.text()); // "Changed by native DOM"

      // Step 5: Demonstrate incompatibility
      console.log("nativeDiv.css:", typeof nativeDiv.css); // "undefined"
      console.log("$div.appendChild:", typeof $div.appendChild); // "undefined"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Native type: HTMLDivElement
Native textContent: Changed by native DOM
jQuery type: jQuery
jQuery text: Changed by native DOM
nativeDiv.css: undefined
$div.appendChild: undefined
```

**Why This Output Occurs**
`document.getElementById()` returns a native `HTMLDivElement` with native properties like `textContent`. `$("#demo")` returns a jQuery object that wraps the same element. The jQuery object has `.css()` and `.text()` but not `.appendChild()`. The native element has `.appendChild()` but not `.css()`. Each API is accessible only through the appropriate object type.

---

**Example 2: Mixing Native and jQuery APIs**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Mixing Native and jQuery APIs</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="btn">Click me</button>

  <script>
    $(function () {
      // Step 1: Native event listener on the button
      var btn = document.getElementById("btn");
      btn.addEventListener("click", function () {
        // Step 2: Use jQuery inside the native callback
        $(this).css("background", "lightgreen");

        // Step 3: Use native DOM API on this
        this.textContent = "Clicked!";
      });

      // Step 4: jQuery event listener on the same button
      $("#btn").on("mouseenter", function () {
        $(this).css("cursor", "pointer");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking the button changes its background to light green and its text to “Clicked!”.
- Hovering over the button changes the cursor to a pointer.

**Why This Output Occurs**
Inside the native `addEventListener` callback, `this` is the native button element. `$(this)` wraps it in a jQuery object for `.css()`, while `this.textContent` uses the native DOM property directly. Both APIs can coexist in the same handler.

### Real-World Cases

- **Third-Party Libraries**: Many libraries (e.g., charting libraries) expect native DOM elements; use `$element.get(0)` to pass them.
- **Performance-Critical Loops**: Native properties like `element.textContent` are faster than jQuery’s `.text()` in tight loops.
- **Custom Elements**: Web Components expose native APIs; wrap them only when jQuery convenience is needed.
- **Legacy Code Integration**: Existing native DOM code can be gradually migrated to jQuery by wrapping elements as needed.

### References

- Learn jQuery — The jQuery Object – https://learn.jquery.com/using-jquery-core/jquery-object/
- MDN Web Docs — Element – https://developer.mozilla.org/en-US/docs/Web/API/Element
- jQuery API — Types: Element – https://api.jquery.com/Types/#Element

---

## Core Concept 7: Converting Between Native DOM and jQuery Objects

### Definitions

**Core Definition**
Conversion between native DOM and jQuery objects is the process of transforming one representation into the other so that the appropriate API can be used.

**Technical Definition**
- **Native → jQuery**: Wrap a native DOM element (or an array of elements) in the `$()` factory function. The resulting jQuery object contains references to the original elements.
- **jQuery → Native**: Access the underlying DOM element(s) via array-style indexing (`$obj[0]`) or the `.get(index)` method. `.get()` with no arguments returns a true JavaScript array of all DOM elements.

**Beginner-Friendly Explanation**
Converting is like taking items out of a toolbox (jQuery object) and putting them on a workbench (native DOM), or vice versa. If you need a screwdriver that only exists on the workbench, you take the item out of the toolbox. If you need the toolbox’s special tools, you put the item back in.

### Purposes

- To access native DOM properties and methods that jQuery does not expose.
- To pass DOM elements to functions or libraries that expect native references.
- To wrap native elements in jQuery when jQuery methods are needed.
- To iterate over DOM elements as a true array using native array methods.
- To compare element identity with `===` using native references.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Native DOM → jQuery:**
```javascript
var $jqueryObj = $(nativeElement);
var $jqueryObj = $(nativeElementArray);
```

**jQuery → Native (single element):**
```javascript
var nativeElement = $jqueryObj[0];        // array-style access
var nativeElement = $jqueryObj.get(0);    // .get() method
var lastElement   = $jqueryObj.get(-1);   // negative index
```

**jQuery → Native (all elements as an array):**
```javascript
var nativeArray = $jqueryObj.get();       // true JavaScript Array
```

**Component Breakdown**

- `$(nativeElement)` : Wraps a single native DOM element in a new jQuery object.
- `$(nativeElementArray)` : Wraps an array of native elements; elements are copied in array order.
- `$jqueryObj[0]` : Fastest method; returns the native element at index `0`.
- `$jqueryObj.get(0)` : Equivalent to `[0]` but slightly slower; supports negative indices.
- `$jqueryObj.get()` : Returns a true array of all native elements.

**Syntax Rules**

1. Wrapping a native element with `$()` creates a **new** jQuery object each time; it does not mutate the native element.
2. Array-style access `[index]` is the fastest conversion method but does not support negative indices.
3. `.get(index)` supports negative indices and returns `undefined` if out of bounds.
4. `.get()` with no arguments returns a true `Array`, not a jQuery object.
5. Passing a jQuery object to `$()` creates a clone that references the same DOM elements.

**Constraints and Limitations**

- Converting a jQuery object to a native element loses access to jQuery methods until it is re-wrapped.
- Native element arrays created by `.get()` are static; they do not update if the DOM changes.
- `$jqueryObj[0]` and `$jqueryObj.get(0)` return the **same** native element; they differ only in syntax and negative-index support.

### Multiple Annotated Complete Code Examples

**Example 1: Native DOM → jQuery → Native (Round Trip)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Conversion — Round Trip</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p id="message">Hello</p>

  <script>
    $(function () {
      // Step 1: Get a native DOM element
      var nativeP = document.getElementById("message");
      console.log("Native type:", nativeP.constructor.name); // "HTMLParagraphElement"

      // Step 2: Convert to jQuery object
      var $p = $(nativeP);
      console.log("jQuery length:", $p.length); // 1

      // Step 3: Use a jQuery method
      $p.css("color", "blue");

      // Step 4: Convert back to native DOM
      var backToNative = $p.get(0);
      console.log("Back to native:", backToNative.constructor.name); // "HTMLParagraphElement"

      // Step 5: Verify identity
      console.log("Same element:", backToNative === nativeP); // true
    });
  </script>
</body>
</html>
```

**Expected Output**
- The paragraph text turns blue.
- Console output:
```
Native type: HTMLParagraphElement
jQuery length: 1
Back to native: HTMLParagraphElement
Same element: true
```

**Why This Output Occurs**
`$(nativeP)` wraps the native element in a jQuery object. After calling `.css()`, `.get(0)` extracts the original native element. The strict equality check confirms that the round-trip returns the identical DOM node.

---

**Example 2: jQuery → Native Array with `.get()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Conversion — .get() Array</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li>Apple</li>
    <li>Banana</li>
    <li>Cherry</li>
  </ul>

  <script>
    $(function () {
      var $fruits = $("li");

      // Step 1: Convert all elements to a true array
      var fruitArray = $fruits.get();
      console.log("Is Array:", Array.isArray(fruitArray)); // true
      console.log("Length:", fruitArray.length);           // 3

      // Step 2: Use native array methods on the result
      fruitArray.forEach(function (element, index) {
        console.log(index + ": " + element.textContent);
      });

      // Step 3: Use native array methods to create a new array
      var texts = fruitArray.map(function (el) {
        return el.textContent.toUpperCase();
      });
      console.log("Uppercase:", texts); // ["APPLE", "BANANA", "CHERRY"]
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Is Array: true
Length: 3
0: Apple
1: Banana
2: Cherry
Uppercase: ["APPLE", "BANANA", "CHERRY"]
```

**Why This Output Occurs**
`.get()` with no arguments returns a true JavaScript array of the three DOM elements. Because it is a real array, native methods like `.forEach()` and `.map()` work directly.

---

**Example 3: Passing a jQuery Object to a Native API**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Conversion — Native API</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="parent"></div>

  <script>
    $(function () {
      // Step 1: Create a new element with jQuery
      var $newChild = $("<span>New child</span>");

      // Step 2: Get the native parent element
      var nativeParent = document.getElementById("parent");

      // Step 3: Pass the native element from the jQuery object
      nativeParent.appendChild($newChild.get(0));

      // Step 4: Verify
      console.log("Parent HTML:", nativeParent.innerHTML);
      console.log("Child count:", nativeParent.children.length); // 1
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Parent HTML: <span>New child</span>
Child count: 1
```

**Why This Output Occurs**
`$newChild` is a jQuery object. `nativeParent.appendChild()` expects a native DOM node, so `$newChild.get(0)` extracts the underlying `<span>` element and passes it to the native method. Without the conversion, passing a jQuery object to `appendChild()` would throw an error.

### Real-World Cases

- **Chart Libraries**: Pass `$canvas.get(0)` to Chart.js or D3.js, which expect native `<canvas>` elements.
- **File API**: Use `$fileInput.get(0).files` to access the native `FileList` from a jQuery-wrapped file input.
- **Focus Management**: `$input.get(0).focus()` calls the native `focus()` method when jQuery’s `.focus()` does not trigger certain native behaviours.
- **Intersection Observer**: Wrap elements in jQuery for selection, then convert to native elements for the `IntersectionObserver` API.
- **Web Components**: Pass `$host.get(0)` to custom element methods that expect native DOM references.

### References

- Learn jQuery — How do I pull a native DOM element from a jQuery object? – https://learn.jquery.com/using-jquery-core/faq/how-do-i-pull-a-native-dom-element-from-a-jquery-object/
- jQuery API — .get() – https://api.jquery.com/get/
- jQuery API — jQuery() – https://api.jquery.com/jQuery/

---

## Summary Table: Core Concepts at a Glance

| Concept | Key Idea | Primary Use | Conversion Direction |
|---|---|---|---|
| `$` Function | Factory for jQuery objects | Selecting, wrapping, creating | — |
| jQuery Object | Array-like wrapper with jQuery API | Batch operations, chaining | — |
| Element Collections | The DOM elements inside a jQuery object | Grouping elements for batch operations | — |
| Implicit Iteration | Automatic looping by setter methods | Applying same operation to many elements | — |
| Method Chaining | Fluent interface via returned jQuery objects | Concise multi-step operations | — |
| Native DOM vs jQuery | Two different APIs for DOM elements | Knowing which methods are available | — |
| Conversion | Moving between native and jQuery representations | Interoperability with native APIs | `$(el)` and `.get()` / `[index]` |

---

## General References

- jQuery Official Website – https://jquery.com/
- jQuery API Documentation – https://api.jquery.com/
- jQuery Learning Center – https://learn.jquery.com/
- MDN Web Docs — Document Object Model (DOM) – https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model
- MDN Web Docs — JavaScript Guide – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide
- W3C DOM Specification – https://www.w3.org/TR/DOM-Level-2-Core/
- Smashing Magazine — The Mystery Of The jQuery Object – https://www.smashingmagazine.com/2014/05/mystery-jquery-object-syntax-basic-introduction/
- O'Reilly — Learning jQuery 3 (Fifth Edition) – https://www.oreilly.com/library/view/learning-jquery-3/9781785882982/