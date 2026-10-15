# Writing jQuery Plugins — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
Writing jQuery plugins is the practice of extending jQuery's functionality by adding custom methods to the `$.fn` prototype or utility functions to the `$` namespace, following established conventions for encapsulation, option management, and chainability.

**Technical Definition:**
A jQuery plugin is a JavaScript function assigned to `$.fn.pluginName` (for chainable instance methods) or `$.pluginName` (for utility functions). When assigned to `$.fn`, the function becomes available on every jQuery object and receives the jQuery collection as its `this` context. Plugins are typically wrapped in an Immediately Invoked Function Expression (IIFE) to protect the `$` alias and create private scope, and they follow conventions for option merging via `$.extend()`, default exposure, implicit iteration with `this.each()`, and return value management for chaining.

**Beginner-Friendly Explanation:**
A jQuery plugin is a way to teach jQuery a new trick. For example, if you frequently need to make text green, you can write a `greenify` plugin. After loading it, you can simply call `$("p").greenify()` instead of writing the color-changing code every time. It is like adding a custom button to a universal remote — once it is there, anyone can use it.

### Key Characteristics

- **Prototype Extension:** Plugins add methods to `$.fn`, making them available on every jQuery object.
- **IIFE Encapsulation:** Plugins are wrapped in Immediately Invoked Function Expressions to protect the `$` alias and create private scope.
- **Chainability Preservation:** Well-written plugins return `this` (the jQuery object) to allow method chaining.
- **Option Merging:** Plugins merge user-supplied options with defaults using `$.extend()`.
- **Implicit Iteration:** Plugins use `this.each()` to operate on every element in the collection.
- **Namespace Isolation:** Plugins use unique names and namespaces to avoid conflicts with other code.
- **Stateful vs. Stateless:** Simple plugins are stateless functions; complex plugins use the Widget Factory for state management and lifecycle methods.

### Prerequisites

- Solid understanding of JavaScript functions, closures, and the `this` keyword
- Working knowledge of jQuery selectors and DOM manipulation
- Familiarity with `$.fn` and how jQuery object methods work
- Understanding of event binding (`.on()`, `.off()`) and event namespacing
- Basic knowledge of IIFEs and `$.extend()`

### Related Programming Areas

- **jQuery Event System:** Plugins bind, unbind, and trigger events using `.on()`, `.off()`, and `.trigger()`.
- **jQuery UI Widget Factory:** An object-oriented base for building stateful, extensible plugins.
- **Design Patterns:** Module pattern, namespace pattern, and factory pattern are directly applicable.
- **Testing Frameworks:** QUnit, Jasmine, and Mocha for plugin unit testing.
- **Package Management:** Publishing to npm and the jQuery Plugin Registry.

### Core Concepts / Features

1. `$.fn.pluginName`
2. Plugin Function
3. Plugin Context
4. Plugin `this`
5. Plugin Return
6. Plugin Options Merging
7. `$.extend()`
8. Plugin Defaults
9. Plugin Methods
10. Plugin Private Methods
11. Plugin Public Methods
12. Plugin Instance Data
13. Plugin Lifecycle
14. Plugin Testing
15. Plugin Documentation
16. Plugin Publishing
17. Plugin Best Practices

---

## 1. `$.fn.pluginName`

### Definitions

**Core Definition:**
`$.fn.pluginName` is the assignment of a plugin function to jQuery's prototype, making the plugin available as a method on every jQuery object.

**Technical Definition:**
`$.fn` is an alias for `jQuery.prototype`. Assigning `$.fn.pluginName = function() {}` adds `pluginName` to the prototype chain of all jQuery objects, so any jQuery collection can invoke it. This is the standard mechanism for creating chainable, instance-based jQuery plugins.

**Beginner-Friendly Explanation:**
`$.fn` is like a master list of all the tools jQuery knows how to use. When you add your plugin to `$.fn`, you are adding a new tool to that list. After that, every jQuery object can use your tool by calling `$(selector).yourPlugin()`.

### Purposes

- To make a plugin available as a method on every jQuery object.
- To enable chainability by operating within the jQuery object context.
- To follow the standard convention for jQuery plugin authoring.
- To allow implicit iteration over multiple elements using `this.each()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    $.fn.pluginName = function(options) {
        // plugin implementation
        return this.each(function() {
            // per-element logic
        });
    };
}(jQuery));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `;` | Leading semicolon protects against concatenated scripts. |
| `(function($) { ... }(jQuery))` | IIFE that receives `jQuery` and names it `$` locally. |
| `$.fn.pluginName` | Assignment of the plugin function to the jQuery prototype. |
| `return this.each(...)` | Iterates over elements and returns the jQuery object for chaining. |

**Syntax Rules:**

1. Always wrap the plugin in an IIFE to protect the `$` alias.
2. Assign the plugin to `$.fn` for instance methods or `$` for utility functions.
3. Return `this` from the plugin (or from `this.each()`) to preserve chainability.
4. Use `this.each()` to handle multiple elements implicitly.

**Constraints and Limitations:**

- Plugins assigned to `$.fn` become available on every jQuery object, potentially causing conflicts.
- Poorly written plugins can break chaining by returning a value other than `this`.
- Plugins that modify global state can cause issues in multi-plugin environments.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Plugin Assignment**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.fn.pluginName</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <p class="target">Paragraph 1</p>
    <p class="target">Paragraph 2</p>

    <script>
        ;(function($) {
            // Assign the plugin to $.fn
            $.fn.highlight = function(color) {
                // 'this' is the jQuery object
                return this.each(function() {
                    $(this).css("background-color", color || "yellow");
                });
            };
        }(jQuery));

        // Use the plugin
        $(".target").highlight("lightblue").addClass("highlighted");

        console.log("Highlighted: " + $(".highlighted").length);
        console.log("Bg: " + $(".target").first().css("background-color"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Highlighted: 2
Bg: rgb(173, 216, 230)
```

**Why This Code Produces That Result:**
The IIFE receives `jQuery` as `$`, ensuring the plugin uses the correct reference even if `$.noConflict()` is called. The plugin is assigned to `$.fn.highlight`, making it available on all jQuery objects. Inside the plugin, `this` refers to the jQuery collection, so `this.each()` iterates over all selected elements. Returning `this.each(...)` preserves chainability, allowing `.addClass("highlighted")` to be chained. Both `<p>` elements receive the background color and the class.

---

## 2. Plugin Function

### Definitions

**Core Definition:**
The plugin function is the JavaScript function assigned to `$.fn.pluginName` that contains the plugin's logic and is invoked when the plugin is called on a jQuery collection.

**Technical Definition:**
The plugin function receives the user-supplied options object as its first argument and operates in the context of the jQuery collection (`this`). It typically merges options with defaults, iterates over the collection with `this.each()`, and returns `this` for chainability. The function is defined inside an IIFE to create a private scope for helper functions and variables.

**Beginner-Friendly Explanation:**
The plugin function is the actual code that runs when you call your plugin. It is the "brain" of the plugin — it takes the options you pass, loops through the elements you selected, and does the work you designed it to do.

### Purposes

- To contain the core logic of the plugin.
- To receive and process user-supplied options.
- To iterate over the jQuery collection and perform per-element operations.
- To return a value that supports chaining.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$.fn.pluginName = function(options) {
    var settings = $.extend({}, $.fn.pluginName.defaults, options);
    return this.each(function() {
        var $this = $(this);
        // Per-element logic using $this and settings
    });
};
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `function(options)` | The plugin function, receiving user options. |
| `var settings = $.extend(...)` | Merges user options with defaults. |
| `this.each(function() { ... })` | Iterates over each element in the collection. |
| `var $this = $(this)` | Caches the current DOM element as a jQuery object. |
| `return this.each(...)` | Returns the jQuery object for chaining. |

**Syntax Rules:**

1. The plugin function should accept an `options` object as its first parameter.
2. Always merge user options with defaults using `$.extend({}, defaults, options)`.
3. Use `this.each()` for implicit iteration over multiple elements.
4. Cache `$(this)` inside the `each()` callback as `$this` to avoid repeated wrapping.
5. Return `this` (or the result of `this.each()`) to preserve chainability.

**Constraints and Limitations:**

