# Creating jQuery Plugins — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Creating a jQuery plugin is the practice of extending the jQuery library by adding new methods to `jQuery.prototype` (aliased as `$.fn`), thereby making custom, reusable functionality available on all jQuery objects returned by the `$()` factory function.

**Technical Definition:** A jQuery plugin is a function assigned to a property of `$.fn`. Because `$.fn` is a direct alias for `jQuery.prototype`, any method added to it becomes available on every jQuery object through prototypal inheritance. The plugin function receives the jQuery collection as its `this` context, iterates over the matched elements using `this.each()`, merges user options with a defaults object using `$.extend()`, and returns `this` to maintain chainability. Plugins are typically wrapped in an Immediately Invoked Function Expression (IIFE) to create a private scope and safely alias the `$` variable, ensuring compatibility with `jQuery.noConflict()`. Stateful plugins may store instance data on individual DOM elements using `$.data()`, and event handlers are scoped using namespaced events like `click.pluginName`.

**Beginner-Friendly Explanation:** A jQuery plugin is a custom tool you add to jQuery's toolbox. jQuery comes with many built-in tools (like `.css()` and `.hide()`), but if you need a new one — say, a method that makes text green or turns a `<div>` into a date picker — you can create a plugin. Once created, any jQuery selection can use your new method just like a built-in one. The plugin architecture provides rules and patterns that keep these custom tools organized, reusable, and compatible with other plugins.

### Key Characteristics

- **Prototype extension:** Plugins extend `$.fn` (which is `jQuery.prototype`), so they become available on all jQuery objects.
- **Chainability:** Plugins return `this` to allow chaining with other jQuery methods.
- **Implicit iteration:** Plugins use `this.each()` to operate on every element in the selection, ensuring compatibility with multi-element selections.
- **IIFE encapsulation:** Plugins are wrapped in an IIFE to create a private scope and safely alias `$`.
- **Deep options merging:** Plugins typically use `$.extend(true, {}, defaults, options)` to deep-merge user options with defaults, preserving nested objects.
- **Public defaults:** Defaults are exposed via `$.fn.pluginName.defaults` for global configuration overrides.
- **Instance isolation:** State is stored per element using `$(this).data('pluginName', instance)` to prevent crossover between instances.
- **Event namespacing:** Event handlers are scoped using namespaces like `click.pluginName` to allow clean unbinding without affecting other handlers.

### Prerequisites

- Basic understanding of JavaScript, particularly functions, objects, prototypes, and closures.
- Familiarity with jQuery fundamentals: selectors, the `$()` factory, and jQuery object methods.
- Knowledge of the `this` keyword and how it behaves inside jQuery methods.
- Basic understanding of the CSS box model and DOM manipulation.

### Related Programming Areas

- **DOM Manipulation:** Plugins encapsulate complex DOM operations behind a simple API.
- **Event Handling:** Plugins often bind, delegate, and unbind event listeners as part of their behavior.
- **State Management:** Stateful plugins maintain internal state and lifecycle (initialization, destruction).
- **UI Widget Development:** The jQuery UI Widget Factory is a formalization of the plugin architecture for stateful widgets.
- **Modular JavaScript:** IIFE wrapping and namespace management align with broader modular JavaScript patterns.

### Core Concepts / Features

This cheat sheet covers eight core concepts: `$.fn` extending mechanics, custom methods, chainability, options objects, default configuration, instance data, event management, `this.each()` iteration, and IIFE wrapping.

---

## Core Concept 1: `$.fn` Extending Mechanics — Assigning Functions Safely Without Rewriting the Object

### Definitions

**Core Definition:** `$.fn` extending is the process of adding new methods to the jQuery prototype by assigning functions to properties of `$.fn`. This makes the methods available on all jQuery objects without modifying the jQuery core.

**Technical Definition:** `$.fn` is a direct alias for `jQuery.prototype`. When `$()` is called, it creates a new instance of the jQuery constructor, and this instance inherits from `jQuery.prototype` (i.e., `$.fn`). Therefore, any method added to `$.fn` becomes a property of every jQuery object. To add a method, you simply assign a function to a new property: `$.fn.myPlugin = function() { ... }`. Inside the plugin, `this` refers to the jQuery collection on which the plugin was invoked.

**Beginner-Friendly Explanation:** Think of jQuery as a factory that produces “jQuery objects” — each one is a wrapper around a set of DOM elements. Every jQuery object shares a common set of methods (like `.css()`, `.hide()`, etc.) because they all inherit from the same prototype. A plugin is just a new method you add to that shared prototype. Once you add it, every jQuery object — present and future — can use it.

### Purposes

