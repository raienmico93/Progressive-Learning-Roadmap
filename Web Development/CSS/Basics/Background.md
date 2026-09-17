# CSS Backgrounds: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
CSS backgrounds control the visual layer painted behind an element's content, including its color, images, gradients, and how these are positioned, sized, repeated, and layered.

**Technical Definition**
According to the W3C CSS Backgrounds and Borders Module Level 3, background properties define the background layers of an element's box. Each background layer consists of a background image, a background position, a background size, a background repeat style, a background attachment, a background origin, and a background clip. The background of an element is painted beneath its content and border, and multiple comma-separated background layers can be stacked on top of one another.

**Beginner-Friendly Explanation**
Think of an element's background as a stack of transparent sheets. The bottom sheet is the background color. On top of that, you can place images, gradients, or patterns. You can decide how big each sheet is, where it sits, whether it repeats, and whether it scrolls with the page or stays fixed. The `background` shorthand lets you set all of these at once.

---

### Key Characteristics

- **Layered painting**: Multiple backgrounds are painted with the first specified layer on top and the last on the bottom.
- **Shorthand property**: The `background` property is a shorthand for eight sub-properties: `background-color`, `background-image`, `background-position`, `background-size`, `background-repeat`, `background-origin`, `background-clip`, and `background-attachment`.
- **Baseline widely available**: The core background properties have been available across browsers since July 2015.
- **Only the last layer can include a color**: When using multiple backgrounds, only the final (bottommost) layer may contain a `background-color`.
- **Images drawn on top of color**: Background images are painted on top of the background color, but the color remains visible through any transparent parts of the images.
- **Gradients are images**: CSS gradients are treated as `<image>` values and can be used anywhere `background-image` is accepted.

---

### Prerequisites

Before learning CSS backgrounds, you should understand:

1. **The CSS Box Model**: How content, padding, border, and margin combine to form an element's total size.
2. **CSS Syntax**: How properties and values are written in a rule set.
3. **CSS Selectors**: How to target elements for styling.
4. **CSS Color Values**: How to specify colors using hex, RGB, HSL, etc.
5. **CSS Units**: How to use length units like `px`, `em`, `rem`, and percentages.

---

### Related Programming Areas

- **CSS Borders**: Borders are drawn on top of backgrounds.
- **CSS Gradients**: Gradients are background images.
- **CSS Transforms**: Backgrounds transform along with their element.
- **CSS Filters**: Filters can be applied to background images.
- **SVG**: SVG images can be used as background images.
- **Design Systems**: Background tokens are used for theming and component styling.
- **Responsive Images**: `background-size` and media queries control how background images adapt.

---

### Core Concepts / Features

The following core concepts are covered using the uniform structure required: **Definitions → Purposes → Syntax Rules and Structure → Multiple Annotated Code Examples → Real-World Cases**.

---

## 1. The `background` Shorthand Property

### Definitions

**Core Definition**
The `background` shorthand property sets all background style properties at once, including color, image, origin, size, and repeat method.

**Technical Definition**
The `background` CSS shorthand property is a shorthand for the following CSS properties: `background-attachment`, `background-clip`, `background-color`, `background-image`, `background-origin`, `background-position`, `background-repeat`, and `background-size`. When multiple comma-separated values are provided, each is a background layer painted on top of the previous layers.

**Beginner-Friendly Explanation**
Instead of writing eight separate lines to set up a background, you can write one `background` declaration. It is like ordering a complete meal instead of each dish separately.

---

### Purposes

- **To set all background properties in a single declaration**, reducing repetition in stylesheets.
- **To layer multiple backgrounds** using comma-separated values.
- **To quickly reset all background properties** to their initial values.
- **To simplify responsive design** by allowing rapid background adjustments.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background: <bg-layer>#, <final-bg-layer>;
```

Where `<bg-layer>` is:

```
<bg-image> || <bg-position> [ / <bg-size> ]? || <repeat-style> || <attachment> || <visual-box> || <visual-box>
```

And `<final-bg-layer>` adds `<background-color>`.

**Breakdown**

| Component | Meaning | Default |
|-----------|---------|---------|
| `<background-color>` | The background color | `transparent` |
| `<bg-image>` | The background image (`url()`, gradient, etc.) | `none` |
| `<bg-position>` | Position of the background image | `0% 0%` |
| `<bg-size>` | Size of the background image | `auto` |
| `<repeat-style>` | How the image repeats | `repeat` |
| `<attachment>` | Whether the background scrolls or is fixed | `scroll` |
| `<visual-box>` (origin) | Origin for positioning | `padding-box` |
| `<visual-box>` (clip) | Area where background is painted | `border-box` |

**Syntax Rules**

- The `<background-color>` can only be specified in the final (last) layer.
- The `<bg-size>` value must be preceded by a `/` after `<bg-position>`.
- The two `<visual-box>` values set `background-origin` and `background-clip` respectively; if only one is provided, it sets both.
- Values can appear in any order, except that `<bg-size>` must follow `<bg-position>`.

**Constraints and Limitations**

- **Only the last layer can have a color**: Attempting to set a color on a non-final layer is invalid.
- **Shorthand resets omitted values**: Any sub-property not specified in the shorthand is reset to its initial value.
- **Image fallback**: If a background image fails to load, the browser treats it as `none`, and the background color becomes visible.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Basic Background Shorthand

**Setup Guide**

1. Create an HTML file named `index.html`.
2. Create a CSS file named `styles.css` and link it in the HTML `<head>`.
3. Paste the code below.

**HTML (`index.html`)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS Background Shorthand Example</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="hero">Welcome to Our Site</div>
</body>
</html>
```

