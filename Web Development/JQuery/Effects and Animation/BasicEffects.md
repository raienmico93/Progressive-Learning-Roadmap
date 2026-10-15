# jQuery Basic Effects — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Basic Effects are the `.show()`, `.hide()`, and `.toggle()` methods that animate an element's visibility by simultaneously adjusting its `display`, `width`, `height`, `opacity`, `padding`, and `margin` properties.

**Technical Definition:**
`.show()`, `.hide()`, and `.toggle()` are implemented on top of jQuery's animation engine and the `"fx"` queue. When called without a duration (or with `duration: 0`), they instantly set the `display` CSS property (`""` or the element's default display for `.show()`, `"none"` for `.hide()`). When called with a duration, they animate the element's dimensions, padding, margin, and opacity from their current values to the target state. jQuery stores the original `display` value in internal data (`jQuery._data(elem, "olddisplay")`) so `.show()` can restore the correct display type (e.g., `"inline-block"` vs. `"block"`).

**Beginner-Friendly Explanation:**
`.show()`, `.hide()`, and `.toggle()` are the simplest jQuery effects. `.hide()` makes an element disappear, `.show()` makes it reappear, and `.toggle()` switches between the two. Without a duration, they happen instantly. With a duration, the element smoothly shrinks or grows and fades in or out.

### Key Characteristics

- **Dual Behavior:** Without a duration, these methods are instant CSS changes (no animation). With a duration, they animate dimensions and opacity.
- **Display Preservation:** jQuery remembers the original `display` value so `.show()` restores the correct display type.
- **Queue Integration:** Animated versions are added to the `"fx"` queue and run sequentially.
- **Callback Support:** Each method accepts a `complete` callback that fires when the animation finishes.
- **Easing Support:** Animations accept an `easing` argument (`"swing"` or `"linear"`).
- **Return Value:** All three methods return the jQuery object for chaining.
- **Version Behavior:** In jQuery 3.x, `.show()`/`.hide()` with no duration are instantaneous; only the animated versions trigger the `"fx"` queue.

### Prerequisites

- Basic knowledge of jQuery selectors and DOM manipulation
- Understanding of CSS `display` property and its values (`block`, `inline`, `inline-block`, `none`)
- Familiarity with JavaScript callbacks
- Basic knowledge of the `"fx"` queue (see Effects Fundamentals)

### Related Programming Areas

- **Effects Fundamentals:** The broader animation engine that powers these methods.
- **Fading Effects:** `.fadeIn()`, `.fadeOut()`, `.fadeToggle()` for opacity-only animations.
- **Sliding Effects:** `.slideUp()`, `.slideDown()`, `.slideToggle()` for height-only animations.
- **Queue Utilities:** `.queue()`, `.dequeue()`, `.stop()`, `.finish()` for managing animations.
- **CSS Display:** The underlying property these effects manipulate.

### Core Concepts / Features

1. `.show()`
2. `.hide()`
3. `.toggle()`
4. Effect Durations
5. `slow`
6. `fast`
7. Milliseconds
8. Effect Callbacks
9. Effect Easing
10. Basic Effects Best Practices

---

## 1. `.show()`

### Definitions

**Core Definition:** `.show()` displays the matched elements, either instantly or with an animation.

**Technical Definition:** `.show([duration][, easing][, complete])` sets the element's `display` to its original value (stored in `jQuery._data(elem, "olddisplay")` or inferred from the tag). With a duration, it animates `width`, `height`, `padding`, `margin`, and `opacity` from their hidden state to their natural values. The animation is queued on `"fx"`.

**Beginner-Friendly Explanation:** `.show()` makes a hidden element visible. Without a duration, it appears instantly. With a duration, it grows and fades in.

### Purposes

- To make a hidden element visible.
- To reveal content with a smooth transition.
- To restore elements hidden by `.hide()` or `display: none`.
- To chain into a sequence of animations.

### Syntax Rules and Structure

