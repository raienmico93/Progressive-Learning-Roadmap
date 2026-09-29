# CSS Timing Functions — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Timing Functions (also called Easing Functions) are mathematical functions that describe the rate at which a CSS property's value changes over the duration of a transition or animation. They control whether motion starts slowly, accelerates, decelerates, or moves at a constant speed, giving animations a natural, physical feel.

**Technical Definition:** The `<easing-function>` CSS data type represents a mathematical function that describes the rate at which a value changes. Easing functions provide a means to transform values by taking an input progress value (typically the elapsed time as a fraction of the total duration) and producing a corresponding transformed output progress value (the interpolated property value at that point). These functions can be specified for CSS transition and animation properties via `transition-timing-function` and `animation-timing-function`. The CSS Easing Functions Module Level 2 defines four categories: linear easing functions (constant rate), cubic Bézier easing functions (smooth variable rate), step easing functions (discrete intervals), and the modern `linear()` function for approximating arbitrary curves.

**Beginner-Friendly Explanation:** Imagine you are driving a car from point A to point B. You could accelerate smoothly, brake gradually, or maintain a constant speed. CSS timing functions let you control exactly how an animation "moves" from start to finish. `linear` is like cruise control — constant speed. `ease-in` is like slowly pressing the gas pedal — it starts slow and speeds up. `ease-out` is like easing off the gas — it starts fast and slows down. And the new `linear()` function lets you draw a custom speed graph with as many points as you need, allowing you to recreate complex effects like bounces and springs directly in CSS.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Rate control** | Timing functions control the rate of change, not the total duration. |
| **Input/output mapping** | Each function maps an input progress (0 to 1) to an output progress (0 to 1). |
| **Four categories** | Linear, cubic Bézier, step, and the multipoint `linear()` function. |
| **Keyword shortcuts** | `ease`, `ease-in`, `ease-out`, `ease-in-out`, `linear` are predefined cubic Bézier curves. |
| **Custom curves** | `cubic-bezier()` and `linear()` allow author-defined curves. |
| **Step functions** | `steps()` creates discrete, robot-like movement. |
| **Applied to transitions and animations** | Used with `transition-timing-function` and `animation-timing-function`. |

---

### Prerequisites

Before studying CSS timing functions, you should understand:

- **CSS Transitions** — the `transition` shorthand and `transition-timing-function`.
- **CSS Animations** — the `animation` shorthand and `@keyframes`.
- **CSS Values and Units** — numbers, percentages, and the `<time>` data type.
- **Coordinate Systems** — basic understanding of X/Y graphs.

---

### Related Programming Areas

- **CSS Animations** — timing functions control the pace of keyframe interpolation.
- **CSS Transitions** — timing functions control the pace of state-change interpolation.
- **UI/UX Design** — easing curves determine the "feel" of an interaction.
- **Game Development** — easing functions in CSS mirror those in game engines.
- **Web Performance** — the choice of timing function affects perceived performance.

---

### Core Concepts / Features

1. Standard Easing Keywords: `ease`, `linear`, `ease-in`, `ease-out`, and `ease-in-out`
2. Custom Bézier Curves: `cubic-bezier(x1, y1, x2, y2)`
3. Advanced Linear Easing: The `linear()` Function
4. Stepping Functions: `steps(number, direction)`

---

## 1. Standard Easing Keywords: Behavioral Mechanics of `ease`, `linear`, `ease-in`, `ease-out`, and `ease-in-out`

### Definitions

**Core Definition:** The standard easing keywords are five predefined cubic Bézier curves that cover the most common animation timing patterns: constant speed (`linear`), slow start (`ease-in`), slow end (`ease-out`), slow start and end (`ease-in-out`), and a balanced default (`ease`).

**Technical Definition:** The non-step keyword values (`ease`, `linear`, `ease-in-out`, etc.) each represent cubic Bézier curves with fixed four-point values, while the `cubic-bezier()` function value allows non-predefined values to be specified. The `linear` keyword is equivalent to `cubic-bezier(0, 0, 1, 1)` and animates at an even speed. The `ease` keyword is equivalent to `cubic-bezier(0.25, 0.1, 0.25, 1.0)` and is the default value; it increases in velocity towards the middle of the animation and slows back down at the end. The `ease-in` keyword is equivalent to `cubic-bezier(0.42, 0, 1.0, 1.0)` and starts slowly, with the speed increasing until complete. The `ease-out` keyword is equivalent to `cubic-bezier(0, 0, 0.58, 1.0)` and starts quickly, slowing down as the animation continues. The `ease-in-out` keyword is equivalent to `cubic-bezier(0.42, 0, 0.58, 1.0)` and transitions slowly, speeds up, and then slows down again.

