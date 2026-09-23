# jQuery Modern Animation Considerations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Modern Animation Considerations refers to the ecosystem of browser-native animation technologies — CSS transitions, CSS animations, and the Web Animations API — that offer performance advantages, greater control, and better integration with the browser rendering pipeline compared to jQuery's JavaScript-driven animation engine.

**Technical Definition**
Modern browser animation technologies operate at different levels of the rendering pipeline. CSS transitions and CSS animations are declarative, defined in stylesheets or inline styles, and executed by the browser's compositor thread when animating compositor-only properties (`transform` and `opacity`). The Web Animations API (WAAPI) provides a JavaScript interface to the same underlying animation engine, offering programmatic control (play, pause, reverse, seek) while retaining the performance benefits of native execution. These technologies avoid the main-thread `setInterval`/`requestAnimationFrame` loops that jQuery's `fx` engine uses, reducing jank and improving battery life on mobile devices .

**Beginner-Friendly Explanation**
jQuery's animation engine works by repeatedly updating CSS values from JavaScript. Modern browsers have built-in animation engines that work faster and smoother because they can run on the GPU instead of the CPU. CSS transitions let you say "when this changes, animate it smoothly." CSS animations let you define complex multi-step sequences. The Web Animations API lets you control those same browser-native animations from JavaScript — play, pause, reverse, and seek — without sacrificing performance.

### Key Characteristics

- **Compositor-Only Properties**: `transform` and `opacity` can be animated entirely on the GPU, skipping layout and paint .
- **Declarative vs Imperative**: CSS transitions and animations are declarative (defined in CSS); WAAPI is imperative (controlled from JavaScript) .
- **Main-Thread Independence**: Native animations can run on the compositor thread, keeping the main thread free for JavaScript execution .
- **Promise-Based Completion**: WAAPI animations return promises, enabling clean sequencing without callbacks .
- **Accessibility Support**: `prefers-reduced-motion` media query is natively supported for respecting user motion preferences .

### Prerequisites

- Basic HTML, CSS, and JavaScript knowledge.
- Understanding of CSS properties and the box model.
- Familiarity with the rendering pipeline (Layout → Paint → Composite).
- Knowledge of JavaScript Promises (for WAAPI).

### Related Programming Areas

- **Rendering Pipeline**: Understanding layout, paint, and composite stages.
- **CSS Transforms and Opacity**: The two compositor-only properties.
- **Web Animations API**: JavaScript interface to the browser animation engine.
- **Performance Profiling**: Chrome DevTools Performance panel and Lighthouse.

### Core Concepts / Features

1. CSS Transitions
2. CSS Animations
3. Web Animations API
4. Performance Implications
5. GPU-Friendly Animation Patterns

---

## Core Concept 1: CSS Transitions

### Definitions

**Core Definition**
CSS Transitions enable smooth interpolation between two states of an element when a property value changes, such as on `:hover` or when a class is toggled.

**Technical Definition**
The `transition` CSS property is a shorthand for `transition-property`, `transition-duration`, `transition-timing-function`, and `transition-delay`. It defines which properties animate, how long the animation takes, the easing curve, and any delay before the animation starts. Transitions are triggered by changes to computed property values, whether from pseudo-classes (`:hover`, `:focus`), JavaScript class changes, or media query changes. They are widely supported across all modern browsers since September 2015 .

**Beginner-Friendly Explanation**
CSS transitions are like setting up a "smooth mode" for CSS changes. You tell the browser "when this property changes, don't jump — glide to the new value over this duration." They are perfect for hover effects, button states, and simple show/hide animations.

### Purposes

- To animate simple property changes (hover, focus, active states) without JavaScript.
- To provide smooth visual feedback for user interactions.
- To reduce JavaScript execution by delegating animation to the browser.
- To leverage GPU compositing for `transform` and `opacity` changes.
- To respect user motion preferences via `prefers-reduced-motion`.

### Syntax Rules and Structure

