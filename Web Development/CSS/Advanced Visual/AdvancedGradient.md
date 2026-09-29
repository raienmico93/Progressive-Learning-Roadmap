# CSS Advanced Gradients — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Advanced Gradients is the applied discipline of using modern CSS colour spaces, complex gradient functions, and layered background techniques to produce sophisticated visual effects — including ultra-smooth wide-gamut colour transitions, repeating geometric patterns, angular conic effects, and procedural micro-patterns — all without image assets.

**Technical Definition:** CSS gradients are defined in the CSS Images Module Level 3 and Level 4, and the CSS Color Module Level 4 and Level 5. Gradients are `<image>` data types created by functions including `linear-gradient()`, `radial-gradient()`, `conic-gradient()`, and their repeating variants (`repeating-linear-gradient()`, `repeating-radial-gradient()`, `repeating-conic-gradient()`). The `<color-interpolation-method>` data type allows authors to override the default colour interpolation space (Oklab) for gradients, enabling interpolation in perceptual colour spaces such as `oklch`, `oklab`, and `display-p3`. Colour interpolation hints (unlabelled percentages placed between two colour stops) control the midpoint and acceleration curve of colour transitions without requiring manual mathematical stops. Multiple gradient layers can be composited via `background-image` with `background-size` and `background-repeat` to construct procedural patterns.

**Beginner-Friendly Explanation:** CSS gradients have evolved far beyond simple two-colour fades. You can now create gradients that interpolate in perceptually uniform colour spaces, which means the colours stay vibrant and never turn muddy grey in the middle. You can build repeating stripes, concentric rings, pie charts, colour wheels, and even complex patterns like plaid and checkerboards — all with pure CSS. And you can control exactly where the midpoint of a colour transition falls, giving you fine-grained control over how colours blend. This cheat sheet covers all of these techniques with practical examples.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Wide-gamut colour support** | Gradients can interpolate in `oklch`, `oklab`, `display-p3`, and other modern colour spaces. |
| **Perceptual uniformity** | Oklab and Oklch prevent the "muddy middle" problem common in sRGB gradients. |
| **Multiple gradient types** | Linear, radial, and conic gradients, each with a repeating variant. |
| **Colour interpolation hints** | Unlabelled percentages control the midpoint of colour transitions. |
| **Layered backgrounds** | Multiple gradients can be stacked with `background-size` and `background-repeat`. |
| **Procedural patterns** | Checkerboards, stripes, dots, plaid, and mesh gradients built entirely from gradients. |
| **Baseline available** | Modern colour spaces and interpolation methods are Baseline 2023+. |

---

### Prerequisites

Before studying CSS Advanced Gradients, you should understand:

- **CSS Gradients fundamentals** — `linear-gradient()`, `radial-gradient()`, `conic-gradient()`.
- **CSS Colour Values** — hex, RGB, HSL, and the newer `oklch()`, `oklab()`, and `display-p3`.
- **CSS Backgrounds** — `background-image`, `background-size`, `background-repeat`, `background-position`.
- **CSS Box Model** — content, padding, border, and margin boxes.
- **Basic Colour Theory** — hue, saturation, lightness, and perceptual uniformity.

---

### Related Programming Areas

- **CSS Colour Module Level 5** — the specification defining modern colour spaces.
- **CSS Images Module Level 4** — the specification defining gradient functions and interpolation hints.
- **CSS Masks** — gradients are commonly used as mask images.
- **CSS Blend Modes** — gradients can be blended with `background-blend-mode`.
- **UI/UX Design** — gradients are fundamental to modern visual design.

---

### Core Concepts / Features

1. Modern Colour Spaces: Oklch, Oklab, and Display-P3
2. Complex Gradient Mathematics: `repeating-linear-gradient()` and `repeating-radial-gradient()`
3. Angular and Conic Patterns: `conic-gradient()`
4. Procedural Micro-Patterns: Layered Gradients
5. Colour Interpolation Hints: Midpoint and Acceleration Control

---

## 1. Modern Colour Spaces: Constructing Ultra-Smooth Gradient Interpolations Using Wide Gamut Profiles

### Definitions

**Core Definition:** Modern colour spaces in CSS gradients allow authors to specify the colour space in which gradient interpolation occurs. Using perceptual colour spaces like `oklch` and `oklab`, or wide-gamut spaces like `display-p3`, produces smoother, more vibrant gradients that avoid the muddy, grey midpoints common in sRGB interpolation.

