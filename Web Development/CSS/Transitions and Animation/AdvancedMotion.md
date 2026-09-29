# CSS Advanced Motion — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Advanced Motion is the applied discipline of building sophisticated, performant, and accessible motion systems using modern CSS features — including staggered sequencing with custom properties, scroll-driven animations, motion paths, compositor-thread offloading, and user-preference-aware motion reduction.

**Technical Definition:** Advanced CSS motion integrates several specifications: the CSS Values and Units Module (custom properties, `calc()`, `sibling-index()`), the CSS Scroll-Driven Animations Module (`scroll()`, `view()`, `animation-timeline`, `animation-range`), the CSS Motion Path Module (`offset-path`, `offset-distance`, `offset-rotate`), the CSS Will Change Module (`will-change`), and the CSS Media Queries Level 5 `prefers-reduced-motion` feature. These features operate within the browser's rendering pipeline, where `transform` and `opacity` animations can be handled by the compositor thread, avoiding layout and paint work. The `prefers-reduced-motion` media query allows authors to respect user-configured accessibility preferences by reducing or replacing non-essential motion.

**Beginner-Friendly Explanation:** Basic CSS animations move things from point A to point B. Advanced motion takes this further: you can stagger animations across a list so items appear one after another, tie animations to scroll position so content reveals as the user scrolls, move elements along custom curved paths, keep animations smooth by letting the GPU handle them, and respect users who have asked their device to reduce motion. These techniques transform a website from "things move" to "things move intelligently."

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Sequenced timing** | Stagger animations using `calc()` and custom properties. |
| **Scroll-linked progress** | Animations driven by scroll position, not time. |
| **Custom paths** | Elements follow arbitrary shapes via `offset-path`. |
| **Compositor-only animation** | `transform` and `opacity` avoid layout and paint. |
| **GPU layer management** | `will-change` and `translateZ(0)` promote elements to layers. |
| **Accessibility-first** | `prefers-reduced-motion` respects user motion preferences. |

---

### Prerequisites

- **CSS Transitions and Animations** — the foundational motion modules.
- **CSS Custom Properties** — for dynamic, reusable motion values.
- **CSS Functions** — `calc()`, `var()`, and math operations.
- **The Browser Rendering Pipeline** — style, layout, paint, composite.
- **CSS Transforms** — the primary animatable property group.

---

### Related Programming Areas

- **Scroll-Driven Animations** — scrollytelling, progress indicators, reveal-on-scroll.
- **Motion Path** — logo animations, orbiting elements, path-following effects.
- **Web Performance** — Core Web Vitals, frame budgets, GPU memory.
- **Web Accessibility** — WCAG 2.3.3 Animation from Interactions.

---

### Core Concepts / Features

1. Motion Sequencing & Staggering
2. Scroll-Driven Animations
3. Motion Paths
4. Performance & Hardware Rendering
5. Motion Accessibility

---

## 1. Motion Sequencing & Staggering: Choreographing Compound Timelines

### Definitions

**Core Definition:** Motion sequencing and staggering is the technique of offsetting the start times of multiple animations so they play in a coordinated sequence rather than simultaneously, creating a cascading or wave-like effect.

**Technical Definition:** Staggered motion is achieved by computing a unique `animation-delay` for each element in a group. The delay is calculated using `calc()` combined with a custom property that represents the element's index or position. Modern CSS provides the `sibling-index()` function, which returns an element's 1-based position among its siblings, and `sibling-count()`, which returns the total number of siblings. These functions allow a single CSS rule to generate a staggered cascade for any number of items without `:nth-child()` selectors or JavaScript. The formula is typically `animation-delay: calc(sibling-index() * <delay-per-item>)`.

**Beginner-Friendly Explanation:** Imagine a row of cards that should appear one after another, like a wave. Instead of setting a manual delay on each card, you use a formula: each card's delay is its position number multiplied by a fixed amount. The first card has a delay of 0ms, the second 100ms, the third 200ms, and so on. The `sibling-index()` function makes this easy because it automatically gives each element its position number.

---

### Purposes

- To create cascading reveals where items appear one after another.
- To choreograph compound timelines where multiple animations are sequenced.
- To add polish and intentionality to list and grid animations.
- To eliminate repetitive `:nth-child()` rules for staggered delays.
- To enable dynamic staggering for lists of any length.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Using sibling-index() */
.item {
    animation-delay: calc(sibling-index() * <delay-per-item>);
}

/* Using a custom property (--i) set in HTML or via nth-child */
.item {
    animation-delay: calc(var(--i) * <delay-per-item>);
}