- To extend jQuery's built-in functionality with custom, reusable methods.
- To make new functionality available on all jQuery objects without modifying the jQuery core library.
- To provide a consistent, chainable API for custom DOM operations.
- To encapsulate complex DOM manipulation logic behind a single method call.
- To create a namespace for related functionality that can be distributed and reused across projects.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
$.fn.pluginName = function( [options] ) {
    // Plugin logic
    return this;
};
```

| Component | Description |
|-----------|-------------|
| `$.fn` | Alias for `jQuery.prototype`; the shared prototype of all jQuery objects. |
| `.pluginName` | The name of the plugin method; must be a valid JavaScript identifier. |
| `function( [options] )` | The plugin implementation; `this` refers to the jQuery collection. |
| `return this` | Returns the jQuery object for chaining. |

**Syntax Rules:**

- The method must be assigned to `$.fn` (or `jQuery.fn`) to become available on jQuery objects.
- Inside the plugin, `this` is the jQuery collection, so you can call other jQuery methods directly (e.g., `this.css(...)`).
- The plugin should return `this` to maintain chainability.
- Only one property should be added to `$.fn` per plugin to avoid namespace pollution.

**Constraints and Limitations:**

- Adding methods to `$.fn` affects **all** jQuery objects globally; plugin names should be chosen carefully to avoid collisions with jQuery core methods or other plugins.
- Plugins that do not return `this` break chainability.
- The `$` alias may be unavailable if `jQuery.noConflict()` is used; plugins should be wrapped in an IIFE that receives `jQuery` and aliases it as `$`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: A Minimal Plugin**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Minimal jQuery plugin demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <a href="#">Link 1</a>
  <a href="#">Link 2</a>
  <a href="#">Link 3</a>
  <p id="output"></p>

  <script>
    // Step 1: Define the plugin by adding a method to $.fn
    $.fn.greenify = function() {
      // Step 2: `this` is the jQuery collection; iterate over each element
      this.each(function() {
        // Step 3: `this` inside .each() is the raw DOM element
        $(this).css("color", "green");
      });
      // Step 4: Return `this` to allow chaining
      return this;
    };

    // Step 5: Use the plugin on a jQuery selection
    $("a").greenify().addClass("greenified");
    $("#output").text("Plugin applied: " + $("a.greenified").length + " links greenified.");
  </script>
</body>
</html>
```

**Expected Output:**
```
Plugin applied: 3 links greenified.
```

**Why this output:** The `$.fn.greenify` plugin is added to the jQuery prototype. When called on `$("a")`, it iterates over all three anchor elements, applying `color: green` to each. Returning `this` allows `.addClass("greenified")` to chain. The final line counts the number of links with the `greenified` class, confirming that all three were processed.

### Real-World Cases

- **Utility plugins:** A `$.fn.center()` plugin centers elements within their parent.
- **Effect plugins:** A `$.fn.fadeToggle()` plugin fades elements in or out based on their current visibility.
- **Form plugins:** A `$.fn.validate()` plugin adds client-side validation to form fields.

---

## Core Concept 2: Custom Methods — Accepting Settings Objects or Action Strings Like `'destroy'`

### Definitions

**Core Definition:** Custom methods are the public API methods that a plugin exposes, allowing consumers to initialize, configure, query, and destroy the plugin instance. These methods typically accept either an options object (for initialization or reconfiguration) or an action string (for invoking specific behaviors like `"destroy"` or `"refresh"`).

**Technical Definition:** A well-designed jQuery plugin method inspects the type of its first argument to determine the mode of operation. If the argument is an object or undefined, the plugin performs initialization (or re-initialization with new options). If the argument is a string, the plugin treats it as a method name, retrieves the corresponding instance from the element's data storage, and invokes the specified method on that instance, passing any additional arguments. This pattern, known as the "method invocation" pattern, allows a single plugin entry point to handle both initialization and subsequent method calls.

**Beginner-Friendly Explanation:** A plugin is like a machine with multiple buttons. You press one button to turn it on (initialization with options), and other buttons to make it do things (method calls like "destroy" or "refresh"). The plugin figures out which button you pressed by looking at what you passed to it. If you pass an object, it thinks you want to set it up; if you pass a string, it thinks you want to call a method.

### Purposes

