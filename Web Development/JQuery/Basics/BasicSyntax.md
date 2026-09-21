# Basic jQuery Syntax: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Basic jQuery syntax is the foundational pattern by which developers interact with the jQuery library: selecting HTML elements and performing actions on them through a concise, fluent interface built around the `$()` function and chainable methods.

**Technical Definition**
Basic jQuery syntax follows the pattern `$(selector).method(arguments)`. The `$` (or `jQuery`) function is a factory that accepts a CSS selector string, a DOM element, an HTML string, or a function, and returns a jQuery object containing a set of matched DOM elements. Methods are invoked on that jQuery object, and most setter methods return the same jQuery object (or a new one) to enable method chaining. Arguments configure the behaviour of each method, and callback functions provide deferred execution for asynchronous operations and event handling.

**Beginner-Friendly Explanation**
Think of jQuery syntax as giving instructions to a robot. You first tell the robot *what* to look at (the selector), then you tell it *what to do* (the method). For example, `$("p").hide()` means “Find every paragraph on the page, and hide them.” The dollar sign `$` is just a short way of saying “jQuery, please do this.”

### Key Characteristics

- **Concise and Readable**: The `$(selector).method()` pattern reads almost like English, reducing boilerplate compared to native DOM APIs.
- **CSS-Compatible Selectors**: jQuery supports nearly all CSS selector expressions, making element selection intuitive for anyone familiar with CSS.
- **Chainable Methods**: Most methods return a jQuery object, allowing multiple operations to be strung together in a single statement.
- **Implicit Iteration**: Methods automatically loop through every element in the matched set, so developers do not write explicit loops.
- **Cross-Browser Consistency**: The same jQuery syntax works across all supported browsers, abstracting away vendor-specific DOM inconsistencies.
- **Callback-Driven Asynchrony**: Callback functions enable deferred execution for events, AJAX, and animations, ensuring code runs at the right time.

### Prerequisites

- Basic knowledge of HTML and CSS selectors.
- Familiarity with JavaScript fundamentals (variables, functions, objects, arrays).
- Understanding of the DOM (Document Object Model) as a tree of HTML elements.
- A text editor and a web browser with developer tools for testing.
- The jQuery library included in the page via a `<script>` tag or CDN link.

### Related Programming Areas

- **DOM Manipulation**: Creating, reading, updating, and deleting HTML elements.
- **Event Handling**: Attaching, triggering, and removing event listeners.
- **AJAX and Asynchronous Programming**: Making HTTP requests and handling responses via callbacks and Promises.
- **Animation and Effects**: Applying built-in animations (`fadeIn`, `slideUp`, `animate`) with optional callbacks.
- **Plugin Development**: Extending `$.fn` to add reusable methods that follow jQuery's chaining conventions.

### Core Concepts / Features

1. General Pattern (`$(selector).method()`)
2. Selector Expression
3. Method Invocation
4. Arguments
5. Return Values
6. Chaining
7. Callback Functions

---

## Core Concept 1: General Pattern (`$(selector).method()`)

### Definitions

**Core Definition**
The general pattern of jQuery syntax is `$(selector).method()`, where `$` is the jQuery function, `selector` identifies one or more HTML elements, and `method()` is the action to perform on those elements.

**Technical Definition**
The `$` symbol is an alias for the `jQuery` function, which is exposed as a property of the global `window` object. When called with a CSS selector string, `$()` traverses the DOM and returns a jQuery object containing all matched elements. The `.method()` portion is a call to a function defined on `jQuery.prototype` (aliased as `$.fn`), and it operates on the collection of elements inside the jQuery object.

**Beginner-Friendly Explanation**
This pattern is like a sentence: “Find the thing (selector), then do something to it (method).” The `$` is the verb “find,” the parentheses contain “what to find,” and the dot introduces “what to do.” For example, `$("div").hide()` means “Find all `<div>` elements and hide them.”

### Purposes

- To provide a uniform, readable syntax for selecting elements and applying actions.
- To abstract away cross-browser DOM inconsistencies.
- To enable implicit iteration so a single method call affects all matched elements.
- To support method chaining for concise, multi-step operations.
- To serve as the entry point for virtually all jQuery operations.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).method();
$(selector).method(argument1, argument2, ...);
```

**Component Breakdown**

- `$` : The jQuery function (alias for `jQuery`). Defined as a property of `window`.
- `(selector)` : The argument passed to `$()`. Can be a CSS selector string, a DOM element, an HTML string, or a function.
- `.` : The member access operator.
- `method()` : A jQuery method called on the returned jQuery object. Parentheses contain any arguments the method accepts.

**Syntax Rules**

1. `$` and `jQuery` are interchangeable; both refer to the same function.
2. The selector must be enclosed in parentheses and, if a string, in quotes (e.g., `$("p")`, not `$p`).
3. Methods are called using dot notation after the jQuery object.
4. Semicolons terminate statements, though JavaScript's automatic semicolon insertion often handles this.
5. The pattern can be extended with chaining: `$(selector).method1().method2()`.

**Constraints and Limitations**

- If no elements match the selector, the jQuery object is empty (`length === 0`), and method calls silently do nothing (no error is thrown).
- The `$` alias may be released in `noConflict()` mode if another library uses `$`.
- Not all methods return a jQuery object; getter methods break the chain.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Selection and Action**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>General Pattern — Hide Paragraphs</title>
  <!-- Step 1: Include jQuery from the official CDN -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
  <p>Paragraph 3</p>

  <script>
    // Step 2: Wait for the DOM to be ready
    $(function () {
      // Step 3: Select all <p> elements and hide them
      $("p").hide();

      // Step 4: Verify that the paragraphs are hidden
      console.log("Hidden paragraphs:", $("p:hidden").length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output**
- All three paragraphs become invisible on the page.
- Console output: 
```
Hidden paragraphs: 3
```

**Why This Output Occurs**
`$("p")` selects all three `<p>` elements and returns a jQuery object. `.hide()` is called on that object and implicitly iterates over all three elements, setting their `display` style to `none`. The `:hidden` selector confirms that all three are now hidden.

---

**Example 2: Selection with a Class and CSS Change**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>General Pattern — Style Change</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div class="box">Box A</div>
  <div class="box">Box B</div>
  <div>Not a box</div>

  <script>
    $(function () {
      // Step 1: Select elements with class "box" and change their border
      $(".box").css("border", "3px solid blue");

      // Step 2: Log how many elements were affected
      console.log("Styled boxes:", $(".box").length); // 2
    });
  </script>
</body>
</html>
```

