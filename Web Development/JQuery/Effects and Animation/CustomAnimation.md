# jQuery Custom Animations — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Custom Animations are effects created with the `.animate()` method, which interpolates one or more numeric CSS properties of an element from their current values to specified target values over a given duration.

**Technical Definition:**
`.animate(properties[, duration][, easing][, complete])` accepts an object of CSS properties and target values, animating each numeric property over the specified duration using jQuery's animation engine and the `"fx"` queue. The engine uses `jQuery.fx.timer` and `requestAnimationFrame` (or `setInterval` in older browsers) to step through values, computing intermediate states based on elapsed time, duration, and the easing function. Supported values include absolute values (`"100px"`), relative values (`"+=50px"`, `"-=20px"`), predefined values (`"show"`, `"hide"`, `"toggle"`), and numeric values (unitless properties or those with default units). Non-numeric properties (e.g., `background-color`) require a plugin like jQuery UI or CSS transitions.

**Beginner-Friendly Explanation:**
`.animate()` lets you create your own animations by specifying which CSS properties to change and what values to change them to. For example, you can make an element grow, move across the screen, or change its opacity — all smoothly over time. You control how long it takes, how it speeds up or slows down, and what happens when it finishes.

### Key Characteristics

- **Numeric Properties Only:** `.animate()` works with CSS properties that have numeric values (`width`, `height`, `top`, `left`, `opacity`, `fontSize`, etc.).
- **Relative Values:** Use `+=` and `-=` prefixes to animate relative to the current value.
- **Predefined Values:** Use `"show"`, `"hide"`, and `"toggle"` to animate to/from the element's natural state.
- **Queue Integration:** All animations are added to the element's `"fx"` queue and run sequentially.
- **Multiple Properties:** Animate multiple properties simultaneously by including them in the properties object.
- **Promise Interface:** Since jQuery 1.8, `.animate()` returns a Promise (via `.promise()`), enabling `.done()`/`.fail()` chaining.
- **Per-Property Easing:** Use `specialEasing` to apply different easings to different properties.
- **Performance:** Animating `opacity` and `transform` is more efficient than animating layout properties like `width` and `top`.

### Prerequisites

- Basic knowledge of jQuery selectors and DOM manipulation
- Understanding of CSS properties and their numeric values
- Familiarity with JavaScript objects and callbacks
- Basic knowledge of the `"fx"` queue (see Effects Fundamentals)
- Familiarity with basic, fading, and sliding effects

### Related Programming Areas

- **Effects Fundamentals:** The broader animation engine.
- **Queue Utilities:** `.queue()`, `.dequeue()`, `.clearQueue()`, `.stop()`, `.finish()` for managing animations.
- **Easing Functions:** `"swing"` and `"linear"` built in; custom easings from jQuery UI.
- **CSS Transitions and Animations:** Native alternatives that are often more performant.
- **Deferred Objects:** Animations return Promises for chaining.
- **Performance Optimization:** Choosing animatable properties and avoiding layout thrashing.

### Core Concepts / Features

1. `.animate()`
2. Animation Properties
3. Animation Duration
4. Animation Easing
5. Animation Callbacks
6. Animation Queue
7. Animation Chaining
8. Relative Values
9. Predefined Values
10. Custom Animations Best Practices

---

## 1. `.animate()`

### Definitions

**Core Definition:** `.animate()` is jQuery's generic method for creating custom animations by interpolating CSS property values over time.

**Technical Definition:** `.animate(properties[, duration][, easing][, complete])` accepts a map of CSS properties and target values. For each property, jQuery reads the current computed value, calculates the difference to the target, and steps through intermediate values over the duration. The animation is added to the `"fx"` queue. Supported value types include absolute (`"100px"`, `100`, `"50%"`), relative (`"+=50"`, `"-=20"`), and predefined (`"show"`, `"hide"`, `"toggle"`). A fourth overload accepts an options object with `duration`, `easing`, `complete`, `queue`, `specialEasing`, `step`, and `progress`.