**Beginner-Friendly Explanation:** These five keywords are the "presets" of animation timing. `linear` is a steady, constant speed — good for continuous animations like spinners. `ease-in` starts slowly and speeds up — good for elements exiting the screen. `ease-out` starts fast and slows down — good for elements entering the screen. `ease-in-out` starts slow, speeds up in the middle, and slows down at the end — good for elements moving from one position to another. `ease` (the default) is similar to `ease-in-out` but with a slightly sharper acceleration at the beginning.

---

### Purposes

- To provide quick, standardised timing patterns without writing custom curves.
- To control the acceleration and deceleration of transitions and animations.
- To give animations a natural, physical feel that mimics real-world motion.
- To serve as the default timing function when none is specified.
- To provide a baseline for comparing custom curves.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition-timing-function: ease | linear | ease-in | ease-out | ease-in-out;
    animation-timing-function: ease | linear | ease-in | ease-out | ease-in-out;
}
```

#### Component Breakdown

| Keyword | Cubic Bézier Equivalent | Behaviour | Best For |
|---|---|---|---|
| `linear` | `cubic-bezier(0, 0, 1, 1)` | Constant speed. | Spinners, progress bars, marquees. |
| `ease` | `cubic-bezier(0.25, 0.1, 0.25, 1)` | Slow start, sharp acceleration, slow end. | Default; general-purpose. |
| `ease-in` | `cubic-bezier(0.42, 0, 1, 1)` | Slow start, accelerating to full speed. | Exit animations, elements leaving. |
| `ease-out` | `cubic-bezier(0, 0, 0.58, 1)` | Fast start, decelerating to a stop. | Entry animations, elements appearing. |
| `ease-in-out` | `cubic-bezier(0.42, 0, 0.58, 1)` | Slow start, speed up, slow end. | Spatial movement, repositioning. |

#### Syntax Rules

1. The `linear` keyword is always interpreted as `linear(0, 1)` and is equivalent to `cubic-bezier(0, 0, 1, 1)`.
2. The `ease` keyword is the default value for both `transition-timing-function` and `animation-timing-function`.
3. When multiple properties are transitioned, each can have its own easing function via a comma-separated list.
4. If fewer easing functions are provided than properties, the remaining properties use the default `ease`.
5. The keywords are case-insensitive.
6. All five keywords are Baseline widely available and supported in every modern browser.

#### Constraints and Limitations

- **Limited expressiveness** — the five keywords cover only basic acceleration patterns; complex effects like bounce and elastic require custom curves.
- **No overshoot** — none of the standard keywords produce values outside the 0–1 range, so they cannot create bounce or elastic effects.
- **Default `ease` is often misapplied** — using `ease` for all animations can feel generic; different directions (entry vs. exit) benefit from different curves.
- **`linear` feels mechanical** — for spatial movement, constant speed can feel robotic; reserve `linear` for progress indicators.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing the Five Standard Easing Keywords

**HTML File (`standard-easings.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Standard Easing Keywords</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="standard-easings.css">
</head>
<body>
    <div class="demo-container">
        <div class="track"><span class="ball linear">linear</span></div>
        <div class="track"><span class="ball ease">ease</span></div>
        <div class="track"><span class="ball ease-in">ease-in</span></div>
        <div class="track"><span class="ball ease-out">ease-out</span></div>
        <div class="track"><span class="ball ease-in-out">ease-in-out</span></div>
    </div>
</body>
</html>
```

**CSS File (`standard-easings.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.demo-container {
    display: flex;
    flex-direction: column;
    gap: 16px;
    max-width: 700px;
    margin: 0 auto;
}

.track {
    position: relative;
    height: 50px;
    background-color: #e0f7fa;
    border-radius: 25px;
    overflow: hidden;
}

.ball {
    position: absolute;
    top: 5px;
    left: 5px;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background-color: #006064;
    color: white;
    font-size: 0.6rem;
    font-weight: bold;
    display: flex;
    align-items: center;
    justify-content: center;

    /* All balls use the same duration and transform */
    transition: transform 2s;
}

