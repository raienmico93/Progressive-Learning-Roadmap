# CSS Keyframe Animations — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Keyframe Animations is a CSS module that allows authors to define multi-step animations by specifying styles at various points (keyframes) along a timeline. Unlike transitions, which interpolate between two states triggered by a property change, keyframe animations can have any number of intermediate steps and can run autonomously without user interaction.

**Technical Definition:** The CSS Animations Module Level 1 describes a way for authors to animate the values of CSS properties over time, using keyframes. The behavior of these keyframe animations can be controlled by specifying their duration, number of repeats, and repeating behavior. The `@keyframes` at-rule controls the intermediate steps in a CSS animation sequence by defining styles for keyframes (or waypoints) along the animation sequence. An animation is applied to an element via the `animation` property (or its sub-properties), which references a named `@keyframes` rule. The animation progresses through its keyframes over the specified duration, with interpolation between keyframes handled by the browser using the specified timing function. The `animation-composition` property (CSS Animations Level 2) allows control over how multiple animations affecting the same property are combined.

**Beginner-Friendly Explanation:** A keyframe animation is like a flipbook. You draw a few important frames — the beginning, the middle, and the end — and the browser fills in all the frames in between. You give your animation a name, define what happens at each keyframe (0%, 50%, 100%), and then tell an element to run that animation for a certain duration. The element can loop the animation, run it backward, pause it, or hold its final state. This makes keyframe animations far more powerful than transitions, which can only interpolate between two states.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Multi-step** | Any number of keyframes can be defined between 0% and 100%. |
| **Autonomous** | Runs without user interaction once applied. |
| **Named and reusable** | The `@keyframes` rule is named and can be referenced by multiple elements. |
| **Loopable** | Can repeat indefinitely or a specific number of times. |
| **Directional control** | Can play forward, backward, or alternate. |
| **State persistence** | `animation-fill-mode` controls styles before and after the animation. |
| **Composable** | Multiple animations can affect the same property with `animation-composition`. |
| **Event-driven** | JavaScript events fire at each stage of the animation lifecycle. |

---

### Prerequisites

Before studying CSS keyframe animations, you should understand:

- **CSS Transitions** — the simpler two-state animation model.
- **CSS Timing Functions** — how easing curves control the pace of interpolation.
- **CSS Transform** — the most common property to animate.
- **CSS Selectors and Specificity** — how animations interact with existing styles.
- **Basic JavaScript Event Handling** — for animation events.

---

### Related Programming Areas

- **CSS Transitions** — the simpler sibling module for state-change animations.
- **Web Animations API** — programmatic control over keyframe animations.
- **CSS Transforms** — the most commonly animated property group.
- **UI/UX Design** — loading spinners, page transitions, attention indicators.
- **Accessibility** — `prefers-reduced-motion` for respecting user preferences.

---

### Core Concepts / Features

1. The Animation Lifecycle: `@keyframes` Syntax, Percentages, and `from`/`to`
2. The Animation Shorthand and Sub-properties
3. Direction and Fill Modes: `animation-direction` and `animation-fill-mode`
4. Composition Control: `animation-composition`
5. Animation Events: `animationstart`, `animationiteration`, `animationend`, `animationcancel`

---

## 1. The Animation Lifecycle: Syntax of the `@keyframes` Block, Mapping Percentages, and Handling the `from` and `to` Keywords

### Definitions

**Core Definition:** The `@keyframes` at-rule is a named block that defines the styles for each step (keyframe) of an animation. Each keyframe is identified by a percentage (0% to 100%) or the keywords `from` and `to`, and contains the CSS declarations to apply at that point in the animation.

**Technical Definition:** A `@keyframes` rule consists of the keyword `@keyframes`, followed by a `<keyframes-name>` (a case-sensitive custom identifier or string), followed by a block containing a list of keyframe rules. Each keyframe rule consists of a comma-separated list of percentage values or the keywords `from` and `to`, followed by a block of property declarations. The keyword `from` is equivalent to the value `0%`, and the keyword `to` is equivalent to the value `100%`. The `@keyframes` at-rule controls the intermediate steps in a CSS animation sequence by defining styles for keyframes (or waypoints) along the animation sequence, giving more control over the intermediate steps than transitions. To use keyframes, create a `@keyframes` rule with a name that is then used by the `animation-name` property to match an animation to its keyframe declaration.

**Beginner-Friendly Explanation:** A `@keyframes` block is like a recipe book for an animation. Each keyframe is a step in the recipe — 0% is the beginning, 100% is the end, and any percentages in between are intermediate steps. You write the styles that should apply at each step, and the browser smoothly transitions between them. The `from` keyword is just a friendlier way to write 0%, and `to` is the same as 100%. You can list the keyframes in any order — the browser sorts them out.