**CSS (`styles.css`)**

```css
.hero {
    /* Shorthand: color, image, position/size, repeat, attachment */
    background: #1a1a2e url('https://mdn.github.io/shared-assets/images/examples/lizard.png') no-repeat center / cover;
    color: white;
    height: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2rem;
}
```

**Expected Output**

The hero section has a dark navy background color, with a lizard image centered and sized to cover the entire area without repeating. The text is white and centered.

**Why This Output Occurs**

The shorthand `background: #1a1a2e url(...) no-repeat center / cover` sets the background color to dark navy, the image to the lizard, disables repeating, centers the image, and sizes it to cover the element. The `background-color` is visible behind any transparent areas of the image.

---

#### Example 2: Layered Backgrounds with Shorthand

**HTML**

```html
<div class="layered-bg">
    <h1>Layered Backgrounds</h1>
</div>
```

**CSS**

```css
.layered-bg {
    /* Multiple layers, comma-separated. First is on top. */
    background:
        url('https://mdn.github.io/shared-assets/images/examples/star.png') no-repeat top right,
        url('https://mdn.github.io/shared-assets/images/examples/bubbles.png') no-repeat bottom left,
        linear-gradient(to bottom, #667eea, #764ba2);
    height: 400px;
    padding: 20px;
    color: white;
    font-size: 2rem;
}
```

**Expected Output**

The element has a purple gradient background, with a bubble image in the bottom-left and a star image in the top-right. The star is on top because it is listed first.

**Why This Output Occurs**

The `background` shorthand accepts comma-separated layers. The first layer (star) is painted on top, the second (bubbles) below it, and the gradient is at the bottom. Only the final layer (the gradient) includes the background color, which is required by the syntax.

---

### Real-World Cases

**Hero Sections**
A hero banner might use `background: url('hero.jpg') no-repeat center / cover;` to create a full-width, full-height image with a fallback color.

**Subtle Patterns**
`background: url('pattern.png') repeat #f8f9fa;` creates a subtle repeating texture over a light gray base.

**Gradient Buttons**
`background: linear-gradient(to right, #667eea, #764ba2);` creates a vibrant gradient without needing an image file.

**Theming**
A theme system might use `background: var(--bg-color) url(var(--bg-image)) no-repeat center;` to allow theme customization via CSS custom properties.

---

## 2. `background-color`

### Definitions

**Core Definition**
The `background-color` property sets the solid background color of an element.

**Technical Definition**
The `background-color` CSS property sets the background color of an element. It is specified as a single `<color>` value. The color is rendered behind any specified `background-image`, and it remains visible through any transparent or translucent parts of the image.

**Beginner-Friendly Explanation**
`background-color` is the simplest background property—it fills the element's background with a single solid color. Think of it as the "base coat" of paint before you add any pictures or patterns on top.

---

### Purposes

- **To fill an element's background with a solid color** for visual separation or branding.
- **To provide a fallback color** when a background image fails to load.
- **To establish a base layer** for layered backgrounds.
- **To create visual contrast** between an element and its surrounding content.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background-color: <color>;
background-color: transparent;   /* Default */
background-color: red;
background-color: #1a1a2e;
background-color: rgb(255 0 153);
background-color: hsl(210 80% 60%);
```

**Syntax Rules**

- Accepts any valid `<color>` value.
- The default value is `transparent`.
- The color is painted behind any background images.
- The background color extends to the edge of the element's border box by default, but `background-clip` can change this.

**Constraints and Limitations**

- **Not visible if covered by an opaque image**: If the background image is fully opaque and sized to cover the element, the background color will not be visible.
- **Accessibility**: Sufficient contrast between text color and background color is essential for readability. WCAG requires a contrast ratio of 4.5:1 for normal text and 3:1 for large text.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Background Color as Fallback

**HTML**

```html
<div class="fallback-demo">
    <p>If the image fails to load, this color shows.</p>
</div>
```

**CSS**

```css
.fallback-demo {
    background-color: #2c3e50; /* Dark slate fallback */
    background-image: url('https://example.com/nonexistent-image.jpg');
    background-size: cover;
    color: white;
    padding: 40px;
    min-height: 200px;
}
```

**Expected Output**

If the image loads, it covers the background. If it fails to load (as in this example), the dark slate color is displayed behind the white text.

**Why This Output Occurs**

`background-color` is always painted behind the background image. When the image fails to load, the browser treats it as `none`, and the background color becomes visible. This is why specifying a fallback color is a best practice.

---

#### Example 2: Background Color with Transparency

**HTML**

```html
<div class="container">
    <div class="overlay">Semi-transparent overlay</div>