```javascript
$(selector).show();
$(selector).show(duration);
$(selector).show(duration, easing);
$(selector).show(duration, easing, complete);
$(selector).show(options);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `duration` | Number (ms) or `"slow"`/`"fast"`. |
| `easing` | `"swing"` (default) or `"linear"`. |
| `complete` | Callback fired when animation finishes. |
| `options` | Object with `duration`, `easing`, `complete`, `queue`, `specialEasing`, `step`, `progress`. |

**Syntax Rules:**
1. No arguments = instant display.
2. Duration triggers animation.
3. Returns the jQuery object for chaining.
4. `this` in the callback is the animated element.

**Constraints and Limitations:**
- Cannot animate elements with `display: none` if their original display is unknown (jQuery infers from tag name).
- Does not work correctly on elements with `visibility: hidden` (use CSS instead).
- Animations on many elements simultaneously can impact performance.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Instant vs. Animated Show**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.show() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 100px; height: 100px; background: green; margin: 5px; }
    </style>
</head>
<body>
    <div class="box" id="instant" style="display:none;">Instant</div>
    <div class="box" id="animated" style="display:none;">Animated</div>
    <button id="showInstant">Show Instant</button>
    <button id="showAnimated">Show Animated</button>
    <div id="output"></div>
    
    <script>
        // Step 1: Instant show
        $("#showInstant").on("click", function() {
            $("#instant").show();
            $("#output").append("Instant show<br>");
        });

        // Step 2: Animated show over 1 second
        $("#showAnimated").on("click", function() {
            $("#animated").show(1000, "swing", function() {
                $("#output").append("Animated show complete<br>");
                console.log("this is:", this.id); // "animated"
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** Clicking "Show Instant" reveals the first box immediately. Clicking "Show Animated" reveals the second box over 1 second, then logs completion.

**Why This Code Produces That Result:** `.show()` without a duration sets `display` instantly. With `1000`, jQuery animates dimensions and opacity over 1 second, then fires the `complete` callback with `this` bound to the element.

### Real-World Cases
- **Revealing Content:** Showing hidden form fields or error messages.
- **Accordion Panels:** Expanding collapsed sections.
- **Modal Dialogs:** Displaying modal content.

---

## 2. `.hide()`

### Definitions

**Core Definition:** `.hide()` hides the matched elements, either instantly or with an animation.

**Technical Definition:** `.hide([duration][, easing][, complete])` sets the element's `display` to `"none"`. Before hiding, jQuery stores the current `display` value in `jQuery._data(elem, "olddisplay")` so it can be restored later. With a duration, it animates dimensions, padding, margin, and opacity to zero before setting `display: none`.

**Beginner-Friendly Explanation:** `.hide()` makes an element disappear. Without a duration, it vanishes instantly. With a duration, it shrinks and fades out.

### Purposes

- To hide elements from view.
- To remove content with a smooth transition.
- To prepare elements for later `.show()` calls.
- To clean up UI state.

### Syntax Rules and Structure

```javascript
$(selector).hide();
$(selector).hide(duration);
$(selector).hide(duration, easing, complete);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `duration` | Number (ms) or `"slow"`/`"fast"`. |
| `easing` | `"swing"` (default) or `"linear"`. |
| `complete` | Callback fired when animation finishes. |

**Syntax Rules:**
1. No arguments = instant hide.
2. Duration triggers animation.
3. Stores original `display` for later `.show()`.
4. Returns the jQuery object.

**Constraints and Limitations:**
- Hiding an already-hidden element with animation has no visible effect.
- Does not remove elements from the DOM (use `.remove()` or `.detach()`).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Instant vs. Animated Hide**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.hide() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 100px; height: 100px; background: red; margin: 5px; }
    </style>