**Technical Definition:** The `<color-interpolation-method>` CSS data type represents the colour space used for interpolation between `<color>` values. It can be used to override the default interpolation colour space for colour-related functional notations such as `color-mix()` and `linear-gradient()`. When interpolating `<color>` values, the interpolation colour space defaults to Oklab. The syntax is `in <rectangular-color-space>` or `in <polar-color-space> <hue-interpolation-method>`. Rectangular colour spaces include `srgb`, `srgb-linear`, `display-p3`, `a98-rgb`, `prophoto-rgb`, `rec2020`, `lab`, `oklab`, `xyz`, `xyz-d50`, and `xyz-d65`. Polar colour spaces include `hsl`, `hwb`, `lch`, and `oklch`. The Oklab colour space (and the older Lab) is designed to be perceptually uniform, and Oklch is preferred when maximising chroma throughout the transition is desired. The `display-p3` colour space offers a wider gamut than sRGB, enabling more saturated colours.

**Beginner-Friendly Explanation:** When you create a gradient from blue to yellow in sRGB (the default), the middle of the gradient often turns a muddy grey. That happens because sRGB is not perceptually uniform — the same numerical change in colour values does not correspond to the same visual change. Perceptual colour spaces like Oklch fix this. They are designed so that equal steps in colour values produce equal visual steps. The result is a gradient that stays vibrant and colourful all the way through. You enable this by writing `linear-gradient(in oklch, blue, yellow)` instead of just `linear-gradient(blue, yellow)`.

---

### Purposes

- To eliminate muddy, grey midpoints in gradients.
- To produce smoother, more visually even colour transitions.
- To access wider colour gamuts (display-p3) for more saturated colours.
- To control the hue interpolation path (shorter, longer, increasing, decreasing).
- To create vibrant, modern gradients that match the capabilities of modern displays.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    background: linear-gradient(in <color-space>, <color-stop>, <color-stop>);
    background: linear-gradient(in <polar-color-space> <hue-interpolation-method>, <color-stop>, <color-stop>);
}
```

#### Component Breakdown

| Component | Description | Values |
|---|---|---|
| `in <color-space>` | The interpolation colour space. | `srgb`, `srgb-linear`, `display-p3`, `a98-rgb`, `prophoto-rgb`, `rec2020`, `lab`, `oklab`, `xyz`, `xyz-d50`, `xyz-d65` |
| `in <polar-color-space>` | A polar colour space. | `hsl`, `hwb`, `lch`, `oklch` |
| `<hue-interpolation-method>` | Optional hue interpolation algorithm. | `shorter`, `longer`, `increasing`, `decreasing` |
| `<color-stop>` | A colour stop. | Any valid `<color>` value |

#### Common Wide-Gamut Colour Values

| Colour Space | Function | Example |
|---|---|---|
| Oklch | `oklch()` | `oklch(70% 0.2 250)` |
| Oklab | `oklab()` | `oklab(70% -0.1 -0.15)` |
| Display-P3 | `color()` | `color(display-p3 1 0.5 0)` |

#### Syntax Rules

1. The `in` keyword precedes the colour space name.
2. The default interpolation space is Oklab when not specified.
3. For polar colour spaces, an optional hue interpolation method can be specified.
4. `shorter` (the default) takes the shortest arc between hues.
5. `longer` takes the longer arc, which can produce more colourful transitions.
6. `oklch` with `longer hue` maximises chroma throughout the transition.
7. The `display-p3` colour space supports more saturated colours than sRGB.
8. `<color-interpolation-method>` is Baseline 2023 Newly Available.

#### Constraints and Limitations

- **Browser support** — Oklch, Oklab, and interpolation methods are relatively new; older browsers ignore them.
- **Gamut clipping** — colours outside the display's gamut may be clipped, producing unexpected results.
- **Performance** — wide-gamut gradients are computationally similar to sRGB gradients.
- **Fallback required** — always provide a fallback for browsers that do not support modern colour spaces.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Oklch Gradient vs. sRGB Gradient

**HTML File (`oklch-gradient.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Oklch Gradient</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="oklch-gradient.css">
</head>
<body>
    <h3>sRGB (muddy middle)</h3>
    <div class="gradient srgb"></div>

    <h3>Oklch (vibrant throughout)</h3>
    <div class="gradient oklch"></div>
</body>
</html>
```

**CSS File (`oklch-gradient.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

h3 {
    font-size: 0.9rem;
    color: #555;
    margin-bottom: 5px;
    margin-top: 20px;
}

.gradient {
    height: 100px;
    border-radius: 12px;
}