**Complete General Syntax**

```css
transition: property duration timing-function delay;
```

**Component Breakdown**

- `property` : The CSS property to animate (e.g., `transform`, `opacity`, `background-color`). Use `all` to animate all changing properties .
- `duration` : Time in seconds (`s`) or milliseconds (`ms`). Required for the transition to occur.
- `timing-function` : Easing curve (e.g., `ease`, `linear`, `ease-in-out`, `cubic-bezier()`). Default is `ease` .
- `delay` : Optional delay before the transition starts. Default is `0s`.

**Syntax Rules**

1. `duration` is the only required component; other components use defaults .
2. Multiple transitions can be comma-separated: `transition: transform 200ms, opacity 300ms`.
3. `transition-property: all` animates every changing property, which can cause unexpected performance issues.
4. Transitions only work between two states — they cannot create multi-step sequences.

**Constraints and Limitations**

- **Two-State Only**: Cannot define intermediate keyframes.
- **Display Transitions**: `display: none` cannot be transitioned directly; use `@starting-style` for entry animations or `transition-behavior: allow-discrete` .
- **Layout Properties**: Animating `width`, `height`, `top`, `left`, `margin`, or `padding` triggers layout recalculation, causing jank .

### Multiple Annotated Complete Code Examples

**Example 1: GPU-Friendly Hover Transition**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Transitions — GPU-Friendly Hover</title>
  <style>
    .card {
      padding: 20px;
      background: steelblue;
      color: white;
      /* Step 1: Define transition for transform only */
      transition: transform 200ms ease-out;
    }

    .card:hover {
      /* Step 2: Change transform — GPU composited */
      transform: translateY(-8px) scale(1.02);
    }
  </style>
</head>
<body>
  <div class="card">Hover me (smooth, GPU-accelerated)</div>
</body>
</html>
```

**Expected Output**
- Hovering over the card lifts it up and scales it slightly, smoothly over 200ms.

**Why This Output Occurs**
`transform` is a compositor-only property. The browser promotes the card to its own layer and animates it on the GPU without recalculating layout or repainting .

---

**Example 2: Transition with Multiple Properties**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Transitions — Multiple Properties</title>
  <style>
    .button {
      padding: 12px 24px;
      background: #3498db;
      color: white;
      border: none;
      /* Step 1: Different durations for different properties */
      transition: 
        transform 150ms ease-out,
        opacity 300ms linear,
        background-color 200ms ease;
    }

    .button:hover {
      transform: scale(1.05);
      opacity: 0.9;
      background: #2980b9;
    }
  </style>
</head>
<body>
  <button class="button">Hover Me</button>
</body>
</html>
```

**Expected Output**
- On hover, the button scales, fades slightly, and changes colour — each with its own timing.

**Why This Output Occurs**
Multiple transitions can be defined with independent durations and easings, allowing fine-grained control over each property's animation .

### Real-World Cases

- **Button Hover States**: `transform: scale(1.05)` for a subtle lift effect.
- **Card Elevation**: `transform: translateY(-4px)` combined with `box-shadow` transitions.
- **Focus Rings**: `opacity` transition for custom focus indicators.
- **Theme Switching**: Smooth `background-color` transitions when toggling dark mode.

### References

- MDN Web Docs — transition CSS property – https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/transition
- MDN Web Docs — @starting-style at-rule – https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@starting-style

---

## Core Concept 2: CSS Animations

### Definitions

**Core Definition**
CSS Animations enable multi-step, keyframe-based animations defined entirely in CSS, offering precise control over intermediate states without JavaScript.

**Technical Definition**
CSS Animations use the `@keyframes` at-rule to define named animation sequences. Each keyframe specifies styles at a percentage point in the animation timeline (e.g., `0%`, `50%`, `100%`). The `animation` shorthand property applies these keyframes to an element, controlling duration, timing function, delay, iteration count, direction, and fill mode. Like transitions, CSS animations of compositor-only properties can run on the GPU .

