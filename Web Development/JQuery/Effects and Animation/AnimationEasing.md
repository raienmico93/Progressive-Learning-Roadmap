# jQuery Animation Easing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:**
jQuery Animation Easing is a mathematical function that determines the rate at which an animation progresses over time, controlling acceleration and deceleration to produce natural, linear, or dramatic motion effects.

**Technical Definition:**
An easing function receives five parameters — `(x, t, b, c, d)` — where `x` is the percentage of time elapsed (0–1), `t` is the milliseconds since animation start, `b` is the beginning value (0), `c` is the total change (1), and `d` is the duration in milliseconds. It returns a normalized progress value that jQuery uses to interpolate CSS property values on each animation frame. jQuery core provides only two easing implementations: `swing` (the default) and `linear`. Additional easing functions are available via plugins, most notably jQuery UI.

**Beginner-Friendly Explanation:**
Easing controls how an animation speeds up or slows down. Without easing (linear), an element moves at a constant speed. With easing (swing), it starts slowly, speeds up in the middle, then slows down at the end — like a car accelerating and braking smoothly. Easing makes animations feel more natural and polished.

### Key Characteristics

- **Two Built-in Easings:** jQuery core ships with only `"swing"` (default) and `"linear"`.
- **Plugin Extensibility:** Custom easings can be registered on `$.easing` or loaded from plugins like jQuery UI.
- **Per-Property Easing:** Since jQuery 1.4, different properties in a single `.animate()` call can use different easing functions via arrays or `specialEasing`.
- **Signature:** `function(x, t, b, c, d)` where `x` is progress (0–1), `t` is elapsed time, `b` is begin value, `c` is change, `d` is duration.
- **Non-Monotonic Easings:** Some easings (e.g., `easeOutBounce`, `easeOutElastic`) produce values outside the 0–1 range, causing overshoot or bounce. Negative values may be clamped for properties like `height` or `opacity`.
- **Performance Impact:** The easing function is called on every animation frame per property; complex custom easings with heavy math can impact performance.

### Prerequisites

- Basic knowledge of jQuery selectors and `.animate()`
- Understanding of the `"fx"` animation queue
- Familiarity with JavaScript functions and mathematical concepts (sine, power, etc.)
- Basic knowledge of CSS properties being animated

### Related Programming Areas

- **Animation Queue:** Easing functions execute within the animation engine's frame loop.
- **Custom Animations:** `.animate()` accepts easing as a parameter.
- **Effects Methods:** `.fadeIn()`, `.slideUp()`, etc., all accept easing arguments.
- **jQuery UI:** Provides over 30 additional easing functions.
- **CSS Easing Functions:** Native CSS `cubic-bezier()` and `steps()` provide similar control for CSS transitions.

### Core Concepts / Features

1. Easing Functions
2. `swing`
3. `linear`
4. Custom Easing
5. `$.easing`
6. jQuery UI Easing
7. Easing Best Practices

---

## 1. Easing Functions

### Definitions

**Core Definition:** An easing function is a mathematical function that maps linear time progress to a non-linear animation progress, controlling the speed curve of an animation.

**Technical Definition:** Easing functions conform to the signature `function(x, t, b, c, d)` where:
- `x` — percentage of time elapsed (0 to 1)
- `t` — milliseconds since animation start
- `b` — beginning value (typically 0)
- `c` — total change (typically 1)
- `d` — total duration in milliseconds

The function returns the interpolated value at the current time. jQuery calls the easing function on every animation frame for each animated property.

**Beginner-Friendly Explanation:** An easing function is a formula that decides how fast or slow the animation moves at each moment. Some start slow, some start fast, some bounce.

### Purposes

- To make animations feel natural and physics-like.
- To emphasize certain transitions (e.g., bounce for attention).
- To control the perceived speed of UI feedback.
- To differentiate concurrent property animations with per-property easing.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
// Standard easing argument
$(el).animate(properties, duration, easingName, complete);

// Per-property easing (array syntax)
$(el).animate({
    width: ["toggle", "swing"],
    opacity: ["toggle", "linear"]
}, 5000);

// Per-property easing (specialEasing syntax)
$(el).animate(
    { width: "toggle", height: "toggle" },
    {
        duration: 5000,
        specialEasing: {
            width: "linear",
            height: "easeOutBounce"
        }
    }
);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `easingName` | String name of a registered easing function (`"swing"`, `"linear"`, or custom). |
| `specialEasing` | Object mapping property names to easing function names. |
| Array syntax | `[targetValue, easingName]` for per-property easing. |

