# jQuery Plugin Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Plugin Patterns are reusable architectural templates for structuring jQuery plugin code, each optimized for different complexity levels, state management needs, and integration requirements.

**Technical Definition:**
jQuery plugin patterns are established code organization strategies that determine how a plugin is assigned to the jQuery prototype (`$.fn`), how options are merged, how state is managed, how methods are exposed, and how events are communicated. These patterns range from the minimal IIFE-wrapped function assignment (Basic Pattern) to the full object-oriented Widget Factory (Factory Pattern), with intermediate patterns addressing namespacing, options management, method dispatch, event communication, and singleton instantiation.

**Beginner-Friendly Explanation:**
A plugin pattern is a recipe for how to write your plugin. Just as a house can be built in different styles (a simple cabin, a multi-story home, or a factory-built modular unit), a jQuery plugin can be written in different patterns. The Basic Pattern is like a simple cabin — quick to build and good for simple tasks. The Widget Factory is like a modular home — more complex to set up but handles state, methods, and events elegantly. Choosing the right pattern depends on what your plugin needs to do.

### Key Characteristics

- **Progressive Complexity:** Patterns range from stateless one-function plugins to stateful object-oriented widgets.
- **Stateless vs. Stateful:** Basic patterns are stateless (perform an action and done); Widget Factory patterns are stateful (maintain state across method calls).
- **Namespace Isolation:** All patterns emphasize claiming only a single name in the jQuery namespace to avoid collisions.
- **Chainability Preservation:** Most patterns return `this` to allow jQuery method chaining.
- **Option Merging:** Patterns that accept options merge user-supplied options with defaults using `$.extend()`.
- **Event Communication:** Advanced patterns use custom events or pub/sub for decoupled communication.
- **Multiple Instantiation Support:** Patterns differ in how they handle multiple plugin instances on the same page.

### Prerequisites

- Solid understanding of JavaScript functions, closures, and the `this` keyword
- Working knowledge of jQuery selectors and DOM manipulation
- Familiarity with `$.fn` and how jQuery object methods work
- Understanding of `$.extend()` for object merging
- Basic knowledge of IIFEs (Immediately Invoked Function Expressions)

### Related Programming Areas

- **jQuery UI Widget Factory:** The official framework for stateful jQuery plugins.
- **JavaScript Design Patterns:** Module pattern, singleton pattern, factory pattern, and observer pattern.
- **Plugin Architecture:** How plugins extend host applications without modification.
- **Event-Driven Programming:** Custom events and pub/sub for plugin communication.
- **Namespace Management:** Avoiding global scope pollution.

### Core Concepts / Features

1. Basic Plugin Pattern
2. Namespaced Plugin Pattern
3. Options Pattern
4. Methods Pattern
5. Events Pattern
6. Callbacks Pattern
7. Singleton Pattern
8. Factory Pattern
9. Plugin Pattern Best Practices

---

## 1. Basic Plugin Pattern

### Definitions

**Core Definition:**
The Basic Plugin Pattern is the simplest jQuery plugin structure, consisting of a single function assigned to `$.fn` that performs an action on the selected elements.

**Technical Definition:**
The Basic Plugin Pattern assigns a function directly to `$.fn.pluginName` inside an IIFE. The function iterates over the collection with `this.each()` and returns `this` for chainability. It has no state management, no method dispatch, and no lifecycle methods. This pattern is suitable for stateless plugins that perform a single action.

**Beginner-Friendly Explanation:**
The Basic Pattern is the "hello world" of jQuery plugins. You write one function, attach it to `$.fn`, loop through the elements, do something, and return `this`. It is perfect for simple tasks like changing colors, adding classes, or applying styles.

### Purposes

- To create simple, stateless plugins with minimal boilerplate.
- To perform a single action on a set of elements.
- To provide a quick starting point for plugin development.
- To serve as the foundation for more complex patterns.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    $.fn.pluginName = function() {
        return this.each(function() {
            // Plugin logic
        });
    };
}(jQuery));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `;(function($) { ... }(jQuery))` | IIFE that protects the `$` alias. |
| `$.fn.pluginName` | Assignment of the plugin to the jQuery prototype. |
| `return this.each(...)` | Iterates over elements and returns `this` for chaining. |

**Syntax Rules:**

1. Always wrap the plugin in an IIFE and pass `jQuery` as `$`.
2. Use `this.each()` for implicit iteration over multiple elements.
3. Return `this` (or `this.each(...)`) to preserve chainability.
4. Claim only a single name in the jQuery namespace.

**Constraints and Limitations:**

