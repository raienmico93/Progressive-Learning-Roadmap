# jQuery CSS Manipulation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery CSS Manipulation is the set of methods for reading computed styles, setting inline styles, managing CSS classes, working with CSS custom properties (variables), and coordinating CSS transitions and animations on DOM elements.

**Technical Definition:**
jQuery CSS manipulation is centered on `.css()`, which serves as both a getter (returning the computed style of the first matched element via `getComputedStyle()`) and a setter (writing inline styles to every matched element). The method normalizes vendor-specific property names (e.g., `float` → `cssFloat`/`styleFloat`), accepts camelCase or kebab-case property names, supports array-based multi-property retrieval as of jQuery 1.9, and supports CSS custom properties as of jQuery 3.2. Complementing `.css()` are the class methods (`.addClass()`, `.removeClass()`, `.toggleClass()`, `.hasClass()`), which apply stylesheet rules without inline-style pollution, and `.cssHooks`, which allow overriding how particular CSS properties are retrieved or set. CSS transitions and animations are typically coordinated by toggling classes and listening for `transitionend`/`animationend` events, because jQuery's `.animate()` does not participate in CSS transition promises.

**Beginner-Friendly Explanation:**
CSS manipulation is how you change how a page looks after it has loaded. jQuery's `.css()` method lets you ask "what color is this element right now?" or say "make it blue." The class methods let you add or remove style labels (classes) so that CSS rules in your stylesheet take effect. You can also change CSS variables (custom properties), trigger smooth transitions, and coordinate animations — all from JavaScript.

### Key Characteristics

- **Getter/Setter Duality:** `.css()` with one argument reads the computed style of the first element; with a value (or object) it writes inline styles to all elements.
- **Computed vs. Inline:** Getters return computed styles (the final rendered value after all stylesheets and inline styles); setters write inline styles, which override stylesheet rules.
- **Normalization:** jQuery translates vendor-prefixed and alternate property names (`float`, `cssFloat`, `styleFloat`) and accepts both `background-color` and `backgroundColor`.
- **Class Preference:** jQuery documentation recommends using `.addClass()`/`.removeClass()`/`.toggleClass()` over `.css()` for production styling because classes keep styles in CSS files and are faster.
- **CSS Custom Properties:** `.css()` supports custom properties (`--my-var`) as of jQuery 3.2, but setting them on `:root` may require native `style.setProperty()`.
- **Transition Coordination:** jQuery cannot directly await CSS transitions; use `transitionend`/`animationend` events or a helper plugin to run code after a CSS-driven effect completes.
- **Performance Threshold:** For more than 20 elements, injecting a `<style>` tag is roughly 60% faster than setting `.css()` on each element.

### Prerequisites

- Basic knowledge of HTML and CSS (selectors, properties, the box model)
- Familiarity with jQuery selectors and chainability
- Understanding of the difference between inline styles and stylesheet rules
- Awareness of computed styles and the CSS Object Model (CSSOM)
- Basic knowledge of CSS transitions and animations

### Related Programming Areas

- **DOM Manipulation:** CSS manipulation is a subset of the broader manipulation category.
- **CSS Classes:** Class methods are the preferred way to apply stylesheet-driven styling.
- **CSS Custom Properties:** Variables enable theme switching and dynamic design tokens.
- **Web Animations:** CSS transitions and keyframe animations are coordinated with JavaScript events.
- **Performance Optimization:** Choosing between `.css()`, classes, and stylesheet injection affects rendering performance.

### Core Concepts / Features

1. `.css()`
2. Getting CSS
3. Setting CSS
4. Multiple CSS Properties
5. CSS Object
6. Computed Styles
7. Inline Styles
8. CSS Classes
9. CSS Variables
10. CSS Transitions
11. CSS Animations
12. CSS Performance
13. CSS Manipulation Best Practices

---

## 1. `.css()`

### Definitions

**Core Definition:**
`.css()` gets the computed value of a CSS property for the first matched element or sets one or more CSS properties on every matched element.

**Technical Definition:**
`.css(propertyName)` returns the computed style value (a string) of the specified property for the first element in the set, using `getComputedStyle()` in standards-based browsers. `.css(propertyName, value)` and `.css(propertiesObject)` set inline styles on every matched element, returning the jQuery object for chaining. `.css(propertyNamesArray)` (jQuery 1.9+) returns an object of property-value pairs. `.css("--custom-property")` retrieves CSS custom properties as of jQuery 3.2, but the property name must be provided as-is (no camelCasing).

**Beginner-Friendly Explanation:**
`.css()` is the main tool for reading and changing styles with jQuery. Read with one argument; set with a value or an object. It works for colors, sizes, fonts, and most CSS properties.

### Purposes

- To read the current rendered style of an element for conditional logic or calculation.
- To apply inline styles dynamically when a class is not practical.
- To set multiple CSS properties in a single, chainable call.
- To retrieve multiple computed style values as an object for batch processing.
- To read or write CSS custom properties in supported jQuery versions.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
// Getter — single property (first element)
var value = $(selector).css("propertyName");

// Getter — multiple properties (object return, jQuery 1.9+)
var styles = $(selector).css(["property1", "property2"]);

// Setter — single property (all elements)
$(selector).css("propertyName", "value");

// Setter — multiple properties (all elements)
$(selector).css({ "property1": "value1", "property2": "value2" });

