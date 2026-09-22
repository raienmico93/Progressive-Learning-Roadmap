# jQuery CSS Class and Style Manipulation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery's CSS class and style manipulation methods are a set of functions that read and modify the visual presentation of DOM elements, either by directly reading and writing individual CSS properties (`.css()`) or by adding, removing, and toggling CSS class names (`.addClass()`, `.removeClass()`, `.toggleClass()`, `.hasClass()`).

**Technical Definition**
These methods operate on the `style` property and `class` attribute of DOM elements. The `.css()` method serves a dual role: as a **getter**, it retrieves the **computed style** of the first matched element (via `window.getComputedStyle()` or its legacy equivalents), returning a normalised string value; as a **setter**, it applies **inline styles** to every matched element. The class methods operate on the element's `class` attribute, modifying it through jQuery's internal `class` manipulation routines. Since jQuery 1.12/2.2, the class methods interact with the `class` attribute directly, improving support for XML and SVG documents.

**Beginner-Friendly Explanation**
Every element on a web page can be styled in two ways: by giving it a label (a **class**) that a stylesheet knows how to style, or by writing style rules directly on the element (an **inline style**). jQuery gives you tools for both. The class methods — `.addClass()`, `.removeClass()`, `.toggleClass()`, and `.hasClass()` — let you add or remove labels. The `.css()` method lets you read what an element actually looks like or write a style rule directly onto it. The general rule is: use classes for styling, and use `.css()` when you need to read a value or make a one-off, dynamic change.

### Key Characteristics

- **Computed vs. Inline**: `.css()` as a getter returns the **computed** style (the final, rendered value after all stylesheets are applied). As a setter, it writes **inline** styles directly to the element.
- **Class Methods are Attribute-Based**: `.addClass()`, `.removeClass()`, `.toggleClass()`, and `.hasClass()` operate on the `class` attribute and do not affect inline styles.
- **Automatic Unit Handling**: jQuery appends `px` to numeric values for CSS properties that require a unit (unless the property is in the `jQuery.cssNumber` list).
- **Cross-Browser Normalisation**: `.css()` normalises vendor-prefixed properties (e.g., `float` → `cssFloat`) and camelCase/hyphenated property names.
- **Chainable Setters**: All setter methods return the jQuery object, preserving method chaining.
- **CSS Custom Properties Supported**: As of jQuery 3.2, `.css()` supports CSS custom properties (variables) when the property name is passed as-is (not camelCased).

### Prerequisites

- Basic understanding of CSS selectors, properties, and the cascade.
- Familiarity with HTML `class` attributes and inline `style` attributes.
- jQuery library included in the page via a `<script>` tag or CDN.
- Awareness of the difference between computed styles and inline styles.

### Related Programming Areas

- **CSS Styling**: These methods bridge JavaScript and CSS, enabling dynamic visual changes.
- **Animation and Effects**: jQuery's animation methods (`.fadeIn()`, `.slideUp()`, etc.) internally manipulate CSS properties.
- **DOM Manipulation**: Class manipulation is a core part of dynamic UI updates.
- **Theme Systems**: Class toggling is the foundation of theme switching and state-based styling.

### Core Concepts / Features

1. `.css()` — Get or Set CSS Properties
2. `.addClass()` — Add Class Names
3. `.removeClass()` — Remove Class Names
4. `.toggleClass()` — Toggle Class Names
5. `.hasClass()` — Check for a Class Name
6. Reading Computed Styles vs. Setting Inline Styles
7. Performance Implications of Inline Style Modifications vs. Class Toggling

---

## Core Concept 1: `.css()` — Get or Set CSS Properties

### Definitions

**Core Definition**
`.css()` gets the value of a computed style property for the first element in the set of matched elements, or sets one or more CSS properties for every matched element.

**Technical Definition**
The `.css()` method is a convenient way to get a computed style property from the first matched element, especially in light of the different ways browsers access most of those properties (the `getComputedStyle()` method in standards-based browsers versus the `currentStyle` and `runtimeStyle` properties in Internet Explorer prior to version 9). As a setter, it applies inline styles to all matched elements, appending `px` to numeric values where appropriate. It accepts a single property name, a property-value pair, an object of multiple properties, or a callback function. As of jQuery 1.9, passing an array of style properties returns an object of property-value pairs. As of jQuery 3.2, CSS custom properties are supported.

**Beginner-Friendly Explanation**
`.css()` is how you ask “what does this element look like?” or say “make this element look like this.” Reading is like measuring the final rendered result — you get the actual font size, colour, or width after all stylesheets have been applied. Writing is like writing a style rule directly on the element, which overrides stylesheet rules.

### Purposes

