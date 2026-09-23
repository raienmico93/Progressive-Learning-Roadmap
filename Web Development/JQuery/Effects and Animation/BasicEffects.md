# jQuery Basic Visibility: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
jQuery Basic Visibility is a set of three core effects methods — `.show()`, `.hide()`, and `.toggle()` — that control whether matched elements are displayed on the page by manipulating the CSS `display` property.

**Technical Definition**
These methods are members of the jQuery effects suite. When called without arguments, they synchronously modify the `display` CSS property of each matched element: `.hide()` sets `display: none` and caches the element's original display value; `.show()` restores the cached display value; `.toggle()` alternates between the two based on the element's current visibility state. When provided with a duration or options object, they transition into asynchronous animation methods that animate `width`, `height`, and `opacity` simultaneously .

**Beginner-Friendly Explanation**
Think of these three methods as a light switch for your HTML elements. `.hide()` turns the light off (the element disappears and takes up no space on the page). `.show()` turns it back on (the element reappears exactly as it was). `.toggle()` is a smart switch that flips itself — if the light is on, it turns it off; if it is off, it turns it on. You can flip the switch instantly or gradually with an animation.

### Key Characteristics

- **Display Property Manipulation**: All three methods operate by changing the CSS `display` property, not `visibility` or `opacity` .
- **State Preservation**: `.hide()` saves the original `display` value in jQuery's internal data cache so `.show()` can restore it faithfully (e.g., `inline` stays `inline`, not `block`) .
- **Instant by Default**: Without a duration argument, the change happens immediately, with no animation .
- **Animation-Capable**: Providing a duration, easing string, or options object triggers an animated transition of width, height, and opacity .
- **Implicit Iteration**: All three methods apply to every element in the matched jQuery collection .
- **Chainable**: Each method returns the jQuery object, enabling method chaining .

### Prerequisites

- Basic HTML and CSS knowledge, particularly the `display` property.
- JavaScript fundamentals: functions, objects, and callbacks.
- A working jQuery setup (library included via CDN or local file).
- Familiarity with jQuery selectors and the jQuery object model.

### Related Programming Areas

- **CSS Display Property**: Understanding `block`, `inline`, `inline-block`, and `none` is essential for predicting behaviour.
- **jQuery Effects Suite**: Related methods include `.fadeIn()`, `.fadeOut()`, `.slideDown()`, and `.slideUp()` .
- **Animation Queuing**: The `queue` option and `.dequeue()` interact with these methods when animations are chained .
- **jQuery Data Cache**: `.hide()` stores state in `jQuery.data()`, which is relevant for debugging and plugin development.

### Core Concepts / Features

1. `.show()`
2. `.hide()`
3. `.toggle()`

---

## Core Concept 1: `.show()`

### Definitions

**Core Definition**
`.show()` displays matched elements by restoring their natural or previously cached `display` state.

**Technical Definition**
`.show()` is a jQuery effects method that removes the `display: none` restriction from matched elements. When called without arguments, it synchronously sets the element's `display` property to the value cached by a prior `.hide()` call, or to the browser's default display value if no cached value exists. When called with a duration, it animates `width`, `height`, and `opacity` from zero to their natural values, then restores the original `display` value .

**Beginner-Friendly Explanation**
`.show()` is the "reveal" command. If an element has been hidden, `.show()` brings it back exactly as it was — same size, same position, same display style. If you provide a number (like `500`), the element grows and fades into view over half a second instead of appearing instantly.

### Purposes

- To display elements that were previously hidden with `.hide()` or `display: none`.
- To restore an element's original display style (e.g., `inline`, `flex`, `grid`) without manually specifying it.
- To animate elements into view with a fade-and-slide effect.
- To conditionally reveal content based on user interaction (e.g., expanding a panel).
- To chain with other jQuery methods for sequenced animations.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: Instant Show (No Arguments)**
```javascript
.show()
```
- No arguments. Displays matched elements immediately, with no animation .

**Syntax 2: Show with Duration and Callback**
```javascript
.show( duration [, complete ] )
```
- `duration` (Number or String): Animation duration in milliseconds, or `"fast"` (200ms) or `"slow"` (600ms). Default is `400` .
- `complete` (Function): Optional callback fired once the animation completes, once per matched element. `this` is set to the DOM element being animated .