.track:hover .ball {
    /* Move all balls to the right on hover */
    transform: translateX(620px);
}

.linear    { transition-timing-function: linear; }
.ease      { transition-timing-function: ease; }
.ease-in   { transition-timing-function: ease-in; }
.ease-out  { transition-timing-function: ease-out; }
.ease-in-out { transition-timing-function: ease-in-out; }
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `standard-easings.html`.
3. Save the CSS code as `standard-easings.css` in the same folder.
4. Open `standard-easings.html` in a web browser.
5. Hover over the demo container. All five balls slide to the right over 2 seconds, but each with a different acceleration pattern.

**Expected Output:** Five balls in separate tracks. On hover, all five translate the same distance in the same duration, but `linear` moves at constant speed, `ease-in` starts slow and accelerates, `ease-out` starts fast and decelerates, `ease-in-out` starts slow, speeds up, and slows down, and `ease` does a similar but slightly sharper version of `ease-in-out`.

**Why This Works:** All five balls have `transition: transform 2s`, so the total duration and distance are identical. The only difference is the `transition-timing-function`, which maps the elapsed time to the interpolated `translateX` value. Because all easing functions map input progress 0→1 to output progress 0→1, the balls all start and end at the same positions — only the speed along the way differs.

---

### Real-World Cases

- **Entry animations:** `ease-out` for elements fading or sliding into view (fast start, gentle stop).
- **Exit animations:** `ease-in` for elements leaving the screen (gentle start, accelerating exit).
- **Spatial repositioning:** `ease-in-out` for elements moving from one position to another.
- **Progress indicators:** `linear` for spinners, progress bars, and looping animations.
- **Default interactions:** `ease` for general-purpose hover and focus transitions.

---

## 2. Custom Bézier Curves: Designing Precise Velocity Profiles Using `cubic-bezier(x1, y1, x2, y2)`

### Definitions

**Core Definition:** The `cubic-bezier()` CSS function allows authors to define a custom cubic Bézier curve with precise control over the acceleration and deceleration profile of an animation.

**Technical Definition:** The `cubic-bezier()` CSS function creates a smooth transition using a cubic Bézier curve. The function accepts four parameters: `<x1>`, `<y1>`, `<x2>`, and `<y2>`, which represent the coordinates of two control points. The x-axis represents the input progress (time) and the y-axis represents the output progress (the interpolated value). The first control point's x-coordinate (`<x1>`) and the second control point's x-coordinate (`<x2>`) must be in the range [0, 1]. The y-coordinates (`<y1>` and `<y2>`) can be outside this range, enabling overshoot and undershoot effects. The curve is defined by four points: P0 and P3 (the start and end, fixed at (0, 0) and (1, 1)) and P1 and P2 (the intermediate control points defined by the function parameters).

**Beginner-Friendly Explanation:** A cubic Bézier curve is a smooth line defined by four points: the start, the end, and two control points that pull the curve in different directions. In CSS, the start and end are fixed at the bottom-left (0,0) and top-right (1,1). You only specify the two control points. By placing them strategically, you can create any easing curve you want — including curves that overshoot the target (going above 1 on the y-axis) and then come back, which is how bounce and elastic effects are approximated.

---

### Purposes