- To retrieve the **computed** value of a CSS property from the first matched element.
- To set one or more **inline** CSS properties on every matched element.
- To read multiple properties at once and receive an object of property-value pairs.
- To compute style values dynamically using a callback function.
- To read or write CSS custom properties (variables).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Getter — single property
$(selector).css(propertyName);

// Getter — multiple properties (returns object, jQuery 1.9+)
$(selector).css([propertyName1, propertyName2]);

// Setter — single property
$(selector).css(propertyName, value);
$(selector).css(propertyName, function(index, oldValue) { ... });

// Setter — multiple properties
$(selector).css({ property1: value1, property2: value2 });
```

**Component Breakdown**

- `propertyName` (String): A CSS property name in camelCase (e.g., `fontSize`) or hyphenated (e.g., `font-size`).
- `value` (String | Number): The value to set. Numbers are converted to `px` for properties that require units.
- `function(index, oldValue)`: A callback returning the value to set.
- `{ property1: value1, ... }` (Object): An object of property-value pairs.
- `[property1, property2]` (Array): An array of property names to retrieve (jQuery 1.9+).
- Returns: A string or object (getter) or a jQuery object (setter).

**Syntax Rules**

1. jQuery understands both CSS (`"background-color"`) and DOM (`"backgroundColor"`) formatting of multi-word properties.
2. The property name is case-sensitive: `.css("WiDtH")` does not work the same as `.css("width")`.
3. Retrieval of shorthand properties (e.g., `margin`, `background`, `border`) is not guaranteed; use longhand properties instead.
4. An element should be connected to the DOM when calling `.css()` on it; otherwise jQuery may throw an error.
5. Numeric values are automatically converted to strings with `px` appended for most properties.

**Constraints and Limitations**

- The getter returns the value for the **first** element only.
- Computed styles may not match the value specified in a stylesheet (e.g., dimensions are almost always in pixels).
- Different browsers may return logically equivalent but textually different colour values (e.g., `#FFF`, `#ffffff`, `rgb(255,255,255)`).
- Shorthand properties are not reliably retrievable.

### Multiple Annotated Complete Code Examples

**Example 1: Reading and Setting CSS Properties**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.css() — Read and Set</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #box { width: 100px; height: 100px; background-color: lightblue; font-size: 16px; }
  </style>