**Beginner-Friendly Explanation**
CSS Animations are like a movie script for your elements. You define what the element should look like at different points in time (0% = beginning, 100% = end), and the browser plays it. Unlike transitions, you can have as many steps as you want and repeat indefinitely.

### Purposes

- To create complex multi-step animations without JavaScript.
- To animate properties that transitions cannot (e.g., `offset-distance` for motion paths).
- To create looping or repeating animations (loading spinners, attention effects).
- To leverage GPU compositing for performance-critical animations.
- To support scroll-driven animations via `animation-timeline` .

### Syntax Rules and Structure

**Complete General Syntax**

```css
@keyframes animation-name {
  0%   { property: value; }
  50%  { property: value; }
  100% { property: value; }
}

.selector {
  animation: name duration timing-function delay iteration-count direction fill-mode;
}
```

**Component Breakdown**

- `@keyframes` : Defines the animation sequence with percentage selectors.
- `animation-name` : References the `@keyframes` name.
- `duration` : Time for one animation cycle.
- `timing-function` : Easing curve (can be per-keyframe).
- `iteration-count` : Number of cycles (`infinite` for endless loops).
- `direction` : `normal`, `reverse`, `alternate`, `alternate-reverse`.

**Syntax Rules**

1. At least two keyframes are required (0% and 100%) .
2. Keyframes can specify any animatable CSS property.
3. Per-keyframe easing is supported via `animation-timing-function` inside a keyframe.
4. `animation-fill-mode` controls styles before/after animation (`forwards`, `backwards`, `both`).
5. Scroll-driven animations use `animation-timeline` instead of time-based progression .

**Constraints and Limitations**

- **Layout Properties**: Animating `width`, `height`, `top`, `left` triggers layout and paint .
- **Animation Timeline Default**: `animation-timeline` resets to `auto` when using the `animation` shorthand .
- **Motion Path Limited**: `offset-path` supports 2D paths; 3D requires `transform` combinations .

### Multiple Annotated Complete Code Examples

**Example 1: Loading Spinner with CSS Animation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Animations — Spinner</title>
  <style>
    @keyframes spin {
      from { transform: rotate(0deg); }
      to   { transform: rotate(360deg); }
    }

    .spinner {
      width: 40px;
      height: 40px;
      border: 4px solid #f3f3f3;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      /* Step 1: Apply animation with infinite iteration */
      animation: spin 1s linear infinite;
    }
  </style>
</head>
<body>
  <div class="spinner"></div>
</body>
</html>
```

**Expected Output**
- A blue circular spinner rotates continuously at a constant speed.

**Why This Output Occurs**
`transform: rotate()` is GPU-composited. The `infinite` iteration count creates a seamless loop .

---

**Example 2: Motion Path Animation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CSS Animations — Motion Path</title>
  <style>
    @keyframes moveAlongPath {
      from { offset-distance: 0%; }
      to   { offset-distance: 100%; }
    }

    .track {
      width: 300px;
      height: 200px;
      background: #f0f0f0;
      position: relative;
    }

    .dot {
      width: 20px;
      height: 20px;
      background: red;
      border-radius: 50%;
      /* Step 1: Define the path */
      offset-path: path("M 20 100 Q 150 20, 280 100");
      /* Step 2: Animate along the path */
      animation: moveAlongPath 3s ease-in-out infinite alternate;
    }
  </style>
</head>
<body>
  <div class="track">
    <div class="dot"></div>
  </div>
</body>
</html>
```

**Expected Output**
- A red dot moves along a curved path from left to right and back, following the quadratic bezier curve defined in `offset-path`.

**Why This Output Occurs**
`offset-path` and `offset-distance` are part of the CSS Motion Path module. The browser interpolates the dot's position along the path over the animation duration .

### Real-World Cases

- **Loading Indicators**: Spinners, pulsing dots, progress bars.
- **Attention Effects**: Shaking, bouncing, or pulsing elements.
- **Motion Paths**: Animating elements along complex curves.
- **Scroll-Driven Effects**: Parallax and reveal animations tied to scroll position .