/* Using sibling-count() for reverse staggering */
.item {
    animation-delay: calc((sibling-count() - sibling-index()) * <delay-per-item>);
}
```

#### Component Breakdown

| Function / Property | Description | Example |
|---|---|---|
| `sibling-index()` | Returns the 1-based index of the element among its siblings. | `sibling-index()` → 1, 2, 3… |
| `sibling-count()` | Returns the total number of siblings. | `sibling-count()` → 10 |
| `--i` (custom property) | A manually assigned index per element. | `--i: 0; --i: 1; --i: 2;` |
| `calc()` | Performs arithmetic with mixed units. | `calc(var(--i) * 100ms)` |

#### Staggering Patterns

| Pattern | Formula | Effect |
|---|---|---|
| Forward stagger | `calc(sibling-index() * 100ms)` | First item first, last item last. |
| Reverse stagger | `calc((sibling-count() - sibling-index()) * 100ms)` | Last item first, first item last. |
| From centre | `calc(abs(sibling-index() - (sibling-count() / 2)) * 100ms)` | Centre items first. |

#### Syntax Rules

1. `sibling-index()` is 1-based, so the first element has an index of 1.
2. To start delays at 0, subtract 1: `calc((sibling-index() - 1) * 100ms)`.
3. `sibling-count()` returns the total number of siblings including the element itself.
4. Custom properties can be set inline (`style="--i: 3"`) or via `:nth-child()`.
5. The delay-per-item value can be a length of time (`100ms`) or a custom property.
6. Browser support for `sibling-index()` and `sibling-count()` is emerging (Chrome 138+, Safari 18.4+).

#### Constraints and Limitations

- **Browser support** — `sibling-index()` and `sibling-count()` are relatively new; check compatibility.
- **Custom property fallback** — for older browsers, use `:nth-child()` to set `--i` manually.
- **Accessibility** — staggered animations should be gated behind `prefers-reduced-motion`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Staggered Card Reveal with `sibling-index()`

**HTML File (`stagger.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Staggered Card Reveal</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="stagger.css">
</head>
<body>
    <div class="card-grid">
        <div class="card">Card 1</div>
        <div class="card">Card 2</div>
        <div class="card">Card 3</div>
        <div class="card">Card 4</div>
        <div class="card">Card 5</div>
    </div>
</body>
</html>
```

**CSS File (`stagger.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 16px;
    max-width: 700px;
    margin: 0 auto;
}