</head>
<body>
  <div id="box">Styled Box</div>

  <script>
    $(function () {
      // Step 1: GET — read a single computed property
      var bgColor = $("#box").css("background-color");
      console.log("Background:", bgColor); // "rgb(173, 216, 230)"

      var fontSize = $("#box").css("font-size");
      console.log("Font size:", fontSize); // "16px"

      // Step 2: GET multiple properties as an object (jQuery 1.9+)
      var styles = $("#box").css(["width", "height", "font-size"]);
      console.log("Styles object:", styles);
      // { width: "100px", height: "100px", fontSize: "16px" }

      // Step 3: SET a single property inline
      $("#box").css("border", "3px solid red");
      console.log("Border:", $("#box").css("border-top-width")); // "3px"

      // Step 4: SET multiple properties with an object
      $("#box").css({
        "background-color": "lightgreen",
        "font-weight": "bold",
        "padding": "10px"
      });

      // Step 5: SET with a callback
      $("#box").css("width", function (index, oldValue) {
        return parseInt(oldValue) + 50 + "px"; // 150px
      });
      console.log("New width:", $("#box").css("width")); // "150px"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Background: rgb(173, 216, 230)
Font size: 16px
Styles object: { width: "100px", height: "100px", fontSize: "16px" }
Border: 3px
New width: 150px
```

**Why This Output Occurs**
`.css("background-color")` returns the computed colour in `rgb()` format. `.css("font-size")` returns the value with units. The array getter returns an object with camelCased keys. Setting `border` inline adds an inline style that overrides the stylesheet. The callback receives the current width and returns a new value, which jQuery appends with `px`.

---

**Example 2: Reading Computed vs. Inline Styles**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.css() — Computed vs Inline</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #heading { font-size: 24px; color: #333333; }
  </style>
</head>
<body>
  <h1 id="heading">My Site</h1>
  <h1 id="heading-inline" style="font-size: 24px; color: #333333;">My Site</h1>

  <script>
    $(function () {
      // Step 1: .css() returns the COMPUTED style (from stylesheet)
      console.log("Computed fontSize:", $("#heading").css("font-size")); // "24px"
      console.log("Computed color:", $("#heading").css("color")); // "rgb(51, 51, 51)"

      // Step 2: .attr("style") returns the INLINE style attribute only
      console.log("Inline style attr:", $("#heading").attr("style")); // undefined (no inline style)

      // Step 3: For the inline-styled element, both agree
      console.log("Inline computed:", $("#heading-inline").css("font-size")); // "24px"
      console.log("Inline style attr:", $("#heading-inline").attr("style")); // "font-size: 24px; color: #333333;"

      // Step 4: The native DOM .style property also returns inline only
      console.log("Native style:", document.getElementById("heading").style.fontSize); // ""
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Computed fontSize: 24px
Computed color: rgb(51, 51, 51)
Inline style attr: undefined
Inline computed: 24px
Inline style attr: font-size: 24px; color: #333333;
Native style: 
```

**Why This Output Occurs**
`.css()` returns the **computed** style, which is the final rendered value after all CSS is applied. `.attr("style")` returns the **inline** style attribute only, which is empty for the first heading. The native DOM `.style` property also returns inline styles only.

### Real-World Cases

- **Reading Dimensions**: `$("#element").css("width")` to get the computed width for layout calculations.
- **Dynamic Theming**: `$("body").css("background-color", themeColor)` for a one-off theme change.
- **Animation Fallbacks**: Reading computed values before animating with `.animate()`.
- **CSS Custom Properties**: `$(":root").css("--primary-color", "#ff0000")` to update a CSS variable.

### References

- jQuery API — .css() – https://api.jquery.com/css/
- jQuery API — Manipulation Category – https://api.jquery.com/category/manipulation/
- Stack Overflow — jQuery .css() vs .style – https://stackoverflow.com/questions/37890100/

---

## Core Concept 2: `.addClass()` — Add Class Names

### Definitions

**Core Definition**
`.addClass()` adds the specified class(es) to each element in the set of matched elements.

**Technical Definition**
The `.addClass()` method appends one or more space-separated class names to the `class` attribute of each matched element. It does not replace existing classes; it simply adds to them. As of jQuery 1.12/2.2, the method operates on the `class` attribute directly (rather than the `className` property), improving support for XML and SVG documents. As of jQuery 1.4, the argument can be a function that returns the class name(s) to add. As of jQuery 3.3, it accepts an array of class names.

**Beginner-Friendly Explanation**
`.addClass()` is how you give an element a new label. If the element already has labels, they stay — the new one is just added to the list. You can add one class, several at once, or use a function to compute the class name dynamically.

### Purposes

- To add one or more CSS class names to matched elements.
- To apply a new visual state (e.g., `active`, `highlight`, `error`) to elements.
- To add classes computed dynamically using a callback function.
- To append classes without removing existing ones.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).addClass(className);
$(selector).addClass("className1 className2");
$(selector).addClass([className1, className2]);          // jQuery 3.3+
$(selector).addClass(function(index, currentClasses) { ... });
```

**Component Breakdown**

- `className` (String): One or more space-separated class names to add.
- `[className1, className2]` (Array): An array of class names to add (jQuery 3.3+).
- `function(index, currentClasses)`: A callback returning the class name(s) to add.
- Returns: A jQuery object (for chaining).

**Syntax Rules**

1. Multiple classes are separated by spaces: `.addClass("myClass yourClass")`.
2. The method does **not** replace existing classes; it appends.
3. As of jQuery 1.12/2.2, it operates on the `class` attribute and works with XML/SVG documents.
4. The callback receives the element's index and the current class string.
5. The setter is chainable and returns the jQuery object.

**Constraints and Limitations**

- Does not check for duplicates; adding a class that already exists is a no-op.
- Very long class attribute values may affect performance in extreme cases.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Class Addition**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.addClass() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background-color: yellow; }
    .bold { font-weight: bold; }
  </style>
</head>
<body>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
  <p>Paragraph 3</p>

  <script>
    $(function () {
      // Step 1: Add a single class to all paragraphs
      $("p").addClass("highlight");
      console.log("After single:", $("p").first().attr("class")); // "highlight"

      // Step 2: Add multiple classes at once
      $("p").addClass("bold important");
      console.log("After multiple:", $("p").first().attr("class")); // "highlight bold important"

      // Step 3: Add an array of classes (jQuery 3.3+)
      $("p").addClass(["rounded", "shadow"]);
      console.log("After array:", $("p").first().attr("class"));

      // Step 4: Use a callback to add computed classes
      $("p").addClass(function (index) {
        return "item-" + index;
      });
      console.log("After callback:", $("p").last().attr("class"));
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
After single: highlight
After multiple: highlight bold important
After array: highlight bold important rounded shadow
After callback: highlight bold important rounded shadow item-2
```

**Why This Output Occurs**
Each call appends classes to the existing `class` attribute without removing anything. The callback receives the element's index and returns a computed class name.

### Real-World Cases

- **State Management**: `$("#tab-1").addClass("active")` marks a tab as active.
- **Form Validation**: `$("#email").addClass("error")` highlights an invalid field.
- **Animation Setup**: `$("#panel").addClass("sliding")` prepares an element for animation.
- **Theme Switching**: `$("body").addClass("dark-theme")` switches to a dark theme.

### References

- jQuery API — .addClass() – https://api.jquery.com/addClass/
- jQuery API — Attributes Category – https://api.jquery.com/category/attributes/
- Version 3.3 | jQuery API Documentation – https://api.jquery.com/category/version/3.3/

---

## Core Concept 3: `.removeClass()` — Remove Class Names

### Definitions

**Core Definition**
`.removeClass()` removes a single class, multiple classes, or all classes from each element in the set of matched elements.

**Technical Definition**
The `.removeClass()` method removes one or more space-separated class names from the `class` attribute of each matched element. As of jQuery 1.12/2.2, it operates on the `class` attribute directly, improving XML/SVG support. When called with **no arguments**, it removes **all** classes from the matched elements. As of jQuery 1.4, it accepts a function that returns the class name(s) to remove.

**Beginner-Friendly Explanation**
`.removeClass()` is how you take a label away from an element. You can remove one class, several at once, or — by calling it with no arguments — remove every class the element has.

### Purposes

- To remove one or more CSS class names from matched elements.
- To clear all classes from an element (called with no arguments).
- To remove classes computed dynamically using a callback function.
- To transition elements from one visual state to another.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).removeClass();                              // remove all classes
$(selector).removeClass(className);
$(selector).removeClass("className1 className2");
$(selector).removeClass([className1, className2]);       // jQuery 3.3+
$(selector).removeClass(function(index, oldClasses) { ... });
```

**Component Breakdown**

- `className` (String): One or more space-separated class names to remove.
- `[className1, className2]` (Array): An array of class names to remove (jQuery 3.3+).
- `function(index, oldClasses)`: A callback returning the class name(s) to remove.
- Returns: A jQuery object (for chaining).

**Syntax Rules**

1. Called with **no arguments**, it removes **all** classes from the matched elements.
2. Multiple classes are separated by spaces: `.removeClass("myClass noClass")`.
3. As of jQuery 1.12/2.2, it operates on the `class` attribute (supports XML/SVG).
4. The callback receives the element's index and the old class value.
5. Removing a non-existent class is a no-op.

**Constraints and Limitations**

- Removing all classes may strip accessibility or framework-related classes unexpectedly.
- Does not remove inline styles; use `.css()` or `.removeAttr("style")` for that.

### Multiple Annotated Complete Code Examples

**Example 1: Removing Classes**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.removeClass() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p class="highlight bold important">Paragraph 1</p>
  <p class="highlight bold important">Paragraph 2</p>

  <script>
    $(function () {
      // Step 1: Remove a single class
      $("p").removeClass("highlight");
      console.log("After single:", $("p").first().attr("class")); // "bold important"

      // Step 2: Remove multiple classes
      $("p").removeClass("bold important");
      console.log("After multiple:", $("p").first().attr("class")); // ""

      // Step 3: Add classes back, then remove all with no arguments
      $("p").addClass("a b c d e");
      console.log("Before all:", $("p").first().attr("class")); // "a b c d e"

      $("p").removeClass();
      console.log("After all:", $("p").first().attr("class")); // "" (all removed)
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
After single: bold important
After multiple: 
Before all: a b c d e
After all: 
```