- To provide a single, unified entry point for all plugin interactions.
- To allow consumers to invoke specific behaviors (e.g., `destroy`, `refresh`, `option`) after initialization.
- To enable re-initialization or reconfiguration of a plugin by passing a new options object.
- To support method chaining by returning the jQuery object from method calls.
- To standardize the API across different plugins, making them easier to learn and use.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
$.fn.pluginName = function( options ) {
    // Method invocation mode
    if (typeof options === "string") {
        var args = Array.prototype.slice.call(arguments, 1);
        return this.each(function() {
            var instance = $.data(this, "pluginName");
            if (instance && typeof instance[options] === "function") {
                instance[options].apply(instance, args);
            }
        });
    }

    // Initialization mode
    return this.each(function() {
        if (!$.data(this, "pluginName")) {
            $.data(this, "pluginName", new Plugin(this, options));
        }
    });
};
```

| Component | Description |
|-----------|-------------|
| `options` | Either an options object (initialization) or a string (method name). |
| `typeof options === "string"` | Checks if the first argument is a method name. |
| `Array.prototype.slice.call(arguments, 1)` | Collects additional arguments to pass to the method. |
| `$.data(this, "pluginName")` | Retrieves the stored plugin instance from the element. |
| `instance[options].apply(instance, args)` | Invokes the method on the instance with the given arguments. |

**Syntax Rules:**

- The plugin should check if the first argument is a string to determine method invocation mode.
- Method names should correspond to functions defined on the plugin instance.
- Initialization should only occur if the plugin has not already been instantiated on the element (checked via `$.data()`).
- The plugin should return `this` in both modes to maintain chainability.

**Constraints and Limitations:**

- The method invocation pattern requires careful argument handling; extra arguments must be collected and passed to the method.
- The plugin must check whether the instance exists before invoking a method; otherwise, a `TypeError` may occur.
- Method names should be documented clearly so consumers know which strings to pass.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: A Plugin with Initialization and Method Invocation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Method invocation demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="box" id="box1">Box 1</div>
  <div class="box" id="box2">Box 2</div>
  <button id="destroyBtn">Destroy All</button>
  <p id="log"></p>

  <script>
    (function($) {
      // Step 1: Define the plugin
      $.fn.toggleBox = function(action) {
        // Step 2: Handle method invocation
        if (typeof action === "string") {
          return this.each(function() {
            var instance = $.data(this, "toggleBox");
            if (instance && typeof instance[action] === "function") {
              instance[action]();
            }
          });
        }

        // Step 3: Initialization mode
        return this.each(function() {
          if (!$.data(this, "toggleBox")) {
            $.data(this, "toggleBox", new ToggleBox(this));
          }
        });
      };

      // Step 4: Define the constructor
      function ToggleBox(element) {
        this.$el = $(element);
        this.visible = true;
        this._init();
      }

      // Step 5: Define prototype methods
      ToggleBox.prototype = {
        _init: function() {
          var self = this;
          this.$el.on("click.toggleBox", function() {
            self.toggle();
          });
        },
        toggle: function() {
          this.visible = !this.visible;
          this.$el.css("display", this.visible ? "block" : "none");
          $("#log").append(this.$el.attr("id") + " toggled to " + (this.visible ? "visible" : "hidden") + "<br>");
        },
        destroy: function() {
          this.$el.off(".toggleBox").removeData("toggleBox");
          $("#log").append(this.$el.attr("id") + " destroyed.<br>");
        }
      };
    }(jQuery));

    // Step 6: Initialize the plugin
    $(".box").toggleBox();

    // Step 7: Invoke the destroy method
    $("#destroyBtn").click(function() {
      $(".box").toggleBox("destroy");
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking a box toggles its visibility and logs the change. Clicking "Destroy All" destroys both plugin instances, unbinds their event handlers, and removes their data.

**Why this output:** The plugin inspects its first argument. When called with no arguments (or an object), it initializes a `ToggleBox` instance on each element. When called with the string `"destroy"`, it retrieves the instance from `$.data()` and invokes its `destroy` method, which cleans up event handlers and data.

### Real-World Cases

- **jQuery UI Widgets:** All jQuery UI widgets support method invocation via strings, such as `$("#datepicker").datepicker("destroy")`.
- **DataTables:** Supports methods like `$("#table").DataTable().ajax.reload()` and `$("#table").DataTable().destroy()`.
- **Custom form validators:** A validator plugin might support methods like `"validate"`, `"reset"`, and `"destroy"`.

---

## Core Concept 3: Chainability — Always Returning `this` to Preserve the Wrapped Set Pipeline

### Definitions

**Core Definition:** Chainability is the jQuery design pattern where every method returns the jQuery object it was called on, allowing multiple methods to be chained together in a single statement.

**Technical Definition:** When a jQuery method returns `this`, it returns the same jQuery collection that the method was called on, enabling subsequent methods to be called on the same collection. This is achieved by ending the plugin function with `return this;` or `return this.each(...);`. The `.each()` method itself returns the jQuery collection, so returning its result maintains chainability while also performing implicit iteration. Chainability is a core jQuery feature that allows concise, readable code like `$("a").greenify().addClass("greenified").fadeIn()`.

**Beginner-Friendly Explanation:** Chainability is like passing a baton in a relay race. Each method does its job and then passes the baton (the jQuery object) to the next method. If a method forgets to pass the baton (by not returning `this`), the chain breaks and you cannot call any more methods. Always returning `this` keeps the relay going.

### Purposes

- To allow multiple operations to be performed on the same jQuery collection in a single statement.
- To make plugin code more concise and readable.
- To maintain consistency with jQuery's built-in methods, which are all chainable (with a few exceptions like `.width()` as a getter).
- To enable plugin composition, where the output of one plugin feeds into another.
- To support fluent interfaces that make complex operations easy to express.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
$.fn.pluginName = function( options ) {
    // Plugin logic
    return this.each(function() {
        // Per-element logic
    });
};
```

| Component | Description |
|-----------|-------------|
| `return this` | Returns the jQuery collection for chaining. |
| `return this.each(...)` | Returns the collection after implicit iteration; preferred pattern. |

**Syntax Rules:**

- The plugin must return `this` (or the result of `this.each(...)`) as its last statement.
- If the plugin returns a value (e.g., a computed property), it is **not** chainable for that call.
- `this.each()` returns the original jQuery collection, so returning its result maintains chainability.
- Methods that return values (like getters) intentionally break chainability; this is a documented exception.

**Constraints and Limitations:**