- To create custom easing profiles that are not available as keywords.
- To fine-tune the acceleration and deceleration of an animation.
- To produce overshoot effects (bounce, elastic) by using y-values outside [0, 1].
- To match the motion feel of a specific design system or brand.
- To replicate easing curves from design tools (Figma, After Effects) in CSS.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition-timing-function: cubic-bezier(x1, y1, x2, y2);
    animation-timing-function: cubic-bezier(x1, y1, x2, y2);
}
```

#### Component Breakdown

| Parameter | Range | Description |
|---|---|---|
| `<x1>` | [0, 1] | X-coordinate of the first control point (time). |
| `<y1>` | Any number | Y-coordinate of the first control point (value). |
| `<x2>` | [0, 1] | X-coordinate of the second control point (time). |
| `<y2>` | Any number | Y-coordinate of the second control point (value). |

#### Common Custom Curves

| Curve Name | `cubic-bezier()` | Character |
|---|---|---|
| Ease In Sine | `cubic-bezier(0.12, 0, 0.39, 0)` | Gentle acceleration. |
| Ease Out Sine | `cubic-bezier(0.61, 1, 0.88, 1)` | Gentle deceleration. |
| Ease In Out Quad | `cubic-bezier(0.45, 0, 0.55, 1)` | Symmetric smooth. |
| Ease Out Back | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Overshoot and settle. |
| Ease Out Expo | `cubic-bezier(0.16, 1, 0.3, 1)` | Rapid start, long settle. |
| Anticipate | `cubic-bezier(0.36, -0.6, 0.66, -0.56)` | Pulls back before moving. |

#### Syntax Rules

1. `cubic-bezier()` accepts exactly four `<number>` values separated by commas.
2. `<x1>` and `<x2>` must be in the range [0, 1]; otherwise, the declaration is invalid.
3. `<y1>` and `<y2>` can be outside [0, 1], enabling overshoot and undershoot.
4. The start point P0 is fixed at (0, 0) and the end point P3 is fixed at (1, 1).
5. If the curve is invalid, CSS ignores the entire property declaration.
6. The curve is evaluated using the x-axis as input progress and the y-axis as output progress.
7. All standard easing keywords are equivalent to specific `cubic-bezier()` values.

#### Constraints and Limitations

- **X values constrained** — `<x1>` and `<x2>` must be between 0 and 1, preventing curves that go backward in time.
- **Y values can overshoot** — values outside [0, 1] create overshoot effects, which can be desirable (bounce) or undesirable (jitter).
- **Four parameters only** — cubic Bézier curves cannot express complex multi-bounce patterns; use `linear()` for those.
- **Not intuitive** — designing curves by typing numbers is not intuitive; visual tools like cubic-bezier.com are recommended.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Overshoot with `cubic-bezier()`

**HTML File (`cubic-bezier.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Custom Cubic Bézier</title>
    <link rel="stylesheet" href="cubic-bezier.css">
</head>
<body>
    <div class="container">
        <div class="box ease-out-back">ease-out-back</div>
        <div class="box anticipate">anticipate</div>
    </div>
</body>
</html>
```

**CSS File (`cubic-bezier.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 60px;
    justify-content: center;
}

.box {
    width: 160px;
    height: 160px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 0.75rem;
    border-radius: 12px;
    cursor: pointer;
    text-align: center;
    padding: 10px;

    transition: transform 600ms;
}

.ease-out-back {
    background-color: #3498db;
    /* Overshoot: y2 = 1.56 pulls beyond the target */
    transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);
}

.anticipate {
    background-color: #e74c3c;
    /* Anticipate: y1 = -0.6 pulls backward before moving */
    transition-timing-function: cubic-bezier(0.36, -0.6, 0.66, -0.56);
}

.box:hover {
    transform: translateY(-60px) scale(1.15);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `cubic-bezier.html` and CSS as `cubic-bezier.css`.
2. Open in a browser.
3. Hover over each box. Observe that `ease-out-back` overshoots its target (goes too far, then settles back), while `anticipate` pulls back slightly before moving forward.

**Expected Output:** Two boxes that move on hover. The blue box overshoots its target and settles back. The red box pulls backward first, then moves forward.

**Why This Works:** In `ease-out-back`, the `y2` value of `1.56` is greater than 1, so the output progress exceeds the target value, causing an overshoot before settling. In `anticipate`, the negative `y1` value of `-0.6` causes the output to go below 0 (backward) before moving forward. These effects are impossible with the standard keywords because their y-values are constrained to [0, 1].

---

### Real-World Cases

- **Playful button hovers:** `cubic-bezier(0.34, 1.56, 0.64, 1)` for a springy, overshooting effect.
- **Anticipation animations:** `cubic-bezier(0.36, -0.6, 0.66, -0.56)` for elements that pull back before launching.
- **Smooth deceleration:** `cubic-bezier(0.16, 1, 0.3, 1)` (ease-out-expo) for modals and dropdowns.
- **Design system tokens:** Defining brand-specific easing curves as CSS custom properties.

---

## 3. Advanced Linear Easing: The `linear()` Function

### Definitions

**Core Definition:** The `linear()` CSS function is a modern easing function that interpolates linearly between a series of specified points, allowing authors to approximate complex curves — including bounce, spring, and elastic effects — that were previously only possible with JavaScript or `@keyframes`.

**Technical Definition:** The `linear()` CSS function creates a transition curve that progresses uniformly between points. As an `<easing-function>`, it creates transitions where the interpolation occurs at a constant rate from beginning to end. The function accepts a comma-separated list of two or more stops, where each stop is a single `<number>` value ranging from 0 to 1. The stops are spread equidistantly by default, but optional `<percentage>` values can define the starting point and ending point of each stop. By providing enough stops, authors can approximate arbitrary easing curves with high precision. The `linear()` function is defined in CSS Easing Functions Level 2.

**Beginner-Friendly Explanation:** Think of `linear()` as a connect-the-dots drawing tool for animation curves. You give it a list of points, and the browser draws straight lines between them. If you provide enough points, the result looks like a smooth curve. This is how you create bounce and spring effects in pure CSS: you generate a list of points from a mathematical function (like a bounce equation) and pass them to `linear()`. The browser then interpolates between each pair of points at a constant rate, approximating the original curve.

---

### Purposes

- To approximate complex easing curves (bounce, spring, elastic) in pure CSS.
- To replicate easing functions from JavaScript or game engines.
- To create custom curves with multiple acceleration and deceleration phases.
- To avoid the limitations of cubic Bézier curves (only four control points).
- To provide a path to native spring and physics-based motion in CSS.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition-timing-function: linear(<stop-list>);
    animation-timing-function: linear(<stop-list>);
}

