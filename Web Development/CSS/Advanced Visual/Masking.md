# CSS Masking — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Masking is a technique that allows authors to partially or fully hide portions of an element by using an image or graphical element as a mask, where the alpha channel or luminance of the mask determines which parts of the element remain visible.

**Technical Definition:** The CSS Masking Module Level 1 defines two means for partially or fully hiding portions of visual elements: clipping and masking. Masking describes how to use another graphical element or image as a luminance or alpha mask to selectively show or hide parts of an element. Unlike clipping, which creates a hard boundary (pixels are either visible or not), masking supports partial transparency, enabling soft fades, gradient reveals, and complex compositing effects. The `mask` shorthand property resets all `mask-*` properties (including `mask-border`) to their initial values, making it the recommended way to override any mask settings earlier in the cascade.

**Beginner-Friendly Explanation:** Think of masking as a stencil placed over an element. The stencil (the mask) determines what you can see through. Where the mask is opaque, the element shows fully. Where the mask is transparent, the element disappears. Where the mask is semi-transparent, the element is partially visible. Unlike `clip-path`, which cuts a hard shape, masking lets you create soft, gradual transitions — like a photo fading to nothing at the bottom edge, or a shimmer effect sweeping across text. You can use gradients, PNG images with alpha channels, or SVG mask elements as your stencil.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Alpha or luminance modes** | Masks can use the alpha channel or the luminance of the mask image. |
| **Soft transitions** | Unlike clipping, masking supports partial transparency and gradient falloffs. |
| **Multiple mask layers** | Multiple mask images can be composited using `add`, `subtract`, `intersect`, or `exclude`. |
| **Flexible sources** | Masks can be CSS gradients, raster images (PNG), or SVG `<mask>` elements. |
| **Reusable and animatable** | Masks can be transitioned and animated for fluid effects. |
| **GPU-accelerated** | Properly implemented masks can run on the compositor thread. |
| **Baseline available** | `mask` and its longhands are Baseline Newly Available since December 2023. |

---

### Prerequisites

Before studying CSS Masking, you should understand:

- **CSS Box Model** — content, padding, border, and margin boxes.
- **CSS Gradients** — `linear-gradient()`, `radial-gradient()`, `conic-gradient()`.
- **CSS Clipping** — `clip-path` for hard-edged shape masking.
- **SVG Basics** — `<mask>` elements and SVG coordinate systems.
- **CSS Transitions and Animations** — for morphing and animating masks.
- **Alpha Channels and Luminance** — the difference between transparency and brightness.

---

### Related Programming Areas

- **CSS Clipping** — the hard-edged counterpart to masking.
- **SVG** — `<mask>` elements provide complex, reusable mask definitions.
- **CSS Gradients** — the most practical source for soft mask falloffs.
- **Web Performance** — mask rasterization and GPU acceleration.
- **UI Design** — image fades, shimmer effects, and creative reveals.

---

### Core Concepts / Features

1. The `mask` Shorthand: Combining `mask-image`, `mask-mode`, `mask-repeat`, `mask-position`, and `mask-size`
2. Gradient Masking: Soft Transparency Falloffs
3. Vector and Asset Masks: SVG `<mask>` and PNG Alpha Channels
4. Mask Compositing: `mask-composite` Operations
5. Performance Impact: Rasterization Costs and Hardware Acceleration

---

## 1. The `mask` Shorthand: Precise Composite Boundaries

### Definitions

**Core Definition:** The `mask` shorthand property sets all `mask-*` properties in a single declaration, including `mask-image`, `mask-mode`, `mask-repeat`, `mask-position`, `mask-size`, and `mask-composite`. It also resets `mask-border` to its initial value.

**Technical Definition:** The `mask` CSS shorthand property hides an element (partially or fully) by masking or clipping the image at specific points. The `mask` shorthand property accepts a comma-separated list of mask layers. Each layer's syntax can include the following values: `<image> <position> / <size> <repeat> <origin> <clip> <composite> <mode>`. All components within a mask layer are optional. However, if the `mask-image` value is omitted, it defaults to a transparent black image, which completely hides the element in that layer. The `mask-position` and `mask-size` values must be separated by a forward slash (`/`), with the size component going after the position. The `mask` shorthand also resets all `mask-border-*` properties to their initial values.