**Syntax 3: Show with Duration, Easing, and Callback**
```javascript
.show( duration [, easing ] [, complete ] )
```
- `easing` (String): `"swing"` (default) or `"linear"`. Added in jQuery 1.4.3 .

**Syntax 4: Show with Options Object**
```javascript
.show( options )
```
- `options` (PlainObject): A map of animation settings including `duration`, `easing`, `queue`, `complete`, `step`, `progress`, `start`, `done`, `fail`, and `always` .

**Syntax 5: Show with Boolean (Toggle Signature)**
```javascript
.show( display )
```
- Not a native `.show()` signature; the Boolean form belongs to `.toggle(display)` .

**Component Breakdown**

- `.show()` : The method call.
- `duration` : Controls animation length. Larger numbers = slower animation.
- `easing` : Controls the rate of change during the animation.
- `complete` : Executes after all animated properties reach their final values.
- `options` : A single object consolidating all animation settings.

**Syntax Rules**

1. `.show()` restores the **cached** display value from a prior `.hide()`, not always `block` .
2. If no cached value exists, `.show()` uses the browser's default display for that element type.
3. With no arguments, `.show()` is roughly equivalent to `.css("display", cachedValue)` but without manual cache management .
4. `display: none !important` in a stylesheet cannot be overridden by `.show()` .
5. When animating, `.show()` animates `width`, `height`, and `opacity` simultaneously .

**Constraints and Limitations**

- **`!important` Conflict**: `.show()` cannot override `display: none !important` in CSS. Use `.addClass()`/`.removeClass()` or `.attr("style", "display: block !important;")` as alternatives .
- **Performance**: Animating many elements simultaneously can cause performance issues, especially on low-powered devices .
- **Responsive Layouts**: If display value differs across viewport sizes, cached values may be incorrect after a resize .
- **jQuery UI Extension**: jQuery UI extends `.show()` with custom effects via `.show(effect, options, duration, complete)`. Without jQuery UI, these signatures do not work .

### Multiple Annotated Complete Code Examples

**Example 1: Instant Show**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.show() — Instant</title>
  <!-- Step 1: Load jQuery -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .box { padding: 20px; background: lightblue; margin: 10px; }
  </style>