### References

- MDN Web Docs — CSS motion path – https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Motion_path
- MDN Web Docs — Scroll-driven animation timelines – https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations/Timelines

---

## Core Concept 3: Web Animations API

### Definitions

**Core Definition**
The Web Animations API (WAAPI) is a JavaScript interface that provides programmatic control over browser-native animations, bridging the gap between CSS animations and JavaScript timing control.

**Technical Definition**
The `Element.animate()` method creates an `Animation` object from keyframes and timing options. The returned object exposes playback controls (`play()`, `pause()`, `reverse()`, `finish()`, `cancel()`), a `playbackRate` property for dynamic speed control, a `currentTime` property for seeking, and a `finished` promise for sequencing. WAAPI animations use the same browser animation engine as CSS animations, meaning compositor-only properties still benefit from GPU acceleration .

**Beginner-Friendly Explanation**
WAAPI lets you create CSS-like animations from JavaScript and then control them like a video player. You can pause, play, reverse, slow down, speed up, or jump to any point in the animation. When the animation finishes, it gives you a promise you can chain.

### Purposes

- To create browser-native animations with JavaScript control.
- To dynamically adjust animation playback (speed, direction, seeking).
- To sequence animations using promises instead of callbacks.
- To cancel or clean up animations programmatically.
- To build animation controllers (sliders, playback buttons) with precision.

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
const animation = element.animate(keyframes, options);
```

**Component Breakdown**

- `keyframes` : An array of objects, each with CSS properties and optional `offset` (0–1) .
- `options` : Either a number (duration in milliseconds) or an object with `duration`, `easing`, `delay`, `iterations`, `direction`, `fill` .

**Syntax Rules**

1. `duration` is in **milliseconds**, not seconds .
2. `iterations` uses the JavaScript keyword `Infinity` for infinite loops, not `"infinite"` .
3. Default easing is `"linear"` (unlike CSS `ease`) .
4. Keyframes can specify `offset` to control timing (e.g., `offset: 0.3` for 30%) .
5. At least two keyframes are required; single-keyframe animations may throw errors .

**Constraints and Limitations**

- **Same Rules as CSS**: WAAPI can only animate properties that CSS can animate .
- **Main-Thread JavaScript**: The `animate()` call runs on the main thread; only the animation execution is offloaded.
- **Browser Support**: WAAPI is widely supported, but some features (e.g., `commitStyles()`) have limited support.

### Multiple Annotated Complete Code Examples

**Example 1: Basic WAAPI Animation**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WAAPI — Basic</title>
</head>
<body>
  <div id="box" style="width:100px; height:100px; background:steelblue;"></div>
  <button id="play">Play Animation</button>

  <script>
    document.getElementById("play").addEventListener("click", () => {
      const box = document.getElementById("box");

      // Step 1: Create animation with keyframes and timing
      const animation = box.animate([
        { transform: "translateX(0px)", opacity: 1 },
        { transform: "translateX(200px)", opacity: 0.5 }
      ], {
        duration: 1000,        // milliseconds
        easing: "ease-out",
        fill: "forwards"       // retain final state
      });

      // Step 2: Chain on completion via promise
      animation.finished.then(() => {
        console.log("Animation completed.");
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Play Animation" moves the box 200px right and fades it to 50% opacity over 1 second.
- Console logs `"Animation completed."` after finishing.

**Why This Output Occurs**
`Element.animate()` creates a browser-native animation. The `finished` promise resolves when the animation reaches its end .

---

**Example 2: Animation Controller with Playback Controls**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WAAPI — Playback Controls</title>
</head>
<body>
  <div id="circle" style="width:60px; height:60px; background:coral; border-radius:50%;"></div>
  <button id="play">Play</button>
  <button id="pause">Pause</button>
  <button id="reverse">Reverse</button>
  <button id="speed">2x Speed</button>

  <script>
    const circle = document.getElementById("circle");

    // Step 1: Create a looping animation, paused initially
    const anim = circle.animate([
      { transform: "translateX(0px)" },
      { transform: "translateX(300px)" }
    ], {
      duration: 2000,
      iterations: Infinity,     // JavaScript Infinity keyword
      direction: "alternate"
    });

    // Step 2: Control playback
    document.getElementById("play").onclick = () => anim.play();
    document.getElementById("pause").onclick = () => anim.pause();
    document.getElementById("reverse").onclick = () => anim.reverse();
    document.getElementById("speed").onclick = () => {
      anim.updatePlaybackRate(2); // smooth speed change
    };
  </script>
</body>
</html>
```