**Beginner-Friendly Explanation:** `.animate()` is the "do it yourself" animation method. You tell it which CSS properties to change and what values to change them to, and it smoothly transitions between the current and target values.

### Purposes

- To create custom animations not covered by built-in effects.
- To animate multiple CSS properties simultaneously.
- To move, resize, or fade elements with precise control.
- To build complex animation sequences.

### Syntax Rules and Structure

```javascript
// Basic syntax
$(selector).animate(properties, duration, easing, complete);

// Options object syntax
$(selector).animate(properties, {
    duration: 1000,
    easing: "swing",
    complete: function() {},
    step: function(now, fx) {},
    progress: function(animation, progress, remainingMs) {},
    queue: true,
    specialEasing: { width: "linear" }
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `properties` | Object of CSS properties and target values. |
| `duration` | Number (ms) or `"slow"`/`"fast"`. Default 400ms. |
| `easing` | `"swing"` (default) or `"linear"`. |
| `complete` | Callback fired when animation finishes. |
| `step` | Callback fired for each animated property on each step. |
| `progress` | Callback fired after each step with progress info. |
| `specialEasing` | Object mapping properties to specific easings. |
| `queue` | Boolean or string; whether to queue the animation. |

**Syntax Rules:**
1. Properties must have numeric values (or be animatable).
2. Multiple properties animate in parallel.
3. Returns the jQuery object for chaining.
4. `.animate()` returns a Promise via `.promise()`.

**Constraints and Limitations:**
- Cannot animate non-numeric properties (e.g., `background-color`) without a plugin.
- Animating layout properties causes reflows.
- Colors require jQuery UI or a plugin.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Custom Animation**

```html
<!DOCTYPE html>
<html>
<head>
    <title>.animate() Basic</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 100px; height: 100px; background: blue;
               position: relative; color: white; }
    </style>