**Beginner-Friendly Explanation:** The `mask` shorthand is a way to write all your masking settings in one line. You tell the browser what image to use as a mask, how to position it, how big it should be, whether it repeats, and how to blend it with other masks. For example, `mask: url(mask.png) 0 0/50px 50px repeat-x` means "use mask.png, position it at the top-left, make it 50×50 pixels, and repeat it horizontally." If you use the shorthand, it also resets any `mask-border` settings you might have, which helps prevent stale styles from leaking through.

---

### Purposes

- To set all mask-related properties in a single, concise declaration.
- To control the mask image source, position, size, repeat, and compositing.
- To reset `mask-border` to its initial value when overriding earlier mask settings.
- To define multiple mask layers in a comma-separated list.
- To provide a consistent syntax that mirrors `background` shorthand patterns.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    mask: <mask-layer>#;
}

<mask-layer> = <mask-reference> || <position> [ / <bg-size> ]? || <repeat-style> || <geometry-box> || [ <geometry-box> | no-clip ] || <compositing-operator> || <masking-mode>
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `<mask-reference>` | The mask image source. | `url(mask.png)`, `linear-gradient(...)` |
| `<position>` | The position of the mask image. | `center`, `0 0`, `40px 20px` |
| `/ <bg-size>` | The size of the mask image. | `/ 50px 50px` |
| `<repeat-style>` | How the mask image repeats. | `repeat-x`, `no-repeat`, `space` |
| `<geometry-box>` | The reference box for the mask. | `border-box`, `padding-box` |
| `<compositing-operator>` | How layers combine. | `add`, `subtract`, `intersect`, `exclude` |
| `<masking-mode>` | Alpha or luminance mode. | `alpha`, `luminance`, `match-source` |

#### Syntax Rules

1. The `mask` shorthand accepts a comma-separated list of mask layers.
2. Within each layer, the `mask-position` and `mask-size` values must be separated by a forward slash (`/`).
3. The `mask-size` value must appear after the `mask-position` value.
4. If `mask-image` is omitted, it defaults to a transparent black image (element fully hidden).
5. The `mask` shorthand resets all `mask-border-*` properties to their initial values.
6. Unspecified components default to their initial values: `mask-mode: match-source`, `mask-position: 0% 0%`, `mask-size: auto`, `mask-repeat: repeat`, `mask-origin: border-box`, `mask-clip: border-box`, `mask-composite: add`.
7. The `mask` property is Baseline Newly Available since December 2023.

#### Constraints and Limitations

- **Shorthand reset behaviour** — the `mask` shorthand resets `mask-border`, which may be surprising if you only wanted to change one mask property.
- **Position/size ambiguity** — the forward slash separator is required to distinguish position from size.
- **Browser support** — `mask` is Baseline Newly Available; older browsers may require `-webkit-mask`.
- **Layer count** — the number of layers is determined by the comma-separated `mask-image` values.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Mask with Gradient and Position