**Expected Output**
- The coral circle moves back and forth continuously.
- Play/Pause/Reverse buttons control playback.
- "2x Speed" doubles the animation speed smoothly.

**Why This Output Occurs**
WAAPI's `Animation` object provides `play()`, `pause()`, `reverse()`, and `updatePlaybackRate()` methods for full playback control .

### Real-World Cases

- **Animation Controllers**: Playback sliders, speed controls, seek bars.
- **Sequenced Modals**: Using `finished` promises to chain modal open → content fade → close.
- **Dynamic Speed**: Adjusting animation speed based on user interaction.
- **Cancellation**: Cleaning up animations when components are destroyed .

### References

- MDN Web Docs — Using the Web Animations API – https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API
- Telerik — The Web Animation API – https://www.telerik.com/blogs/web-animation-api

---

## Core Concept 4: Performance Implications

### Definitions

**Core Definition**
Performance implications in animation refer to the impact of animation choices on the browser's rendering pipeline, frame rate, and overall user experience.

**Technical Definition**
The browser rendering pipeline consists of five stages: JavaScript → Style → Layout → Paint → Composite. Animations that trigger Layout or Paint require the browser to recalculate element geometry and redraw pixels, consuming main-thread time and potentially causing jank (dropped frames). Animations that only trigger Composite — those using `transform` and `opacity` on promoted layers — can run at 60fps or higher on the compositor thread, independent of main-thread JavaScript execution .

**Beginner-Friendly Explanation**
Every animation causes the browser to do work. Some animations are "cheap" (the GPU can handle them alone), and some are "expensive" (the CPU has to recalculate the entire page layout). Cheap animations stay smooth; expensive ones cause stuttering, especially on mobile devices.

### Purposes

- To understand why some animations feel smooth and others feel janky.
- To choose properties that minimise main-thread work.
- To diagnose performance issues using browser DevTools.
- To prioritise GPU-accelerated animations for better mobile performance.
- To avoid layout thrashing and paint storms.

### Syntax Rules and Structure

**Rendering Pipeline Stages**

| Stage | Triggered By | Performance Cost |
|---|---|---|
| **Layout** | `width`, `height`, `top`, `left`, `margin`, `padding` | High (recalculates geometry) |
| **Paint** | `background-color`, `box-shadow`, `border-radius` | Medium (redraws pixels) |
| **Composite** | `transform`, `opacity` | Low (GPU handles) |

**Safe vs Unsafe Properties**

| GPU-Safe (Compositor-Only) | CPU-Expensive (Layout/Paint) |
|---|---|
| `transform` (translate, scale, rotate) | `width`, `height` |
| `opacity` | `top`, `left`, `right`, `bottom` |
| `filter` (with caution) | `margin`, `padding` |
| `clip-path` | `font-size`, `line-height`  |

**Syntax Rules**

1. Animate only `transform` and `opacity` for guaranteed GPU compositing .
2. Promote animated elements with `will-change: transform` or `transform: translateZ(0)` .
3. Avoid over-promoting elements — each layer consumes memory .
4. Use `prefers-reduced-motion` to disable animations for users who prefer reduced motion .

**Constraints and Limitations**