// Callback setter
$(selector).css("propertyName", function(index, oldValue) {
    return newValue;
});
```

**Component Breakdown:**

| Form | Returns | Scope |
|------|---------|-------|
| `.css(name)` | String (computed value) | First element only. |
| `.css([names])` | Object of property-value pairs | First element only. |
| `.css(name, value)` | jQuery object | All elements (setter). |
| `.css(object)` | jQuery object | All elements (setter). |
| `.css(name, fn)` | jQuery object | All elements (computed setter). |

**Syntax Rules:**

1. Property names may be camelCase (`backgroundColor`) or kebab-case (`background-color`); jQuery normalizes both.
2. The getter returns computed values, which may differ from authored values (e.g., `#333` becomes `rgb(51, 51, 51)`).
3. Shorthand property retrieval (e.g., `margin`, `border`) is not guaranteed across browsers; query longhand properties (e.g., `borderTopWidth`) for consistency.
4. The element must be connected to the DOM when calling `.css()`; detached elements may cause errors.
5. For CSS custom properties, pass the name exactly as written (`"--my-var"`); camelCasing will not work.

**Constraints and Limitations:**

- Setters write inline styles, which override stylesheet rules and are discouraged for production styling.
- Getter values are computed, not authored; do not compare them to authored strings without normalization.
- `.css()` is slower than class manipulation for large collections; for more than 20 elements, consider a stylesheet injection.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading and Setting CSS with `.css()`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.css() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 100px; color: #333; padding: 10px; }
    </style>
</head>
<body>
    <div class="box" id="target">Box</div>

    <script>
        // Step 1: Get a single computed property
        console.log("Color: " + $("#target").css("color"));
        // rgb(51, 51, 51) — computed, not authored #333

        // Step 2: Get multiple properties as an object (jQuery 1.9+)
        var styles = $("#target").css(["width", "padding-top", "font-size"]);
        console.log("Width: " + styles.width);          // 100px
        console.log("Padding: " + styles["padding-top"]); // 10px

        // Step 3: Set a single property
        $("#target").css("background-color", "#e0e0e0");
        console.log("Bg: " + $("#target").css("background-color"));
        // rgb(224, 224, 224)

        // Step 4: Set multiple properties with an object
        $("#target").css({
            "border": "2px solid blue",
            "font-weight": "bold"
        });
        console.log("Border: " + $("#target").css("border-top-width")); // 2px
    </script>
</body>
</html>
```

**Expected Output:**
```
Color: rgb(51, 51, 51)
Width: 100px
Padding: 10px
Bg: rgb(224, 224, 224)
Border: 2px
```

**Why This Code Produces That Result:**
The getter returns the computed color `rgb(51, 51, 51)`, not the authored `#333`. The array getter returns an object with pixel values for dimensions. The setter applies inline styles that override the stylesheet. The final read confirms the border width.

### Real-World Cases

- **Theme Reading:** Reading computed background colors to determine the active theme.
- **Dynamic Sizing:** Setting widths or heights based on container measurements.
- **Debugging:** Reading computed styles to verify CSS application.
- **CSS Variable Fallback:** Reading custom properties for feature detection.

---

## 2. Getting CSS

### Definitions

**Core Definition:**
Getting CSS means reading the computed value of one or more CSS properties from the first matched element using `.css()`.

**Technical Definition:**
The getter form of `.css()` returns the computed style value resolved by the browser's CSSOM. Computed values are the result of applying all applicable stylesheets, inline styles, inheritance, and browser defaults, resolved to absolute units where possible (colors as `rgb()`/`rgba()`, dimensions in pixels). The getter always operates on the first element in the set.

**Beginner-Friendly Explanation:**
Getting CSS is like asking "what does this element actually look like right now?" The answer is the final rendered value, not just what is written in one stylesheet.

### Purposes

- To read the final rendered style for conditional logic.
- To verify that stylesheets and classes are producing the expected appearance.
- To retrieve dimensions or colors for canvas rendering or calculations.
- To inspect computed values for debugging or testing.

### Syntax Rules and Structure

```javascript
var value = $(selector).css("propertyName");
var styles = $(selector).css(["property1", "property2"]);
```

**Component Breakdown:**

| Form | Returns |
|------|---------|
| `.css(name)` | String (computed value). |
| `.css([names])` | Object of property-value pairs. |

**Syntax Rules:**

1. The getter returns only the first element's computed style.
2. Colors are returned as `rgb()` or `rgba()` strings.
3. Dimensions are almost always returned in pixels.
4. Shorthand properties may return inconsistent strings; query longhand properties.

**Constraints and Limitations:**

- Computed values may differ from authored values due to inheritance and normalization.
- Detached elements may cause errors when calling `.css()`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading Computed Values**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Getting CSS</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 50%; color: #00f; font-size: 14px; }
    </style>
</head>
<body>
    <div class="box">Box</div>

    <script>
        // Step 1: Read computed color
        console.log("Color: " + $(".box").css("color")); // rgb(0, 0, 255)

        // Step 2: Read computed width (percentage resolved to pixels)
        console.log("Width: " + $(".box").css("width")); // e.g., 400px

        // Step 3: Read multiple properties
        var s = $(".box").css(["font-size", "width"]);
        console.log("Font: " + s["font-size"]); // 14px
        console.log("Width: " + s.width);       // resolved pixel value
    </script>
</body>
</html>
```

**Expected Output:**
```
Color: rgb(0, 0, 255)
Width: [resolved pixel value]
Font: 14px
Width: [resolved pixel value]
```

**Why This Code Produces That Result:**
The authored color `#00f` is returned as `rgb(0, 0, 255)`. The percentage width is resolved to pixels because computed styles return used values. The font size is returned in pixels. This demonstrates the computed-style nature of `.css()` getters.

### Real-World Cases

- **Theme Detection:** Reading `background-color` to determine light/dark mode.
- **Animation Planning:** Reading current dimensions before animating.
- **Canvas Rendering:** Reading computed colors to draw matching UI elements.
- **Responsive Logic:** Reading resolved widths to adjust layout behavior.

---

## 3. Setting CSS

### Definitions

**Core Definition:**
Setting CSS means applying one or more CSS property-value pairs as inline styles on every matched element using `.css()`.