---

### Purposes

- To define the intermediate steps of a multi-step animation.
- To give authors precise control over how an animation progresses.
- To create reusable animation definitions that can be applied to multiple elements.
- To enable complex motion sequences that transitions cannot express.
- To provide a named reference that the `animation-name` property can target.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
@keyframes <keyframes-name> {
    <keyframe-selector> {
        <property>: <value>;
    }
}

/* Keyframe selector values */
<keyframe-selector> = from | to | <percentage> | <timeline-range-name> <percentage>
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `@keyframes` | The at-rule keyword. | `@keyframes` |
| `<keyframes-name>` | A case-sensitive name (custom identifier or string). | `slide-in`, `"pulse"` |
| `from` | Equivalent to `0%`. | `from { opacity: 0; }` |
| `to` | Equivalent to `100%`. | `to { opacity: 1; }` |
| `<percentage>` | A percentage through the animation. | `50% { transform: scale(1.2); }` |
| `<timeline-range-name>` | Optional timeline range (scroll-driven animations). | `entry 0%` |

#### Syntax Rules

1. `from` is equivalent to `0%`; `to` is equivalent to `100%`.
2. Keyframe percentages can be listed in any order; the browser sorts them chronologically.
3. A keyframe selector can be a comma-separated list of percentages and/or keywords: `0%, 100% { ... }`.
4. If a keyframe rule does not specify `0%`/`from` or `100%`/`to`, the browser uses the element's existing styles for the missing states.
5. The `@keyframes` name is case-sensitive.
6. Properties that are not animatable are ignored within keyframes.
7. The `!important` flag on a declaration inside a keyframe is ignored.

#### Constraints and Limitations

- **Animatable properties only** — only properties that can be interpolated can be animated; discrete properties require `transition-behavior: allow-discrete` or similar mechanisms.
- **No layout-triggering properties** — animating `width`, `height`, `top`, or `left` triggers layout on every frame and should be avoided.
- **Name collisions** — if two `@keyframes` rules share the same name, the last one defined wins.
- **Vendor prefixes** — older browsers required `@-webkit-keyframes`, but this is no longer necessary for modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Pulse Animation