**Syntax Rules:**
1. If easing is omitted, `"swing"` is used.
2. Per-property easing overrides the global easing argument.
3. Array syntax and `specialEasing` are mutually exclusive in the same call.
4. Custom easings must be registered before use.

**Constraints and Limitations:**
- jQuery core only provides `"swing"` and `"linear"`.
- Invalid easing names may cause errors or fall back to default.
- Per-property easing requires jQuery 1.4+.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing Easing Functions**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Easing Comparison</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        .box { width: 60px; height: 60px; margin: 10px; position: relative; }
    </style>
</head>
<body>
    <div class="box" id="swingBox" style="background:red;">Swing</div>
    <div class="box" id="linearBox" style="background:blue;">Linear</div>
    <button id="go">Animate</button>

    <script>
        $("#go").on("click", function() {
            // Step 1: Animate with swing (accelerates then decelerates)
            $("#swingBox").animate({ left: "400px" }, 2000, "swing");

            // Step 2: Animate with linear (constant speed)
            $("#linearBox").animate({ left: "400px" }, 2000, "linear");
        });
    </script>
</body>
</html>
```

**Expected Output:** Both boxes move right over 2 seconds. The red box (swing) accelerates in the middle and decelerates at the end; the blue box (linear) moves at constant speed.

**Why This Code Produces That Result:** The `"swing"` easing applies a sine-based acceleration curve, while `"linear"` returns the raw progress value unchanged, resulting in constant speed.

### Real-World Cases

- **UI Feedback:** Using `"swing"` for buttons and menus feels natural.
- **Progress Indicators:** Using `"linear"` for progress bars maintains constant speed.
- **Attention-Seeking:** Using `"easeOutBounce"` from jQuery UI for notifications.
- **Smooth Scrolling:** Using `"easeInOutCubic"` for page scroll animations.

---

## 2. `swing`

### Definitions

**Core Definition:** `"swing"` is jQuery's default easing function, which accelerates in the middle of the animation and decelerates at the beginning and end.

**Technical Definition:** The swing function is defined as `0.5 - Math.cos(p * Math.PI) / 2`, where `p` is the percentage of time elapsed (0–1). This produces a smooth sine-wave acceleration curve.

**Beginner-Friendly Explanation:** Swing starts slow, speeds up in the middle, then slows down at the end — like a car pulling away from a stop and braking smoothly.

### Purposes
- To provide a natural, physics-like default for all animations.
- To avoid the mechanical feel of linear motion.
- To serve as the default easing when none is specified.

### Syntax Rules and Structure

```javascript
$(el).animate({ left: "200px" }, 1000);            // swing (default)
$(el).animate({ left: "200px" }, 1000, "swing");   // explicit
```

**Syntax Rules:**
1. Default easing for all effects methods.
2. Used when no easing argument is provided.
3. Can be specified explicitly by name.

**Constraints and Limitations:**
- Not suitable for constant-speed animations.
- The deceleration at the end can make rough animations more noticeable.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Explicit Swing Easing**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Swing Easing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" style="width:80px;height:80px;background:purple;position:relative;"></div>
    <button id="go">Animate with Swing</button>
    <script>
        $("#go").on("click", function() {
            $("#box").animate(
                { left: "300px", opacity: 0.5 },
                1500,
                "swing",
                function() {
                    console.log("Swing animation complete");
                }
            );
        });
    </script>
</body>
</html>
```

**Expected Output:** The box moves right while fading to 50% opacity, accelerating in the middle and decelerating at the end.

**Why This Code Produces That Result:** The `"swing"` easing applies the sine-based curve to both `left` and `opacity` simultaneously.

### Real-World Cases
- **General UI Animations:** Default for most effects.
- **Menu Transitions:** Natural feel for dropdowns and accordions.
- **Modal Entrances:** Smooth acceleration and deceleration.

---

## 3. `linear`

### Definitions

**Core Definition:** `"linear"` is an easing function that progresses at a constant pace throughout the animation.

**Technical Definition:** The linear function is defined as `function(p) { return p; }`, where `p` is the percentage of time elapsed. It returns the progress value unchanged, resulting in constant-speed motion.

**Beginner-Friendly Explanation:** Linear means constant speed — no acceleration, no deceleration, just steady motion from start to finish.

### Purposes
- To animate at a constant, predictable speed.
- To avoid the deceleration artifacts of swing on slow animations.
- To create mechanical or progress-based motions.

