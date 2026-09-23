# jQuery Custom Animation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Custom Animation in jQuery refers to the `.animate()` method, which creates arbitrary CSS property animations by transitioning numeric values over a specified duration with controllable easing and completion callbacks.

**Technical Definition**
`.animate()` is a jQuery effects method that performs a custom animation of a set of CSS properties. It accepts a plain object of CSS properties and target values, a duration in milliseconds (or `"fast"`/`"slow"`), an optional easing function name, and an optional completion callback. The method animates each numeric property from its current computed value to the target value over the specified duration, using the `fx` queue by default. Non-numeric properties cannot be animated without plugins such as jQuery Color for colour values .

**Beginner-Friendly Explanation**
`.animate()` is like a customisable transition tool. Instead of using fixed effects like "fade" or "slide," you tell jQuery exactly which CSS properties to change, what values to change them to, and how long the change should take. For example, you can say "animate the width to 70% and opacity to 0.4 over 1.5 seconds" and jQuery smoothly transitions those properties.

### Key Characteristics

- **Arbitrary CSS Properties**: Animates any numeric CSS property (width, height, left, opacity, fontSize, etc.) .
- **Relative Values**: Supports `+=` and `-=` prefixes for relative animations .
- **Toggle/Show/Hide Values**: Accepts `"toggle"`, `"show"`, and `"hide"` strings for automatic visibility management .
- **Per-Property Easing**: Since jQuery 1.4, each property can have its own easing function .
- **Queue Integration**: Animations are queued by default but can be run in parallel with `queue: false` .
- **Completion Callbacks**: Supports `complete`, `done`, `fail`, `always`, `start`, `step`, and `progress` callbacks .

### Prerequisites

- Basic HTML, CSS, and JavaScript knowledge.
- Understanding of CSS numeric properties (width, height, opacity, etc.).
- jQuery library included via CDN or local file.
- Familiarity with jQuery selectors and effects methods.

### Related Programming Areas

- **jQuery Effects Suite**: `.fadeIn()`, `.slideDown()`, `.show()`, `.hide()`.
- **Animation Queuing**: `.queue()`, `.dequeue()`, `.stop()`, `.finish()`.
- **jQuery UI**: Extends `.animate()` with colour animation and additional easings .
- **Promise API**: `.promise()` for collection-wide completion callbacks .

### Core Concepts / Features

1. `.animate()` — Core method
2. CSS Property Animation
3. Duration
4. Easing
5. Completion Callbacks

---

## Core Concept 1: `.animate()`

### Definitions

**Core Definition**
`.animate()` is the jQuery method that performs a custom animation of a set of CSS properties, transitioning each from its current value to a specified target value over a given duration.

**Technical Definition**
`.animate()` accepts a plain object of CSS properties and target values as its first required parameter. It animates each numeric property using the jQuery `fx` engine, which interpolates values over the duration using the specified easing function. The method integrates with jQuery's animation queue, returning the jQuery object for chaining. Non-numeric properties (e.g., colours) require the jQuery Color plugin. The method does not automatically make hidden elements visible, unlike `.slideDown()` or `.fadeIn()` .

**Beginner-Friendly Explanation**
`.animate()` is the "do it yourself" animation method. You provide a list of CSS changes — like "make the box 200 pixels wider and 50% transparent" — and jQuery smoothly performs those changes over the time you specify.

### Purposes

- To animate arbitrary numeric CSS properties not covered by built-in effects.
- To create custom transitions for specific UI interactions.
- To chain multiple property animations in a single call.
- To animate relative values (e.g., move 50px to the right from current position).
- To control animation flow with per-property easing and detailed callbacks.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: Basic Signature**
```javascript
.animate( properties [, duration ] [, easing ] [, complete ] )
```
- `properties` (PlainObject): **Required**. A map of CSS properties and target values .
- `duration` (Number or String): Optional. Default `400`; `"fast"` = 200ms, `"slow"` = 600ms .
- `easing` (String): Optional. `"swing"` (default) or `"linear"` .
- `complete` (Function): Optional. Executed when animation completes .

**Syntax 2: Options Object Signature**
```javascript
.animate( properties, options )
```
- `options` (PlainObject): Contains `duration`, `easing`, `queue`, `specialEasing`, `step`, `progress`, `complete`, `done`, `fail`, `always`, `start` .