</head>
<body>
    <div class="box" id="box1">Box 1</div>
    <div class="box" id="box2">Box 2</div>
    <button id="hideInstant">Hide Instant</button>
    <button id="hideAnimated">Hide Animated</button>
    <div id="output"></div>
    <script>
        // Step 1: Instant hide
        $("#hideInstant").on("click", function() {
            $("#box1").hide();
            $("#output").append("Box 1 hidden instantly<br>");
        });

        // Step 2: Animated hide over 800ms
        $("#hideAnimated").on("click", function() {
            $("#box2").hide(800, "linear", function() {
                $("#output").append("Box 2 hide animation complete<br>");
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** Clicking "Hide Instant" makes Box 1 vanish immediately. Clicking "Hide Animated" shrinks and fades Box 2 over 800ms.

**Why This Code Produces That Result:** `.hide()` with no duration sets `display: none` instantly. With `800`, jQuery animates dimensions and opacity, then sets `display: none` and fires the callback.

### Real-World Cases
- **Dismissing Notifications:** Fading out alert banners.
- **Collapsing Sections:** Hiding accordion content.
- **Form Validation:** Hiding error messages after correction.

---

## 3. `.toggle()`

### Definitions

**Core Definition:** `.toggle()` displays or hides the matched elements, depending on their current visibility state.

**Technical Definition:** `.toggle([duration][, easing][, complete])` checks each element's current visibility (using `.is(":hidden")`) and calls `.show()` if hidden or `.hide()` if visible. With a duration, the corresponding animation runs. This method is the visibility toggle counterpart to `.fadeToggle()` and `.slideToggle()`.

**Beginner-Friendly Explanation:** `.toggle()` is like a light switch — if the element is visible, it hides it; if hidden, it shows it.

### Purposes

- To toggle visibility with a single method call.
- To implement show/hide buttons.
- To create expandable/collapsible UI elements.
- To simplify conditional visibility logic.

### Syntax Rules and Structure

```javascript
$(selector).toggle();
$(selector).toggle(duration);
$(selector).toggle(duration, easing, complete);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `duration` | Number (ms) or `"slow"`/`"fast"`. |
| `easing` | `"swing"` (default) or `"linear"`. |
| `complete` | Callback fired after animation. |

**Syntax Rules:**
1. No arguments = instant toggle.
2. Duration triggers animation.
3. Determines action per element based on current visibility.
4. Returns the jQuery object.

**Constraints and Limitations:**
- In a collection, elements with different visibility states are toggled independently.
- Does not toggle CSS classes (use `.toggleClass()` for that).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Toggle Visibility**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.toggle() Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #panel { width: 200px; height: 100px; background: lightblue; padding: 10px; }
    </style>
</head>
<body>
    <button id="toggleBtn">Toggle Panel</button>
    <div id="panel">Panel Content</div>
    <div id="output"></div>

    <script>
        // Step 1: Toggle panel visibility with animation
        $("#toggleBtn").on("click", function() {
            $("#panel").toggle(500, "swing", function() {
                var isVisible = $(this).is(":visible");
                $("#output").text("Panel is now " + (isVisible ? "visible" : "hidden"));
                console.log("Toggled, visible:", isVisible);
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** Each click toggles the panel; the output text updates to "visible" or "hidden".

**Why This Code Produces That Result:** `.toggle()` checks the panel's visibility and calls the appropriate animation. The callback checks `.is(":visible")` to determine the new state.

### Real-World Cases
- **Expand/Collapse Buttons:** Toggling detail sections.
- **Menu Toggles:** Showing/hiding navigation menus.
- **Dark Mode Panels:** Toggling settings panels.

---

## 4. Effect Durations

### Definitions

**Core Definition:** Effect duration is the time (in milliseconds or named alias) over which an animation completes.

**Technical Definition:** Duration can be a number (milliseconds), `"slow"` (600ms), or `"fast"` (200ms). Default is 400ms. `$.fx.speeds` defines these aliases and can be customized. Duration `0` makes the effect instantaneous.

**Beginner-Friendly Explanation:** Duration is how long the animation takes. `"fast"` is quick, `"slow"` is leisurely, and numbers give you exact control.

### Purposes
- To control animation speed.
- To match the pace of user interactions.
- To create subtle or dramatic effects.

### Syntax Rules and Structure

```javascript
$(el).show(1000);       // 1 second
$(el).show("slow");     // 600ms
$(el).show("fast");     // 200ms
$(el).show();           // 400ms (default)
$(el).show(0);          // instant
```

**Syntax Rules:**
1. Number = milliseconds.
2. `"slow"` = 600ms, `"fast"` = 200ms.
3. Default = 400ms.
4. `0` = instant.

**Constraints and Limitations:**
- Aliases can be customized via `$.fx.speeds`.
- Very short durations may appear instant.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing Durations**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Effect Durations</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 80px; height: 80px; background: purple; margin: 5px; }
    </style>
</head>
<body>
    <div class="box" id="fastBox">Fast</div>
    <div class="box" id="slowBox">Slow</div>
    <div class="box" id="msBox">1500ms</div>
    <button id="go">Hide All</button>
    <script>
        $("#go").on("click", function() {
            // Step 1: Different durations
            $("#fastBox").hide("fast");        // 200ms
            $("#slowBox").hide("slow");        // 600ms
            $("#msBox").hide(1500);            // 1.5s
        });
    </script>
</body>
</html>
```

**Expected Output:** The three boxes hide at different speeds — fast, slow, and 1.5 seconds.

**Why This Code Produces That Result:** Each element animates independently with its own duration.

### Real-World Cases
- **Hover Effects:** `"fast"` for responsiveness.
- **Modal Animations:** `"slow"` for drama.
- **Precise Timing:** Milliseconds for choreographed sequences.

---

## 5. `slow`

### Definitions

**Core Definition:** `"slow"` is a duration alias representing 600 milliseconds.

**Technical Definition:** `$.fx.speeds.slow = 600`. Can be customized globally.

**Beginner-Friendly Explanation:** `"slow"` means 600 milliseconds (0.6 seconds).

### Purposes
- To use a readable, memorable duration.
- To apply a consistent "slow" speed across animations.

### Syntax Rules and Structure

```javascript
$(el).hide("slow");
$.fx.speeds.slow = 1000; // customize
```

**Syntax Rules:**
1. Default 600ms.
2. Customizable via `$.fx.speeds`.

**Constraints and Limitations:**
- Changing `$.fx.speeds.slow` affects all animations using `"slow"`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using "slow"**

```html
<!DOCTYPE html>
<html>
<head>
    <title>slow Duration</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" style="width:100px;height:100px;background:green;"></div>
    <button id="go">Hide Slowly</button>
    <script>
        $("#go").on("click", function() {
            $("#box").hide("slow", function() {
                console.log("Hidden after ~600ms");
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** The box fades/shrinks over ~600ms.

**Why This Code Produces That Result:** `"slow"` maps to `$.fx.speeds.slow`, which defaults to 600ms.

### Real-World Cases
- **Noticeable Transitions:** Slow fades for emphasis.
- **Consistent Timing:** Using `"slow"` across a project.

---

## 6. `fast`

### Definitions

**Core Definition:** `"fast"` is a duration alias representing 200 milliseconds.

**Technical Definition:** `$.fx.speeds.fast = 200`. Can be customized globally.

**Beginner-Friendly Explanation:** `"fast"` means 200 milliseconds (0.2 seconds).

### Purposes
- To use a readable, memorable quick duration.
- To apply snappy animations.

### Syntax Rules and Structure

```javascript
$(el).hide("fast");
$.fx.speeds.fast = 100; // customize
```

**Syntax Rules:**
1. Default 200ms.
2. Customizable via `$.fx.speeds`.

**Constraints and Limitations:**
- Very short durations may not be perceptible.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using "fast"**

```html
<!DOCTYPE html>
<html>
<head>
    <title>fast Duration</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" style="width:100px;height:100px;background:orange;"></div>
    <button id="go">Hide Fast</button>
    <script>
        $("#go").on("click", function() {
            $("#box").hide("fast", function() {
                console.log("Hidden after ~200ms");
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** The box hides quickly over ~200ms.

**Why This Code Produces That Result:** `"fast"` maps to `$.fx.speeds.fast`, which defaults to 200ms.

### Real-World Cases
- **Snappy UI:** Fast feedback for clicks.
- **Hover Transitions:** Quick show/hide on hover.

---

## 7. Milliseconds

### Definitions

**Core Definition:** Milliseconds is the numeric duration unit for jQuery effects, providing precise control over animation timing.

**Technical Definition:** Any number passed as the duration argument is interpreted as milliseconds. `0` makes the effect instantaneous. The animation engine steps through values at intervals defined by `$.fx.interval` (default 13ms).

**Beginner-Friendly Explanation:** Milliseconds let you set the exact duration — like 1500 for 1.5 seconds.

### Purposes
- To precisely control animation timing.
- To match animation duration to task duration.
- To create choreographed sequences.

### Syntax Rules and Structure

```javascript
$(el).show(1000);    // 1 second
$(el).hide(500);     // half a second
$(el).toggle(2000);  // 2 seconds
```

**Syntax Rules:**
1. Number = milliseconds.
2. `0` = instant.
3. Affects both duration and queue timing.

**Constraints and Limitations:**
- Very long durations may feel sluggish.
- Extremely short durations may not animate visually.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Precise Millisecond Timing**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Millisecond Durations</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" style="width:100px;height:100px;background:blue;"></div>
    <button id="go">Hide in 2500ms</button>
    <script>
        $("#go").on("click", function() {
            var start = Date.now();
            $("#box").hide(2500, function() {
                var elapsed = Date.now() - start;
                console.log("Animation took ~" + elapsed + "ms");
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** The box hides over approximately 2500ms. Console logs the actual elapsed time.

**Why This Code Produces That Result:** The numeric duration `2500` tells jQuery to animate over 2.5 seconds. The actual time may vary slightly due to frame timing.

### Real-World Cases
- **Progress Bars:** Duration matching known task time.
- **Choreographed Sequences:** Precise multi-element timing.
- **Testing:** Measuring animation duration.

---

## 8. Effect Callbacks

### Definitions

**Core Definition:** Effect callbacks are functions passed as the final argument to effect methods, executed when the animation completes.

**Technical Definition:** The `complete` callback is invoked after the animation finishes, with `this` bound to the animated DOM element. It fires once per element in the collection. The callback is queued after the animation on the `"fx"` queue.

**Beginner-Friendly Explanation:** Effect callbacks let you run code when an animation finishes.

### Purposes
- To chain actions after an animation.
- To update UI state when transitions complete.
- To coordinate sequences.

### Syntax Rules and Structure

```javascript
$(el).hide(500, function() {
    // `this` is the animated element
    console.log("Done");
});
```

**Syntax Rules:**
1. Callback is the last argument.
2. `this` refers to the animated element.
3. Fires once per element.
4. Runs after the animation completes.

**Constraints and Limitations:**
- Does not fire if the animation is stopped with `.stop()` (unless `.stop(true, true)`).
- Runs on the main thread.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Chaining via Callbacks**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Effect Callbacks</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" style="width:100px;height:100px;background:red;"></div>
    <div id="output"></div>
    <script>
        // Step 1: Hide, then show, then report
        $("#box").hide(500, function() {
            $(this).css("background", "blue");
            $(this).show(500, function() {
                $("#output").text("Sequence complete, color is blue");
                console.log("this is box:", this.id);
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** The box hides, turns blue, shows again, then the output displays "Sequence complete, color is blue".

**Why This Code Produces That Result:** The first callback fires after hide, changes the color, and starts show with a nested callback that updates the output.

### Real-World Cases
- **Multi-Step UI:** Chaining hide/show with state updates.
- **Notifications:** Showing a message after a fade-out.

---

## 9. Effect Easing

### Definitions

**Core Definition:** Effect easing is the function that determines the acceleration and deceleration of an animation over time.

**Technical Definition:** jQuery provides two built-in easings: `"swing"` (default) and `"linear"`. `"swing"` applies a sine-based acceleration curve; `"linear"` applies constant speed. Custom easings require jQuery UI or a plugin.

**Beginner-Friendly Explanation:** Easing controls how the animation speeds up or slows down. "Swing" starts slow, speeds up, then slows; "linear" is constant.

### Purposes
- To make animations feel natural.
- To create dramatic or subtle effects.
- To match motion to context.

### Syntax Rules and Structure

```javascript
$(el).hide(500, "swing");
$(el).show(500, "linear");
```

**Syntax Rules:**
1. Default is `"swing"`.
2. `"linear"` is constant speed.
3. Custom easings require a plugin.

**Constraints and Limitations:**
- Only two built-in easings.
- Invalid easing names may cause errors or fall back to default.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing Swing and Linear**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Effect Easing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width:80px;height:80px;margin:5px; }
    </style>
</head>
<body>
    <div class="box" id="swing" style="background:red;"></div>
    <div class="box" id="linear" style="background:blue;"></div>
    <button id="go">Hide</button>
    <script>
        $("#go").on("click", function() {
            $("#swing").hide(1500, "swing");
            $("#linear").hide(1500, "linear");
        });
    </script>
</body>
</html>
```

**Expected Output:** Both boxes hide over 1.5 seconds; the red box (swing) accelerates/decelerates, the blue box (linear) shrinks at constant speed.

**Why This Code Produces That Result:** The easing function controls the rate of change per frame.

### Real-World Cases
- **Natural Motion:** `"swing"` for most UI effects.
- **Progress Bars:** `"linear"` for constant speed.

---

## 10. Basic Effects Best Practices

### Definitions

**Core Definition:** Basic Effects Best Practices are guidelines for using `.show()`, `.hide()`, and `.toggle()` effectively, performantly, and accessibly.

**Technical Definition:** Best practices encompass stopping pending animations before starting new ones, using instant versions for initial state, respecting reduced-motion preferences, preferring CSS transitions for simple cases, avoiding animating many elements simultaneously, and using `.finish()` to complete animations instantly.

### Purposes

- To create smooth, performant effects.
- To prevent animation buildup.
- To respect accessibility preferences.
- To maintain clean, readable code.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Stop before re-animating | Rapid clicks queue effects | `.stop(true, true).toggle()` |
| 2 | Use instant for initial state | `$(el).hide(500)` on load | `$(el).hide()` |
| 3 | Respect reduced motion | Always animate | Check `prefers-reduced-motion` |
| 4 | Prefer CSS transitions | jQuery for hover | CSS `transition` |
| 5 | Avoid many simultaneous animations | Animate 100 elements | Batch or use CSS |
| 6 | Use `.finish()` for instant completion | `.stop()` leaves partial state | `.finish()` |
| 7 | Cache selectors | `$("#x").hide(); $("#x").show()` | `var $x = $("#x");` |
| 8 | Use callbacks over `.delay()` for logic | `.delay(500).show()` | `setTimeout(fn, 500)` |

**Syntax Rules:**
1. Call `.stop(true, true)` before starting a new animation on the same element.
2. Use instant effects for initial page state.
3. Check `prefers-reduced-motion` and disable animations when appropriate.
4. Prefer CSS transitions for simple hover/focus effects.
5. Use `.finish()` when you need an animation to complete immediately.

**Constraints and Limitations:**
- `.stop()` may leave elements in intermediate states.
- Repeated rapid animations can cause performance issues.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Best Practices**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Basic Effects Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #panel { width:200px;padding:10px;background:#eee;display:none; }
    </style>
</head>
<body>
    <button id="toggle">Toggle Panel</button>
    <div id="panel">Panel Content</div>
    <div id="output"></div>

    <script>
        // ✅ BEST PRACTICE: Respect reduced motion
        var reduceMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
        if (reduceMotion) {
            $.fx.off = true;
        }

        // ✅ BEST PRACTICE: Cache selectors
        var $panel = $("#panel");

        // ✅ BEST PRACTICE: Instant initial state
        // Panel starts with display:none in CSS (no animation on load)

        $("#toggle").on("click", function() {
            // ✅ BEST PRACTICE: Stop before re-animating
            $panel.stop(true, true).toggle(400, function() {
                var visible = $(this).is(":visible");
                $("#output").text("Panel is " + (visible ? "open" : "closed"));
                console.log("Toggled, visible:", visible);
            });
        });

        // ✅ BEST PRACTICE: Use .finish() when needed
        // $panel.finish().hide();
    </script>
</body>
</html>
```

**Expected Output:** Each click toggles the panel smoothly; rapid clicks don't queue up. The output text updates. On reduced-motion systems, toggling is instant.

**Why This Code Produces That Result:** The code respects reduced motion, caches selectors, uses instant initial state, and stops pending animations before starting new ones. This results in smooth, accessible, performant effects.

### Real-World Cases

- **Accessible UIs:** Respecting reduced-motion preferences.
- **Responsive Interactions:** Preventing animation buildup on rapid clicks.
- **Performance-Critical Apps:** Using CSS transitions and instant initial states.

---

## References

- jQuery API Documentation — .show() - https://api.jquery.com/show/
- jQuery API Documentation — .hide() - https://api.jquery.com/hide/
- jQuery API Documentation — .toggle() - https://api.jquery.com/toggle/
- jQuery API Documentation — Effects Category - https://api.jquery.com/category/effects/
- jQuery API Documentation — Basics - https://api.jquery.com/category/effects/basics/
- jQuery API Documentation — jQuery.fx.speeds - https://api.jquery.com/jQuery.fx.speeds/
- jQuery API Documentation — jQuery.fx.off - https://api.jquery.com/jQuery.fx.off/
- jQuery API Documentation — .stop() - https://api.jquery.com/stop/
- jQuery API Documentation — .finish() - https://api.jquery.com/finish/
- jQuery Learning Center — Custom Effects - https://learn.jquery.com/effects/custom-effects/
- MDN Web Docs — prefers-reduced-motion - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN Web Docs — CSS Transitions - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions
- W3C — CSS Easing Functions Level 1 - https://www.w3.org/TR/css-easing-1/