</div>
```

**CSS**

```css
.container {
    background-image: url('https://mdn.github.io/shared-assets/images/examples/bubbles.png');
    background-size: cover;
    height: 300px;
    padding: 20px;
}

.overlay {
    background-color: rgba(0, 0, 0, 0.6); /* 60% black */
    color: white;
    padding: 20px;
    border-radius: 8px;
}
```

**Expected Output**

The container shows the bubble image. The overlay div has a semi-transparent black background, allowing the bubbles to show through while still providing contrast for the white text.

**Why This Output Occurs**

`rgba(0, 0, 0, 0.6)` creates a black background at 60% opacity. Because the background color is semi-transparent, the image behind it is partially visible, creating a tinted overlay effect.

---

### Real-World Cases

**Card Backgrounds**
`background-color: #ffffff` on a card creates a clean surface that separates the card from the page background.

**Error States**
`background-color: #f8d7da` on a form field indicates an error, often combined with a red border.

**Dark Mode**
`background-color: light-dark(#ffffff, #1a1a2e)` switches the background color automatically based on the user's color scheme preference.

**Button Hover States**
`background-color: #0d6efd` on hover provides visual feedback that the button is interactive.

---

## 3. `background-image`

### Definitions

**Core Definition**
The `background-image` property sets one or more background images on an element.

**Technical Definition**
The `background-image` CSS property sets one or more background images on an element. Each image is specified as the keyword `none` or as an `<image>` value. Multiple images are specified as a comma-separated list, with the first image drawn on top of the others. The background color is drawn beneath all images.

**Beginner-Friendly Explanation**
`background-image` lets you put pictures or gradients behind your content. You can stack multiple images on top of each other, and each one can be positioned, sized, and repeated independently.

---

### Purposes

- **To display decorative images** behind content without using `<img>` elements.
- **To create gradients** using CSS gradient functions.
- **To layer multiple visual effects** on a single element.
- **To display patterns or textures** that repeat across an element.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background-image: none;
background-image: url('image.jpg');
background-image: linear-gradient(to right, red, blue);
background-image: url('top.png'), url('bottom.png');
```

**Syntax Rules**

- Each background image is either `none` or an `<image>` value.
- Multiple images are separated by commas; the first image is closest to the user.
- The background color is always painted beneath all images.
- If an image cannot be loaded, the browser treats it as `none`.

**Constraints and Limitations**

- **Image failure**: If a specified image cannot be loaded, it is treated as `none`, and the background color (or underlying layers) shows through.
- **Performance**: Large background images can slow page load times; consider using optimized formats like WebP or AVIF.
- **Accessibility**: Decorative background images should not convey information that is essential for understanding the content; use `<img>` with `alt` text for meaningful images.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Multiple Background Images

**HTML**

```html
<div class="multi-image">
    <h2>Multiple Backgrounds</h2>