</head>
<body>
  <!-- Step 2: Element initially hidden via inline style -->
  <div id="panel" class="box" style="display: none;">
    This panel was hidden.
  </div>

  <button id="showBtn">Show Panel</button>

  <script>
    $(function () {
      // Step 3: Bind click handler to the button
      $("#showBtn").on("click", function () {
        // Step 4: Show the panel instantly (no animation)
        $("#panel").show();
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Initially, the blue panel is invisible.
- After clicking "Show Panel", the panel appears immediately, with no animation.

**Why This Output Occurs**
The panel has `style="display: none;"`. Calling `.show()` without arguments removes that inline style and restores the element to its default display value (`block` for a `<div>`). No transition occurs because no duration was specified .

---

**Example 2: Animated Show with Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.show() — Animated with Callback</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .card { padding: 20px; background: #f0f0f0; border: 1px solid #ccc; margin: 10px; }
  </style>
</head>
<body>
  <div id="card" class="card" style="display: none;">
    Animated content here.
  </div>

  <button id="animateBtn">Reveal Card (Slow)</button>

  <script>
    $(function () {
      $("#animateBtn").on("click", function () {
        // Step 1: Show with 'slow' (600ms) and a completion callback
        $("#card").show("slow", function () {
          // Step 2: 'this' refers to the DOM element being animated
          console.log("Animation complete for:", this.id);

          // Step 3: Chain additional changes after reveal
          $(this).css("border-color", "green");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking the button causes the card to grow and fade into view over 600 milliseconds.
- After the animation finishes, the console logs `"Animation complete for: card"` and the border turns green.

**Why This Output Occurs**
Providing `"slow"` as the duration converts `.show()` into an animation method. jQuery animates the card's `width`, `height`, and `opacity` from zero to their natural values. The callback fires once per matched element after all animated properties complete, with `this` bound to the DOM element .

---

**Example 3: Show with Options Object**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.show() — Options Object</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" style="display:none; padding:15px; background:lightyellow;">
    Options-based reveal.
  </div>
  <button id="go">Show with Linear Easing</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        $("#box").show({
          duration: 800,
          easing: "linear",
          complete: function () {
            console.log("Done with linear easing.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The box expands and fades in over 800 milliseconds with a constant-rate (linear) transition.
- Console logs `"Done with linear easing."` after completion.

**Why This Output Occurs**
The options object provides `duration: 800` (milliseconds), `easing: "linear"` (constant speed), and a `complete` callback. jQuery reads these settings and applies them to the animation sequence .

### Real-World Cases

- **Accordion Panels**: Revealing the content of a collapsed accordion section with a smooth animation.
- **Modal Dialogs**: Showing a previously hidden modal after a trigger button is clicked.
- **Error Messages**: Displaying validation error messages that were pre-rendered but hidden.
- **Lazy Loading Indicators**: Showing a loading spinner before an AJAX request and hiding it afterward.

### References

- jQuery API — .show() – https://api.jquery.com/show/
- jQuery Learning Center — Intro to Effects – https://learn.jquery.com/effects/intro-to-effects/
- jQuery UI API — .show() (Extended) – https://api.jqueryui.com/1.11/show/

---

## Core Concept 2: `.hide()`

### Definitions

**Core Definition**
`.hide()` hides matched elements by setting their CSS `display` property to `none`, while caching the original display value for later restoration.

**Technical Definition**
`.hide()` is a jQuery effects method that removes matched elements from the visual flow of the document. When called without arguments, it synchronously sets `display: none` and stores the element's prior `display` value in jQuery's internal data cache. When called with a duration, it animates `width`, `height`, and `opacity` to zero, then sets `display: none` to remove the element from layout .

**Beginner-Friendly Explanation**
`.hide()` is the "disappear" command. The element vanishes and leaves no empty gap — the page closes up as if the element was never there. But jQuery remembers what the element looked like before, so when you call `.show()` later, it comes back exactly right. With a duration, the element shrinks and fades away instead of vanishing instantly.

### Purposes

- To hide elements without removing them from the DOM.
- To collapse content sections and reclaim their layout space.
- To animate elements out of view with a fade-and-shrink effect.
- To cache the original display value for faithful restoration.
- To prepare elements for later revelation via `.show()` or `.toggle()`.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: Instant Hide (No Arguments)**
```javascript
.hide()
```
- No arguments. Hides matched elements immediately, with no animation .

**Syntax 2: Hide with Duration and Callback**
```javascript
.hide( duration [, complete ] )
```
- `duration` (Number or String): Animation duration in milliseconds, or `"fast"` (200ms) or `"slow"` (600ms). Default is `400` .
- `complete` (Function): Optional callback fired once per matched element after animation .

**Syntax 3: Hide with Duration, Easing, and Callback**
```javascript
.hide( duration [, easing ] [, complete ] )
```
- `easing` (String): `"swing"` (default) or `"linear"`. Added in jQuery 1.4.3 .

**Syntax 4: Hide with Options Object**
```javascript
.hide( options )
```
- `options` (PlainObject): Animation settings including `duration`, `easing`, `queue`, `complete`, `step`, `progress`, and Promise-related callbacks .

**Component Breakdown**

- `.hide()` : The method call.
- `duration` : Controls how long the element takes to disappear.
- `easing` : Controls acceleration during the animation.
- `complete` : Executes after the element is fully hidden.
- `options` : Consolidates all animation settings into one object.

**Syntax Rules**

1. `.hide()` caches the element's current `display` value before setting it to `none` .
2. Without a duration, `.hide()` executes immediately and **overrides the animation queue** .
3. When animating, `.hide()` reduces `width`, `height`, and `opacity` to zero, then sets `display: none` .
4. The callback receives no arguments, but `this` is the DOM element being hidden .
5. If multiple elements are matched, the callback fires once per element, not once for the whole collection .

**Constraints and Limitations**

- **Performance**: Hiding many elements with animation can be slow; test with performance tools .
- **Responsive Layouts**: Cached display values may not match the display value required at a different viewport size .
- **No `!important` Override**: `.hide()` cannot hide an element whose CSS rule uses `display: block !important` .
- **Callback Repetition**: In a multi-element collection, the callback fires for each element, which can lead to unexpected repetition if not anticipated .

### Multiple Annotated Complete Code Examples

**Example 1: Instant Hide**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.hide() — Instant</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <p id="notice">This notice will be hidden.</p>
  <button id="hideBtn">Hide Notice</button>

  <script>
    $(function () {
      $("#hideBtn").on("click", function () {
        // Step 1: Hide the paragraph instantly
        $("#notice").hide();

        // Step 2: Verify it is no longer visible
        console.log("Display:", $("#notice").css("display")); // "none"
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking the button makes the paragraph disappear immediately.
- Console logs `"Display: none"`.

**Why This Output Occurs**
`.hide()` with no arguments sets the paragraph's inline `display` style to `none`. The element remains in the DOM but is removed from the rendered layout. jQuery has cached its original `display` value (`block`) for future restoration .

---

**Example 2: Animated Hide with Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.hide() — Animated</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="banner" style="padding:20px; background:orange;">
    This banner will slide away.
  </div>
  <button id="dismiss">Dismiss Banner</button>

  <script>
    $(function () {
      $("#dismiss").on("click", function () {
        // Step 1: Hide with animation over 1 second
        $("#banner").hide(1000, function () {
          // Step 2: Callback runs after banner is fully hidden
          console.log("Banner hidden:", this.id);

          // Step 3: Remove the button as well
          $("#dismiss").hide();
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Dismiss Banner" causes the orange banner to shrink and fade out over 1 second.
- After the banner is hidden, the console logs `"Banner hidden: banner"` and the dismiss button also disappears.

**Why This Output Occurs**
The duration `1000` triggers the animation sequence: width, height, and opacity reduce to zero over one second, then `display: none` is applied. The callback fires once the banner is fully hidden, with `this` bound to the banner element .

---

**Example 3: Hide with Options and Easing**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.hide() — Options Object</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="message" style="padding:15px; background:lightcoral;">
    This message will fade out linearly.
  </div>
  <button id="fade">Fade Out</button>

  <script>
    $(function () {
      $("#fade").on("click", function () {
        $("#message").hide({
          duration: 1200,
          easing: "linear",
          complete: function () {
            console.log("Message hidden with linear easing.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The message box fades and shrinks away over 1.2 seconds at a constant rate.
- Console logs `"Message hidden with linear easing."` after completion.

**Why This Output Occurs**
The options object specifies a 1200ms duration with `linear` easing, meaning the animation progresses at a constant speed rather than the default `swing` (which accelerates then decelerates). The callback confirms completion .

### Real-World Cases

- **Dismissible Alerts**: Hiding a notification banner after the user clicks a close button.
- **Collapsible Sidebars**: Hiding a navigation sidebar to reclaim screen space.
- **Loading States**: Hiding a loading spinner once data has been fetched.
- **Form Field Clearing**: Hiding conditional form fields when a controlling checkbox is unchecked.

### References

- jQuery API — .hide() – https://api.jquery.com/hide/
- jQuery Learning Center — Intro to Effects – https://learn.jquery.com/effects/intro-to-effects/
- jQuery API — Version 1.4.3 Changes – https://api.jquery.com/category/version/1.4.3/

---

## Core Concept 3: `.toggle()`

### Definitions

**Core Definition**
`.toggle()` displays or hides matched elements based on their current visibility state, flipping between shown and hidden with each invocation.

**Technical Definition**
`.toggle()` is a jQuery effects method that inspects the current computed `display` value of each matched element. If the element is visible (`display` is not `none`), `.toggle()` hides it; if the element is hidden (`display: none`), `.toggle()` shows it. When called without arguments, the change is instantaneous. When a duration, options object, or single callback function is provided, `.toggle()` becomes an animation method that animates `width`, `height`, and `opacity` in the appropriate direction .

**Beginner-Friendly Explanation**
`.toggle()` is the "flip-flop" command. One click hides; the next click shows. It saves you from writing `if (visible) { hide(); } else { show(); }`. With a duration, the element shrinks away or grows into view with animation, depending on its current state.

### Purposes

- To alternate an element's visibility with a single method call.
- To simplify code that would otherwise require conditional visibility checks.
- To animate elements in and out with one consistent method.
- To support Boolean-forced display states via `.toggle(true)` or `.toggle(false)`.
- To chain visibility toggles with other animations.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: Instant Toggle (No Arguments)**
```javascript
.toggle()
```
- No arguments. Flips visibility immediately, with no animation .

**Syntax 2: Toggle with Duration and Callback**
```javascript
.toggle( duration [, complete ] )
```
- `duration` (Number or String): Animation duration in milliseconds, or `"fast"` (200ms) or `"slow"` (600ms). Default is `400` .
- `complete` (Function): Optional callback fired once per matched element after animation .

**Syntax 3: Toggle with Duration, Easing, and Callback**
```javascript
.toggle( duration [, easing ] [, complete ] )
```
- `easing` (String): `"swing"` (default) or `"linear"`. Added in jQuery 1.4.3 .

**Syntax 4: Toggle with Options Object**
```javascript
.toggle( options )
```
- `options` (PlainObject): Animation settings including `duration`, `easing`, `queue`, `complete`, and Promise callbacks .

**Syntax 5: Toggle with Boolean Force**
```javascript
.toggle( display )
```
- `display` (Boolean): `true` shows the element; `false` hides it. Added in jQuery 1.3 .

**Component Breakdown**

- `.toggle()` : The method call.
- `duration` : Animation length.
- `easing` : Animation acceleration profile.
- `complete` : Post-animation callback.
- `options` : Consolidated animation settings.
- `display` : Boolean that forces the state rather than toggling.

**Syntax Rules**

1. `.toggle()` checks the current `display` value: `none` triggers a show; anything else triggers a hide .
2. The Boolean signature `.toggle(true)` always shows; `.toggle(false)` always hides, regardless of current state .
3. Passing a **single function** as the first argument triggers the animation signature, **not** the deprecated event-binding signature .
4. The deprecated event signature `.toggle(handler, handler)` was removed in jQuery 1.9 .
5. When animating, the direction of the animation (grow/shrink) depends on the current state.

**Constraints and Limitations**

- **Deprecated Event Overload**: The `.toggle(handler, handler)` event-binding signature was deprecated in jQuery 1.8 and **removed in 1.9**. Do not use it .
- **Function Ambiguity**: A single function argument triggers the animation callback, not event binding. This can confuse developers migrating from the old event signature .
- **Performance**: Same concerns as `.show()` and `.hide()` when animating many elements .
- **No Cached State for Booleans**: The Boolean force signature does not rely on cached values; it directly sets `display` .

### Multiple Annotated Complete Code Examples

**Example 1: Instant Toggle**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.toggle() — Instant</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="content" style="padding:20px; background:lightgreen;">
    Toggle me on and off.
  </div>
  <button id="toggleBtn">Toggle Content</button>

  <script>
    $(function () {
      $("#toggleBtn").on("click", function () {
        // Step 1: Toggle visibility instantly
        $("#content").toggle();

        // Step 2: Log the current state
        var state = $("#content").css("display") === "none" ? "hidden" : "visible";
        console.log("Content is now:", state);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- First click: the green content box disappears; console logs `"Content is now: hidden"`.
- Second click: the box reappears; console logs `"Content is now: visible"`.
- Subsequent clicks alternate between these states.

**Why This Output Occurs**
`.toggle()` inspects the current `display` value. Initially, the box is visible (`display: block`), so the first click hides it. On the second click, `display` is `none`, so `.toggle()` shows it again .

---

**Example 2: Animated Toggle with Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.toggle() — Animated</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="slide" style="padding:20px; background:lightblue;">
    Click the button to animate my visibility.
  </div>
  <button id="animToggle">Toggle with Animation</button>

  <script>
    $(function () {
      $("#animToggle").on("click", function () {
        // Step 1: Toggle with 500ms animation and callback
        $("#slide").toggle(500, function () {
          // Step 2: Log the element's ID after animation
          console.log("Animation finished for:", this.id);

          // Step 3: Check final display state
          var isHidden = $(this).css("display") === "none";
          console.log("Is hidden:", isHidden);
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- First click: the blue box shrinks and fades away over 500ms; console logs `"Animation finished for: slide"` and `"Is hidden: true"`.
- Second click: the box grows and fades back in; console logs the same message but `"Is hidden: false"`.

**Why This Output Occurs**
Providing the duration `500` converts `.toggle()` into an animation method. The direction of the animation depends on the current state. The callback fires after the animation completes, with `this` bound to the animated DOM element .

---

**Example 3: Boolean-Forced Toggle**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.toggle() — Boolean Force</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="indicator" style="padding:10px; background:red; color:white;">
    Status Indicator
  </div>
  <button id="showIt">Show</button>
  <button id="hideIt">Hide</button>

  <script>
    $(function () {
      // Step 1: Force show with .toggle(true)
      $("#showIt").on("click", function () {
        $("#indicator").toggle(true);
        console.log("Forced show.");
      });

      // Step 2: Force hide with .toggle(false)
      $("#hideIt").on("click", function () {
        $("#indicator").toggle(false);
        console.log("Forced hide.");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Show" always displays the red indicator, even if it is already visible.
- Clicking "Hide" always hides the indicator, even if it is already hidden.

**Why This Output Occurs**
The Boolean signature `.toggle(true)` explicitly shows the element without checking its current state. `.toggle(false)` explicitly hides it. This is useful when the desired state is known in advance and should not depend on the current visibility .

### Real-World Cases

- **FAQ Accordions**: Clicking a question toggles the visibility of its answer.
- **Mobile Navigation Menus**: A hamburger icon toggles the visibility of the navigation drawer.
- **Dark Mode Toggles**: Toggling the visibility of certain themed elements.
- **Debug Panels**: A developer tool button toggles a debug information panel.
- **Show/Hide Password Fields**: Toggling the visibility of a password input's plain-text counterpart.

### References

- jQuery API — .toggle() (Effects) – https://api.jquery.com/toggle/
- jQuery API — .toggle() (Event, Removed) – https://api.jquery.com/toggle-event/
- jQuery API — Deprecated 1.8 – https://api.jquery.com/category/deprecated/deprecated-1.8/
- jQuery Learning Center — Intro to Effects – https://learn.jquery.com/effects/intro-to-effects/

---

## Summary Comparison Table

| Feature | `.show()` | `.hide()` | `.toggle()` |
|---|---|---|---|
| **Primary Action** | Displays hidden elements | Hides visible elements | Alternates visibility |
| **Cache Behaviour** | Restores cached `display` | Caches current `display` | Uses cached value |
| **Default Animation** | None (instant) | None (instant) | None (instant) |
| **Duration Default** | 400ms (when animating) | 400ms (when animating) | 400ms (when animating) |
| **Boolean Signature** | No | No | Yes (`.toggle(true/false)`) |
| **Deprecated Overload** | No | No | Yes (event binding, removed 1.9) |
| **Return Value** | jQuery object | jQuery object | jQuery object |
| **Chainable** | Yes | Yes | Yes |

---

## Important Notes on Deprecation and Version-Specific Behaviour

1. **Event `.toggle()` Deprecated**: The `.toggle(handler, handler)` signature for binding alternating click handlers was deprecated in jQuery 1.8 and **removed in jQuery 1.9** . Do not use it in modern code.
2. **Easing Parameter Added**: The optional `easing` string parameter was added in jQuery 1.4.3 .
3. **Boolean Toggle Added**: The `.toggle(display)` Boolean signature was added in jQuery 1.3 .
4. **jQuery UI Extension**: jQuery UI extends all three methods with custom effect signatures. These do not work without jQuery UI loaded .
5. **Performance Warning**: Official documentation notes that animating many elements with these methods may cause performance issues and recommends performance testing .

---

## References

- jQuery API — .show() – https://api.jquery.com/show/
- jQuery API — .hide() – https://api.jquery.com/hide/
- jQuery API — .toggle() (Effects) – https://api.jquery.com/toggle/
- jQuery API — .toggle() (Event, Removed) – https://api.jquery.com/toggle-event/
- jQuery API — Deprecated 1.8 – https://api.jquery.com/category/deprecated/deprecated-1.8/
- jQuery Learning Center — Intro to Effects – https://learn.jquery.com/effects/intro-to-effects/
- jQuery UI API — .show() – https://api.jqueryui.com/1.11/show/
- jQuery API — Version 1.4.3 – https://api.jquery.com/category/version/1.4.3/
- MDN Web Docs — display (CSS) – https://developer.mozilla.org/en-US/docs/Web/CSS/display