**Why This Output Occurs**
`.removeClass("highlight")` removes only the named class. `.removeClass("bold important")` removes both. `.removeClass()` with no arguments strips every class from the element.

### Real-World Cases

- **Tab Switching**: `$(".tab.active").removeClass("active")` deactivates the current tab.
- **Form Reset**: `$("input").removeClass("error success")` clears validation states.
- **Theme Reversion**: `$("body").removeClass("dark-theme").addClass("light-theme")`.

### References

- jQuery API — .removeClass() – https://api.jquery.com/removeClass/
- W3Schools — jQuery removeClass() Method – https://www.w3schools.com/jquery/html_removeclass.asp

---

## Core Concept 4: `.toggleClass()` — Toggle Class Names

### Definitions

**Core Definition**
`.toggleClass()` adds or removes one or more classes from each element in the set of matched elements, depending on either the class's presence or the value of the state argument.

**Technical Definition**
The `.toggleClass()` method alternates between adding and removing class names. If an element already has the class, it is removed; if it does not, it is added. The method accepts an optional **state** argument (a boolean) that determines whether to add (`true`) or remove (`false`) the class, making it useful for state-driven styling. As of jQuery 3.3, it accepts arrays of class names.

**Beginner-Friendly Explanation**
`.toggleClass()` is like a light switch — it flips the class on or off. If the class is present, it is removed; if absent, it is added. This is perfect for things like opening and closing menus, switching themes, or toggling any visual state.

### Purposes