**HTML File (`mask-basic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basic CSS Mask</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="mask-basic.css">
</head>
<body>
    <div class="masked-box">Fading Content</div>
</body>
</html>
```

**CSS File (`mask-basic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.masked-box {
    width: 300px;
    height: 200px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 1.5rem;
    font-weight: bold;
    background: linear-gradient(135deg, #667eea, #764ba2);
    border-radius: 12px;

    /* Mask: fade from fully visible at the top to transparent at the bottom */
    mask: linear-gradient(
        to bottom,
        black 0%,
        black 60%,
        transparent 100%
    );
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `mask-basic.html`.
3. Save the CSS code as `mask-basic.css` in the same folder.
4. Open `mask-basic.html` in a modern browser.
5. Observe the gradient box fades to transparent at the bottom.

**Expected Output:** A gradient-filled box with white text. The box is fully visible at the top and gradually fades to transparent at the bottom, revealing the background behind it.

**Why This Works:** The `mask: linear-gradient(to bottom, black 0%, black 60%, transparent 100%)` uses a linear gradient as the mask. Black areas of the mask represent fully opaque (visible), and transparent areas represent fully hidden. The gradient smoothly transitions from black to transparent between 60% and 100%, creating the fade effect.

---

### Real-World Cases

- **Image fades:** Fading the bottom of a hero image into the page background.
- **Shimmer effects:** A moving gradient mask that reveals a shimmer on buttons or text.
- **Creative reveals:** Using multiple mask layers to create complex reveal animations.
- **Text clipping:** Using a mask to reveal an image only through the shape of text.

---

## 2. Gradient Masking: Soft Transparency Falloffs

### Definitions

**Core Definition:** Gradient masking uses CSS gradients (`linear-gradient()`, `radial-gradient()`, `conic-gradient()`) as mask images to create smooth, gradual transitions between fully visible and fully hidden areas of an element.

**Technical Definition:** CSS gradients are functions that create a progressive transition between two or more colours. When used as a mask image via `mask-image` or the `mask` shorthand, the browser interprets the gradient's alpha channel (by default) or luminance (when `mask-mode: luminance` is set) to determine the mask values. A `linear-gradient` produces a straight-line fade, a `radial-gradient` produces a circular or elliptical fade, and a `conic-gradient` produces an angular sweep. The gradient stops define where the mask transitions from opaque to transparent. Colours with full opacity (e.g., `black`, `white`, `rgba(0,0,0,1)`) represent fully visible areas, while `transparent` or `rgba(0,0,0,0)` represents fully hidden areas. Intermediate values create partial transparency.

**Beginner-Friendly Explanation:** A gradient mask is like a soft-edged stencil. Instead of a hard cut, the mask gradually fades from solid to clear. `linear-gradient` is a straight fade — imagine a photo fading out from the bottom. `radial-gradient` is a circular fade — imagine a vignette effect around the edges. `conic-gradient` is an angular fade — imagine a pie chart reveal sweeping around. Because the fade is gradual, the masked element appears to dissolve smoothly rather than being cut off.

---

### Purposes

- To create smooth fade-out effects on images and content.
- To build vignette effects using radial gradients.
- To create angular reveals and sweeping animations with conic gradients.
- To achieve soft edges that `clip-path` cannot produce.
- To build shimmer and glow effects with moving gradient masks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    mask-image: linear-gradient(<direction>, <color-stop-list>);
    mask-image: radial-gradient(<shape> <size> at <position>, <color-stop-list>);
    mask-image: conic-gradient(from <angle> at <position>, <color-stop-list>);
}
```

#### Component Breakdown

| Gradient Function | Description | Best For |
|---|---|---|
| `linear-gradient()` | Fades along a straight line. | Edge fades, slide reveals. |
| `radial-gradient()` | Fades from a centre point outward. | Vignettes, spotlight effects. |
| `conic-gradient()` | Fades around a centre point. | Pie-chart reveals, angular sweeps. |
| `repeating-linear-gradient()` | Repeats a linear fade pattern. | Striped masks, scanlines. |

#### Syntax Rules

1. Gradient colour stops with full opacity (e.g., `black`, `#000`) represent fully visible areas.
2. `transparent` or `rgba(0,0,0,0)` represents fully hidden areas.
3. Intermediate opacity values create partial transparency.
4. `mask-mode: alpha` (the default for CSS gradients) uses the gradient's alpha channel.
5. `mask-mode: luminance` uses the gradient's brightness values instead.
6. For conic gradients, stop positions must be percentages or angular values (not unitless numbers).
7. All gradient masking is supported wherever `mask-image` is supported.

#### Constraints and Limitations

- **Alpha vs. luminance** — with `alpha` mode, `transparent` is the key; with `luminance` mode, black is hidden and white is visible.
- **Performance** — animating gradient masks can trigger repaints; use `transform` on the masked element instead where possible.
- **Browser prefixes** — older browsers require `-webkit-mask-image` for gradient masks.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Radial Gradient Vignette

**HTML File (`gradient-radial.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Radial Gradient Mask</title>
    <link rel="stylesheet" href="gradient-radial.css">
</head>
<body>
    <div class="vignette"></div>
</body>
</html>
```

**CSS File (`gradient-radial.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.vignette {
    width: 300px;
    height: 300px;
    background: linear-gradient(135deg, #3498db, #e74c3c);
    border-radius: 50%;

    /* Radial gradient mask: fully visible in centre, fading to transparent at edges */
    mask-image: radial-gradient(
        circle at center,
        black 40%,
        transparent 70%
    );
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `gradient-radial.html` and CSS as `gradient-radial.css`.
2. Open in a browser.
3. Observe the circular element has a soft vignette effect — fully visible in the centre, fading to transparent at the edges.

**Expected Output:** A circular gradient element with a soft, vignette-like fade. The centre is fully opaque, and the edges dissolve smoothly into transparency.

**Why This Works:** The `radial-gradient(circle at center, black 40%, transparent 70%)` creates a mask that is fully opaque (black) up to 40% of the radius, then fades to transparent by 70%. This creates the soft vignette effect.

---

#### Example 2: Conic Gradient Sweep

**HTML File (`gradient-conic.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Conic Gradient Mask</title>
    <link rel="stylesheet" href="gradient-conic.css">
</head>
<body>
    <div class="sweep"></div>
</body>
</html>
```

**CSS File (`gradient-conic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

@keyframes sweep {
    from { mask-image: conic-gradient(from 0deg, black 0deg 90deg, transparent 90deg 360deg); }
    to   { mask-image: conic-gradient(from 360deg, black 0deg 90deg, transparent 90deg 360deg); }
}

.sweep {
    width: 250px;
    height: 250px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    border-radius: 50%;

    /* Conic gradient mask that sweeps around */
    animation: sweep 3s linear infinite;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `gradient-conic.html` and CSS as `gradient-conic.css`.
2. Open in a browser.
3. Observe the circular element revealing itself in a sweeping, angular motion.

**Expected Output:** A circular gradient element that is revealed by a rotating conic gradient mask, creating a pie-chart-like sweeping animation.

**Why This Works:** The `conic-gradient(from 0deg, black 0deg 90deg, transparent 90deg 360deg)` creates a mask that is opaque for the first 90 degrees and transparent for the rest. The animation rotates the gradient's starting angle from 0deg to 360deg, creating the sweeping reveal effect.

---

### Real-World Cases

- **Hero image fades:** `mask-image: linear-gradient(to bottom, black 70%, transparent 100%)` for a hero image that fades into the page.
- **Vignette effects:** `radial-gradient` for a spotlight or vignette on avatars and cards.
- **Shimmer animations:** A moving linear-gradient mask that sweeps across a button.
- **Pie-chart reveals:** `conic-gradient` masks for angular progress indicators.

---

## 3. Vector and Asset Masks: SVG `<mask>` Assets and Raster Alpha Channels

### Definitions

**Core Definition:** Vector and asset masks use SVG `<mask>` elements or raster images (such as PNGs with alpha channels) as the mask source, providing complex shapes and reusable mask definitions beyond what gradients can express.

**Technical Definition:** A mask image may be interpreted using one of two different methods with regards to calculating the mask values: using the alpha channel of the mask image, or using the luminance of the mask image. The `mask-mode` property sets whether the mask reference defined by `mask-image` is treated as a luminance or alpha mask. For SVG `<mask>` elements, the `mask-type` property (or the SVG `mask-type` attribute) specifies the preferred masking mode, which can be overridden by the CSS `mask-mode` property. When a PNG with an alpha channel is used as a mask, the alpha values determine visibility — opaque areas reveal, transparent areas hide. When `mask-mode: luminance` is used, the brightness of the image determines visibility — white reveals, black hides. SVG `<mask>` elements can contain multiple shapes, gradients, and even text, making them the most flexible masking source.

**Beginner-Friendly Explanation:** Gradients are great for simple fades, but what if you want a star-shaped mask, or a mask based on the shape of text? That is where SVG `<mask>` and PNG images come in. An SVG mask is like a vector stencil — you draw the shape in SVG, and the browser uses it to determine what parts of the element to show. A PNG mask uses the image's transparency: where the PNG is opaque, the element shows; where it is transparent, the element hides. You can also use luminance mode, where white reveals and black hides. This opens up a world of creative masking possibilities.

---

### Purposes

- To use complex, non-gradient shapes as masks.
- To reuse a single mask definition across multiple elements.
- To create text-shaped masks (masking an image to the shape of text).
- To use hand-drawn or organic shapes from design tools.
- To leverage SVG's precision for intricate mask patterns.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* SVG mask reference */
selector {
    mask-image: url(#mask-id);
    mask-mode: alpha | luminance | match-source;
}

/* PNG alpha mask */
selector {
    mask-image: url(mask.png);
    mask-mode: alpha; /* or luminance */
}

/* Inline SVG mask element */
<svg>
    <mask id="mask-id" mask-type="alpha">
        <circle cx="50%" cy="50%" r="50%" fill="white" />
    </mask>
</svg>
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `url(#id)` | References an SVG `<mask>` by ID. | `url(#star-mask)` |
| `url(path.png)` | References a raster image. | `url(mask.png)` |
| `mask-mode: alpha` | Uses the alpha channel of the mask. | Default for PNG images. |
| `mask-mode: luminance` | Uses the luminance (brightness) of the mask. | Default for SVG `<mask>`. |
| `mask-type` | SVG attribute specifying the preferred mode. | `mask-type="alpha"` |

#### Syntax Rules

1. SVG `<mask>` elements are referenced by their `id` attribute using `url(#id)`.
2. PNG images with alpha channels are referenced by file path using `url(path.png)`.
3. The default `mask-mode` is `match-source`: for SVG `<mask>` elements, luminance is used; for other images, alpha is used.
4. The `mask-type` attribute on an SVG `<mask>` element specifies its preferred masking mode, which can be overridden by CSS `mask-mode`.
5. For luminance masks, white areas reveal the element and black areas hide it.
6. For alpha masks, opaque areas reveal the element and transparent areas hide it.
7. `mask-image: url(masks.svg#star)` references a specific `<mask>` element within an SVG file.

#### Constraints and Limitations

- **CORS restrictions** — external SVG mask files may be blocked by CORS in some browsers.
- **Alpha vs. luminance confusion** — the default mode depends on the source type, which can be confusing.
- **PNG file size** — PNGs with full alpha channels can be large; consider using SVG where possible.
- **Browser support** — SVG mask references are widely supported, but `mask-mode` is Baseline Newly Available (2023).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: SVG Star Mask with Luminance Mode

**HTML File (`svg-mask.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SVG Mask</title>
    <link rel="stylesheet" href="svg-mask.css">
</head>
<body>
    <!-- Inline SVG with mask definition -->
    <svg width="0" height="0" xmlns="http://www.w3.org/2000/svg">
        <mask id="star-mask" mask-type="luminance">
            <polygon points="50,0 61,35 98,35 68,57 79,91 50,70 21,91 32,57 2,35 39,35"
                     fill="white" />
        </mask>
    </svg>

    <div class="star-masked"></div>
</body>
</html>
```

**CSS File (`svg-mask.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.star-masked {
    width: 250px;
    height: 250px;
    background: linear-gradient(135deg, #f093fb, #f5576c);
    /* Reference the SVG mask by ID */
    mask-image: url(#star-mask);
    mask-mode: luminance;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `svg-mask.html` and CSS as `svg-mask.css`.
2. Open in a browser.
3. Observe that the gradient element is clipped to a star shape.

**Expected Output:** A star-shaped element filled with a pink gradient. The SVG mask defines the star shape, and the luminance mode uses the white fill to reveal the element.

**Why This Works:** The `<mask id="star-mask">` contains a white polygon (a star). The `mask-image: url(#star-mask)` references this mask, and `mask-mode: luminance` uses the brightness of the white polygon to determine visibility. White areas reveal the element; black or transparent areas hide it.

---

#### Example 2: PNG Alpha Mask

**HTML File (`png-mask.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PNG Alpha Mask</title>
    <link rel="stylesheet" href="png-mask.css">
</head>
<body>
    <div class="png-masked"></div>
</body>
</html>
```

**CSS File (`png-mask.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.png-masked {
    width: 300px;
    height: 300px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    /* Use a PNG with an alpha channel as the mask */
    mask-image: url('heart-mask.png');
    mask-mode: alpha;
    mask-size: contain;
    mask-repeat: no-repeat;
    mask-position: center;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `png-mask.html` and CSS as `png-mask.css`.
2. Place a PNG image with a heart shape and a transparent background in the same folder, named `heart-mask.png`.
3. Open in a browser.
4. Observe that the gradient element is clipped to the shape of the heart.

**Expected Output:** A heart-shaped element filled with a purple gradient. The PNG's alpha channel determines which parts of the element are visible.

**Why This Works:** The `mask-image: url('heart-mask.png')` uses the PNG's alpha channel as the mask. Opaque areas of the PNG (the heart shape) reveal the element, while transparent areas hide it. The `mask-size: contain` ensures the PNG fits within the element, and `mask-position: center` centres it.

---

### Real-World Cases

- **Text-shaped image reveals:** Using an SVG mask based on text to reveal an image through letter shapes.
- **Brand logo masks:** Using a logo as a mask to reveal a gradient or photo.
- **Organic shape masks:** Hand-drawn SVG blobs for creative section dividers.
- **Pattern masks:** Using a repeating PNG pattern as a mask for textured reveals.

---

## 4. Mask Compositing: Layering Multiple Mask Channels

### Definitions

**Core Definition:** Mask compositing is the process of combining multiple mask layers into a single final mask using the `mask-composite` property, which accepts the Porter-Duff compositing operators `add`, `subtract`, `intersect`, and `exclude`.

**Technical Definition:** The `mask-composite` CSS property represents a compositing operation used on the current mask layer with the mask layers below it. The property accepts a comma-separated list of `<compositing-operator>` keywords, each representing a Porter-Duff compositing operator that defines how the current mask layer interacts with the mask layers below it. The operators are: `add` (the source is placed over the destination; this is the default), `subtract` (the source is placed where it falls outside the destination), `intersect` (the source replaces the destination where they overlap), and `exclude` (the non-overlapping regions of source and destination are combined). The number of compositing values is matched to the number of mask images; if there are fewer values, they are repeated cyclically.

**Beginner-Friendly Explanation:** Imagine you have several stencils. `mask-composite` tells the browser how to stack them. `add` puts one on top of another (union). `subtract` cuts one shape out of another (difference). `intersect` keeps only the parts where both stencils overlap. `exclude` keeps only the parts where they do not overlap. This lets you build complex masks by combining simple shapes — for example, a circle with a star cut out of the middle.

---

### Purposes

- To combine multiple mask layers into a single complex mask.
- To create cut-out effects (subtract) and intersection effects.
- To build complex icons and shapes from simple primitives.
- To reduce the need for a single, overly complex mask image.
- To enable dynamic mask composition with CSS.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    mask-image: url(mask1.png), url(mask2.png);
    mask-composite: add | subtract | intersect | exclude;
}
```

#### Component Breakdown

| Operator | Description | Effect |
|---|---|---|
| `add` | Source placed over destination (default). | Union of both masks. |
| `subtract` | Source placed where it falls outside destination. | Source minus destination. |
| `intersect` | Source replaces destination where they overlap. | Intersection of both masks. |
| `exclude` | Non-overlapping regions of source and destination combined. | Symmetric difference. |

#### Syntax Rules

1. `mask-composite` accepts one or more comma-separated `<compositing-operator>` values.
2. The first mask image is placed on top of the ones that follow it.
3. The number of compositing values is matched to the number of mask images.
4. If there are fewer values than layers, the list is repeated cyclically.
5. The default value is `add`.
6. Mask layers are composited in order, with each layer composited against the result of the layers below it.
7. `mask-composite` is Baseline Newly Available since December 2023.

#### Constraints and Limitations

- **Layer order matters** — the first mask image is the source, and the ones below are the destination.
- **Browser support** — `mask-composite` is Baseline Newly Available; older browsers may not support all operators.
- **Complexity** — compositing multiple layers can be hard to reason about without visual testing.
- **Performance** — each additional layer increases the rasterization cost.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Circle with Star Cut-Out

**HTML File (`composite.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mask Composite</title>
    <link rel="stylesheet" href="composite.css">
</head>
<body>
    <div class="composite-mask"></div>
</body>
</html>
```

**CSS File (`composite.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.composite-mask {
    width: 300px;
    height: 300px;
    background: linear-gradient(135deg, #667eea, #764ba2);

    /* Two mask layers: a circle and a star */
    mask-image:
        radial-gradient(circle at center, black 40%, transparent 70%),
        linear-gradient(135deg, black 0%, black 100%);

    /* Subtract the circle mask from the star mask */
    mask-composite: subtract;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `composite.html` and CSS as `composite.css`.
2. Open in a browser that supports `mask-composite`.
3. Observe that the element is masked to a shape that is the star minus the circle.

**Expected Output:** A gradient element masked to the shape of a star with a circular hole cut out of its centre.

**Why This Works:** The first mask layer is a radial gradient (the circle), and the second is a linear gradient (the full square). The `mask-composite: subtract` places the circle (source) where it falls outside the square (destination), effectively subtracting the circle from the full square.

---

### Real-World Cases

- **Icon masks:** Building complex icons from simple geometric primitives.
- **Cut-out effects:** Creating a hole in an element using `subtract`.
- **Intersection reveals:** Using `intersect` to reveal content only where two masks overlap.
- **Symmetric patterns:** Using `exclude` for XOR-like mask effects.

---

## 5. Performance Impact: Rasterization Costs and Hardware Acceleration

### Definitions

**Core Definition:** Mask performance refers to the browser's rendering cost when applying masks, including rasterization work, GPU memory usage, and whether the mask can be composited on the GPU thread without triggering repaints.

**Technical Definition:** Browsers process CSS through style calculation, layout, paint, and compositing. Masks, like filters and large shadows, increase painting or GPU work. When an element has a mask, the browser must rasterize both the element and the mask, then composite them. For animated masks, the browser may promote the element to its own compositing layer to avoid repainting the entire page on each frame. The `will-change: mask` property can hint to the browser to pre-promote the element to a GPU layer during idle time, preventing first-frame stutter. Animated gradient masks that use `mask-position` or `mask-size` keyframes can be GPU-accelerated, resulting in smoother motion and improved scroll performance, as browsers can manage position changes off the main thread. However, applying masks to massive screen areas can cause significant rasterization costs, especially on lower-powered devices.

**Beginner-Friendly Explanation:** Masks are not free. The browser has to do extra work to figure out which pixels to show and which to hide. For small elements like buttons or avatars, this is negligible. But if you mask a full-screen hero image, the browser has to rasterize a huge mask and composite it every frame — that can slow things down. The good news is that modern browsers can hand mask work off to the GPU, especially if you animate the mask using `mask-position` or `mask-size` rather than changing the mask image itself. The `will-change: mask` hint tells the browser to prepare a GPU layer ahead of time.

---

### Purposes

- To understand when masks are cheap and when they are expensive.
- To optimise mask animations for GPU compositing.
- To avoid performance pitfalls with full-screen masks.
- To use `will-change` strategically to prevent first-frame stutter.
- To choose the right mask source (gradient vs. SVG vs. PNG) for performance.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    /* Hint that the mask will change */
    will-change: mask;
    /* Or promote the element to a compositor layer */
    transform: translateZ(0);
}

/* Animate mask-position/size for GPU-accelerated masks */
@keyframes shimmer {
    from { mask-position: -100% 0; }
    to   { mask-position: 200% 0; }
}
```

#### Component Breakdown

| Technique | Description | Performance Impact |
|---|---|---|
| Static mask | A mask that never changes. | Low — rasterized once. |
| Animated `mask-position` | Moving the mask across the element. | Good — can be GPU-accelerated. |
| Animated `mask-image` | Changing the mask image itself. | Poor — triggers repaints. |
| `will-change: mask` | Pre-promotes to GPU layer. | Helps — prevents first-frame stutter. |
| Full-screen mask | Mask applied to viewport-sized element. | Expensive — large rasterization area. |

#### Syntax Rules

1. Animate `mask-position` and `mask-size` rather than `mask-image` for GPU-accelerated masks.
2. Use `will-change: mask` sparingly — it consumes GPU memory.
3. Remove `will-change` after the animation completes to free memory.
4. Prefer gradient masks over large raster image masks for performance.
5. Test on lower-powered devices to ensure smooth animation.
6. Use `transform` on the masked element to promote it to a compositor layer.
7. The `mask` shorthand and longhands are Baseline Newly Available (2023).

#### Constraints and Limitations

- **GPU memory** — each promoted layer consumes VRAM proportional to its size.
- **Rasterization cost** — large masks require more texture memory and rasterization time.
- **Browser differences** — GPU acceleration behaviour varies between browsers.
- **`will-change` overuse** — applying it to too many elements causes layer explosion.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: GPU-Accelerated Shimmer Mask

**HTML File (`performance.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Performant Mask Animation</title>
    <link rel="stylesheet" href="performance.css">
</head>
<body>
    <button class="shimmer-btn">Shimmer Effect</button>
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

@keyframes shimmer {
    from { mask-position: -200% 0; }
    to   { mask-position: 300% 0; }
}

.shimmer-btn {
    padding: 20px 48px;
    font-size: 1.2rem;
    font-weight: bold;
    color: white;
    background: #3498db;
    border: none;
    border-radius: 12px;
    cursor: pointer;

    /* A gradient mask that creates a shimmer band */
    mask-image: linear-gradient(
        90deg,
        transparent 0%,
        transparent 40%,
        rgba(0, 0, 0, 0.8) 50%,
        transparent 60%,
        transparent 100%
    );
    mask-size: 200% 100%;
    mask-repeat: no-repeat;

    /* Animate the mask position (GPU-friendly) */
    animation: shimmer 2.5s linear infinite;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `performance.html` and CSS as `performance.css`.
2. Open in a browser.
3. Observe the shimmer band sweeping across the button repeatedly.

**Expected Output:** A button with a shimmering band that sweeps across it in a loop, using a gradient mask animated via `mask-position`.

**Why This Works:** The `mask-image` creates a gradient with a semi-transparent band in the middle. Animating `mask-position` from `-200%` to `300%` moves the band across the button. Because `mask-position` is a compositor-friendly property, the browser can potentially handle this animation on the GPU thread, resulting in smooth performance.

---

### Real-World Cases

- **Shimmer buttons:** GPU-accelerated shimmer effects using `mask-position` animations.
- **Image reveals:** Animating `mask-position` to reveal images on scroll.
- **Full-screen masks:** Using `will-change: mask` and `transform: translateZ(0)` to promote large masked elements to their own layer.
- **Performance-sensitive UIs:** Testing mask performance on mobile devices and low-powered hardware.

---

## References

- MDN Web Docs — `mask` - https://developer.mozilla.org/en-US/docs/Web/CSS/mask
- MDN Web Docs — `mask-mode` - https://developer.mozilla.org/en-US/docs/Web/CSS/mask-mode
- MDN Web Docs — `mask-composite` - https://developer.mozilla.org/en-US/docs/Web/CSS/mask-composite
- MDN Web Docs — `mask-image` - https://developer.mozilla.org/en-US/docs/Web/CSS/mask-image
- MDN Web Docs — Multiple masks - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_masking/Multiple_masks
- W3C — CSS Masking Module Level 1 - https://drafts.csswg.org/css-masking-1/
- CSS-Tricks — `mask-composite` - https://css-tricks.com/almanac/properties/m/mask/mask-composite/
- web.dev — CSS masking - https://web.dev/learn/css/masking
- Can I Use — CSS `mask` - https://caniuse.com/css-masks
- Can I Use — `mask-composite` - https://caniuse.com/mdn-css_properties_mask-composite
- DevBytes — Animate using CSS mask images for smooth and performant animations - https://devbytes.co.in/news/animate-using-css-mask-images-for-smooth-and-performant-animations
- LinkedIn — CSS Masking: Soft Fades and Shapes with `mask-image` - https://www.linkedin.com/posts/roeeyossef_css-frontend-webdev-activity-7437923256856211456-JOH4