/* Stop list syntax */
linear(<number> [<percentage> [<percentage>]]?, ...)
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<number>` | The output progress value (0 to 1). | `0.25` |
| `<percentage>` (first) | The starting point of this stop in time. | `75%` |
| `<percentage>` (second) | The ending point of this stop in time. | `100%` |

#### Syntax Rules

1. `linear()` accepts a comma-separated list of two or more stops.
2. Each stop is a `<number>` (output progress) with optional `<percentage>` values (timing).
3. Without percentages, stops are spread equidistantly. `linear(0, 0.25, 1)` uses 0.25 at the 50% mark.
4. One percentage defines the starting point: `linear(0, 0.25 75%, 1)` uses 0.25 at 75%.
5. Two percentages define a hold: `linear(0, 0.25 25% 75%, 1)` holds 0.25 from 25% to 75%.
6. The `linear` keyword is always interpreted as `linear(0, 1)`.
7. The function is supported in all modern browsers (Chrome 113+, Firefox 112+, Safari 17.2+).

#### Constraints and Limitations

- **Approximation, not exact** — `linear()` creates a piecewise linear approximation; more stops mean higher precision.
- **File size** — complex curves with hundreds of stops can increase CSS file size.
- **Generation tooling** — complex curves are best generated by tools rather than hand-written.
- **Not a true spring** — `linear()` approximates spring physics but does not simulate them in real time.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Bounce Easing with `linear()`

**HTML File (`linear-easing.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>linear() Bounce Easing</title>
    <link rel="stylesheet" href="linear-easing.css">
</head>
<body>
    <div class="track">
        <div class="ball">Bounce</div>
    </div>
</body>
</html>
```

**CSS File (`linear-easing.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.track {
    position: relative;
    height: 80px;
    background-color: #e0f7fa;
    border-radius: 40px;
    overflow: hidden;
}

.ball {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background-color: #e74c3c;
    color: white;
    font-weight: bold;
    font-size: 0.7rem;
    display: flex;
    align-items: center;
    justify-content: center;

    /* Bounce easing approximated with linear() stops */
    transition: transform 1.5s linear(
        0, 0.004, 0.016, 0.035, 0.063, 0.098, 0.141, 0.191, 0.25,
        0.316, 0.391, 0.473, 0.563, 0.66, 0.766, 0.879, 1,
        1.031, 1.055, 1.07, 1.078, 1.078, 1.07, 1.055, 1.031, 1,
        0.969, 0.945, 0.93, 0.922, 0.922, 0.93, 0.945, 0.969, 1,
        0.984, 0.977, 0.977, 0.984, 1
    );
}