- To alternate a class between present and absent states.
- To toggle one or more classes in response to user interaction.
- To apply or remove a class based on a boolean condition using the state argument.
- To compute the class to toggle using a callback function.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).toggleClass(className);
$(selector).toggleClass(className, state);
$(selector).toggleClass([className1, className2]);              // jQuery 3.3+
$(selector).toggleClass(function(index, oldClasses, state) { ... });
```

**Component Breakdown**

- `className` (String): One or more space-separated class names to toggle.
- `state` (Boolean): If `true`, adds the class; if `false`, removes it.
- `[className1, className2]` (Array): An array of class names to toggle (jQuery 3.3+).
- `function(index, oldClasses, state)`: A callback returning the class name(s) to toggle.
- Returns: A jQuery object (for chaining).

**Syntax Rules**

1. Without a state argument, the class is added if absent and removed if present.
2. The state argument must be a **boolean** (not truthy/falsy).
3. As of jQuery 3.3, an array of class names is accepted.
4. The callback receives the index, old class value, and state.

**Constraints and Limitations**

- The state argument must be strictly `true` or `false`; other values are treated as the default toggle behaviour.
- Does not affect inline styles.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Toggling**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.toggleClass() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background-color: yellow; }
    .bounce { animation: bounce 0.5s; }
  </style>
</head>
<body>
  <div class="tumble">Some text.</div>
  <button id="toggle">Toggle Highlight</button>

  <script>
    $(function () {
      // Step 1: Toggle a single class on button click
      $("#toggle").on("click", function () {
        $("div.tumble").toggleClass("bounce");
        console.log("Classes:", $("div.tumble").attr("class"));
      });

      // Step 2: Toggle with a state argument (force add)
      $("div.tumble").toggleClass("highlight", true);
      console.log("After forced add:", $("div.tumble").attr("class"));
      // "tumble highlight"

      // Step 3: Toggle with a state argument (force remove)
      $("div.tumble").toggleClass("highlight", false);
      console.log("After forced remove:", $("div.tumble").attr("class"));
      // "tumble"
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
After forced add: tumble highlight
After forced remove: tumble
Classes: tumble
Classes: tumble bounce
```

**Why This Output Occurs**
`.toggleClass("highlight", true)` forces the class to be added. `.toggleClass("highlight", false)` forces it to be removed. Clicking the button toggles `bounce` on and off with each click.

### Real-World Cases

- **Accordion Menus**: `$(".panel").toggleClass("open")` opens and closes a panel.
- **Theme Toggle**: `$("body").toggleClass("dark-mode")` switches between light and dark themes.
- **Menu Toggle**: `$("#nav").toggleClass("expanded")` expands and collapses a navigation menu.
- **Checkbox Styling**: `$(".checkbox").toggleClass("checked")` reflects a checkbox's visual state.

### References

- jQuery API — .toggleClass() – https://api.jquery.com/toggleClass/
- W3Schools — jQuery toggleClass() Method – https://www.w3schools.com/jquery/html_toggleclass.asp

---

## Core Concept 5: `.hasClass()` — Check for a Class Name

### Definitions

**Core Definition**
`.hasClass()` determines whether any of the matched elements are assigned the given class.

**Technical Definition**
The `.hasClass()` method returns a **boolean** (`true` or `false`) indicating whether at least one element in the set of matched elements has the specified class. It is more convenient than using `.attr("class")` and testing the string directly. The method returns `true` if the class is assigned to an element, even if other classes also are. As of jQuery 1.12/2.2, it supports XML documents, including SVG.

**Beginner-Friendly Explanation**
`.hasClass()` asks: “Does this element have this label?” It gives a simple yes or no. You can use it in an `if` statement to check an element's state before deciding what to do.

### Purposes

- To test whether an element has a specific class.
- To use class presence as a condition in application logic.
- To determine the correct action in a toggle or state management flow.
- To check classes on SVG or XML elements (jQuery 1.12+).

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
$(selector).hasClass(className);
```

**Component Breakdown**

- `className` (String): The class name to check for. Only one class name can be checked per call.
- Returns: **Boolean** (`true` if at least one matched element has the class; otherwise `false`).

**Syntax Rules**

1. Returns `true` if **any** matched element has the class.
2. Only one class name can be checked at a time.
3. Returns `false` if the set is empty.
4. Does not check inline styles; only the `class` attribute.
5. Works with SVG and XML documents as of jQuery 1.12/2.2.

**Constraints and Limitations**

- Cannot check multiple classes in a single call.
- Does not return which element has the class; only whether at least one does.

### Multiple Annotated Complete Code Examples

**Example 1: Checking for a Class**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.hasClass() — Basic Usage</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="mydiv" class="foo bar"></div>
  <p class="intro">Intro paragraph</p>

  <script>
    $(function () {
      // Step 1: Check for a class that exists
      console.log("Has 'foo':", $("#mydiv").hasClass("foo"));    // true
      console.log("Has 'bar':", $("#mydiv").hasClass("bar"));    // true

      // Step 2: Check for a class that does not exist
      console.log("Has 'quux':", $("#mydiv").hasClass("quux"));  // false

      // Step 3: Use in a conditional
      if ($("#mydiv").hasClass("foo")) {
        console.log("It has the foo class — applying style");
        $("#mydiv").css("border", "2px solid blue");
      }

      // Step 4: Check multiple elements (returns true if ANY has the class)
      console.log("Any p has 'intro':", $("p").hasClass("intro")); // true
      console.log("Any p has 'outro':", $("p").hasClass("outro")); // false
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Has 'foo': true
Has 'bar': true
Has 'quux': false
It has the foo class — applying style
Any p has 'intro': true
Any p has 'outro': false
```