**Component Breakdown**

- `.animate()` : The method call.
- `properties` : Object mapping CSS property names to target values.
- `duration` : Animation length in milliseconds.
- `easing` : Rate-of-change profile name.
- `complete` : Post-animation callback function.

**Syntax Rules**

1. At least one CSS property must be provided in the `properties` object.
2. Property names use camelCase (e.g., `borderWidth`) or CSS hyphenated names in quotes .
3. Values are treated as pixels unless units (`em`, `%`) are specified .
4. `"show"`, `"hide"`, and `"toggle"` strings manage display state automatically .
5. Relative values use `+=` or `-=` prefixes .

**Constraints and Limitations**

- **Colour Animation**: Colours cannot be animated without the jQuery Color plugin .
- **Shorthand Properties**: `font`, `background`, and `border` shorthand are not fully supported .
- **Hidden Elements**: `.animate()` does not make hidden elements visible .
- **Directional Properties**: `top`, `left`, etc., have no effect unless `position` is not `static` .

### Multiple Annotated Complete Code Examples

**Example 1: Basic Multi-Property Animation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.animate() — Basic</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #block { width: 100px; height: 100px; background: lightblue; position: relative; }
  </style>
</head>
<body>
  <div id="block">Animate me</div>
  <button id="go">Animate</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        // Step 1: Animate width, opacity, and fontSize simultaneously
        $("#block").animate({
          width: "200px",        // target width
          opacity: 0.5,          // target opacity
          fontSize: "20px"       // target font size
        }, 1000);                // duration: 1 second
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Animate" expands the block to 200px wide, fades it to 50% opacity, and increases font size to 20px over 1 second.

**Why This Output Occurs**
The `properties` object defines three CSS properties with target values. jQuery interpolates each from current to target over 1000ms .

---

**Example 2: Relative Animation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.animate() — Relative Values</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #box { width: 50px; height: 50px; background: red; position: relative; }
  </style>