**Technical Definition:**
`.css(propertyName, value)` and `.css(propertiesObject)` write to each element's inline `style` attribute. Inline styles have higher specificity than stylesheet rules (short of `!important`), so they override classes and element selectors. Setters return the jQuery object, enabling chaining.

**Beginner-Friendly Explanation:**
Setting CSS is like writing style rules directly on the element. The change takes effect immediately and overrides styles from external stylesheets.

### Purposes

- To apply dynamic styles based on calculations or user input.
- To update visual state in response to events.
- To set dimensions, visibility, or colors programmatically.
- To override stylesheet rules when necessary.

### Syntax Rules and Structure

```javascript
// Single property
$(selector).css("color", "red");

// Multiple properties (object)
$(selector).css({ "color": "red", "font-size": "16px" });

// Callback setter
$(selector).css("width", function(index, oldWidth) {
    return parseFloat(oldWidth) * 2 + "px";
});
```

**Component Breakdown:**

| Form | Effect |
|------|--------|
| `.css(name, value)` | Sets one inline property on all elements. |
| `.css(object)` | Sets multiple inline properties on all elements. |
| `.css(name, fn)` | Computes value per element. |

**Syntax Rules:**

1. Numeric values for dimension properties may be automatically suffixed with `px` (prior to jQuery 4.0; check `jQuery.cssNumber`).
2. The setter affects every element in the collection.
3. Property names may be camelCase or kebab-case.

**Constraints and Limitations:**

- Inline styles override stylesheets, making maintenance harder.
- `.css()` as a setter is slower than class manipulation for large collections.
- Does not animate; use CSS transitions or `.animate()` for smooth changes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Setting Styles Dynamically**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Setting CSS</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="panel" style="padding: 10px;">Panel</div>

    <script>
        // Step 1: Set a single property
        $("#panel").css("background-color", "#e0e0e0");
        console.log("Bg: " + $("#panel").css("background-color"));

        // Step 2: Set multiple properties
        $("#panel").css({
            "border": "2px solid #333",
            "color": "#0066cc"
        });
        console.log("Border: " + $("#panel").css("border-top-width"));
        console.log("Color: " + $("#panel").css("color"));

        // Step 3: Set width via callback
        $("#panel").css("width", function(i, old) {
            return parseFloat(old || 100) * 1.5 + "px";
        });
        console.log("Width: " + $("#panel").css("width"));
    </script>
</body>
</html>
```

**Expected Output:**
```
Bg: rgb(224, 224, 224)
Border: 2px
Color: rgb(0, 102, 204)
Width: [computed value]
```

**Why This Code Produces That Result:**
The single and object setters apply inline styles immediately. The callback setter reads the current width and multiplies it, demonstrating per-element computed setting.

### Real-World Cases

- **Progress Bars:** Setting `width` based on percentage completion.
- **Highlighting:** Setting `background-color` temporarily.
- **Visibility Toggles:** Setting `display: none` or `display: block`.
- **Dynamic Layouts:** Setting `margin` or `padding` based on container size.

---

## 4. Multiple CSS Properties

### Definitions

**Core Definition:**
Multiple CSS properties refers to setting or getting several CSS properties in a single `.css()` call, using an object (setter) or an array (getter).

**Technical Definition:**
As of jQuery 1.9, passing an array of property names to `.css()` returns an object of property-value pairs. Passing an object to `.css()` sets all specified properties on every matched element. This reduces method calls and chain length.

**Beginner-Friendly Explanation:**
Instead of writing `.css("color", "red").css("background", "yellow")`, you can write `.css({ color: "red", background: "yellow" })`. It is faster and cleaner.

### Purposes

- To set several properties without multiple method calls.
- To retrieve several computed values as a single object.
- To improve readability and reduce code duplication.
- To batch style changes for performance.

### Syntax Rules and Structure

```javascript
// Set multiple properties
$(selector).css({
    "color": "red",
    "background-color": "yellow",
    "font-size": "16px"
});

// Get multiple properties (jQuery 1.9+)
var styles = $(selector).css(["color", "background-color", "font-size"]);
```

**Component Breakdown:**

| Form | Returns | Effect |
|------|---------|--------|
| `.css(object)` | jQuery object | Sets all properties. |
| `.css(array)` | Object | Gets all requested computed values. |

**Syntax Rules:**

1. Object keys may be camelCase or kebab-case.
2. Array getters return an object whose keys match the requested property names.
3. Setters return `this` for chaining.

**Constraints and Limitations:**

- Array getters return values only for the first element.
- Object setters write inline styles, with the same specificity implications as single-property setters.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Batch Setting and Getting**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Multiple Properties</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box">Box</div>

    <script>
        // Step 1: Set multiple properties in one call
        $("#box").css({
            "width": "200px",
            "height": "100px",
            "background-color": "#d0e0ff",
            "border": "1px solid #3366cc"
        });

        // Step 2: Get multiple properties in one call
        var s = $("#box").css(["width", "height", "background-color"]);
        console.log("Width: " + s.width);               // 200px
        console.log("Height: " + s.height);             // 100px
        console.log("Bg: " + s["background-color"]);    // rgb(208, 224, 255)
    </script>
</body>
</html>
```

**Expected Output:**
```
Width: 200px
Height: 100px
Bg: rgb(208, 224, 255)
```

**Why This Code Produces That Result:**
The object setter applies all four properties in one call. The array getter returns an object with computed values for the requested properties. This is more efficient than multiple `.css()` calls.

### Real-World Cases

- **Card Layouts:** Setting width, height, background, and border together.
- **Theme Application:** Applying multiple color and font properties at once.
- **Animation Setup:** Setting initial dimensions and positions in one call.

---

## 5. CSS Object

### Definitions

**Core Definition:**
A CSS object is a plain JavaScript object whose keys are CSS property names and whose values are the corresponding CSS values, used with `.css()` for bulk setting or returned by array getters.