### Syntax Rules and Structure

```javascript
$(el).animate({ left: "200px" }, 1000, "linear");
$(el).fadeIn(500, "linear");
```

**Syntax Rules:**
1. Returns progress unchanged.
2. Can be used with any effect method.
3. Performs well because it requires no complex math.

**Constraints and Limitations:**
- Lacks the natural feel of swing.
- May appear mechanical or robotic.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Linear Easing for Constant Speed**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Linear Easing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #track { width: 500px; height: 20px; background: #eee; position: relative; }
        #progress { width: 0%; height: 100%; background: green; }
    </style>
</head>
<body>
    <div id="track"><div id="progress"></div></div>
    <button id="go">Start Progress</button>
    <script>
        $("#go").on("click", function() {
            $("#progress").animate(
                { width: "100%" },
                3000,
                "linear",
                function() {
                    console.log("Progress complete");
                }
            );
        });
    </script>
</body>
</html>
```

**Expected Output:** The progress bar fills from 0% to 100% over 3 seconds at a constant rate.

**Why This Code Produces That Result:** The `"linear"` easing returns the raw progress value, so the width increases uniformly each frame.

### Real-World Cases
- **Progress Bars:** Constant speed matches user expectations.
- **Loading Indicators:** Predictable timing.
- **Marquee Animations:** Continuous scrolling.

---

## 4. Custom Easing

### Definitions

**Core Definition:** Custom easing is a user-defined easing function registered on `$.easing` that provides a non-standard acceleration curve.

**Technical Definition:** A custom easing is a function with the signature `function(x, t, b, c, d)` added to the `$.easing` object. It can use any mathematical formula (polynomial, trigonometric, exponential) to produce a desired motion curve. Once registered, its name can be passed to any effect method's `easing` argument.

**Beginner-Friendly Explanation:** Custom easing lets you create your own motion curve — like a bounce, elastic, or custom acceleration — by writing a math function.

### Purposes
- To create unique motion effects not available in core or plugins.
- To implement precise physics-based or stylized animations.
- To match brand-specific motion guidelines.

### Syntax Rules and Structure

**Complete General Syntax:**

```javascript
// Register a custom easing
$.easing.myEasing = function(x, t, b, c, d) {
    // x: progress (0-1), t: elapsed ms, b: begin, c: change, d: duration
    return adjustedProgress;
};

// Use it
$(el).animate({ left: "200px" }, 1000, "myEasing");
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `x` | Percentage of time elapsed (0–1). |
| `t` | Milliseconds since animation start. |
| `b` | Beginning value (typically 0). |
| `c` | Total change (typically 1). |
| `d` | Duration in milliseconds. |

**Syntax Rules:**
1. Must be registered on `$.easing` before use.
2. Returns a normalized progress value.
3. Can use `x` alone (most common) or `t`, `b`, `c`, `d` for more control.
4. Works with `.animate()`, `.fadeIn()`, `.slideUp()`, etc.

**Constraints and Limitations:**
- Heavy math can impact animation performance.
- Non-monotonic easings may produce values outside 0–1.
- Negative values may be clamped for some properties.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Registering a Custom Easing**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Custom Easing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <style>
        #box { width: 80px; height: 80px; background: orange; position: relative; }
    </style>
</head>
<body>
    <div id="box"></div>
    <button id="go">Custom Ease</button>
    <script>
        // Step 1: Register a custom easing (easeInCubic)
        $.easing.easeInCubic = function(x, t, b, c, d) {
            return c * Math.pow(t / d, 3) + b;
        };

        // Step 2: Use the custom easing
        $("#go").on("click", function() {
            $("#box").animate(
                { left: "400px" },
                2000,
                "easeInCubic",
                function() {
                    console.log("Custom ease complete");
                }
            );
        });
    </script>
