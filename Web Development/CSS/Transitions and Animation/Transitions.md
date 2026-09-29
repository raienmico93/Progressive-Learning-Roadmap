# CSS Transitions — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Transitions is a CSS module that defines how CSS properties change from one value to another over a specified duration. It allows authors to create smooth, gradual visual changes between two states of an element without using JavaScript or complex keyframe animations.

**Technical Definition:** The CSS Transitions Module Level 1 defines properties that specify the necessary parameters to complete a transition, including the property to transition, the duration, the timing function, and the delay. Transitions are triggered by state changes (such as `:hover`, `:focus`, `:active`, or class changes) and animate the affected properties from their current computed values to the new values. The browser interpolates the values at each frame using the specified timing function. Transitions are defined on the element itself (the base state) and apply whenever the specified property changes. The module also defines the `transition-behavior` property and the `@starting-style` at-rule for transitioning to and from `display: none` and other discrete properties.

**Beginner-Friendly Explanation:** A CSS transition is like a video editor's cross-fade between two clips. When you hover over a button, instead of the background colour changing instantly, it gradually fades from blue to green over 300 milliseconds. You tell the browser which property to watch, how long the change should take, and what kind of motion curve to use. The browser handles all the intermediate frames for you, creating a smooth animation without any JavaScript.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **State-triggered** | Transitions are initiated by changes in computed values, typically caused by pseudo-classes or class toggles. |
| **Interpolable properties** | Transitions work on properties whose values can be interpolated (e.g., `opacity`, `transform`, `color`). |
| **Defined on the base state** | Transition declarations are placed on the element's default state, not the state change. |
| **Composite-only performance** | Transitions on `transform` and `opacity` are handled by the compositor thread. |
| **Discrete property support** | `transition-behavior: allow-discrete` enables transitions on `display` and other discrete properties. |
| **Entry animations** | `@starting-style` defines the starting point for transitions when an element first appears. |
| **JavaScript events** | `transitionstart`, `transitionrun`, `transitionend`, and `transitioncancel` allow runtime integration. |

---

### Prerequisites

- **CSS Box Model** — content, padding, border, and margin.
- **CSS Properties and Values** — lengths, colours, timing functions, and interpolation.
- **CSS Pseudo-classes** — `:hover`, `:focus`, `:active`, `:checked`.
- **The Browser Rendering Pipeline** — style, layout, paint, and composite stages.
- **CSS Custom Properties** — optional, but useful for managing transition tokens.

---

### Related Programming Areas

- **CSS Animations** — `@keyframes` for autonomous, multi-step animation.
- **CSS Transforms** — the most common transition targets (`transform`, `opacity`).
- **Web Performance** — compositor thread and GPU acceleration.
- **Web Animations API** — programmatic control over transitions.
- **Accessibility** — `prefers-reduced-motion` for respecting user preferences.

---

### Core Concepts / Features

1. Core Transition Properties: `transition-property`, `transition-duration`, and `transition-delay`
2. Transition Shorthand: Strict Ordering Rules
3. Transitioning Discrete Properties: `transition-behavior: allow-discrete`
4. The `@starting-style` Rule: Entry States for Disconnected Elements
5. Transition Events: `transitionstart`, `transitionrun`, `transitionend`, and `transitioncancel`

---

## 1. Core Transition Properties: `transition-property`, `transition-duration`, and `transition-delay`

### Definitions

**Core Definition:** The core transition properties define what to animate (`transition-property`), how long the animation takes (`transition-duration`), and how long to wait before starting (`transition-delay`).

**Technical Definition:** The `transition-property` property specifies the name or names of the CSS properties to which transitions should be applied. The `transition-duration` property specifies the duration over which transitions should occur, expressed in seconds (`s`) or milliseconds (`ms`). The `transition-delay` property specifies an optional delay before the transition starts. Multiple properties can be transitioned simultaneously by providing comma-separated lists for each property, with each list item corresponding to the property at the same index. The `transition-timing-function` property (often used alongside these) specifies the easing curve for the interpolation.