**Expected Output**
- The two `<div class="box">` elements receive a blue border.
- The third `<div>` remains unchanged.
- Console output: 
```
Styled boxes: 2
```

**Why This Output Occurs**
`$(".box")` selects only the elements with the `box` class. The `.css()` method applies the border style to each matched element. The third `<div>` does not have the `box` class and is therefore not selected.

### Real-World Cases

- **Hiding Loading Spinners**: `$(".spinner").hide()` after an AJAX request completes.
- **Highlighting Form Errors**: `$(".error").addClass("highlight")` to visually flag invalid fields.
- **Disabling Buttons During Submission**: `$("button[type=submit]").prop("disabled", true)` to prevent double submissions.
- **Animating Page Sections**: `$("#panel").slideDown()` to reveal a hidden panel.

### References

- W3Schools — jQuery Syntax – https://www.w3schools.com/jquery/jquery_syntax.asp
- jQuery Learning Center — How jQuery Works – https://learn.jquery.com/about-jquery/how-jquery-works/
- jQuery API — jQuery() – https://api.jquery.com/jQuery/

---

## Core Concept 2: Selector Expression

### Definitions

**Core Definition**
A selector expression is the string argument passed to `$()` that identifies which HTML elements to select from the DOM. jQuery supports nearly all CSS selector syntax plus its own custom selectors.

**Technical Definition**
The selector expression is a string that jQuery parses using its internal `Sizzle` selector engine (or native `querySelectorAll` when possible in modern browsers). Selectors can be classified into several categories: basic CSS selectors (element, ID, class, universal), hierarchy selectors (descendant, child, sibling), attribute selectors, basic filters (`:first`, `:last`, `:eq`), content filters (`:contains`, `:empty`), visibility filters (`:hidden`, `:visible`), and form-related selectors (`:input`, `:checked`, `:selected`).

**Beginner-Friendly Explanation**
A selector expression is like a description you give to a search engine. You can describe elements by their tag name (`"div"`), their ID (`"#header"`), their class (`".intro"`), their attributes (`"[href]"`), or even their position in the DOM (`"ul > li:first"`). jQuery uses this description to find every element that matches.

### Purposes

- To precisely identify which DOM elements to operate on.
- To allow complex selections using CSS-compatible syntax without manual DOM traversal.
- To support filtering by position, content, attributes, and form state.
- To combine multiple selectors into a single query for efficiency.
- To enable context-based selection (searching within a specific parent element).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selectorString)
$(selectorString, context)
```

**Component Breakdown**

- `selectorString` : A string containing a CSS selector, jQuery custom selector, or a combination thereof.
- `context` : (Optional) A DOM element, document, or jQuery object to use as the search scope.

**Selector Categories**

| Category | Syntax | Example | Description |
|---|---|---|---|
| Element | `"element"` | `$("p")` | All `<p>` elements |
| ID | `"#id"` | `$("#header")` | The element with `id="header"` |
| Class | `".class"` | `$(".intro")` | All elements with `class="intro"` |
| Universal | `"*"` | `$("*")` | All elements |
| Multiple | `"sel1, sel2"` | `$("h1, h2")` | All `<h1>` and `<h2>` elements |
| Descendant | `"ancestor descendant"` | `$("div p")` | All `<p>` inside `<div>` |
| Child | `"parent > child"` | `$("ul > li")` | Direct `<li>` children of `<ul>` |
| Attribute | `"[attr]"` | `$("[href]")` | Elements with an `href` attribute |
| Attribute Equals | `"[attr='val']"` | `$("[type='text']")` | Elements where `type="text"` |
| First | `":first"` | `$("li:first")` | The first matched `<li>` |
| Last | `":last"` | `$("li:last")` | The last matched `<li>` |
| Eq | `":eq(n)"` | `$("li:eq(2)")` | The third matched `<li>` (zero-indexed) |
| Contains | `":contains(text)"` | `$("p:contains('Hello')")` | Paragraphs containing "Hello" |
| Hidden | `":hidden"` | `$("div:hidden")` | All hidden `<div>` elements |
| Visible | `":visible"` | `$("div:visible")` | All visible `<div>` elements |
| Checked | `":checked"` | `$("input:checked")` | All checked checkboxes/radios |

**Syntax Rules**

1. Selector strings must be enclosed in quotes when passed to `$()`.
2. Multiple selectors can be combined with commas: `$("h1, h2, h3")`.
3. Whitespace in descendant selectors matters: `"div p"` selects descendants, while `"div > p"` selects direct children.
4. Attribute values containing special characters should be quoted: `$("[data-id='123']")`.
5. jQuery custom selectors (e.g., `:eq()`, `:contains()`) are not part of the CSS standard and are implemented by jQuery's selector engine.

**Constraints and Limitations**

- jQuery custom selectors (e.g., `:eq()`) cannot be used inside `document.querySelectorAll()`.
- Performance: complex selectors, especially those using custom filters, are slower than simple ID or class selectors.
- The `:hidden` and `:visible` selectors depend on the element's computed style and layout, which may cause performance overhead.
- Context-based selection is equivalent to `$(context).find(selector)`, which may be slower than a single compound selector.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Selectors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Selectors — Basic Types</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <h1 id="title">Welcome</h1>
  <p class="intro">Intro paragraph</p>
  <p>Regular paragraph</p>
  <a href="https://example.com">Link</a>

  <script>
    $(function () {
      // Step 1: ID selector
      console.log("ID:", $("#title").text()); // "Welcome"

      // Step 2: Class selector
      console.log("Class count:", $(".intro").length); // 1

      // Step 3: Element selector
      console.log("P count:", $("p").length); // 2

      // Step 4: Attribute selector
      console.log("Links with href:", $("[href]").length); // 1

      // Step 5: Multiple selector
      console.log("H1 + P count:", $("h1, p").length); // 3
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
ID: Welcome
Class count: 1
P count: 2
Links with href: 1
H1 + P count: 3
```