</body>
</html>
```

**Expected Output:** The box moves right, starting very slowly and accelerating dramatically toward the end.

**Why This Code Produces That Result:** The `easeInCubic` function cubes the normalized time, producing a curve that starts flat and rises steeply.

### Real-World Cases
- **Brand Motion:** Custom curves matching brand guidelines.
- **Physics Simulations:** Projectile or spring motion.
- **Staggered Reveals:** Custom acceleration for choreographed sequences.

---

## 5. `$.easing`

### Definitions

**Core Definition:** `$.easing` is the jQuery namespace object where all easing functions are registered and accessed by name.

**Technical Definition:** `jQuery.easing` is an object whose keys are easing names and whose values are easing functions. jQuery core populates it with `swing` and `linear`. Custom easings and plugin easings (e.g., jQuery UI, danro's easing plugin) add properties to this object.

**Beginner-Friendly Explanation:** `$.easing` is the "dictionary" where all easing functions live. You add your own by assigning `$.easing.myName = function(...) {}`.

### Purposes
- To provide a central registry for easing functions.
- To allow plugins to extend available easings.
- To access or override existing easings.

### Syntax Rules and Structure

```javascript
// Register a single easing
$.easing.myEase = function(x, t, b, c, d) { return x; };

// Register multiple easings via $.extend
$.extend($.easing, {
    easeInCubic: function(x, t, b, c, d) {
        return c * Math.pow(t / d, 3) + b;
    },
    easeOutCubic: function(x, t, b, c, d) {
        return c * (Math.pow(t / d - 1, 3) + 1) + b;
    }
});

// Access an easing
var fn = $.easing.swing;
```

**Syntax Rules:**
1. Keys are easing names used in `.animate()`.
2. Values are functions with signature `(x, t, b, c, d)`.
3. `$.extend()` registers multiple easings at once.
4. Built-in easings can be overridden.

**Constraints and Limitations:**
- Overriding built-in easings affects all code on the page.
- No validation of easing function signatures.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Registering Multiple Easings via `$.extend`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>$.easing Registration</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="box" style="width:80px;height:80px;background:teal;position:relative;"></div>
    <button id="go">Animate</button>
    <script>
        // Step 1: Register multiple easings at once
        $.extend($.easing, {
            easeInQuad: function(x, t, b, c, d) {
                return c * (t /= d) * t + b;
            },
            easeOutQuad: function(x, t, b, c, d) {
                return -c * (t /= d) * (t - 2) + b;
            }
        });

        // Step 2: Use the registered easings
        $("#go").on("click", function() {
            $("#box").animate(
                { left: "400px" },
                1500,
                "easeInQuad",
                function() {
                    console.log("easeInQuad complete");
                }
            );
        });
    </script>
</body>
</html>
```

**Expected Output:** The box accelerates from slow to fast, completing the animation with a quick finish.

**Why This Code Produces That Result:** The `easeInQuad` function squares the normalized time, producing a quadratic acceleration curve.

### Real-World Cases
- **Plugin Development:** Registering easing functions for a plugin.
- **Batch Registration:** Adding multiple easings from a library.

---

## 6. jQuery UI Easing

### Definitions

**Core Definition:** jQuery UI Easing is a suite of over 30 additional easing functions provided by the jQuery UI library, ranging from quadratic/cubic/quartic/quintic variations to specialized effects like bounce and elastic.

**Technical Definition:** jQuery UI extends `$.easing` with functions such as `easeInQuad`, `easeOutQuad`, `easeInOutQuad`, `easeInCubic`, `easeOutCubic`, `easeInOutCubic`, `easeInQuart`, `easeOutQuart`, `easeInOutQuart`, `easeInQuint`, `easeOutQuint`, `easeInOutQuint`, `easeInSine`, `easeOutSine`, `easeInOutSine`, `easeInExpo`, `easeOutExpo`, `easeInOutExpo`, `easeInCirc`, `easeOutCirc`, `easeInOutCirc`, `easeInElastic`, `easeOutElastic`, `easeInOutElastic`, `easeInBack`, `easeOutBack`, `easeInOutBack`, `easeInBounce`, `easeOutBounce`, and `easeInOutBounce`.

**Beginner-Friendly Explanation:** jQuery UI gives you a huge menu of easing options — from subtle sine waves to dramatic bounces and elastic springs.

### Purposes
- To provide a wide variety of motion effects for different UI contexts.
- To enable dramatic effects like bouncing and elastic overshoot.
- To offer fine-grained control over acceleration curves.

### Syntax Rules and Structure

```html
<!-- Include jQuery UI -->
<script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
```

```javascript
// Use any jQuery UI easing
$(el).animate({ left: "200px" }, 1000, "easeOutBounce");
$(el).slideUp(500, "easeInCubic");
```

**Syntax Rules:**
1. Requires jQuery UI library to be loaded.
2. Easing names are passed as strings.
3. Works with all jQuery effect methods.

**Constraints and Limitations:**
- Adds a significant library size if not already using jQuery UI.
- Some easings (bounce, elastic) produce values outside 0–1.
- Negative values may be clamped for `height` and `opacity`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using jQuery UI Easings**