.srgb {
    /* Default sRGB interpolation — often produces a muddy middle */
    background: linear-gradient(90deg, #0033ff, #ffcc00);
}

.oklch {
    /* Oklch interpolation — vibrant and perceptually even */
    background: linear-gradient(in oklch, #0033ff, #ffcc00);
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `oklch-gradient.html`.
3. Save the CSS code as `oklch-gradient.css` in the same folder.
4. Open `oklch-gradient.html` in a modern browser.
5. Compare the two gradients: the sRGB gradient likely has a darker, greyer middle, while the Oklch gradient stays vibrant.

**Expected Output:** Two gradient bars. The top (sRGB) gradient transitions from blue to yellow with a muddy, desaturated middle. The bottom (Oklch) gradient transitions from blue to yellow with vibrant, saturated colours throughout.

**Why This Works:** The `linear-gradient(in oklch, #0033ff, #ffcc00)` specifies that interpolation should occur in the Oklch colour space. Because Oklch is perceptually uniform, the intermediate colours appear equally spaced in visual perception, avoiding the desaturated midpoint that occurs in sRGB.

---

#### Example 2: Display-P3 Wide Gamut Gradient

**HTML File (`p3-gradient.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Display-P3 Gradient</title>
    <link rel="stylesheet" href="p3-gradient.css">
</head>
<body>
    <div class="p3-gradient"></div>
</body>
</html>
```

**CSS File (`p3-gradient.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.p3-gradient {
    height: 150px;
    border-radius: 12px;
    /* Interpolate in display-p3 for wider gamut colours */
    background: linear-gradient(
        in display-p3,
        color(display-p3 1 0 0),
        color(display-p3 0 1 0)
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `p3-gradient.html` and CSS as `p3-gradient.css`.
2. Open in a browser on a P3-capable display (modern MacBooks, iPads, and many smartphones).
3. Observe that the red-to-green gradient is more vibrant than an sRGB equivalent.

**Expected Output:** A red-to-green gradient using the display-p3 colour space. On a P3 display, the colours are noticeably more saturated than sRGB.

**Why This Works:** The `in display-p3` clause tells the browser to interpolate in the Display-P3 colour space, which has a wider gamut than sRGB. The `color(display-p3 ...)` functions define the endpoint colours in the P3 space. On a P3-capable display, the result is more saturated and vibrant.

---

### Real-World Cases

- **Brand gradients:** Vibrant gradients that maintain brand colours without muddy midpoints.
- **Dark mode:** Perceptually smooth gradients that look even in both light and dark themes.
- **Data visualisation:** Colour scales that are perceptually uniform for accurate data representation.
- **Hero sections:** Wide-gamut gradients that pop on modern displays.

---

## 2. Complex Gradient Mathematics: `repeating-linear-gradient()` and `repeating-radial-gradient()` Scales Alongside Sharp, Sub-Pixel Colour Stops

### Definitions

**Core Definition:** Repeating gradients are gradient functions that repeat their colour stop list infinitely in both directions. When combined with sharp colour stops (two stops at the same position), they produce hard-edged stripes, rings, and geometric patterns.

**Technical Definition:** The `repeating-linear-gradient()` and `repeating-radial-gradient()` functions create images consisting of repeating gradients. They take the same values and are interpreted the same as their respective non-repeating siblings. When rendered, the colour stops are repeated infinitely in both directions, with their positions shifted by multiples of the difference between the last colour stop and the first. Sharp transitions are achieved by placing two colour stops at the same position: the browser applies the "0deg color stop fixup," where a colour stop with a position less than the previous stop is fixed up to be equal to the position of the colour stop before it, producing infinitesimal transitions between colour stops with different colours, effectively producing solid colour segments. This technique is used to create stripes, barber poles, concentric rings, and starburst patterns.

**Beginner-Friendly Explanation:** Imagine a striped shirt. The stripes repeat across the fabric. `repeating-linear-gradient()` works the same way — you define one set of stripes and the browser repeats them infinitely. The "sharp stops" trick is how you make the stripes have hard edges instead of soft fades. Instead of `red 0%, blue 100%`, you write `red 0%, red 50%, blue 50%, blue 100%` — the colour changes instantly at the 50% mark. Combined with repeating gradients, this creates crisp stripes, rings, and geometric patterns.

---

### Purposes

- To create repeating stripe patterns (horizontal, vertical, diagonal).
- To build concentric ring patterns with `repeating-radial-gradient()`.
- To produce sharp, hard-edged colour transitions without fades.
- To create barber-pole and starburst effects.
- To build efficient patterns that repeat automatically.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Repeating linear gradient */
selector {
    background: repeating-linear-gradient(
        <angle> | to <side-or-corner>,
        <color-stop>, <color-stop>, ...
    );
}

/* Repeating radial gradient */
selector {
    background: repeating-radial-gradient(
        <shape> <size> at <position>,
        <color-stop>, <color-stop>, ...
    );
}

/* Sharp colour stops (two stops at the same position) */
background: repeating-linear-gradient(
    90deg,
    red 0px, red 20px,
    blue 20px, blue 40px
);
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<angle>` | The direction of the gradient line. | `45deg`, `90deg` |
| `<shape>` | The shape of the radial gradient. | `circle`, `ellipse` |
| `<size>` | The size of the ending shape. | `closest-side`, `farthest-corner` |
| `<color-stop>` | A colour and optional position. | `red 0px`, `blue 20px` |
| Sharp stop | Two stops at the same position. | `red 20px, blue 20px` |

#### Syntax Rules

1. `repeating-linear-gradient()` takes the same arguments as `linear-gradient()`.
2. `repeating-radial-gradient()` takes the same arguments as `radial-gradient()`.
3. The colour stops repeat infinitely in both directions, shifted by the difference between the last and first stop.
4. Sharp transitions are created by placing two different colours at the same stop position.
5. The "0deg color stop fixup" ensures that a stop at a lower position is fixed up to equal the previous stop's position.
6. This technique is used to create stripes, barber poles, and checkerboard patterns.
7. Repeating gradients are Baseline widely available.

#### Constraints and Limitations

- **Pattern size** — the repeating pattern's size is determined by the distance between the first and last colour stops.
- **Angle precision** — diagonal repeating gradients can produce moiré patterns on some displays.
- **Performance** — repeating gradients are rendered efficiently by the browser; no noticeable overhead.
- **Sharp stops and anti-aliasing** — sharp transitions may appear slightly jagged on low-resolution displays.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Diagonal Stripes with Sharp Stops

**HTML File (`stripes.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sharp Stripe Gradient</title>
    <link rel="stylesheet" href="stripes.css">
</head>
<body>
    <div class="stripes"></div>
</body>
</html>
```

**CSS File (`stripes.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.stripes {
    width: 300px;
    height: 200px;
    border-radius: 12px;
    /* Sharp diagonal stripes with repeating-linear-gradient */
    background: repeating-linear-gradient(
        45deg,
        #3498db 0px, #3498db 20px,
        #2980b9 20px, #2980b9 40px
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `stripes.html` and CSS as `stripes.css`.
2. Open in a browser.
3. Observe the crisp diagonal stripes in two shades of blue.

**Expected Output:** A rectangular element filled with sharp, 45-degree diagonal stripes alternating between two blue shades.

**Why This Works:** The `repeating-linear-gradient(45deg, ...)` creates a gradient at a 45-degree angle. The colour stops `#3498db 0px, #3498db 20px, #2980b9 20px, #2980b9 40px` define a 40px repeating pattern: 20px of the first blue, then 20px of the second blue, repeating infinitely. Because the stops share positions (20px appears twice), the transition is instant, creating sharp edges.

---

#### Example 2: Concentric Rings with `repeating-radial-gradient()`

**HTML File (`rings.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Concentric Rings</title>
    <link rel="stylesheet" href="rings.css">
</head>
<body>
    <div class="rings"></div>
</body>
</html>
```

**CSS File (`rings.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.rings {
    width: 250px;
    height: 250px;
    border-radius: 50%;
    /* Concentric rings with repeating-radial-gradient */
    background: repeating-radial-gradient(
        circle at center,
        #e74c3c 0px, #e74c3c 15px,
        #c0392b 15px, #c0392b 30px
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `rings.html` and CSS as `rings.css`.
2. Open in a browser.
3. Observe the concentric red rings radiating from the centre.

**Expected Output:** A circular element filled with concentric rings of two red shades, repeating every 30px.

**Why This Works:** The `repeating-radial-gradient(circle at center, ...)` creates rings radiating from the centre. The colour stops define a 30px repeating pattern: 15px of the first red, then 15px of the second red. The pattern repeats infinitely outward, creating concentric rings.

---

### Real-World Cases

- **Barber-pole animations:** `repeating-linear-gradient()` with animated `background-position`.
- **Progress indicators:** Repeating stripes for loading bars.
- **Fabric textures:** Repeating gradients for plaid, gingham, and denim effects.
- **Concentric UI elements:** Rings for avatars, badges, and decorative borders.

---

## 3. Angular and Conic Patterns: Building Intricate Pie Charts, Colour Wheels, and Metallic Ray Textures Using `conic-gradient()`

### Definitions

**Core Definition:** The `conic-gradient()` CSS function creates an image consisting of a gradient with colour transitions rotated around a centre point. It is used to create pie charts, colour wheels, loading spinners, and angular patterns.

**Technical Definition:** The `conic-gradient()` function creates an image consisting of a gradient with colour transitions rotated around a centre point (rather than radiating from the centre). Example conic gradients include pie charts and colour wheels. The syntax is `conic-gradient([from <angle>]? [at <position>]?, <angular-color-stop-list>)`. The `from` keyword defines the gradient rotation in a clockwise direction. The `at` keyword defines the centre of the gradient. Angular colour stops are specified with angles (`0deg` to `360deg`) or percentages. Sharp transitions (hard stops) are created by placing two colours at the same angle. The `repeating-conic-gradient()` function repeats the pattern around the full 360-degree rotation.

**Beginner-Friendly Explanation:** Imagine a clock face. `conic-gradient()` paints colours around the face, starting at 12 o'clock and moving clockwise. You can create a pie chart by setting hard stops at specific angles — for example, `red 0deg 90deg, blue 90deg 180deg, green 180deg 270deg, yellow 270deg 360deg` creates a four-colour pie chart. You can create a colour wheel by transitioning through the full hue spectrum. And with `repeating-conic-gradient()`, you can create starburst patterns and metallic ray textures.

---

### Purposes

- To create pie charts and donut charts with hard colour stops.
- To build colour wheels and hue pickers.
- To create loading spinners with angular gradient trails.
- To produce starburst and ray textures with repeating conic gradients.
- To build metallic and radial UI effects.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    background: conic-gradient(
        [from <angle>]? [at <position>]?,
        <angular-color-stop-list>
    );
    background: repeating-conic-gradient(
        [from <angle>]? [at <position>]?,
        <angular-color-stop-list>
    );
}
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `from <angle>` | Starting angle (clockwise). | `from 45deg` |
| `at <position>` | Centre position. | `at 50% 50%` |
| `<angular-color-stop>` | A colour with an optional angle or percentage. | `red 0deg 90deg` |
| Hard stop | Two colours at the same angle. | `red 90deg, blue 90deg` |

#### Syntax Rules

1. The `from` keyword sets the starting angle (default is 0deg, which is 12 o'clock).
2. The `at` keyword sets the centre of the gradient (default is `center`).
3. Angular stops are specified with angles (`0deg` to `360deg`) or percentages.
4. Hard stops (two colours at the same angle) create sharp edges.
5. `repeating-conic-gradient()` repeats the pattern around the full 360-degree rotation.
6. The `conic-gradient()` function is Baseline widely available since November 2020.
7. Conic gradients are excellent for pie charts, colour wheels, and loading indicators.

#### Constraints and Limitations

- **Pie charts as background images are not accessible** — conic gradients are background images and cannot convey semantic information to assistive technology. Use them decoratively only.
- **Anti-aliasing** — hard conic stops may appear slightly jagged on low-resolution displays.
- **Browser support** — widely available, but older browsers may not support `from` and `at` syntax.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Pie Chart with Hard Stops

**HTML File (`pie-chart.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Conic Pie Chart</title>
    <link rel="stylesheet" href="pie-chart.css">
</head>
<body>
    <div class="pie"></div>
</body>
</html>
```

**CSS File (`pie-chart.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.pie {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    /* Four-segment pie chart with hard stops */
    background: conic-gradient(
        #3498db 0deg 90deg,
        #e74c3c 90deg 180deg,
        #27ae60 180deg 270deg,
        #f39c12 270deg 360deg
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `pie-chart.html` and CSS as `pie-chart.css`.
2. Open in a browser.
3. Observe the four-colour pie chart.

**Expected Output:** A circular element divided into four equal quadrants: blue, red, green, and orange.

**Why This Works:** The `conic-gradient()` function paints each colour between its specified angles. `#3498db 0deg 90deg` fills the first quadrant (from 12 o'clock to 3 o'clock). Each subsequent colour fills the next quadrant. Because the stops are hard (90deg appears twice), the transitions are sharp.

---

#### Example 2: Colour Wheel

**HTML File (`colour-wheel.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Colour Wheel</title>
    <link rel="stylesheet" href="colour-wheel.css">
</head>
<body>
    <div class="wheel"></div>
</body>
</html>
```

**CSS File (`colour-wheel.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.wheel {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    /* Full colour wheel using conic gradient */
    background: conic-gradient(
        hsl(0, 100%, 50%),
        hsl(45, 100%, 50%),
        hsl(90, 100%, 50%),
        hsl(135, 100%, 50%),
        hsl(180, 100%, 50%),
        hsl(225, 100%, 50%),
        hsl(270, 100%, 50%),
        hsl(315, 100%, 50%),
        hsl(360, 100%, 50%)
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `colour-wheel.html` and CSS as `colour-wheel.css`.
2. Open in a browser.
3. Observe the smooth colour wheel transitioning through the full hue spectrum.

**Expected Output:** A circular element with a smooth colour wheel transition from red through yellow, green, cyan, blue, magenta, and back to red.

**Why This Works:** The `conic-gradient()` function interpolates between consecutive HSL colours, each 45 degrees apart. Because the interpolation is smooth (no hard stops), the result is a continuous colour wheel.

---

### Real-World Cases

- **Pie charts:** Conic gradients with hard stops for data visualisation.
- **Loading spinners:** A conic gradient with a transparent-to-colour transition, rotated with CSS animation.
- **Starburst backgrounds:** `repeating-conic-gradient()` with alternating transparent and semi-transparent stops.
- **Metallic rays:** `repeating-conic-gradient()` with subtle colour variations for a brushed metal effect.

---

## 4. Procedural Micro-Patterns: Layering Multiple Background Gradients

### Definitions

**Core Definition:** Procedural micro-patterns are complex, repeating patterns — such as checkerboards, plaid, dots, and mesh gradients — constructed by layering multiple CSS gradients on top of each other using `background-image`, with `background-size` and `background-repeat` controlling the tiling.

**Technical Definition:** The `background-image` property accepts a comma-separated list of `<image>` values. Each gradient in the list is a separate background layer. The first layer is painted on top, and subsequent layers are painted behind. By combining gradients with different angles, sizes, and colours — and using `background-size` to control the tiling scale and `background-repeat` to control repetition — authors can construct mathematically defined patterns. A checkerboard, for example, can be built from two 45-degree linear gradients offset by half a tile, or from four linear gradients each containing one quarter of the dark squares. Plaid is created by layering several overlapping gradients with transparency. Mesh gradients are created by stacking multiple radial gradients at different positions.

**Beginner-Friendly Explanation:** Think of CSS gradients as sheets of coloured glass. You can stack several sheets on top of each other, and where they overlap, you get new colours and patterns. A checkerboard is made by stacking two gradient sheets that each create diagonal stripes, offset so that they form squares. Plaid is made by stacking several striped sheets at different angles. Dots are made by repeating a radial gradient. By controlling the size and position of each sheet with `background-size` and `background-position`, you can create an infinite variety of patterns — all without a single image file.

---

### Purposes

- To create complex patterns without image assets.
- To reduce page weight by replacing pattern images with pure CSS.
- To create scalable patterns that look crisp at any size.
- To build decorative backgrounds for sections and cards.
- To construct textures for a more tactile visual design.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    background-image: 
        <gradient-1>,
        <gradient-2>,
        ...;
    background-size: <size-1>, <size-2>, ...;
    background-repeat: <repeat-1>, <repeat-2>, ...;
    background-position: <position-1>, <position-2>, ...;
}
```

#### Component Breakdown

| Property | Description | Example |
|---|---|---|
| `background-image` | Comma-separated list of gradient layers. | `linear-gradient(...), radial-gradient(...)` |
| `background-size` | The tile size for each layer. | `30px 30px, 60px 60px` |
| `background-repeat` | How each layer repeats. | `repeat, no-repeat` |
| `background-position` | The offset for each layer. | `0 0, 15px 15px` |

#### Common Pattern Formulas

| Pattern | Technique | Example |
|---|---|---|
| Checkerboard | Two 45° gradients with hard stops. | `linear-gradient(45deg, #000 25%, transparent 25%, transparent 75%, #000 75%)` |
| Stripes | `repeating-linear-gradient()` with hard stops. | `repeating-linear-gradient(90deg, red 0 20px, blue 20px 40px)` |
| Dots | `radial-gradient()` with small circles. | `radial-gradient(circle, #000 2px, transparent 2px)` |
| Plaid | Overlapping semi-transparent stripes at different angles. | Multiple `repeating-linear-gradient()` layers |
| Mesh | Multiple radial gradients at different positions. | `radial-gradient(at 20% 30%, red, transparent), radial-gradient(at 70% 60%, blue, transparent)` |

#### Syntax Rules

1. The first gradient in `background-image` is painted on top.
2. Each layer can have its own `background-size`, `background-repeat`, and `background-position`.
3. If `background-size` has fewer values than `background-image`, the list is repeated cyclically.
4. `background-repeat: repeat` (the default) tiles the gradient across the element.
5. `background-size` controls the tile dimensions for each layer.
6. `background-position` offsets each layer, which is useful for aligning patterns.
7. All gradient functions are Baseline widely available.

#### Constraints and Limitations

- **Performance** — many gradient layers can increase rasterisation time.
- **Complexity** — complex patterns are difficult to debug without visual testing.
- **Anti-aliasing** — fine patterns may appear slightly blurry on non-integer pixel boundaries.
- **Accessibility** — decorative patterns should not reduce text contrast.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: CSS Checkerboard

**HTML File (`checkerboard.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Checkerboard</title>
    <link rel="stylesheet" href="checkerboard.css">
</head>
<body>
    <div class="checkerboard"></div>
</body>
</html>
```

**CSS File (`checkerboard.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.checkerboard {
    width: 300px;
    height: 300px;
    border-radius: 12px;
    /* Two 45-degree gradients offset by half a tile */
    background-image:
        linear-gradient(45deg, #2c3e50 25%, transparent 25%, transparent 75%, #2c3e50 75%),
        linear-gradient(45deg, #2c3e50 25%, transparent 25%, transparent 75%, #2c3e50 75%);
    background-size: 60px 60px;
    background-position: 0 0, 30px 30px;
    background-color: #ecf0f1;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `checkerboard.html` and CSS as `checkerboard.css`.
2. Open in a browser.
3. Observe the crisp checkerboard pattern.

**Expected Output:** A 300×300 element with a checkerboard pattern of dark blue and light grey squares.

**Why This Works:** Two identical `linear-gradient(45deg, ...)` layers are used. Each gradient creates a 60×60 tile containing two dark squares and two transparent squares. The second layer is offset by 30px (half a tile) in both directions, which fills in the transparent squares of the first layer, creating the alternating checkerboard.

---

#### Example 2: Plaid Pattern

**HTML File (`plaid.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Plaid</title>
    <link rel="stylesheet" href="plaid.css">
</head>
<body>
    <div class="plaid"></div>
</body>
</html>
```

**CSS File (`plaid.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.plaid {
    width: 300px;
    height: 300px;
    border-radius: 12px;
    /* Overlapping semi-transparent stripes create plaid */
    background-image:
        repeating-linear-gradient(
            0deg,
            transparent 0px, transparent 20px,
            rgba(231, 76, 60, 0.4) 20px, rgba(231, 76, 60, 0.4) 40px
        ),
        repeating-linear-gradient(
            90deg,
            transparent 0px, transparent 20px,
            rgba(52, 152, 219, 0.4) 20px, rgba(52, 152, 219, 0.4) 40px
        ),
        repeating-linear-gradient(
            45deg,
            transparent 0px, transparent 30px,
            rgba(255, 255, 255, 0.3) 30px, rgba(255, 255, 255, 0.3) 60px
        );
    background-color: #2c3e50;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `plaid.html` and CSS as `plaid.css`.
2. Open in a browser.
3. Observe the plaid pattern with intersecting coloured stripes.

**Expected Output:** A 300×300 element with a plaid pattern created by overlapping semi-transparent red, blue, and white stripes on a dark background.

**Why This Works:** Three `repeating-linear-gradient()` layers are stacked. The first creates horizontal red stripes, the second creates vertical blue stripes, and the third creates diagonal white stripes. The semi-transparent colours (`rgba(..., 0.4)`) allow the layers to blend, creating the characteristic plaid intersections.

---

### Real-World Cases

- **Textile backgrounds:** Plaid, gingham, and denim textures for fashion and lifestyle sites.
- **Dashboard backgrounds:** Subtle dot grids and mesh gradients.
- **Hero sections:** Geometric patterns that add visual interest without images.
- **Card backgrounds:** Subtle micro-patterns for depth.

---

## 5. Colour Interpolation Hints: Controlling the Midpoint Shift and Acceleration Curve

### Definitions

**Core Definition:** A colour interpolation hint (also called a transition hint or midpoint hint) is an unlabelled percentage value placed between two colour stops in a gradient. It specifies where the "halfway colour" — the point at which the colour is an equal blend of the two surrounding stops — should occur.

**Technical Definition:** Between two colour stops, the line's colour is interpolated between the colours of the two colour stops, with the interpolation taking place in premultiplied RGBA space. By default, this interpolation is linear. However, if a colour hint is provided between two colour stops, the interpolation is non-linear and controlled by the hint. The hint specifies where the "halfway colour" occurs. The mathematical model is as follows: determine the hint's location as a percentage of the distance between the two colour stops (H, between 0 and 1). For any point between the stops (P, between 0 and 1), the colour weighting C = P^(log_H(0.5)). The colour at that point is a linear blend between the two stops, blending (1 − C) of the first stop and C of the second stop. If the hint is placed halfway between the two stops, the result is ordinary linear interpolation. If the hint is placed anywhere else, it dictates the position of the "halfway point" and produces smooth, even blends between the colour stops and the "halfway point." There can be at most one colour interpolation hint between any two given normal colour stops.

**Beginner-Friendly Explanation:** Normally, a gradient from red to blue is 50% red and 50% blue at the midpoint. A colour hint lets you move that midpoint. If you write `red, 70%, blue`, the gradient reaches the 50-50 blend at 70% of the way along, meaning the red stays dominant longer and then transitions more quickly to blue. This gives you precise control over the acceleration curve of the colour transition without adding extra colour stops. It is like controlling the ease-in/ease-out of a colour fade.

---

### Purposes

- To control where the midpoint of a colour transition occurs.
- To create non-linear colour transitions without adding manual stops.
- To fine-tune the pacing of a gradient to match a design.
- To make colour transitions feel more natural or dramatic.
- To reduce the number of colour stops required for a desired effect.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    background: linear-gradient(
        <direction>,
        <color-stop>,
        <hint>,
        <color-stop>
    );
}

/* Example */
background: linear-gradient(
    to right,
    red 0%,
    30%,        /* Hint: the 50-50 midpoint occurs at 30% */
    blue 100%
);
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<color-stop>` | A colour with an optional position. | `red 0%` |
| `<hint>` | An unlabelled percentage. | `30%` |
| `<color-stop>` | The second colour. | `blue 100%` |

#### Syntax Rules

1. A hint is an unlabelled percentage placed between two colour stops.
2. There can be at most one hint between any two colour stops.
3. If the hint is at 50%, the interpolation is linear (the default).
4. If the hint is below 50%, the first colour dominates longer.
5. If the hint is above 50%, the second colour dominates longer.
6. The colour weighting formula is `C = P^(log_H(0.5))`.
7. Hints work in all gradient functions (`linear-gradient()`, `radial-gradient()`, `conic-gradient()`).
8. Hints are Baseline widely available.

#### Constraints and Limitations

- **One hint per pair** — you cannot place two hints between the same two colour stops.
- **No negative or >100% hints** — hints must be valid percentages.
- **Subtlety** — the effect of a hint can be subtle; test with extreme values first.
- **Browser support** — hints are widely supported, but always test.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Gradient with and without a Hint

**HTML File (`hint.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gradient Hint</title>
    <link rel="stylesheet" href="hint.css">
</head>
<body>
    <h3>Without hint (linear midpoint at 50%)</h3>
    <div class="gradient no-hint"></div>

    <h3>With hint at 20% (red dominates longer)</h3>
    <div class="gradient with-hint"></div>
</body>
</html>
```

**CSS File (`hint.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

h3 {
    font-size: 0.9rem;
    color: #555;
    margin-bottom: 5px;
    margin-top: 20px;
}

.gradient {
    height: 80px;
    border-radius: 12px;
}

.no-hint {
    /* Standard linear interpolation: midpoint at 50% */
    background: linear-gradient(to right, #e74c3c 0%, #3498db 100%);
}

.with-hint {
    /* Hint at 20%: the 50-50 blend occurs at 20% along the gradient */
    background: linear-gradient(to right, #e74c3c 0%, 20%, #3498db 100%);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `hint.html` and CSS as `hint.css`.
2. Open in a browser.
3. Compare the two gradients: the one with the hint stays red longer and transitions to blue more quickly.

**Expected Output:** Two gradient bars. The first transitions evenly from red to blue with the midpoint at 50%. The second stays predominantly red until 20% of the way along, then transitions quickly to blue.

**Why This Works:** The `20%` hint between the two colour stops tells the browser that the 50-50 blend of red and blue should occur at 20% of the gradient's length. This means the gradient stays closer to red for the first 20% and then accelerates toward blue, creating a non-linear transition.

---

### Real-World Cases

- **Brand gradients:** Controlling the pacing to emphasise a brand colour.
- **UI backgrounds:** Subtle gradients that transition more naturally.
- **Hero sections:** Dramatic colour shifts with controlled acceleration.
- **Data visualisation:** Colour scales with perceptually tuned transitions.

---

## References

- MDN Web Docs — `<color-interpolation-method>` - https://developer.mozilla.org/en-US/docs/Web/CSS/color-interpolation-method
- MDN Web Docs — `conic-gradient()` - https://developer.mozilla.org/en-US/docs/Web/CSS/gradient/conic-gradient
- MDN Web Docs — `repeating-linear-gradient()` - https://developer.mozilla.org/en-US/docs/Web/CSS/gradient/repeating-linear-gradient
- MDN Web Docs — `repeating-radial-gradient()` - https://developer.mozilla.org/en-US/docs/Web/CSS/gradient/repeating-radial-gradient
- MDN Web Docs — Using CSS gradients - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_images/Using_CSS_gradients
- CSS-Tricks — What You Need to Know About CSS Color Interpolation - https://css-tricks.com/what-you-need-to-know-about-css-color-interpolation/
- CSS-Tricks — No-Jank CSS Stripes - https://css-tricks.com/no-jank-css-stripes/
- CSS-Tricks — CSS Tricks That Use Only One Gradient - https://css-tricks.com/css-tricks-that-use-only-one-gradient/
- W3C — CSS Image Values and Replaced Content Module Level 4 - https://www.w3.org/TR/css-images-4/
- W3C — CSS Color Module Level 5 - https://www.w3.org/TR/css-color-5/
- Can I Use — CSS `conic-gradient()` - https://caniuse.com/css-conic-gradients
- Can I Use — CSS `oklch()` - https://caniuse.com/mdn-css_types_color_oklch
- Can I Use — `color-interpolation-method` - https://caniuse.com/mdn-css_types_color_interpolation_method