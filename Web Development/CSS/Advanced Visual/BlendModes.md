# CSS Blend Modes — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Blend Modes are a CSS module that defines how an element's content (or background) blends with the content or background of the elements behind it. The two primary properties are `mix-blend-mode`, which blends an entire element with its backdrop, and `background-blend-mode`, which blends the multiple background layers of a single element together.

**Technical Definition:** The CSS Compositing and Blending Module Level 1 defines the `mix-blend-mode` and `background-blend-mode` properties, which allow authors to specify how colours should appear when elements overlap. The module provides 16 blending modes, which are divided into two categories: separable blend modes (which consider each colour component, such as RGB, individually) and non-separable blend modes (which treat all colour components as equivalent, operating in the HSL colour space). The `mix-blend-mode` property sets how an element's content should blend with the content of the element's parent and the element's background. The `background-blend-mode` property defines how an element's background images and background colour blend together. The `isolation` property creates a new stacking context, which prevents blend modes from bleeding into deeper parent layers.

**Beginner-Friendly Explanation:** When two layers of content overlap on a web page, the top layer normally just sits on top of the bottom layer and hides it. Blend modes change that. Instead of simply covering what is behind, the top layer's colours are mathematically combined with the colours of the layer behind it, creating a new, composite colour. Think of it like mixing paint or overlaying two photos in Photoshop. You can darken, lighten, add contrast, or create colour-based effects. The `mix-blend-mode` property blends a whole element with whatever is behind it, while `background-blend-mode` blends an element's own background layers together.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Two blending properties** | `mix-blend-mode` (element vs. backdrop) and `background-blend-mode` (background layers). |
| **16 blending modes** | Divided into separable (RGB-based) and non-separable (HSL-based) categories. |
| **Separable vs. non-separable** | Separable modes operate on individual RGB channels; non-separable modes operate on HSL components. |
| **Isolation control** | The `isolation: isolate` property creates a stacking context that contains blend effects. |
| **Stacking context creation** | A non-`normal` `mix-blend-mode` creates a stacking context. |
| **Baseline widely available** | Both `mix-blend-mode` and `background-blend-mode` have been supported in all major browsers since January 2020. |
| **Accessibility implications** | Blend modes can affect text contrast and should be tested against WCAG standards. |

---

### Prerequisites

Before studying CSS Blend Modes, you should understand:

- **CSS Stacking Contexts** — how elements are layered in the visual order.
- **CSS Backgrounds** — `background-image`, `background-color`, and multiple background layers.
- **CSS Colour Models** — RGB, HSL, and alpha channels.
- **CSS Opacity** — how transparency affects blending.
- **The `z-index` Property** — how layering is controlled.

---

### Related Programming Areas

- **CSS Filters** — colour manipulation that can be combined with blend modes.
- **CSS Opacity** — transparency that interacts with blending.
- **SVG Compositing** — SVG has its own blend mode features.
- **UI/UX Design** — duotone images, creative overlays, and visual effects.
- **Web Accessibility** — ensuring text remains readable over blended backgrounds.

---

### Core Concepts / Features

1. Element Intersecting: `mix-blend-mode`
2. Multi-Background Compositing: `background-blend-mode`
3. Blending Formulas: Darkening, Lightening, Contrast, and Colour Isolation
4. Isolation Contexts: `isolation: isolate`
5. Text Accessibility: Dynamic Contrast with `difference`

---

## 1. Element Intersecting: Merging Overlapping DOM Element Layers Together Using `mix-blend-mode`

### Definitions

**Core Definition:** The `mix-blend-mode` CSS property sets how an element's content should blend with the content of the element's parent and the element's background. It applies blending to the entire element, including its pseudo-elements.

**Technical Definition:** The `mix-blend-mode` property is defined in the CSS Compositing and Blending Module Level 1. It accepts a `<blend-mode>` value or the special values `plus-darker` and `plus-lighter`. When an element has a `mix-blend-mode` other than `normal`, it creates a new stacking context. The blending occurs between the element's rendered content (the source) and the content of its backdrop (the destination), which is everything painted behind it within the same stacking context. The blend mode is applied per-pixel, using the colours of the source and destination to compute a new colour value. The `plus-lighter` value uses the plus-lighter compositing operator and is useful for cross-fade effects, preventing unwanted blinking when two overlapping elements animate their opacity in opposite directions.