**Technical Definition:**
`.css(propertiesObject)` iterates over the object's enumerable properties and sets each as an inline style. Keys may be camelCase (`backgroundColor`) or kebab-case (`background-color`). When an array is passed to `.css()`, the returned object uses the requested property names as keys. CSS objects are also used internally by `.animate()` and `jQuery.cssHooks`.

**Beginner-Friendly Explanation:**
A CSS object is a convenient way to group style rules in JavaScript, like a mini stylesheet written as an object.

### Purposes

- To define style sets once and reuse them across elements.
- To build dynamic style objects based on application state.
- To pass style configurations to plugins or animation methods.
- To read multiple computed values as a structured object.

### Syntax Rules and Structure

```javascript
var styles = {
    "color": "red",
    "background-color": "yellow",
    "font-size": "16px"
};

$(selector).css(styles);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Object keys | CSS property names (camelCase or kebab-case). |
| Object values | CSS values (strings or numbers). |

**Syntax Rules:**

1. The object is processed synchronously for each element.
2. Values that are numbers may be converted to pixels for dimension properties.
3. The same object can be reused for multiple selections.

**Constraints and Limitations:**

- Objects are shallow; nested objects are not interpreted as nested CSS.
- Property order in the object does not affect the result (except for shorthand overlaps).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reusable CSS Object**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Object</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div class="card">Card 1</div>
    <div class="card">Card 2</div>

    <script>
        // Step 1: Define a reusable style object
        var cardStyles = {
            "padding": "10px",
            "border": "1px solid #ccc",
            "background-color": "#f9f9f9",
            "margin-bottom": "10px"
        };

        // Step 2: Apply the same object to multiple selections
        $(".card").css(cardStyles);
        console.log("Cards: " + $(".card").length); // 2
        console.log("Bg: " + $(".card").first().css("background-color")); // rgb(249, 249, 249)
    </script>
</body>
</html>
```

**Expected Output:**
```
Cards: 2
Bg: rgb(249, 249, 249)
```

**Why This Code Produces That Result:**
The `cardStyles` object is applied to both cards in a single `.css()` call, demonstrating reuse and bulk application.

### Real-World Cases

- **Component Styling:** Defining a style object for a reusable UI component.
- **Theme Objects:** Switching between different CSS object themes.
- **Plugin Configuration:** Passing a style object as a plugin option.

---

## 6. Computed Styles

### Definitions

**Core Definition:**
Computed styles are the final, resolved style values that the browser calculates after applying all stylesheets, inline styles, inheritance, and defaults.

**Technical Definition:**
The browser's `getComputedStyle()` method returns a `CSSStyleDeclaration` object containing the resolved values of all CSS properties for an element. jQuery's `.css()` getter uses this method (with legacy fallbacks) and returns the computed value as a string. Computed values are normalized: colors become `rgb()`/`rgba()`, dimensions become pixels, and relative units are resolved.

**Beginner-Friendly Explanation:**
Computed styles are what the element actually looks like, not just what one stylesheet says. If multiple rules apply, the browser combines them, and `.css()` tells you the final result.

### Purposes

- To read the actual rendered value for calculations or conditional logic.
- To verify that stylesheets and classes are producing the intended appearance.
- To retrieve normalized values for canvas or SVG rendering.
- To avoid parsing authored values that may be in different units.

### Syntax Rules and Structure

```javascript
var color = $(selector).css("color");       // rgb(...)
var width = $(selector).css("width");       // pixels
var styles = $(selector).css(["color", "width"]);
```

**Component Breakdown:**

| Aspect | Computed Value |
|--------|----------------|
| Colors | `rgb()` or `rgba()` |
| Dimensions | Pixels (usually) |
| Shorthand | May be browser-dependent |

**Syntax Rules:**

1. Computed values may differ from authored values (`#333` → `rgb(51, 51, 51)`).
2. Relative units (`em`, `%`) are resolved to pixels for dimensions.
3. Shorthand properties may return inconsistent strings; use longhand properties.

**Constraints and Limitations:**