**Beginner-Friendly Explanation:** `transition-property` is the "what" — it tells the browser which CSS property to watch. `transition-duration` is the "how long" — it tells the browser how many seconds or milliseconds the change should take. `transition-delay` is the "when" — it tells the browser to wait a bit before starting. If you want to transition multiple properties with different timings, you provide comma-separated lists, and the browser matches them up by order.

---

### Purposes

- To specify which CSS properties should animate when their values change.
- To control the total duration of the transition.
- To add a delay before the transition begins.
- To animate multiple properties with different durations and delays.
- To provide the foundational parameters that the `transition` shorthand combines.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition-property: none | all | <custom-ident>#;
    transition-duration: <time>#;
    transition-delay: <time>#;
    transition-timing-function: <easing-function>#;
}
```

#### Component Breakdown

| Property | Accepted Values | Description |
|---|---|---|
| `transition-property` | `none`, `all`, `<custom-ident>#` | The property or properties to transition. |
| `transition-duration` | `<time>#` | The duration of the transition in seconds or milliseconds. |
| `transition-delay` | `<time>#` | The delay before the transition starts. |
| `transition-timing-function` | `<easing-function>#` | The easing curve. |

#### Syntax Rules

1. `transition-property` accepts a comma-separated list of property names or the keywords `none` or `all`.
2. `transition-duration` accepts a comma-separated list of time values; the default is `0s` (no transition).
3. `transition-delay` accepts a comma-separated list of time values; the default is `0s` (no delay).
4. When multiple properties are specified, each list item is matched by index across the sub-properties.
5. If a list is shorter than the longest list, the values are repeated cyclically.
6. If no `transition-property` is set, `all` is used by default when a duration is specified.
7. `transition-property: none` disables all transitions.
8. Negative values are not allowed for `transition-duration` but are allowed for `transition-delay` (negative delay causes the transition to start partway through).

#### Constraints and Limitations

- **Non-interpolable properties** — properties like `display` and `visibility` are discrete and cannot be transitioned without `transition-behavior: allow-discrete`.
- **Layout-triggering properties** — transitioning `width`, `height`, `top`, or `left` triggers layout on every frame and should be avoided in favour of `transform`.
- **Performance-sensitive** — only `transform` and `opacity` are compositor-only and can be animated without layout or paint work.
- **Default value** — `transition-duration` defaults to `0s`, so a transition declaration without a duration has no effect.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Single Property Transition with Delay