.track:hover .ball {
    transform: translateX(600px);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `linear-easing.html` and CSS as `linear-easing.css`.
2. Open in a browser.
3. Hover over the track. The ball moves to the right with a bouncing motion, overshooting and settling back multiple times.

**Expected Output:** A red ball that bounces to the right, overshooting the target and oscillating back and forth before settling.

**Why This Works:** The `linear()` function receives a list of 40 stops that approximate a bounce curve. The stops initially rise from 0 to 1, then oscillate above and below 1 (creating the bounce), and finally settle at 1. The browser interpolates linearly between each pair of stops, creating a piecewise linear approximation of the bounce curve. More stops would produce a smoother curve.

---

### Real-World Cases

- **Spring animations:** Generating `linear()` stop lists from spring physics equations for native CSS springs.
- **Bounce effects:** Approximating bounce easing for playful UI interactions.
- **Elastic animations:** Creating elastic overshoot effects for drawers and panels.
- **Design tool export:** Design tools exporting complex easing curves as `linear()` stop lists.

---

## 4. Stepping Functions: `steps(number, direction)`

### Definitions

**Core Definition:** The `steps()` CSS function divides the animation duration into a specified number of equal-length intervals, producing a discrete, staircase-like progression instead of a smooth interpolation.

**Technical Definition:** The `steps()` CSS function defines a transition that divides the input time into a specified number of intervals that are equal in length. This subclass of step functions is sometimes called staircase functions. The function accepts two parameters: an integer representing the number of equidistant intervals, and an optional `<step-position>` keyword that specifies when the jump between values occurs. The possible step positions are `jump-start` (or `start`), `jump-end` (or `end`), `jump-none`, and `jump-both`. If omitted, the step position defaults to `end`. The `step-start` keyword is equivalent to `steps(1, jump-start)` and `step-end` is equivalent to `steps(1, jump-end)`.

**Beginner-Friendly Explanation:** `steps()` is for animations that should move in discrete jumps rather than smoothly. Imagine a clock with a ticking second hand — it does not sweep smoothly, it jumps from one second to the next. `steps(12, end)` divides the animation into 12 equal intervals, with the value jumping at the end of each interval. The direction keyword controls when the jump happens: `jump-start` jumps at the beginning of each interval, `jump-end` (the default) jumps at the end, `jump-none` removes the jumps at the very start and end, and `jump-both` adds jumps at both the start and end.

---

### Purposes

- To create discrete, stepped animations (typewriter effects, ticking clocks, sprite sheets).
- To divide an animation into equal segments for sprite-based frame animation.
- To create robot-like or mechanical movement.
- To produce stepped progress indicators.
- To synchronise animations with discrete state changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition-timing-function: steps(<integer>, <step-position>);
    animation-timing-function: steps(<integer>, <step-position>);
}

/* Step position keywords */
steps(n, jump-start) /* or start */
steps(n, jump-end)   /* or end (default) */
steps(n, jump-none)
steps(n, jump-both)
```

#### Component Breakdown

| Parameter | Description | Values |
|---|---|---|
| `<integer>` | Number of equal intervals. | Positive integer > 0 (or > 1 for `jump-none`). |
| `<step-position>` | When the jump occurs. | `jump-start`, `jump-end`, `jump-none`, `jump-both`. |

#### Step Position Behaviour

| Position | Jump Behaviour | Equivalent Keyword |
|---|---|---|
| `jump-start` | First jump at the beginning of the animation. | `step-start` (when n=1) |
| `jump-end` | Last jump at the end of the animation. | `step-end` (when n=1) |
| `jump-none` | No jump at the start or end; each step holds. | — |
| `jump-both` | Jumps at both the start and end. | — |

#### Syntax Rules

1. The first parameter must be a positive integer greater than 0 (or greater than 1 for `jump-none`).
2. The second parameter is optional; if omitted, it defaults to `end` (equivalent to `jump-end`).
3. `step-start` is equivalent to `steps(1, jump-start)`; `step-end` is equivalent to `steps(1, jump-end)`.
4. The number of steps applies to each segment if the animation has multiple keyframe segments.
5. Steps are equal in duration; the animation holds each value for one step, then jumps to the next.
6. Browser support: Baseline widely available since July 2015.

#### Constraints and Limitations

- **No smooth motion** — `steps()` produces discrete jumps; it is unsuitable for smooth spatial movement.
- **Sprite sheet alignment** — the number of steps must match the number of frames in the sprite sheet.
- **`jump-none` requires n > 1** — `steps(1, jump-none)` is invalid because there would be no jump at all.
- **Timing precision** — the last step may not land exactly on the final value if the duration does not divide evenly.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Step Positions

**HTML File (`steps.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>steps() Comparison</title>
    <link rel="stylesheet" href="steps.css">