- Computed values reflect the current rendering state; they may be affected by zoom or browser extensions.
- The getter returns values only for the first element.
- Detached elements may not return computed styles.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Computed vs. Authored Values**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Computed Styles</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { color: #333; width: 50%; font-size: 1.2em; }
    </style>
</head>
<body>
    <div class="box">Box</div>

    <script>
        // Step 1: Computed color (authored as #333)
        console.log("Color: " + $(".box").css("color")); // rgb(51, 51, 51)

        // Step 2: Computed width (authored as 50%)
        console.log("Width: " + $(".box").css("width")); // resolved pixel value

        // Step 3: Computed font-size (authored as 1.2em)
        console.log("Font: " + $(".box").css("font-size")); // resolved pixel value
    </script>
</body>
</html>
```

**Expected Output:**
```
Color: rgb(51, 51, 51)
Width: [resolved pixel value]
Font: [resolved pixel value]
```

**Why This Code Produces That Result:**
The browser resolves `#333` to `rgb(51, 51, 51)`, the percentage width to pixels, and the `em` font size to pixels. This demonstrates that `.css()` returns computed, not authored, values.

### Real-World Cases

- **Responsive Breakpoints:** Reading computed widths to detect layout mode.
- **Color Matching:** Reading computed colors to synchronize canvas drawings.
- **Testing:** Asserting computed values in QUnit tests.

---

## 7. Inline Styles

### Definitions

**Core Definition:**
Inline styles are CSS property-value pairs written directly on an element's `style` attribute, applied by `.css()` setters and having higher specificity than stylesheet rules.

**Technical Definition:**
Setting `.css()` writes to `element.style`, the inline style declaration. Inline styles override author stylesheet rules unless the stylesheet uses `!important`. They are useful for dynamic, element-specific values but are discouraged for production styling because they separate style from CSS files and increase maintenance burden.

**Beginner-Friendly Explanation:**
Inline styles are styles written right on the HTML tag. They win over stylesheet rules, so they are good for one-off dynamic changes but messy if overused.

### Purposes

- To apply one-off dynamic values that cannot be pre-defined in a class.
- To override stylesheet rules temporarily.
- To set computed dimensions or positions from JavaScript.
- To prototype styles quickly during development.

### Syntax Rules and Structure

```javascript
$(selector).css("color", "red");
// Equivalent to: element.style.color = "red";
```

**Component Breakdown:**

| Aspect | Inline Style |
|--------|-------------|
| Location | `element.style` |
| Specificity | Higher than stylesheet rules (except `!important`) |
| Persistence | Until overwritten or removed |

**Syntax Rules:**

1. Inline styles are set per element.
2. They override classes and element selectors.
3. They can be removed with `.css("property", "")` or `.removeAttr("style")`.

**Constraints and Limitations:**

- Inline styles are harder to maintain than classes.
- They increase HTML size and can cause specificity conflicts.
- `.css()` inline setters are slower than class changes for large collections.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Inline Style Override**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Inline Styles</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { color: blue; }
    </style>
</head>
<body>
    <div class="box" id="target">Box</div>

    <script>
        // Step 1: Stylesheet sets color to blue
        console.log("Before: " + $("#target").css("color")); // rgb(0, 0, 255)

        // Step 2: Inline style overrides to red
        $("#target").css("color", "red");
        console.log("After: " + $("#target").css("color")); // rgb(255, 0, 0)

        // Step 3: Remove inline style
        $("#target").css("color", "");
        console.log("Removed: " + $("#target").css("color")); // rgb(0, 0, 255) — back to stylesheet
    </script>
</body>
</html>
```

**Expected Output:**
```
Before: rgb(0, 0, 255)
After: rgb(255, 0, 0)
Removed: rgb(0, 0, 255)
```

**Why This Code Produces That Result:**
The inline style overrides the stylesheet rule. Removing the inline value (empty string) restores the stylesheet value, demonstrating the specificity relationship.

### Real-World Cases

- **Drag-and-Drop:** Setting `left`/`top` inline during dragging.
- **Progress Indicators:** Setting `width` inline based on percentage.
- **Temporary Highlights:** Setting a background color briefly, then removing it.

---

## 8. CSS Classes

### Definitions

**Core Definition:**
CSS classes are reusable style identifiers that are added, removed, toggled, and checked using jQuery's class methods, applying stylesheet rules without inline-style pollution.

**Technical Definition:**
`.addClass()`, `.removeClass()`, `.toggleClass()`, and `.hasClass()` manipulate the `class` attribute (abstracting `classList`). Classes connect elements to stylesheet rules, keeping style definitions in CSS files. jQuery documentation recommends class methods over `.css()` for production styling because they are faster and more maintainable.

**Beginner-Friendly Explanation:**
Classes are labels that tell the browser which stylesheet rules apply. Instead of writing styles directly on the element, you add a class, and the CSS file handles the appearance.

### Purposes

- To apply predefined styles without inline CSS.
- To toggle visual states (active, selected, error, success).
- To trigger CSS transitions and animations.
- To keep styling in CSS files for maintainability.

### Syntax Rules and Structure

```javascript
$(selector).addClass("className");
$(selector).removeClass("className");
$(selector).toggleClass("className");
var has = $(selector).hasClass("className");
```

**Component Breakdown:**

| Method | Effect |
|--------|--------|
| `.addClass("a")` | Adds class `a` if absent. |
| `.removeClass("a")` | Removes class `a` if present. |
| `.toggleClass("a")` | Adds if absent, removes if present. |
| `.hasClass("a")` | Returns `true` if any element has `a`. |

**Syntax Rules:**

1. Multiple classes can be space-separated.
2. `.removeClass()` with no arguments removes all classes.
3. `.toggleClass("a", state)` forces add (`true`) or remove (`false`).
4. Class names are case-sensitive.

**Constraints and Limitations:**

- Classes must be defined in a stylesheet to have a visual effect.
- `.hasClass()` checks the collection, not individual elements.
- Toggling does not animate; combine with CSS transitions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Class-Based Styling**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Classes</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .active { background: lightgreen; }
        .error { border: 1px solid red; }
    </style>
</head>
<body>
    <div id="box">Box</div>
    <button id="toggle">Toggle Active</button>

    <script>
        // Step 1: Add a class
        $("#box").addClass("active");
        console.log("Active: " + $("#box").hasClass("active")); // true
        console.log("Bg: " + $("#box").css("background-color")); // rgb(144, 238, 144)

        // Step 2: Toggle the class
        $("#toggle").on("click", function() {
            $("#box").toggleClass("active");
        });

        // Step 3: Check class before adding error
        if (!$("#box").hasClass("error")) {
            $("#box").addClass("error");
        }
    </script>
</body>
</html>
```

**Expected Output:**
```
Active: true
Bg: rgb(144, 238, 144)
```

**Why This Code Produces That Result:**
`.addClass("active")` applies the `.active` stylesheet rule. `.hasClass()` verifies it. Toggling alternates the class, and the conditional prevents duplicate `.error` additions.

### Real-World Cases

- **Active Navigation:** Adding `.active` to the current menu item.
- **Form Validation:** Adding `.error` to invalid fields.
- **Theme Switching:** Adding `.dark-theme` to the body.
- **Reveal Animations:** Adding `.visible` to trigger CSS transitions.

---

## 9. CSS Variables

### Definitions

**Core Definition:**
CSS variables (custom properties) are user-defined property names prefixed with `--` that store values reusable throughout a stylesheet and accessible from JavaScript.

**Technical Definition:**
CSS custom properties are defined with `--name: value` and referenced with `var(--name)`. jQuery `.css()` supports reading and writing custom properties as of jQuery 3.2, but the property name must be passed as-is (no camelCasing). Setting custom properties on `:root` via `.css()` may not work consistently; the reliable approach is native `element.style.setProperty("--name", value)`.

**Beginner-Friendly Explanation:**
CSS variables are like named placeholders for values. You define `--main-color: blue` once and use `var(--main-color)` everywhere. Changing the variable updates all usages. jQuery can read them, but for writing, the native `setProperty` method is safer.

### Purposes

- To define reusable design tokens (colors, spacing, fonts) in one place.
- To enable theme switching by changing a few variables.
- To allow JavaScript to read and modify CSS variables dynamically.
- To reduce duplication across stylesheets.

### Syntax Rules and Structure

```javascript
// Read a custom property (jQuery 3.2+)
var value = $(selector).css("--my-var");

// Set a custom property (native approach — recommended)
$(selector)[0].style.setProperty("--my-var", "red");

// Read via native API
var nativeVal = $(selector)[0].style.getPropertyValue("--my-var");
```

**Component Breakdown:**

| Approach | Read | Write |
|----------|------|-------|
| jQuery `.css()` | `.css("--var")` | `.css("--var", value)` (may not work on `:root`) |
| Native `style` | `style.getPropertyValue("--var")` | `style.setProperty("--var", value)` |

**Syntax Rules:**

1. Custom property names must be passed exactly as written (`"--my-var"`).
2. `.css()` camelCasing does not apply to custom properties.
3. For reliable writing, use `element.style.setProperty()`.
4. Custom properties inherit through the DOM tree.

**Constraints and Limitations:**

- jQuery `.css()` support for custom properties is version-dependent (3.2+).
- Setting on `:root` via `.css()` may not work; use native `setProperty`.
- Older browsers may not support CSS variables at all.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Reading and Writing CSS Variables**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Variables</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        :root { --main-color: blue; }
        .box { color: var(--main-color); }
    </style>
</head>
<body>
    <div class="box">Box</div>

    <script>
        // Step 1: Read a CSS variable via jQuery
        console.log("jQuery read: " + $(":root").css("--main-color")); // blue

        // Step 2: Write using native setProperty (reliable)
        $(":root")[0].style.setProperty("--main-color", "red");
        console.log("After write: " + $(".box").css("color")); // rgb(255, 0, 0)

        // Step 3: Read via native API
        console.log("Native read: " + $(":root")[0].style.getPropertyValue("--main-color")); // red
    </script>
</body>
</html>
```

**Expected Output:**
```
jQuery read: blue
After write: rgb(255, 0, 0)
Native read: red
```

**Why This Code Produces That Result:**
The variable is defined on `:root` and inherited by `.box`. Reading via `.css()` returns `"blue"`. Writing via native `setProperty` changes the variable, and the computed color updates to red. Native reading confirms the new value.

### Real-World Cases

- **Theme Switching:** Changing `--primary-color` to switch themes.
- **Dynamic Spacing:** Adjusting `--gutter-width` based on viewport.
- **Design Tokens:** Centralizing brand colors and fonts.

---

## 10. CSS Transitions

### Definitions

**Core Definition:**
CSS transitions are smooth interpolations between two CSS property values, defined in stylesheets and triggered by class or style changes.

**Technical Definition:**
A CSS transition is defined with `transition: property duration timing-function delay`. When a property changes (e.g., via `.addClass()` or `.css()`), the browser animates from the old value to the new value. jQuery cannot directly await a CSS transition; use the `transitionend` event (and vendor-prefixed variants) to run code after the transition completes. jQuery's `.animate()` does not participate in CSS transition promises.

**Beginner-Friendly Explanation:**
CSS transitions make style changes happen smoothly instead of instantly. You define the transition in CSS, then use jQuery to change the value or toggle a class. To run code when the animation finishes, listen for the `transitionend` event.

### Purposes

- To create smooth visual changes without JavaScript animation loops.
- To separate animation logic (CSS) from interaction logic (JavaScript).
- To improve performance by leveraging browser-native interpolation.
- To coordinate post-transition actions with `transitionend`.

### Syntax Rules and Structure

**CSS:**

```css
.box {
    transition: width 0.3s ease, background-color 0.3s ease;
}
.box.wide {
    width: 300px;
    background-color: lightblue;
}
```

**JavaScript:**

```javascript
$(".box").addClass("wide");

$(".box").one("transitionend", function(event) {
    // Runs after the transition completes
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `transition` | CSS shorthand for property, duration, timing, delay. |
| Class toggle | Triggers the transition by changing values. |
| `transitionend` | Event fired when the transition completes. |

**Syntax Rules:**

1. Transitions require a change in the property value (from a stylesheet rule, class, or inline style).
2. Use `.one()` for one-time transition-end handlers.
3. Bind to `transitionend` (and vendor-prefixed variants if supporting older browsers).
4. Multiple properties may fire multiple `transitionend` events; filter by `event.originalEvent.propertyName`.

**Constraints and Limitations:**

- jQuery promises do not resolve on CSS transitions.
- Transitions do not run on properties that are not animatable (e.g., `display`).
- Transition events bubble; use `event.target` to identify the element.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Class-Triggered Transition with Completion Handler**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Transitions</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box {
            width: 100px; height: 100px; background: #eee;
            transition: width 0.5s ease, background-color 0.5s ease;
        }
        .box.wide { width: 300px; background: lightblue; }
    </style>
</head>
<body>
    <div class="box">Box</div>
    <button id="grow">Grow</button>

    <script>
        // Step 1: Trigger the transition by toggling a class
        $("#grow").on("click", function() {
            $(".box").toggleClass("wide");

            // Step 2: Run code after the transition ends
            $(".box").one("transitionend", function(event) {
                console.log("Transition finished: " + event.originalEvent.propertyName);
            });
        });
    </script>
</body>
</html>
```

**Expected Output (after clicking Grow):**
```
Transition finished: width
Transition finished: background-color
```

**Why This Code Produces That Result:**
Toggling `.wide` changes `width` and `background-color`, both of which have transitions defined. The browser animates the changes and fires `transitionend` for each property. The `.one()` handler logs the property name. This demonstrates the standard pattern of class-triggered CSS transitions with JavaScript completion handling.

### Real-World Cases

- **Hover Effects:** Transitioning color and shadow on hover.
- **Accordion Panels:** Transitioning height when opening/closing.
- **Modal Fade-In:** Transitioning opacity and transform.
- **Responsive Layouts:** Transitioning widths when the viewport changes.

---

## 11. CSS Animations

### Definitions

**Core Definition:**
CSS animations are keyframe-based effects defined with `@keyframes` and applied via the `animation` property, triggered and coordinated with jQuery.

**Technical Definition:**
A CSS animation defines multiple keyframes (0%, 50%, 100%) with property values and applies them using `animation: name duration timing-function iteration-count direction`. jQuery can add or remove animation classes and listen for `animationend` to run completion code. Like transitions, CSS animations are not integrated with jQuery's animation queue or promises.

**Beginner-Friendly Explanation:**
CSS animations are more powerful than transitions because they can have multiple steps (keyframes). You define the animation in CSS, apply it by adding a class, and listen for `animationend` when it finishes.

### Purposes

- To create complex, multi-step animations without JavaScript.
- To trigger animations via class changes.
- To run code after an animation completes with `animationend`.
- To separate animation design (CSS) from interaction logic (JavaScript).

### Syntax Rules and Structure

**CSS:**

```css
@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.1); }
    100% { transform: scale(1); }
}
.pulse { animation: pulse 0.6s ease; }
```

**JavaScript:**

```javascript
$(".box").addClass("pulse");
$(".box").one("animationend", function() {
    $(".box").removeClass("pulse");
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@keyframes` | Defines the animation steps. |
| `animation` | Applies the animation with duration and timing. |
| `animationend` | Event fired when the animation completes. |

**Syntax Rules:**

1. Add the animation class to start the animation.
2. Use `.one("animationend", handler)` to run code once.
3. Remove the class if the animation should be re-triggerable.
4. Vendor-prefixed events may be needed for older browsers.

**Constraints and Limitations:**

- CSS animations do not integrate with jQuery `.animate()` or promises.
- Re-triggering often requires removing and re-adding the class (with a reflow between).
- Animation events bubble; use `event.target` for specificity.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Keyframe Animation with Completion Cleanup**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Animations</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }
        .pulse { animation: pulse 0.6s ease; }
    </style>
</head>
<body>
    <div class="box">Box</div>
    <button id="animate">Animate</button>

    <script>
        $("#animate").on("click", function() {
            // Step 1: Add the animation class
            $(".box").addClass("pulse");

            // Step 2: Listen for animation end
            $(".box").one("animationend", function() {
                console.log("Animation finished");
                // Step 3: Remove the class so it can be re-triggered
                $(this).removeClass("pulse");
            });
        });
    </script>
</body>
</html>
```

**Expected Output (after clicking Animate):**
```
Animation finished
```

**Why This Code Produces That Result:**
Adding `.pulse` starts the keyframe animation. The `animationend` event fires when the animation completes, and the handler removes the class, allowing the animation to be triggered again on the next click.

### Real-World Cases

- **Notification Shakes:** Animating an error message shake.
- **Loading Spinners:** Keyframe rotation animations.
- **Attention Pulses:** Pulsing a button to draw attention.
- **Page Transitions:** Animating view changes.

---

## 12. CSS Performance

### Definitions

**Core Definition:**
CSS performance refers to the efficiency with which style changes are applied and rendered, including the choice between inline styles, classes, and stylesheet injection.

**Technical Definition:**
Setting inline styles via `.css()` on many elements causes repeated style recalculation and can trigger layout thrashing. Class manipulation is faster because the browser can optimize class-based style resolution. For more than 20 elements, injecting a `<style>` tag is reported to be nearly 60% faster than setting `.css()` on each element. Batch DOM reads and writes to avoid layout thrashing.

**Beginner-Friendly Explanation:**
Changing styles can be slow if you do it one element at a time. Using classes is faster. If you need to change many elements, adding a `<style>` tag to the page is even faster.

### Purposes

- To minimize style recalculation and rendering time.
- To avoid layout thrashing when reading and writing styles.
- To choose the most efficient method for the number of elements.
- To keep animations smooth.

### Syntax Rules and Structure

**Performance Strategies:**

| Strategy | When to Use |
|----------|-------------|
| `.addClass()` / `.toggleClass()` | Few elements, predefined styles. |
| `.css()` single property | Few elements, dynamic value. |
| Stylesheet injection (`<style>`) | More than 20 elements. |
| Batch reads and writes | Avoid read/write interleaving. |

**Syntax Rules:**

1. Prefer classes over `.css()` for repeated styling.
2. For large collections, inject a `<style>` tag once.
3. Cache jQuery selections to avoid repeated DOM queries.
4. Separate read and write phases to avoid forced synchronous layouts.

**Constraints and Limitations:**

- Stylesheet injection modifies the document head and may require cleanup.
- Class changes still trigger style recalculation but are generally faster than inline styles.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Stylesheet Injection for Many Elements**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Performance</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <a href="#" class="swedberg">Link 1</a>
    <a href="#" class="swedberg">Link 2</a>
    <!-- ... 30 more links ... -->

    <script>
        // Step 1: For many elements, inject a style rule
        $("<style>")
            .prop("type", "text/css")
            .html(".swedberg { color: #0769ad; }")
            .appendTo("head");

        console.log("Color: " + $(".swedberg").first().css("color"));
        // rgb(7, 105, 173)
    </script>
</body>
</html>
```

**Expected Output:**
```
Color: rgb(7, 105, 173)
```

**Why This Code Produces That Result:**
A single `<style>` rule applies to all `.swedberg` links at once, avoiding repeated `.css()` calls. This is the recommended approach for large collections.

### Real-World Cases

- **Large Data Tables:** Styling many rows via a single stylesheet rule.
- **Dynamic Themes:** Injecting a theme stylesheet instead of styling each element.
- **Infinite Scroll:** Styling loaded items with a class or stylesheet rule.

---

## 13. CSS Manipulation Best Practices

### Definitions

**Core Definition:**
CSS manipulation best practices are guidelines for choosing between `.css()`, classes, CSS variables, and stylesheet injection to achieve performant, maintainable, and secure styling.

**Technical Definition:**
Best practices encompass preferring classes over inline `.css()` for production styling, using `.css()` only for dynamic values that cannot be predefined, batching style changes, using CSS variables for theming, using native `setProperty()` for custom properties on `:root`, coordinating CSS transitions/animations with `transitionend`/`animationend`, and avoiding CSS injection from untrusted input.

**Beginner-Friendly Explanation:**
These are the rules that keep your styling code fast, safe, and easy to maintain. They help you choose the right tool for each situation.

### Purposes

- To choose the correct styling method for each use case.
- To maximize rendering performance.
- To keep styles maintainable and separated from logic.
- To avoid security issues from CSS injection.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Prefer classes over `.css()` | `.css("color", "red")` | `.addClass("error")` |
| 2 | Use `.css()` for dynamic values | `.addClass("width-" + px)` (unbounded classes) | `.css("width", px)` |
| 3 | Batch style changes | Alternate read/write in a loop | Separate read pass, write pass |
| 4 | Inject stylesheet for many elements | `.css()` on 100 elements | `<style>` rule |
| 5 | Use `setProperty()` for CSS variables on `:root` | `.css("--var", "red")` on `:root` | `style.setProperty("--var", "red")` |
| 6 | Listen for `transitionend`/`animationend` | Assume `.css()` callback | `.one("transitionend", fn)` |
| 7 | Sanitize CSS values from untrusted input | `.css(userProp, userValue)` | Validate/sanitize before setting |

**Syntax Rules:**

1. Use `.addClass()`/`.removeClass()` for predefined styles.
2. Use `.css()` only for values computed at runtime (dimensions, positions, progress).
3. For more than 20 elements, prefer a stylesheet rule.
4. Use CSS variables for theme-able design tokens.
5. Never allow untrusted input to control CSS property names or `url()` values without validation.

**Constraints and Limitations:**

- Classes require stylesheet definitions; `.css()` works without external CSS.
- Stylesheet injection is powerful but can be harder to clean up.
- CSS variable writing on `:root` requires native APIs for reliability.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Choosing the Right Method**

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSS Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .error { border: 2px solid red; }
        :root { --progress-width: 0%; }
    </style>
</head>
<body>
    <div id="progress" style="height: 10px; background: green; width: 0;"></div>
    <input id="email" type="text">

    <script>
        // ✅ Use a class for predefined state
        $("#email").addClass("error");

        // ✅ Use .css() for dynamic computed values
        $("#progress").css("width", "75%");

        // ✅ Use native setProperty for CSS variables on :root
        document.documentElement.style.setProperty("--progress-width", "75%");

        // ✅ Use a stylesheet rule for many elements (not shown in loop)
        console.log("Progress width: " + $("#progress").css("width")); // 75%
    </script>
</body>
</html>
```

**Expected Output:**
```
Progress width: 75%
```

**Why This Code Produces That Result:**
The `.error` class applies a predefined border style. The dynamic progress width uses `.css()` because the value is computed at runtime. The CSS variable is set with native `setProperty` for reliability on `:root`. Each method is chosen according to its strengths.

### Real-World Cases

- **Form Validation:** Adding `.error` classes instead of setting inline borders.
- **Progress Indicators:** Setting `width` inline based on percentage.
- **Theme Systems:** Using CSS variables for colors and spacing.
- **Large Lists:** Injecting a stylesheet rule for repeated item styling.

---

## References

- jQuery API Documentation — `.css()` - https://api.jquery.com/css/
- jQuery API Documentation — Style Properties - https://api.jquery.com/category/manipulation/style-properties/
- jQuery Learning Center — Use Stylesheets for Changing CSS on Many Elements - https://raw.githubusercontent.com/jquery/learn.jquery.com/7b49f826153fc27f510322eb56f23f3b4743be29/pages/performance/use-stylesheets-for-changing-css.md
- Stack Overflow — Modify CSS variables / custom properties in jQuery - https://stackoverflow.com/questions/49048192/modify-css-variables-custom-properties-in-jquery
- Stack Overflow — jQuery promises only work on animations performed by jQuery, not CSS transitions - https://stackoverflow.com/revisions/04878576-a2f3-4980-a808-bbed2b89fbc2/view-source
- Treehouse Blog — Using jQuery to Detect When CSS3 Animations and Transitions End - https://blog.teamtreehouse.com/using-jquery-to-detect-when-css3-animations-and-transitions-end
- jQuery API Documentation — `jQuery.cssHooks` - https://api.jquery.com/jQuery.cssHooks/
- Stack Overflow — jQuery performance: `.css()` vs `.addClass()` - https://stackoverflow.com/questions/2341598/jquery-performance-css-or-addclass
- MDN Web Docs — CSS Custom Properties (`--*`) - https://developer.mozilla.org/en-US/docs/Web/CSS/--*
- MDN Web Docs — `transitionend` event - https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionend_event
- MDN Web Docs — `animationend` event - https://developer.mozilla.org/en-US/docs/Web/API/Element/animationend_event
- OWASP — CSS Injection - https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/11-Client-side_Testing/05-Testing_for_CSS_Injection