- No state management; the plugin cannot remember anything between calls.
- No method dispatch; there is only one function.
- No lifecycle methods (create, destroy).
- Cannot accept complex options without additional code.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Color Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Basic Plugin Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <p class="text">Paragraph 1</p>
    <p class="text">Paragraph 2</p>

    <script>
        ;(function($) {
            $.fn.makeGreen = function() {
                return this.each(function() {
                    $(this).css("color", "green");
                });
            };
        }(jQuery));

        $(".text").makeGreen().addClass("greenified");
        console.log("Greenified: " + $(".greenified").length);
        console.log("Color: " + $(".text").first().css("color"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Greenified: 2
Color: rgb(0, 128, 0)
```

**Why This Code Produces That Result:**
The IIFE protects `$`, and the plugin is assigned to `$.fn.makeGreen`. Inside the plugin, `this` is the jQuery collection, and `this.each()` iterates over each `<p>` element. Returning `this.each(...)` returns the jQuery object, allowing `.addClass("greenified")` to be chained. Both paragraphs turn green and receive the class.

### Real-World Cases

- **Simple Animations:** A `fadeOutSlow()` plugin for consistent fade-out effects.
- **CSS Utilities:** A `centerText()` plugin for text alignment.
- **DOM Helpers:** A `removeEmpty()` plugin for cleaning up empty elements.
- **Quick Prototyping:** Testing an idea before committing to a more complex pattern.

---

## 2. Namespaced Plugin Pattern

### Definitions

**Core Definition:**
The Namespaced Plugin Pattern organizes plugin methods under a unique namespace to avoid collisions and improve code organization when working with multiple plugins or components.

**Technical Definition:**
The Namespaced Pattern creates a namespace object on `$` (e.g., `$.myPlugin = $.myPlugin || {}`) and attaches public methods and defaults to that namespace. The main plugin function is assigned to `$.fn.pluginName`, but its auxiliary methods are exposed on the namespace object. This pattern is useful when multiple plugins share a common prefix or belong to the same project.

**Beginner-Friendly Explanation:**
Imagine you have three plugins: `format`, `validate`, and `serialize`. Without a namespace, they might conflict with other `format` plugins. With a namespace, you group them under `$.myProject.format`, `$.myProject.validate`, and `$.myProject.serialize`. Now they are clearly part of the same project and do not conflict with anything else.

### Purposes

- To avoid name collisions between multiple plugins and jQuery core.
- To group related plugin methods under a single, memorable namespace.
- To improve code organization in large projects with multiple plugins.
- To expose public utility methods without polluting `$.fn`.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    $.myNamespace = $.myNamespace || {};

    $.myNamespace.utilityMethod = function() {
        // Public utility
    };

    $.fn.myPlugin = function() {
        // Plugin uses $.myNamespace.utilityMethod()
        return this.each(function() {
            // ...
        });
    };
}(jQuery));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$.myNamespace = $.myNamespace \|\| {}` | Creates the namespace if it does not exist. |
| `$.myNamespace.method` | Public method attached to the namespace. |
| `$.fn.myPlugin` | The main plugin function. |

**Syntax Rules:**

1. Use `$.myNamespace = $.myNamespace || {}` to avoid overwriting an existing namespace.
2. Claim only a single name in the jQuery namespace.
3. Expose auxiliary methods on the namespace, not on `$.fn`.
4. Deep namespaces (e.g., `$.myProject.pluginA`) can be created by nesting objects.

**Constraints and Limitations:**

- Namespace collisions are still possible if two projects choose the same namespace name.
- Exposing too much in the namespace can lead to API bloat.
- Deep namespacing adds complexity to method invocation.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Namespaced Text Tools**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Namespaced Plugin Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <p class="text">Hello World</p>

    <script>
        ;(function($) {
            $.textTools = $.textTools || {};

            $.textTools.bold = function(text) {
                return "<strong>" + text + "</strong>";
            };

            $.textTools.italic = function(text) {
                return "<em>" + text + "</em>";
            };

            $.fn.formatText = function(options) {
                var settings = $.extend({
                    bold: true,
                    italic: false
                }, options);

                return this.each(function() {
                    var $this = $(this);
                    var text = $this.text();
                    if (settings.bold) text = $.textTools.bold(text);
                    if (settings.italic) text = $.textTools.italic(text);
                    $this.html(text);
                });
            };
        }(jQuery));

        $(".text").formatText({ italic: true });
        console.log($(".text").html());
    </script>
</body>
</html>
```

**Expected Output:**
```
<strong><em>Hello World</em></strong>
```

**Why This Code Produces That Result:**
The `$.textTools` namespace groups the `bold` and `italic` methods. The `$.fn.formatText` plugin uses these namespace methods internally. Users can also call `$.textTools.bold("text")` directly. The namespace prevents `bold` and `italic` from polluting the global scope or conflicting with other plugins.

### Real-World Cases

- **jQuery UI:** All widgets use the `ui.` namespace (e.g., `$.ui.dialog`).
- **Custom Corporate Libraries:** Companies use `$.myCompany.pluginA`, `$.myCompany.pluginB`.
- **Utility Libraries:** `$.cookie`, `$.storage`, `$.url` — each is a namespace for related utility functions.

---

## 3. Options Pattern

### Definitions

**Core Definition:**
The Options Pattern is a plugin structure that accepts a configuration object, merges it with default settings using `$.extend()`, and exposes defaults for global customization.

**Technical Definition:**
The Options Pattern defines a defaults object as `$.fn.pluginName.defaults`, merges user options with defaults inside the plugin function using `$.extend({}, defaults, options)`, and uses the merged settings throughout the plugin. This pattern is used in nearly all jQuery plugins that accept configuration.

**Beginner-Friendly Explanation:**
The Options Pattern is like a settings menu. The plugin has default settings (e.g., color = red). The user can pass an object to override any of those settings (e.g., `{ color: "blue" }`). The plugin combines the defaults with the user's choices to create a complete configuration.

### Purposes

- To provide sensible defaults while allowing user customization.
- To support both global default overrides and per-call overrides.
- To avoid mutating the defaults object across multiple plugin calls.
- To document available options through the defaults object.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    $.fn.pluginName = function(options) {
        var settings = $.extend({}, $.fn.pluginName.defaults, options);
        return this.each(function() {
            // Use settings.optionName
        });
    };

    $.fn.pluginName.defaults = {
        option1: value1,
        option2: value2
    };
}(jQuery));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `options` | User-supplied configuration object. |
| `$.fn.pluginName.defaults` | The plugin's default settings. |
| `$.extend({}, defaults, options)` | Creates a new merged settings object. |
| `settings` | The merged configuration used by the plugin. |

**Syntax Rules:**

1. Always pass an empty object `{}` as the first argument to `$.extend()` to avoid mutating defaults.
2. Expose defaults as `$.fn.pluginName.defaults` for global customization.
3. For deep merging of nested objects, pass `true` as the first argument.
4. Document all available options and their default values.

**Constraints and Limitations:**

- Shallow merging does not handle nested objects; use deep merging for complex configurations.
- Global defaults modification affects all future plugin calls, not existing instances.
- Undocumented options are a common source of bugs.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Options Pattern with Global and Per-Call Overrides**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Options Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="box">Box 1</div>
    <div class="box">Box 2</div>

    <script>
        ;(function($) {
            $.fn.boxify = function(options) {
                var settings = $.extend({}, $.fn.boxify.defaults, options);
                return this.each(function() {
                    $(this).css({
                        backgroundColor: settings.bgColor,
                        padding: settings.padding
                    });
                });
            };

            $.fn.boxify.defaults = {
                bgColor: "#f0f0f0",
                padding: "10px"
            };
        }(jQuery));

        $(".box").boxify();
        console.log("Default: " + $(".box").first().css("background-color"));

        $.fn.boxify.defaults.bgColor = "#e0e0e0";
        $(".box").boxify();
        console.log("Global: " + $(".box").first().css("background-color"));

        $(".box").boxify({ bgColor: "#d0d0d0" });
        console.log("Per-call: " + $(".box").first().css("background-color"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Default: rgb(240, 240, 240)
Global: rgb(224, 224, 224)
Per-call: rgb(208, 208, 208)
```

**Why This Code Produces That Result:**
The `$.extend({}, defaults, options)` call creates a new settings object each time, preventing mutation of the defaults. The first call uses all defaults. After `$.fn.boxify.defaults.bgColor` is changed, subsequent calls use the new default. Per-call options override both the global defaults and the original defaults for that specific invocation.

### Real-World Cases

- **Date Pickers:** Options for date format, language, and minimum date.
- **Sliders:** Options for minimum, maximum, step, and orientation.
- **Modals:** Options for width, height, and animation speed.
- **Form Validation:** Options for error messages and validation rules.

---

## 4. Methods Pattern

### Definitions

**Core Definition:**
The Methods Pattern allows a plugin to expose multiple public methods that can be invoked by passing the method name as a string to the main plugin function.

**Technical Definition:**
The Methods Pattern uses a `methods` object to store public methods and a dispatcher in the main plugin function that checks the first argument. If it is a string, the corresponding method is called with the remaining arguments. If it is an object or undefined, the `init` method is called. This pattern is used for stateful plugins that need more than one entry point.

**Beginner-Friendly Explanation:**
The Methods Pattern is like a TV remote with multiple buttons. Instead of one button that does everything, you have separate buttons for "channel up," "channel down," "volume up," etc. You press a button (pass a method name) and the remote performs that specific action.

### Purposes

- To expose multiple public methods from a single plugin.
- To enable stateful interactions beyond simple initialization.
- To provide a consistent API for programmatic control.
- To allow plugin users to invoke behavior by name.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    var methods = {
        init: function(options) { /* ... */ },
        method1: function() { /* ... */ },
        method2: function() { /* ... */ }
    };

    $.fn.myPlugin = function(method) {
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
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `methods` | Object holding all public methods. |
| `$.fn.myPlugin` | Dispatcher function that routes to the correct method. |
| `methods[method].apply(...)` | Calls the method with the remaining arguments. |
| `methods.init.apply(...)` | Calls `init` if the first argument is an object or undefined. |

**Syntax Rules:**

1. The `methods` object should contain an `init` method for initialization.
2. The dispatcher checks if the first argument is a string (method name) or an object (options).
3. Use `.apply()` to pass the remaining arguments to the method.
4. Throw an error for unknown method names using `$.error()`.

**Constraints and Limitations:**

- String-based method dispatch lacks IDE autocompletion and type checking.
- All methods share the same `this` context (the jQuery collection).
- Adding too many methods bloats the plugin's API.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Methods Pattern with Counter Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Methods Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="counter">0</div>

    <script>
        ;(function($) {
            var methods = {
                init: function(options) {
                    var settings = $.extend({
                        initial: 0,
                        step: 1
                    }, options);
                    return this.each(function() {
                        var $this = $(this);
                        $this.data("counter", {
                            value: settings.initial,
                            step: settings.step
                        });
                        $this.text(settings.initial);
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

        $("#counter").counter({ initial: 10, step: 5 });
        console.log("Initial: " + $("#counter").text());

        $("#counter").counter("increment");
        console.log("After increment: " + $("#counter").text());

        console.log("getValue: " + $("#counter").counter("getValue"));
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
The dispatcher checks if the first argument is a string. If so, it calls the corresponding method from the `methods` object. The `init` method stores state on the element via `.data("counter")`. The `increment` method retrieves that state, updates the value, and updates the display. The `getValue` method returns the current value. The string-based dispatch allows multiple public methods from a single plugin.

### Real-World Cases

- **Date Picker:** Methods like `getDate()`, `setDate()`, `show()`, `hide()`.
- **Dialog:** Methods like `open()`, `close()`, `isOpen()`, `destroy()`.
- **Slider:** Methods like `value()`, `values()`, `enable()`, `disable()`.

---

## 5. Events Pattern

### Definitions

**Core Definition:**
The Events Pattern allows a plugin to trigger custom events that the host application can listen for, enabling decoupled communication between the plugin and other parts of the application.

**Technical Definition:**
The Events Pattern uses `this._trigger()` in the Widget Factory or `$(element).trigger()` in stateless plugins to fire custom events. Events follow a naming convention (`pluginname:eventName` or `pluginnameeventname` in Widget Factory). The host application listens for these events using `.on()`.

**Beginner-Friendly Explanation:**
The Events Pattern is like a doorbell. The plugin rings the doorbell (triggers an event) when something happens (e.g., a date is selected). The application hears the doorbell and responds (e.g., updates a calendar). The plugin does not need to know what the application does — it just rings the bell.

### Purposes

- To notify the host application of plugin state changes.
- To enable loose coupling between the plugin and application logic.
- To provide hooks for extending plugin behavior without modifying the source.
- To allow multiple listeners to respond to the same plugin event.

### Syntax Rules and Structure

**Complete General Syntax (Widget Factory):**

```javascript
this._trigger("eventName", null, { dataKey: dataValue });
```

**Complete General Syntax (Stateless):**

```javascript
$(element).trigger("pluginname:eventName", { dataKey: dataValue });
```

**Complete General Syntax (Listening):**

```javascript
$(element).on("pluginname:eventName", function(event, data) {
    // Handle event
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `_trigger()` | Widget Factory method for triggering events. |
| `"eventName"` | The event name; Widget Factory prefixes it with the widget name. |
| `null` | The original event (or null if none). |
| `{ dataKey: dataValue }` | Data payload passed to event handlers. |

**Syntax Rules:**

1. Use `this._trigger()` inside Widget Factory plugins.
2. For stateless plugins, use `$(element).trigger("pluginname:eventName", data)`.
3. Event names should be descriptive and follow the `pluginname:eventName` convention.
4. Listen for events using `.on("pluginname:eventName", handler)`.

**Constraints and Limitations:**

- Widget Factory event names are automatically lowercased and concatenated.
- Custom events do not bubble unless created with `bubbles: true`.
- Triggering events too frequently can cause performance issues.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Custom Event in a Widget**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Events Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
</head>
<body>
    <div id="colorWidget">Red</div>

    <script>
        $.widget("demo.colorWidget", {
            options: { color: "red" },

            _create: function() {
                this.element.css("color", this.options.color);
            },

            setColor: function(newColor) {
                var previous = this.options.color;
                this.options.color = newColor;
                this.element.css("color", newColor);
                this._trigger("colorChanged", null, {
                    previous: previous,
                    current: newColor
                });
            }
        });

        $("#colorWidget").colorWidget();

        $("#colorWidget").on("colorwidgetcolorchanged", function(event, data) {
            console.log("Previous: " + data.previous);
            console.log("Current: " + data.current);
        });

        $("#colorWidget").colorWidget("setColor", "blue");
    </script>
</body>
</html>
```

**Expected Output:**
```
Previous: red
Current: blue
```

**Why This Code Produces That Result:**
The `_trigger("colorChanged", ...)` call fires the event `colorwidgetcolorchanged` (the Widget Factory lowercases and concatenates the widget name and event name). The handler receives the event object and the data payload. The `previous` value is captured before the change, and the `current` value is the new color. The host application listens for the event and responds accordingly.

### Real-World Cases

- **Date Picker:** `datepicker:select` event fired when a date is selected.
- **Slider:** `slidechange` event fired when the slider value changes.
- **Dialog:** `dialogopen` and `dialogclose` events.
- **Autocomplete:** `autocompleteselect` and `autocompletechange` events.

---

## 6. Callbacks Pattern

### Definitions

**Core Definition:**
The Callbacks Pattern allows users to pass functions as options that the plugin invokes at specific points during execution, enabling custom behavior without modifying the plugin source.

**Technical Definition:**
Callbacks are defined as properties in the options object (e.g., `onChange: function() {}`) and invoked by the plugin at the appropriate time using `settings.onChange.call(this, args)`. The `.call()` method ensures the correct `this` context. Callbacks differ from events: callbacks are passed at initialization time and are invoked by the plugin; events are triggered on elements and can be listened to by multiple handlers.

**Beginner-Friendly Explanation:**
A callback is a function you give to the plugin that it calls back later. For example, you might tell a date picker: "When the user selects a date, call this function." The plugin stores your function and invokes it when the date changes.

### Purposes

- To allow users to inject custom behavior at specific plugin lifecycle points.
- To provide a simple, direct way to respond to plugin actions.
- To enable customization without requiring the user to bind events.
- To support one-time or per-invocation callback logic.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$.fn.myPlugin = function(options) {
    var settings = $.extend({
        onChange: function() {}
    }, options);

    return this.each(function() {
        // ...
        if (typeof settings.onChange === "function") {
            settings.onChange.call(this, value);
        }
    });
};
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `onChange` | Callback option name (convention: `on` prefix). |
| `settings.onChange.call(this, value)` | Invokes the callback with the correct `this` context. |
| `typeof settings.onChange === "function"` | Guards against non-function values. |

**Syntax Rules:**

1. Prefix callback option names with `on` (e.g., `onChange`, `onComplete`).
2. Always check that the callback is a function before invoking it.
3. Use `.call()` or `.apply()` to set the correct `this` context.
4. Provide a default no-op function for callbacks to avoid errors.

**Constraints and Limitations:**

- Only one callback per option can be supplied (unlike events).
- Callbacks are not chainable; they are invoked by the plugin.
- Errors in callbacks can break plugin execution if not handled.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Callback Option in a Plugin**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Callbacks Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="message">Hello</div>

    <script>
        ;(function($) {
            $.fn.transformText = function(options) {
                var settings = $.extend({
                    transform: function(text) {
                        return text.toUpperCase();
                    },
                    onComplete: function(result) {}
                }, options);

                return this.each(function() {
                    var $this = $(this);
                    var original = $this.text();
                    var transformed = settings.transform(original);
                    $this.text(transformed);
                    settings.onComplete.call(this, transformed);
                });
            };
        }(jQuery));

        $("#message").transformText({
            onComplete: function(result) {
                console.log("Transformed: " + result);
            }
        });
    </script>
</body>
</html>
```

**Expected Output:**
```
Transformed: HELLO
```

**Why This Code Produces That Result:**
The plugin merges user options with defaults. The `transform` callback (default: uppercase) is called with the original text. The `onComplete` callback is invoked with `.call(this, transformed)`, ensuring that `this` inside the callback refers to the DOM element. The user-supplied `onComplete` logs the transformed text.

### Real-World Cases

- **Animation Plugins:** `onComplete` callback fired when an animation finishes.
- **AJAX Plugins:** `onSuccess`, `onError`, and `onComplete` callbacks.
- **Form Validation:** `onValid` and `onInvalid` callbacks for custom feedback.
- **Drag-and-Drop:** `onDragStart`, `onDrag`, and `onDrop` callbacks.

---

## 7. Singleton Pattern

### Definitions

**Core Definition:**
The Singleton Pattern ensures that only one instance of a plugin exists across the entire page, typically by storing the instance in a closure or on the jQuery namespace.

**Technical Definition:**
The Singleton Pattern creates a single object instance using an IIFE that returns an object and assigns it to a global variable or namespace property using `MY_SINGLETON_OBJ = MY_SINGLETON_OBJ || (function() { ... })()`. The instance is created only once; subsequent calls return the existing instance.

**Beginner-Friendly Explanation:**
The Singleton Pattern is like a hotel front desk — there is only one, and everyone uses the same one. No matter how many times you call the plugin, you get the same instance. This is useful for things like configuration managers, global tooltips, or application-wide modals.

### Purposes

- To ensure a single, shared instance of a plugin or utility.
- To provide a central point of control for global behavior.
- To avoid redundant initialization of expensive resources.
- To enable shared state across multiple parts of an application.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
;(function($) {
    var instance = null;

    $.mySingleton = function() {
        if (!instance) {
            instance = {
                property: value,
                method: function() { }
            };
        }
        return instance;
    };
}(jQuery));
```

**Complete General Syntax (Closure-Based):**

```javascript
;(function($) {
    var singleton = (function() {
        var instance;
        function createInstance() {
            return { /* ... */ };
        }
        return {
            getInstance: function() {
                if (!instance) {
                    instance = createInstance();
                }
                return instance;
            }
        };
    })();

    $.mySingleton = singleton;
}(jQuery));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `instance` | Private variable holding the single instance. |
| `if (!instance)` | Check to see if the instance already exists. |
| `return instance` | Returns the existing or newly created instance. |

**Syntax Rules:**

1. Store the instance in a private variable inside the IIFE.
2. Check if the instance exists before creating it.
3. Expose the singleton via `$.mySingleton` or a namespace property.
4. For jQuery plugins that must be singletons, store the instance on `$` rather than on DOM elements.

**Constraints and Limitations:**

- Singletons are global state; they can make testing and reasoning about code harder.
- The singleton pattern can hide dependencies, making code less modular.
- In jQuery plugin contexts, singletons are less common than per-element instances.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Singleton Configuration Manager**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Singleton Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="log"></div>

    <script>
        ;(function($) {
            var instance = null;

            $.appConfig = function() {
                if (!instance) {
                    instance = {
                        settings: {
                            theme: "light",
                            language: "en"
                        },
                        get: function(key) {
                            return this.settings[key];
                        },
                        set: function(key, value) {
                            this.settings[key] = value;
                        }
                    };
                }
                return instance;
            };
        }(jQuery));

        var config1 = $.appConfig();
        var config2 = $.appConfig();

        console.log("Same instance: " + (config1 === config2));
        console.log("Theme: " + config1.get("theme"));

        config1.set("theme", "dark");
        console.log("Theme after change: " + config2.get("theme"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Same instance: true
Theme: light
Theme after change: dark
```

**Why This Code Produces That Result:**
The `instance` variable is stored in the closure and is only created once. Both `config1` and `config2` reference the same object, so `config1 === config2` is `true`. When `config1.set("theme", "dark")` is called, the change is reflected in `config2` because they share the same instance. This is the core behavior of the Singleton Pattern.

### Real-World Cases

- **Configuration Managers:** A single object holding application-wide settings.
- **Modal Managers:** Ensuring only one modal is open at a time.
- **Tooltip Managers:** A single tooltip instance reused across multiple elements.
- **Event Bus:** A single pub/sub object for application-wide communication.

---

## 8. Factory Pattern

### Definitions

**Core Definition:**
The Factory Pattern uses a factory function to create plugin instances, typically via the jQuery UI Widget Factory (`$.widget()`), which manages state, lifecycle, and a consistent API.

**Technical Definition:**
The Factory Pattern is implemented through `$.widget("namespace.widgetName", { ... })`, which creates a stateful plugin with lifecycle methods (`_create`, `_init`, `_destroy`), option management, method invocation, and event triggering. The factory handles instance creation, storage, and retrieval, providing a consistent API across all widgets.

**Beginner-Friendly Explanation:**
The Factory Pattern is like a car factory. You provide a blueprint (the widget definition), and the factory produces cars (widget instances) with consistent features: they all have doors, engines, and steering wheels. You do not have to build each car from scratch — the factory handles the common parts.

### Purposes

- To create stateful plugins with a consistent API.
- To manage plugin lifecycle (creation, initialization, destruction).
- To provide multiple methods and events through a single plugin.
- To remove boilerplate associated with state management.
- To enable extension and customization through a standardized interface.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
$.widget("namespace.widgetName", {
    options: {
        option1: defaultValue1
    },

    _create: function() {
        // Initialization
    },

    _init: function() {
        // Re-initialization
    },

    publicMethod: function() {
        // Public method
    },

    _destroy: function() {
        // Cleanup
    }
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `"namespace.widgetName"` | The widget name; namespace is required. |
| `options` | Default options object. |
| `_create` | Called when the widget is first initialized. |
| `_init` | Called on re-initialization. |
| `publicMethod` | Public methods callable by name. |
| `_destroy` | Called when the widget is destroyed. |

**Syntax Rules:**

1. The widget name must contain a namespace (e.g., `"custom.progressbar"`). Only one level of namespace is allowed.
2. The `ui` namespace is reserved for official jQuery UI widgets; create your own namespace.
3. Use `this.element` (a jQuery object) and `this.options` inside the widget.
4. Use `this._trigger()` to fire custom events.
5. Call `_super()` to invoke parent methods when extending widgets.

**Constraints and Limitations:**

- The Widget Factory adds a dependency on jQuery UI (or at least the widget factory file).
- Widget Factory method invocation uses string-based dispatch.
- Widgets are single-element oriented; each element gets its own instance.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Progress Bar Widget**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Factory Pattern</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
</head>
<body>
    <div id="progress"></div>

    <script>
        $.widget("demo.progressbar", {
            options: {
                value: 0
            },

            _create: function() {
                this.element.addClass("progressbar");
                this._update();
            },

            _update: function() {
                this.element.text(this.options.value + "%");
            },

            setValue: function(newValue) {
                this.options.value = newValue;
                this._update();
                this._trigger("change", null, { value: newValue });
            },

            _destroy: function() {
                this.element.removeClass("progressbar").text("");
            }
        });

        $("#progress").progressbar({ value: 20 });
        console.log("Initial: " + $("#progress").text());

        $("#progress").progressbar("setValue", 50);
        console.log("Updated: " + $("#progress").text());

        $("#progress").progressbar("destroy");
        console.log("After destroy: " + $("#progress").text());
    </script>
</body>
</html>
```

**Expected Output:**
```
Initial: 20%
Updated: 50%
After destroy: 
```

**Why This Code Produces That Result:**
The `$.widget()` factory creates a stateful plugin. The `_create` method adds a class and displays the initial value. The `setValue` public method updates the value and triggers a `change` event. The `_destroy` method cleans up when the widget is destroyed. The Widget Factory manages instance creation, option storage, and method dispatch automatically.

### Real-World Cases

- **jQuery UI Widgets:** All jQuery UI widgets (dialog, datepicker, slider) use the Widget Factory.
- **Data Tables:** Complex table plugins use factory patterns for state management.
- **Form Builders:** Drag-and-drop form builders use factory patterns for field configuration.
- **Rich Text Editors:** Editors use factory patterns for extensibility.

---

## 9. Plugin Pattern Best Practices

### Definitions

**Core Definition:**
Plugin pattern best practices are empirically established guidelines for choosing and implementing the right jQuery plugin pattern for a given use case, ensuring performance, maintainability, and compatibility.

**Technical Definition:**
Best practices encompass choosing the simplest pattern that meets requirements, wrapping all patterns in IIFEs, claiming only a single name in the jQuery namespace, returning `this` for chainability, exposing defaults for global customization, namespacing events for cleanup, and documenting the chosen pattern and its trade-offs.

**Beginner-Friendly Explanation:**
These are the proven rules for choosing and using plugin patterns. They help you pick the right pattern for your plugin and implement it correctly.

### Purposes

- To choose the right pattern for the right job.
- To ensure plugins are performant, maintainable, and conflict-free.
- To prevent common mistakes that lead to broken plugins.
- To establish a professional standard for plugin development.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Why |
|---|----------|-----|
| 1 | Choose the simplest pattern that works | Avoid over-engineering. |
| 2 | Wrap in IIFE | Protect `$` and avoid global pollution. |
| 3 | Claim only one namespace name | Avoid collisions. |
| 4 | Return `this` for chainability | Maintain jQuery's fluent API. |
| 5 | Expose defaults | Allow global customization. |
| 6 | Namespace events | Enable clean unbinding. |
| 7 | Use Widget Factory for stateful plugins | Remove boilerplate. |
| 8 | Document the pattern | Help users understand the API. |

**Syntax Rules:**

1. Use the Basic Pattern for simple, stateless plugins.
2. Use the Options Pattern when the plugin accepts configuration.
3. Use the Methods Pattern when the plugin needs multiple public methods.
4. Use the Widget Factory for complex, stateful widgets.
5. Always follow the jQuery plugin authoring conventions.

**Constraints and Limitations:**

- No single pattern is right for every situation.
- Over-engineering with the Widget Factory for simple plugins adds unnecessary complexity.
- Under-engineering with the Basic Pattern for stateful plugins leads to fragile code.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Choosing the Right Pattern**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Pattern Selection</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="log"></div>

    <script>
        // ✅ BASIC PATTERN: Simple, stateless
        ;(function($) {
            $.fn.makeRed = function() {
                return this.each(function() {
                    $(this).css("color", "red");
                });
            };
        }(jQuery));

        // ✅ OPTIONS PATTERN: Configurable, stateless
        ;(function($) {
            $.fn.styled = function(options) {
                var settings = $.extend({}, $.fn.styled.defaults, options);
                return this.each(function() {
                    $(this).css({
                        color: settings.color,
                        fontSize: settings.fontSize
                    });
                });
            };
            $.fn.styled.defaults = {
                color: "#333",
                fontSize: "14px"
            };
        }(jQuery));

        // ✅ METHODS PATTERN: Stateful, multiple methods
        ;(function($) {
            var methods = {
                init: function(options) {
                    return this.each(function() {
                        $(this).data("counter", { value: 0 });
                    });
                },
                increment: function() {
                    return this.each(function() {
                        var $this = $(this);
                        var data = $this.data("counter");
                        data.value++;
                        $this.text(data.value);
                    });
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

        console.log("Patterns loaded successfully");
    </script>
</body>
</html>
```

**Expected Output:**
```
Patterns loaded successfully
```

**Why This Code Produces That Result:**
Three different patterns are demonstrated, each chosen for a specific use case: the Basic Pattern for a simple stateless action, the Options Pattern for configurable styling, and the Methods Pattern for stateful counter behavior. This illustrates the principle of choosing the simplest pattern that meets the requirements.

### Real-World Cases

- **Simple Utilities:** Use the Basic Pattern for one-off actions.
- **Configurable Components:** Use the Options Pattern for themable widgets.
- **Interactive Components:** Use the Methods Pattern for stateful UI controls.
- **Complex Widgets:** Use the Widget Factory for full-featured components.

---

## References

- Addy Osmani — Essential jQuery Plugin Patterns - https://www.smashingmagazine.com/2011/10/essential-jquery-plugin-patterns/
- GitHub — contextworks/jquery-plugin-patterns - https://github.com/contextworks/jquery-plugin-patterns
- jQuery Learning Center — Why Use the Widget Factory? - https://learn.jquery.com/jquery-ui/widget-factory/why-use-the-widget-factory/
- jQuery Learning Center — How To Use the Widget Factory - https://learn.jquery.com/jquery-ui/widget-factory/how-to-use-the-widget-factory/
- jQuery Learning Center — How to Create a Basic Plugin - https://learn.jquery.com/plugins/basic-plugin-creation/
- jQuery Learning Center — Advanced Plugin Concepts - https://learn.jquery.com/plugins/advanced-plugin-concepts/
- jQuery API Documentation — Widget Factory - https://api.jqueryui.com/jQuery.widget/
- jQuery API Documentation — Widget Plugin Bridge - https://api.jqueryui.com/jQuery.widget.bridge/
- SitePoint — A Basic jQuery Plugin using the Module Pattern - https://www.sitepoint.com/basic-jquery-plugin-module-pattern/
- Stack Overflow — jQuery Plugin Singleton Pattern - https://stackoverflow.com/questions/11495996/is-this-a-valid-jquery-plugin-pattern
- Stack Overflow — jQuery Plugin Callbacks - https://stackoverflow.com/questions/10202447/jquery-plugin-adding-callback-functionality
- Viget — A Few jQuery Plugin Patterns - https://www.viget.com/articles/a-few-jquery-plugin-patterns/
- Learning jQuery, Fourth Edition — Chapter 8: Developing Plugins - https://archive.org/stream/9781782163145LEARNINGJQUERYFOURTHEDITION