**HTML File (`core-properties.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Core Transition Properties</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="core-properties.css">
</head>
<body>
    <!-- Button with a delayed background-color transition -->
    <button class="btn">Hover Me</button>
</body>
</html>
```

**CSS File (`core-properties.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.btn {
    padding: 16px 40px;
    font-size: 1rem;
    font-weight: bold;
    color: white;
    background-color: #3498db;
    border: none;
    border-radius: 10px;
    cursor: pointer;

    /* The property to transition */
    transition-property: background-color;
    /* The duration of the transition */
    transition-duration: 400ms;
    /* A 100ms delay before the transition starts */
    transition-delay: 100ms;
    /* The easing curve */
    transition-timing-function: ease-in-out;
}

.btn:hover {
    /* The target value that triggers the transition */
    background-color: #e74c3c;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `core-properties.html`.
3. Save the CSS code as `core-properties.css` in the same folder.
4. Open `core-properties.html` in a web browser.
5. Hover over the button. Observe that after a 100ms delay, the background colour smoothly transitions from blue to red over 400ms.

**Expected Output:** A blue button that, on hover, waits 100ms and then smoothly fades to red over 400ms.

**Why This Works:** The `transition-property: background-color` tells the browser to watch only the `background-color` property. The `transition-duration: 400ms` sets the total animation time. The `transition-delay: 100ms` makes the browser wait before starting. The `transition-timing-function: ease-in-out` smooths the start and end of the animation.

---

#### Example 2: Multiple Properties with Different Timings

**HTML File (`multi-property.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multiple Transition Properties</title>
    <link rel="stylesheet" href="multi-property.css">
</head>
<body>
    <div class="card">Hover to transform</div>
</body>
</html>
```

**CSS File (`multi-property.css`):**

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
    width: 260px;
    padding: 40px 24px;
    background-color: white;
    color: #1a1a1a;
    border-radius: 16px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    text-align: center;
    font-weight: bold;
    cursor: pointer;

    /* Multiple properties with different durations */
    transition-property: transform, box-shadow, background-color;
    transition-duration: 200ms, 300ms, 500ms;
    transition-timing-function: ease-out, ease-out, linear;
    transition-delay: 0ms, 0ms, 100ms;
}

.card:hover {
    transform: translateY(-8px) scale(1.02);
    box-shadow: 0 16px 40px rgba(0, 0, 0, 0.15);
    background-color: #f0f7ff;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `multi-property.html` and CSS as `multi-property.css`.
2. Open in a browser.
3. Hover over the card. Observe that the transform completes quickly (200ms), the shadow takes a bit longer (300ms), and the background colour change is delayed by 100ms and takes 500ms.

**Expected Output:** A card that lifts and scales in 200ms, grows a larger shadow in 300ms, and fades its background colour after a 100ms delay over 500ms.

**Why This Works:** The comma-separated lists for each sub-property allow different properties to have different timings. The browser matches each property by index: `transform` gets 200ms, `box-shadow` gets 300ms, and `background-color` gets 500ms with a 100ms delay. This creates a layered, choreographed effect.

---

### Real-World Cases

- **Button hover effects:** Transitioning `background-color` and `transform` for a tactile feel.
- **Card interactions:** Transitioning `box-shadow` and `transform` for a lift effect.
- **Form focus states:** Transitioning `border-color` and `box-shadow` for smooth focus rings.
- **Navigation menus:** Transitioning `opacity` and `transform` for dropdown reveals.

---

## 2. Transition Shorthand: Mastering the Strict Ordering Rules

### Definitions

**Core Definition:** The `transition` shorthand property sets `transition-property`, `transition-duration`, `transition-timing-function`, and `transition-delay` in a single declaration, with strict ordering rules to resolve the ambiguity between duration and delay.

**Technical Definition:** The `transition` CSS property is a shorthand property for `transition-property`, `transition-duration`, `transition-timing-function`, and `transition-delay`. The shorthand accepts one or more comma-separated transitions. Within each transition, the first time value encountered is assigned to `transition-duration`, and the second time value is assigned to `transition-delay`. The `<easing-function>` value is assigned to `transition-timing-function`, and the `<custom-ident>` or keyword is assigned to `transition-property`. The order of values within each transition is flexible, except for the two time values, which must be in the order duration then delay.

**Beginner-Friendly Explanation:** The `transition` shorthand is convenient, but the two time values are ambiguous. To resolve this, the browser uses a strict rule: the first time value is the duration, and the second time value is the delay. So `transition: opacity 1s 200ms ease` means 1 second duration and 200 milliseconds delay. If you write `transition: opacity 200ms 1s ease`, that would mean 200ms duration and 1s delay. The order of the two time values matters, and getting it wrong swaps the timing.

---

### Purposes

- To write transition declarations concisely in a single line.
- To reduce repetition when setting all four transition sub-properties.
- To apply multiple transitions with different values in one declaration.
- To simplify stylesheets and improve readability.
- To provide the standard syntax used in most CSS frameworks and examples.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition: <single-transition>#;
}

<single-transition> = [ none | <single-transition-property> ] || <time> || <easing-function> || <time>
```

#### Component Breakdown

| Component | Description | Position in Shorthand |
|---|---|---|
| `<single-transition-property>` | The property to transition. | Any position. |
| `<time>` (first) | The duration. | First time value. |
| `<easing-function>` | The timing function. | Any position. |
| `<time>` (second) | The delay. | Second time value. |

#### Syntax Rules

1. The first time value in each transition is always the **duration**.
2. The second time value in each transition is always the **delay**.
3. If only one time value is present, it is the duration and the delay defaults to `0s`.
4. The property name can appear before, between, or after the time values and easing function.
5. Multiple transitions are separated by commas.
6. The default values are: `transition-property: all`, `transition-duration: 0s`, `transition-timing-function: ease`, `transition-delay: 0s`.
7. If the property name is omitted, `all` is assumed.
8. If the easing function is omitted, `ease` is assumed.

#### Constraints and Limitations

- **Time value ambiguity** — the order of the two time values is the only way to distinguish duration from delay.
- **No explicit labels** — unlike some other shorthands, the `transition` shorthand does not support labels.
- **Readability** — long comma-separated shorthand declarations can be difficult to read; consider using longhand properties for complex transitions.
- **Browser support** — the shorthand is supported in all modern browsers.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Duration vs. Delay Ordering

**HTML File (`shorthand-order.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Transition Shorthand Ordering</title>
    <link rel="stylesheet" href="shorthand-order.css">
</head>
<body>
    <!-- Correct ordering: duration then delay -->
    <div class="box correct">1s duration, 500ms delay</div>
    <!-- Incorrect ordering: swapped -->
    <div class="box incorrect">500ms duration, 1s delay</div>
</body>
</html>
```

**CSS File (`shorthand-order.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    gap: 40px;
}

.box {
    width: 200px;
    padding: 40px 20px;
    background-color: #3498db;
    color: white;
    border-radius: 12px;
    text-align: center;
    font-weight: bold;
    cursor: pointer;
}

.correct {
    /* First time = duration (1s), second time = delay (500ms) */
    transition: transform 1s 500ms ease-out;
}

.incorrect {
    /* First time = duration (500ms), second time = delay (1s) */
    transition: transform 500ms 1s ease-out;
}

.box:hover {
    transform: scale(1.2) rotate(10deg);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `shorthand-order.html` and CSS as `shorthand-order.css`.
2. Open in a browser.
3. Hover over the left box. Observe that it waits 500ms and then slowly scales and rotates over 1 second.
4. Hover over the right box. Observe that it waits 1 second and then quickly scales and rotates over 500ms.

**Expected Output:** Two boxes with dramatically different timing behaviour. The left box is slow with a short delay; the right box is fast with a long delay.

**Why This Works:** The `transition` shorthand interprets the first time value as the duration and the second as the delay. In `.correct`, `1s 500ms` means 1 second duration and 500ms delay. In `.incorrect`, `500ms 1s` means 500ms duration and 1 second delay. Swapping the order swaps the timing.

---

### Real-World Cases

- **Standard hover effects:** `transition: all 0.3s ease` for simple, uniform transitions.
- **Complex multi-property effects:** `transition: transform 0.3s ease-out, box-shadow 0.5s ease-in-out, opacity 0.2s linear` for layered motion.
- **Design systems:** Defining reusable transition tokens as custom properties and using them in the shorthand.
- **Component libraries:** Using the shorthand for consistent transition behaviour across components.

---

## 3. Transitioning Discrete Properties: `transition-behavior: allow-discrete`

### Definitions

**Core Definition:** The `transition-behavior` property with the `allow-discrete` value enables transitions on discrete properties (like `display` and `visibility`) that do not normally interpolate between values.

**Technical Definition:** The `transition-behavior` CSS property specifies whether transitions will be started for properties whose animation behavior is discrete. The `allow-discrete` value allows transitions to be started for discrete properties. When a discrete property transitions, the property flips from its start value to its end value at the 50% point of the transition. This is particularly useful for `display: none` because the element must remain rendered during the transition so its other properties (like `opacity` or `transform`) can animate before the element is removed from the layout. The `@starting-style` rule is often used in conjunction with `transition-behavior: allow-discrete` to define the entry state for elements transitioning from `display: none`.

**Beginner-Friendly Explanation:** Normally, `display: none` cannot be transitioned — it is either on or off. With `transition-behavior: allow-discrete`, the browser keeps the element visible during the transition, allowing properties like `opacity` and `transform` to animate. The `display` property itself flips from `none` to `block` at the 50% point, but by then the other properties have already animated. This is how you create smooth enter and exit animations for modals, dialogs, and popovers.

---

### Purposes

- To enable smooth enter and exit animations for elements that use `display: none`.
- To transition `visibility` and other discrete properties.
- To animate the appearance and disappearance of dialogs, modals, and popovers.
- To combine `display` transitions with `opacity` and `transform` for complete enter/exit choreography.
- To provide a native CSS solution for animations that previously required JavaScript.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    transition-behavior: normal | allow-discrete;
    /* Shorthand with allow-discrete */
    transition: <property> <duration> <easing> <delay> allow-discrete;
}
```

#### Component Breakdown

| Value | Description |
|---|---|
| `normal` | Default. Discrete properties do not transition. |
| `allow-discrete` | Discrete properties transition at the 50% point. |

#### Syntax Rules

1. `transition-behavior: allow-discrete` allows discrete properties to transition.
2. The discrete property flips at the 50% point of the transition duration.
3. To animate an element from `display: none` to `display: block`, use `allow-discrete` and ensure the other properties (`opacity`, `transform`) have transition durations.
4. The `@starting-style` rule defines the starting values for the entry transition.
5. The `transition` shorthand accepts `allow-discrete` as a value.
6. Browser support: Chrome 117+, Safari 17.4+, Firefox 129+.

#### Constraints and Limitations

- **Browser support** — `transition-behavior` is relatively new; older browsers ignore it and show instant state changes.
- **Fallback behaviour** — without `allow-discrete`, the element appears and disappears instantly.
- **Timing** — the discrete property flips at 50%, which may not be the desired timing for all animations.
- **Performance** — transitioning `display` does not trigger layout when used with `@starting-style` and `allow-discrete` because the element is kept in the layout during the transition.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Smooth Modal Enter and Exit

**HTML File (`allow-discrete.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>allow-discrete Modal</title>
    <link rel="stylesheet" href="allow-discrete.css">
</head>
<body>
    <button class="open-btn" onclick="document.querySelector('.modal').showModal()">
        Open Modal
    </button>

    <dialog class="modal">
        <h2>Modal Dialog</h2>
        <p>This dialog animates in and out smoothly.</p>
        <button onclick="document.querySelector('.modal').close()">Close</button>
    </dialog>
</body>
</html>
```

**CSS File (`allow-discrete.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.open-btn {
    padding: 14px 28px;
    font-size: 1rem;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}

.modal {
    /* Start hidden */
    opacity: 0;
    transform: translateY(20px) scale(0.95);

    /* Transition opacity and transform, plus display with allow-discrete */
    transition: opacity 300ms ease-out,
                transform 300ms ease-out,
                display 300ms allow-discrete,
                overlay 300ms allow-discrete;
}

.modal[open] {
    opacity: 1;
    transform: translateY(0) scale(1);
}

/* Starting style for the entry transition */
@starting-style {
    .modal[open] {
        opacity: 0;
        transform: translateY(20px) scale(0.95);
    }
}

/* Dialog backdrop */
.modal::backdrop {
    background-color: rgba(0, 0, 0, 0.5);
    transition: background-color 300ms ease-out;
}

@starting-style {
    .modal[open]::backdrop {
        background-color: rgba(0, 0, 0, 0);
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `allow-discrete.html` and CSS as `allow-discrete.css`.
2. Open in a browser that supports `allow-discrete` (Chrome 117+, Safari 17.4+).
3. Click "Open Modal". Observe the modal fades in, slides up, and scales up smoothly.
4. Click "Close". Observe the modal fades out, slides down, and scales down.

**Expected Output:** A modal dialog that animates smoothly when opened and closed, using native `<dialog>` element and `transition-behavior: allow-discrete`.

**Why This Works:** The `display` property is transitioned with `allow-discrete`, which keeps the element rendered during the transition. The `opacity` and `transform` properties animate smoothly. The `@starting-style` rule defines the entry state so the browser knows what values to transition from when the element first becomes visible.

---

### Real-World Cases

- **Dialog modals:** Smooth enter and exit animations for native `<dialog>` elements.
- **Popovers:** Animating the appearance and disappearance of popover content.
- **Dropdown menus:** Fading and sliding menus in and out of view.
- **Toast notifications:** Sliding notifications in and out with `display: none` toggling.

---

## 4. The `@starting-style` Rule: Setting Up Entry States for Elements Transitioning from a Disconnected State

### Definitions

**Core Definition:** The `@starting-style` at-rule defines the starting values for a transition when an element first appears or when it transitions from `display: none`. It solves the problem of elements appearing instantly without animating because they had no previous value to transition from.

**Technical Definition:** The `@starting-style` CSS at-rule is used to define starting styles for an element when it is first rendered or when it transitions from `display: none`. Without `@starting-style`, the browser has no "before" value to transition from, so the element appears instantly at its final state. The `@starting-style` rule provides those initial values, allowing the browser to animate from them to the element's final state. It can be used at the top level of a stylesheet or nested inside a style rule.

**Beginner-Friendly Explanation:** When an element first appears on the page, the browser does not know what values to animate from. `@starting-style` tells the browser: "When this element first appears, start it at these values." For example, you can tell the browser that a modal should start with `opacity: 0` and `transform: translateY(20px)`, and then transition to `opacity: 1` and `transform: translateY(0)`. Without `@starting-style`, the modal would just pop into existence at full opacity.

---

### Purposes

- To define entry animations for elements that are first rendered or transitioned from `display: none`.
- To provide a "before" state for transitions when the element has no previous computed values.
- To enable smooth enter animations for dialogs, popovers, and dynamically inserted elements.
- To work in conjunction with `transition-behavior: allow-discrete` for complete enter/exit animations.
- To eliminate the need for JavaScript to set initial inline styles for entry animations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Top-level @starting-style */
@starting-style {
    selector {
        /* Starting values */
    }
}

/* Nested @starting-style */
selector {
    @starting-style {
        /* Starting values */
    }
}
```

#### Component Breakdown

| Usage | Description |
|---|---|
| Top-level | Applies globally to the specified selector. |
| Nested | Applies only when the parent selector matches. |

#### Syntax Rules

1. `@starting-style` must be used with a selector that matches the element.
2. The values inside `@starting-style` are the starting values for the transition.
3. The element must have a transition defined for the properties being animated.
4. `@starting-style` is used with `transition-behavior: allow-discrete` for `display: none` transitions.
5. The rule can be nested inside a style rule for scoped application.
6. Browser support: Chrome 117+, Safari 17.4+, Firefox 129+.
7. The `@starting-style` rule only applies when the element is first rendered or when it transitions from `display: none`.

#### Constraints and Limitations

- **Browser support** — relatively new; older browsers ignore the rule and show instant state changes.
- **Only for entry transitions** — `@starting-style` does not apply to exit transitions; use the element's default state for that.
- **Requires transition** — without a `transition` declaration on the element, `@starting-style` has no effect.
- **Specificity** — the values inside `@starting-style` are treated as having lower specificity than the element's normal styles.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Entry Animation for a Dialog

**HTML File (`starting-style.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@starting-style Dialog</title>
    <link rel="stylesheet" href="starting-style.css">
</head>
<body>
    <button onclick="document.querySelector('.dialog').showModal()">Open Dialog</button>

    <dialog class="dialog">
        <h2>Animated Dialog</h2>
        <p>This dialog animates in with a slide and fade.</p>
        <button onclick="document.querySelector('.dialog').close()">Close</button>
    </dialog>
</body>
</html>
```

**CSS File (`starting-style.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

button {
    padding: 12px 24px;
    font-size: 1rem;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
}

.dialog {
    /* Final state: visible and in position */
    opacity: 1;
    transform: translateY(0) scale(1);

    /* Transition all relevant properties */
    transition: opacity 400ms ease-out,
                transform 400ms ease-out,
                display 400ms allow-discrete,
                overlay 400ms allow-discrete;
}

/* Starting state for entry animation */
@starting-style {
    .dialog[open] {
        opacity: 0;
        transform: translateY(30px) scale(0.95);
    }
}

.dialog::backdrop {
    background-color: rgba(0, 0, 0, 0.5);
    transition: background-color 400ms ease-out;
}

@starting-style {
    .dialog[open]::backdrop {
        background-color: rgba(0, 0, 0, 0);
    }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `starting-style.html` and CSS as `starting-style.css`.
2. Open in a browser that supports `@starting-style` (Chrome 117+, Safari 17.4+).
3. Click "Open Dialog". Observe the dialog slides up, scales up, and fades in. The backdrop also fades in from transparent to dark.
4. Click "Close". Observe the dialog animates out.

**Expected Output:** A dialog that animates smoothly on entry using `@starting-style` to define the starting values, and `allow-discrete` to keep the dialog rendered during the transition.

**Why This Works:** The `@starting-style` rule defines the starting values (`opacity: 0`, `transform: translateY(30px) scale(0.95)`) for the dialog when it first becomes visible. The `transition` property animates from those values to the final values (`opacity: 1`, `transform: translateY(0) scale(1)`). The `transition-behavior: allow-discrete` on `display` and `overlay` keeps the dialog rendered during the transition. Without `@starting-style`, the dialog would appear instantly at full opacity.

---

### Real-World Cases

- **Native dialogs:** Entry animations for `<dialog>` elements.
- **Popovers:** Animating the appearance of popover content.
- **Dropdown menus:** Fading and sliding menus in on first open.
- **Toast notifications:** Sliding notifications in from the edge of the screen.

---

## 5. Transition Events: Binding Logic to JavaScript Runtime

### Definitions

**Core Definition:** Transition events are JavaScript events fired by the browser at different stages of a CSS transition, allowing developers to bind runtime logic to the transition lifecycle.

**Technical Definition:** The CSS Transitions specification defines four transition events: `transitionrun` (fired when the transition is created, before any delay), `transitionstart` (fired when the transition actually begins, after the delay), `transitionend` (fired when the transition completes), and `transitioncancel` (fired when the transition is cancelled). Each event is a `TransitionEvent` object with properties including `propertyName`, `elapsedTime`, `pseudoElement`, and `target`. These events allow JavaScript to synchronise application logic with CSS transition timing.

**Beginner-Friendly Explanation:** CSS transitions are visual, but sometimes you need to run JavaScript when a transition starts, ends, or is cancelled. For example, you might want to remove an element from the DOM after its exit animation completes, or update a status indicator when a transition finishes. The transition events give you hooks into the transition lifecycle. `transitionrun` fires when the transition is created, `transitionstart` fires when it actually begins, `transitionend` fires when it finishes, and `transitioncancel` fires when it is interrupted.

---

### Purposes

- To execute JavaScript logic when a transition starts (`transitionrun`, `transitionstart`).
- To perform cleanup or state updates when a transition completes (`transitionend`).
- To handle interrupted transitions (`transitioncancel`).
- To synchronise application state with visual animation state.
- To chain animations or trigger follow-up actions after a transition.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
element.addEventListener('transitionrun', (event) => { /* ... */ });
element.addEventListener('transitionstart', (event) => { /* ... */ });
element.addEventListener('transitionend', (event) => { /* ... */ });
element.addEventListener('transitioncancel', (event) => { /* ... */ });

/* Or using the on* properties */
element.ontransitionend = (event) => { /* ... */ };
```

#### Component Breakdown

| Event | Fired When | Use Case |
|---|---|---|
| `transitionrun` | The transition is created (before delay). | Pre-transition setup. |
| `transitionstart` | The transition begins (after delay). | Start visual feedback. |
| `transitionend` | The transition completes. | Cleanup, DOM removal, state update. |
| `transitioncancel` | The transition is cancelled. | Revert state, handle interruptions. |

#### TransitionEvent Properties

| Property | Description |
|---|---|
| `propertyName` | The name of the CSS property that transitioned. |
| `elapsedTime` | The amount of time the transition has been running, in seconds. |
| `pseudoElement` | The pseudo-element the transition applies to (if any). |
| `target` | The element the transition applies to. |

#### Syntax Rules

1. Events are fired on the element that has the transition.
2. `transitionrun` fires before any delay; `transitionstart` fires after the delay.
3. `transitionend` fires when the transition completes successfully.
4. `transitioncancel` fires when the transition is interrupted (e.g., the property changes again before completion).
5. The `propertyName` property identifies which transition ended.
6. Multiple transitions on the same element fire separate events.
7. Browser support: all modern browsers.

#### Constraints and Limitations

- **No event for multiple properties** — each property transition fires its own event; you must filter by `propertyName`.
- **Cancellation behaviour** — `transitioncancel` does not fire if the transition is never started (e.g., zero duration).
- **Timing precision** — `elapsedTime` may not be exactly the duration due to frame timing.
- **Vendor prefixes** — older browsers required `webkitTransitionEnd`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Using `transitionend` to Remove an Element

**HTML File (`transition-events.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Transition Events</title>
    <link rel="stylesheet" href="transition-events.css">
</head>
<body>
    <button class="remove-btn">Remove Card</button>
    <div class="card">I will animate out and then be removed.</div>

    <script>
        const card = document.querySelector('.card');
        const btn = document.querySelector('.remove-btn');

        card.addEventListener('transitionrun', () => {
            console.log('transitionrun: Transition created');
        });

        card.addEventListener('transitionstart', () => {
            console.log('transitionstart: Transition began');
        });

        card.addEventListener('transitionend', (event) => {
            console.log(`transitionend: ${event.propertyName} finished in ${event.elapsedTime}s`);
            if (event.propertyName === 'opacity') {
                card.remove();
                console.log('Card removed from the DOM.');
            }
        });

        card.addEventListener('transitioncancel', () => {
            console.log('transitioncancel: Transition was cancelled');
        });

        btn.addEventListener('click', () => {
            card.classList.add('removing');
        });
    </script>
</body>
</html>
```

**CSS File (`transition-events.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
}

.remove-btn {
    padding: 12px 24px;
    font-size: 1rem;
    background-color: #e74c3c;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    margin-bottom: 20px;
}

.card {
    width: 300px;
    padding: 40px 24px;
    background-color: #3498db;
    color: white;
    border-radius: 12px;
    text-align: center;
    font-weight: bold;

    transition: opacity 500ms ease-out,
                transform 500ms ease-out;
}

.card.removing {
    opacity: 0;
    transform: translateY(-20px) scale(0.9);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `transition-events.html` and CSS as `transition-events.css`.
2. Open in a browser.
3. Open DevTools → Console.
4. Click "Remove Card". Observe the console logs: `transitionrun`, `transitionstart`, then `transitionend` for both `opacity` and `transform`.
5. After the `opacity` transition ends, the card is removed from the DOM.

**Expected Output:** The card fades out and slides up. The console logs the transition lifecycle, and the card is removed from the DOM after the `opacity` transition ends.

**Why This Works:** The `transitionrun` event fires when the transition is created. `transitionstart` fires when the transition actually begins. `transitionend` fires when each property transition completes, and the script checks `event.propertyName` to remove the card only after the `opacity` transition finishes. `transitioncancel` would fire if the transition were interrupted.

---

### Real-World Cases

- **Animating elements out before removal:** Using `transitionend` to remove elements from the DOM after their exit animation.
- **Chaining animations:** Triggering the next animation when the previous one ends.
- **Progress indicators:** Using `transitionrun` and `transitionstart` to show loading states.
- **Cancellation handling:** Using `transitioncancel` to revert state when a user interrupts an animation.

---

## References

- MDN Web Docs — Using CSS transitions - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transitions/Using_CSS_transitions
- MDN Web Docs — `transition` - https://developer.mozilla.org/en-US/docs/Web/CSS/transition
- MDN Web Docs — `transition-property` - https://developer.mozilla.org/en-US/docs/Web/CSS/transition-property
- MDN Web Docs — `transition-duration` - https://developer.mozilla.org/en-US/docs/Web/CSS/transition-duration
- MDN Web Docs — `transition-delay` - https://developer.mozilla.org/en-US/docs/Web/CSS/transition-delay
- MDN Web Docs — `transition-behavior` - https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior
- MDN Web Docs — `@starting-style` - https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style
- MDN Web Docs — `transitionend` event - https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionend_event
- W3C — CSS Transitions Module Level 1 - https://www.w3.org/TR/css-transitions-1/
- W3C — CSS Transitions Module Level 2 - https://www.w3.org/TR/css-transitions-2/
- web.dev — Animating dialog and popover elements with CSS `@starting-style` - https://web.dev/articles/css-starting-style
- Chrome for Developers — Four new CSS features for smooth entry and exit animations - https://developer.chrome.com/blog/entry-exit-animations
- CSS-Tricks — Using CSS transitions - https://css-tricks.com/almanac/properties/t/transition/
- Can I Use — `@starting-style` - https://caniuse.com/css-starting-style
- Can I Use — `transition-behavior` - https://caniuse.com/mdn-css_properties_transition-behavior