```html
<!DOCTYPE html>
<html>
<head>
    <title>jQuery UI Easing</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
    <style>
        .box { width: 70px; height: 70px; margin: 8px; position: relative; }
    </style>
</head>
<body>
    <div class="box" id="bounce" style="background:red;">Bounce</div>
    <div class="box" id="elastic" style="background:blue;">Elastic</div>
    <div class="box" id="back" style="background:green;">Back</div>
    <button id="go">Animate</button>

    <script>
        $("#go").on("click", function() {
            // Step 1: Bounce easing
            $("#bounce").animate({ left: "400px" }, 1500, "easeOutBounce");

            // Step 2: Elastic easing
            $("#elastic").animate({ left: "400px" }, 1500, "easeOutElastic");

            // Step 3: Back easing (overshoots then settles)
            $("#back").animate({ left: "400px" }, 1500, "easeOutBack");
        });
    </script>
</body>
</html>
```

**Expected Output:** The red box bounces to a stop, the blue box springs elastically, and the green box overshoots then settles back.

**Why This Code Produces That Result:** Each jQuery UI easing applies its specific mathematical curve: bounce simulates a bouncing ball, elastic simulates a spring, and back overshoots the target before settling.

### Real-World Cases
- **Notifications:** `easeOutBounce` for attention-grabbing alerts.
- **Modals:** `easeOutBack` for playful entrances.
- **Page Transitions:** `easeInOutQuad` for smooth navigation.
- **Loading Spinners:** `easeInOutSine` for rhythmic rotation.

---

## 7. Easing Best Practices

### Definitions

**Core Definition:** Easing Best Practices are guidelines for selecting and applying easing functions to create smooth, performant, and contextually appropriate animations.

**Technical Definition:** Best practices encompass choosing easings that match the interaction context (natural for UI, constant for progress, dramatic for attention), avoiding overuse of complex easings, being aware of performance implications, respecting `prefers-reduced-motion`, using per-property easing judiciously, and preferring CSS transitions for simple cases.

### Purposes

- To create animations that feel natural and intentional.
- To maintain performance across devices.
- To respect accessibility preferences.
- To avoid visual jarring or inconsistency.

### Syntax Rules and Structure

**Key Best Practices:**

| # | Practice | Bad Example | Good Example |
|---|----------|-------------|--------------|
| 1 | Match easing to context | Bounce for progress bar | Linear for progress, swing for UI |
| 2 | Avoid overusing dramatic easings | Bounce everywhere | Bounce for notifications only |
| 3 | Use `swing` as default | Custom easing for everything | Swing for general UI |
| 4 | Respect reduced motion | Always animate | Check `prefers-reduced-motion` |
| 5 | Prefer CSS for simple cases | jQuery for hover easing | CSS `transition-timing-function` |
| 6 | Be aware of performance | Complex math on many elements | Simple easings for bulk animations |
| 7 | Use per-property easing intentionally | Random easing per property | Coordinated easing for related properties |
| 8 | Test on slow devices | Assume smooth | Profile on low-end hardware |

**Syntax Rules:**
1. Use `"swing"` for most UI animations.
2. Use `"linear"` for progress and constant-speed motions.
3. Use jQuery UI easings for dramatic effects sparingly.
4. Check `window.matchMedia("(prefers-reduced-motion: reduce)")` and disable animations when appropriate.
5. Prefer CSS transitions for simple hover/focus effects.

**Constraints and Limitations:**
- Complex easings increase CPU usage.
- Non-monotonic easings may produce unexpected values.
- jQuery UI adds significant library weight.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Applying Easing Best Practices**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Easing Best Practices</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
    <style>
        #card { width: 150px; height: 100px; background: #eee; padding: 15px; }
        #progress { width: 0%; height: 20px; background: green; margin-top: 10px; }
    </style>
</head>
<body>
    <div id="card">Card</div>
    <div id="progress"></div>
    <button id="go">Animate</button>

    <script>
        // ✅ BEST PRACTICE: Respect reduced motion
        var reduceMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
        if (reduceMotion) {
            $.fx.off = true;
        }

        $("#go").on("click", function() {
            // ✅ BEST PRACTICE: Use swing for UI card (natural feel)
            $("#card").animate(
                { height: "200px" },
                { duration: 600, easing: "swing" }
            );

            // ✅ BEST PRACTICE: Use linear for progress (constant speed)
            $("#progress").animate(
                { width: "100%" },
                { duration: 2000, easing: "linear" }
            );

            // ✅ BEST PRACTICE: Use dramatic easing sparingly (for notifications only)
            // $("#notification").animate({ top: "0px" }, 800, "easeOutBounce");
        });
    </script>