</head>
<body>
    <div id="box">Box</div>
    <button id="go">Animate</button>
    <div id="output"></div>

    <script>
        // Step 1: Animate multiple properties
        $("#go").on("click", function() {
            $("#box").animate({
                left: "300px",       // move right
                width: "200px",      // grow wider
                height: "150px",     // grow taller
                opacity: 0.5         // fade to 50%
            }, 1500, "swing", function() {
                // Step 2: Callback fires when complete
                $("#output").text("Animation complete");
                console.log("Final width:", $(this).width());
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** Clicking "Animate" moves the box right while growing it wider and taller, and fading it to 50% opacity over 1.5 seconds. The output displays "Animation complete".

**Why This Code Produces That Result:** `.animate()` interpolates each property from its current value to the target over 1500ms using `"swing"` easing. All properties animate in parallel. The `complete` callback fires when all properties reach their targets.

### Real-World Cases
- **Sliding Panels:** Animating `left`/`top` for custom slides.
- **Progress Bars:** Animating `width`.
- **Image Galleries:** Animating `opacity` and positioning.
- **Game Elements:** Animating positions with precise timing.

---

## 2. Animation Properties

### Definitions

**Core Definition:** Animation properties are the CSS properties specified in the `.animate()` properties object that jQuery interpolates over the animation duration.

**Technical Definition:** jQuery supports animating any CSS property whose value can be expressed as a number (with or without a unit). Common animatable properties include `width`, `height`, `top`, `left`, `right`, `bottom`, `margin`, `padding`, `opacity`, `fontSize`, `lineHeight`, `borderWidth`, and `letterSpacing`. Properties that require color interpolation (`color`, `backgroundColor`, `borderColor`) require jQuery UI or a plugin. CSS custom properties (variables) are supported in jQuery 3.2+ when using `"var(--name)"` syntax.

**Beginner-Friendly Explanation:** Animation properties are the CSS things you want to change — like width, height, position, or opacity.

### Purposes
- To specify which visual aspects of an element should change.
- To animate multiple properties simultaneously.

### Syntax Rules and Structure

```javascript
$(el).animate({
    width: "200px",
    height: "150px",
    opacity: 0.5,
    fontSize: "20px"
}, 1000);
```

**Supported Properties:**

| Property | Unit | Notes |
|----------|------|-------|
| `width`, `height` | px, %, em | Layout property (reflow) |
| `top`, `left`, `right`, `bottom` | px, % | Requires `position` |
| `margin`, `padding` | px, em | Layout property |
| `opacity` | unitless (0–1) | GPU-accelerated |
| `fontSize`, `lineHeight` | px, em | Layout property |
| `borderWidth` | px | Layout property |
| `letterSpacing` | px | Layout property |

**Syntax Rules:**
1. Values can be numbers (default unit `px` for most properties) or strings with units.
2. `opacity` is unitless (0–1).
3. Multiple properties animate in parallel.
4. Non-numeric properties are set instantly at the start.

**Constraints and Limitations:**
- Colors require a plugin.
- `transform` is not directly animatable by `.animate()` (use CSS transitions or a plugin).
- Animating layout properties causes reflows.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Animating Various Properties**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Animation Properties</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 100px; height: 100px; background: green;
               position: relative; padding: 10px; color: white; }
    </style>
</head>
<body>
    <div id="box">Box</div>
    <button id="go">Animate</button>
    <div id="output"></div>

    <script>
        $("#go").on("click", function() {
            $("#box").animate({
                width: "250px",
                height: "150px",
                padding: "30px",
                fontSize: "24px",
                opacity: 0.6,
                top: "50px",
                left: "100px"
            }, 2000, "linear", function() {
                $("#output").text("All properties animated");
                console.log("Width:", $(this).css("width"));
                console.log("Font size:", $(this).css("font-size"));
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** The box grows, gains padding, increases font size, fades to 60%, and moves right and down over 2 seconds.

**Why This Code Produces That Result:** Each property in the object is animated from its current value to the target over 2000ms with linear easing.

### Real-World Cases
- **Card Expansion:** Animating width/height.
- **Text Effects:** Animating fontSize/letterSpacing.
- **Overlay Transitions:** Animating opacity.

---

## 3. Animation Duration

### Definitions

**Core Definition:** Animation duration is the time over which `.animate()` interpolates properties from their current values to the target values.

**Technical Definition:** Duration can be a number (milliseconds) or `"slow"` (600ms) / `"fast"` (200ms). Default is 400ms. Duration `0` makes the animation instantaneous. The animation engine steps at intervals defined by `$.fx.interval` (default 13ms).

**Beginner-Friendly Explanation:** Duration is how long the custom animation takes.

### Purposes
- To control animation speed.
- To match the pace of user interactions.
- To create subtle or dramatic effects.

### Syntax Rules and Structure

```javascript
$(el).animate({ width: "200px" }, 1000);   // 1 second
$(el).animate({ width: "200px" }, "slow"); // 600ms
$(el).animate({ width: "200px" }, "fast"); // 200ms
$(el).animate({ width: "200px" });         // 400ms (default)
$(el).animate({ width: "200px" }, 0);      // instant
```

**Syntax Rules:**
1. Number = milliseconds.
2. `"slow"` = 600ms, `"fast"` = 200ms.
3. Default = 400ms.
4. `0` = instant.

**Constraints and Limitations:**
- Very short durations may appear instant.
- Long durations may feel sluggish.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing Durations**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Animation Duration</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 80px; height: 80px; margin: 5px; position: relative; }
    </style>
</head>
<body>
    <div class="box" id="fast" style="background:red;"></div>
    <div class="box" id="slow" style="background:blue;"></div>
    <div class="box" id="custom" style="background:green;"></div>
    <button id="go">Animate</button>
    <script>
        $("#go").on("click", function() {
            $("#fast").animate({ left: "300px" }, "fast");    // 200ms
            $("#slow").animate({ left: "300px" }, "slow");    // 600ms
            $("#custom").animate({ left: "300px" }, 1500);    // 1.5s
        });
    </script>
</body>
</html>
```

**Expected Output:** The three boxes move right at different speeds.

**Why This Code Produces That Result:** Each element animates independently with its own duration.

### Real-World Cases
- **Hover Effects:** `"fast"` for responsiveness.
- **Modal Transitions:** `"slow"` for drama.
- **Precise Choreography:** Milliseconds for sequences.

---

## 4. Animation Easing

### Definitions

**Core Definition:** Animation easing is a function that determines the rate of change of an animation over time, controlling acceleration and deceleration.

**Technical Definition:** jQuery provides two built-in easings: `"swing"` (default, sine-based acceleration) and `"linear"` (constant speed). `.animate()` accepts an `easing` argument and supports `specialEasing` for per-property easing. Custom easings require jQuery UI or a plugin.

**Beginner-Friendly Explanation:** Easing controls how the animation speeds up or slows down.

### Purposes
- To make animations feel natural.
- To create dramatic or subtle effects.
- To apply different motion curves to different properties.

### Syntax Rules and Structure

```javascript
// Global easing
$(el).animate({ width: "200px" }, 1000, "linear");

// Per-property easing
$(el).animate(
    { width: "200px", height: "100px" },
    { duration: 1000, specialEasing: { width: "linear", height: "swing" } }
);
```

**Syntax Rules:**
1. Default easing is `"swing"`.
2. `"linear"` is constant speed.
3. `specialEasing` overrides easing per property.
4. Custom easings require a plugin.

**Constraints and Limitations:**
- Only two built-in easings.
- Invalid easing names may cause errors.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Per-Property Easing**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Animation Easing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 100px; height: 100px; background: purple; position: relative; }
    </style>
</head>
<body>
    <div id="box"></div>
    <button id="go">Animate</button>
    <script>
        $("#go").on("click", function() {
            $("#box").animate(
                { width: "300px", height: "50px" },
                {
                    duration: 1500,
                    specialEasing: {
                        width: "linear",   // constant speed
                        height: "swing"    // accelerate/decelerate
                    }
                }
            );
        });
    </script>
</body>
</html>
```

**Expected Output:** The box widens at constant speed while its height changes with acceleration/deceleration.

**Why This Code Produces That Result:** `specialEasing` applies different easing functions to each property during the same animation.

### Real-World Cases
- **Natural Motion:** `"swing"` for UI elements.
- **Progress Bars:** `"linear"` for constant speed.
- **Mixed Effects:** Different easings for coordinated properties.

---

## 5. Animation Callbacks

### Definitions

**Core Definition:** Animation callbacks are functions passed to `.animate()` that execute at specific points during or after the animation.

**Technical Definition:** `.animate()` supports three callbacks: `complete` (after animation), `step` (per property per step), and `progress` (after each step). The `complete` callback receives no arguments and has `this` bound to the element. `step(now, fx)` receives the current value and the `fx` object. `progress(animation, progress, remainingMs)` receives the animation, progress (0–1), and remaining milliseconds.

**Beginner-Friendly Explanation:** Callbacks let you run code during or after the animation.

### Purposes
- To chain actions after animation completion.
- To update UI state during animation.
- To create custom progress indicators.

### Syntax Rules and Structure

```javascript
$(el).animate(
    { width: "200px" },
    {
        duration: 1000,
        complete: function() { /* after animation */ },
        step: function(now, fx) { /* per property per step */ },
        progress: function(anim, progress, remaining) { /* per step */ }
    }
);
```

**Syntax Rules:**
1. `complete` is called once per element.
2. `step` is called for each animated property on each frame.
3. `progress` is called once per frame per element.
4. All callbacks have `this` bound to the element.

**Constraints and Limitations:**
- Callbacks run on the main thread.
- `complete` does not fire if the animation is stopped (unless `.stop(true, true)`).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using step and progress**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Animation Callbacks</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 100px; height: 100px; background: orange; }
        #progress { width: 0%; height: 20px; background: green; }
    </style>
</head>
<body>
    <div id="box"></div>
    <div id="progress"></div>
    <div id="output"></div>
    <button id="go">Animate</button>

    <script>
        $("#go").on("click", function() {
            $("#box").animate(
                { width: "400px", opacity: 0.3 },
                {
                    duration: 2000,
                    step: function(now, fx) {
                        // Fires for each property on each step
                        if (fx.prop === "width") {
                            $("#output").text("Width: " + Math.round(now) + "px");
                        }
                    },
                    progress: function(anim, progress, remaining) {
                        // Update a progress bar
                        $("#progress").css("width", (progress * 100) + "%");
                    },
                    complete: function() {
                        $("#output").append("<br>Animation complete!");
                        console.log("Done");
                    }
                }
            );
        });
    </script>
</body>
</html>
```

**Expected Output:** The box widens and fades. The output displays the current width; the progress bar fills up; and "Animation complete!" appears at the end.

**Why This Code Produces That Result:** `step` fires for each property on each frame, `progress` fires once per frame with the overall progress, and `complete` fires at the end.

### Real-World Cases
- **Progress Bars:** Using `progress` to update UI.
- **Value Displays:** Using `step` to show current values.
- **Chaining:** Using `complete` to start the next animation.

---

## 6. Animation Queue

### Definitions

**Core Definition:** The animation queue (`"fx"`) is the per-element queue that holds pending animations and executes them sequentially.

**Technical Definition:** Each `.animate()` call adds the animation to the element's `"fx"` queue. If the queue is empty, the animation starts immediately; otherwise, it waits. The `queue` option can be set to `false` to run the animation immediately without queueing, or to a custom queue name.

**Beginner-Friendly Explanation:** The animation queue ensures animations run one after another.

### Purposes
- To sequence animations automatically.
- To prevent conflicting animations.
- To control animation order.

### Syntax Rules and Structure

```javascript
// Queued (default)
$(el).animate({ left: "100px" }, 500).animate({ top: "100px" }, 500);

// Not queued
$(el).animate({ left: "100px" }, { duration: 500, queue: false });
```

**Syntax Rules:**
1. Animations auto-queue on `"fx"`.
2. `queue: false` runs the animation immediately.
3. `queue: "name"` uses a custom queue.
4. `.stop()` and `.clearQueue()` control the queue.

**Constraints and Limitations:**
- Animations on the same element run sequentially.
- Use separate elements for parallel animations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Queued vs. Non-Queued**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Animation Queue</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 60px; height: 60px; margin: 5px; position: relative; }
    </style>
</head>
<body>
    <div class="box" id="queued" style="background:red;"></div>
    <div class="box" id="parallel" style="background:blue;"></div>
    <button id="go">Animate</button>
    
    <script>
        $("#go").on("click", function() {
            // Queued: left, then top
            $("#queued")
                .animate({ left: "200px" }, 1000)
                .animate({ top: "100px" }, 1000);

            // Non-queued: left and top simultaneously
            $("#parallel")
                .animate({ left: "200px" }, { duration: 1000, queue: false })
                .animate({ top: "100px" }, { duration: 1000, queue: false });
        });
    </script>
</body>
</html>
```

**Expected Output:** The red box moves right, then down (sequentially). The blue box moves right and down simultaneously.

**Why This Code Produces That Result:** The first animation queues both movements. The second uses `queue: false` to run both animations in parallel.

### Real-World Cases
- **Animation Sequences:** Multi-step movements.
- **Parallel Effects:** Simultaneous property changes.

---

## 7. Animation Chaining

### Definitions

**Core Definition:** Animation chaining is the practice of calling multiple `.animate()` methods (and other effect methods) in a single chain, each queued after the previous.

**Technical Definition:** Because `.animate()` returns the jQuery object, methods can be chained. Each animation is added to the `"fx"` queue and runs after the previous completes. Chaining can mix `.animate()` with `.fadeIn()`, `.slideUp()`, `.delay()`, etc.

**Beginner-Friendly Explanation:** Chaining lets you write "do this, then this, then this" in one line.

### Purposes
- To create multi-step animation sequences concisely.
- To improve code readability.
- To coordinate complex transitions.

### Syntax Rules and Structure

```javascript
$(el)
    .animate({ left: "100px" }, 500)
    .animate({ top: "100px" }, 500)
    .fadeOut(500)
    .fadeIn(500);
```

**Syntax Rules:**
1. Each method returns the jQuery object.
2. Animations queue sequentially.
3. Can mix different effect types.

**Constraints and Limitations:**
- Long chains can be hard to debug.
- Errors in one step don't stop the chain.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Chained Custom Animations**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Animation Chaining</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 80px; height: 80px; background: teal; position: relative; }
    </style>
</head>
<body>
    <div id="box"></div>
    <button id="go">Run Chain</button>
    <div id="output"></div>

    <script>
        $("#go").on("click", function() {
            $("#box")
                .animate({ left: "200px" }, 800)
                .animate({ top: "150px" }, 800)
                .animate({ left: "0px", top: "0px" }, 1200, function() {
                    $("#output").text("Chain complete!");
                    console.log("All animations done");
                });
        });
    </script>
</body>
</html>
```

**Expected Output:** The box moves right, then down, then back to the origin, then displays "Chain complete!".

**Why This Code Produces That Result:** Each `.animate()` is queued on `"fx"` and runs after the previous completes.

### Real-World Cases
- **Guided Tours:** Highlighting elements in sequence.
- **Loading Sequences:** Multi-step reveals.
- **Interactive Stories:** Choreographed movements.

---

## 8. Relative Values

### Definitions

**Core Definition:** Relative values in `.animate()` use `+=` or `-=` prefixes to animate a property relative to its current value.

**Technical Definition:** `$(el).animate({ width: "+=50px" })` animates the width from its current value to current + 50px. This works for numeric properties and is useful when the current value is not known in advance.

**Beginner-Friendly Explanation:** Relative values let you say "make it 50px wider" instead of "make it 150px wide."

### Purposes
- To animate relative to the current value without knowing it.
- To create incremental animations.
- To build toggle-like behaviors.

### Syntax Rules and Structure

```javascript
$(el).animate({ width: "+=50px" }, 500);   // grow by 50px
$(el).animate({ left: "-=20px" }, 500);    // move left by 20px
$(el).animate({ opacity: "-=0.2" }, 500);  // reduce opacity by 0.2
```

**Syntax Rules:**
1. Use `+=` to increase, `-=` to decrease.
2. Include units (`px`, `em`, `%`).
3. Works with numeric properties.

**Constraints and Limitations:**
- Values can go negative if not careful.
- Does not work with non-numeric properties.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Relative Animation**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Relative Values</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 100px; height: 100px; background: coral;
               position: relative; }
    </style>
</head>
<body>
    <div id="box">Box</div>
    <button id="grow">Grow</button>
    <button id="move">Move Right</button>
    <button id="shrink">Shrink</button>
    <div id="output"></div>

    <script>
        $("#grow").on("click", function() {
            $("#box").animate({ width: "+=50px", height: "+=50px" }, 500);
        });

        $("#move").on("click", function() {
            $("#box").animate({ left: "+=100px" }, 500);
        });

        $("#shrink").on("click", function() {
            $("#box").animate({ width: "-=30px", height: "-=30px" }, 500, function() {
                $("#output").text("Final width: " + $(this).width() + "px");
            });
        });
    </script>
</body>
</html>
```

**Expected Output:** Each click grows, moves, or shrinks the box relative to its current state. The shrink button displays the final width.

**Why This Code Produces That Result:** `+=` and `-=` compute the target relative to the current value at animation start.

### Real-World Cases
- **Incremental UI:** Growing panels on repeated clicks.
- **Toggles:** Shrinking/growing elements without knowing the current size.

---

## 9. Predefined Values

### Definitions

**Core Definition:** Predefined values in `.animate()` are special string values (`"show"`, `"hide"`, `"toggle"`) that animate properties to or from the element's natural state.

**Technical Definition:** When a property's target value is `"show"`, jQuery animates it to its natural value (as if `.show()` were called). `"hide"` animates to 0 or `display: none`. `"toggle"` chooses based on current visibility. These are syntactic shortcuts for common animations.

**Beginner-Friendly Explanation:** Predefined values let you use `"show"`, `"hide"`, or `"toggle"` as animation targets instead of specific numbers.

### Purposes
- To animate to/from natural state without knowing values.
- To toggle properties with a single animation.
- To simplify common animations.

### Syntax Rules and Structure

```javascript
$(el).animate({ height: "toggle" }, 500);
$(el).animate({ opacity: "show" }, 500);
$(el).animate({ width: "hide" }, 500);
```

**Syntax Rules:**
1. `"show"` animates to natural value.
2. `"hide"` animates to 0/none.
3. `"toggle"` chooses based on state.

**Constraints and Limitations:**
- Only works for animatable properties.
- Behavior may vary for different properties.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using Predefined Values**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Predefined Values</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 100px; height: 100px; background: gold; }
    </style>
</head>
<body>
    <div id="box">Box</div>
    <button id="toggle">Toggle Height</button>
    <button id="fade">Toggle Opacity</button>

    <script>
        $("#toggle").on("click", function() {
            $("#box").animate({ height: "toggle" }, 600);
        });

        $("#fade").on("click", function() {
            $("#box").animate({ opacity: "toggle" }, 600);
        });
    </script>
</body>
</html>
```

**Expected Output:** Clicking "Toggle Height" collapses or expands the box's height. Clicking "Toggle Opacity" fades the box in or out.

**Why This Code Produces That Result:** `"toggle"` checks the current state and animates accordingly.

### Real-World Cases
- **Accordion Animations:** Toggling height.
- **Fade Toggles:** Toggling opacity.

---

## 10. Custom Animations Best Practices

### Definitions

**Core Definition:** Custom Animations Best Practices are guidelines for using `.animate()` effectively, performantly, and accessibly.

**Technical Definition:** Best practices encompass stopping pending animations before starting new ones, animating performant properties (opacity, transform), respecting `prefers-reduced-motion`, preferring CSS transitions for simple cases, using `specialEasing` carefully, avoiding overly long chains, cleaning up animations, and using the options object for complex configurations.

### Purposes

- To create smooth, performant custom animations.
- To prevent animation buildup and jank.
- To respect accessibility preferences.
- To maintain clean, readable code.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Stop before re-animating | Rapid clicks queue animations | `.stop(true, true).animate(...)` |
| 2 | Animate performant properties | `animate({width})` | `animate({opacity})` |
| 3 | Respect reduced motion | Always animate | Check `prefers-reduced-motion` |
| 4 | Prefer CSS transitions | jQuery for hover | CSS `transition` |
| 5 | Use options object for complex configs | Many positional args | `animate(props, {duration, easing, complete})` |
| 6 | Avoid overly long chains | 20-step chain | Break into functions |
| 7 | Use `specialEasing` sparingly | Mixing easings randomly | Intentional per-property easing |
| 8 | Use `.finish()` for instant completion | `.stop()` leaves partial state | `.finish()` |
| 9 | Cache selectors | Repeated `$("#x")` | `var $x = $("#x");` |
| 10 | Use relative values when appropriate | Hardcoding values | `"+=50px"` |

**Syntax Rules:**
1. Call `.stop(true, true)` before starting a new animation on the same element.
2. Prefer `opacity` and `transform` over layout properties.
3. Check `window.matchMedia("(prefers-reduced-motion: reduce)")` and disable animations when appropriate.
4. Use the options object for complex configurations.
5. Use `.finish()` when you need an animation to complete immediately.

**Constraints and Limitations:**
- `.stop()` may leave elements in intermediate states.
- Layout animations cause reflows.
- Colors require plugins.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Custom Animation Best Practices**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Custom Animation Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #card { width: 200px; height: 150px; background: #eee;
                padding: 15px; position: relative; }
    </style>
</head>
<body>
    <div id="card">Card Content</div>
    <button id="expand">Expand</button>
    <button id="collapse">Collapse</button>
    <div id="output"></div>

    <script>
        // ✅ BEST PRACTICE: Respect reduced motion
        var reduceMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
        if (reduceMotion) {
            $.fx.off = true;
        }

        // ✅ BEST PRACTICE: Cache selectors
        var $card = $("#card");

        // ✅ BEST PRACTICE: Use options object
        $("#expand").on("click", function() {
            $card.stop(true, true).animate(
                { height: "300px", width: "400px", opacity: 1 },
                {
                    duration: 600,
                    easing: "swing",
                    complete: function() {
                        $("#output").text("Expanded");
                    }
                }
            );
        });

        $("#collapse").on("click", function() {
            $card.stop(true, true).animate(
                { height: "150px", width: "200px", opacity: 0.8 },
                {
                    duration: 600,
                    complete: function() {
                        $("#output").text("Collapsed");
                    }
                }
            );
        });

        // ✅ BEST PRACTICE: Clean up with .finish() if needed
        // $card.finish();
    </script>
</body>
</html>
```

**Expected Output:** Clicking "Expand" grows the card to 400×300 at full opacity; "Collapse" shrinks it back. Rapid clicks don't queue animations. On reduced-motion systems, changes are instant.

**Why This Code Produces That Result:** The code respects reduced motion, caches selectors, uses the options object, and stops pending animations before starting new ones. This results in smooth, accessible, performant custom animations.

### Real-World Cases

- **Card Expansions:** Growing/shrinking panels.
- **Interactive Dashboards:** Animating widget sizes.
- **Game UI:** Precise position and size animations.

---

## Comparison of Animation Value Types

| Value Type | Example | Description |
|------------|---------|-------------|
| Absolute | `"200px"`, `100` | Target is the exact value |
| Relative | `"+=50px"`, `"-=20px"` | Target is current ± value |
| Predefined | `"show"`, `"hide"`, `"toggle"` | Target is natural/hidden state |
| Unitless | `0.5` (opacity) | For unitless properties |
| Percentage | `"50%"` | Percentage of parent |

---

## References

- jQuery API Documentation — .animate() - https://api.jquery.com/animate/
- jQuery API Documentation — Effects Category - https://api.jquery.com/category/effects/
- jQuery API Documentation — Custom Animations - https://api.jquery.com/animate/#custom-animations
- jQuery API Documentation — .stop() - https://api.jquery.com/stop/
- jQuery API Documentation — .finish() - https://api.jquery.com/finish/
- jQuery API Documentation — .delay() - https://api.jquery.com/delay/
- jQuery API Documentation — jQuery.fx.off - https://api.jquery.com/jQuery.fx.off/
- jQuery API Documentation — jQuery.fx.speeds - https://api.jquery.com/jQuery.fx.speeds/
- jQuery API Documentation — Queue and Dequeue - https://learn.jquery.com/effects/uses-of-queue-and-dequeue/
- jQuery Learning Center — Custom Effects - https://learn.jquery.com/effects/custom-effects/
- jQuery UI — Easing Functions - https://api.jqueryui.com/easings/
- MDN Web Docs — prefers-reduced-motion - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN Web Docs — CSS Transitions - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions
- MDN Web Docs — CSS Animations - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations
- W3C — CSS Easing Functions Level 1 - https://www.w3.org/TR/css-easing-1/