- **Layer Explosion**: Promoting too many elements creates memory pressure and can hurt performance .
- **Non-Composited Animations**: Animations on `width`, `height`, or `top` increase Cumulative Layout Shift (CLS) .
- **will-change Overuse**: Setting `will-change` on every element is counterproductive .

### Multiple Annotated Complete Code Examples

**Example 1: Good vs Bad Animation Pattern**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Performance — Good vs Bad</title>
  <style>
    /* BAD: Animating top triggers layout */
    .bad {
      position: relative;
      top: 0;
      transition: top 300ms;
    }
    .bad:hover {
      top: -10px; /* Layout recalculated every frame */
    }

    /* GOOD: Animating transform uses GPU */
    .good {
      transition: transform 300ms;
      will-change: transform; /* Hint browser to promote */
    }
    .good:hover {
      transform: translateY(-10px); /* Compositor only */
    }
  </style>
</head>
<body>
  <div class="bad" style="padding:20px; background:salmon; margin:10px;">
    Bad: animating top (janky)
  </div>
  <div class="good" style="padding:20px; background:lightgreen; margin:10px;">
    Good: animating transform (smooth)
  </div>
</body>
</html>
```

**Expected Output**
- Hovering the "Bad" box causes it to move upward but may stutter on slower devices.
- Hovering the "Good" box moves smoothly at 60fps.

**Why This Output Occurs**
Animating `top` triggers layout recalculation for every frame. Animating `transform` is handled by the compositor and skips layout and paint .

---

**Example 2: Using `will-change` Correctly**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Performance — will-change</title>
  <style>
    .card {
      padding: 20px;
      background: steelblue;
      color: white;
      /* Hint that transform will change */
      will-change: transform;
      transition: transform 250ms ease-out;
    }

    .card:hover {
      transform: scale(1.05) translateY(-4px);
    }
  </style>
</head>
<body>
  <div class="card">Hover me (will-change hint)</div>
</body>
</html>
```

**Expected Output**
- The card scales and lifts smoothly on hover with no jank.

**Why This Output Occurs**
`will-change: transform` tells the browser to promote the element to its own compositor layer before the animation starts, eliminating the promotion cost during the animation .

### Real-World Cases

- **Mobile Performance**: Prioritising `transform`/`opacity` for smooth 60fps on low-end devices .
- **CLS Reduction**: Using composited animations to avoid Cumulative Layout Shift .
- **Battery Life**: Reducing main-thread work to improve battery on mobile devices .
- **Scroll Performance**: Avoiding layout-triggering animations during scroll .

### References

- web.dev — Stick to Compositor-Only Properties – https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count
- Chrome for Developers — Non-Composited Animations – https://developer.chrome.com/docs/lighthouse/performance/non-composited-animations
- GitHub — Motion Performance Rules – https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/animation-motion-design/rules/motion-performance.md

---

## Core Concept 5: GPU-Friendly Animation Patterns

### Definitions

**Core Definition**
GPU-Friendly Animation Patterns are established techniques and property choices that ensure animations are handled by the compositor thread, minimising main-thread work and maximising frame rate.

**Technical Definition**
GPU-friendly patterns leverage compositor-only properties (`transform`, `opacity`, `filter`, `clip-path`) and browser hints (`will-change`, `translateZ`) to promote animated elements to their own compositor layers. The browser then interpolates these properties on the GPU without triggering Layout or Paint. Patterns include the FLIP technique for layout transitions, staggered animations to reduce simultaneous work, and Intersection Observer for pausing off-screen animations .

**Beginner-Friendly Explanation**
These patterns are "best practices" that keep animations running on the fast track. Instead of animating `width` (which makes the browser recalculate everything), you animate `transform: scaleX()` (which the GPU handles instantly). The FLIP technique lets you animate position changes using transforms even when the element needs to move to a new layout position.

### Purposes

- To ensure animations run at 60fps or higher on all devices.
- To minimise main-thread JavaScript execution during animation.
- To reduce battery drain on mobile devices.
- To maintain smooth scrolling and interaction responsiveness.
- To create professional-feeling micro-interactions.