- Forgetting to return `this` is one of the most common jQuery plugin mistakes; it silently breaks chainability.
- If a plugin returns `this.each()` but the callback inside `.each()` returns a value, that value is ignored; `.each()` always returns the collection.
- Some plugins intentionally break chainability when they need to return a computed value (e.g., a plugin that reads an element's state).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Chainable Plugin**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Chainability demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p class="message">Hello</p>
  <p class="message">World</p>

  <script>
    // Step 1: Define a chainable plugin
    $.fn.highlight = function(color) {
      return this.each(function() {
        $(this).css("background-color", color || "yellow");
      });
    };

    // Step 2: Chain the plugin with other jQuery methods
    $(".message")
      .highlight("lightblue")
      .css("color", "navy")
      .fadeIn(300);
  </script>
</body>
</html>
```

**Expected Output:** Both paragraphs have a light blue background, navy text, and fade in over 300ms.

**Why this output:** The `.highlight()` plugin returns `this.each(...)`, which returns the jQuery collection. This allows `.css()` and `.fadeIn()` to be chained on the same collection. If the plugin had not returned `this`, the chain would break after `.highlight()`.

### Real-World Cases

- **jQuery UI:** Widget methods like `.progressbar("value", 50)` return the jQuery object, enabling further chaining.
- **Slick Carousel:** Methods like `.slick("slickNext")` return the carousel instance for chaining.
- **Form plugins:** A validation plugin might chain `.validate().highlightErrors().focusFirstError()`.

---

## Core Concept 4: Options Objects — Deep Extending Configurations Using `$.extend(true, {}, defaults, options)`

### Definitions

**Core Definition:** An options object is a plain JavaScript object passed to a plugin to customize its behavior. Deep extending is the process of recursively merging the options object with a defaults object, preserving nested objects and arrays.

**Technical Definition:** `$.extend(true, {}, defaults, options)` performs a deep (recursive) merge of the `defaults` object and the `options` object into a new empty object (`{}`). The `true` flag enables recursive merging, meaning nested objects and arrays are merged property-by-property rather than being overwritten wholesale. The empty object `{}` serves as the target, ensuring that neither `defaults` nor `options` is modified. The result is a new settings object that contains all default properties, overridden by any matching properties from the user-provided options.

**Beginner-Friendly Explanation:** Imagine you have a recipe (the defaults) and you want to make a few changes (the options). Instead of rewriting the entire recipe, you just tell the plugin which ingredients to change. Deep extending means that if a default ingredient is a list (like an array of spices), your changes are added to the list rather than replacing the whole list. The `{}` ensures the original recipe is not modified.

### Purposes

- To provide sensible default configuration for a plugin while allowing consumers to override specific options.
- To deep-merge nested option objects, preserving default properties that the user did not specify.
- To avoid modifying the original defaults or options objects.
- To create a clean settings object that can be safely stored and used throughout the plugin's lifetime.
- To enable global configuration overrides via `$.fn.pluginName.defaults` while still allowing per-instance customization.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
var settings = $.extend(true, {}, $.fn.pluginName.defaults, options);
```

| Component | Description |
|-----------|-------------|
| `true` | Enables deep (recursive) merging. |
| `{}` | The target object; a new empty object that receives the merged properties. |
| `$.fn.pluginName.defaults` | The plugin's default options object. |
| `options` | The user-provided options object. |

**Syntax Rules:**

- The `true` flag must be the first argument to enable deep merging.
- The second argument should be a new empty object `{}` to avoid modifying the defaults or options.
- The defaults object should be exposed as a public property of the plugin (e.g., `$.fn.pluginName.defaults`).
- Properties from `options` override matching properties in `defaults`; unmatched default properties are preserved.

**Constraints and Limitations:**

- Deep extending a cyclical data structure will result in an error.
- Object wrappers on primitive types (String, Boolean, Number) are not deep-extended.
- Passing `false` as the first argument to `$.extend()` is not supported.
- Versions prior to 3.4 had a security issue where `__proto__` properties could extend `Object.prototype`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Deep Extending Nested Options**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Deep extend demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p id="output"></p>

  <script>
    // Step 1: Define defaults with nested objects
    var defaults = {
      animation: {
        duration: 400,
        easing: "swing"
      },
      color: "blue"
    };

    // Step 2: User options that override only some nested properties
    var options = {
      animation: {
        duration: 1000
      },
      color: "red"
    };

    // Step 3: Deep extend
    var settings = $.extend(true, {}, defaults, options);

    // Step 4: Verify the result
    $("#output").html(
      "animation.duration: " + settings.animation.duration + "<br>" +
      "animation.easing: " + settings.animation.easing + "<br>" +
      "color: " + settings.color
    );
  </script>
</body>
</html>
```

**Expected Output:**
```
animation.duration: 1000
animation.easing: swing
color: red
```

**Why this output:** The deep extend recursively merges the `animation` object. The user's `duration: 1000` overrides the default `duration: 400`, but the default `easing: "swing"` is preserved because the user did not specify an easing. The top-level `color` property is overridden from `"blue"` to `"red"`. Without the `true` flag, the entire `animation` object would be replaced, losing the default `easing`.

### Real-World Cases

- **jQuery UI:** Widgets define `options` objects and use deep extending to merge user options with defaults.
- **Charting libraries:** A chart plugin might have nested options for axes, legends, and tooltips; deep extending preserves unspecified nested properties.
- **Form plugins:** A validation plugin might have nested rules for different field types; deep extending allows per-field overrides.

---

## Core Concept 5: Default Configuration — Exposing Public Defaults via `$.fn.pluginName.defaults`

### Definitions

**Core Definition:** Public defaults are the default configuration values for a plugin, exposed as a property on the plugin function itself (e.g., `$.fn.pluginName.defaults`), allowing consumers to view and globally override the defaults before initializing any instances.

**Technical Definition:** By assigning the defaults object to `$.fn.pluginName.defaults`, the plugin makes its default configuration publicly accessible. Consumers can modify these defaults globally (e.g., `$.fn.pluginName.defaults.option = value`) to affect all future instances, or they can pass per-instance options to override the defaults for a single instance. The merge order is: public defaults, then global overrides (if any), then instance-specific options — with instance-specific options taking the highest precedence.

**Beginner-Friendly Explanation:** Public defaults are like the factory settings on a device. They are the values the plugin uses if you do not specify anything. By exposing them, the plugin lets you change the factory settings for all devices at once, or you can customize a single device when you set it up. This is useful for applying consistent settings across an application.

### Purposes

- To provide documentation of the plugin's default configuration.
- To allow global configuration overrides without modifying the plugin source code.
- To enable a consistent baseline configuration across all instances of a plugin.
- To simplify per-instance configuration by reducing the number of options that must be specified.
- To support application-wide theming or behavior customization.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
// Expose defaults
$.fn.pluginName.defaults = {
    option1: defaultValue1,
    option2: defaultValue2
};

// Global override
$.fn.pluginName.defaults.option1 = newValue;

// Instance-specific override
$(selector).pluginName({ option1: instanceValue });
```

**Syntax Rules:**

- The defaults object should be exposed immediately after defining the plugin.
- Global overrides must be set **before** any instances are initialized.
- Instance-specific options take precedence over global overrides.
- The defaults object should not be modified by the plugin at runtime; use `$.extend()` to create a merged settings object.

**Constraints and Limitations:**

- Global overrides affect only instances initialized **after** the override is set.
- Deep merging is recommended when defaults contain nested objects; use `$.extend(true, {}, defaults, options)`.
- Not all plugins expose their defaults publicly; some use a private defaults object.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Global and Instance-Specific Configuration**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Public defaults demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <p class="message" id="msg1">Message 1</p>
  <p class="message" id="msg2">Message 2</p>
  <p id="log"></p>

  <script>
    // Step 1: Define the plugin with exposed defaults
    $.fn.highlight = function(options) {
      var settings = $.extend({}, $.fn.highlight.defaults, options);
      return this.each(function() {
        $(this).css({
          backgroundColor: settings.bgColor,
          color: settings.textColor
        });
      });
    };

    // Step 2: Expose defaults
    $.fn.highlight.defaults = {
      bgColor: "yellow",
      textColor: "black"
    };

    $(function() {
      // Step 3: Global override — affects all future instances
      $.fn.highlight.defaults.bgColor = "lightgreen";

      // Step 4: Instance-specific override
      $("#msg2").highlight({ bgColor: "lightcoral" });

      // Step 5: Verify
      $("#log").text("Global default bgColor: " + $.fn.highlight.defaults.bgColor);
    });
  </script>
</body>
</html>
```

**Expected Output:** `#msg1` has a light green background; `#msg2` has a light coral background. The log shows “Global default bgColor: lightgreen”.

**Why this output:** The global override changes the default `bgColor` from `"yellow"` to `"lightgreen"`. When `#msg1` is initialized (without options), it uses the global default. When `#msg2` is initialized with `{ bgColor: "lightcoral" }`, the instance-specific option overrides the global default. The log confirms the global default value.

### Real-World Cases

- **DataTables:** Global defaults can be set via `$.fn.dataTable.defaults` to apply consistent pagination and language settings across all tables.
- **Tooltipster:** Global defaults can be set via `$.tooltipster.setDefaults()` to apply consistent styling and behavior to all tooltips.
- **jQuery Validation:** Global defaults can be set via `$.validator.setDefaults()` to apply consistent validation rules and messages.

---

## Core Concept 6: Instance Data — Isolating State on Individual DOM Elements Using `$(this).data('pluginName', ...)`

### Definitions

**Core Definition:** Instance data is the practice of storing a plugin instance on its associated DOM element using jQuery's `.data()` method, keyed by the plugin name. This isolates state per element and prevents crossover between multiple plugin instances.

**Technical Definition:** When a plugin is initialized on an element, a new instance object (often created via a constructor function) is stored on the DOM element using `$(element).data("pluginName", instance)` or `$.data(element, "pluginName", instance)`. This allows the plugin to retrieve the instance later — for example, when a method is invoked via a string argument — using `$(element).data("pluginName")`. The data is stored on the element itself, so different elements have independent instances with independent state. When the plugin is destroyed, the data should be removed using `.removeData()` to prevent memory leaks.

**Beginner-Friendly Explanation:** Imagine you have multiple identical machines (DOM elements) running the same plugin. Each machine needs its own settings and state. Instead of keeping all the settings in one global place, the plugin attaches a little notebook (the instance data) to each machine. When you want to talk to a specific machine, you read its notebook. This way, changing the settings on one machine does not affect the others.

### Purposes

- To isolate plugin state per element, preventing one instance from affecting another.
- To enable method invocation via string arguments by retrieving the stored instance.
- To maintain a reference to the plugin instance for cleanup during destruction.
- To avoid global variables and namespace pollution.
- To support multiple independent instances of the same plugin on a single page.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
// Storing instance data
$.data(element, "pluginName", instance);

// Retrieving instance data
var instance = $.data(element, "pluginName");

// Removing instance data
$.removeData(element, "pluginName");
```

**Syntax Rules:**

- The data key should be the plugin name (or a unique identifier) to avoid collisions with other plugins.
- Use `$.data()` (the lower-level utility) or `$(element).data()` (the jQuery object method); both are equivalent.
- Check for an existing instance before initializing to avoid duplicate initialization.
- Remove the data when the plugin is destroyed to prevent memory leaks.

**Constraints and Limitations:**

- If the DOM element is removed without calling `destroy`, the data may be orphaned (though jQuery cleans up data for removed elements in most cases).
- Using `$(element).data()` returns a deep copy in some jQuery versions; use `$.data()` for direct access to the original object.
- The data key must be unique; using a generic key like `"plugin"` can cause collisions with other plugins.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Instance Data for Method Invocation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Instance data demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="counter" id="counter1">Counter 1: 0</div>
  <div class="counter" id="counter2">Counter 2: 0</div>
  <button id="incrementBoth">Increment Both</button>
  <p id="log"></p>

  <script>
    (function($) {
      $.fn.counter = function(action) {
        if (typeof action === "string") {
          return this.each(function() {
            var instance = $.data(this, "counter");
            if (instance && typeof instance[action] === "function") {
              instance[action]();
            }
          });
        }

        return this.each(function() {
          if (!$.data(this, "counter")) {
            $.data(this, "counter", new Counter(this));
          }
        });
      };

      function Counter(element) {
        this.$el = $(element);
        this.count = 0;
        this._init();
      }

      Counter.prototype = {
        _init: function() {
          this._update();
        },
        increment: function() {
          this.count++;
          this._update();
        },
        _update: function() {
          this.$el.text(this.$el.attr("id") + ": " + this.count);
        },
        destroy: function() {
          this.$el.removeData("counter").off(".counter");
        }
      };
    }(jQuery));

    // Initialize
    $(".counter").counter();

    // Increment both via method invocation
    $("#incrementBoth").click(function() {
      $(".counter").counter("increment");
      $("#log").text("Incremented both counters.");
    });
  </script>
</body>
</html>
```

**Expected Output:** Clicking “Increment Both” increments each counter independently. Counter 1 and Counter 2 each maintain their own count.

**Why this output:** Each element stores its own `Counter` instance via `$.data(this, "counter", ...)`. When `counter("increment")` is called, the plugin retrieves each element's instance and invokes its `increment` method. The counts are independent because each instance has its own `count` property.

### Real-World Cases

- **jQuery UI Widgets:** Store the widget instance on the element using `$.data(element, widgetFullName)`.
- **Slick Carousel:** Stores the carousel instance on the element and retrieves it for method calls.
- **Custom sliders:** A slider plugin stores its instance per element to manage independent slider positions.

---

## Core Concept 7: Event Management — Scoping Plugin Events Using Custom Event Namespaces Like `click.pluginName`

### Definitions

**Core Definition:** Event namespacing is a jQuery feature that allows event handlers to be grouped under a namespace, making it possible to remove or trigger specific handlers without affecting other handlers bound to the same event type on the same element.

**Technical Definition:** An event name can be qualified by one or more namespaces using dot notation: `"click.pluginName"` or `"click.pluginName.simple"`. The namespace is not part of the event type; it is a label that can be used with `.off()` to remove only the handlers bound under that namespace. For example, `$(element).off("click.pluginName")` removes all click handlers bound with the `pluginName` namespace, without disturbing other click handlers on the same element. Namespaces are similar to CSS classes: they are not hierarchical, and only one name needs to match for the handler to be removed.

**Beginner-Friendly Explanation:** Imagine you have several people listening to the same conversation (event handlers bound to the same event). Event namespacing is like giving each person a colored badge. When you want to ask everyone with a blue badge to leave, you just say “blue badge, please leave.” The other people stay. This is useful for plugins because it lets them clean up their own event handlers without accidentally removing handlers bound by other code.

### Purposes

- To allow a plugin to remove only its own event handlers without affecting handlers bound by other plugins or application code.
- To enable clean destruction of a plugin instance by unbinding all handlers bound by that plugin.
- To group related event handlers under a common namespace for easier management.
- To avoid namespace collisions between multiple plugins that bind handlers to the same event type on the same element.
- To support targeted triggering of namespaced events.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
// Binding with a namespace
$(selector).on("click.pluginName", handler);

// Removing all handlers with the namespace
$(selector).off("click.pluginName");

// Removing all handlers for the plugin (any event type)
$(selector).off(".pluginName");

// Triggering only namespaced handlers
$(selector).trigger("click.pluginName");
```

| Component | Description |
|-----------|-------------|
| `"click.pluginName"` | A click event handler bound with the `pluginName` namespace. |
| `.off("click.pluginName")` | Removes click handlers with the `pluginName` namespace. |
| `.off(".pluginName")` | Removes all handlers (any event type) with the `pluginName` namespace. |

**Syntax Rules:**

- The namespace follows the event type, separated by a dot: `event.namespace`.
- Multiple namespaces can be chained: `event.namespace1.namespace2`.
- The namespace is not part of the event type; `click.pluginName` still listens for standard click events.
- `.off(".pluginName")` removes all handlers bound with that namespace, regardless of event type.
- Namespaces should contain only upper/lowercase letters and digits.

**Constraints and Limitations:**

- Namespaces are not hierarchical; `"click.myPlugin.simple"` defines two namespaces (`myPlugin` and `simple`), and removing with either one will remove the handler.
- If two plugins use the same namespace, removing one will remove the other's handlers as well; choose unique namespace names.
- Namespacing does not affect event propagation or the event object; it is purely a management mechanism.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Namespaced Event Cleanup**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Event namespacing demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <button id="target">Click me</button>
  <button id="destroyPlugin">Destroy Plugin Handlers</button>
  <p id="log"></p>

  <script>
    // Step 1: Bind a plugin handler with a namespace
    $("#target").on("click.myPlugin", function() {
      $("#log").append("Plugin handler fired.<br>");
    });

    // Step 2: Bind a non-namespaced handler (application code)
    $("#target").on("click", function() {
      $("#log").append("Application handler fired.<br>");
    });

    // Step 3: Destroy only the plugin handler
    $("#destroyPlugin").click(function() {
      $("#target").off("click.myPlugin");
      $("#log").append("Plugin handlers removed.<br>");
    });
  </script>
</body>
</html>
```

**Expected Output:** Before clicking “Destroy Plugin Handlers,” clicking the target logs both “Plugin handler fired” and “Application handler fired.” After destroying, clicking the target logs only “Application handler fired.”

**Why this output:** The plugin handler is bound with the `.myPlugin` namespace, while the application handler is bound without a namespace. Calling `.off("click.myPlugin")` removes only the namespaced handler, leaving the application handler intact. This demonstrates how namespacing enables clean, targeted cleanup.

### Real-World Cases

- **jQuery UI Widgets:** Bind handlers with a unique namespace (e.g., `.progressbar`) and remove them all with `this.element.off(".progressbar")` during destruction.
- **Slick Carousel:** Binds handlers with `.slick` namespace and removes them on destroy.
- **Custom modal plugins:** Bind `keydown.modal`, `click.modal`, and `resize.modal` handlers and remove them all with `.off(".modal")`.

---

## Enhanced Topic: The `this.each()` Loop — Iterating Correctly Through the Matched Set of Elements

### Definitions

**Core Definition:** `this.each()` is a jQuery method that iterates over each element in the jQuery collection, executing a provided callback function once per element. It is the standard mechanism for applying plugin logic to every element in a selection.

**Technical Definition:** `this.each( function(index, element) )` iterates over the elements in the jQuery collection. Inside the callback, `this` refers to the current DOM element (not a jQuery object), and the callback receives the element's index and the raw DOM element as arguments. To use jQuery methods on the current element, you must wrap it with `$(this)`. The `.each()` method returns the original jQuery collection, so `return this.each(...)` is the idiomatic way to perform per-element logic while maintaining chainability.

**Beginner-Friendly Explanation:** When you select multiple elements with jQuery, you get a collection (like a list). `this.each()` is a loop that goes through each element in that list, one by one, and does something to it. Inside the loop, `this` refers to the current element, but it is a raw DOM element, not a jQuery object. To use jQuery methods, you wrap it with `$(this)`. The loop always returns the original collection, so you can keep chaining.

### Purposes

- To apply plugin logic to every element in a jQuery collection, regardless of how many elements are selected.
- To ensure that plugins work correctly when called on multi-element selections.
- To provide a consistent iteration mechanism across all jQuery plugins.
- To maintain chainability by returning the result of `.each()`, which is the original collection.
- To access the index and DOM element of each item in the collection.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
return this.each(function(index, element) {
    // `this` is the raw DOM element
    // `index` is the element's position in the collection
    // `element` is the raw DOM element (same as `this`)
    $(this).css("color", "green");
});
```

| Component | Description |
|-----------|-------------|
| `this` | The jQuery collection (outside the callback). |
| `.each( function )` | Iterates over each element in the collection. |
| `index` | The element's zero-based index in the collection. |
| `element` | The raw DOM element (same as `this` inside the callback). |
| `$(this)` | Wraps the raw DOM element in a jQuery object. |

**Syntax Rules:**

- `this.each()` should be the return value of the plugin to maintain chainability.
- Inside the callback, `this` is the raw DOM element; use `$(this)` to call jQuery methods.
- The callback can optionally receive `index` and `element` arguments.
- Returning `false` from the callback breaks out of the loop early (like `break` in a for loop); returning `true` continues to the next iteration.

**Constraints and Limitations:**

- Forgetting to use `$(this)` inside the callback (and trying to call jQuery methods on `this` directly) causes errors because `this` is a raw DOM element.
- The `.each()` loop is synchronous; it does not wait for asynchronous operations inside the callback.
- Modifying the collection during iteration (e.g., adding or removing elements) can lead to unexpected behavior.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Correct and Incorrect Use of `this.each()`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>this.each demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
</head>
<body>
  <div class="item">Item 1</div>
  <div class="item">Item 2</div>
  <div class="item">Item 3</div>
  <p id="log"></p>

  <script>
    // Step 1: Define a plugin using this.each correctly
    $.fn.colorize = function(color) {
      return this.each(function(index) {
        // `this` is the raw DOM element
        // Correct: wrap with $()
        $(this).css("color", color);
        $("#log").append("Colored item " + index + "<br>");
      });
    };

    // Step 2: Use the plugin
    $(".item").colorize("blue");
  </script>
</body>
</html>
```

**Expected Output:** All three items are colored blue, and the log displays “Colored item 0”, “Colored item 1”, and “Colored item 2”.

**Why this output:** The `.each()` loop iterates over each element. Inside the callback, `$(this)` wraps the raw DOM element, allowing `.css()` to be called. The `index` argument provides the element's position, which is logged.

### Real-World Cases

- **All jQuery plugins:** Every well-written jQuery plugin uses `this.each()` to iterate over the selection.
- **jQuery UI Widgets:** The Widget Factory internally uses `.each()` to create a separate widget instance for each element in the collection.
- **Animation plugins:** A plugin that animates multiple elements uses `.each()` to apply the animation to each one independently.

---

## Enhanced Topic: IIFE Wrapping — Protecting the `$` Alias from Conflict

### Definitions

**Core Definition:** An Immediately Invoked Function Expression (IIFE) is a JavaScript function that is defined and executed immediately. In the context of jQuery plugins, an IIFE is used to create a private scope, safely alias the `$` variable, and prevent global namespace pollution.

**Technical Definition:** An IIFE is written as `(function($) { ... }(jQuery));`. The function is wrapped in parentheses to make it an expression, and then immediately invoked with `jQuery` as the argument. Inside the IIFE, the parameter `$` is a local variable that references the global `jQuery` object, even if `jQuery.noConflict()` has been called and the global `$` has been reassigned to another library. This ensures that the plugin can safely use `$` internally without conflicting with other libraries. The IIFE also provides a private scope for variables and helper functions that should not be exposed globally.

**Beginner-Friendly Explanation:** The `$` symbol is very popular — many JavaScript libraries want to use it. If your plugin assumes `$` means jQuery, but another library has taken over `$`, your plugin breaks. An IIFE solves this by creating a private room where `$` always means jQuery, no matter what is happening outside. You pass jQuery into the room, call it `$` inside, and the plugin works safely.

### Purposes

- To protect the `$` alias from being overwritten by other libraries using `jQuery.noConflict()`.
- To create a private scope for variables and helper functions that should not be exposed globally.
- To prevent global namespace pollution from temporary variables and functions.
- To ensure that the plugin works correctly regardless of the global state of the `$` variable.
- To follow the established jQuery plugin convention, making the plugin compatible with other well-written plugins.

### Syntax Rules and Structure

**Complete General Syntax:**
```javascript
;(function($) {
    "use strict";

    // Private variables and helper functions
    var defaults = { ... };

    // Plugin definition
    $.fn.pluginName = function(options) {
        // Plugin logic
        return this.each(function() {
            // ...
        });
    };

}(jQuery));
```

| Component | Description |
|-----------|-------------|
| `;(function($) { ... }(jQuery));` | The IIFE wrapper. The leading semicolon protects against previous scripts missing a semicolon. |
| `"use strict";` | Optional. Enables strict mode for the plugin code. |
| `var defaults = { ... };` | Private variables and helper functions, not exposed globally. |
| `$.fn.pluginName = function(...)` | The plugin definition, using the local `$` alias. |
| `}(jQuery));` | Passes the global `jQuery` object into the IIFE as the `$` parameter. |

**Syntax Rules:**

- The IIFE must receive `jQuery` as an argument and name the parameter `$`.
- A leading semicolon (`;`) before the IIFE protects against concatenation issues with other scripts.
- `"use strict";` should be placed at the top of the IIFE for stricter error checking.
- Private variables and helper functions should be declared inside the IIFE, not on the global scope.

**Constraints and Limitations:**

- Forgetting the leading semicolon can cause issues if the previous script does not end with a semicolon.
- The IIFE must be properly closed with `}(jQuery));` — a missing parenthesis causes a syntax error.
- The IIFE does not automatically execute on DOM ready; if the plugin needs to manipulate the DOM, the consumer must initialize it after the DOM is ready.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: IIFE-Wrapped Plugin**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>IIFE plugin demo</title>
  <script src="https://code.jquery.com/jquery-4.0.0.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/other-library/1.0/other.js"></script>
</head>
<body>
  <p class="message">Hello</p>
  <p id="log"></p>

  <script>
    // Step 1: Simulate another library taking over $
    var $ = "I am not jQuery!";

    // Step 2: Define the plugin inside an IIFE
    ;(function($) {
      "use strict";

      // Private defaults
      var defaults = {
        color: "green"
      };

      // Plugin definition
      $.fn.greenify = function(options) {
        var settings = $.extend({}, defaults, options);
        return this.each(function() {
          $(this).css("color", settings.color);
        });
      };
    }(jQuery));

    // Step 3: Use the plugin (the global $ is not jQuery, but the plugin still works)
    jQuery(".message").greenify();
    jQuery("#log").text("Plugin applied successfully despite $ conflict.");
  </script>
</body>
</html>
```

**Expected Output:** The message is colored green, and the log displays “Plugin applied successfully despite $ conflict.”

**Why this output:** The global `$` variable has been reassigned to a string by the simulated other library. However, the plugin is wrapped in an IIFE that receives `jQuery` and aliases it as `$` locally. Inside the IIFE, `$` always refers to jQuery, so the plugin works correctly. The consumer uses `jQuery` instead of `$` to access the plugin.

### Real-World Cases

- **Every well-written jQuery plugin:** The IIFE wrapper is the standard convention for jQuery plugins.
- **jQuery UI:** All jQuery UI widgets are wrapped in IIFEs.
- **WordPress themes and plugins:** WordPress uses `jQuery.noConflict()` extensively, so plugins must be IIFE-wrapped to use `$` safely.

---

## References

- How to Create a Basic Plugin — https://learn.jquery.com/plugins/basic-plugin-creation/
- How To Use the Widget Factory — https://learn.jquery.com/jquery-ui/widget-factory/how-to-use-the-widget-factory/
- jQuery.extend() — https://api.jquery.com/jQuery.extend/
- .on() | jQuery API Documentation — https://api.jquery.com/on/
- jQuery Plugin Development Best Practices — https://github.com/jquery-boilerplate/jquery-boilerplate
- Ben Alman — Immediately-Invoked Function Expression (IIFE) — http://benalman.com/news/2010/11/immediately-invoked-function-expression/
- jQuery Plugin Pattern with Data Persistence — https://stackoverflow.com/questions/9926248/jquery-plugin-pattern-with-data-persistence
- jQuery Boilerplate — https://jqueryboilerplate.com/
- jQuery Learning Center — Plugins — https://learn.jquery.com/plugins/
- jQuery API Documentation — .data() — https://api.jquery.com/data/
- jQuery API Documentation — .each() — https://api.jquery.com/each/
- jQuery API Documentation — .off() — https://api.jquery.com/off/