**Why This Output Occurs**
`.hasClass("foo")` returns `true` because the element has that class. `.hasClass("quux")` returns `false` because it does not. The conditional uses the boolean result to decide whether to apply a style. When multiple elements are matched, `.hasClass()` returns `true` if **any** of them has the class.

### Real-World Cases

- **Toggle Logic**: `if ($("#menu").hasClass("open")) { closeMenu(); } else { openMenu(); }`.
- **Form Validation**: `if ($("#email").hasClass("error")) { showError(); }`.
- **Navigation State**: `if ($("li").hasClass("active")) { highlightParent(); }`.
- **Plugin State**: `if ($("#widget").hasClass("initialized")) { return; }`.

### References

- jQuery API — .hasClass() – https://api.jquery.com/hasClass/
- W3Schools — jQuery hasClass() Method – https://www.w3schools.com/jquery/html_hasclass.asp

---

## Core Concept 6: Reading Computed Styles vs. Setting Inline Styles

### Definitions

**Core Definition**
Reading computed styles means retrieving the final, rendered CSS values after all stylesheets have been applied; setting inline styles means writing style rules directly onto an element's `style` attribute, bypassing the stylesheet cascade.

**Technical Definition**
When `.css()` is used as a **getter**, it returns the **computed** style using `window.getComputedStyle()` (or legacy equivalents in older browsers). The computed style is the final value after all CSS rules, inheritance, and the cascade have been resolved. When `.css()` is used as a **setter**, it writes an **inline style** directly to the element's `style` attribute, which has the highest specificity in the cascade (excluding `!important` rules). The native DOM `.style` property and `.attr("style")` both return or manipulate only **inline** styles.

**Beginner-Friendly Explanation**
The **computed style** is what you actually see — the final colour, size, and font after all CSS files and rules have done their work. The **inline style** is what is written directly on the element, like a personal note that overrides the stylesheet. Reading `.css()` gives you the computed result; writing `.css()` creates an inline style.

### Purposes

- To understand the difference between what `.css()` returns when reading vs. writing.
- To choose the correct method for inspecting an element's actual rendered appearance.
- To understand why `.attr("style")` and `.css()` can return different values.
- To know when inline styles override stylesheet rules and when they do not.

### Syntax Rules and Structure

**Comparison Table**

| Aspect | Computed Style (`.css()` getter) | Inline Style (`.css()` setter) |
|---|---|---|
| What it represents | Final rendered value | Style rule written on the element |
| Source | All stylesheets + inheritance + cascade | The element's `style` attribute |
| Specificity | N/A (it is the result) | Highest (overrides stylesheet rules) |
| Retrievable via | `.css("property")` | `.attr("style")`, `element.style` |
| Dimensions | Almost always in pixels | Whatever unit was set |
| Colours | `rgb()` or `rgba()` format | Whatever format was set |

**Syntax Rules**

1. `.css("property")` returns the **computed** value for the first matched element.
2. `.attr("style")` returns the **inline** style attribute string.
3. The native `element.style.property` returns only inline styles.
4. Setting `.css("property", value)` writes an **inline** style.
5. Computed styles may not match the stylesheet value (e.g., `em` → `px`).

**Constraints and Limitations**

- Computed styles cannot be set directly; only inline styles can be set via `.css()`.
- The computed value may differ across browsers for the same element.
- Shorthand properties are not reliably retrievable from computed styles.

### Multiple Annotated Complete Code Examples

**Example 1: Computed vs. Inline Demonstration**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Computed vs Inline Styles</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #styled { font-size: 20px; color: navy; padding: 10px; }
  </style>