### Syntax Rules and Structure

**Core GPU-Friendly Patterns**

**Pattern 1: Transform-Only Movement**
```css
/* Instead of animating left/top */
.element {
  transform: translate(x, y);
  transition: transform 200ms;
}
```

**Pattern 2: FLIP Technique for Layout Changes**
```javascript
// First: Record current position
const first = element.getBoundingClientRect();

// Last: Apply final layout change (no animation)
element.classList.add("moved");

// Invert: Calculate the difference
const last = element.getBoundingClientRect();
const deltaX = first.left - last.left;
const deltaY = first.top - last.top;

// Play: Animate from the delta to zero
element.animate([
  { transform: `translate(${deltaX}px, ${deltaY}px)` },
  { transform: "translate(0, 0)" }
], 300);
```

**Pattern 3: Will-Change Hint**
```css
.animated-element {
  will-change: transform, opacity;
}
```

**Pattern 4: Prefers-Reduced-Motion**
```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Pattern 5: Pause Off-Screen Animations**
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.style.animationPlayState = "running";
    } else {
      entry.target.style.animationPlayState = "paused";
    }
  });
});
```

**Syntax Rules**

1. Use `transform` for all positional changes (`translate`, `scale`, `rotate`).
2. Use `opacity` for fade effects.
3. Apply `will-change` only to elements that will actually animate.
4. Use FLIP for layout-position transitions that cannot be achieved with transforms alone .
5. Respect `prefers-reduced-motion` .
6. Pause animations when elements are off-screen .

**Constraints and Limitations**

- **FLIP Complexity**: Requires measuring before and after states; more code than simple transitions.
- **Layer Memory**: Each promoted layer consumes GPU memory .
- **Filter Caution**: `filter` can be GPU-accelerated but may trigger paint in some browsers .

### Multiple Annotated Complete Code Examples

**Example 1: FLIP Technique for Layout Transition**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GPU Patterns — FLIP</title>
  <style>
    .container {
      display: flex;
      gap: 10px;
      padding: 20px;
    }
    .item {
      width: 80px;
      height: 80px;
      background: steelblue;
      border-radius: 8px;
    }
    .item.expanded {
      width: 200px;
    }
  </style>
</head>
<body>
  <div class="container" id="container">
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
  </div>
  <button id="toggle">Toggle Layout</button>

  <script>
    document.getElementById("toggle").addEventListener("click", () => {
      const items = document.querySelectorAll(".item");

      items.forEach(item => {
        // FIRST: Record current position
        const first = item.getBoundingClientRect();

        // LAST: Apply new layout (no animation)
        item.classList.toggle("expanded");

        // INVERT: Calculate the change
        const last = item.getBoundingClientRect();
        const deltaX = first.left - last.left;

        // PLAY: Animate from delta to zero
        item.animate([
          { transform: `translateX(${deltaX}px)` },
          { transform: "translateX(0)" }
        ], {
          duration: 300,
          easing: "ease-out"
        });
      });
    });
  </script>
</body>
</html>
```

**Expected Output**
- Clicking "Toggle Layout" smoothly animates the items as they change width, using transforms instead of animating `width` directly.

**Why This Output Occurs**
The FLIP technique measures the before and after states, then uses `transform: translateX()` to create the illusion of layout animation while keeping the animation on the compositor .

---

**Example 2: Prefers-Reduced-Motion**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GPU Patterns — Reduced Motion</title>
  <style>
    .box {
      width: 100px;
      height: 100px;
      background: coral;
      transition: transform 500ms ease-out;
    }

    .box:hover {
      transform: scale(1.2) rotate(5deg);
    }

    /* Respect user motion preferences */
    @media (prefers-reduced-motion: reduce) {
      .box {
        transition: none;
      }
      .box:hover {
        transform: none;
      }
    }
  </style>
</head>
<body>
  <div class="box">Hover me</div>
  <p>If your OS has "Reduce Motion" enabled, the animation will be disabled.</p>
</body>
</html>
```