@keyframes slide-up {
    from {
        opacity: 0;
        transform: translateY(30px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.card {
    padding: 40px 20px;
    background-color: #3498db;
    color: white;
    border-radius: 12px;
    text-align: center;
    font-weight: bold;

    /* Apply the animation */
    animation: slide-up 500ms ease-out both;

    /* Stagger: each card delays by 100ms × its index */
    animation-delay: calc((sibling-index() - 1) * 100ms);
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `stagger.html`.
3. Save the CSS code as `stagger.css` in the same folder.
4. Open `stagger.html` in a modern browser that supports `sibling-index()`.
5. Observe the cards revealing one after another in a cascading sequence.

**Expected Output:** Five cards that slide up and fade in sequentially, with each card starting 100ms after the previous one.

**Why This Works:** The `sibling-index()` function returns 1 for the first card, 2 for the second, and so on. Subtracting 1 makes the first delay 0ms. Multiplying by 100ms gives each card a unique delay. The `animation-delay` property applies the computed delay, creating the cascade.

---

#### Example 2: Custom Property Fallback for Older Browsers

**HTML File (`stagger-fallback.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stagger Fallback</title>
    <link rel="stylesheet" href="stagger-fallback.css">
</head>
<body>
    <ul class="list">
        <li style="--i: 0">Item 1</li>
        <li style="--i: 1">Item 2</li>
        <li style="--i: 2">Item 3</li>
        <li style="--i: 3">Item 4</li>
    </ul>
</body>
</html>
```

**CSS File (`stagger-fallback.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.list {
    list-style: none;
    padding: 0;
    margin: 0 auto;
    max-width: 400px;
}

@keyframes fade-in {
    from { opacity: 0; transform: translateX(-20px); }
    to   { opacity: 1; transform: translateX(0); }
}

.list li {
    padding: 16px 24px;
    background-color: #006064;
    color: white;
    border-radius: 8px;
    margin-bottom: 8px;
    font-weight: bold;

    animation: fade-in 400ms ease-out both;
    /* Use the --i custom property set inline */
    animation-delay: calc(var(--i) * 80ms);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `stagger-fallback.html` and CSS as `stagger-fallback.css`.
2. Open in any browser.
3. Observe the list items sliding in from the left, one after another.

**Expected Output:** Four list items that fade in and slide from the left with an 80ms stagger between each.

**Why This Works:** The `--i` custom property is set inline on each `<li>` element. The `calc(var(--i) * 80ms)` formula multiplies the index by 80ms to produce a unique delay for each item. This approach works in all browsers that support custom properties.

---

### Real-World Cases

- **Dashboard widgets:** Staggered entrance of KPI cards on page load.
- **Navigation menus:** Sequential reveal of menu items on mobile.
- **Image galleries:** Cascading fade-in of gallery items as they load.
- **Onboarding steps:** Step-by-step reveal of form fields or instructions.

---

## 2. Scroll-Driven Animations: Connecting Animations to Layout Scrolling

### Definitions

**Core Definition:** Scroll-driven animations are animations whose progress is controlled by the scroll position of a scroll container rather than by elapsed time. The animation advances as the user scrolls, and reverses when they scroll back.

**Technical Definition:** CSS scroll-driven animations, defined in the CSS Scroll-Driven Animations Module, allow authors to animate property values based on a progression along a scroll-based timeline instead of the default time-based document timeline. Two types of timelines exist: **scroll progress timelines**, where the scroll position of a scroll container (scroller) is mapped to progress (0% at the start, 100% at the end), and **view progress timelines**, where the visibility of an element (the subject) within a scroller is tracked as progress. These timelines are applied to an element using the `animation-timeline` property, with the `scroll()` and `view()` functions creating anonymous timelines. The `animation-range` property (and its `animation-range-start` / `animation-range-end` longhands) adjusts where along the timeline the animation begins and ends, using named ranges such as `entry`, `exit`, `cover`, and `contain`.

**Beginner-Friendly Explanation:** Normally, an animation runs on a timer — it starts, plays for its duration, and ends. Scroll-driven animations replace the timer with the scroll bar. As you scroll down, the animation progresses. Scroll back up and it reverses. This is perfect for effects like content that fades in as it enters the viewport, progress bars that fill as you read, or images that zoom as you scroll. You connect an animation to scroll with `animation-timeline: scroll()` or `view()`, and you can control exactly which part of the scroll triggers the animation with `animation-range`.

---

### Purposes

- To create scrollytelling experiences where content reveals as the user scrolls.
- To animate reading progress indicators.
- To fade, slide, or zoom elements as they enter and exit the viewport.
- To replace JavaScript scroll listeners with native CSS.
- To synchronise animation progress directly with scroll position.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Scroll progress timeline */
.animated-element {
    animation-timeline: scroll();
    animation-range: 0% 100%;
}

/* View progress timeline */
.reveal-element {
    animation-timeline: view();
    animation-range: entry 0% cover 40%;
}

/* Named timeline */
.scroller {
    scroll-timeline: --my-timeline block;
}
.animated {
    animation-timeline: --my-timeline;
}
```

#### Component Breakdown

| Property / Function | Description | Values |
|---|---|---|
| `animation-timeline` | Connects an animation to a timeline. | `scroll()`, `view()`, `<custom-ident>` |
| `scroll()` | Creates an anonymous scroll progress timeline. | `scroll(<axis>)` — `block`, `inline`, `x`, `y` |
| `view()` | Creates an anonymous view progress timeline. | `view(<axis>)` — `block`, `inline`, `x`, `y` |
| `animation-range` | Sets the start and end of the animation's range. | `<timeline-range-name>` + `<length-percentage>` |

#### Timeline Range Names

| Range Name | Description |
|---|---|
| `cover` | The full range: from the element first touching the viewport to it leaving completely. |
| `contain` | The range where the element is fully contained within the viewport. |
| `entry` | The range from the element first entering to it being fully visible. |
| `exit` | The range from the element starting to leave to it being fully gone. |

#### Syntax Rules

1. `animation-timeline: scroll()` creates an anonymous timeline tied to the nearest ancestor scroll container.
2. `animation-timeline: view()` creates an anonymous timeline tied to the element's visibility within the nearest scroll container.
3. `animation-range: entry 25% cover 50%` starts the animation at 25% into the `entry` range and ends at 50% into the `cover` range.
4. The `animation-duration` should be set to `auto` when using scroll-driven timelines.
5. The `animation-range` shorthand must be declared after the `animation` shorthand to avoid being reset.
6. Scroll-driven animations are supported in Chrome/Edge 115+, Firefox 144+, and Safari (with polyfill).

#### Constraints and Limitations

- **Browser support** — limited; Chrome and Edge have the most complete support; Safari requires a polyfill.
- **Performance** — scroll-driven animations on `transform` and `opacity` are compositor-friendly.
- **Fallback** — browsers without support show the static styles; use `@supports` to provide fallbacks.
- **`animation-range` reset** — the `animation` shorthand resets `animation-range` to `normal`; declare `animation-range` after `animation`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Content Reveal on Scroll with `view()`

**HTML File (`scroll-reveal.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scroll Reveal</title>
    <link rel="stylesheet" href="scroll-reveal.css">
</head>
<body>
    <div class="hero">Scroll Down</div>
    <div class="content">
        <div class="reveal-card">
            <h2>First Card</h2>
            <p>This card fades in as it enters the viewport.</p>
        </div>
        <div class="reveal-card">
            <h2>Second Card</h2>
            <p>This one fades in too.</p>
        </div>
        <div class="reveal-card">
            <h2>Third Card</h2>
            <p>And this one.</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`scroll-reveal.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f5f5f5;
}

.hero {
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #006064;
    color: white;
    font-size: 2rem;
    font-weight: bold;
}

.content {
    padding: 60px 20px;
    max-width: 600px;
    margin: 0 auto;
}

@keyframes reveal {
    from {
        opacity: 0;
        transform: translateY(40px) scale(0.95);
    }
    to {
        opacity: 1;
        transform: translateY(0) scale(1);
    }
}

.reveal-card {
    background-color: white;
    border-radius: 12px;
    padding: 30px;
    margin-bottom: 60px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);

    /* Apply the animation with a view progress timeline */
    animation: reveal 1s linear both;
    animation-timeline: view();
    /* Start when the element enters, finish when it covers 40% */
    animation-range: entry 0% cover 40%;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `scroll-reveal.html` and CSS as `scroll-reveal.css`.
2. Open in a browser that supports scroll-driven animations (Chrome 115+).
3. Scroll down: the hero fills the viewport, and as you scroll, each card fades in and slides up as it enters the viewport.

**Expected Output:** Cards that fade in and slide up as they enter the viewport, with the animation progress tied directly to scroll position.

**Why This Works:** The `animation-timeline: view()` creates an anonymous view progress timeline for each card. The `animation-range: entry 0% cover 40%` specifies that the animation starts when the card begins to enter the viewport and completes when the card has covered 40% of the viewport. The `animation-duration` is implicitly `auto` for scroll-driven timelines.

---

#### Example 2: Reading Progress Bar with `scroll()`

**HTML File (`progress-bar.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reading Progress</title>
    <link rel="stylesheet" href="progress-bar.css">
</head>
<body>
    <div class="progress-bar"></div>
    <article>
        <h1>Article Title</h1>
        <p>Scroll to see the progress bar fill as you read.</p>
        <div class="spacer"></div>
        <div class="spacer"></div>
        <div class="spacer"></div>
        <div class="spacer"></div>
        <div class="spacer"></div>
    </article>
</body>
</html>
```

**CSS File (`progress-bar.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f5f5f5;
}

.progress-bar {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 6px;
    background-color: #3498db;
    transform-origin: left;
    transform: scaleX(0);

    /* Scale the bar based on scroll progress */
    animation: grow linear both;
    animation-timeline: scroll();
}

@keyframes grow {
    from { transform: scaleX(0); }
    to   { transform: scaleX(1); }
}

article {
    max-width: 600px;
    margin: 0 auto;
    padding: 40px 20px;
}

.spacer {
    height: 300px;
    background: linear-gradient(#e0f7fa, #b2ebf2);
    border-radius: 8px;
    margin: 20px 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `progress-bar.html` and CSS as `progress-bar.css`.
2. Open in a modern browser.
3. Scroll down. Observe the progress bar at the top filling from left to right as you scroll.

**Expected Output:** A blue progress bar at the top of the page that fills proportionally to the scroll position.

**Why This Works:** The `animation-timeline: scroll()` creates an anonymous scroll progress timeline tied to the document scroller. The `animation: grow linear both` animates the `transform: scaleX()` from 0 to 1. Because the timeline is tied to scroll position, the bar's width grows as the user scrolls down and shrinks as they scroll back up.

---

### Real-World Cases

- **Scrollytelling articles:** Content reveals as the user scrolls through a narrative.
- **Reading progress indicators:** A thin bar at the top of the page showing scroll progress.
- **Image galleries:** Images fade and scale as they enter the viewport.
- **Product pages:** Features highlight as the user scrolls through sections.

---

## 3. Motion Paths: Steering Elements Along Arbitrary Vectors

### Definitions

**Core Definition:** CSS Motion Path is a CSS module that allows authors to animate any graphical object along a custom path — a shape of any kind — rather than along straight lines or simple arcs.

**Technical Definition:** The CSS Motion Path Module Level 1 provides offset transforms: transforms that align a point on an element to an offset distance along an offset path, optionally rotating the transformed element to follow the path direction. The `offset-path` property specifies the path for the element to follow, accepting shape functions such as `path()`, `circle()`, `ellipse()`, `inset()`, and `ray()`. The `offset-distance` property specifies how far along the path the element is positioned, expressed as a `<length-percentage>`. The `offset-rotate` property controls whether the element rotates to follow the path direction. The `offset-anchor` property specifies which point on the element is aligned to the path, and `offset-position` defines the starting position when `offset-path` is `none`.

**Beginner-Friendly Explanation:** Normally, if you want an element to move, you move it in a straight line with `translate()`. Motion path lets you define a curvy path — like a circle, an S-curve, or a heart shape — and the element follows that path. You define the path with `offset-path`, control how far along the path the element is with `offset-distance`, and decide whether the element should rotate to face the direction of travel with `offset-rotate`.

---

### Purposes

- To animate elements along custom curved paths.
- To create orbiting or path-following effects.
- To build engaging logo animations and loading indicators.
- To position elements using polar coordinates.
- To add motion that follows the natural contours of a design.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    offset-path: path("M 0 0 L 100 100") | circle(50%) | ellipse(50% 30%) | ray(45deg) | inset(10px);
    offset-distance: <length-percentage>;
    offset-rotate: auto | <angle>;
    offset-anchor: auto | <position>;
    offset-position: auto | <position>;
    offset: <offset-position>? <offset-path> <offset-distance>? <offset-rotate>?;
}
```

#### Component Breakdown

| Property | Description | Values |
|---|---|---|
| `offset-path` | Defines the path to follow. | `path()`, `circle()`, `ellipse()`, `inset()`, `ray()` |
| `offset-distance` | How far along the path the element is. | `<length-percentage>` |
| `offset-rotate` | Whether the element rotates with the path. | `auto`, `<angle>`, `reverse` |
| `offset-anchor` | The point on the element aligned to the path. | `auto`, `<position>` |
| `offset-position` | Starting position when no path is defined. | `auto`, `<position>` |

#### Syntax Rules

1. `offset-path: path("...")` uses SVG path syntax.
2. `offset-distance` accepts percentages relative to the total path length.
3. `offset-rotate: auto` rotates the element to follow the path direction.
4. `offset-rotate: reverse` rotates the element opposite to the path direction.
5. The `offset` shorthand combines all offset properties.
6. Motion path is supported in all modern browsers (Baseline since August 2023).

#### Constraints and Limitations

- **SVG path syntax** — `path()` requires learning SVG path commands (`M`, `L`, `C`, `A`, `Z`).
- **Performance** — offset-path animations are compositor-friendly when animating `offset-distance`.
- **Browser support** — all modern browsers support motion path, but older browsers do not.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Circular Motion Path

**HTML File (`motion-path.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Motion Path</title>
    <link rel="stylesheet" href="motion-path.css">
</head>
<body>
    <div class="scene">
        <div class="orbiter"></div>
    </div>
</body>
</html>
```

**CSS File (`motion-path.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
}

.scene {
    position: relative;
    width: 400px;
    height: 400px;
    background-color: #e0f7fa;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
}

@keyframes orbit {
    from { offset-distance: 0%; }
    to   { offset-distance: 100%; }
}

.orbiter {
    width: 60px;
    height: 60px;
    background-color: #e74c3c;
    border-radius: 50%;
    /* Define the circular path */
    offset-path: circle(150px at 50% 50%);
    /* Rotate the element to follow the path */
    offset-rotate: auto;
    /* Animate the offset distance */
    animation: orbit 3s linear infinite;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `motion-path.html` and CSS as `motion-path.css`.
2. Open in a browser.
3. Observe the red circle orbiting around the centre of the scene.

**Expected Output:** A red circle that travels along a circular path, completing one full orbit every 3 seconds.

**Why This Works:** The `offset-path: circle(150px at 50% 50%)` defines a circular path with a radius of 150px centered in the scene. The `animation: orbit 3s linear infinite` animates `offset-distance` from 0% to 100%, moving the element along the path. The `offset-rotate: auto` rotates the element to follow the path direction.

---

#### Example 2: SVG Path with `offset-path`

**HTML File (`svg-path.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SVG Motion Path</title>
    <link rel="stylesheet" href="svg-path.css">
</head>
<body>
    <div class="scene">
        <svg class="path-visual" viewBox="0 0 400 200">
            <path d="M 20 180 C 100 20, 300 20, 380 180" 
                  fill="none" stroke="#b2ebf2" stroke-width="4" stroke-dasharray="8 4"/>
        </svg>
        <div class="ball"></div>
    </div>
</body>
</html>
```

**CSS File (`svg-path.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
}

.scene {
    position: relative;
    width: 400px;
    height: 200px;
}

.path-visual {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

@keyframes follow-path {
    from { offset-distance: 0%; }
    to   { offset-distance: 100%; }
}

.ball {
    position: absolute;
    width: 24px;
    height: 24px;
    background-color: #e74c3c;
    border-radius: 50%;

    /* Define the SVG path */
    offset-path: path("M 20 180 C 100 20, 300 20, 380 180");
    offset-rotate: auto;
    animation: follow-path 2s ease-in-out infinite alternate;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `svg-path.html` and CSS as `svg-path.css`.
2. Open in a browser.
3. Observe the red ball following the dashed curve from left to right and back.

**Expected Output:** A red ball that travels along an S-curve defined by the SVG path, reversing direction on each iteration.

**Why This Works:** The `offset-path: path("M 20 180 C 100 20, 300 20, 380 180")` defines an S-curve using SVG path commands. The `animation: follow-path 2s ease-in-out infinite alternate` animates `offset-distance` from 0% to 100% and back. The dashed visual path is drawn with an SVG `<path>` element for reference.

---

### Real-World Cases

- **Logo animations:** A logo icon following a curved path on page load.
- **Loading indicators:** A spinner that traces a custom path.
- **Product showcases:** A product image orbiting around a central point.
- **Interactive maps:** A marker following a route line.

---

## 4. Performance & Hardware Rendering: Offloading Animation Loops to the Compositor Thread

### Definitions

**Core Definition:** Compositor-thread animation is the browser's ability to handle `transform` and `opacity` changes entirely on the GPU compositor thread, without involving the main thread's layout or paint stages. This keeps animations smooth even when the main thread is busy.

**Technical Definition:** The browser rendering pipeline consists of four stages: style, layout, paint, and composite. Animations on `transform` and `opacity` can skip the layout and paint stages entirely, allowing the compositor thread to interpolate the values and update the screen at 60fps (or higher) without blocking on the main thread. To enable this, the animated element must be promoted to its own compositor layer, typically via `will-change: transform` or `transform: translateZ(0)`. Each promoted layer consumes GPU memory and requires management, so over-promotion (e.g., applying `will-change` to every element) degrades performance rather than improving it. The `will-change` property should be applied just before an animation begins and removed after it completes.

**Beginner-Friendly Explanation:** The main thread is where JavaScript runs, and it is often busy. The compositor thread is a separate background thread that can move and fade layers of pixels using the GPU. When you animate `transform` and `opacity`, the browser can hand the work off to the compositor thread, which keeps the animation smooth even if the main thread is busy. The `will-change` property tells the browser to prepare a GPU layer ahead of time so the animation starts smoothly. But you should use it sparingly — each layer takes up video memory, and too many layers will actually slow things down.

---

### Purposes

- To keep animations smooth at 60fps even when the main thread is busy.
- To avoid expensive layout and paint work during animation.
- To enable GPU acceleration for transform and opacity animations.
- To manage layer promotion responsibly to avoid GPU memory exhaustion.
- To profile and debug animation performance using browser DevTools.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    /* Promote to compositor layer */
    will-change: transform;
    /* Or for older browsers */
    transform: translateZ(0);
}

/* Remove after animation */
selector.animation-complete {
    will-change: auto;
}
```

#### Component Breakdown

| Property / Value | Description |
|---|---|
| `will-change: transform` | Hints that the element will animate its transform. |
| `will-change: opacity` | Hints that the element will animate its opacity. |
| `transform: translateZ(0)` | Forces layer promotion in older browsers. |
| `will-change: auto` | Releases the layer after animation. |

#### Performance Budget

| Metric | Target |
|---|---|
| Frame budget | 16.66ms per frame (60fps) |
| Compositor time | 4–5ms per frame |
| Maximum promoted layers | 3–4 per view |

#### Syntax Rules

1. Only `transform` and `opacity` are compositor-only properties.
2. `will-change` should be applied just before the animation starts.
3. Remove `will-change` after the animation completes to free GPU memory.
4. Never use `* { will-change: transform; }` — this creates a layer for every element.
5. Use `transform: translateZ(0)` as a fallback for browsers that do not support `will-change`.
6. Each compositor layer consumes VRAM proportional to its size.
7. Use Chrome DevTools Layers panel to monitor layer count and compositor time.

#### Constraints and Limitations

- **GPU memory** — excessive layers cause memory pressure and degrade performance.
- **Layer explosion** — applying `will-change` to every element creates hundreds of layers.
- **Static elements** — promoting static elements wastes memory with no benefit.
- **Removal required** — failing to remove `will-change` after animation keeps the layer alive indefinitely.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct `will-change` Usage

**HTML File (`performance.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Performance Optimisation</title>
    <link rel="stylesheet" href="performance.css">
</head>
<body>
    <div class="card" id="card">Hover to animate</div>

    <script>
        const card = document.getElementById('card');
        card.addEventListener('mouseenter', () => {
            card.style.willChange = 'transform';
        });
        card.addEventListener('transitionend', () => {
            card.style.willChange = 'auto';
        });
    </script>
</body>
</html>
```

**CSS File (`performance.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.card {
    width: 200px;
    padding: 40px;
    background-color: #3498db;
    color: white;
    border-radius: 16px;
    text-align: center;
    font-weight: bold;
    cursor: pointer;

    /* Only transition compositor-friendly properties */
    transition: transform 300ms ease-out,
                opacity 300ms ease-out;
}

.card:hover {
    transform: translateY(-10px) scale(1.05);
    opacity: 0.9;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `performance.html` and CSS as `performance.css`.
2. Open in a browser.
3. Hover over the card. The JavaScript applies `will-change: transform` on mouseenter and removes it on `transitionend`.
4. Open DevTools → More Tools → Layers. Observe that a compositor layer is created only during the hover animation.

**Expected Output:** A card that lifts and scales on hover, with a compositor layer created only for the duration of the animation.

**Why This Works:** The JavaScript applies `will-change: transform` when the mouse enters the card, promoting it to a compositor layer just before the animation begins. On `transitionend`, the `will-change` is set back to `auto`, releasing the layer. This avoids the memory cost of a permanent layer while ensuring the animation runs on the compositor thread.

---

### Real-World Cases

- **Hover animations on cards and buttons:** Applying `will-change` on hover only.
- **Scroll-triggered reveals:** Adding `will-change` when elements approach the viewport.
- **Modal dialogs:** Applying `will-change` just before opening and removing after closing.
- **Parallax effects:** Promoting only the moving layers, not the entire page.

---

## 5. Motion Accessibility: Honoring User Device Criteria with `prefers-reduced-motion`

### Definitions

**Core Definition:** `prefers-reduced-motion` is a CSS media feature that detects whether the user has enabled a device setting to minimise non-essential motion. It allows authors to provide a reduced-motion experience for users with vestibular disorders, motion sensitivity, or personal preferences.

**Technical Definition:** The `prefers-reduced-motion` CSS media feature is used to detect if a user has enabled a setting on their device to minimise the amount of non-essential motion. It has two values: `no-preference` (the user has made no preference known) and `reduce` (the user has enabled reduced motion on their device). When `reduce` is active, authors should remove or replace motion-based animations. WCAG Technique C39 specifies that using `prefers-reduced-motion` to prevent motion satisfies Success Criterion 2.3.3 (Animation from Interactions). The inverse approach — defining static styles by default and adding motion only inside `@media (prefers-reduced-motion: no-preference)` — is recommended for progressive enhancement.

**Beginner-Friendly Explanation:** Some people experience nausea, dizziness, or discomfort when they see certain kinds of animation — especially large movements, scaling, or parallax effects. Operating systems provide a "reduce motion" setting for these users. The `prefers-reduced-motion` media query lets your website respect that setting. If the user has enabled reduced motion, you can disable decorative animations, reduce their intensity, or replace them with simple opacity fades. This is not just good practice — it is a WCAG accessibility requirement.

---

### Purposes

- To respect users who have enabled reduced motion on their device.
- To prevent vestibular discomfort caused by large or unexpected motion.
- To comply with WCAG 2.3.3 (Animation from Interactions).
- To provide a more comfortable experience for motion-sensitive users.
- To progressively enhance motion for users who have not requested reduction.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Approach 1: Disable motion for reduce */
@media (prefers-reduced-motion: reduce) {
    .animated-element {
        animation: none;
        transition: none;
    }
}

/* Approach 2: Only add motion when no preference */
@media (prefers-reduced-motion: no-preference) {
    .animated-element {
        animation: slide-in 500ms ease-out;
    }
}

/* Reduce, don't remove: keep opacity, remove movement */
@media (prefers-reduced-motion: reduce) {
    .animated-element {
        animation: fade-in 200ms ease-out;
    }
}
```

#### Component Breakdown

| Value | Description | Behaviour |
|---|---|---|
| `no-preference` | User has made no preference known. | Motion is allowed. |
| `reduce` | User has enabled reduced motion. | Motion should be reduced or removed. |

#### Syntax Rules

1. `@media (prefers-reduced-motion: reduce)` targets users who want reduced motion.
2. `@media (prefers-reduced-motion: no-preference)` targets users who have no preference (motion is allowed).
3. The inverse approach (static by default, motion only in `no-preference`) is recommended for progressive enhancement.
4. Do not remove all animations — keep essential motion and replace large movements with opacity fades.
5. The value is `reduce`, not `none`.
6. `prefers-reduced-motion` is Baseline widely available (since January 2020).

#### Constraints and Limitations

- **Not a guarantee** — the media query reflects a preference, not a medical diagnosis.
- **Essential motion** — some motion may be essential to functionality (e.g., a loading spinner); consider providing a reduced-motion alternative.
- **JavaScript detection** — `window.matchMedia('(prefers-reduced-motion: reduce)')` allows JavaScript to detect the preference.
- **Testing** — use browser DevTools to simulate the reduced motion preference.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Reduced Motion for a Scroll Reveal Animation

**HTML File (`reduced-motion.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reduced Motion</title>
    <link rel="stylesheet" href="reduced-motion.css">
</head>
<body>
    <div class="hero">Scroll Down</div>
    <div class="content">
        <div class="reveal-card">
            <h2>Accessible Card</h2>
            <p>This card respects your reduced motion preference.</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`reduced-motion.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f5f5f5;
}

.hero {
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #006064;
    color: white;
    font-size: 2rem;
    font-weight: bold;
}

.content {
    padding: 60px 20px;
    max-width: 600px;
    margin: 0 auto;
}

.reveal-card {
    background-color: white;
    border-radius: 12px;
    padding: 30px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);

    /* Default: motion allowed */
    animation: slide-up 800ms ease-out both;
    animation-timeline: view();
    animation-range: entry 0% cover 40%;
}

@keyframes slide-up {
    from {
        opacity: 0;
        transform: translateY(40px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Reduce motion: remove movement, keep fade */
@media (prefers-reduced-motion: reduce) {
    .reveal-card {
        animation: fade-in 200ms ease-out both;
    }

    @keyframes fade-in {
        from { opacity: 0; }
        to   { opacity: 1; }
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `reduced-motion.html` and CSS as `reduced-motion.css`.
2. Open in a browser.
3. Enable "Reduce Motion" in your OS accessibility settings.
4. Reload the page. Observe that the card fades in instead of sliding up.

**Expected Output:** For users with reduced motion enabled, the card fades in with a simple opacity transition instead of sliding up. For all other users, the card slides up and fades in.

**Why This Works:** The default animation is defined outside the media query. The `@media (prefers-reduced-motion: reduce)` block overrides the animation with a simpler fade-in that removes the spatial movement (translateY) while keeping the opacity change. This respects the user's preference while still providing a visible transition.

---

### Real-World Cases

- **Scrollytelling articles:** Replace slide and zoom animations with simple fades for reduced-motion users.
- **Loading spinners:** Replace spinning animations with a static loading indicator or a gentle pulse.
- **Parallax effects:** Disable parallax scrolling entirely for reduced-motion users.
- **Hover animations:** Reduce or remove transform-based hover effects.

---

## References

- MDN Web Docs — CSS scroll-driven animations - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations
- MDN Web Docs — CSS motion path - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Motion_path
- MDN Web Docs — `prefers-reduced-motion` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion
- MDN Web Docs — `animation-range-start` - https://developer.mozilla.org/en-US/docs/Web/CSS/animation-range-start
- MDN Web Docs — `offset-path` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/offset-path
- MDN Web Docs — `will-change` - https://developer.mozilla.org/en-US/docs/Web/CSS/will-change
- W3C — CSS Scroll-Driven Animations - https://www.w3.org/TR/scroll-animations-1/
- W3C — Motion Path Module Level 1 - https://www.w3.org/TR/motion-1/
- W3C — CSS Will Change Module Level 1 - https://www.w3.org/TR/css-will-change-1/
- W3C — WCAG Technique C39 - https://www.w3.org/WAI/WCAG22/Techniques/css/C39
- web.dev — Stick to Compositor-Only Properties and Manage Layer Count - https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count
- Chrome for Developers — Animate elements on scroll with scroll-driven animations - https://developer.chrome.com/docs/css-ui/scroll-driven-animations
- CSS-Tricks — `sibling-index()` - https://css-tricks.com/almanac/functions/s/sibling-index/
- CSS-Tricks — `offset-path` - https://css-tricks.com/almanac/properties/o/offset-path/
- Smashing Magazine — Advanced Tree Counting: Mathematical Layouts With `sibling-index()` And `sibling-count()` - https://www.smashingmagazine.com/2026/05/advanced-tree-counting-mathematical-layouts-sibling-index-sibling-count/