**HTML File (`keyframes-basic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basic @keyframes Animation</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="keyframes-basic.css">
</head>
<body>
    <!-- Element with a pulse animation -->
    <div class="pulse-box">Pulse</div>
</body>
</html>
```

**CSS File (`keyframes-basic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

/* Define the keyframes animation */
@keyframes pulse {
    /* from is equivalent to 0% */
    from {
        transform: scale(1);
        background-color: #3498db;
    }
    /* 50% is the midpoint */
    50% {
        transform: scale(1.2);
        background-color: #e74c3c;
    }
    /* to is equivalent to 100% */
    to {
        transform: scale(1);
        background-color: #3498db;
    }
}

.pulse-box {
    width: 150px;
    height: 150px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 1.2rem;
    border-radius: 16px;
    background-color: #3498db;

    /* Apply the animation */
    animation-name: pulse;
    animation-duration: 1.5s;
    animation-iteration-count: infinite;
    animation-timing-function: ease-in-out;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `keyframes-basic.html`.
3. Save the CSS code as `keyframes-basic.css` in the same folder.
4. Open `keyframes-basic.html` in a web browser.
5. Observe the box pulsing — it scales up and changes colour at the midpoint, then returns to its original state.

**Expected Output:** A blue box that continuously scales up to 1.2× and turns red at the 50% mark, then returns to blue and normal size. The animation loops indefinitely.

**Why This Works:** The `@keyframes pulse` rule defines three keyframes: `from` (0%), `50%`, and `to` (100%). The browser interpolates between these keyframes over the 1.5-second duration. The `ease-in-out` timing function smooths the transitions between keyframes. The `animation-iteration-count: infinite` makes the animation loop forever.

---

#### Example 2: Using `from` and `to` with Multiple Keyframe Selectors

**HTML File (`keyframes-from-to.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>from and to Keywords</title>
    <link rel="stylesheet" href="keyframes-from-to.css">
</head>
<body>
    <div class="slide-box">Slide</div>
</body>
</html>
```

**CSS File (`keyframes-from-to.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

@keyframes slide-in {
    /* from = 0% */
    from {
        opacity: 0;
        transform: translateX(-100px);
    }
    /* to = 100% */
    to {
        opacity: 1;
        transform: translateX(0);
    }
}

@keyframes color-shift {
    /* Comma-separated selectors apply the same styles */
    0%, 100% {
        background-color: #3498db;
    }
    50% {
        background-color: #e67e22;
    }
}

.slide-box {
    width: 200px;
    padding: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 1.5rem;
    border-radius: 12px;
    text-align: center;

    /* Two animations applied with comma separation */
    animation: slide-in 800ms ease-out forwards,
               color-shift 2s ease-in-out infinite;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `keyframes-from-to.html` and CSS as `keyframes-from-to.css`.
2. Open in a browser.
3. Observe the box slides in from the left and fades in over 800ms, then continuously shifts its background colour between blue and orange.

**Expected Output:** A box that slides in from the left and fades in, then cycles through a colour animation indefinitely.

**Why This Works:** The `slide-in` animation uses `from` and `to` to define the entry animation. The `color-shift` animation uses a comma-separated selector (`0%, 100%`) to apply the same styles at both the start and end. Both animations run simultaneously because they are comma-separated in the `animation` shorthand.

---

### Real-World Cases

- **Loading spinners:** A `@keyframes` animation rotating an element 360 degrees.
- **Attention indicators:** A pulse animation drawing attention to a notification badge.
- **Page transitions:** A slide-in animation for modals and dialogs.
- **Typing effects:** A keyframe animation that reveals characters one by one.

---

## 2. The Animation Shorthand and Sub-properties: Correctly Layering `animation-name`, `-duration`, `-delay`, `-iteration-count`, and `-play-state`

### Definitions

**Core Definition:** The `animation` shorthand property sets all eight animation sub-properties in a single declaration. Each sub-property controls a different aspect of the animation, from its name and duration to its delay, iteration count, and play state.

**Technical Definition:** The `animation` shorthand is a shorthand property for `animation-name`, `animation-duration`, `animation-timing-function`, `animation-delay`, `animation-iteration-count`, `animation-direction`, `animation-fill-mode`, `animation-play-state`, and `animation-composition`. The shorthand accepts a comma-separated list of animations, each with its own set of values. For each animation, the first time value encountered is assigned to `animation-duration`, and the second time value is assigned to `animation-delay`. If a value is not specified for a sub-property, its initial value is used. The `animation-name` is the only value that is a `<custom-ident>` or string; all other values are keywords, times, or numbers.

**Beginner-Friendly Explanation:** The `animation` shorthand is like ordering a pizza with all the toppings in one sentence. Instead of writing eight separate lines (`animation-name`, `animation-duration`, `animation-delay`, etc.), you write one line: `animation: slide-in 1s ease-out 200ms infinite alternate forwards running`. The order is flexible, but the two time values have a strict rule: the first time is the duration, the second is the delay. If you only provide one time value, it is the duration, and the delay defaults to zero.

---

### Purposes

- To write animation declarations concisely in a single line.
- To reduce repetition when setting multiple animation sub-properties.
- To apply multiple animations simultaneously with comma-separated values.
- To simplify stylesheets and improve readability.
- To provide the standard syntax used in most CSS frameworks and examples.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    animation: <single-animation>#;
}

<single-animation> = <time> || <easing-function> || <time> || <single-animation-iteration-count> || <single-animation-direction> || <single-animation-fill-mode> || <single-animation-play-state> || [ none | <keyframes-name> ] || <single-animation-composition>
```

#### Component Breakdown

| Sub-property | Initial Value | Accepted Values | Description |
|---|---|---|---|
| `animation-name` | `none` | `<custom-ident>`, `none` | The name of the `@keyframes` rule. |
| `animation-duration` | `0s` | `<time>` | How long one cycle takes. |
| `animation-timing-function` | `ease` | `<easing-function>` | The easing curve. |
| `animation-delay` | `0s` | `<time>` | Delay before the animation starts. |
| `animation-iteration-count` | `1` | `<number>`, `infinite` | How many times to repeat. |
| `animation-direction` | `normal` | `normal`, `reverse`, `alternate`, `alternate-reverse` | Playback direction. |
| `animation-fill-mode` | `none` | `none`, `forwards`, `backwards`, `both` | Styles before/after animation. |
| `animation-play-state` | `running` | `running`, `paused` | Whether the animation is playing. |
| `animation-composition` | `replace` | `replace`, `add`, `accumulate` | How multiple animations combine. |

#### Shorthand Ordering Rules

The `animation` shorthand uses a flexible ordering system, but with two important constraints:

1. **The first `<time>` value** is always `animation-duration`.
2. **The second `<time>` value** is always `animation-delay`.
3. All other values can appear in any order.
4. The `animation-name` is the value that matches a `@keyframes` name.

#### Syntax Rules

1. The `animation` shorthand accepts comma-separated animations.
2. If a sub-property is omitted, its initial value is used.
3. The first time value is duration; the second time value is delay.
4. If `animation-name` is omitted, `none` is used (no animation).
5. Multiple animations can be applied by comma-separating the shorthand values.
6. `animation-play-state: paused` can pause an animation; `running` resumes it.
7. `animation-iteration-count: infinite` loops the animation forever.

#### Constraints and Limitations

- **Time value ambiguity** — the two time values must be in the order duration then delay.
- **No explicit labels** — you cannot label which time is which; you must rely on order.
- **Readability** — long shorthand declarations can be hard to read; use longhand properties for clarity.
- **`animation-composition` support** — relatively new; check browser support.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Using the Animation Shorthand

**HTML File (`animation-shorthand.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animation Shorthand</title>
    <link rel="stylesheet" href="animation-shorthand.css">
</head>
<body>
    <div class="spinner"></div>
    <button class="toggle-btn">Pause / Resume</button>

    <script>
        const spinner = document.querySelector('.spinner');
        const btn = document.querySelector('.toggle-btn');
        btn.addEventListener('click', () => {
            const current = getComputedStyle(spinner).animationPlayState;
            spinner.style.animationPlayState = current === 'running' ? 'paused' : 'running';
        });
    </script>
</body>
</html>
```

**CSS File (`animation-shorthand.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 30px;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

.spinner {
    width: 80px;
    height: 80px;
    border: 8px solid #e0f7fa;
    border-top-color: #006064;
    border-radius: 50%;

    /* Shorthand: name duration timing-function delay iteration-count direction fill-mode play-state */
    animation: spin 1s linear 0s infinite normal none running;
}

.toggle-btn {
    padding: 12px 28px;
    font-size: 1rem;
    background-color: #006064;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `animation-shorthand.html` and CSS as `animation-shorthand.css`.
2. Open in a browser.
3. Observe the spinner rotating continuously.
4. Click the button to pause and resume the animation.

**Expected Output:** A rotating spinner that can be paused and resumed via the button. The animation uses the shorthand with `animation-play-state` set to `running`.

**Why This Works:** The `animation` shorthand sets all sub-properties in one declaration. The `spin` keyframes rotate the element 360 degrees. The `1s linear` sets duration and timing function. The `infinite` sets iteration count. The `running` sets the play state. The JavaScript toggle changes `animation-play-state` between `running` and `paused`.

---

### Real-World Cases

- **Loading spinners:** `animation: spin 1s linear infinite;` for a continuous rotation.
- **Attention pulses:** `animation: pulse 2s ease-in-out infinite;` for a looping pulse.
- **Slide-in modals:** `animation: slide-in 400ms ease-out forwards;` for a one-time entrance.
- **Pausable animations:** Using `animation-play-state` to pause/resume animations via JavaScript.

---

## 3. Direction and Fill Modes: Manipulating Playback Tracks via `animation-direction` and State Persistence via `animation-fill-mode`

### Definitions

**Core Definition:** `animation-direction` controls whether an animation plays forward, backward, or alternates between the two. `animation-fill-mode` controls which styles apply before the animation starts and after it ends.

**Technical Definition:** The `animation-direction` property sets whether an animation should play forward, backward, or alternate back and forth between playing the sequence forward and backward. Values are `normal` (forward), `reverse` (backward), `alternate` (forward then backward), and `alternate-reverse` (backward then forward). The `animation-fill-mode` property sets how a CSS animation applies styles to its target before and after its execution. Values are `none` (no styles applied), `forwards` (retain the last keyframe), `backwards` (apply the first keyframe during the delay), and `both` (apply both forwards and backwards fill). If `animation-fill-mode` is set to `backwards` or `both`, the first frame of the animation, as defined by `animation-direction`, will be displayed during the `animation-delay`. Then the last frame of the animation, as defined by `animation-direction`, will be displayed if `animation-fill-mode` is set to `forwards` or `both`.

**Beginner-Friendly Explanation:** `animation-direction` controls the playback direction. `normal` plays from 0% to 100%. `reverse` plays from 100% to 0%. `alternate` plays forward, then backward, then forward again, like a yoyo. `alternate-reverse` starts backward and then goes forward. `animation-fill-mode` controls what the element looks like before and after the animation. `forwards` keeps the final keyframe's styles after the animation ends — useful for slide-in animations that should stay in place. `backwards` applies the first keyframe's styles during the delay period. `both` does both.

---

### Purposes

- To control the playback direction of an animation.
- To create yoyo-like alternating animations.
- To hold the final state of an animation after it completes.
- To apply the starting state during the animation delay.
- To combine both fill behaviours for complete state management.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    animation-direction: normal | reverse | alternate | alternate-reverse;
    animation-fill-mode: none | forwards | backwards | both;
}
```

#### Component Breakdown

| Property | Value | Description |
|---|---|---|
| `animation-direction` | `normal` | Plays forward (0% to 100%). |
| `animation-direction` | `reverse` | Plays backward (100% to 0%). |
| `animation-direction` | `alternate` | Forward then backward, repeating. |
| `animation-direction` | `alternate-reverse` | Backward then forward, repeating. |
| `animation-fill-mode` | `none` | No fill; element uses its own styles. |
| `animation-fill-mode` | `forwards` | Retains the last keyframe after the animation. |
| `animation-fill-mode` | `backwards` | Applies the first keyframe during the delay. |
| `animation-fill-mode` | `both` | Applies both forwards and backwards fill. |

#### Fill Mode and Direction Interaction

| `animation-direction` | `animation-fill-mode: forwards` (last keyframe) | `animation-fill-mode: backwards` (first keyframe) |
|---|---|---|
| `normal` | 100% or `to` | 0% or `from` |
| `reverse` | 0% or `from` | 100% or `to` |
| `alternate` | 0% or `from` (even iterations) | 0% or `from` |
| `alternate-reverse` | 100% or `to` (even iterations) | 100% or `to` |

#### Syntax Rules

1. `animation-direction: alternate` plays forward on odd iterations and backward on even iterations.
2. `animation-fill-mode: forwards` retains the last keyframe encountered during execution.
3. `animation-fill-mode: backwards` applies the first relevant keyframe during the delay period.
4. `animation-fill-mode: both` combines both behaviours.
5. The "last keyframe" depends on `animation-direction` and `animation-iteration-count`.
6. Fill modes do not affect the animation itself — only the styles before and after.
7. All values are Baseline widely available.

#### Constraints and Limitations

- **No effect without animation** — fill modes only matter when an animation is applied.
- **Stacking context** — if a new stacking context is created during the animation, the target element retains the stacking context after the animation has finished when `forwards` or `both` is used.
- **`will-change` behaviour** — animated properties behave as if included in a `will-change` property value when fill modes are used.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Fill Modes

**HTML File (`fill-modes.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animation Fill Modes</title>
    <link rel="stylesheet" href="fill-modes.css">
</head>
<body>
    <div class="container">
        <div class="box none">none</div>
        <div class="box forwards">forwards</div>
        <div class="box backwards">backwards</div>
        <div class="box both">both</div>
    </div>
</body>
</html>
```

**CSS File (`fill-modes.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 30px;
    justify-content: center;
}

@keyframes slide-and-color {
    from {
        transform: translateX(0);
        background-color: #3498db;
    }
    to {
        transform: translateX(100px);
        background-color: #e74c3c;
    }
}

.box {
    width: 120px;
    height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 0.8rem;
    border-radius: 12px;
    background-color: #3498db;

    /* All boxes use the same animation with a delay */
    animation-name: slide-and-color;
    animation-duration: 1s;
    animation-delay: 500ms;
    animation-timing-function: ease-out;
}

.none {
    animation-fill-mode: none;
}

.forwards {
    animation-fill-mode: forwards;
}

.backwards {
    animation-fill-mode: backwards;
}

.both {
    animation-fill-mode: both;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `fill-modes.html` and CSS as `fill-modes.css`.
2. Open in a browser.
3. Observe the four boxes during the 500ms delay: the `backwards` and `both` boxes already show the `from` keyframe (blue), while `none` and `forwards` show the element's default blue.
4. After the animation ends: the `forwards` and `both` boxes retain the `to` keyframe (red, moved right), while `none` and `backwards` return to their default blue.

**Expected Output:** Four boxes that behave differently based on their `animation-fill-mode`. The `none` box shows no fill; `forwards` retains the final state; `backwards` applies the start state during the delay; `both` does both.

**Why This Works:** The `animation-fill-mode` property controls the styles applied before and after the animation. `backwards` applies the first keyframe during the delay period. `forwards` retains the last keyframe after the animation ends. `both` combines both. `none` applies neither.

---

#### Example 2: Alternating Direction

**HTML File (`direction.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animation Direction</title>
    <link rel="stylesheet" href="direction.css">
</head>
<body>
    <div class="container">
        <div class="box normal">normal</div>
        <div class="box alternate">alternate</div>
        <div class="box alternate-reverse">alternate-reverse</div>
    </div>
</body>
</html>
```

**CSS File (`direction.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.container {
    display: flex;
    gap: 40px;
    justify-content: center;
}

@keyframes slide {
    from { transform: translateX(0); }
    to { transform: translateX(150px); }
}

.box {
    width: 100px;
    height: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 0.7rem;
    border-radius: 12px;
    background-color: #3498db;

    animation-name: slide;
    animation-duration: 1s;
    animation-iteration-count: infinite;
    animation-timing-function: ease-in-out;
}

.normal {
    animation-direction: normal;
}

.alternate {
    animation-direction: alternate;
}

.alternate-reverse {
    animation-direction: alternate-reverse;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `direction.html` and CSS as `direction.css`.
2. Open in a browser.
3. Observe the three boxes: `normal` jumps back to the start after reaching the end; `alternate` smoothly yoyos back and forth; `alternate-reverse` starts by moving backward first.

**Expected Output:** Three boxes with different playback directions. `normal` moves right, jumps back, and repeats. `alternate` moves right, then left, then right again. `alternate-reverse` moves left, then right, then left again.

**Why This Works:** The `animation-direction` property controls the playback direction. `normal` always plays forward, causing a jump when the animation restarts. `alternate` plays forward on odd iterations and backward on even iterations, creating a smooth yoyo effect. `alternate-reverse` starts backward.

---

### Real-World Cases

- **Yoyo animations:** `alternate` for breathing or pulsing effects that should not jump.
- **Slide-in modals:** `forwards` to keep the modal in its final position after sliding in.
- **Attention badges:** `alternate` for a bouncing notification badge.
- **Progress indicators:** `forwards` to hold the final progress state.

---

## 4. Composition Control: Overriding Stacked Keyframes with `animation-composition`

### Definitions

**Core Definition:** The `animation-composition` property controls what happens when multiple animations affect the same CSS property simultaneously. It allows animations to replace, add to, or accumulate with the underlying value.

**Technical Definition:** The `animation-composition` CSS property specifies the composite operation to use when multiple animations affect the same property simultaneously. Accepted values are: `replace` (the effect value replaces the underlying value — the default), `add` (the effect value is added to the underlying value), and `accumulate` (the effect value is combined with the underlying value). The difference between `add` and `accumulate` is subtle: addition appends the effect value to the underlying value (e.g., `blur(2) blur(3)`), while accumulation mathematically adds them (e.g., `blur(5)`). The `animation-composition` property is defined in CSS Animations Level 2.

**Beginner-Friendly Explanation:** Imagine you have a base style on an element, like `transform: translateX(50px)`. You then apply an animation that also changes `transform`. By default, the animation's value replaces the base value entirely — that is `replace`. With `add`, the animation's value is added after the base value, like stacking two transformations. With `accumulate`, the animation's value is mathematically combined with the base value, like adding two numbers together. This gives you fine control over how animations interact with existing styles.

---

### Purposes

- To control how multiple animations affecting the same property are combined.
- To add an animation's effect on top of an existing base style without replacing it.
- To mathematically accumulate numeric values (like translations or blurs).
- To create complex layered animations without wrapping elements.
- To provide fine-grained control over the composition of animation effects.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    animation-composition: replace | add | accumulate;
    /* Multiple animations */
    animation-composition: replace, add, accumulate;
}
```

#### Component Breakdown

| Value | Description | Example |
|---|---|---|
| `replace` | The effect value replaces the underlying value. (Default) | `transform: translateX(100px)` replaces the base `transform` |
| `add` | The effect value is added to the underlying value. | `transform: translateX(50px) rotate(45deg) translateX(100px)` |
| `accumulate` | The effect value is mathematically combined with the underlying value. | `transform: translateX(150px) rotate(45deg)` |

#### Syntax Rules

1. The default value is `replace`.
2. `add` appends the effect value to the underlying value.
3. `accumulate` combines the effect value with the underlying value mathematically.
4. For non-additive types, `accumulate` and `add` may behave the same.
5. Multiple animations can have different `animation-composition` values via a comma-separated list.
6. `animation-composition` is defined in CSS Animations Level 2.
7. Browser support: Chrome 112+, Edge 112+, Firefox 115+, Safari 16+.

#### Constraints and Limitations

- **Additive types only** — `add` and `accumulate` only work for properties whose values are additive (e.g., `transform`, `filter`, `blur`).
- **Browser support** — relatively new; older browsers ignore the property and use `replace`.
- **Not in the shorthand** — `animation-composition` is not yet included in the `animation` shorthand in all browsers.
- **Complexity** — the difference between `add` and `accumulate` can be subtle and requires testing.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Composition Values

**HTML File (`composition.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animation Composition</title>
    <link rel="stylesheet" href="composition.css">
</head>
<body>
    <div class="container">
        <div class="box replace">replace</div>
        <div class="box add">add</div>
        <div class="box accumulate">accumulate</div>
    </div>
</body>
</html>
```

**CSS File (`composition.css`):**

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

@keyframes move-and-rotate {
    to {
        transform: translateX(100px) rotate(45deg);
    }
}

.box {
    width: 120px;
    height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 0.75rem;
    border-radius: 12px;
    background-color: #3498db;

    /* Base transform applied to all boxes */
    transform: translateX(50px) rotate(0deg);

    /* Animation applied to all boxes */
    animation: move-and-rotate 1s ease-out forwards;
}

.replace {
    animation-composition: replace;
}

.add {
    animation-composition: add;
}

.accumulate {
    animation-composition: accumulate;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `composition.html` and CSS as `composition.css`.
2. Open in a browser that supports `animation-composition` (Chrome 112+, Firefox 115+, Safari 16+).
3. Observe the three boxes at the end of the animation:
   - `replace`: the base `transform` is replaced by the animation's `transform`, so the box moves 100px and rotates 45deg.
   - `add`: the animation's `transform` is added after the base, so the box first moves 50px, then moves 100px and rotates 45deg.
   - `accumulate`: the animation's `translateX(100px)` is mathematically added to the base `translateX(50px)`, resulting in `translateX(150px)` and a 45-degree rotation.

**Expected Output:** Three boxes that end up in different positions because of their `animation-composition` values.

**Why This Works:** The `animation-composition` property controls how the animation's keyframe values interact with the element's existing `transform` value. `replace` overwrites the base value. `add` appends the animation's value to the base. `accumulate` mathematically adds the animation's value to the base. This is particularly useful when you have a base transform and want an animation to modify it without replacing it.

---

### Real-World Cases

- **Layered transforms:** Adding a hover animation on top of a base transform without replacing it.
- **Filter animations:** Accumulating multiple `blur()` filters for a progressive blur effect.
- **Complex motion:** Building multi-layered animations without wrapping elements.
- **Component libraries:** Allowing animations to compose with user-defined transforms.

---

## 5. Animation Events: Capturing Lifecycle Triggers Using `animationstart`, `animationiteration`, `animationend`, and `animationcancel`

### Definitions

**Core Definition:** Animation events are JavaScript events fired by the browser at different stages of a CSS animation's lifecycle — when it starts, when each iteration completes, when it ends, and when it is cancelled.

**Technical Definition:** The CSS Animations specification defines four animation events: `animationstart` (fired at the start of the animation; if there is an `animation-delay`, this event fires once the delay period has expired), `animationiteration` (fired at the end of each iteration of an animation, except when an `animationend` event would fire at the same time), `animationend` (fired when the animation finishes), and `animationcancel` (fired when the animation stops running in a way that does not fire an `animationend` event, such as a change in the `animation-name` that removes the animation). Each event is an `AnimationEvent` object with properties including `animationName`, `elapsedTime`, and `pseudoElement`.

**Beginner-Friendly Explanation:** Animation events let you run JavaScript when something happens during an animation. `animationstart` fires when the animation begins (after any delay). `animationiteration` fires at the end of each loop, which is useful for counting how many times an animation has run. `animationend` fires when the animation finishes completely. `animationcancel` fires when the animation is interrupted — for example, if you remove the animation class before it finishes. These events are how you synchronise application logic with visual animation timing.

---

### Purposes

- To execute JavaScript when an animation starts (`animationstart`).
- To perform actions at the end of each iteration (`animationiteration`).
- To clean up or update state when an animation completes (`animationend`).
- To handle interrupted animations (`animationcancel`).
- To synchronise application state with visual animation state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
element.addEventListener('animationstart', (event) => { /* ... */ });
element.addEventListener('animationiteration', (event) => { /* ... */ });
element.addEventListener('animationend', (event) => { /* ... */ });
element.addEventListener('animationcancel', (event) => { /* ... */ });

/* Or using the on* properties */
element.onanimationend = (event) => { /* ... */ };
```

#### Component Breakdown

| Event | Fired When | Use Case |
|---|---|---|
| `animationstart` | The animation begins (after delay). | Start visual feedback. |
| `animationiteration` | Each iteration completes (except the last). | Count loops, update progress. |
| `animationend` | The animation finishes completely. | Cleanup, DOM removal, state update. |
| `animationcancel` | The animation is cancelled or interrupted. | Revert state, handle interruptions. |

#### AnimationEvent Properties

| Property | Description |
|---|---|
| `animationName` | The name of the `@keyframes` animation. |
| `elapsedTime` | The amount of time the animation has been running, in seconds. |
| `pseudoElement` | The pseudo-element the animation applies to (if any). |
| `target` | The element the animation applies to. |

#### Syntax Rules

1. Events are fired on the element that has the animation.
2. `animationstart` fires after the delay period has expired.
3. `animationiteration` fires at the end of each iteration except the last.
4. `animationend` fires when the animation finishes.
5. `animationcancel` fires when the animation is interrupted or removed.
6. The `animationName` property identifies which animation the event belongs to.
7. All events are Baseline widely available.

#### Constraints and Limitations

- **No event for multiple properties** — each animation fires its own event; you must filter by `animationName`.
- **Cancellation behaviour** — `animationcancel` does not fire if the animation completes normally.
- **Timing precision** — `elapsedTime` may not be exactly the duration due to frame timing.
- **Vendor prefixes** — older browsers required `webkitAnimationEnd`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Using `animationend` to Remove an Element

**HTML File (`animation-events.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animation Events</title>
    <link rel="stylesheet" href="animation-events.css">
</head>
<body>
    <button class="add-btn">Add Toast</button>
    <div class="toast-container"></div>

    <script>
        const container = document.querySelector('.toast-container');
        const addBtn = document.querySelector('.add-btn');

        addBtn.addEventListener('click', () => {
            const toast = document.createElement('div');
            toast.className = 'toast';
            toast.textContent = 'This toast will disappear!';
            toast.addEventListener('animationstart', () => {
                console.log('animationstart: Toast appeared');
            });
            toast.addEventListener('animationend', (event) => {
                console.log(`animationend: ${event.animationName} finished in ${event.elapsedTime}s`);
                toast.remove();
                console.log('Toast removed from the DOM.');
            });
            toast.addEventListener('animationcancel', () => {
                console.log('animationcancel: Toast animation was cancelled');
            });
            container.appendChild(toast);
        });
    </script>
</body>
</html>
```

**CSS File (`animation-events.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.add-btn {
    padding: 12px 28px;
    font-size: 1rem;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    margin-bottom: 20px;
}

.toast-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

@keyframes toast-in-out {
    0% {
        opacity: 0;
        transform: translateY(-20px);
    }
    15% {
        opacity: 1;
        transform: translateY(0);
    }
    85% {
        opacity: 1;
        transform: translateY(0);
    }
    100% {
        opacity: 0;
        transform: translateY(-20px);
    }
}

.toast {
    padding: 16px 24px;
    background-color: #006064;
    color: white;
    border-radius: 10px;
    font-weight: bold;

    animation: toast-in-out 3s ease-in-out forwards;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `animation-events.html` and CSS as `animation-events.css`.
2. Open in a browser.
3. Open DevTools → Console.
4. Click "Add Toast". Observe the console logs: `animationstart`, then `animationend` after 3 seconds, then the toast is removed from the DOM.

**Expected Output:** A toast notification that animates in, stays visible for a few seconds, animates out, and is automatically removed from the DOM when the animation ends.

**Why This Works:** The `animationstart` event fires when the toast animation begins. The `animationend` event fires when the animation completes, and the script removes the toast from the DOM. The `animationcancel` event would fire if the animation were interrupted (e.g., if the toast were removed before the animation finished).

---

### Real-World Cases

- **Toast notifications:** Using `animationend` to remove toasts after their exit animation.
- **Progress indicators:** Using `animationiteration` to count iterations and update a progress bar.
- **Page transitions:** Using `animationend` to navigate to a new page after a transition animation.
- **Interactive feedback:** Using `animationstart` to show a loading state.

---

## References

- MDN Web Docs — `@keyframes` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@keyframes
- MDN Web Docs — `animation` - https://developer.mozilla.org/en-US/docs/Web/CSS/animation
- MDN Web Docs — `animation-fill-mode` - https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode
- MDN Web Docs — `animation-direction` - https://developer.mozilla.org/en-US/docs/Web/CSS/animation-direction
- MDN Web Docs — `animation-composition` - https://developer.mozilla.org/en-US/docs/Web/CSS/animation-composition
- MDN Web Docs — AnimationEvent - https://developer.mozilla.org/en-US/docs/Web/API/AnimationEvent
- W3C — CSS Animations Module Level 1 - https://www.w3.org/TR/css-animations-1/
- W3C — CSS Animations Module Level 2 - https://www.w3.org/TR/css-animations-2/
- Chrome for Developers — Combine multiple animation effects with `animation-composition` - https://developer.chrome.com/docs/css-ui/css-animation-composition
- CSS-Tricks — `animation` - https://css-tricks.com/almanac/properties/a/animation/
- Can I Use — CSS Animation - https://caniuse.com/css-animation
- Can I Use — `animation-composition` - https://caniuse.com/mdn-css_properties_animation-composition