**Expected Output**
- With normal motion settings: the box scales and rotates on hover.
- With reduced motion enabled: no animation occurs.

**Why This Output Occurs**
The `prefers-reduced-motion` media query detects the user's accessibility preference and disables animations accordingly .

### Real-World Cases

- **Accordion Menus**: FLIP for smooth open/close without layout jank.
- **Image Galleries**: Transform-based hover effects for smooth scaling.
- **Scroll Animations**: Intersection Observer to trigger animations only when visible .
- **Accessibility**: Respecting `prefers-reduced-motion` for users with vestibular disorders .
- **Mobile Performance**: Staggering animations to avoid overwhelming low-end devices .

### References

- GitHub — Micro-interactions and Motion – https://github.com/Eneryleen/ai-web-design-codex/blob/main/07-aesthetics-and-trends/micro-interactions-and-motion.md
- GitHub — Animation Performance Optimization – https://github.com/dylantarre/animation-principles/blob/master/skills/12-by-problem-type/performance-optimization/SKILL.md
- Aurelia Docs — Animation Performance Optimization – https://docs.aurelia.io/developer-guides/animation/animation

---

## Summary Comparison Table

| Technology | Control | Performance | Best For |
|---|---|---|---|
| **CSS Transitions** | Declarative (CSS) | GPU for `transform`/`opacity` | Simple state changes (hover, focus) |
| **CSS Animations** | Declarative (CSS) | GPU for `transform`/`opacity` | Multi-step, looping, complex sequences |
| **Web Animations API** | Imperative (JS) | GPU for `transform`/`opacity` | Programmatic control, playback, sequencing |
| **jQuery `.animate()`** | Imperative (JS) | Main-thread (CPU) | Legacy projects, simple effects |

---

## Important Notes on Browser Support and Deprecation

1. **CSS Transitions**: Baseline widely available since September 2015 .
2. **Web Animations API**: Supported in all modern browsers (Chrome 36+, Firefox 48+, Safari 13.1+).
3. **`prefers-reduced-motion`**: Supported in all modern browsers since 2020.
4. **`@starting-style`**: Newer feature for entry animations; check browser support before production use .
5. **Scroll-Driven Animations**: `animation-timeline` is supported in Chrome 115+ and Edge 115+; check compatibility .
6. **jQuery `.animate()`**: Still supported but runs on the main thread; consider migrating to CSS or WAAPI for performance-critical animations.

---

## References

- MDN Web Docs — transition CSS property – https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/transition
- MDN Web Docs — @starting-style at-rule – https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@starting-style
- MDN Web Docs — CSS motion path – https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Motion_path
- MDN Web Docs — Scroll-driven animation timelines – https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations/Timelines
- MDN Web Docs — Using the Web Animations API – https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API
- MDN Web Docs — easing-function CSS type – https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/easing-function
- web.dev — Stick to Compositor-Only Properties – https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count
- Chrome for Developers — Non-Composited Animations – https://developer.chrome.com/docs/lighthouse/performance/non-composited-animations
- Telerik — The Web Animation API – https://www.telerik.com/blogs/web-animation-api
- GitHub — Motion Performance Rules – https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/animation-motion-design/rules/motion-performance.md
- GitHub — Micro-interactions and Motion – https://github.com/Eneryleen/ai-web-design-codex/blob/main/07-aesthetics-and-trends/micro-interactions-and-motion.md
- GitHub — Animation Performance Optimization – https://github.com/dylantarre/animation-principles/blob/master/skills/12-by-problem-type/performance-optimization/SKILL.md
- GitHub — GSAP Performance – https://github.com/greensock/gsap-skills/blob/main/skills/gsap-performance/SKILL.md
- Aurelia Docs — Animation Performance Optimization – https://docs.aurelia.io/developer-guides/animation/animation