</body>
</html>
```

**Expected Output:** Clicking "Animate" expands the card with a natural swing motion and fills the progress bar at constant speed. On reduced-motion systems, changes are instant.

**Why This Code Produces That Result:** The code respects reduced motion, uses `"swing"` for natural UI motion, and `"linear"` for constant-speed progress, demonstrating context-appropriate easing selection.

### Real-World Cases

- **Design Systems:** Standardizing easing across an application.
- **Accessible UIs:** Respecting reduced-motion preferences.
- **Performance-Critical Apps:** Using simple easings for bulk animations.
- **Brand Guidelines:** Selecting custom easings that match brand identity.

---

## Easing Function Reference

### jQuery Core (Built-in)

| Name | Formula | Description |
|------|---------|-------------|
| `swing` | `0.5 - cos(p * π) / 2` | Default; slow-fast-slow |
| `linear` | `p` | Constant speed |

### jQuery UI (Selected Common Easings)

| Name | Behavior |
|------|----------|
| `easeInQuad` | Slow start, accelerating |
| `easeOutQuad` | Fast start, decelerating |
| `easeInOutQuad` | Slow-fast-slow |
| `easeInCubic` | Very slow start, strong acceleration |
| `easeOutCubic` | Fast start, strong deceleration |
| `easeInOutCubic` | Slow-fast-slow with cubic curve |
| `easeInSine` | Gentle sine-based acceleration |
| `easeOutSine` | Gentle sine-based deceleration |
| `easeInOutSine` | Gentle sine-based both ways |
| `easeInExpo` | Extreme slow start |
| `easeOutExpo` | Extreme fast start |
| `easeInOutExpo` | Extreme both ways |
| `easeInCirc` | Circular curve acceleration |
| `easeOutCirc` | Circular curve deceleration |
| `easeInElastic` | Elastic spring start |
| `easeOutElastic` | Elastic spring end |
| `easeInBack` | Pulls back before moving |
| `easeOutBack` | Overshoots then settles |
| `easeInBounce` | Bounces at start |
| `easeOutBounce` | Bounces at end |
| `easeInOutBounce` | Bounces at both ends |

### Custom Easing Examples

```javascript
// Quadratic
$.easing.easeInQuad = function(x, t, b, c, d) {
    return c * (t /= d) * t + b;
};

// Cubic
$.easing.easeOutCubic = function(x, t, b, c, d) {
    return c * ((t = t / d - 1) * t * t + 1) + b;
};

// Bounce (simplified)
$.easing.myBounce = function(x, t, b, c, d) {
    if ((t /= d) < (1 / 2.75)) {
        return c * (7.5625 * t * t) + b;
    } else if (t < (2 / 2.75)) {
        return c * (7.5625 * (t -= (1.5 / 2.75)) * t + 0.75) + b;
    } else {
        return c * (7.5625 * (t -= (2.25 / 2.75)) * t + 0.9375) + b;
    }
};
```

---

## References

- jQuery API Documentation — .animate() (Easing) - https://api.jquery.com/animate/
- jQuery API Documentation — jQuery.easing - https://api.jquery.com/jQuery.easing/
- jQuery UI API — Easings - https://api.jqueryui.com/easings/
- jQuery UI API (Chinese) — 缓动 - https://api.jqueryui.com.cn/easings/
- Stack Overflow — jQuery easing function variables - https://stackoverflow.com/questions/5916058/jquery-easing-function-variables-comprehension
- Stack Overflow — Custom easing in jQuery - https://stackoverflow.com/questions/2473215/customize-easing-in-jquery
- GitHub — danro/jquery-easing - https://github.com/danro/jquery-easing
- GitHub — danro/easing-js - https://github.com/danro/easing-js
- TutorialsPoint — How to add easing effect to your animation with jQuery - https://www.tutorialspoint.com/article/How-to-add-easing-effect-to-your-animation-with-jQuery
- Robert Penner — Easing Functions - http://www.robertpenner.com/easing/
- MDN Web Docs — prefers-reduced-motion - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- MDN Web Docs — CSS Easing Functions - https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function
- W3C — CSS Easing Functions Level 1 - https://www.w3.org/TR/css-easing-1/