- The plugin function must not use arrow functions if it needs to access `this` as the jQuery collection.
- If the plugin returns a value other than `this`, chainability is broken for that method.
- The plugin function is called once per jQuery collection, not once per element.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Plugin Function with Option Merging**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Function</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="box">Box 1</div>
    <div class="box">Box 2</div>

    <script>
        ;(function($) {
            $.fn.boxStyle = function(options) {
                // Merge user options with defaults
                var settings = $.extend({}, $.fn.boxStyle.defaults, options);

                // Iterate over each element
                return this.each(function() {
                    var $this = $(this);
                    $this.css({
                        backgroundColor: settings.bgColor,
                        padding: settings.padding,
                        border: settings.border
                    });
                });
            };

            $.fn.boxStyle.defaults = {
                bgColor: "#f0f0f0",
                padding: "10px",
                border: "1px solid #ccc"
            };
        }(jQuery));

        // Use with default options
        $(".box").boxStyle();
        console.log("Default bg: " + $(".box").first().css("background-color"));

        // Use with custom options
        $(".box").boxStyle({ bgColor: "#e0e0e0", padding: "20px" });
        console.log("Custom bg: " + $(".box").first().css("background-color"));
        console.log("Custom padding: " + $(".box").first().css("padding"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Default bg: rgb(240, 240, 240)
Custom bg: rgb(224, 224, 224)
Custom padding: 20px
```

**Why This Code Produces That Result:**
The plugin function receives the `options` object and merges it with `$.fn.boxStyle.defaults` using `$.extend({}, defaults, options)`. The empty `{}` prevents mutation of the defaults object. The `this.each()` loop applies the merged styles to every `.box` element. The first call uses all defaults; the second call overrides `bgColor` and `padding` while keeping the default `border`. Returning `this.each(...)` preserves chainability.

---

## 3. Plugin Context

### Definitions

**Core Definition:**
Plugin context refers to the value of `this` inside the plugin function, which is the jQuery collection on which the plugin was called.

**Technical Definition:**
When a plugin function is invoked as `$(selector).pluginName()`, jQuery sets the `this` context of the plugin function to the jQuery object representing the selected elements. Inside the `this.each()` callback, however, `this` refers to the individual DOM element, not the jQuery object. This distinction is critical for correctly accessing elements and their properties.

**Beginner-Friendly Explanation:**
Context is about "what am I working with right now." When you are in the main plugin function, `this` is the whole group of elements you selected. When you are inside the `each` loop, `this` is just one element from that group. Understanding this difference helps you write code that works correctly for one element or many.

### Purposes

- To access the jQuery collection of selected elements.
- To iterate over elements individually using `this.each()`.
- To distinguish between the collection and individual elements.
- To correctly cache and manipulate DOM elements within the plugin.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$.fn.pluginName = function() {
    // 'this' is the jQuery collection
    return this.each(function() {
        // 'this' is the individual DOM element
        var $this = $(this); // Wrap the DOM element
    });
};
```

**Component Breakdown:**

| Context | Value | Usage |
|---------|-------|-------|
| Inside plugin function | jQuery collection | `this.each()`, `this.length` |
| Inside `each()` callback | DOM element | `$(this)` to wrap, `this.tagName` for native properties |

**Syntax Rules:**

1. Inside the plugin function, `this` is the jQuery object; do not wrap it in `$()`.
2. Inside the `each()` callback, `this` is the native DOM element; wrap it in `$()` to use jQuery methods.
3. Cache `$(this)` as `$this` inside the `each()` callback if you need to reference it multiple times.
4. Use `this` (without `$`) for native DOM properties like `tagName`, `id`, or `className`.

**Constraints and Limitations:**

- Arrow functions do not have their own `this` context; do not use them for plugin functions or `each()` callbacks if you need `this` to refer to the element.
- The context changes between the plugin function and the `each()` callback; confusing them leads to bugs.
- `this` inside nested functions (e.g., event handlers) may not refer to the element unless explicitly bound.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Distinguishing Context Levels**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Context</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="item" data-id="1">Item 1</div>
    <div class="item" data-id="2">Item 2</div>

    <script>
        ;(function($) {
            $.fn.showContext = function() {
                // 'this' is the jQuery collection
                console.log("Collection length: " + this.length);

                return this.each(function() {
                    // 'this' is the DOM element
                    var $this = $(this);
                    console.log("Tag: " + this.tagName);
                    console.log("Data ID: " + $this.data("id"));
                    console.log("Text: " + $this.text());
                });
            };
        }(jQuery));

        $(".item").showContext();
    </script>
</body>
</html>
```

**Expected Output:**
```
Collection length: 2
Tag: DIV
Data ID: 1
Text: Item 1
Tag: DIV
Data ID: 2
Text: Item 2
```

**Why This Code Produces That Result:**
Inside the plugin function, `this` is the jQuery collection of two `.item` elements, so `this.length` is 2. Inside the `each()` callback, `this` is the native DOM element for each iteration. `this.tagName` accesses the native property directly. `$this.data("id")` wraps the element and uses jQuery's `.data()` method to retrieve the `data-id` attribute. This demonstrates the distinction between the jQuery collection context and the individual element context.

---

## 4. Plugin `this`

### Definitions

**Core Definition:**
Plugin `this` refers to the value of the `this` keyword at different points within a plugin, which varies between the jQuery collection, the DOM element, and possibly other objects depending on the function scope.

**Technical Definition:**
In a jQuery plugin assigned to `$.fn`, `this` at the top level of the plugin function is the jQuery collection. Inside `this.each()`, `this` is the raw DOM element. Inside event handlers bound by the plugin, `this` is the element that triggered the event (unless overridden by `.call()` or `.apply()`). Understanding these context shifts is essential for writing correct plugin code.

**Beginner-Friendly Explanation:**
`this` is like a chameleon — it changes depending on where you are in the code. At the top of your plugin, it means "all the elements I selected." Inside the each loop, it means "the one element I am working on right now." Inside an event handler, it means "the element that was clicked." Knowing where you are tells you what `this` means.

### Purposes

- To access the jQuery collection at the plugin level.
- To access individual DOM elements within iteration.
- To access the event-triggering element inside handlers.
- To correctly pass context to callbacks using `.call()` or `.apply()`.

### Syntax Rules and Structure

**Complete General Syntax (Context by Scope):**

```javascript
$.fn.pluginName = function() {
    // this = jQuery collection
    return this.each(function() {
        // this = DOM element
        $(this).on("click", function() {
            // this = the clicked element (DOM element)
        });
    });
};
```

**Component Breakdown:**

| Scope | `this` Value | jQuery Wrapping Needed? |
|-------|-------------|------------------------|
| Plugin function top level | jQuery collection | No |
| `each()` callback | DOM element | Yes: `$(this)` |
| Event handler inside plugin | DOM element (trigger) | Yes: `$(this)` |
| Callback invoked via `.call()` | Whatever is passed | Depends |

**Syntax Rules:**

1. At the plugin function's top level, `this` is the jQuery collection; do not wrap it in `$()`.
2. Inside `this.each()`, `this` is the DOM element; wrap it as `$(this)` for jQuery methods.
3. Inside event handlers, `this` is the element that triggered the event; wrap it as `$(this)`.
4. Use `.call(this, args)` or `.apply(this, args)` to control `this` when invoking callbacks.

**Constraints and Limitations:**

- Arrow functions do not bind their own `this`; avoid them in plugin functions and `each()` callbacks.
- `this` inside asynchronous callbacks (e.g., `setTimeout`) may not refer to the element unless captured in a variable.
- Nested functions create new `this` contexts unless explicitly bound.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Tracking `this` Across Scopes**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin this</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <button class="btn" data-id="1">Button 1</button>
    <button class="btn" data-id="2">Button 2</button>
    <div id="log"></div>

    <script>
        ;(function($) {
            $.fn.trackable = function() {
                // 'this' = jQuery collection
                var $collection = this;
                console.log("Collection: " + $collection.length);

                return this.each(function() {
                    // 'this' = DOM element
                    var $element = $(this);
                    var id = $element.data("id");

                    // Bind a click handler
                    $element.on("click.trackable", function() {
                        // 'this' = the clicked DOM element
                        $("#log").append("<p>Clicked: " + $(this).text() +
                            " (id=" + $(this).data("id") + ")</p>");
                    });
                });
            };
        }(jQuery));

        $(".btn").trackable();
    </script>
</body>
</html>
```

**Expected Output (after clicking Button 1 and Button 2):**
```
Collection: 2
Clicked: Button 1 (id=1)
Clicked: Button 2 (id=2)
```

**Why This Code Produces That Result:**
At the top of the plugin function, `this` is the jQuery collection of two buttons, so `$collection.length` is 2. Inside the `each()` callback, `this` is the individual DOM button element, which is wrapped as `$element`. Inside the click handler, `this` is the specific button that was clicked, so `$(this).text()` returns the correct button's text. Each scope has a different `this`, and the code correctly handles each one.

---

## 5. Plugin Return

### Definitions

**Core Definition:**
Plugin return refers to the value returned from the plugin function, which determines whether the plugin supports jQuery method chaining.

**Technical Definition:**
A chainable plugin returns `this` (the jQuery object) or `this.each(...)` (which returns the jQuery object). If a plugin returns a different value (e.g., a string, number, or DOM element), chainability is broken for that method. Returning `this` is a best practice for all plugins that modify state, while getter plugins (e.g., `.width()`, `.text()`) intentionally return values.

**Beginner-Friendly Explanation:**
Returning `this` from your plugin is like saying "I'm done with my part, here's the same group of elements for the next method to work on." If you return something else, the chain breaks and the next jQuery method has nothing to work with.

### Purposes

- To enable jQuery method chaining after the plugin call.
- To maintain consistency with jQuery's fluent API style.
- To allow multiple operations in a single expression.
- To distinguish between chainable action plugins and value-returning getter plugins.

### Syntax Rules and Structure

**Complete General Syntax (Chainable Plugin):**

```javascript
$.fn.pluginName = function() {
    return this.each(function() {
        // per-element logic
    });
};
```

**Complete General Syntax (Getter Plugin):**

```javascript
$.fn.getSomething = function() {
    return this.first().data("something");
};
```

**Component Breakdown:**

| Return Value | Chainable? | Use Case |
|-------------|-----------|----------|
| `this` | Yes | Action plugins that modify elements. |
| `this.each(...)` | Yes | Iterating action plugins. |
| A string/number/object | No | Getter plugins. |
| `undefined` | No | Plugins that do not return anything (bad practice). |

**Syntax Rules:**

1. Return `this` or `this.each(...)` from action plugins to preserve chainability.
2. Use `return this.each(...)` rather than `this.each(...)` followed by `return this`.
3. For getter plugins, return the computed value and document that chainability is broken.
4. Do not return `$(this)` inside the plugin function unless `this` is a DOM element (it is already a jQuery object).

**Constraints and Limitations:**

- Returning a value other than `this` breaks chaining for that method.
- Getter plugins that return values cannot be chained with subsequent jQuery methods.
- Some plugins intentionally return values (e.g., `.data()`, `.width()`) and are not chainable.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Chainable vs. Getter Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Return</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="box" data-value="10">Box 1</div>
    <div class="box" data-value="20">Box 2</div>

    <script>
        ;(function($) {
            // Chainable action plugin
            $.fn.setColor = function(color) {
                return this.each(function() {
                    $(this).css("color", color);
                });
            };

            // Getter plugin
            $.fn.firstValue = function() {
                return this.first().data("value");
            };
        }(jQuery));

        // Chainable usage
        $(".box").setColor("red").addClass("colored");
        console.log("Colored: " + $(".colored").length); // Output: 2

        // Getter usage (chain broken after)
        var value = $(".box").firstValue();
        console.log("First value: " + value); // Output: 10
        // $(".box").firstValue().addClass(...) — ERROR
    </script>
</body>
</html>
```

**Expected Output:**
```
Colored: 2
First value: 10
```

**Why This Code Produces That Result:**
`setColor` returns `this.each(...)`, which returns the jQuery object, so `.addClass("colored")` can be chained. `firstValue` returns a number (the data value of the first element), so chainability is broken. This demonstrates the intentional distinction between chainable action plugins and value-returning getter plugins. Both are valid patterns, but they must be documented clearly.

---

## 6. Plugin Options Merging

### Definitions

**Core Definition:**
Plugin options merging is the process of combining user-supplied configuration options with the plugin's default settings to produce a complete configuration object.

**Technical Definition:**
Options merging is performed using `$.extend({}, $.fn.pluginName.defaults, options)`. The first argument is an empty object that receives the merged properties, the second is the defaults object, and the third is the user-supplied options. This creates a new settings object without mutating either the defaults or the user's options object.

**Beginner-Friendly Explanation:**
Options merging is like filling out a form with default values already printed on it. If the user fills in a field, their value is used. If they leave it blank, the default value stays. The result is a complete form with every field filled in.

### Purposes

- To provide sensible defaults while allowing user customization.
- To avoid mutating the defaults object across multiple plugin calls.
- To support both global default overrides and per-call overrides.
- To simplify plugin configuration with a single options object.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
var settings = $.extend({}, $.fn.pluginName.defaults, options);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `{}` | The target object that receives merged properties (prevents mutation). |
| `$.fn.pluginName.defaults` | The plugin's default settings. |
| `options` | The user-supplied options object. |
| `settings` | The merged configuration used by the plugin. |

**Syntax Rules:**

1. Always pass an empty object `{}` as the first argument to `$.extend()` to avoid mutating the defaults.
2. User options override defaults on a per-property basis (shallow merge by default).
3. For deep merging (nested objects), pass `true` as the first argument: `$.extend(true, {}, defaults, options)`.
4. Expose defaults as `$.fn.pluginName.defaults` for global customization.
5. Document all available options and their default values.

**Constraints and Limitations:**

- Shallow merging does not handle nested objects; use deep merging for complex configurations.
- Global defaults modification affects all future plugin calls, not existing instances.
- Undocumented options are a common source of bugs and confusion.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Options Merging with Global and Per-Call Overrides**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Options Merging</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="item">Item 1</div>
    <div class="item">Item 2</div>

    <script>
        ;(function($) {
            $.fn.styled = function(options) {
                // Merge with empty {} to protect defaults
                var settings = $.extend({}, $.fn.styled.defaults, options);

                return this.each(function() {
                    $(this).css({
                        color: settings.color,
                        fontSize: settings.fontSize,
                        fontWeight: settings.fontWeight
                    });
                });
            };

            $.fn.styled.defaults = {
                color: "#333",
                fontSize: "14px",
                fontWeight: "normal"
            };
        }(jQuery));

        // Default usage
        $(".item").styled();
        console.log("Default color: " + $(".item").first().css("color"));

        // Global override
        $.fn.styled.defaults.color = "#0066cc";
        $(".item").styled();
        console.log("Global override: " + $(".item").first().css("color"));

        // Per-call override
        $(".item").styled({ color: "#cc0000", fontWeight: "bold" });
        console.log("Per-call color: " + $(".item").first().css("color"));
        console.log("Per-call weight: " + $(".item").first().css("font-weight"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Default color: rgb(51, 51, 51)
Global override: rgb(0, 102, 204)
Per-call color: rgb(204, 0, 0)
Per-call weight: 700
```

**Why This Code Produces That Result:**
The `$.extend({}, defaults, options)` call creates a new settings object each time, preventing mutation of the defaults. The first call uses all defaults. After `$.fn.styled.defaults.color` is changed to blue, subsequent calls use blue unless overridden per-call. The per-call options `{ color: "#cc0000", fontWeight: "bold" }` override both the global defaults and the original defaults for that specific invocation.

---

## 7. `$.extend()`

### Definitions

**Core Definition:**
`$.extend()` is a jQuery utility function that merges the properties of two or more objects into a target object.

**Technical Definition:**
`$.extend([deep], target, object1 [, objectN])` copies the enumerable properties of each source object into the target object. If `deep` is `true`, the merge is recursive, which is useful for merging nested option objects. When used in plugins, `$.extend({}, defaults, options)` creates a new object containing the merged settings without modifying either source object.

**Beginner-Friendly Explanation:**
`$.extend()` is like combining two decks of cards. You start with a blank table (the empty `{}`), deal out all the cards from the first deck (the defaults), and then deal out all the cards from the second deck (the user's options). If a card appears in both decks, the second one wins. The result is a complete deck with no duplicates.

### Purposes

- To merge user-supplied options with plugin defaults.
- To create a new object without mutating the source objects.
- To support deep merging of nested configuration objects.
- To serve as the foundation for plugin option management.

### Syntax Rules and Structure

**Complete General Syntax (Shallow Merge):**

```javascript
$.extend(target, object1, object2);
```

**Complete General Syntax (Deep Merge):**

```javascript
$.extend(true, target, object1, object2);
```

**Complete General Syntax (Plugin Usage):**

```javascript
var settings = $.extend({}, $.fn.pluginName.defaults, options);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `true` (optional) | Enables deep (recursive) merging. |
| `target` | The object to receive merged properties. |
| `object1, object2` | Source objects whose properties are copied. |

**Syntax Rules:**

1. Always pass an empty object `{}` as the target when merging options to avoid mutating the defaults.
2. Use `true` as the first argument for deep merging of nested objects.
3. Properties from later objects override properties from earlier objects.
4. `$.extend()` returns the modified target object.
5. For plugin defaults, define them as `$.fn.pluginName.defaults` and merge with user options.

**Constraints and Limitations:**

- Shallow merging does not handle nested objects; nested objects are replaced entirely rather than merged.
- Deep merging can be slower for very large objects.
- `$.extend()` does not clone functions; it copies references.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Shallow vs. Deep Merging**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.extend()</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="output"></div>

    <script>
        // Shallow merge
        var defaults = {
            color: "red",
            nested: { a: 1, b: 2 }
        };
        var options = {
            color: "blue",
            nested: { c: 3 }
        };

        var shallow = $.extend({}, defaults, options);
        console.log("Shallow color: " + shallow.color);         // blue
        console.log("Shallow nested: " + JSON.stringify(shallow.nested)); // {"c":3}

        // Deep merge
        var deep = $.extend(true, {}, defaults, options);
        console.log("Deep color: " + deep.color);               // blue
        console.log("Deep nested: " + JSON.stringify(deep.nested)); // {"a":1,"b":2,"c":3}

        // Verify defaults are not mutated
        console.log("Defaults color: " + defaults.color);       // red
        console.log("Defaults nested: " + JSON.stringify(defaults.nested)); // {"a":1,"b":2}
    </script>
</body>
</html>
```

**Expected Output:**
```
Shallow color: blue
Shallow nested: {"c":3}
Deep color: blue
Deep nested: {"a":1,"b":2,"c":3}
Defaults color: red
Defaults nested: {"a":1,"b":2}
```

**Why This Code Produces That Result:**
The shallow merge replaces the entire `nested` object with the user's `nested` object, losing the `a` and `b` properties. The deep merge recursively merges the nested objects, preserving `a` and `b` while adding `c`. Both merges correctly override the `color` property with the user's value. The empty `{}` target ensures that the `defaults` object is not mutated, as confirmed by the final console output.

---

## 8. Plugin Defaults

### Definitions

**Core Definition:**
Plugin defaults are the predefined configuration values that a plugin uses when no user options are provided.

**Technical Definition:**
Defaults are stored as a property on the plugin function (e.g., `$.fn.pluginName.defaults`) and merged with user options via `$.extend({}, defaults, options)`. Exposing defaults publicly allows users to modify them globally with a single line of code, affecting all future plugin calls.

**Beginner-Friendly Explanation:**
Defaults are the "factory settings" of your plugin. If the user does not specify a color, the plugin uses red. If the user does not specify a speed, the plugin uses 300ms. Exposing defaults lets users change these factory settings globally without passing options every time.

### Purposes

- To provide sensible out-of-the-box behavior without requiring configuration.
- To allow global customization with minimal code.
- To serve as documentation for available options.
- To enable per-call overrides while maintaining a baseline configuration.

### Syntax Rules and Structure

**Complete General Syntax (Defining Defaults):**

```javascript
$.fn.pluginName.defaults = {
    option1: defaultValue1,
    option2: defaultValue2
};
```

**Complete General Syntax (Using Defaults):**

```javascript
var settings = $.extend({}, $.fn.pluginName.defaults, options);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$.fn.pluginName.defaults` | Public property holding the default options. |
| `$.extend({}, defaults, options)` | Merges defaults with user options. |
| `$.fn.pluginName.defaults.option1 = newValue` | Modifies a default globally. |

**Syntax Rules:**

1. Define defaults **after** the plugin function, so the plugin can reference them.
2. Always merge with an empty `{}` to avoid mutating the defaults object.
3. Document the defaults object as the source of truth for available options.
4. Modify defaults before calling the plugin, not after initialization.

**Constraints and Limitations:**

- Changing defaults affects all **future** plugin calls, not existing instances.
- Defaults are shallow-merged by default; nested objects require deep merging.
- Exposing defaults means users can break the plugin by setting invalid values.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Exposing and Modifying Defaults**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Defaults</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <p class="text">Text 1</p>
    <p class="text">Text 2</p>

    <script>
        ;(function($) {
            $.fn.textStyle = function(options) {
                var settings = $.extend({}, $.fn.textStyle.defaults, options);
                return this.each(function() {
                    $(this).css({
                        color: settings.color,
                        fontSize: settings.fontSize
                    });
                });
            };

            // Define defaults
            $.fn.textStyle.defaults = {
                color: "#333",
                fontSize: "14px"
            };
        }(jQuery));

        // Default usage
        $(".text").textStyle();
        console.log("Default color: " + $(".text").first().css("color"));

        // Modify global defaults
        $.fn.textStyle.defaults.color = "#0066cc";
        $.fn.textStyle.defaults.fontSize = "18px";
        $(".text").textStyle();
        console.log("Modified color: " + $(".text").first().css("color"));
        console.log("Modified size: " + $(".text").first().css("font-size"));

        // Per-call override
        $(".text").textStyle({ color: "#cc0000" });
        console.log("Per-call color: " + $(".text").first().css("color"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Default color: rgb(51, 51, 51)
Modified color: rgb(0, 102, 204)
Modified size: 18px
Per-call color: rgb(204, 0, 0)
```

**Why This Code Produces That Result:**
The defaults are defined as `$.fn.textStyle.defaults`. When the plugin is called with no options, the defaults are used. Modifying `$.fn.textStyle.defaults.color` and `$.fn.textStyle.defaults.fontSize` changes the global defaults, affecting subsequent calls. Per-call options override both the modified global defaults and the original defaults for that specific invocation.

---

## 9. Plugin Methods

### Definitions

**Core Definition:**
Plugin methods are the callable functions exposed by a plugin, including the main plugin function and any additional public methods for interacting with plugin instances.

**Technical Definition:**
For stateless plugins, methods are functions assigned to `$.fn`. For stateful plugins, methods are defined as properties on the plugin instance or prototype and invoked by passing the method name as a string to the plugin function (e.g., `$("#el").myPlugin("methodName", args)`). Public methods are exposed for external use, while private methods are kept internal.

**Beginner-Friendly Explanation:**
Plugin methods are the actions your plugin can perform. A simple plugin might have just one method (the main function). A complex plugin (like a date picker) might have methods for `open()`, `close()`, `getDate()`, and `setDate()`. You call these methods by passing the method name as a string.

### Purposes

- To provide a public API for controlling and querying plugin instances.
- To enable stateful interactions beyond simple one-time initialization.
- To allow plugin users to invoke behavior programmatically.
- To expose extension points for third-party customization.

### Syntax Rules and Structure

**Complete General Syntax (Stateless Plugin Method):**

```javascript
$.fn.myPlugin = function() { /* main method */ };
$.fn.myPlugin.secondaryMethod = function() { /* additional method */ };
```

**Complete General Syntax (Stateful Plugin Methods):**

```javascript
$.fn.myPlugin = function(method) {
    if (methods[method]) {
        return methods[method].apply(this, Array.prototype.slice.call(arguments, 1));
    } else if (typeof method === "object" || !method) {
        return methods.init.apply(this, arguments);
    } else {
        $.error("Method " + method + " does not exist");
    }
};
```

**Component Breakdown:**

| Method Type | Invocation | Use Case |
|-------------|------------|----------|
| `$.fn.myPlugin` | `$(sel).myPlugin()` | Main plugin entry point. |
| Instance method | `$(sel).myPlugin("method", args)` | Stateful plugin method invocation. |
| Namespace method | `$.myPlugin.method()` | Utility function independent of elements. |

**Syntax Rules:**

1. Method names should be camelCase (e.g., `changeColor`, not `change_color`).
2. Private methods should be prefixed with an underscore (e.g., `_updateDisplay`).
3. Methods that return a value should not be chainable; methods that modify state should return `this`.
4. Expose only methods that are safe for public consumption.

**Constraints and Limitations:**

- Stateless plugins cannot maintain state between method calls; use `.data()` or the Widget Factory for stateful behavior.
- String-based method dispatch lacks IDE autocompletion and type checking.
- Adding too many public methods bloats the plugin's API surface.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Stateful Plugin with Public Methods**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Methods</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="counter">0</div>

    <script>
        ;(function($) {
            var methods = {
                init: function(options) {
                    var settings = $.extend({
                        initialValue: 0,
                        step: 1
                    }, options);
                    return this.each(function() {
                        var $this = $(this);
                        $this.data("counter", {
                            value: settings.initialValue,
                            step: settings.step
                        });
                        $this.text(settings.initialValue);
                    });
                },
                increment: function() {
                    return this.each(function() {
                        var $this = $(this);
                        var data = $this.data("counter");
                        data.value += data.step;
                        $this.text(data.value);
                    });
                },
                getValue: function() {
                    return this.first().data("counter").value;
                }
            };

            $.fn.counter = function(method) {
                if (methods[method]) {
                    return methods[method].apply(this,
                        Array.prototype.slice.call(arguments, 1));
                } else if (typeof method === "object" || !method) {
                    return methods.init.apply(this, arguments);
                } else {
                    $.error("Method " + method + " does not exist");
                }
            };
        }(jQuery));

        // Initialize
        $("#counter").counter({ initialValue: 10, step: 5 });
        console.log("Initial: " + $("#counter").text()); // 10

        // Call public methods
        $("#counter").counter("increment");
        console.log("After increment: " + $("#counter").text()); // 15

        console.log("getValue: " + $("#counter").counter("getValue")); // 15
    </script>
</body>
</html>
```

**Expected Output:**
```
Initial: 10
After increment: 15
getValue: 15
```

**Why This Code Produces That Result:**
The `methods` object holds the plugin's public methods. The main plugin function checks if the first argument is a string (a method name) or an object (options). If it is a method name, it calls that method with the remaining arguments. If it is an object or undefined, it calls `init`. State is stored on the element via `.data("counter")`, allowing the value to persist between method calls. The `getValue` method returns a value (breaking chainability), while `increment` returns `this` (preserving chainability).

---

## 10. Plugin Private Methods

### Definitions

**Core Definition:**
Plugin private methods are functions defined inside the plugin's IIFE closure that are not exposed to the public API and can only be called from within the plugin itself.

**Technical Definition:**
Private methods are declared as variables inside the IIFE (e.g., `function privateHelper() {}` or `var privateHelper = function() {}`) but outside the `$.fn` assignment. Because they are in the closure scope, they are accessible to the plugin function and any public methods defined in the same closure, but not from outside.

**Beginner-Friendly Explanation:**
Private methods are like internal tools in a workshop. Only the workers (the plugin's own functions) can use them. Customers (the plugin's users) cannot see or call them directly. This keeps the public API clean and prevents users from depending on implementation details.

### Purposes

- To encapsulate implementation details that should not be exposed publicly.
- To prevent users from accidentally breaking internal logic.
- To keep the public API surface small and focused.
- To allow refactoring internal helpers without breaking external code.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    // Private method
    function privateHelper(element, options) {
        // Internal logic
    }

    $.fn.pluginName = function(options) {
        var settings = $.extend({}, $.fn.pluginName.defaults, options);
        return this.each(function() {
            privateHelper(this, settings);
        });
    };
}(jQuery));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `function privateHelper()` | Private function inside the IIFE but outside `$.fn`. |
| `$.fn.pluginName` | Public plugin function that calls the private helper. |

**Syntax Rules:**

1. Define private methods inside the IIFE but outside the `$.fn` assignment.
2. Use function declarations or `var` assignments for private methods.
3. Private methods cannot be called from outside the IIFE closure.
4. Use a naming convention (e.g., leading underscore) for private methods in stateful plugins.

**Constraints and Limitations:**

- Private methods cannot be unit-tested directly from outside the closure (though they can be tested indirectly through public methods).
- Private methods cannot be overridden or extended by plugin users.
- Adding too many private methods can make the plugin harder to debug.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Private Helper in a Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Private Methods</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="item" data-price="10">Item 1</div>
    <div class="item" data-price="20">Item 2</div>

    <script>
        ;(function($) {
            // Private method — not exposed to the public
            function formatPrice(price) {
                return "$" + price.toFixed(2);
            }

            $.fn.showPrice = function(options) {
                var settings = $.extend({
                    prefix: "Price: "
                }, options);

                return this.each(function() {
                    var $this = $(this);
                    var price = parseFloat($this.data("price"));
                    // Use the private method
                    $this.text(settings.prefix + formatPrice(price));
                });
            };
        }(jQuery));

        // Use the plugin
        $(".item").showPrice();
        console.log($(".item").first().text()); // Price: $10.00
        console.log($(".item").last().text());  // Price: $20.00

        // Cannot call formatPrice from outside
        // console.log(formatPrice(10)); // ERROR: formatPrice is not defined
    </script>
</body>
</html>
```

**Expected Output:**
```
Price: $10.00
Price: $20.00
```

**Why This Code Produces That Result:**
The `formatPrice` function is defined inside the IIFE but outside `$.fn.showPrice`, making it private. The plugin function calls `formatPrice` internally to format the price. The plugin user cannot call `formatPrice` directly because it is not in the global scope or on the jQuery object. This keeps the public API clean and focused on the plugin's purpose.

---

## 11. Plugin Public Methods

### Definitions

**Core Definition:**
Plugin public methods are functions exposed as part of the plugin's API that can be called by plugin users to interact with or control the plugin.

**Technical Definition:**
For stateless plugins, public methods are additional functions assigned to the plugin function (e.g., `$.fn.myPlugin.format = function() {}`) or exposed on a namespace object (e.g., `$.myPlugin.publicMethod()`). For stateful plugins, public methods are defined on the plugin instance and invoked by passing the method name as a string to the main plugin function.

**Beginner-Friendly Explanation:**
Public methods are the tools you give to your users. They are the documented, supported ways to interact with your plugin. Users can call them to change settings, get information, or trigger behavior.

### Purposes

- To provide a documented API for plugin interaction.
- To allow users to query and modify plugin state.
- To expose extension points for customization.
- To enable programmatic control of plugin behavior.

### Syntax Rules and Structure

**Complete General Syntax (Namespace Public Method):**

```javascript
$.fn.myPlugin.format = function(text) {
    return "<strong>" + text + "</strong>";
};
```

**Complete General Syntax (Stateful Public Method):**

```javascript
$.fn.myPlugin = function(method) {
    if (methods[method]) {
        return methods[method].apply(this, Array.prototype.slice.call(arguments, 1));
    }
    // ...
};

var methods = {
    publicMethod: function() {
        // implementation
    }
};
```

**Component Breakdown:**

| Type | Definition | Invocation |
|------|-----------|------------|
| Namespace method | `$.fn.myPlugin.method = function() {}` | `$.fn.myPlugin.method()` |
| Instance method | Defined in `methods` object | `$(sel).myPlugin("method", args)` |
| Utility method | `$.myPlugin.method = function() {}` | `$.myPlugin.method()` |

**Syntax Rules:**

1. Public methods should be documented with their parameters and return values.
2. Public methods that modify state should return `this` or the plugin instance for chainability.
3. Public methods that return values break chainability but provide useful data.
4. Expose only methods that are safe and intended for public use.

**Constraints and Limitations:**

- Adding too many public methods bloats the plugin's API.
- Public methods become part of the plugin's contract and cannot be removed without breaking compatibility.
- String-based method dispatch lacks IDE autocompletion and type checking.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Namespace Public Method**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Public Methods</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <p class="text">Hello World</p>

    <script>
        ;(function($) {
            $.fn.boldify = function() {
                return this.each(function() {
                    var $this = $(this);
                    var markup = $this.html();
                    // Call the public method
                    $this.html($.fn.boldify.format(markup));
                });
            };

            // Public method — accessible as $.fn.boldify.format()
            $.fn.boldify.format = function(text) {
                return "<strong>" + text + "</strong>";
            };
        }(jQuery));

        // Use the plugin
        $(".text").boldify();
        console.log($(".text").html()); // <strong>Hello World</strong>

        // Use the public method directly
        var formatted = $.fn.boldify.format("Custom");
        console.log(formatted); // <strong>Custom</strong>
    </script>
</body>
</html>
```

**Expected Output:**
```
<strong>Hello World</strong>
<strong>Custom</strong>
```

**Why This Code Produces That Result:**
The `format` method is exposed as a property of the plugin function: `$.fn.boldify.format`. The main plugin function calls this public method internally. Users can also call the public method directly to format text without applying the plugin to an element. This provides both an internal helper and a public utility API.

---

## 12. Plugin Instance Data

### Definitions

**Core Definition:**
Plugin instance data refers to the state information stored on a DOM element using jQuery's `.data()` method, allowing the plugin to maintain per-element state across method calls.

**Technical Definition:**
Plugins store their instance (or state object) on the DOM element using `$(element).data("pluginName", instance)`. This allows the plugin to retrieve the instance later, check if it has already been initialized, and provide access to its methods and properties. jQuery automatically cleans up this data when elements are removed via `.remove()`, but not via `.detach()`.

**Beginner-Friendly Explanation:**
Instance data is like a locker for each element. When the plugin is applied to an element, it stores its state (settings, variables, etc.) in that element's locker. Later, when the user calls a method on the plugin, it retrieves the state from the locker. Each element has its own locker, so multiple elements can have independent plugin instances.

### Purposes

- To maintain per-element state for stateful plugins.
- To prevent re-initialization of a plugin on the same element.
- To provide access to the plugin instance from outside.
- To enable multiple independent plugin instances on different elements.

### Syntax Rules and Structure

**Complete General Syntax (Storing Instance Data):**

```javascript
$(element).data("pluginName", instance);
```

**Complete General Syntax (Retrieving Instance Data):**

```javascript
var instance = $(element).data("pluginName");
```

**Complete General Syntax (Checking for Existing Instance):**

```javascript
if (!$(element).data("pluginName")) {
    $(element).data("pluginName", new Plugin(element, options));
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `"pluginName"` | A unique key for the plugin's data. |
| `instance` | The plugin instance object containing state and methods. |
| `$(element).data(key)` | Retrieves the stored instance. |
| `$(element).data(key, value)` | Stores the instance. |

**Syntax Rules:**

1. Use a unique key (e.g., `"plugin_" + pluginName`) to avoid conflicts with other plugins.
2. Check if the instance already exists before creating a new one.
3. Store the instance immediately after creation.
4. jQuery automatically removes `.data()` when elements are removed via `.remove()`.

**Constraints and Limitations:**

- Instance data is not automatically cleaned up when elements are detached via `.detach()`.
- Storing too much data on elements can increase memory usage.
- The data key must be unique to avoid conflicts.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Stateful Plugin with Instance Data**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Instance Data</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="counter" data-initial="5">5</div>
    <div class="counter" data-initial="10">10</div>

    <script>
        ;(function($) {
            function Counter(element, options) {
                this.element = element;
                this.$element = $(element);
                this.options = $.extend({
                    step: 1
                }, options);
                this.value = parseInt(this.$element.data("initial"), 10);
            }

            Counter.prototype.increment = function() {
                this.value += this.options.step;
                this.$element.text(this.value);
                return this;
            };

            Counter.prototype.getValue = function() {
                return this.value;
            };

            $.fn.counter = function(options) {
                return this.each(function() {
                    var $this = $(this);
                    // Check if instance already exists
                    var instance = $this.data("counter");
                    if (!instance) {
                        // Create and store new instance
                        instance = new Counter(this, options);
                        $this.data("counter", instance);
                    }
                    // If options passed, update the instance
                    if (options) {
                        instance.options = $.extend(instance.options, options);
                    }
                });
            };
        }(jQuery));

        // Initialize both counters
        $(".counter").counter();

        // Increment the first counter
        $(".counter").first().data("counter").increment();
        console.log("Counter 1: " + $(".counter").first().text()); // 6

        // Increment the second counter
        $(".counter").last().data("counter").increment();
        console.log("Counter 2: " + $(".counter").last().text()); // 11

        // Verify independence
        console.log("Counter 1 again: " + $(".counter").first().text()); // 6
    </script>
</body>
</html>
```

**Expected Output:**
```
Counter 1: 6
Counter 2: 11
Counter 1 again: 6
```

**Why This Code Produces That Result:**
Each `.counter` element has its own `Counter` instance stored via `.data("counter")`. The first counter starts at 5 and increments to 6. The second counter starts at 10 and increments to 11. The instances are independent because each element has its own data store. Retrieving the instance via `.data("counter")` allows calling its public methods directly.

---

## 13. Plugin Lifecycle

### Definitions

**Core Definition:**
Plugin lifecycle refers to the distinct phases a plugin instance goes through from initialization to destruction, including creation, configuration, operation, and cleanup.

**Technical Definition:**
In the jQuery UI Widget Factory, the lifecycle is explicitly defined through `_create()` (initialization), `_init()` (re-initialization), `_setOption()` (option changes), public methods (operation), and `_destroy()` (cleanup). For manually written plugins, the lifecycle is implemented by the developer using instance data and cleanup methods.

**Beginner-Friendly Explanation:**
A plugin's lifecycle is like the life of a plant: it is planted (initialized), grows and changes (operates), and eventually is removed (destroyed). Understanding the lifecycle helps you know when to set up, when to respond to changes, and when to clean up.

### Purposes

- To provide a structured approach to plugin state management.
- To ensure proper initialization and cleanup.
- To allow options to be changed after initialization.
- To prevent memory leaks by cleaning up on destruction.

### Syntax Rules and Structure

**Complete General Syntax (Widget Factory Lifecycle):**

```javascript
$.widget("ns.myWidget", {
    options: { ... },

    _create: function() {
        // Initialization
    },

    _init: function() {
        // Re-initialization
    },

    _setOption: function(key, value) {
        // Handle option changes
    },

    _destroy: function() {
        // Cleanup
    }
});
```

**Complete General Syntax (Manual Lifecycle):**

```javascript
$.fn.myPlugin = function(options) {
    return this.each(function() {
        var $this = $(this);
        var instance = $this.data("myPlugin");
        if (!instance) {
            // Create
            instance = new MyPlugin(this, options);
            $this.data("myPlugin", instance);
        } else if (options) {
            // Update
            instance.setOptions(options);
        }
    });
};

$.fn.myPlugin.destroy = function() {
    return this.each(function() {
        var instance = $(this).data("myPlugin");
        if (instance) {
            instance.destroy();
            $(this).removeData("myPlugin");
        }
    });
};
```

**Component Breakdown:**

| Phase | Widget Factory Method | Manual Implementation |
|-------|----------------------|----------------------|
| Creation | `_create()` | Constructor function |
| Re-initialization | `_init()` | Check for existing instance |
| Option changes | `_setOption()` | `setOptions()` method |
| Operation | Public methods | Public methods |
| Destruction | `_destroy()` | `destroy()` method |

**Syntax Rules:**

1. The Widget Factory automatically calls `_create()` on first initialization.
2. `_destroy()` is called when the widget is destroyed via `.widgetName("destroy")`.
3. The base `destroy()` method unbinds namespaced events, removes instance data, and removes widget classes.
4. For manual plugins, implement cleanup explicitly in the `destroy()` method.

**Constraints and Limitations:**

- Manual lifecycle management requires more code and is error-prone.
- The Widget Factory adds a dependency on jQuery UI (or at least the widget factory file).
- Destroying a plugin without proper cleanup can lead to memory leaks.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Widget Factory Lifecycle**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Lifecycle</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
</head>
<body>
    <div id="widget">Widget</div>

    <script>
        $.widget("demo.lifecycleWidget", {
            options: {
                color: "red"
            },

            // Initialization
            _create: function() {
                console.log("_create: Initializing widget");
                this.element.css("color", this.options.color);
                this.element.on("click.lifecycleWidget", function() {
                    console.log("Widget clicked");
                });
            },

            // Re-initialization
            _init: function() {
                console.log("_init: Re-initializing widget");
            },

            // Option changes
            _setOption: function(key, value) {
                console.log("_setOption: " + key + " = " + value);
                this._super(key, value);
                if (key === "color") {
                    this.element.css("color", value);
                }
            },

            // Cleanup
            _destroy: function() {
                console.log("_destroy: Cleaning up widget");
                this.element.off(".lifecycleWidget");
                this.element.css("color", "");
            }
        });

        // Create
        $("#widget").lifecycleWidget();
        console.log("Color: " + $("#widget").css("color")); // red

        // Change option
        $("#widget").lifecycleWidget("option", "color", "blue");
        console.log("New color: " + $("#widget").css("color")); // blue

        // Destroy
        $("#widget").lifecycleWidget("destroy");
        console.log("After destroy: " + $("#widget").css("color")); // black
    </script>
</body>
</html>
```

**Expected Output:**
```
_create: Initializing widget
Color: rgb(255, 0, 0)
_setOption: color = blue
New color: rgb(0, 0, 255)
_destroy: Cleaning up widget
After destroy: rgb(0, 0, 0)
```

**Why This Code Produces That Result:**
The Widget Factory manages the lifecycle automatically. `_create` runs on initialization, setting the color and binding a namespaced click handler. `_setOption` runs when the `color` option is changed, updating the CSS. `_destroy` runs when the widget is destroyed, unbinding the namespaced event and clearing the color. The base `destroy()` method (called automatically) also removes instance data and widget classes.

---

## 14. Plugin Testing

### Definitions

**Core Definition:**
Plugin testing is the practice of writing automated tests to verify that a plugin behaves correctly under various conditions, using a JavaScript testing framework.

**Technical Definition:**
Plugin testing typically uses frameworks such as QUnit (the jQuery team's preferred framework), Jasmine, or Mocha. Tests cover initialization, option handling, method invocation, event triggering, and cleanup. QUnit is used by the jQuery project itself and provides a `#qunit-fixture` element for isolating DOM manipulations during tests.

**Beginner-Friendly Explanation:**
Testing your plugin means writing code that checks if your plugin works correctly. For example, you might write a test that says "when I call my plugin with a red color option, the element should turn red." If the element turns blue instead, the test fails, and you know something is wrong.

### Purposes

- To verify that the plugin works as expected.
- To catch regressions when making changes.
- To document the plugin's behavior through executable examples.
- To ensure the plugin works across different jQuery versions and browsers.
- To provide confidence when refactoring or adding features.

### Syntax Rules and Structure

**Complete General Syntax (QUnit Test):**

```javascript
QUnit.test("pluginName initializes correctly", function(assert) {
    var $fixture = $("#qunit-fixture");
    $fixture.append("<div id='test'>Test</div>");

    $("#test").pluginName({ color: "red" });

    assert.equal($("#test").css("color"), "rgb(255, 0, 0)",
        "Element color should be red");
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `QUnit.test()` | Defines a test case with a name and function. |
| `assert.equal()` | Asserts that two values are equal. |
| `#qunit-fixture` | A DOM container that is reset between tests. |

**Syntax Rules:**

1. Use a testing framework like QUnit, Jasmine, or Mocha.
2. Isolate tests using a fixture element (QUnit provides `#qunit-fixture`).
3. Test initialization, options, methods, events, and destruction.
4. Run tests against multiple jQuery versions to ensure compatibility.
5. Use assertions to verify expected outcomes.

**Constraints and Limitations:**

- Testing DOM manipulations requires a DOM environment (browser or jsdom).
- Asynchronous plugin behavior requires asynchronous test support (`assert.async()` in QUnit).
- Private methods cannot be tested directly; test them indirectly through public methods.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: QUnit Test for a Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Testing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://code.jquery.com/qunit/qunit-2.19.4.js"></script>
    <link rel="stylesheet" href="https://code.jquery.com/qunit/qunit-2.19.4.css">
</head>
<body>
    <div id="qunit"></div>
    <div id="qunit-fixture"></div>

    <script>
        // Plugin definition
        ;(function($) {
            $.fn.colorize = function(options) {
                var settings = $.extend({
                    color: "red"
                }, options);
                return this.each(function() {
                    $(this).css("color", settings.color);
                });
            };
        }(jQuery));

        // Test 1: Default color
        QUnit.test("colorize applies default color", function(assert) {
            var $fixture = $("#qunit-fixture");
            $fixture.append("<div id='test1'>Test</div>");
            $("#test1").colorize();
            assert.equal($("#test1").css("color"), "rgb(255, 0, 0)",
                "Default color should be red");
        });

        // Test 2: Custom color
        QUnit.test("colorize applies custom color", function(assert) {
            var $fixture = $("#qunit-fixture");
            $fixture.append("<div id='test2'>Test</div>");
            $("#test2").colorize({ color: "blue" });
            assert.equal($("#test2").css("color"), "rgb(0, 0, 255)",
                "Custom color should be blue");
        });

        // Test 3: Multiple elements
        QUnit.test("colorize applies to multiple elements", function(assert) {
            var $fixture = $("#qunit-fixture");
            $fixture.append("<div class='multi'>A</div><div class='multi'>B</div>");
            $(".multi").colorize({ color: "green" });
            assert.equal($(".multi").length, 2, "Should match 2 elements");
            assert.equal($(".multi").first().css("color"), "rgb(0, 128, 0)",
                "First element should be green");
        });
    </script>
</body>
</html>
```

**Expected Output (QUnit test results):**
```
Tests completed: 3
Assertions: 4
Passed: 4
Failed: 0
```

**Why This Code Produces That Result:**
Each test appends a new element to `#qunit-fixture`, which QUnit resets between tests. Test 1 verifies the default color. Test 2 verifies a custom color. Test 3 verifies that the plugin works on multiple elements and that both elements receive the correct color. All assertions pass because the plugin behaves as expected.

---

## 15. Plugin Documentation

### Definitions

**Core Definition:**
Plugin documentation is the written explanation of a plugin's purpose, installation, usage, options, methods, events, and examples, intended to help users understand and use the plugin.

**Technical Definition:**
Plugin documentation typically includes a README file with installation instructions, a usage section, an options table, an API reference for methods and events, and code examples. JSDoc is a common standard for inline code documentation, and tools like docco and dox can generate documentation from JSDoc comments.

**Beginner-Friendly Explanation:**
Documentation is the instruction manual for your plugin. It tells users what your plugin does, how to install it, how to use it, what options are available, and what methods they can call. Good documentation makes the difference between a plugin that people can use and one that sits unused.

### Purposes

- To explain what the plugin does and why it is useful.
- To provide installation and setup instructions.
- To document all options, methods, events, and callbacks.
- To show usage examples for common scenarios.
- To reduce support requests by answering common questions.

### Syntax Rules and Structure

**Complete General Syntax (JSDoc Comment):**

```javascript
/**
 * Initializes the plugin on the selected elements.
 * @param {Object} [options] - Configuration options.
 * @param {string} [options.color="red"] - The color to apply.
 * @param {number} [options.speed=300] - Animation speed in milliseconds.
 * @returns {jQuery} The jQuery object for chaining.
 * @example
 * $(".element").myPlugin({ color: "blue" });
 */
$.fn.myPlugin = function(options) { ... };
```

**Component Breakdown:**

| Documentation Element | Description |
|----------------------|-------------|
| Plugin name | The name of the plugin. |
| Description | What the plugin does. |
| Installation | How to install (npm, CDN, download). |
| Usage | Basic usage example. |
| Options | Table of options with types and defaults. |
| Methods | List of public methods. |
| Events | Custom events triggered by the plugin. |
| Examples | Code examples for common scenarios. |

**Syntax Rules:**

1. Use JSDoc comments for inline documentation of functions and options.
2. Write a README.md file with installation, usage, and API reference.
3. Include code examples that users can copy and paste.
4. Document all options with their types and default values.
5. Keep documentation up to date with the plugin's behavior.

**Constraints and Limitations:**

- Documentation requires ongoing maintenance as the plugin evolves.
- Overly verbose documentation can be overwhelming; focus on clarity and conciseness.
- Inline comments should explain why, not what (the code shows what).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Documented Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Plugin Documentation</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="demo">Demo</div>

    <script>
        ;(function($) {
            /**
             * Applies a highlight effect to elements.
             *
             * @param {Object} [options] - Configuration options.
             * @param {string} [options.color="yellow"] - The highlight color.
             * @param {string} [options.padding="5px"] - The padding to apply.
             * @returns {jQuery} The jQuery object for chaining.
             *
             * @example
             * // Apply with defaults
             * $(".element").highlight();
             *
             * @example
             * // Apply with custom color
             * $(".element").highlight({ color: "lightblue" });
             */
            $.fn.highlight = function(options) {
                var settings = $.extend({
                    color: "yellow",
                    padding: "5px"
                }, options);

                return this.each(function() {
                    $(this).css({
                        backgroundColor: settings.color,
                        padding: settings.padding
                    });
                });
            };

            // Document defaults
            /**
             * Default options for the highlight plugin.
             * @type {Object}
             * @property {string} color="yellow" - Default highlight color.
             * @property {string} padding="5px" - Default padding.
             */
            $.fn.highlight.defaults = {
                color: "yellow",
                padding: "5px"
            };
        }(jQuery));

        // Usage
        $(".demo").highlight({ color: "lightgreen" });
        console.log("Bg: " + $(".demo").css("background-color"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Bg: rgb(144, 238, 144)
```

**Why This Code Produces That Result:**
The JSDoc comments document the plugin's purpose, parameters, return value, and usage examples. The `@param` tags specify the options object and its properties with types and defaults. The `@returns` tag documents the return value. The `@example` tags show how to use the plugin. This documentation can be processed by JSDoc tools to generate HTML documentation.

---

## 16. Plugin Publishing

### Definitions

**Core Definition:**
Plugin publishing is the process of packaging, distributing, and making a plugin available for others to install and use, typically via npm or a plugin registry.

**Technical Definition:**
Publishing a jQuery plugin to npm involves creating a `package.json` file with metadata (name, version, description, main file, keywords, author, license, repository, dependencies), logging into npm, and running `npm publish`. The jQuery Plugin Registry has been in read-only mode since 2015 and now recommends publishing to npm.

**Beginner-Friendly Explanation:**
Publishing your plugin is like putting it on a shelf in a store where other developers can find and download it. npm is the most common store for JavaScript packages. Once published, anyone can install your plugin with a simple command like `npm install your-plugin-name`.

### Purposes

- To share the plugin with the developer community.
- To make the plugin easily installable via package managers.
- To establish a versioned release for users to depend on.
- To provide a central location for documentation and issue tracking.

### Syntax Rules and Structure

**Complete General Syntax (package.json):**

```json
{
  "name": "your-plugin-name",
  "version": "1.0.0",
  "description": "Description of your jQuery plugin",
  "main": "path-to-main-file.js",
  "keywords": ["jquery", "jquery-plugin", "ecosystem:jquery"],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/your-plugin-name.git"
  },
  "peerDependencies": {
    "jquery": ">=3.5"
  }
}
```

**Complete General Syntax (Publishing Commands):**

```bash
npm login
npm publish
```

**Component Breakdown:**

| Field | Description |
|-------|-------------|
| `name` | The package name (unique on npm). |
| `version` | Semantic version (e.g., `1.0.0`). |
| `main` | The entry point file. |
| `keywords` | Search keywords; include `"jquery"` and `"jquery-plugin"`. |
| `peerDependencies` | jQuery version requirement. |

**Syntax Rules:**

1. Create a `package.json` file in the root directory.
2. Include `"jquery"` and `"jquery-plugin"` in the keywords array.
3. Use semantic versioning (SemVer) for version numbers.
4. Log in to npm and run `npm publish` to publish.
5. Include a `README.md` with documentation.

**Constraints and Limitations:**

- Package names must be unique on npm.
- Once published, a version cannot be unpublished (though it can be deprecated).
- The jQuery Plugin Registry is read-only; publish to npm instead.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Complete Publishing Workflow**

```bash
# Step 1: Create the plugin file
# File: jquery.myplugin.js
;(function($) {
    $.fn.myPlugin = function(options) {
        var settings = $.extend({
            color: "red"
        }, options);
        return this.each(function() {
            $(this).css("color", settings.color);
        });
    };
}(jQuery));

# Step 2: Create package.json
# File: package.json
{
  "name": "jquery-myplugin",
  "version": "1.0.0",
  "description": "A simple jQuery plugin that changes text color.",
  "main": "jquery.myplugin.js",
  "keywords": ["jquery", "jquery-plugin", "ecosystem:jquery", "color"],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/jquery-myplugin.git"
  },
  "peerDependencies": {
    "jquery": ">=3.5"
  }
}

# Step 3: Create README.md
# File: README.md
# jQuery MyPlugin
A simple jQuery plugin that changes text color.
## Installation
npm install jquery-myplugin
## Usage
$(".element").myPlugin({ color: "blue" });

# Step 4: Log in to npm
npm login

# Step 5: Publish
npm publish
```

**Expected Output:**
```
+ jquery-myplugin@1.0.0
```

**Why This Code Produces That Result:**
The `package.json` file contains all required metadata. The `keywords` array includes `"jquery"` and `"jquery-plugin"` for discoverability on npm. The `peerDependencies` field specifies that jQuery 3.5 or higher is required. After logging in, `npm publish` packages the plugin and uploads it to the npm registry. The output shows the package name and version that was published.

---

## 17. Plugin Best Practices

### Definitions

**Core Definition:**
Plugin best practices are empirically established guidelines for writing jQuery plugins that are performant, maintainable, conflict-free, and easy to use.

**Technical Definition:**
Best practices encompass IIFE encapsulation, unique naming and namespacing, chainability preservation, option merging with `$.extend()`, defaults exposure, event namespacing for cleanup, proper use of `this.each()`, avoiding global scope pollution, documenting the API, and using the Widget Factory for stateful plugins.

**Beginner-Friendly Explanation:**
These are the proven rules that experienced jQuery developers follow. They help you write plugins that do not break other code, are easy for others to use, and continue to work as your application grows.

### Purposes

- To ensure plugins are compatible with other libraries and plugins.
- To provide a consistent, predictable API for plugin users.
- To prevent memory leaks and performance degradation.
- To facilitate debugging, testing, and maintenance.
- To establish a professional standard for plugin distribution.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Wrap in IIFE | `$.fn.myPlugin = function() {}` (global scope) | `;(function($) { ... }(jQuery))` |
| 2 | Use unique naming | `$.fn.tooltip = function() {}` | `$.fn.myCompanyTooltip = function() {}` |
| 3 | Return `this` for chaining | `return $(this);` | `return this.each(...)` |
| 4 | Merge options with empty `{}` | `$.extend(defaults, options)` | `$.extend({}, defaults, options)` |
| 5 | Expose defaults | `var defaults = { ... }` (private) | `$.fn.myPlugin.defaults = { ... }` |
| 6 | Use `this.each()` | `this.css("color", "red")` | `return this.each(function() { ... })` |
| 7 | Namespace events | `.on("click", fn)` | `.on("click.myPlugin", fn)` |
| 8 | Clean up on destroy | No cleanup | `.off(".myPlugin")`, remove data |
| 9 | Document the API | No documentation | JSDoc comments, README |
| 10 | Use Widget Factory for stateful plugins | Manual state management | `$.widget("ns.widget", { ... })` |

**Syntax Rules:**

1. Always wrap plugins in an IIFE and pass `jQuery` as `$`.
2. Return `this` from chainable methods; use `return this.each(...)` for iteration.
3. Merge options with `$.extend({}, defaults, options)` to avoid mutating defaults.
4. Namespace all events (e.g., `"click.myPlugin"`) for clean unbinding.
5. Document all options, methods, events, and callbacks.
6. Use the Widget Factory for plugins that need state management.

**Constraints and Limitations:**

- Best practices add initial development overhead but reduce long-term maintenance costs.
- Over-namespacing can make event names verbose.
- Not all plugins need the full Widget Factory; simple plugins should stay simple.
- Documentation is often neglected but is critical for adoption.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Complete Best-Practice Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Best Practices Plugin</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="widget">Widget 1</div>
    <div class="widget">Widget 2</div>

    <script>
        // ✅ IIFE wrapper
        ;(function($) {
            // ✅ Unique, descriptive name
            $.fn.myCompanyWidget = function(options) {
                // ✅ Merge with empty {} to protect defaults
                var settings = $.extend({}, $.fn.myCompanyWidget.defaults, options);

                // ✅ Use this.each() for multiple elements
                return this.each(function() {
                    var $this = $(this);
                    $this.css({
                        backgroundColor: settings.bgColor,
                        padding: settings.padding
                    });

                    // ✅ Namespace events
                    $this.on("click.myCompanyWidget", function() {
                        $this.toggleClass("active");
                    });

                    $this.data("myCompanyWidget", settings);
                });
            };

            // ✅ Expose defaults
            $.fn.myCompanyWidget.defaults = {
                bgColor: "#f0f0f0",
                padding: "10px"
            };

            // ✅ Provide a destroy method
            $.fn.myCompanyWidget.destroy = function() {
                return this.each(function() {
                    var $this = $(this);
                    $this.off(".myCompanyWidget");
                    $this.removeData("myCompanyWidget");
                    $this.removeClass("active");
                });
            };
        }(jQuery)); // ✅ Pass jQuery

        // Usage
        $(".widget").myCompanyWidget({ bgColor: "#e0e0e0" });
        console.log("Widgets: " + $(".widget").length); // 2
        console.log("Bg: " + $(".widget").first().css("background-color"));

        // Cleanup
        $(".widget").myCompanyWidget.destroy();
        console.log("Active after destroy: " + $(".widget.active").length); // 0
    </script>
</body>
</html>
```

**Expected Output:**
```
Widgets: 2
Bg: rgb(224, 224, 224)
Active after destroy: 0
```

**Why This Code Produces That Result:**
Every best practice is applied: the IIFE protects `$`, the plugin name is unique, options are merged with an empty object, `this.each()` handles multiple elements, events are namespaced (`.myCompanyWidget`), defaults are exposed, a destroy method cleans up events and data, and jQuery is passed to the IIFE. The destroy method removes the namespaced click handler, the stored data, and any active classes, demonstrating proper cleanup.

---

## References

- jQuery Learning Center — How to Create a Basic Plugin - https://learn.jquery.com/plugins/basic-plugin-creation/
- jQuery Learning Center — Advanced Plugin Concepts - https://learn.jquery.com/plugins/advanced-plugin-concepts/
- jQuery Learning Center — Publishing jQuery Plugins to npm - https://learn.jquery.com/plugins/publishing-plugins/
- jQuery Learning Center — Plugins - https://learn.jquery.com/plugins/
- jQuery API Documentation — jQuery.extend() - https://api.jquery.com/jQuery.extend/
- jQuery API Documentation — jQuery.fn - https://api.jquery.com/jQuery.fn/
- jQuery UI API Documentation — Widget Factory - https://api.jqueryui.com/jQuery.widget/
- jQuery UI API Documentation — Widget Plugin Bridge - https://api.jqueryui.com/jQuery.widget.bridge/
- Addy Osmani — jQuery Plugin Patterns - https://github.com/addyosmani/jquery-plugin-patterns
- jQuery Boilerplate — GitHub Repository - https://github.com/jquery-boilerplate/jquery-boilerplate
- QUnit — Official Documentation - https://qunitjs.com/
- npm — Publishing Packages - https://docs.npmjs.com/cli/commands/npm-publish
- npm — Semantic Versioning - https://docs.npmjs.com/about-semantic-versioning
- JSDoc — Official Documentation - https://jsdoc.app/
- Ben Alman — Immediately Invoked Function Expression - http://benalman.com/news/2010/11/immediately-invoked-function-expression/
- SitePoint — A Basic jQuery Plugin using the Module Pattern - https://www.sitepoint.com/basic-jquery-plugin-module-pattern/
- Stack Overflow — How to create a jQuery plugin with methods? - https://stackoverflow.com/questions/1117086/how-to-create-a-jquery-plugin-with-methods
- Learning jQuery, Fourth Edition — Chapter 8: Developing Plugins - https://archive.org/stream/9781782163145LEARNINGJQUERYFOURTHEDITION