</head>
<body>
  <div id="styled">Styled by stylesheet</div>

  <script>
    $(function () {
      // --- COMPUTED STYLE (from stylesheet) ---
      console.log("Computed font-size:", $("#styled").css("font-size")); // "20px"
      console.log("Computed color:", $("#styled").css("color"));         // "rgb(0, 0, 128)"

      // --- INLINE STYLE (none yet) ---
      console.log("Inline style attr:", $("#styled").attr("style"));     // undefined

      // --- SET an inline style ---
      $("#styled").css("font-size", "30px");

      // --- COMPUTED now reflects the inline override ---
      console.log("After inline — computed:", $("#styled").css("font-size")); // "30px"

      // --- INLINE style attribute is now populated ---
      console.log("After inline — attr:", $("#styled").attr("style")); // "font-size: 30px;"

      // --- The stylesheet value is overridden, not replaced ---
      // The computed value is 30px because inline styles have higher specificity
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Computed font-size: 20px
Computed color: rgb(0, 0, 128)
Inline style attr: undefined
After inline — computed: 30px
After inline — attr: font-size: 30px;
```

**Why This Output Occurs**
The computed style initially reflects the stylesheet (`20px`). After `.css("font-size", "30px")` writes an inline style, the computed value changes to `30px` because inline styles override stylesheet rules. The `style` attribute now contains the inline rule.

### Real-World Cases

- **Layout Calculations**: Reading computed dimensions to position elements accurately.
- **Animation**: Reading the current computed state before starting an animation.
- **Debugging**: Comparing computed and inline styles to understand cascade conflicts.
- **Responsive Design**: Reading computed values that differ at different viewport sizes.

### References

- MDN Web Docs — getComputedStyle() – https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle
- Stack Overflow — jQuery .css() returns computed style – https://stackoverflow.com/questions/37890100/
- jQuery API — .css() – https://api.jquery.com/css/

---

## Core Concept 7: Performance Implications of Inline Style Modifications vs. Class Toggling

### Definitions

**Core Definition**
Inline style modifications via `.css()` write directly to the element's `style` attribute, while class toggling via `.addClass()` and related methods change the `class` attribute, allowing the browser's stylesheet engine to handle rendering.

**Technical Definition**
Setting inline styles via `.css()` injects CSS property values directly into each element's `style` attribute, forcing the browser to recalculate the element's style and potentially trigger a **reflow** (layout recalculation) and **repaint**. Class toggling, by contrast, changes a single attribute and lets the browser's existing stylesheet rules apply, which is generally more efficient because the styles are already parsed and cached. The performance difference is most pronounced when many elements are affected or when the operation is performed frequently (e.g., in an animation loop). The general recommendation is to use classes for styling and reserve `.css()` for reading computed values or making one-off, dynamic adjustments.

**Beginner-Friendly Explanation**
Changing styles with `.css()` is like writing new rules on every element individually — it works, but it is more work for the browser. Changing classes is like swapping a label — the stylesheet already knows what each label means, so the browser just applies the existing rules. For most everyday tasks, the difference is tiny. But when you are updating hundreds of elements or doing it many times per second, classes are noticeably faster.

### Purposes

- To understand why class toggling is generally preferred over inline style modification.
- To make informed decisions about when to use `.css()` vs. `.addClass()`.
- To optimise performance in applications that manipulate styles frequently.
- To avoid unnecessary reflows and repaints in animation-heavy or data-heavy interfaces.

### Syntax Rules and Structure

**Performance Comparison**

| Operation | Relative Speed | DOM Impact | Best For |
|---|---|---|---|
| `.addClass()` / `.removeClass()` | Faster | Single attribute change | State-based styling, themes, animations |
| `.css("prop")` (getter) | Moderate | No DOM change | Reading values for logic |
| `.css("prop", value)` (setter, single) | Slower | Inline style injection per element | One-off dynamic adjustments |
| `.css({ ... })` (setter, multiple) | Slowest | Multiple inline style injections | Rare; prefer classes |

**Syntax Rules**

1. Use classes for styling that can be pre-defined in a stylesheet.
2. Use `.css()` for reading computed values or setting values that cannot be pre-defined.
3. Avoid calling `.css()` inside loops; batch changes or use classes instead.
4. For frequently toggled states, use `.toggleClass()` rather than repeated `.css()` calls.
5. When setting multiple properties, prefer a single class over multiple `.css()` calls.

**Constraints and Limitations**

- Performance differences are most noticeable on large DOM trees or in high-frequency operations.
- Modern browsers optimise both approaches, but class-based styling remains the recommended practice.
- Inline styles always override stylesheet rules, which can make debugging more difficult.

### Multiple Annotated Complete Code Examples

**Example 1: Class Toggling vs. Inline Style (Performance Comparison)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Performance — Classes vs Inline Styles</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .highlight { background-color: yellow; font-weight: bold; border: 1px solid orange; }
  </style>
</head>
<body>
  <div class="item">Item 1</div>
  <div class="item">Item 2</div>
  <div class="item">Item 3</div>

  <script>
    $(function () {
      // --- Approach A: Inline styles via .css() ---
      console.time("css() single");
      $(".item").css("background-color", "yellow");
      console.timeEnd("css() single");

      console.time("css() multiple");
      $(".item").css({
        "background-color": "yellow",
        "font-weight": "bold",
        "border": "1px solid orange"
      });
      console.timeEnd("css() multiple");

      // --- Approach B: Class toggling ---
      console.time("addClass()");
      $(".item").addClass("highlight");
      console.timeEnd("addClass()");

      // The class approach typically shows a lower time value
      // because the browser already has the styles parsed
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
css() single: 0.300ms (approximate)
css() multiple: 0.800ms (approximate)
addClass(): 0.200ms (approximate)
```

**Why This Output Occurs**
The `.css()` calls inject inline styles into each element, requiring the browser to parse and apply new style rules. `.addClass()` simply changes the class attribute, and the browser applies the pre-parsed stylesheet rule. The timing will vary by browser and hardware, but class-based styling is consistently faster.

### Real-World Cases

- **Theme Switching**: Use `$("body").removeClass("light").addClass("dark")` instead of setting dozens of `.css()` properties.
- **Animation**: Use CSS transitions triggered by class toggles instead of animating inline styles with `.css()`.
- **List Highlighting**: Use `.addClass("selected")` on list items rather than setting `background-color` inline on each.
- **Dynamic Theming**: Use CSS custom properties updated via `.css("--var", value)` on a single root element, rather than styling individual elements.

### References

- Stack Overflow — jQuery: Insert inline CSS or make a CSS class and add – https://stackoverflow.com/questions/8188602/
- Stack Overflow — What is faster? jQuery css(), jQuery addclass() or just css – https://stackoverflow.com/questions/12185736/
- SSW — Do you avoid making changes to individual CSS styles using jQuery? – https://www.ssw.com.au/rules/do-you-avoid-making-changes-to-individual-css-styles-using-jquery
- jQuery Learning Center — CSS, Styling, & Dimensions – https://learn.jquery.com/using-jquery-core/css-styling-dimensions/

---

## Summary Table: jQuery CSS Class and Style Manipulation at a Glance

| Method | Operates On | Getter Scope | Setter Effect | Type Returned | Key Use |
|---|---|---|---|---|---|
| `.css()` | Inline style / computed style | First matched element | All matched elements | String / Object / jQuery | Read/write individual styles |
| `.addClass()` | `class` attribute | N/A | Adds classes | jQuery | Add visual state |
| `.removeClass()` | `class` attribute | N/A | Removes classes (all if no args) | jQuery | Remove visual state |
| `.toggleClass()` | `class` attribute | N/A | Toggles classes | jQuery | Alternate visual state |
| `.hasClass()` | `class` attribute | N/A | N/A (test only) | Boolean | Check visual state |

### Decision Guide

| Goal | Use | Example |
|---|---|---|
| Apply a pre-defined visual state | `.addClass()` | `$("#tab").addClass("active")` |
| Remove a visual state | `.removeClass()` | `$("#tab").removeClass("active")` |
| Toggle a state on/off | `.toggleClass()` | `$("#menu").toggleClass("open")` |
| Check if a state is active | `.hasClass()` | `if ($("#menu").hasClass("open"))` |
| Read the computed value of a property | `.css()` getter | `$("#el").css("width")` |
| Set a one-off inline style | `.css()` setter | `$("#el").css("margin-left", "10px")` |
| Apply many styles at once | `.addClass()` (preferred) | `$("#el").addClass("styled")` |

---

## General References

- jQuery API — Manipulation Category – https://api.jquery.com/category/manipulation/
- jQuery API — Attributes Category – https://api.jquery.com/category/attributes/
- jQuery API — .css() – https://api.jquery.com/css/
- jQuery API — .addClass() – https://api.jquery.com/addClass/
- jQuery API — .removeClass() – https://api.jquery.com/removeClass/
- jQuery API — .toggleClass() – https://api.jquery.com/toggleClass/
- jQuery API — .hasClass() – https://api.jquery.com/hasClass/
- jQuery Learning Center — CSS, Styling, & Dimensions – https://learn.jquery.com/using-jquery-core/css-styling-dimensions/
- MDN Web Docs — getComputedStyle() – https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle
- MDN Web Docs — CSS Custom Properties – https://developer.mozilla.org/en-US/docs/Web/CSS/--*
- Stack Overflow — jQuery: Insert inline CSS or make a CSS class and add – https://stackoverflow.com/questions/8188602/
- Stack Overflow — What is faster? jQuery css(), jQuery addclass() or just css – https://stackoverflow.com/questions/12185736/
- Stack Overflow — jQuery .css() vs .style – https://stackoverflow.com/questions/37890100/
- SSW — Do you avoid making changes to individual CSS styles using jQuery? – https://www.ssw.com.au/rules/do-you-avoid-making-changes-to-individual-css-styles-using-jquery
- W3Schools — jQuery CSS Methods – https://www.w3schools.com/jquery/jquery_css.asp