**Why This Output Occurs**
Each selector targets a specific subset of elements. `#title` matches the `<h1>` by its ID. `.intro` matches the paragraph with that class. `"p"` matches both paragraphs. `"[href]"` matches the anchor. `"h1, p"` combines both element types, matching three elements total.

---

**Example 2: Hierarchy and Filter Selectors**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Selectors — Hierarchy and Filters</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul id="fruits">
    <li>Apple</li>
    <li>Banana</li>
    <li>Cherry</li>
    <li>Date</li>
  </ul>
  <p>Outside paragraph</p>

  <script>
    $(function () {
      // Step 1: Descendant selector
      console.log("Descendants:", $("#fruits li").length); // 4

      // Step 2: First and last filters
      console.log("First:", $("#fruits li:first").text()); // "Apple"
      console.log("Last:", $("#fruits li:last").text());   // "Date"

      // Step 3: eq() filter (zero-indexed)
      console.log("Index 1:", $("#fruits li:eq(1)").text()); // "Banana"

      // Step 4: Contains filter
      console.log("Contains 'Ch':", $("li:contains('Ch')").text()); // "Cherry"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Descendants: 4
First: Apple
Last: Date
Index 1: Banana
Contains 'Ch': Cherry
```

**Why This Output Occurs**
The descendant selector `"#fruits li"` finds all four list items inside the fruit list. `:first` and `:last` select the first and last items. `:eq(1)` selects the element at index 1 (the second item, "Banana"). `:contains('Ch')` finds the list item whose text contains "Ch" ("Cherry").

### Real-World Cases

- **Navigation Menus**: `$("nav > ul > li")` selects top-level menu items.
- **Form Validation**: `$("input[type='email']")` selects email fields for validation.
- **Table Row Selection**: `$("table tr:even")` selects even rows for zebra striping.
- **Dynamic Content**: `$(".comment:contains('spam')")` finds comments containing specific text.
- **Tab Interfaces**: `$(".tab:visible")` finds the currently visible tab panel.

### References

- jQuery API — Selectors – https://api.jquery.com/category/selectors/
- jQuery API — Basic Selectors – https://api.jquery.com/category/selectors/basic-css-selectors/
- MDN Web Docs — CSS Selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors
- W3Schools — jQuery Selectors – https://www.w3schools.com/jquery/jquery_selectors.asp

---

## Core Concept 3: Method Invocation

### Definitions

**Core Definition**
Method invocation is the act of calling a jQuery method on a jQuery object using dot notation, causing the method to execute its operation on the elements in the object's collection.

**Technical Definition**
In jQuery, methods are functions defined on `jQuery.prototype` (aliased as `$.fn`). When invoked as `$(selector).method()`, the method's `this` context is set to the jQuery object on which it was called. Most jQuery methods use `this.each()` internally to apply their operation to every element in the collection. Setter methods typically return the jQuery object itself, while getter methods return the underlying value.

**Beginner-Friendly Explanation**
Method invocation is simply “telling the robot what to do.” After you have selected elements with `$()`, you add a dot and the name of the action you want to perform. For example, `.hide()` tells jQuery to hide the selected elements. `.css("color", "red")` tells jQuery to change their colour.

### Purposes

- To perform a specific operation on the selected DOM elements.
- To configure the operation through arguments passed to the method.
- To enable implicit iteration over all elements in the collection.
- To support chaining by returning the jQuery object from setter methods.
- To provide a consistent API for DOM manipulation, events, effects, and AJAX.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).method();
$(selector).method(argument1);
$(selector).method(argument1, argument2);
$(selector).method(function(index, value) { ... });
```

**Component Breakdown**

- `$(selector)` : The jQuery object containing matched elements.
- `.method` : The name of the jQuery method to invoke (e.g., `.css`, `.addClass`, `.on`).
- `(arguments)` : Optional arguments that configure the method's behaviour.

**Syntax Rules**

1. Method names are case-sensitive: `.addClass()` is valid; `.addclass()` is not.
2. Arguments are passed inside parentheses, separated by commas.
3. Methods without arguments are still called with empty parentheses: `.hide()`.
4. Some methods accept callback functions as arguments: `.each(function(index, element) { ... })`.
5. Methods can be chained: `$(selector).method1().method2()`.

**Constraints and Limitations**

- Calling a jQuery method on a non-jQuery object (e.g., a native DOM element or a string) throws an error.
- Getter methods (e.g., `.text()` without arguments, `.val()`) break the chain because they return strings, numbers, or native elements.
- Some methods have version-specific behaviour; always check the API documentation for compatibility.

### Multiple Annotated Complete Code Examples

**Example 1: Common Method Invocations**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Method Invocation — Common Methods</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="target">Original content</div>
  <button id="btn">Click me</button>

  <script>
    $(function () {
      // Step 1: .text() — set text content (setter)
      $("#target").text("Updated content");
      console.log($("#target").text()); // "Updated content"

      // Step 2: .css() — apply a style (setter)
      $("#target").css("color", "green");
      console.log($("#target").css("color")); // "rgb(0, 128, 0)"

      // Step 3: .addClass() — add a CSS class (setter)
      $("#target").addClass("active");
      console.log($("#target").attr("class")); // "active"

      // Step 4: .on() — attach an event handler
      $("#btn").on("click", function () {
        $("#target").toggle(); // toggle visibility
      });

      // Step 5: .attr() — getter (returns first element's attribute)
      console.log($("#btn").attr("id")); // "btn"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The div text changes to “Updated content,” turns green, and gains the `active` class.
- Clicking the button toggles the div's visibility.
- Console output:
```
Updated content
rgb(0, 128, 0)
active
btn
```

**Why This Output Occurs**
Each method invocation performs a specific operation. `.text("Updated content")` sets the text and returns the jQuery object (setter). `.css("color", "green")` sets the colour. `.addClass("active")` adds the class. `.on("click", ...)` attaches an event listener. `.attr("id")` without a value is a getter that returns the first element's ID.

---

**Example 2: Methods with Callback Arguments**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Method Invocation — Callbacks</title>
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
      // Step 1: .each() — iterate with a callback
      $("li").each(function (index, element) {
        // 'this' refers to the current DOM element
        console.log("Index " + index + ": " + $(this).text());
      });

      // Step 2: .css() with a callback (computed value per element)
      $("li").css("font-weight", function (index, currentValue) {
        return index === 1 ? "bold" : "normal";
      });

      // Step 3: Verify the second item is bold
      console.log("Second item weight:", $("li").eq(1).css("font-weight")); // "700"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Index 0: Item 1
Index 1: Item 2
Index 2: Item 3
Second item weight: 700
```

**Why This Output Occurs**
`.each()` iterates over all `<li>` elements, passing the index and native DOM element to the callback. Inside the callback, `$(this)` wraps the current element in a jQuery object. `.css()` with a callback computes the value per element; returning `"bold"` for index 1 and `"normal"` for others. The browser reports `"700"` as the computed `font-weight` for bold text.

### Real-World Cases

- **Form Submission**: `$("form").on("submit", handler)` attaches a submit handler.
- **Styling Lists**: `$("ul li:even").css("background", "#f0f0f0")` applies zebra striping.
- **AJAX Requests**: `$.ajax({ url: "...", success: callback })` invokes an AJAX method with options.
- **Animations**: `$(".box").fadeOut(400, function() { $(this).remove(); })` fades out and removes elements.

### References

- jQuery API — Category: Methods – https://api.jquery.com/category/methods/
- jQuery API — .each() – https://api.jquery.com/each/
- jQuery API — .css() – https://api.jquery.com/css/

---

## Core Concept 4: Arguments

### Definitions

**Core Definition**
Arguments are the values passed to jQuery methods inside parentheses, configuring how the method behaves.

**Technical Definition**
In JavaScript, arguments are the actual values supplied when a function is called, as opposed to parameters, which are the named variables in the function definition. jQuery methods accept varying numbers and types of arguments depending on their purpose. Many methods are overloaded: the same method can act as a getter (no arguments), a setter (one or more arguments), or a callback-based setter (a function argument).

**Beginner-Friendly Explanation**
Arguments are like the settings you choose when using an app. When you call `.css()`, you can pass two arguments — the property name and the value — to specify exactly what style to change. When you call `.hide()`, you pass no arguments because the method already knows what to do.

### Purposes

- To configure the behaviour of a method (e.g., which CSS property to set, which event to listen for).
- To provide data to the method (e.g., the value to assign).
- To supply callback functions for asynchronous or per-element execution.
- To enable method overloading, where the same method performs different actions based on the arguments provided.
- To pass options objects for complex configurations (e.g., `$.ajax({ ... })`).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).method();                      // no arguments (getter or default action)
$(selector).method(value);                 // one argument (setter)
$(selector).method(property, value);       // two arguments (setter with property)
$(selector).method(function(index, val) { ... }); // callback argument
$(selector).method({ option1: val1, option2: val2 }); // options object
```

**Component Breakdown**

- `()` : Empty parentheses indicate no arguments.
- `(value)` : A single argument, typically the value to set.
- `(property, value)` : Two arguments: the property to change and the new value.
- `(function)` : A callback function that receives the index and current value.
- `({ options })` : An object literal containing multiple named options.

**Syntax Rules**

1. Arguments are separated by commas: `.css("color", "red")`.
2. String arguments must be quoted: `.addClass("active")`.
3. Callback arguments receive parameters defined by the method (commonly `index` and `value`).
4. When a function is passed as an argument, it should **not** be invoked (no `()` after the function name).
5. jQuery methods often inspect the number and type of arguments to determine the operation mode.

**Constraints and Limitations**

- Passing the wrong type of argument (e.g., a number where a string is expected) may cause unexpected behaviour or silent failure.
- Some methods require specific argument combinations; for example, `.css()` requires either one argument (getter) or two arguments (setter).
- The `arguments` object inside functions is array-like but not a true array; it lacks array methods like `.slice()` without conversion.

### Multiple Annotated Complete Code Examples

**Example 1: Argument Variations for a Single Method**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Arguments — Method Overloading</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" style="width:100px;height:100px;background:gray;"></div>

  <script>
    $(function () {
      // --- .css() with one argument (getter) ---
      var bg = $("#box").css("background-color");
      console.log("Getter result:", bg); // "rgb(128, 128, 128)"

      // --- .css() with two arguments (setter) ---
      $("#box").css("background-color", "orange");
      console.log("After set:", $("#box").css("background-color")); // "rgb(255, 165, 0)"

      // --- .css() with an object argument (multiple properties) ---
      $("#box").css({
        "border": "2px solid black",
        "border-radius": "8px"
      });

      // --- .css() with a callback (per-element value) ---
      $("div").css("opacity", function (index, current) {
        return 0.5 + index * 0.1; // 0.5 for first, 0.6 for second, etc.
      });

      console.log("Opacity:", $("#box").css("opacity")); // "0.5"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Getter result: rgb(128, 128, 128)
After set: rgb(255, 165, 0)
Opacity: 0.5
```

**Why This Output Occurs**
`.css("background-color")` with one argument is a getter that returns the computed colour. `.css("background-color", "orange")` with two arguments is a setter. `.css({ ... })` with an object sets multiple properties. `.css("opacity", function)` uses a callback to compute the value per element.

---

**Example 2: Callback Arguments in `.each()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Arguments — Callback Parameters</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <ul>
    <li data-id="a">Alpha</li>
    <li data-id="b">Beta</li>
    <li data-id="c">Gamma</li>
  </ul>

  <script>
    $(function () {
      // Step 1: Use .each() with a callback that receives index and element
      $("li").each(function (index, element) {
        // 'index' is the zero-based position
        // 'element' is the native DOM element
        // 'this' is also the native DOM element
        console.log("Index: " + index);
        console.log("Text: " + element.textContent);
        console.log("Data ID: " + $(element).data("id"));
        console.log("---");
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Index: 0
Text: Alpha
Data ID: a
---
Index: 1
Text: Beta
Data ID: b
---
Index: 2
Text: Gamma
Data ID: c
---
```

**Why This Output Occurs**
`.each()` passes two arguments to the callback: `index` (the position in the collection) and `element` (the native DOM element). Inside the callback, `element.textContent` accesses the native property, while `$(element).data("id")` wraps the element in a jQuery object to use the `.data()` method.

### Real-World Cases

- **AJAX Options**: `$.ajax({ url: "/api/data", method: "GET", success: function(data) { ... } })` passes an options object.
- **Animation Configuration**: `$(".box").fadeOut(400, "swing", function() { ... })` passes duration, easing, and callback arguments.
- **Event Data**: `$("button").on("click", { key: "value" }, handler)` passes data to the event handler via `event.data`.
- **Attribute Setting**: `$("img").attr({ src: "image.jpg", alt: "Photo" })` sets multiple attributes at once.

### References

- MDN Web Docs — Functions: Arguments – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#function_parameters
- jQuery API — .css() – https://api.jquery.com/css/
- jQuery API — .each() – https://api.jquery.com/each/
- jQuery API — .on() – https://api.jquery.com/on/

---

## Core Concept 5: Return Values

### Definitions

**Core Definition**
Return values are the values that jQuery methods produce after execution. Most setter methods return the jQuery object itself (enabling chaining), while getter methods return the underlying data (strings, numbers, DOM elements, etc.).

**Technical Definition**
jQuery methods are designed around two primary return behaviours. Setter methods — those that modify elements or state — return the jQuery object they were called on, or a new jQuery object. This is the foundation of method chaining. Getter methods — those called without arguments to retrieve information — return the raw value, such as a string from `.text()`, a number from `.width()`, or a native DOM element from `.get()`. Traversal methods return a **new** jQuery object containing a different set of elements, and `.end()` can be used to revert to the previous selection.

**Beginner-Friendly Explanation**
A return value is what a method “gives back” after it runs. Most jQuery methods give back the same jQuery object you started with, so you can immediately call another method on it. But some methods give back a simple value, like the text inside an element or the width of a box. Those methods stop the chain because you can't call jQuery methods on a plain string or number.

### Purposes

- To enable method chaining by returning the jQuery object from setter methods.
- To provide access to element data through getter methods.
- To support traversal and filtering by returning new jQuery objects.
- To allow `.end()` to recover earlier selections in a chain.
- To integrate with native JavaScript by returning native values where appropriate.

### Syntax Rules and Structure

**Return Type Categories**

| Method Type | Return Value | Chainable? | Example |
|---|---|---|---|
| Setter | jQuery object | Yes | `.css("color", "red")` |
| Getter | String, Number, etc. | No | `.text()`, `.width()` |
| Traversal | New jQuery object | Yes | `.find()`, `.filter()` |
| Conversion | Native DOM element/Array | No | `.get()`, `[0]` |
| Utility | Varies | Depends | `$.each()` returns original collection |

**Syntax Rules**

1. Check the API documentation for each method's return type before chaining.
2. Setter methods return the jQuery object, allowing further method calls.
3. Getter methods return raw values and break the chain.
4. Traversal methods return new jQuery objects, not the original; use `.end()` to revert.
5. `.get()` returns native DOM elements; `[index]` is a shorthand for the same.

**Constraints and Limitations**

- Chaining after a getter throws a `TypeError` because the returned value is not a jQuery object.
- Traversal methods do not modify the original jQuery object; they push a new selection onto an internal stack.
- `.end()` only works if there is a previous selection on the stack.

### Multiple Annotated Complete Code Examples

**Example 1: Setter vs Getter Return Values**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Return Values — Setter vs Getter</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p id="demo">Hello</p>

  <script>
    $(function () {
      // --- Setter returns jQuery object (chainable) ---
      var $result = $("#demo").text("World");
      console.log("Setter returns:", $result.constructor.name); // "jQuery"
      console.log("Can chain?", typeof $result.addClass);       // "function"

      // --- Getter returns a string (not chainable) ---
      var text = $("#demo").text();
      console.log("Getter returns:", typeof text);              // "string"
      console.log("Can chain?", typeof text.addClass);          // "undefined"

      // --- Traversal returns a new jQuery object ---
      var $children = $("#demo").find("span");
      console.log("Traversal returns:", $children.constructor.name); // "jQuery"
      console.log("Length:", $children.length);                       // 0
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Setter returns: jQuery
Can chain? function
Getter returns: string
Can chain? undefined
Traversal returns: jQuery
Length: 0
```

**Why This Output Occurs**
`.text("World")` is a setter that returns the jQuery object, so `.addClass` is available. `.text()` is a getter that returns a string; strings have no `.addClass` method. `.find("span")` returns a new jQuery object (empty, since no `<span>` exists inside the paragraph).

---

**Example 2: Using `.end()` to Recover a Previous Selection**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Return Values — .end()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="container">
    <p>First</p>
    <p>Second</p>
    <p>Third</p>
  </div>

  <script>
    $(function () {
      // Step 1: Start with #container, find all p, select the second one
      var $second = $("#container")
        .find("p")          // new selection: all three paragraphs
        .eq(1);             // new selection: the second paragraph

      console.log("Second text:", $second.text()); // "Second"

      // Step 2: Use .end() to go back to all paragraphs
      var $all = $second.end();
      console.log("All paragraphs:", $all.length); // 3

      // Step 3: Chain further after .end()
      $second
        .text("Modified")
        .end()              // back to all paragraphs
        .css("color", "blue");
    });
  </script>
</body>
</html>
```

**Expected Output**
- The second paragraph text becomes “Modified.”
- All three paragraphs turn blue.
- Console output:
```
Second text: Second
All paragraphs: 3
```

**Why This Output Occurs**
`.find("p")` pushes a new selection (all paragraphs) onto the stack. `.eq(1)` pushes another selection (the second paragraph). `.end()` pops the stack back to all paragraphs. The final `.css("color", "blue")` applies to all three paragraphs because `.end()` reverted the selection.

### Real-World Cases

- **Conditional Logic**: `if ($("#element").length > 0) { ... }` uses the return value of `.length`.
- **Data Retrieval**: `var value = $("input").val()` retrieves a form value for validation.
- **Chaining Setup**: `$("div").addClass("active").css("color", "red")` relies on setter return values.
- **Selection Reversion**: `$("ul").find("li").eq(0).addClass("first").end().addClass("has-first")` uses `.end()` to apply classes at different levels.

### References

- jQuery API — .end() – https://api.jquery.com/end/
- jQuery API — .get() – https://api.jquery.com/get/
- Learn jQuery — Working with Selections – https://learn.jquery.com/using-jquery-core/working-with-selections/
- jQuery API — Types – https://api.jquery.com/Types/

---

## Core Concept 6: Chaining

### Definitions

**Core Definition**
Chaining is the technique of calling multiple jQuery methods in a single statement, one after another, because each method returns a jQuery object that the next method can operate on.

**Technical Definition**
Method chaining in jQuery is enabled by the convention that setter methods and traversal methods return a jQuery object. Each method in the chain is invoked on the return value of the previous method, creating a fluent interface. Traversal methods push their results onto an internal stack, and `.end()` pops the stack to return to a previous selection. The chain executes from left to right, with each method receiving the jQuery object produced by the method before it.

**Beginner-Friendly Explanation**
Chaining is like giving a robot a list of instructions in one breath: “Find the heading, change its text, add a class, then fade it in.” Instead of writing four separate lines, you write one line that flows from one action to the next. Each action hands the result to the next action.

### Purposes

- To write concise, readable code by eliminating intermediate variables.
- To express a sequence of related DOM operations as a single fluent statement.
- To reduce the risk of stale variable references.
- To allow `.end()` to recover earlier selections within a chain.
- To improve performance by reducing repeated DOM queries.

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
- `.method1()` : A method that returns a jQuery object.
- `.method2(argument)` : The next method, called on the result of `method1`.
- `.method3()` : The final method; if it returns a jQuery object, the chain can continue.

**Syntax Rules**

1. Only methods that return a jQuery object can be chained.
2. Getter methods (e.g., `.text()` without arguments, `.val()`) break the chain.
3. `.end()` returns the previous jQuery object in the chain, allowing recovery of an earlier selection.
4. Chains can span multiple lines for readability; each `.method()` on a new line is valid.
5. The chain executes from left to right.

**Constraints and Limitations**

- Excessive chaining reduces readability and makes debugging harder.
- Chaining after a getter throws a `TypeError`.
- `.end()` only works if there is a previous selection on the internal stack.
- Some methods (e.g., `.animate()` without a callback) return a jQuery object; others (e.g., `.get()`) do not.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Chaining**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Chaining — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background: yellow; }
    .bold { font-weight: bold; }
  </style>
</head>
<body>
  <h1>Original Heading</h1>

  <script>
    $(function () {
      // Step 1: Chain multiple methods in one statement
      $("h1")
        .text("Chained!")           // setter: returns jQuery object
        .addClass("highlight")      // setter: returns jQuery object
        .addClass("bold")           // setter: returns jQuery object
        .css("color", "darkblue");  // setter: returns jQuery object

      // Step 2: Verify the result
      console.log("Text:", $("h1").text());            // "Chained!"
      console.log("Classes:", $("h1").attr("class"));  // "highlight bold"
      console.log("Color:", $("h1").css("color"));     // "rgb(0, 0, 139)"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The heading text becomes “Chained!” with a yellow background, bold font, and dark blue colour.
- Console output:
```
Text: Chained!
Classes: highlight bold
Color: rgb(0, 0, 139)
```

**Why This Output Occurs**
Each method returns the jQuery object, allowing the next method to be called on it. `.text("Chained!")` sets the text. `.addClass("highlight")` adds the class and returns the object. `.addClass("bold")` adds the second class. `.css("color", "darkblue")` sets the colour. All operations occur on the same `<h1>` element.

---

**Example 2: Chaining with Traversal and `.end()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Chaining — Traversal and .end()</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="list">
    <li>Item A</li>
    <li>Item B</li>
    <li>Item C</li>
  </div>

  <script>
    $(function () {
      // Step 1: Chain traversal, filtering, and manipulation
      $("#list")
        .find("li")              // new selection: all <li>
        .eq(1)                   // new selection: the second <li>
        .text("Modified B")      // set text on second <li>
        .end()                   // revert to all <li>
        .css("color", "purple")  // apply colour to all <li>
        .end()                   // revert to #list
        .css("border", "1px solid gray"); // apply border to #list

      // Step 2: Verify
      console.log("Items:", $("#list li").length); // 3
      console.log("Second item:", $("#list li").eq(1).text()); // "Modified B"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The second list item's text becomes “Modified B.”
- All list items turn purple.
- The `#list` div gains a gray border.
- Console output:
```
Items: 3
Second item: Modified B
```

**Why This Output Occurs**
`.find("li")` pushes all list items onto the stack. `.eq(1)` narrows the selection to the second item. `.text("Modified B")` modifies it. `.end()` pops back to all `<li>` elements, and `.css("color", "purple")` applies to all of them. A second `.end()` pops back to `#list`, and the border is applied to the container.

### Real-World Cases

- **Form Initialisation**: `$("input").val("").addClass("cleared").prop("disabled", false)` resets multiple form fields.
- **Animation Sequences**: `$(".panel").slideUp().delay(200).fadeIn()` chains animations.
- **DOM Construction**: `$("<div>").addClass("card").append($("<p>").text("Hello")).appendTo("body")` builds and inserts an element.
- **Event Setup**: `$("button").addClass("btn").on("click", handler)` styles and binds in one chain.

### References

- Learn jQuery — Working with Selections – https://learn.jquery.com/using-jquery-core/working-with-selections/
- jQuery API — .end() – https://api.jquery.com/end/
- jQuery API — Traversing – https://api.jquery.com/category/traversing/

---

## Core Concept 7: Callback Functions

### Definitions

**Core Definition**
A callback function is a function passed as an argument to another jQuery method, to be executed at a later time — typically after an event occurs, an animation completes, or an AJAX request finishes.

**Technical Definition**
In jQuery, callback functions are used extensively for asynchronous operations. When a method accepts a function as an argument, it stores that function and invokes it when the appropriate condition is met. Inside the callback, the keyword `this` typically refers to the current DOM element, and the function may receive arguments such as the event object, the index, or the response data, depending on the method. jQuery uses an internal `$.Callbacks()` utility to manage lists of callback functions.

**Beginner-Friendly Explanation**
A callback function is like leaving a note for a friend: “When you finish this task, do this next.” In jQuery, you often pass a callback to say, “When this button is clicked, run this code” or “When this animation finishes, run this function.” The callback ensures your code runs at exactly the right moment, not before.

### Purposes

- To execute code in response to user events (clicks, keypresses, etc.).
- To run code after an animation or AJAX request completes.
- To process each element in a collection with `.each()`.
- To compute values per element in setters like `.css()` and `.attr()`.
- To manage asynchronous flow in AJAX and Deferred/Promise chains.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).method(function(argument1, argument2) {
  // code to execute
});
```

**Component Breakdown**

- `$(selector).method` : A jQuery method that accepts a callback.
- `function(argument1, argument2) { ... }` : An anonymous (or named) function passed as the argument.
- The parameters the callback receives depend on the method:
  - `.each(function(index, element))` : `index` (zero-based position), `element` (native DOM element).
  - `.on("click", function(event))` : `event` (jQuery event object).
  - `.animate(..., function())` : No standard parameters; `this` is the animated element.
  - `$.ajax({ success: function(data, textStatus, jqXHR))` : Response data and status.

**Syntax Rules**

1. The callback function is passed **without** parentheses: `.on("click", handler)`, not `.on("click", handler())`.
2. Inside the callback, `this` refers to the DOM element that triggered the event or the current element in an iteration.
3. Use `$(this)` to wrap the current DOM element in a jQuery object.
4. Callback parameters are optional; you only declare the ones you need.
5. Callbacks can be named functions or anonymous function expressions.

**Constraints and Limitations**

- If a jQuery method does not provide a callback option, the operation is synchronous, and you can proceed on the next line.
- Callbacks in animations do not receive standard arguments; use `this` to access the element.
- Error handling in AJAX callbacks requires the `error` or `fail` callback (or `.catch()` on the jqXHR object).
- Callback functions execute in the order they are registered for a given event, but asynchronous operations may complete in unpredictable order.

### Multiple Annotated Complete Code Examples

**Example 1: Event Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Callbacks — Event Handler</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="myButton">Click me</button>
  <p id="output"></p>

  <script>
    $(function () {
      // Step 1: Attach a click event with a callback
      $("#myButton").on("click", function (event) {
        // 'event' is the jQuery event object
        // 'this' is the native DOM element that was clicked
        console.log("Button clicked!");
        console.log("Event type:", event.type);       // "click"
        console.log("Element ID:", this.id);          // "myButton"

        // Step 2: Update the output paragraph
        $("#output").text("You clicked the button!");

        // Step 3: Prevent default behaviour (if needed)
        event.preventDefault();
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking the button updates the paragraph text to “You clicked the button!”.
- Console output:
```
Button clicked!
Event type: click
Element ID: myButton
```

**Why This Output Occurs**
`.on("click", callback)` attaches the callback to the button's click event. When clicked, the callback receives the event object. `this` refers to the button element. The callback updates the `#output` paragraph and prevents the default action (though buttons have no default action in this context).

---

**Example 2: AJAX Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Callbacks — AJAX</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="result">Loading...</div>

  <script>
    $(function () {
      // Step 1: Make an AJAX request with success and error callbacks
      $.ajax({
        url: "https://jsonplaceholder.typicode.com/posts/1",
        method: "GET",
        dataType: "json",
        success: function (data, textStatus, jqXHR) {
          // Step 2: Success callback — runs when request succeeds
          console.log("Status:", textStatus); // "success"
          $("#result").text("Title: " + data.title);
        },
        error: function (jqXHR, textStatus, errorThrown) {
          // Step 3: Error callback — runs when request fails
          console.log("Error:", errorThrown);
          $("#result").text("Failed to load data.");
        },
        complete: function () {
          // Step 4: Complete callback — always runs
          console.log("Request completed.");
        }
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The `#result` div displays the fetched post title.
- Console output:
```
Status: success
Request completed.
```

**Why This Output Occurs**
`$.ajax()` sends an HTTP GET request. When the server responds successfully, the `success` callback receives the parsed JSON data. The `complete` callback runs regardless of success or failure. If the request fails, the `error` callback would run instead.

### Real-World Cases

- **Form Validation on Submit**: `$("form").on("submit", function(event) { event.preventDefault(); validate(); })`.
- **Infinite Scroll**: `$(window).on("scroll", function() { if (nearBottom()) { loadMore(); } })`.
- **Animation Sequences**: `$(".box").fadeOut(400, function() { $(this).slideUp(); })`.
- **Dynamic Content Loading**: `$.get("/api/data", function(data) { $("#container").html(data); })`.
- **Deferred Chains**: `$.when(promise1, promise2).done(function() { ... })`.

### References

- jQuery API — .on() – https://api.jquery.com/on/
- jQuery API — jQuery.Callbacks() – https://api.jquery.com/jQuery.Callbacks/
- jQuery API — jQuery.ajax() – https://api.jquery.com/jQuery.ajax/
- MDN Web Docs — Callback function – https://developer.mozilla.org/en-US/docs/Glossary/Callback_function

---

## Summary Table: Basic jQuery Syntax at a Glance

| Concept | Key Idea | Example | Chainable? |
|---|---|---|---|
| General Pattern | `$(selector).method()` | `$("p").hide()` | Yes (setters) |
| Selector Expression | CSS-based element description | `$(".intro")`, `$("ul > li")` | N/A |
| Method Invocation | Calling a jQuery method with dot notation | `$("#box").css("color", "red")` | Yes |
| Arguments | Values configuring method behaviour | `.css("color", "red")`, `.on("click", fn)` | N/A |
| Return Values | jQuery object (setters) or raw value (getters) | `.text()` → string; `.addClass()` → jQuery | Depends |
| Chaining | Multiple methods in one statement | `$("div").addClass("a").css("color", "red")` | Yes |
| Callback Functions | Deferred execution functions | `.on("click", function() { ... })` | N/A |

---

## General References

- jQuery Official Website – https://jquery.com/
- jQuery API Documentation – https://api.jquery.com/
- jQuery Learning Center – https://learn.jquery.com/
- Learn jQuery — How jQuery Works – https://learn.jquery.com/about-jquery/how-jquery-works/
- W3Schools — jQuery Syntax – https://www.w3schools.com/jquery/jquery_syntax.asp
- MDN Web Docs — JavaScript Functions – https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions
- MDN Web Docs — CSS Selectors – https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors
- jQuery API — Category: Selectors – https://api.jquery.com/category/selectors/
- jQuery API — Category: Methods – https://api.jquery.com/category/methods/
- jQuery API — .end() – https://api.jquery.com/end/
- jQuery API — jQuery.Callbacks() – https://api.jquery.com/jQuery.Callbacks/