**Beginner-Friendly Explanation:** `mix-blend-mode` is like telling one element to "blend in" with whatever is behind it. If you have a coloured box on top of a photo, `mix-blend-mode: multiply` makes the box's colour multiply with the photo's colours, creating a darker, tinted effect. The blending applies to the whole element — its background, its text, its borders — and even its pseudo-elements. This is the property you use for creative effects like duotone images, colourful overlays, and text that interacts with its background.

---

### Purposes

- To blend an entire element with the content behind it.
- To create duotone and colour-tinting effects on images.
- To build creative overlays that interact with background content.
- To achieve cross-fade effects without blinking using `plus-lighter`.
- To apply blend modes to text, borders, and pseudo-elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    mix-blend-mode: <blend-mode> | plus-darker | plus-lighter;
}

/* <blend-mode> values */
mix-blend-mode: normal;
mix-blend-mode: multiply;
mix-blend-mode: screen;
mix-blend-mode: overlay;
mix-blend-mode: darken;
mix-blend-mode: lighten;
mix-blend-mode: color-dodge;
mix-blend-mode: color-burn;
mix-blend-mode: hard-light;
mix-blend-mode: soft-light;
mix-blend-mode: difference;
mix-blend-mode: exclusion;
mix-blend-mode: hue;
mix-blend-mode: saturation;
mix-blend-mode: color;
mix-blend-mode: luminosity;
```

#### Component Breakdown

| Value | Description | Category |
|---|---|---|
| `normal` | The final colour is the top colour (no blending). | Separable |
| `multiply` | Multiplies top and bottom colours; white has no effect, black leads to black. | Separable |
| `screen` | Inverse of multiply; black has no effect, white leads to white. | Separable |
| `overlay` | Multiply if bottom is darker, screen if bottom is lighter. | Separable |
| `darken` | Chooses the darkest value per colour channel. | Separable |
| `lighten` | Chooses the lightest value per colour channel. | Separable |
| `color-dodge` | Divides bottom by inverse of top; brightens. | Separable |
| `color-burn` | Inverts bottom, divides by top, inverts; darkens. | Separable |
| `hard-light` | Multiply if top is darker, screen if top is lighter. | Separable |
| `soft-light` | Softer version of hard-light. | Separable |
| `difference` | Subtracts the darker colour from the lighter. | Separable |
| `exclusion` | Like difference but with less contrast. | Separable |
| `hue` | Hue of top, saturation and luminosity of bottom. | Non-separable |
| `saturation` | Saturation of top, hue and luminosity of bottom. | Non-separable |
| `color` | Hue and saturation of top, luminosity of bottom. | Non-separable |
| `luminosity` | Luminosity of top, hue and saturation of bottom. | Non-separable |
| `plus-darker` | Plus-darker compositing operator. | Special |
| `plus-lighter` | Plus-lighter compositing operator; for cross-fades. | Special |

#### Syntax Rules

1. `mix-blend-mode` applies to all elements, including pseudo-elements.
2. A `mix-blend-mode` value other than `normal` creates a new stacking context.
3. The blend is computed per-pixel between the element (source) and its backdrop (destination).
4. Blending occurs within the current stacking context; `isolation: isolate` can contain it.
5. Changes between blend modes are not interpolated; the change occurs immediately.
6. `plus-lighter` is useful for cross-fade effects, preventing unwanted blinking when two overlapping elements animate their opacity in opposite directions.
7. `mix-blend-mode` is Baseline widely available since January 2020.

#### Constraints and Limitations

- **Stacking context side effect** — applying `mix-blend-mode` creates a new stacking context, which can affect `z-index` behaviour.
- **Performance** — blending requires extra compositing work; excessive use can degrade performance.
- **Accessibility** — blended text may have unpredictable contrast; test against WCAG standards.
- **No interpolation** — blend mode changes are not animated smoothly; they switch instantly.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Duotone Image Effect with `mix-blend-mode`

**HTML File (`mix-blend.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Duotone with mix-blend-mode</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="mix-blend.css">
</head>
<body>
    <!-- A container with a background image and a colour overlay -->
    <div class="duotone">
        <div class="overlay"></div>
    </div>