</div>
```

**CSS**

```css
.multi-image {
    background-image:
        url('https://mdn.github.io/shared-assets/images/examples/star.png'),
        url('https://mdn.github.io/shared-assets/images/examples/bubbles.png'),
        linear-gradient(to bottom, #667eea, #764ba2);
    background-repeat: no-repeat, no-repeat, no-repeat;
    background-position: top right, bottom left, center;
    height: 400px;
    padding: 20px;
    color: white;
}
```

**Expected Output**

The element has a purple gradient background, a star in the top-right, and bubbles in the bottom-left. The star is on top, followed by the bubbles, with the gradient at the bottom.

**Why This Output Occurs**

The `background-image` property accepts a comma-separated list. Each image is assigned a corresponding `background-repeat` and `background-position` value. The first image (star) is painted on top of the second (bubbles), which is on top of the third (gradient).

---

#### Example 2: Gradient as Background Image

**HTML**

```html
<button class="gradient-btn">Gradient Button</button>
```

**CSS**

```css
.gradient-btn {
    background-image: linear-gradient(135deg, #e94560, #0d6efd);
    color: white;
    border: none;
    padding: 12px 32px;
    border-radius: 8px;
    font-size: 16px;
    cursor: pointer;
}

.gradient-btn:hover {
    background-image: linear-gradient(135deg, #0d6efd, #e94560);
}
```

**Expected Output**

The button has a gradient from red-pink to blue. On hover, the gradient direction reverses.

**Why This Output Occurs**

`linear-gradient(135deg, #e94560, #0d6efd)` creates a gradient image that transitions from red-pink to blue at a 135-degree angle. The hover state uses a different gradient, creating a visual feedback effect.

---

### Real-World Cases

**Hero Banners**
`background-image: url('hero.jpg');` displays a large image behind a headline and call-to-action button.

**Icon Overlays**
A card might use a small icon image as a background-image in a corner, with `background-position: top right` and `background-repeat: no-repeat`.

**Patterned Backgrounds**
`background-image: url('pattern.png');` with `background-repeat: repeat` creates a seamless pattern across an element.

**Gradient Text**
`background-image: linear-gradient(...); background-clip: text; color: transparent;` creates text filled with a gradient.

---

## 4. `background-size`

### Definitions

**Core Definition**
The `background-size` property sets the size of an element's background image.

**Technical Definition**
The `background-size` CSS property sets the size of the element's background image. The image can be left at its natural size, stretched, or constrained to fit the available space. It accepts `<length>`, `<percentage>`, `auto`, `contain`, and `cover` values.

**Beginner-Friendly Explanation**
`background-size` controls how big the background image appears. You can make it a specific size, stretch it to cover the whole element, or shrink it to fit inside without cropping.

---

### Purposes

- **To stretch an image to cover an element** using `cover`.
- **To fit an image entirely within an element** using `contain`.
- **To set exact dimensions** for a background image.
- **To create responsive backgrounds** that adapt to element size.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background-size: auto;
background-size: cover;
background-size: contain;
background-size: 300px;
background-size: 300px 200px;
background-size: 50% 50%;
background-size: cover, contain;  /* For multiple images */
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `auto` | Natural size; maintains aspect ratio |
| `cover` | Scales to cover the entire container, cropping if necessary |
| `contain` | Scales to fit entirely within the container, may leave empty space |
| `<length>` | Fixed width and/or height |
| `<percentage>` | Percentage of the background positioning area |

**Syntax Rules**

- A single value sets the width; height defaults to `auto`.
- Two values set width and height respectively.
- Negative values are not allowed.
- When multiple background images are specified, each can have its own `background-size`, separated by commas.

**Constraints and Limitations**

- **`cover` crops**: If the image's aspect ratio differs from the element's, `cover` will crop the image, potentially cutting off important content.
- **`contain` leaves gaps**: If the image is smaller than the element, `contain` may leave empty space filled by the background color.
- **Percentages**: Percentages for `background-size` are relative to the background positioning area, which is determined by `background-origin` (default: padding box).

---

### Multiple Annotated Complete Code Examples

#### Example 1: Cover vs. Contain

**HTML**

```html
<div class="cover-demo">Cover</div>
<div class="contain-demo">Contain</div>
```

**CSS**

```css
.cover-demo,
.contain-demo {
    height: 200px;
    width: 300px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/lizard.png');
    background-repeat: no-repeat;
    background-position: center;
    background-color: #f8f9fa;
    margin-bottom: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    color: white;
    text-shadow: 1px 1px 2px black;
}

.cover-demo {
    background-size: cover; /* Fills the entire area, may crop */
}

.contain-demo {
    background-size: contain; /* Fits entirely inside, may leave gaps */
}
```

**Expected Output**

The "Cover" element is completely filled by the image, which may be cropped. The "Contain" element shows the entire image with empty space (background color) around it.

**Why This Output Occurs**

`cover` scales the image to the smallest size that completely covers the container, cropping any overflow. `contain` scales the image to the largest size that fits entirely within the container, leaving empty space if the aspect ratios differ.

---

#### Example 2: Exact Dimensions

**HTML**

```html
<div class="exact-size"></div>
```

**CSS**

```css
.exact-size {
    width: 400px;
    height: 300px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/star.png');
    background-size: 80px 80px; /* 80px wide, 80px tall */
    background-repeat: repeat;
    background-color: #1a1a2e;
}
```

**Expected Output**

The star image is repeated in 80×80 pixel tiles across the element.

**Why This Output Occurs**

`background-size: 80px 80px` sets each tile of the repeating background image to exactly 80 pixels wide and 80 pixels tall. The `repeat` value tiles the image across the entire element.

---

### Real-World Cases

**Full-Screen Hero Images**
`background-size: cover;` ensures a hero image fills the entire viewport without distortion, cropping as needed.

**Product Thumbnails**
`background-size: contain;` displays the entire product image within a fixed-size thumbnail, leaving space around it.

**Pattern Tiles**
`background-size: 50px 50px;` creates a repeating pattern of a specific tile size.

**Responsive Avatars**
`background-size: cover;` on a circular avatar ensures the face fills the circle without distortion.

---

## 5. `background-repeat`

### Definitions

**Core Definition**
The `background-repeat` property sets how background images are repeated (tiled).

**Technical Definition**
The `background-repeat` CSS property sets how background images are repeated. A background image can be repeated along the horizontal and vertical axes, or not repeated at all. It accepts one or two `<repeat-style>` keyword values.

**Beginner-Friendly Explanation**
`background-repeat` controls whether the background image repeats like a wallpaper pattern, and if so, in which directions.

---

### Purposes

- **To tile a small image across an element** for a seamless pattern.
- **To prevent an image from repeating**, showing it only once.
- **To repeat an image only horizontally or only vertically** for striped effects.
- **To distribute repeated images evenly** using `space` or `round`.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
/* Single value */
background-repeat: repeat;
background-repeat: repeat-x;
background-repeat: repeat-y;
background-repeat: no-repeat;
background-repeat: space;
background-repeat: round;

/* Two values: horizontal vertical */
background-repeat: repeat space;
background-repeat: no-repeat round;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `repeat` | Tiles in both directions (default) |
| `repeat-x` | Tiles horizontally only |
| `repeat-y` | Tiles vertically only |
| `no-repeat` | No tiling; image shown once |
| `space` | Tiles without clipping; distributes images evenly with gaps |
| `round` | Tiles and scales images so they fit an integer number of times |

**Syntax Rules**

- A single value is a shorthand for two values (horizontal and vertical).
- `repeat-x` equals `repeat no-repeat`; `repeat-y` equals `no-repeat repeat`.
- The first value in the two-value syntax controls horizontal repetition; the second controls vertical repetition.

**Constraints and Limitations**

- **`space` and `round` are less common**: Browser support is good, but they are less widely used than `repeat` and `no-repeat`.
- **Clipping**: With `repeat`, the last image may be clipped if it does not fit exactly.
- **`background-position` ignored with `space`**: When `space` is used and more than one image fits, `background-position` is ignored.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Common Repeat Values

**HTML**

```html
<div class="repeat-demo repeat">repeat</div>
<div class="repeat-demo repeat-x">repeat-x</div>
<div class="repeat-demo repeat-y">repeat-y</div>
<div class="repeat-demo no-repeat">no-repeat</div>
```

**CSS**

```css
.repeat-demo {
    width: 300px;
    height: 150px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/star.png');
    background-size: 40px 40px;
    background-color: #f8f9fa;
    margin-bottom: 8px;
    font-family: monospace;
    display: flex;
    align-items: center;
    justify-content: center;
}

.repeat { background-repeat: repeat; }
.repeat-x { background-repeat: repeat-x; }
.repeat-y { background-repeat: repeat-y; }
.no-repeat { background-repeat: no-repeat; }
```

**Expected Output**

- `repeat`: Stars tile in both directions.
- `repeat-x`: Stars tile horizontally only (one row).
- `repeat-y`: Stars tile vertically only (one column).
- `no-repeat`: A single star is shown.

**Why This Output Occurs**

Each value controls the tiling behavior. `repeat` tiles in both axes, `repeat-x` limits tiling to the horizontal axis, `repeat-y` limits it to the vertical axis, and `no-repeat` disables tiling entirely.

---

#### Example 2: Space vs. Round

**HTML**

```html
<div class="space-demo">space</div>
<div class="round-demo">round</div>
```

**CSS**

```css
.space-demo,
.round-demo {
    width: 350px;
    height: 100px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/star.png');
    background-size: 60px 60px;
    background-color: #1a1a2e;
    margin-bottom: 8px;
    font-family: monospace;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
}

.space-demo {
    background-repeat: space; /* Even gaps, no clipping */
}

.round-demo {
    background-repeat: round; /* Scaled to fit evenly */
}
```

**Expected Output**

- `space`: The star images are distributed evenly with gaps between them, and no image is clipped.
- `round`: The star images are scaled slightly so an integer number fits exactly within the element, with no clipping and no gaps.

**Why This Output Occurs**

`space` repeats the image as many times as possible without clipping, then distributes the remaining space as gaps between images. `round` repeats the image and scales it so that a whole number of images fits exactly, preventing both clipping and gaps.

---

### Real-World Cases

**Seamless Patterns**
`background-repeat: repeat;` with a small tile image creates a seamless wallpaper pattern.

**Striped Backgrounds**
`background-repeat: repeat-y;` with a narrow image creates vertical stripes.

**Single Icons**
`background-repeat: no-repeat;` ensures a single icon appears in a specific corner of an element.

**Responsive Patterns**
`background-repeat: round;` adapts the pattern to different element sizes without clipping or gaps.

---

## 6. `background-position`

### Definitions

**Core Definition**
The `background-position` property sets the initial position of each background image.

**Technical Definition**
The `background-position` CSS property sets the initial position for each background image. The position is relative to the position layer set by `background-origin`. It accepts one or more `<position>` values, which can be keywords, lengths, or percentages.

**Beginner-Friendly Explanation**
`background-position` tells the browser where to place the background image within the element. You can center it, put it in a corner, or use precise coordinates.

---

### Purposes

- **To center a background image** using `center`.
- **To place an image in a specific corner** using keyword values.
- **To offset an image by a precise amount** using lengths or percentages.
- **To position multiple images independently** using comma-separated values.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
/* Keyword syntax */
background-position: left top;
background-position: center;
background-position: right bottom;

/* Length/percentage syntax */
background-position: 20px 40px;
background-position: 50% 50%;
background-position: 10% 20px;

/* Three/four-value syntax (offset from edge) */
background-position: right 20px bottom 40px;
background-position: left 10% top 30%;
```

**Breakdown**

| Value Type | Meaning |
|------------|---------|
| Keyword | `left`, `right`, `top`, `bottom`, `center` |
| `<length>` | Fixed offset from the left/top edge |
| `<percentage>` | Percentage of the available space |

**Syntax Rules**

- One value sets horizontal position; vertical defaults to `center`.
- Two values set horizontal and vertical positions respectively.
- Three or four values use keywords with offsets (e.g., `right 20px bottom 40px`).
- Percentages are relative to the difference between the element's size and the image's size.

**Constraints and Limitations**

- **Invalid combinations**: `top top` and `left right` are invalid; you cannot use two vertical or two horizontal keywords.
- **Percentage behavior**: `50% 50%` centers the image, not the top-left corner. This is different from length values.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Keyword Positions

**HTML**

```html
<div class="pos-demo top-left">top left</div>
<div class="pos-demo top-right">top right</div>
<div class="pos-demo center">center</div>
<div class="pos-demo bottom-left">bottom left</div>
```

**CSS**

```css
.pos-demo {
    width: 200px;
    height: 120px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/star.png');
    background-size: 50px 50px;
    background-repeat: no-repeat;
    background-color: #f8f9fa;
    border: 1px solid #ccc;
    margin-bottom: 8px;
    font-family: monospace;
    display: flex;
    align-items: center;
    justify-content: center;
}

.top-left { background-position: left top; }
.top-right { background-position: right top; }
.center { background-position: center; }
.bottom-left { background-position: left bottom; }
```

**Expected Output**

The star appears in the specified corner or center of each element.

**Why This Output Occurs**

Keyword values position the image relative to the corresponding edges. `center` places the image at the center of the element.

---

#### Example 2: Offset Positioning

**HTML**

```html
<div class="offset-demo">Offset: right 20px bottom 10px</div>
```

**CSS**

```css
.offset-demo {
    width: 300px;
    height: 150px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/star.png');
    background-size: 40px 40px;
    background-repeat: no-repeat;
    background-position: right 20px bottom 10px;
    background-color: #1a1a2e;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: monospace;
}
```

**Expected Output**

The star is positioned 20px from the right edge and 10px from the bottom edge.

**Why This Output Occurs**

The four-value syntax `right 20px bottom 10px` sets the image's right edge 20px from the container's right edge and its bottom edge 10px from the container's bottom edge. This provides precise control over positioning.

---

### Real-World Cases

**Hero Image Positioning**
`background-position: center;` ensures a hero image is centered regardless of element size.

**Logo Placement**
`background-position: right 20px center;` places a logo on the right side, vertically centered.

**Pattern Offsets**
`background-position: 10px 10px;` offsets a repeating pattern for a specific visual effect.

**Multi-Image Layouts**
Different background-position values for each layer in a multi-background setup create complex composite designs.

---

## 7. `background-attachment`

### Definitions

**Core Definition**
The `background-attachment` property sets whether a background image's position is fixed within the viewport or scrolls with its containing block.

**Technical Definition**
The `background-attachment` CSS property sets whether a background image's position is fixed within the viewport, or scrolls with its containing block. It accepts the keyword values `scroll`, `fixed`, and `local`.

**Beginner-Friendly Explanation**
`background-attachment` controls whether the background image stays still when you scroll or moves along with the content.

---

### Purposes

- **To create a parallax-like effect** using `fixed`, where the background stays in place while content scrolls.
- **To make the background scroll with the element's content** using `local`.
- **To make the background scroll with the page** using `scroll` (default).

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background-attachment: scroll;
background-attachment: fixed;
background-attachment: local;
background-attachment: scroll, fixed;  /* For multiple images */
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `scroll` | Background fixed relative to the element itself (default) |
| `fixed` | Background fixed relative to the viewport |
| `local` | Background fixed relative to the element's contents (scrolls with content) |

**Syntax Rules**

- The property can accept multiple comma-separated values for multiple background layers.
- When `background-attachment: fixed` is set, `background-origin` is ignored.

**Constraints and Limitations**

- **`fixed` on mobile**: Fixed backgrounds can cause performance issues on mobile devices and may not work as expected.
- **`background-origin` ignored with `fixed`**: When `background-attachment: fixed`, the background positioning area is the viewport, and `background-origin` has no effect.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Fixed vs. Scroll vs. Local

**HTML**

```html
<div class="scroll-container">
    <div class="fixed-bg">fixed</div>
    <div class="scroll-bg">scroll</div>
    <div class="local-bg">local</div>
</div>
```

**CSS**

```css
.scroll-container {
    height: 400px;
    overflow-y: scroll;
    border: 2px solid #333;
    padding: 10px;
}

.fixed-bg,
.scroll-bg,
.local-bg {
    height: 250px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/bubbles.png');
    background-size: 100px 100px;
    background-repeat: repeat;
    margin-bottom: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-weight: bold;
    text-shadow: 1px 1px 2px black;
}

.fixed-bg { background-attachment: fixed; }
.scroll-bg { background-attachment: scroll; }
.local-bg { background-attachment: local; }
```

**Expected Output**

- `fixed`: The background stays fixed relative to the viewport; scrolling the container does not move the background.
- `scroll`: The background is fixed relative to the element; scrolling the container does not move the background within the element.
- `local`: The background scrolls with the element's contents.

**Why This Output Occurs**

`fixed` attaches the background to the viewport. `scroll` attaches it to the element's border. `local` attaches it to the element's scrollable content area.

---

### Real-World Cases

**Parallax Sections**
`background-attachment: fixed;` creates a parallax effect where the background appears to stay still while content scrolls over it.

**Scrollable Text Areas**
`background-attachment: local;` ensures that a background pattern scrolls along with the text inside a scrollable `<textarea>` or `<div>`.

**Fixed Hero Images**
`background-attachment: fixed;` on a hero section creates an engaging visual effect, though it should be used cautiously on mobile.

---

## 8. `background-origin` and `background-clip`

### Definitions

**Core Definition**
`background-origin` sets the origin (starting point) for background positioning, while `background-clip` sets the area where the background is painted.

**Technical Definition**
The `background-origin` property sets the background's origin: from the border start, inside the border, or inside the padding. The `background-clip` property sets whether an element's background extends underneath its border box, padding box, or content box.

**Beginner-Friendly Explanation**
`background-origin` decides where the background image starts from (the border, the padding, or the content area). `background-clip` decides where the background stops being painted (it can be clipped to the border, padding, content, or even the text itself).

---

### Purposes

- **To position backgrounds relative to different box edges** using `background-origin`.
- **To clip backgrounds to specific areas** using `background-clip`.
- **To create gradient text** using `background-clip: text`.
- **To control whether the background extends under the border** using `background-clip: padding-box`.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background-origin: border-box | padding-box | content-box;
background-clip: border-box | padding-box | content-box | text;
```

**Breakdown**

| Value | Meaning |
|-------|---------|
| `border-box` | The background is positioned/painted relative to the border box |
| `padding-box` | The background is positioned/painted relative to the padding box (default for origin) |
| `content-box` | The background is positioned/painted relative to the content box |
| `text` | The background is clipped to the foreground text (clip only) |

**Syntax Rules**

- `background-origin` is ignored when `background-attachment: fixed`.
- `background-clip: text` requires `color: transparent` to be visible.
- `background-clip: border-box` has a visual effect only when the border is partially opaque.

**Constraints and Limitations**

- **`background-clip: text` support**: Requires the `-webkit-` prefix in some browsers.
- **Root element behavior**: `background-clip` has no effect on the root element (`<html>`) because it has a different background painting area.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Origin and Clip Comparison

**HTML**

```html
<div class="box origin-border">border-box</div>
<div class="box origin-padding">padding-box</div>
<div class="box origin-content">content-box</div>
```

**CSS**

```css
.box {
    width: 200px;
    height: 100px;
    border: 20px solid rgba(233, 69, 96, 0.5); /* Semi-transparent border */
    padding: 20px;
    background-image: url('https://mdn.github.io/shared-assets/images/examples/star.png');
    background-size: 30px 30px;
    background-repeat: repeat;
    background-color: #1a1a2e;
    margin-bottom: 16px;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: monospace;
}

.origin-border { background-origin: border-box; }
.origin-padding { background-origin: padding-box; }
.origin-content { background-origin: content-box; }
```

**Expected Output**

The star pattern starts from different edges: the border edge, the padding edge, or the content edge. Because the border is semi-transparent, the background is visible beneath it in the `border-box` case.

**Why This Output Occurs**

`background-origin` determines where the background positioning area begins. `border-box` starts from the outer edge of the border, `padding-box` starts from the inner edge of the border, and `content-box` starts from the content area.

---

#### Example 2: Gradient Text with Background-Clip

**HTML**

```html
<h1 class="gradient-text">Gradient Text</h1>
```

**CSS**

```css
.gradient-text {
    background-image: linear-gradient(135deg, #e94560, #0d6efd, #155724);
    background-clip: text;
    -webkit-background-clip: text; /* For WebKit browsers */
    color: transparent;             /* Make text transparent so background shows */
    font-size: 4rem;
    font-weight: bold;
    text-align: center;
}
```

**Expected Output**

The text is filled with a three-color gradient instead of a solid color.

**Why This Output Occurs**

`background-clip: text` clips the background image to the shape of the text. With `color: transparent`, the text itself becomes invisible, allowing the gradient to show through. The `-webkit-` prefix ensures compatibility with Safari and older Chrome versions.

---

### Real-World Cases

**Gradient Text**
`background-clip: text` with a gradient is a popular technique for eye-catching headlines.

**Border-Aware Backgrounds**
`background-origin: border-box` with a semi-transparent border creates a seamless background that extends under the border.

**Content-Only Backgrounds**
`background-clip: content-box` ensures the background does not extend under padding, useful for creating inset effects.

**Image Masks**
`background-clip: text` combined with a background image creates image-filled text.

---

## 9. Multiple Backgrounds

### Definitions

**Core Definition**
Multiple backgrounds allow stacking several background images and gradients on a single element.

**Technical Definition**
Multiple backgrounds are specified as a comma-separated list of background layers. The first background provided is drawn on top, and the last is drawn at the bottom. Only the final background can include a `background-color`.

**Beginner-Friendly Explanation**
You can layer several background images on top of each other, like stacking transparent sheets of paper. The first one you list is on top.

---

### Purposes

- **To create complex visual compositions** using several images and gradients.
- **To combine a pattern with a gradient** for a richer background.
- **To add decorative elements** (icons, shapes) on top of a base background.
- **To create depth and texture** without using multiple HTML elements.

---

### Syntax Rules and Structure

**Complete General Syntax**

```css
background: background1, background2, ..., backgroundN;
```

Where each `backgroundN` can include image, position, size, repeat, attachment, origin, and clip. Only `backgroundN` (the last) can include a color.

**Syntax Rules**

- Layers are separated by commas.
- The first layer is painted on top of the others.
- Each layer can have its own `background-size`, `background-position`, `background-repeat`, etc., specified as comma-separated values in the individual properties.
- `background-color` can only be specified in the final layer.

**Constraints and Limitations**

- **Only the last layer can have a color**: This is a strict rule of the syntax.
- **Complexity**: Multi-layer backgrounds can be difficult to debug without browser developer tools.
- **Performance**: Multiple large images can impact rendering performance.

---

### Multiple Annotated Complete Code Examples

#### Example 1: Three-Layer Background

**HTML**

```html
<div class="three-layer">
    <h2>Three Background Layers</h2>
</div>
```

**CSS**

```css
.three-layer {
    background:
        url('https://mdn.github.io/shared-assets/images/examples/star.png') no-repeat top right,
        url('https://mdn.github.io/shared-assets/images/examples/bubbles.png') no-repeat bottom left,
        linear-gradient(to bottom, #667eea, #764ba2);
    height: 400px;
    padding: 20px;
    color: white;
    font-size: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

**Expected Output**

A gradient background with a star in the top-right corner and bubbles in the bottom-left corner. The star is on top, then bubbles, then the gradient.

**Why This Output Occurs**

The shorthand lists three layers. The first (star) is painted on top, the second (bubbles) below it, and the third (gradient) at the bottom. Only the gradient (last layer) implicitly provides the background color.

---

#### Example 2: Pattern Over Gradient

**HTML**

```html
<div class="pattern-over-gradient">
    <p>Content goes here</p>
</div>
```

**CSS**

```css
.pattern-over-gradient {
    background:
        repeating-linear-gradient(
            45deg,
            rgba(255, 255, 255, 0.1) 0px,
            rgba(255, 255, 255, 0.1) 10px,
            transparent 10px,
            transparent 20px
        ),
        linear-gradient(to bottom, #e94560, #0d6efd);
    height: 300px;
    padding: 20px;
    color: white;
    font-size: 1.5rem;
}
```

**Expected Output**

A blue-to-red gradient with a subtle diagonal stripe pattern overlaid on top.

**Why This Output Occurs**

The first layer is a repeating linear gradient that creates diagonal stripes at 10% white opacity. The second layer is a blue-to-red gradient. The stripes are painted on top of the gradient, creating a textured effect.

---

### Real-World Cases

**Hero Banners with Texture**
A hero section might layer a subtle noise texture over a gradient to add depth and visual interest.

**Product Cards**
A product card might layer a small "sale" badge image on top of a product photo background.

**Decorative Section Dividers**
A section divider might layer a repeating pattern over a solid color to create a decorative band.

**Complex Illustrations**
Multiple gradients and images can be combined to create complex background illustrations without external image editors.

---

## Constraints, Limitations, and Version-Specific Notes

- **Only the last layer can include `background-color`**: This is a fundamental rule of the multi-background syntax.
- **`background-origin` is ignored with `background-attachment: fixed`**: When the background is fixed to the viewport, the origin is always the viewport.
- **`background-clip: text` requires `-webkit-` prefix**: For broader browser compatibility, include both `background-clip: text` and `-webkit-background-clip: text`.
- **Fixed backgrounds on mobile**: `background-attachment: fixed` can cause performance issues and unexpected behavior on mobile devices.
- **Image failure handling**: If a background image cannot be loaded, it is treated as `none`, and the background color shows through.
- **Percentage sizing for `background-size`**: Percentages are relative to the background positioning area, which is determined by `background-origin`.
- **`background-clip` on root element**: The `background-clip` property has no effect when specified on the root element (`<html>`).
- **`space` and `round` for `background-repeat`**: These values are well-supported but less commonly used; `background-position` is ignored when `space` is used and more than one image fits.
- **Browser support for gradient interpolation**: Modern CSS allows specifying the interpolation color space for gradients using `in oklch`, `in lab`, etc. Support varies by browser and version.

---

## References

- MDN Web Docs — `background` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background
- MDN Web Docs — `background-color` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-color
- MDN Web Docs — `background-image` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-image
- MDN Web Docs — `background-size` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-size
- MDN Web Docs — `background-repeat` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-repeat
- MDN Web Docs — `background-position` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-position
- MDN Web Docs — `background-attachment` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-attachment
- MDN Web Docs — `background-clip` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-clip
- MDN Web Docs — `background-origin` CSS Property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/background-origin
- MDN Web Docs — Using Multiple Backgrounds - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Backgrounds_and_borders/Using_multiple_backgrounds
- MDN Web Docs — Resizing Background Images with `background-size` - https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Backgrounds_and_borders/Resizing_background_images
- W3C — CSS Backgrounds and Borders Module Level 3 - https://drafts.csswg.org/css-backgrounds-3/
- W3C — CSS Backgrounds and Borders Module Level 4 - https://drafts.csswg.org/css-backgrounds-4/
- MDN Web Docs — `<position>` CSS Type - https://developer.mozilla.org/en-US/docs/Web/CSS/position_value
- MDN Web Docs — `<image>` CSS Type - https://developer.mozilla.org/en-US/docs/Web/CSS/image
- MDN Web Docs — CSS Gradients - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_images/Using_CSS_gradients