</head>
<body>
    <div class="demo">
        <div class="row">
            <span class="label">jump-start</span>
            <div class="track"><div class="ball jump-start"></div></div>
        </div>
        <div class="row">
            <span class="label">jump-end</span>
            <div class="track"><div class="ball jump-end"></div></div>
        </div>
        <div class="row">
            <span class="label">jump-none</span>
            <div class="track"><div class="ball jump-none"></div></div>
        </div>
        <div class="row">
            <span class="label">jump-both</span>
            <div class="track"><div class="ball jump-both"></div></div>
        </div>
    </div>
</body>
</html>
```

**CSS File (`steps.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.demo {
    display: flex;
    flex-direction: column;
    gap: 20px;
    max-width: 700px;
    margin: 0 auto;
}

.row {
    display: flex;
    align-items: center;
    gap: 20px;
}

.label {
    width: 100px;
    font-size: 0.8rem;
    font-weight: bold;
    color: #333;
    text-align: right;
}

.track {
    position: relative;
    flex: 1;
    height: 50px;
    background-color: #e0f7fa;
    border-radius: 25px;
    overflow: hidden;
}

.ball {
    position: absolute;
    top: 5px;
    left: 5px;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background-color: #006064;

    animation: slide 2s infinite alternate;
}

@keyframes slide {
    from { transform: translateX(0); }
    to   { transform: translateX(600px); }
}

.jump-start { animation-timing-function: steps(6, jump-start); }
.jump-end   { animation-timing-function: steps(6, jump-end); }
.jump-none  { animation-timing-function: steps(6, jump-none); }
.jump-both  { animation-timing-function: steps(6, jump-both); }
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `steps.html` and CSS as `steps.css`.
2. Open in a browser.
3. Observe the four balls: each jumps in 6 discrete steps, but the timing of the jumps differs.

**Expected Output:** Four balls animating across their tracks in 6 discrete jumps. `jump-start` jumps at the beginning of each interval, `jump-end` at the end, `jump-none` holds each step without jumping at the extremes, and `jump-both` jumps at both ends.

**Why This Works:** All four balls use `steps(6, ...)` with different step positions. `jump-start` makes the first jump immediately, so the ball starts at its first stepped value. `jump-end` makes the last jump at the end, so the ball starts at 0 and reaches the final value at the end. `jump-none` removes the first and last jumps, so the ball holds the first and last values for a full interval. `jump-both` adds jumps at both ends, creating 6 jumps across 6 steps but with an extra jump at the start and end.

---

### Real-World Cases

- **Typewriter effects:** `steps(n)` for character-by-character text reveals.
- **Sprite sheet animation:** `steps(n)` to cycle through a sprite sheet's frames.
- **Ticking clocks:** `steps(60)` for a second hand that jumps each second.
- **Stepped progress bars:** `steps(n)` for discrete loading indicators.
- **Robot-like movement:** `steps(n)` for mechanical, non-smooth motion.

---

## References

- MDN Web Docs — `<easing-function>` - https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function
- MDN Web Docs — `cubic-bezier()` - https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function/cubic-bezier
- MDN Web Docs — `linear()` - https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function/linear
- MDN Web Docs — `steps()` - https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function/steps
- MDN Web Docs — `animation-timing-function` - https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timing-function
- MDN Web Docs — `transition-timing-function` - https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function
- W3C — CSS Easing Functions Level 2 - https://www.w3.org/TR/css-easing-2/
- Chrome for Developers — Create complex animation curves in CSS with the `linear()` easing function - https://developer.chrome.com/docs/css-ui/css-linear-easing-function
- CSS-Tricks — `cubic-bezier()` - https://css-tricks.com/almanac/functions/c/cubic-bezier/
- CSS-Tricks — `steps()` - https://css-tricks.com/almanac/functions/s/steps/
- Can I Use — `linear()` easing function - https://caniuse.com/mdn-css_types_easing-function_linear-function
- Can I Use — `steps()` - https://caniuse.com/mdn-css_types_easing-function_steps
- MDN Blog — Creating custom easing effects in CSS animations using the `linear()` function - https://developer.mozilla.org/en-US/blog/custom-easing-effects-css-linear-function/