</body>
</html>
```

**CSS File (`mix-blend.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.duotone {
    width: 400px;
    height: 300px;
    background-image: url('https://picsum.photos/400/300');
    background-size: cover;
    background-position: center;
    border-radius: 16px;
    overflow: hidden;
    position: relative;
}

.overlay {
    position: absolute;
    inset: 0;
    /* A purple-to-pink gradient overlay */
    background: linear-gradient(135deg, #667eea, #764ba2);
    /* Blend the overlay with the image behind it */
    mix-blend-mode: multiply;
    opacity: 0.8;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `mix-blend.html`.
3. Save the CSS code as `mix-blend.css` in the same folder.
4. Open `mix-blend.html` in a browser.
5. Observe that the image is tinted with a purple-to-pink gradient, creating a duotone effect.

**Expected Output:** A photo with a purple-pink duotone effect. The gradient overlay's colours are multiplied with the photo's colours, darkening and tinting the image.

**Why This Works:** The `.overlay` element has `mix-blend-mode: multiply`, which multiplies its gradient colours with the background image's colours. White areas in the overlay have no effect (multiply by white = no change), while darker colours darken the image. The `opacity: 0.8` softens the effect.

---

#### Example 2: Cross-Fade with `plus-lighter`

**HTML File (`plus-lighter.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cross-Fade with plus-lighter</title>
    <link rel="stylesheet" href="plus-lighter.css">
</head>
<body>
    <div class="crossfade">
        <div class="layer layer-a">A</div>
        <div class="layer layer-b">B</div>
    </div>

    <script>
        const layerA = document.querySelector('.layer-a');
        const layerB = document.querySelector('.layer-b');
        let toggle = false;
        setInterval(() => {
            toggle = !toggle;
            layerA.style.opacity = toggle ? '0' : '1';
            layerB.style.opacity = toggle ? '1' : '0';
        }, 2000);
    </script>
</body>
</html>
```

**CSS File (`plus-lighter.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.crossfade {
    position: relative;
    width: 300px;
    height: 200px;
    border-radius: 12px;
    overflow: hidden;
}

.layer {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 3rem;
    font-weight: bold;
    color: white;
    /* Use plus-lighter to prevent blinking during cross-fade */
    mix-blend-mode: plus-lighter;
    transition: opacity 0.8s ease;
}

.layer-a {
    background: linear-gradient(135deg, #3498db, #2980b9);
    opacity: 1;
}

.layer-b {
    background: linear-gradient(135deg, #e74c3c, #c0392b);
    opacity: 0;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `plus-lighter.html` and CSS as `plus-lighter.css`.
2. Open in a browser.
3. Observe the two layers cross-fading smoothly every 2 seconds.

**Expected Output:** Two coloured layers that cross-fade smoothly without the "blink" that would normally occur when one fades out and the other fades in simultaneously.

**Why This Works:** The `plus-lighter` blend mode adds the colours of the two layers together, so during the cross-fade, the sum of the two layers' opacities remains constant, preventing the background from showing through and causing a blink.

---

### Real-World Cases

- **Duotone images:** `mix-blend-mode: multiply` or `screen` for stylised photo treatments.
- **Cross-fades:** `plus-lighter` for smooth transitions between two layers.
- **Creative overlays:** Colour overlays on images for hero sections.
- **Text effects:** Blend text with a background image for artistic typography.

---

## 2. Multi-Background Compositing: Layering and Blending Multiple Images Within a Single Element Using `background-blend-mode`

### Definitions

**Core Definition:** The `background-blend-mode` CSS property defines how an element's background images and background colour blend together. It applies blending to an element's own background layers, not to the element as a whole.

**Technical Definition:** The `background-blend-mode` property is defined in the CSS Compositing and Blending Module Level 1. It accepts a comma-separated list of `<blend-mode>` values, one for each background layer. The blend modes are defined in the same order as the `background-image` property. If the number of blend modes does not match the number of background images, the list is truncated or repeated to match. The blending occurs between the background layers themselves, not between the element and its backdrop. This means `background-blend-mode` is isolated from the rest of the page by default.

**Beginner-Friendly Explanation:** If an element has multiple background images stacked on top of each other, `background-blend-mode` lets you blend them together. For example, you could have a texture image and a colour gradient as two background layers, and blend them so the texture shows through the gradient. Unlike `mix-blend-mode`, which blends the element with the page behind it, `background-blend-mode` only affects the layers inside the element's own background.

---

### Purposes

- To blend multiple background images together within a single element.
- To apply a colour overlay to a background image using `background-color`.
- To create textured gradients by blending a texture image with a colour.
- To layer decorative patterns and shapes with blend modes.
- To achieve complex background effects without extra elements.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    background-image: url(image1.png), url(image2.png), linear-gradient(...);
    background-blend-mode: <blend-mode>, <blend-mode>, <blend-mode>;
}
```

#### Component Breakdown

| Component | Description |
|---|---|
| `background-image` | The list of background layers. |
| `background-color` | The bottom-most background layer. |
| `background-blend-mode` | Comma-separated list of blend modes, one per layer. |

#### Syntax Rules

1. `background-blend-mode` accepts a comma-separated list of `<blend-mode>` values.
2. The blend modes are matched to the background layers in the same order as `background-image`.
3. If there are more blend modes than layers, the extra values are ignored.
4. If there are fewer blend modes than layers, the list is repeated cyclically.
5. `background-blend-mode` is isolated from the rest of the page — it does not blend with elements behind the element.
6. The `background-color` participates in blending as the bottom-most layer.
7. `background-blend-mode` is Baseline widely available since January 2020.

#### Constraints and Limitations

- **Isolation by default** — `background-blend-mode` does not blend with the page backdrop; it only blends the element's own background layers.
- **Layer order matters** — the first background image is the topmost layer.
- **No `isolation` needed** — unlike `mix-blend-mode`, `background-blend-mode` is inherently isolated.
- **Performance** — blending multiple large background images can be rasterization-intensive.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Textured Gradient with `background-blend-mode`

**HTML File (`bg-blend.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Textured Gradient with background-blend-mode</title>
    <link rel="stylesheet" href="bg-blend.css">
</head>
<body>
    <div class="textured-gradient"></div>
</body>
</html>
```

**CSS File (`bg-blend.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.textured-gradient {
    width: 400px;
    height: 300px;
    border-radius: 16px;

    /* Two background layers: a gradient and a texture */
    background-image:
        linear-gradient(135deg, #667eea, #764ba2),
        url('https://www.transparenttextures.com/patterns/carbon-fibre.png');

    /* Blend the gradient with the texture */
    background-blend-mode: overlay;

    background-size: cover, 200px 200px;
    background-position: center, center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `bg-blend.html` and CSS as `bg-blend.css`.
2. Open in a browser.
3. Observe that the gradient has a subtle texture blended into it, creating a more tactile surface.

**Expected Output:** A purple gradient with a carbon-fibre texture blended into it using the `overlay` blend mode. The result is a textured, dimensional background.

**Why This Works:** The `background-image` property defines two layers: a linear gradient (top) and a texture image (bottom). The `background-blend-mode: overlay` blends them together. The gradient's colours are overlaid on the texture, creating a textured gradient effect.

---

### Real-World Cases

- **Textured backgrounds:** Blending a noise or texture image with a gradient for subtle depth.
- **Colour overlays:** Blending a solid colour with a photo background using `background-blend-mode: multiply`.
- **Duotone backgrounds:** Combining a gradient with a grayscale image.
- **Pattern overlays:** Blending a geometric pattern with a solid colour.

---

## 3. Blending Formulas: Categorising and Mastering Standard Math Profiles

### Definitions

**Core Definition:** Blend modes are mathematical formulas that take the colours of two layers (the source and the destination) and compute a new colour. They are categorised into separable modes (operating on individual RGB channels) and non-separable modes (operating on HSL components). The modes can be grouped by their visual effect: darkening, lightening, contrast, and colour isolation.

**Technical Definition:** The CSS Compositing and Blending Module defines 16 blend modes. Separable blend modes consider each colour component (RGB) individually. The separable modes are: `normal`, `multiply`, `screen`, `overlay`, `darken`, `lighten`, `color-dodge`, `color-burn`, `hard-light`, `soft-light`, `difference`, and `exclusion`. Non-separable blend modes treat all colour components as equivalent and operate in the HSL colour space. The non-separable modes are: `hue`, `saturation`, `color`, and `luminosity`. Each mode performs a specific mathematical operation on the colour values of the source and destination layers.

**Beginner-Friendly Explanation:** Blend modes are like different mixing rules for colours. `multiply` multiplies the colours together, which always makes the result darker (like stacking transparent films). `screen` does the opposite and makes things lighter (like projecting two slides onto the same screen). `overlay` combines both — it makes dark areas darker and light areas lighter. The "non-separable" modes like `hue` and `color` work differently: they take one aspect of the top layer (like its hue) and combine it with other aspects of the bottom layer (like its brightness), creating colour-based effects rather than brightness-based ones.

---

### Purposes

- To understand the visual effect of each blend mode before applying it.
- To choose the right mode for a specific design goal (darkening, lightening, contrast, colour).
- To combine modes creatively for complex effects.
- To predict how a blend mode will interact with different background colours.
- To categorise modes for easier selection in design tools and CSS.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    mix-blend-mode: <blend-mode>;
    background-blend-mode: <blend-mode>;
}
```

#### Component Breakdown

##### Darkening Modes

| Mode | Formula | Effect |
|---|---|---|
| `multiply` | Source × Destination | White has no effect; black leads to black. Darkens. |
| `darken` | min(Source, Destination) per channel | Chooses the darkest value per channel. |
| `color-burn` | Invert, divide, invert | Increases contrast; midtones more saturated. |

##### Lightening Modes

| Mode | Formula | Effect |
|---|---|---|
| `screen` | Invert, multiply, invert | Black has no effect; white leads to white. Lightens. |
| `lighten` | max(Source, Destination) per channel | Chooses the lightest value per channel. |
| `color-dodge` | Divide destination by inverse of source | Brightens; pure black has no effect. |

##### Contrast Modes

| Mode | Formula | Effect |
|---|---|---|
| `overlay` | Multiply if bottom darker, screen if bottom lighter | Increases contrast; 50% grey is neutral. |
| `hard-light` | Multiply if top darker, screen if top lighter | Similar to overlay but layers swapped. Harsh. |
| `soft-light` | Softer version of hard-light | Gentle contrast; like a diffused spotlight. |

##### Colour Isolation Modes (Non-Separable)

| Mode | Formula | Effect |
|---|---|---|
| `hue` | Hue of top + saturation/luminosity of bottom | Applies the top's hue to the bottom. |
| `saturation` | Saturation of top + hue/luminosity of bottom | Applies the top's saturation to the bottom. |
| `color` | Hue + saturation of top + luminosity of bottom | Colourises the bottom with the top's colour. |
| `luminosity` | Luminosity of top + hue/saturation of bottom | Applies the top's brightness to the bottom. |

##### Comparison Modes

| Mode | Effect |
|---|---|
| `difference` | Subtracts darker from lighter; black has no effect, white inverts. |
| `exclusion` | Like difference but with less contrast; same colour = 50% grey. |

#### Syntax Rules

1. Separable modes operate on individual RGB channels.
2. Non-separable modes operate on HSL components and cannot be broken down into RGB operations.
3. `multiply` and `screen` are opposites: multiply darkens, screen lightens.
4. `overlay` and `hard-light` are opposites (layers swapped).
5. `difference` and `exclusion` are similar, with exclusion having lower contrast.
6. `hue`, `saturation`, `color`, and `luminosity` are inverses of each other in various combinations.
7. Blend mode changes are not interpolated; they switch instantly.

#### Constraints and Limitations

- **No interpolation** — you cannot animate between blend modes smoothly.
- **Predictability** — some modes (especially non-separable ones) can produce surprising results with certain colour combinations.
- **Browser differences** — minor rendering differences may occur between browsers.
- **Accessibility** — blend modes can reduce text contrast; always test.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Darkening, Lightening, and Contrast Modes

**HTML File (`formulas.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blend Mode Formulas</title>
    <link rel="stylesheet" href="formulas.css">
</head>
<body>
    <div class="gallery">
        <div class="item">
            <div class="bg"></div>
            <div class="overlay multiply">multiply</div>
        </div>
        <div class="item">
            <div class="bg"></div>
            <div class="overlay screen">screen</div>
        </div>
        <div class="item">
            <div class="bg"></div>
            <div class="overlay overlay">overlay</div>
        </div>
        <div class="item">
            <div class="bg"></div>
            <div class="overlay difference">difference</div>
        </div>
    </div>
</body>
</html>
```

**CSS File (`formulas.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.gallery {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    justify-content: center;
}

.item {
    position: relative;
    width: 200px;
    height: 200px;
    border-radius: 12px;
    overflow: hidden;
}

.bg {
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, #3498db 50%, #e74c3c 50%);
}

.overlay {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    font-size: 1.1rem;
    /* A semi-transparent purple overlay */
    background: rgba(155, 89, 182, 0.8);
}

.multiply { mix-blend-mode: multiply; }
.screen { mix-blend-mode: screen; }
.overlay { mix-blend-mode: overlay; }
.difference { mix-blend-mode: difference; }
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `formulas.html` and CSS as `formulas.css`.
2. Open in a browser.
3. Observe the four different blend effects on the same background.

**Expected Output:** Four panels with the same background (a blue/red gradient) and the same purple overlay, but each using a different blend mode: `multiply` darkens, `screen` lightens, `overlay` increases contrast, and `difference` creates an inverted effect.

**Why This Works:** Each blend mode applies a different mathematical formula to the source (purple overlay) and destination (blue/red gradient) colours. `multiply` darkens by multiplying the colours. `screen` lightens by inverting, multiplying, and inverting. `overlay` applies multiply or screen based on the destination's brightness. `difference` subtracts the darker colour from the lighter.

---

### Real-World Cases

- **Duotone images:** `multiply` for dark duotones, `screen` for light duotones.
- **Text overlays:** `difference` or `exclusion` for auto-inverting text.
- **Creative backgrounds:** `overlay` for textured gradients.
- **Colour grading:** `hue`, `saturation`, `color`, or `luminosity` for artistic colour effects.

---

## 4. Isolation Contexts: Preventing Blend Modes from Bleeding into Deep Parent Layers

### Definitions

**Core Definition:** The `isolation` CSS property determines whether an element must create a new stacking context. When set to `isolate`, it creates a new stacking context, which prevents its children's blend modes from blending with elements behind the isolated element.

**Technical Definition:** The `isolation` CSS property is defined in the CSS Compositing and Blending Module Level 1. It accepts two values: `auto` (the default) and `isolate`. Setting `isolation: isolate` turns the element into a stacking context. This is important for blend modes because `mix-blend-mode` blends an element with its backdrop — everything painted behind it within the same stacking context. If you have a component whose children use `mix-blend-mode`, those children might blend with elements outside the component, creating unexpected results. By setting `isolation: isolate` on the component's root, you create a new stacking context, so the children's blending is confined to the component. This does not apply to `background-blend-mode`, which is already isolated because it operates on the element's own background layers.

**Beginner-Friendly Explanation:** Imagine you are painting on a piece of glass. If the glass is transparent, your paint blends with whatever is behind it. If you place a white sheet of paper behind the glass, your paint only blends with the paper — it is isolated. The `isolation: isolate` property is like putting that white sheet of paper behind your component. It stops your component's blend modes from interacting with the rest of the page. This is essential for reusable components that should look consistent regardless of where they are placed.

---

### Purposes

- To contain blend modes within a component or section.
- To prevent child blend modes from affecting the parent or page background.
- To create predictable, reusable components with blend effects.
- To isolate a component's blending from the surrounding content.
- To ensure that a component's visual appearance is consistent regardless of context.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    isolation: auto | isolate;
}
```

#### Component Breakdown

| Value | Description |
|---|---|
| `auto` | Default. The element may or may not create a stacking context depending on other properties. |
| `isolate` | The element creates a new stacking context. |

#### Syntax Rules

1. `isolation: isolate` always creates a new stacking context.
2. A new stacking context contains its children's `mix-blend-mode` effects.
3. `isolation: isolate` does not affect `background-blend-mode`, which is already isolated.
4. Any property that creates a stacking context (e.g., `opacity < 1`, `transform`, `filter`) also isolates blend modes.
5. `isolation: isolate` is the explicit, semantic way to create an isolation context.
6. The property is not inherited.
7. `isolation` is Baseline widely available.

#### Constraints and Limitations

- **Stacking context side effects** — creating a stacking context can affect `z-index` behaviour and containing blocks for fixed-position elements.
- **Overuse** — adding `isolation: isolate` unnecessarily creates extra stacking contexts, which can complicate layout.
- **Not for `background-blend-mode`** — the property has no effect on `background-blend-mode`, which is inherently isolated.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Isolating a Component's Blend Effects

**HTML File (`isolation.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Isolation Context</title>
    <link rel="stylesheet" href="isolation.css">
</head>
<body>
    <div class="page">
        <div class="component isolated">
            <div class="blend-layer"></div>
            <p>Isolated Component</p>
        </div>
        <div class="component not-isolated">
            <div class="blend-layer"></div>
            <p>Non-Isolated Component</p>
        </div>
    </div>
</body>
</html>
```

**CSS File (`isolation.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background: linear-gradient(135deg, #667eea, #764ba2);
}

.page {
    display: flex;
    gap: 30px;
    justify-content: center;
    flex-wrap: wrap;
}

.component {
    position: relative;
    width: 250px;
    height: 200px;
    border-radius: 16px;
    overflow: hidden;
    background: #2c3e50;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
}

.isolated {
    /* Create a new stacking context to contain the blend */
    isolation: isolate;
}

.not-isolated {
    /* No isolation — blend will bleed to the page background */
}

.blend-layer {
    position: absolute;
    inset: 0;
    background: #e74c3c;
    /* Blend with whatever is behind */
    mix-blend-mode: multiply;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `isolation.html` and CSS as `isolation.css`.
2. Open in a browser.
3. Observe the difference: the `isolated` component's blend is contained within the component, while the `not-isolated` component's blend interacts with the page background.

**Expected Output:** The isolated component's red blend layer multiplies only with the component's dark background, producing a consistent result. The non-isolated component's red layer multiplies with both the component background and the page gradient, producing an unpredictable, context-dependent result.

**Why This Works:** The `.isolated` class applies `isolation: isolate`, which creates a new stacking context. The blend layer's `mix-blend-mode: multiply` now blends only with the component's own background. Without isolation, the blend layer blends with everything behind it in the same stacking context, including the page gradient.

---

### Real-World Cases

- **Reusable components:** Isolating blend effects so components look consistent in any context.
- **Page sections:** Isolating a section's blend effects from the rest of the page.
- **Card components:** Ensuring card overlays blend only with the card, not the page.
- **Layout stability:** Preventing blend modes from leaking between sections.

---

## 5. Text Accessibility: Utilising Blend Profiles to Create Dynamic, High-Contrast Text

### Definitions

**Core Definition:** Text accessibility with blend modes involves using blend modes — particularly `difference` and `exclusion` — to create text that automatically adapts its colour based on the background behind it, aiming to maintain readability across variable backgrounds. However, blend modes do not guarantee WCAG-compliant contrast and must be tested carefully.

**Technical Definition:** The `difference` blend mode subtracts the darker of the two colours from the lighter, producing a result that is the complement of the background. For white text, `mix-blend-mode: difference` creates a text colour that is the complement of whatever is behind it. This means white text on a white background becomes black, and white text on a black background remains white. The `exclusion` mode is similar but with lower contrast, producing a 50% grey result when colours are identical rather than black. While these modes create a visually dynamic effect, they do not guarantee accessible contrast — the WCAG contrast ratio requirement of 4.5:1 for normal text is not automatically met, and the result depends entirely on the background colours. It is essential to test readability and provide a fallback for important text.

**Beginner-Friendly Explanation:** Imagine text that always stays readable no matter what is behind it. `mix-blend-mode: difference` does something like that — it makes the text the opposite of the background colour. On a white background, white text becomes black. On a black background, it stays white. This is great for text over images or gradients that vary in brightness. However, it is not a magic bullet for accessibility. On some backgrounds, the result might be a grey colour that does not have enough contrast. You still need to test it and have a fallback for critical text.

---

### Purposes

- To create text that automatically inverts based on the background.
- To improve readability over images and gradients.
- To add a dynamic, modern visual effect to headings and navigation.
- To reduce the need for manual colour adjustments over variable backgrounds.
- To provide a visually engaging alternative to solid text colours.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    color: white;
    mix-blend-mode: difference; /* or exclusion */
}
```

#### Component Breakdown

| Mode | Effect on White Text |
|---|---|
| `difference` | Inverts to the complement of the background (high contrast). |
| `exclusion` | Similar to difference but with lower contrast (50% grey on same colour). |

#### Syntax Rules

1. `mix-blend-mode: difference` on white text produces a text colour that is the complement of the background.
2. The effect is per-pixel, so text over a split background will have different colours on different parts.
3. `exclusion` is a softer alternative with lower contrast.
4. Blend modes do not guarantee WCAG-compliant contrast; test with contrast tools.
5. Provide a fallback for critical text (e.g., a solid colour or `text-shadow`).
6. Subpixel antialiasing is disabled on elements with `mix-blend-mode`, which can affect text rendering.
7. The effect works best with light text on dark backgrounds and vice versa.

#### Constraints and Limitations

- **No WCAG guarantee** — contrast ratios are not guaranteed and depend on the background colours.
- **Subpixel antialiasing disabled** — text may appear slightly thinner or less crisp on some displays.
- **Unpredictable on complex backgrounds** — results can vary widely over photos and gradients.
- **Fallback required** — always provide a readable fallback for important text.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Auto-Inverting Text with `difference`

**HTML File (`text-difference.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Auto-Inverting Text</title>
    <link rel="stylesheet" href="text-difference.css">
</head>
<body>
    <div class="hero">
        <h1>Auto-Inverting Text</h1>
        <p>This text adapts to the background behind it.</p>
    </div>
</body>
</html>
```

**CSS File (`text-difference.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 0;
}

.hero {
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    /* A background with both light and dark areas */
    background: linear-gradient(90deg, #ffffff 0%, #ffffff 50%, #000000 50%, #000000 100%);
}

.hero h1 {
    font-size: 4rem;
    font-weight: bold;
    color: white;
    
    /* Invert against the background */
    mix-blend-mode: difference;
    margin: 0 0 16px;
    text-align: center;
}

.hero p {
    font-size: 1.5rem;
    color: white;
    mix-blend-mode: difference;
    text-align: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `text-difference.html` and CSS as `text-difference.css`.
2. Open in a browser.
3. Observe that the text on the white half of the background is black, and the text on the black half is white.

**Expected Output:** A split-screen hero with white on the left and black on the right. The text over the white area appears black, and the text over the black area appears white — the text colour automatically inverts based on the background.

**Why This Works:** The `mix-blend-mode: difference` on white text subtracts the background colour from white. On a white background, the result is black (255 − 255 = 0). On a black background, the result is white (255 − 0 = 255). This creates an automatic inversion effect that maintains readability across the split background.

---

### Real-World Cases

- **Hero sections:** Auto-inverting headlines over split or variable backgrounds.
- **Navigation bars:** Text over images or gradients that vary in brightness.
- **Image captions:** Captions that remain readable over photos.
- **⚠️ Accessibility caution:** Always test contrast ratios and provide a fallback for critical text. The `difference` mode does not guarantee WCAG compliance.

---

## References

- MDN Web Docs — `mix-blend-mode` - https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode
- MDN Web Docs — `background-blend-mode` - https://developer.mozilla.org/en-US/docs/Web/CSS/background-blend-mode
- MDN Web Docs — `<blend-mode>` - https://developer.mozilla.org/en-US/docs/Web/CSS/blend-mode
- MDN Web Docs — `isolation` - https://developer.mozilla.org/en-US/docs/Web/CSS/isolation
- MDN Web Docs — CSS compositing and blending - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_compositing_and_blending
- web.dev — Blend Modes - https://web.dev/learn/css/blend-modes
- CSS-Tricks — `mix-blend-mode` - https://css-tricks.com/almanac/properties/m/mix-blend-mode/
- W3C — CSS Compositing and Blending Module Level 1 - https://drafts.fxtf.org/compositing-1/
- Can I Use — `mix-blend-mode` - https://caniuse.com/css-mix-blend-mode
- Can I Use — `background-blend-mode` - https://caniuse.com/css-background-blend-mode
- Stack Overflow — Dynamically change colours with CSS so background and text colour aren't clashing - https://stackoverflow.com/questions/78946522/dynamically-change-colours-with-css-so-background-and-text-colour-arent-clashing