</head>
<body>
  <div id="box"></div>
  <button id="moveRight">Move Right (+=50px)</button>
  <button id="moveLeft">Move Left (-=50px)</button>

  <script>
    $(function () {
      $("#moveRight").on("click", function () {
        // Step 1: Move 50px to the right relative to current position
        $("#box").animate({ left: "+=50px" }, "slow");
      });

      $("#moveLeft").on("click", function () {
        // Step 2: Move 50px to the left relative to current position
        $("#box").animate({ left: "-=50px" }, "slow");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Move Right" shifts the red box 50px right from its current position.
- Clicking "Move Left" shifts it 50px left.
- Repeated clicks queue up relative movements.

**Why This Output Occurs**
The `+=` prefix tells jQuery to compute the target as current value plus 50px, rather than an absolute value .

---

**Example 3: Toggle Value with Options Object**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>.animate() — Toggle and Options</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="panel" style="padding:20px; background:lightgreen;">
    This panel animates with options.
  </div>
  <button id="toggleBtn">Animate Toggle</button>

  <script>
    $(function () {
      $("#toggleBtn").on("click", function () {
        // Step 1: Animate width and opacity with toggle behaviour
        $("#panel").animate({
          width: "toggle",       // shrink to hide or expand to show
          opacity: "toggle"      // fade out or fade in
        }, {
          duration: 600,
          easing: "linear",
          complete: function () {
            console.log("Animation complete. Display:", $(this).css("display"));
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- First click: panel shrinks and fades out; console logs `"Animation complete. Display: none"`.
- Second click: panel expands and fades in; console logs `"Animation complete. Display: block"`.

**Why This Output Occurs**
The `"toggle"` string tells jQuery to automatically determine the direction based on current state. The options object provides duration, easing, and callback .

### Real-World Cases

- **Interactive Panels**: Animating a settings panel's width and opacity when toggled.
- **Progress Indicators**: Animating a bar's width to represent loading progress.
- **Hover Effects**: Custom animations for menu items or buttons on hover.
- **Modal Transitions**: Animating modals with custom scale and opacity effects.

### References

- jQuery API — .animate() – https://api.jquery.com/animate/
- jQuery Learning Center — Custom Effects with .animate() – https://learn.jquery.com/effects/custom-effects/

---

## Core Concept 2: CSS Property Animation

### Definitions

**Core Definition**
CSS Property Animation is the process of smoothly transitioning one or more numeric CSS properties from their current values to specified target values.

**Technical Definition**
jQuery's `.animate()` method interpolates numeric CSS properties over time. Each property is animated from its computed current value to the target value. Non-numeric properties (colours, background images) require plugins. Property values are treated as pixels unless units are specified. Relative values use `+=`/`-=` prefixes. The `"show"`, `"hide"`, and `"toggle"` keywords manage display state .

**Beginner-Friendly Explanation**
Not every CSS property can be animated — only ones that have numeric values that make sense to gradually change. You can animate width, height, opacity, font-size, and position values. You cannot animate things like colours without a plugin.

### Purposes

- To animate layout dimensions (width, height, margin, padding).
- To animate visual properties (opacity, fontSize).
- To animate positional properties (left, top, scrollTop).
- To create custom visual transitions beyond built-in effects.
- To chain multiple property animations in a single call.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
.animate({
  propertyName: targetValue,
  anotherProperty: "+=50px",
  visibilityProperty: "toggle"
}, duration, easing, callback);
```

**Component Breakdown**

- `propertyName` : CSS property in camelCase (e.g., `borderWidth`) or quoted hyphenated form.
- `targetValue` : Numeric value, string with units, relative (`+=`/`-=`), or `"toggle"`/`"show"`/`"hide"`.

**Syntax Rules**

1. Animated properties must be numeric or use supported keywords .
2. Property names use camelCase or quoted CSS names .
3. Values without units default to pixels .
4. Relative animations require a starting value .
5. Non-numeric properties require plugins .

**Constraints and Limitations**

- Colour animation requires jQuery Color plugin .
- Shorthand properties (`font`, `background`, `border`) are not fully supported .
- `scrollTop` and `scrollLeft` can be animated on any element .
- `position: static` elements ignore `top`/`left` animations .

### Multiple Annotated Complete Code Examples

**Example 1: Animating Dimensions and Position**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Property Animation — Dimensions</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #card { width: 100px; height: 80px; background: #3498db; position: relative; color: white; }
  </style>
</head>
<body>
  <div id="card">Card</div>
  <button id="grow">Grow and Move</button>

  <script>
    $(function () {
      $("#grow").on("click", function () {
        // Animate multiple numeric properties
        $("#card").animate({
          width: "250px",           // absolute width
          height: "+=40px",         // relative height increase
          left: "100px",            // position change
          marginTop: "20px"         // margin animation
        }, 800);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The card expands to 250px wide, grows 40px taller, moves 100px right, and gains 20px top margin over 800ms.

**Why This Output Occurs**
All four properties are numeric and animatable. Relative and absolute values can be mixed in the same call .

---

**Example 2: Animating `scrollTop`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Property Animation — scrollTop</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    #scrollBox { width: 200px; height: 100px; overflow-y: scroll; border: 1px solid #ccc; }
    .content { height: 600px; }
  </style>
</head>
<body>
  <div id="scrollBox">
    <div class="content">Scrollable content here...</div>
  </div>
  <button id="scrollBtn">Animate Scroll to Bottom</button>

  <script>
    $(function () {
      $("#scrollBtn").on("click", function () {
        // Animate scrollTop to scroll the container
        $("#scrollBox").animate({
          scrollTop: 500    // scroll down 500px
        }, 1000);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking the button smoothly scrolls the content inside `#scrollBox` down by 500px over 1 second.

**Why This Output Occurs**
`scrollTop` is a non-style property that can be animated. jQuery interpolates its value over the duration .

### Real-World Cases

- **Accordion Panels**: Animating height to expand/collapse content.
- **Progress Bars**: Animating width to show completion percentage.
- **Image Sliders**: Animating `left` or `marginLeft` to slide images.
- **Sticky Headers**: Animating `top` or `marginTop` on scroll.

### References

- jQuery API — .animate() – https://api.jquery.com/animate/
- jQuery Learning Center — Custom Effects – https://learn.jquery.com/effects/custom-effects/

---

## Core Concept 3: Duration

### Definitions

**Core Definition**
Duration is the length of time an animation takes to complete, specified in milliseconds or using the string shortcuts `"fast"` and `"slow"`.

**Technical Definition**
The `duration` parameter controls how long `.animate()` takes to interpolate properties from current to target values. It accepts numbers (milliseconds), `"slow"` (600ms), or `"fast"` (200ms). Default is 400ms. Duration can be set globally for all effects via `jQuery.fx.speeds` .

**Beginner-Friendly Explanation**
Duration is simply "how long the animation takes." You can say "500" for half a second, "slow" for a slower animation, or "fast" for a quick one.

### Purposes

- To control the speed of custom animations.
- To synchronise animations across multiple elements.
- To create snappy or leisurely transitions depending on context.
- To provide user-controlled speed preferences.
- To integrate with queue-based animation sequences.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
.animate( properties, duration )
.animate( properties, duration, easing )
.animate( properties, duration, easing, complete )
```

**Component Breakdown**

- `duration` : Number (milliseconds) or String (`"fast"`, `"slow"`).

**Syntax Rules**

1. Default duration is 400ms .
2. `"fast"` = 200ms; `"slow"` = 600ms .
3. Duration can be set globally: `jQuery.fx.speeds.fast = 150` .
4. Higher values = slower animations .

**Constraints and Limitations**

- Very short durations (< 50ms) may appear instant.
- Very long durations may feel sluggish to users.
- Duration does not affect the order of queued animations.

### Multiple Annotated Complete Code Examples

**Example 1: Comparing Durations**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Duration — Comparison</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .box { width: 50px; height: 50px; background: blue; margin: 10px; }
  </style>
</head>
<body>
  <div class="box" id="fastBox"></div>
  <div class="box" id="slowBox"></div>
  <button id="go">Animate Both</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        // Fast animation: 200ms
        $("#fastBox").animate({ width: "200px" }, "fast");

        // Slow animation: 600ms
        $("#slowBox").animate({ width: "200px" }, "slow");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The first box expands to 200px in 200ms.
- The second box expands to 200px in 600ms.
- Both reach the same final state at different times.

**Why This Output Occurs**
`"fast"` maps to 200ms and `"slow"` to 600ms. Both animations perform the same property change but at different speeds .

### Real-World Cases

- **Loading Spinners**: Fast animations for quick feedback.
- **Modal Transitions**: Slow animations for dramatic effect.
- **User Preferences**: Respecting `prefers-reduced-motion` by using shorter durations.

### References

- jQuery API — .animate() – https://api.jquery.com/animate/
- jQuery API — jQuery.fx.speeds – https://api.jquery.com/jQuery.fx.speeds/

---

## Core Concept 4: Easing

### Definitions

**Core Definition**
Easing is a function that determines the rate of change during an animation, controlling how the speed of the transition varies over time.

**Technical Definition**
An easing function specifies the speed at which an animation progresses at different points. jQuery core ships with two easings: `swing` (default), which starts slowly, accelerates, then decelerates, and `linear`, which progresses at a constant pace. Additional easings require plugins such as jQuery UI. Since jQuery 1.4, per-property easing is supported .

**Beginner-Friendly Explanation**
Easing is the "personality" of the animation. `linear` moves at a steady speed — like a car on cruise control. `swing` starts slowly, speeds up in the middle, and slows down at the end — like a natural human motion.

### Purposes

- To create natural-looking motion with acceleration and deceleration.
- To control the feel of UI transitions.
- To apply different easings to different properties in one animation.
- To use advanced easings (bounce, elastic) via plugins.
- To differentiate the character of various UI effects.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: Single Easing for All Properties**
```javascript
.animate( properties, duration, easing, complete )
```

**Syntax 2: Per-Property Easing (Array Syntax)**
```javascript
.animate({
  width: [ "toggle", "swing" ],
  opacity: [ "toggle", "linear" ]
}, duration, easing, complete )
```
- The first member is the target value; the second is the easing function .

**Syntax 3: `specialEasing` Option**
```javascript
.animate( properties, {
  specialEasing: {
    width: "linear",
    height: "easeOutBounce"
  },
  duration: 5000,
  complete: callback
})
```
- Maps properties to specific easings .

**Component Breakdown**

- `easing` : String naming the easing function.
- Array syntax: `[value, easing]` per property.
- `specialEasing` : Object mapping properties to easing names.

**Syntax Rules**

1. Built-in easings: `"swing"` (default) and `"linear"` .
2. Per-property easing requires array syntax or `specialEasing` .
3. Undefined per-property easing falls back to the method's `easing` argument .
4. Additional easings require jQuery UI or other plugins .

**Constraints and Limitations**

- Only `swing` and `linear` available without plugins .
- `easeOutBounce` and others require jQuery UI .
- Some easings produce negative intermediate values; jQuery may clamp them at zero .

### Multiple Annotated Complete Code Examples

**Example 1: Built-in Easing Comparison**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Easing — Swing vs Linear</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .box { width: 50px; height: 50px; margin: 5px; }
  </style>
</head>
<body>
  <div class="box" id="swingBox" style="background:blue;"></div>
  <div class="box" id="linearBox" style="background:green;"></div>
  <button id="go">Animate</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        // Swing: slow start, fast middle, slow end
        $("#swingBox").animate({ width: "300px" }, 1000, "swing");

        // Linear: constant speed throughout
        $("#linearBox").animate({ width: "300px" }, 1000, "linear");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Both boxes reach 300px width after 1 second, but the blue box (swing) starts and ends slower, while the green box (linear) moves at a constant speed.

**Why This Output Occurs**
`swing` uses a trigonometric curve for natural acceleration; `linear` progresses at a constant rate .

---

**Example 2: Per-Property Easing**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Easing — Per-Property</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" style="width:100px; height:100px; background:purple; opacity:1;"></div>
  <button id="go">Animate with Mixed Easing</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        // Width uses swing; opacity uses linear
        $("#box").animate({
          width: [ "300px", "swing" ],
          opacity: [ 0.3, "linear" ]
        }, 1200);
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Width expands with swing easing (accelerates/decelerates).
- Opacity fades with linear easing (constant rate).
- Both complete after 1.2 seconds.

**Why This Output Occurs**
Array syntax `[value, easing]` assigns different easing functions to different properties .

### Real-World Cases

- **Modal Pop-ups**: Using `easeOutBounce` for playful entrance.
- **Menu Dropdowns**: Using `swing` for natural-feeling reveals.
- **Loading Bars**: Using `linear` for steady progress indication.
- **Attention Effects**: Using `easeOutElastic` for bouncing notifications.

### References

- jQuery API — .animate() – https://api.jquery.com/animate/
- jQuery UI API — Easings – https://api.jqueryui.com/easings/
- jQuery Learning Center — Custom Effects – https://learn.jquery.com/effects/custom-effects/

---

## Core Concept 5: Completion Callbacks

### Definitions

**Core Definition**
Completion callbacks are functions executed after an animation finishes, allowing actions to be performed when the transition is complete.

**Technical Definition**
`.animate()` supports multiple callback types: `complete` (fires once per element when its animation finishes), `done` (promise-style, fires when the animation's Promise resolves), `always` (fires regardless of success/failure), `fail` (fires on failure), `start` (fires when animation begins), `step` (fires for each animated property at each step), and `progress` (fires once per step regardless of property count). These are called on a per-element basis .

**Beginner-Friendly Explanation**
Callbacks are "what to do next" functions. You tell jQuery "animate this, and when you're done, run this function." This lets you chain actions or trigger logic after the animation.

### Purposes

- To execute code after an animation completes.
- To chain multiple animations in sequence.
- To update UI state after transitions finish.
- To handle animation success or failure.
- To perform per-step operations during animation.

### Syntax Rules and Structure

**Complete General Syntaxes**

**Syntax 1: `complete` in Basic Signature**
```javascript
.animate( properties, duration, easing, complete )
```

**Syntax 2: `complete` in Options Object**
```javascript
.animate( properties, {
  complete: function() { ... }
})
```

**Syntax 3: Promise-style `done`**
```javascript
.animate( properties, duration ).done(function() { ... })
```
- Fires once after all elements complete .

**Component Breakdown**

- `complete` : Function called per element when animation finishes .
- `done` : Promise callback, fires once for entire collection .
- `start` : Fires when animation begins .
- `step` : Fires for each property at each step .
- `progress` : Fires once per step .

**Syntax Rules**

1. `complete` fires **once per matched element**, not once for the collection .
2. Use `.promise().done()` for collection-wide callbacks .
3. Inside callbacks, `this` refers to the DOM element being animated .
4. `done` receives the Promise and a boolean indicating if animation jumped to end .

**Constraints and Limitations**

- `complete` fires multiple times in multi-element collections .
- Passing a function **call** (e.g., `alert('done')`) instead of a function **reference** executes immediately .
- `done` and `complete` have different timing priorities .

### Multiple Annotated Complete Code Examples

**Example 1: `complete` Callback**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Callbacks — complete</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" style="width:100px; height:100px; background:orange;"></div>
  <button id="go">Animate</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        $("#box").animate({
          width: "300px",
          opacity: 0.5
        }, 800, "swing", function () {
          // Step 1: Callback runs after animation
          console.log("Animation done for:", this.id);
          console.log("Final width:", $(this).css("width"));
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- The box expands and fades over 800ms.
- Console logs `"Animation done for: box"` and `"Final width: 300px"`.

**Why This Output Occurs**
The `complete` callback fires after all animated properties reach their targets. `this` is the box element .

---

**Example 2: `done` vs `complete` with Multiple Elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Callbacks — done vs complete</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <style>
    .box { width: 50px; height: 50px; margin: 5px; background: teal; }
  </style>
</head>
<body>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <button id="go">Animate All</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        var $boxes = $(".box");

        // complete fires 3 times (once per element)
        $boxes.animate({ width: "150px" }, 600, function () {
          console.log("complete for element:", this.id || "box");
        });

        // done fires 1 time (after all elements finish)
        $boxes.promise().done(function () {
          console.log("All animations complete (done).");
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
complete for element: box
complete for element: box
complete for element: box
All animations complete (done).
```

**Why This Output Occurs**
`complete` fires per element; `done` fires once for the entire collection after all promises resolve .

---

**Example 3: `start` and `step` Callbacks**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Callbacks — start and step</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="box" style="width:50px; height:50px; background:red;"></div>
  <button id="go">Animate with Step Logging</button>

  <script>
    $(function () {
      $("#go").on("click", function () {
        $("#box").animate({
          width: "250px"
        }, {
          duration: 500,
          start: function () {
            console.log("Animation started.");
          },
          step: function (now, fx) {
            // Log current width value at each step
            console.log("Step:", fx.prop, "=", now);
          },
          complete: function () {
            console.log("Animation complete.");
          }
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output (Console)**
```
Animation started.
Step: width = 50
Step: width = 75
Step: width = 100
...
Animation complete.
```

**Why This Output Occurs**
`start` fires before the first step. `step` fires for each animated property at each frame, providing the current value and Tween object .

### Real-World Cases

- **Sequential Animations**: Chaining animations with `complete` callbacks.
- **Loading Completion**: Hiding spinners after content loads.
- **Form Validation**: Triggering validation after a shake animation.
- **Multi-Element Coordination**: Using `done` to act after all items animate.

### References

- jQuery API — .animate() – https://api.jquery.com/animate/
- jQuery API — .promise() – https://api.jquery.com/promise/
- Stack Overflow — Difference between done and complete – https://stackoverflow.com/questions/17125787/

---

## Summary Comparison Table

| Concept | Key Parameter | Default | Purpose |
|---|---|---|---|
| `.animate()` | `properties` (required) | — | Core custom animation method |
| CSS Property | Numeric values | Pixels | What gets animated |
| Duration | Milliseconds or string | 400ms | How long animation takes |
| Easing | `"swing"` or `"linear"` | `"swing"` | Rate of change profile |
| Complete Callback | Function | None | Runs after animation |

---

## Important Notes on Version-Specific Behaviour

1. **Per-Property Easing**: Added in jQuery 1.4 .
2. **`done` Callback**: Added in jQuery 1.8 .
3. **Colour Animation**: Requires jQuery Color plugin .
4. **jQuery UI Extensions**: Extends `.animate()` with colour animation and additional easings .
5. **Global Effects Control**: `jQuery.fx.off = true` disables all animations, setting duration to 0 .

---

## References

- jQuery API — .animate() – https://api.jquery.com/animate/
- jQuery Learning Center — Custom Effects with .animate() – https://learn.jquery.com/effects/custom-effects/
- jQuery UI API — Easings – https://api.jqueryui.com/easings/
- jQuery API — .promise() – https://api.jquery.com/promise/
- Stack Overflow — Difference between done and complete – https://stackoverflow.com/questions/17125787/