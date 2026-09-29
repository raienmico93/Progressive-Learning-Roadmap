# CSS Aspect Ratio — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** The CSS `aspect-ratio` property defines a preferred width-to-height ratio for an element's box, allowing the browser to calculate and reserve the correct amount of layout space before the element's content loads. It is the primary modern solution for creating responsive media containers and preventing Cumulative Layout Shift (CLS).

**Technical Definition:** The `aspect-ratio` property is defined in the CSS Box Sizing Module Level 4. It is specified as one or both of the keyword `auto` or a `<ratio>`. When a `<ratio>` is specified, the box's preferred aspect ratio is the specified ratio of `width` / `height`; if the height is omitted, it defaults to `1`. When both `auto` and a `<ratio>` are specified together, `auto` is used if the element is a replaced element with a natural aspect ratio (such as an `<img>` element); otherwise, the specified ratio is used as the preferred aspect ratio. At least one of the box's sizes must be automatic for `aspect-ratio` to have any effect. For replaced elements, the specified ratio is used until the content is loaded; after loading, the `auto` value causes the intrinsic aspect ratio of the loaded content to take over.

**Beginner-Friendly Explanation:** The `aspect-ratio` property lets you tell the browser the shape of a box before the content inside it loads. You write `aspect-ratio: 16 / 9` and the browser reserves a box with that shape. When the image or video loads, it fits perfectly into the reserved space, and nothing on the page jumps around. This is the modern replacement for the old "padding-top hack," which required calculating a percentage and using absolutely positioned children.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Space reservation** | Reserves layout space before content loads, preventing CLS. |
| **Responsive by default** | The ratio is maintained as the container resizes. |
| **Replaced element awareness** | Uses intrinsic ratio if `auto` is specified with a replaced element. |
| **Works with one auto dimension** | At least one of `width` or `height` must be `auto`. |
| **Pairs with `object-fit`** | Controls how content is scaled within the aspect-ratio box. |
| **Legacy alternative** | The padding-top hack is the pre-`aspect-ratio` solution. |

---

### Prerequisites

- **CSS Box Model** — content, padding, border, and margin.
- **CSS `width` and `height`** — how dimensions are declared.
- **Replaced elements** — the distinction between replaced and non-replaced elements.
- **`object-fit` and `object-position`** — controlling how replaced content scales.
- **Cumulative Layout Shift (CLS)** — the Core Web Vitals metric for visual stability.

---

### Related Programming Areas

- **Web Performance** — CLS, LCP, and image optimisation.
- **Responsive Design** — fluid layouts and media containers.
- **CSS Object Model** — `object-fit` and `object-position`.
- **HTML `width` and `height` attributes** — providing intrinsic ratio information.
- **Core Web Vitals** — CLS is a key metric for user experience.

---

### Core Concepts / Features

1. The `aspect-ratio` Property: Syntax, Auto-Fallback, and Structural Implementation
2. Responsive Media Containers: Images, Video Embeds, and Card Components
3. Preventing Layout Shifts: Mitigating CLS for Core Web Vitals
4. Media Scaling Mechanics: Pairing `aspect-ratio` with `object-fit` and `object-position`

---

## 1. The `aspect-ratio` Property: Syntax, Auto-Fallback, and Structural Implementation

### Definitions

**Core Definition:** The `aspect-ratio` property defines a preferred width-to-height ratio for an element's box. It accepts the `auto` keyword, a `<ratio>` value, or both together.

**Technical Definition:** The `aspect-ratio` property is specified as one or both of the keyword `auto` or a `<ratio>`. The `auto` keyword causes replaced elements with an intrinsic aspect ratio to use that ratio; otherwise, the box has no preferred aspect ratio. Size calculations involving the intrinsic aspect ratio always work with the content box dimensions. The `<ratio>` value sets the box's preferred aspect ratio to the specified ratio of `width` / `height`; if the height is omitted, it defaults to `1`. When both `auto` and a `<ratio>` are specified, the `auto` value takes precedence if the element is a replaced element with a natural aspect ratio (like an `<img>`), otherwise the specified ratio is used. The specified aspect ratio is used in the calculation of `auto` sizes and some other layout functions.

**Beginner-Friendly Explanation:** You can write `aspect-ratio: 16 / 9` to create a widescreen box, `aspect-ratio: 1 / 1` for a square, or `aspect-ratio: 4 / 3` for a photo. If you are working with an `<img>` element and want the browser to use the image's own natural ratio instead of a fixed one, you write `aspect-ratio: auto`. If you want a fallback ratio before the image loads but the natural ratio after, you write `aspect-ratio: auto 3/4` — the browser uses the 3:4 ratio until the image loads, then switches to the image's intrinsic ratio.

---

### Purposes

- To define a preferred width-to-height ratio for a box.
- To reserve layout space before content loads, preventing CLS.
- To create consistent media containers across different content sizes.
- To use the intrinsic aspect ratio of a replaced element when available.
- To provide a modern replacement for the padding-top hack.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    aspect-ratio: auto;
    aspect-ratio: <ratio>;
    aspect-ratio: auto <ratio>;
}
```

#### Component Breakdown

| Value | Description |
|---|---|
| `auto` | Replaced elements use intrinsic ratio; otherwise no preferred ratio. |
| `<ratio>` | The box's preferred width-to-height ratio (e.g., `16 / 9`). |
| `auto <ratio>` | Uses `auto` if the element has an intrinsic ratio; otherwise uses the specified ratio. |

#### Syntax Rules

1. At least one of the element's dimensions must be `auto` for `aspect-ratio` to take effect.
2. When both `auto` and a `<ratio>` are specified, `auto` takes precedence for replaced elements with a natural ratio.
3. For non-replaced elements, the specified ratio is always used.
4. The ratio is specified as `width / height`; if the height is omitted, it defaults to `1`.
5. The specified ratio is used in the calculation of `auto` sizes.
6. Size calculations involving the preferred aspect ratio work with the dimensions of the box specified by `box-sizing`.

#### Constraints and Limitations

- **Requires one auto dimension** — if both `width` and `height` are explicitly set, `aspect-ratio` has no effect.
- **Replaced element interaction** — on `<img>` elements, the intrinsic ratio of the loaded image may override the CSS ratio (depending on the `auto` keyword).
- **Does not size content** — `aspect-ratio` only reserves the box; it does not resize the content inside. Use `object-fit` for content sizing.
- **Browser support** — supported in all modern browsers (Chrome 88+, Firefox 89+, Safari 15+).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `aspect-ratio` with Fixed Width

**HTML File (`aspect-basic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basic Aspect Ratio</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="aspect-basic.css">
</head>
<body>
    <!-- Box with 16:9 aspect ratio and fixed width -->
    <div class="box">
        <p>16:9 Aspect Ratio Box</p>
    </div>
</body>
</html>
```

**CSS File (`aspect-basic.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.box {
    /* Fixed width */
    width: 400px;
    /* Preferred aspect ratio: 16:9 */
    aspect-ratio: 16 / 9;
    /* Visible styling */
    background-color: #006064;
    color: white;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `aspect-basic.html`.
3. Save the CSS code as `aspect-basic.css` in the same folder.
4. Open `aspect-basic.html` in a web browser.
5. Observe that the box is 400px wide and 225px tall (400 × 9/16 = 225).

**Expected Output:** A dark teal box with a 16:9 aspect ratio. The width is fixed at 400px, and the height is automatically calculated to maintain the ratio.

**Why This Works:** The `width: 400px` sets the inline size. The `aspect-ratio: 16 / 9` calculates the block size from the width and the ratio. Because the height is not explicitly set (it defaults to `auto`), the browser uses the aspect ratio to determine the height.

---

#### Example 2: `aspect-ratio: auto` for Replaced Elements

**HTML File (`aspect-auto.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aspect Ratio Auto</title>
    <link rel="stylesheet" href="aspect-auto.css">
</head>
<body>
    <!-- Image with auto aspect ratio: uses the image's intrinsic ratio -->
    <img class="auto-img"
         src="https://via.placeholder.com/400x300"
         alt="Placeholder with 4:3 ratio"
         width="400"
         height="300">
    <p>This image uses its intrinsic 4:3 aspect ratio.</p>

    <!-- Image with fallback ratio: uses 3:4 until the image loads -->
    <img class="fallback-img"
         src="https://via.placeholder.com/400x300"
         alt="Placeholder with fallback ratio"
         width="400"
         height="300">
    <p>This image uses a 3:4 fallback ratio until the image loads.</p>
</body>
</html>
```

**CSS File (`aspect-auto.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.auto-img {
    /* Use the image's intrinsic aspect ratio */
    aspect-ratio: auto;
    max-width: 100%;
    height: auto;
    display: block;
    margin-bottom: 10px;
}

.fallback-img {
    /* Use 3:4 ratio until the image loads, then use intrinsic ratio */
    aspect-ratio: auto 3 / 4;
    max-width: 100%;
    height: auto;
    display: block;
    margin-bottom: 10px;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `aspect-auto.html` and CSS as `aspect-auto.css`.
2. Open in a browser with a throttled network connection.
3. Observe the first image: it uses its natural 4:3 ratio.
4. Observe the second image: it initially reserves a 3:4 box, then switches to the 4:3 ratio when the image loads.

**Expected Output:** Two images with the same source but different aspect ratio behaviours. The first uses its intrinsic ratio; the second uses a fallback ratio until the image loads, then switches to the intrinsic ratio.

**Why This Works:** The `aspect-ratio: auto` on `.auto-img` tells the browser to use the image's intrinsic ratio. The `aspect-ratio: auto 3 / 4` on `.fallback-img` tells the browser to use the 3:4 ratio as a placeholder until the image loads, then switch to the intrinsic ratio.

---

### Real-World Cases

- **Video embeds:** `aspect-ratio: 16 / 9` on iframe wrappers for YouTube and Vimeo embeds.
- **Product images:** Reserving space for product photos in e-commerce grids.
- **Avatar containers:** `aspect-ratio: 1 / 1` for square avatar containers.
- **Card media:** Consistent aspect ratio boxes for card thumbnails.

---

## 2. Responsive Media Containers: Images, Video Embeds, and Card Components

### Definitions

**Core Definition:** Responsive media containers use `aspect-ratio` to create boxes that maintain a fixed width-to-height ratio while scaling fluidly with their parent container. They are used for images, video embeds, and card media areas.

**Technical Definition:** A responsive media container is typically a block-level element with `width: 100%` and `aspect-ratio: <ratio>`. The container reserves the correct space for its content before the content loads. For video embeds (`<iframe>`), the container is paired with a child element (`width: 100%; height: 100%`) that fills the reserved box. For images, the `aspect-ratio` is often combined with `object-fit` to control how the image is scaled within the box. This pattern is the modern replacement for the padding-top hack, which required a parent container with `position: relative` and `padding-top: <percentage>`, and an absolutely positioned child.

**Beginner-Friendly Explanation:** A responsive media container is a box that keeps its shape no matter how wide the screen is. You set `width: 100%` so it fills its parent, and `aspect-ratio: 16 / 9` so it keeps a widescreen shape. Then you put your image or video inside, and it fills the box. This is how you make a YouTube embed that resizes perfectly on any screen, or a product image that always looks right in a card.

---

### Purposes

- To create media containers that scale fluidly while maintaining their shape.
- To reserve the correct space for images, videos, and embeds.
- To provide a modern replacement for the padding-top hack.
- To ensure consistent media presentation across different devices.
- To work in conjunction with `object-fit` for content scaling control.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Image container */
.media-container {
    width: 100%;
    aspect-ratio: 16 / 9;
    overflow: hidden;
}
.media-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

/* Video embed container */
.video-container {
    width: 100%;
    aspect-ratio: 16 / 9;
}
.video-container iframe {
    width: 100%;
    height: 100%;
    border: none;
}
```

#### Component Breakdown

| Element | Property | Value | Purpose |
|---|---|---|---|
| Container | `width` | `100%` | Fluid width. |
| Container | `aspect-ratio` | `16 / 9` | Preferred shape. |
| Container | `overflow` | `hidden` | Clip overflowing content. |
| Image | `width` | `100%` | Fill container width. |
| Image | `height` | `100%` | Fill container height. |
| Image | `object-fit` | `cover` | Crop to fill box. |

#### Syntax Rules

1. The container must have `width: 100%` (or another fluid width) for responsive behaviour.
2. The `aspect-ratio` on the container reserves the correct height.
3. For images, use `object-fit` to control how the image fills the box.
4. For iframes, set `width: 100%` and `height: 100%` on the iframe to fill the container.
5. Use `overflow: hidden` to clip content that overflows the container.
6. The container's aspect ratio can be changed at breakpoints using media queries or container queries.

#### Constraints and Limitations

- **Image ratio mismatch** — if the image's intrinsic ratio differs from the container's ratio, `object-fit` is required to avoid distortion.
- **iframe sizing** — the iframe must be set to `width: 100%; height: 100%` to fill the container.
- **Border and padding** — if the container has a border or padding, the aspect ratio may be affected depending on `box-sizing`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Responsive Video Embed

**HTML File (`video-embed.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Video Embed</title>
    <link rel="stylesheet" href="video-embed.css">
</head>
<body>
    <div class="video-container">
        <iframe src="https://www.youtube.com/embed/dQw4w9WgXcQ"
                title="Video"
                allowfullscreen></iframe>
    </div>
    <p>
        The video container uses <code>aspect-ratio: 16 / 9</code> to reserve
        the correct space before the iframe loads.
    </p>
</body>
</html>
```

**CSS File (`video-embed.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
    line-height: 1.6;
}

.video-container {
    /* Fluid width */
    width: 100%;
    /* 16:9 aspect ratio */
    aspect-ratio: 16 / 9;
    /* Clip overflow */
    overflow: hidden;
    /* Visible styling */
    background-color: #000;
    border-radius: 10px;
    margin-bottom: 20px;
}

.video-container iframe {
    /* Fill the reserved box */
    width: 100%;
    height: 100%;
    border: none;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `video-embed.html` and CSS as `video-embed.css`.
2. Open in a browser with a throttled network connection.
3. Observe that the black container reserves the correct 16:9 space before the iframe loads. The paragraph below does not shift when the video appears.

**Expected Output:** A black container with a 16:9 aspect ratio that reserves space for the embedded video. The video fills the container once loaded, with zero layout shift.

**Why This Works:** The `aspect-ratio: 16 / 9` on `.video-container` calculates the height from the container's width. Because the aspect ratio is known upfront, the browser allocates the space before the iframe loads. The iframe's `width: 100%` and `height: 100%` fill the reserved box.

---

#### Example 2: Responsive Card Media

**HTML File (`card-media.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Responsive Card Media</title>
    <link rel="stylesheet" href="card-media.css">
</head>
<body>
    <div class="card-grid">
        <div class="card">
            <div class="card-media">
                <img src="https://via.placeholder.com/800x450" alt="Card 1">
            </div>
            <div class="card-body">
                <h3>Card 1</h3>
                <p>Short description.</p>
            </div>
        </div>
        <div class="card">
            <div class="card-media">
                <img src="https://via.placeholder.com/800x600" alt="Card 2">
            </div>
            <div class="card-body">
                <h3>Card 2</h3>
                <p>A much longer description that takes up more space.</p>
            </div>
        </div>
    </div>
</body>
</html>
```

**CSS File (`card-media.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    max-width: 700px;
    margin: 0 auto;
}

.card {
    background-color: white;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card-media {
    /* Reserve a 16:9 box for the image */
    aspect-ratio: 16 / 9;
    width: 100%;
    overflow: hidden;
}

.card-media img {
    /* Fill the box and crop to cover */
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.card-body {
    padding: 15px;
}

.card-body h3 {
    margin: 0 0 5px;
}

.card-body p {
    margin: 0;
    color: #555;
    font-size: 0.9rem;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `card-media.html` and CSS as `card-media.css`.
2. Open in a browser.
3. Observe that both cards have the same 16:9 media area, even though the images have different intrinsic ratios (16:9 and 4:3).

**Expected Output:** Two cards with identical 16:9 media areas. The images fill the areas and are cropped by `object-fit: cover` to fit the ratio.

**Why This Works:** The `.card-media` container reserves a 16:9 box. The `object-fit: cover` on the image scales it to fill the box while preserving its aspect ratio, cropping the overflow. This creates consistent media areas across cards with different image ratios.

---

### Real-World Cases

- **Video embeds:** YouTube, Vimeo, and other iframe embeds.
- **Product images:** Consistent aspect ratio product photos in e-commerce grids.
- **Blog post thumbnails:** Featured images with a consistent aspect ratio.
- **Hero banners:** Full-width hero images with a fixed aspect ratio.

---

## 3. Preventing Layout Shifts: Mitigating Cumulative Layout Shift (CLS) for Core Web Vitals

### Definitions

**Core Definition:** Cumulative Layout Shift (CLS) is a Core Web Vitals metric that measures the amount of unexpected visual movement of page content during loading and interaction. The `aspect-ratio` property, combined with HTML `width` and `height` attributes, is the primary mechanism for preventing CLS caused by media loading.

**Technical Definition:** CLS is calculated by multiplying the impact fraction (the fraction of the viewport affected by the shift) by the distance fraction (the distance the element moved, as a fraction of the viewport). Google recommends a CLS score of 0.1 or less at the 75th percentile for a good user experience; scores above 0.25 are considered poor. In responsive design, CLS is primarily caused by images, videos, iframes, ads, and web fonts that load without their dimensions being specified in advance. The primary mitigation techniques are: setting explicit `width` and `height` attributes on media elements; using the CSS `aspect-ratio` property to reserve space before content loads; setting `min-height` on dynamic containers; and using `font-display: optional` or preloading to reduce font-swap shifts. Reserving exact dimensions for images, ads, and dynamic embeds eliminates layout shifts, reducing CLS from 0.25+ to under 0.1.

**Beginner-Friendly Explanation:** CLS is what happens when you are reading a webpage and suddenly the text jumps down because an image loaded above it. It is frustrating and can cause you to click the wrong thing. The fix is simple: tell the browser how much space an element will need before it loads. For images, add `width` and `height` attributes. For ads and dynamic content, set a `min-height` on their container. For responsive containers, use `aspect-ratio`. This way, the browser reserves the space, and when the content finally arrives, nothing moves.

---

### Purposes

- To ensure visual stability by preventing unexpected content movement during loading.
- To improve the Core Web Vitals CLS score, which affects search ranking.
- To provide a better user experience by allowing users to maintain their reading position.
- To reduce the risk of misclicks on buttons, links, or form elements.
- To align layout behaviour with the browser's rendering pipeline by providing complete geometry upfront.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Reserve space for images using width/height attributes */
img {
    max-width: 100%;
    height: auto;
}

/* Reserve space with aspect-ratio */
.media-container {
    aspect-ratio: 16 / 9;
    width: 100%;
}

/* Reserve space with min-height */
.dynamic-content {
    min-height: 250px;
}

/* Reserve space for scrollbars */
html {
    scrollbar-gutter: stable;
}
```

#### Component Breakdown

| Technique | Description | CLS Impact |
|---|---|---|
| `width` + `height` attributes | HTML attributes define intrinsic aspect ratio. | ✅ Eliminates media CLS. |
| `aspect-ratio` | Explicitly sets the aspect ratio of a box. | ✅ Eliminates container CLS. |
| `min-height` | Reserves vertical space for dynamic content. | ✅ Reduces dynamic CLS. |
| `scrollbar-gutter: stable` | Reserves space for the scrollbar. | ✅ Eliminates scrollbar-induced shifts. |
| `font-display: optional` | Prevents font swap after block period. | ✅ Eliminates font-swap CLS. |

#### Syntax Rules

1. Always include `width` and `height` attributes on `<img>` and `<video>` elements.
2. The browser calculates the aspect ratio from the attributes and reserves space before the image loads.
3. `aspect-ratio` on a container reserves space for embedded content (iframes, videos).
4. `min-height` on a container reserves space for dynamic content (ads, widgets).
5. `scrollbar-gutter: stable` prevents layout shifts caused by scrollbar appearance.
6. `font-display: optional` prevents font-swap CLS but may mean the custom font is not used on first visit.

#### Constraints and Limitations

- **`aspect-ratio` requires a width** — it does not work on elements with `width: auto` unless a width is otherwise established.
- **`min-height` is a minimum** — if content exceeds it, the container still grows, potentially causing shift.
- **Font-swap CLS** — `font-display: swap` can cause shifts if the fallback and custom fonts have different metrics; use `optional` or preload for critical fonts.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Preventing Image and Ad Layout Shift

**HTML File (`cls-prevention.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CLS Prevention</title>
    <link rel="stylesheet" href="cls-prevention.css">
</head>
<body>
    <!-- Image with width and height: space reserved -->
    <img src="https://via.placeholder.com/800x450"
         width="800"
         height="450"
         alt="Placeholder with reserved space">

    <p>
        The image above has <code>width</code> and <code>height</code> attributes,
        so the browser reserves the correct aspect ratio (16:9) even before the
        image file downloads. No layout shift occurs.
    </p>

    <!-- Ad container with min-height: space reserved -->
    <div class="ad-container">
        <p>Ad content will appear here after loading.</p>
    </div>

    <p>
        The container above has a <code>min-height</code> set, so the space is
        reserved even before the ad content loads.
    </p>
</body>
</html>
```

**CSS File (`cls-prevention.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    max-width: 700px;
    margin: 0 auto;
    padding: 20px;
    line-height: 1.6;
}

img {
    /* Responsive images: fill container width */
    max-width: 100%;
    /* Height is auto, but the browser uses the width/height attributes
       to calculate the aspect ratio and reserve space */
    height: auto;
    display: block;
    margin-bottom: 20px;
    background-color: #f0f0f0; /* Placeholder colour while loading */
}

.ad-container {
    /* Reserve vertical space for dynamic ad content */
    min-height: 250px;
    background-color: #fff9c4;
    border: 2px dashed #f9a825;
    padding: 15px;
    margin-bottom: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #666;
}

/* Optional: reserve space for scrollbar */
html {
    scrollbar-gutter: stable;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `cls-prevention.html` and CSS as `cls-prevention.css`.
2. Open in a browser with a throttled network connection (DevTools → Network → Slow 3G).
3. Observe that the image area is reserved before the image loads; the surrounding text does not jump when the image appears.
4. Observe that the ad container has a fixed minimum height, so the text below does not shift when ad content loads.

**Expected Output:** An image with a visible placeholder area that occupies the correct 16:9 aspect ratio from the start. Below it, a yellow dashed container with a reserved 250px height. The text does not move when the image or ad content loads.

**Why This Works:** The `width` and `height` attributes on the `<img>` element allow the browser to calculate the intrinsic aspect ratio (800:450 = 16:9) and reserve the correct space before the image file is downloaded. The `min-height` on `.ad-container` reserves vertical space for the dynamic ad content. Both techniques give the browser complete geometry upfront, eliminating reflows and layout shifts.

---

### Real-World Cases

- **E-commerce product pages:** Product images with `width` and `height` attributes prevent layout shifts that could cause users to misclick "Add to Cart."
- **News sites:** Reserving space for ad slots and cookie banners prevents content from jumping as these elements load.
- **Progressive web apps (PWAs):** Using `aspect-ratio` on media containers ensures a stable layout during offline-to-online transitions.

---

## 4. Media Scaling Mechanics: Pairing `aspect-ratio` with `object-fit` and `object-position`

### Definitions

**Core Definition:** The `object-fit` property controls how the content of a replaced element (such as an `<img>` or `<video>`) is resized to fit its container's box. The `object-position` property controls the alignment of that content within the box. When paired with `aspect-ratio`, these properties create precise control over media scaling and cropping.

**Technical Definition:** The `object-fit` CSS property sets how the content of a replaced element should be resized to fit its container. It accepts five keyword values: `fill` (the default, which stretches the content to fill the box, ignoring aspect ratio), `contain` (scales the content to fit within the box while preserving aspect ratio, producing letterboxing), `cover` (scales the content to fill the box while preserving aspect ratio, cropping overflow), `none` (renders the content at its intrinsic size, ignoring the box dimensions), and `scale-down` (renders the content at the smaller of `none` or `contain`). The `object-position` property specifies the alignment of the replaced element's contents within the element's box, accepting one to four values that define a 2D position. When `aspect-ratio` is applied to a container, `object-fit: cover` ensures the image fills the box without distortion, and `object-position` controls which part of the image remains visible.

**Beginner-Friendly Explanation:** When you put an image into a box that has a different shape than the image, something has to give. `object-fit: cover` makes the image fill the box by cropping the overflow — like cropping a photo to fit a frame. `object-fit: contain` makes the image fit inside the box without cropping, leaving empty space around it. `object-position` then lets you choose which part of the image stays visible when it is cropped — for example, aligning a portrait photo to the top so the face is not cut off. Together with `aspect-ratio`, these properties give you complete control over how media looks in a responsive container.

---

### Purposes

- To control how replaced content is resized within a fixed-dimension container.
- To preserve aspect ratio while filling a container (`cover`) or fitting within it (`contain`).
- To prevent image distortion when the container's aspect ratio differs from the image's.
- To align the visible portion of a cropped image using `object-position`.
- To create consistent card, thumbnail, and hero image treatments.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    object-fit: fill | contain | cover | none | scale-down;
    object-position: <position>;
}
```

#### Component Breakdown

| `object-fit` Value | Behaviour | Aspect Ratio | Overflow |
|---|---|---|---|
| `fill` | Stretches content to fill box. | Ignored | None (distorted) |
| `contain` | Scales content to fit inside box. | Preserved | Letterboxing |
| `cover` | Scales content to fill box. | Preserved | Cropped |
| `none` | Renders at intrinsic size. | Preserved | Overflow |
| `scale-down` | Smaller of `none` or `contain`. | Preserved | Letterboxing or none |

| `object-position` Syntax | Example | Description |
|---|---|---|
| Keyword | `top`, `bottom`, `left`, `right`, `center` | Single-axis alignment. |
| Percentage | `25% 75%` | Horizontal and vertical position. |
| Length | `0 0`, `10px 20px` | Absolute offsets. |

#### Syntax Rules

1. `object-fit` applies only to replaced elements (`<img>`, `<video>`, `<iframe>`, `<embed>`, `<object>`).
2. The default value is `fill`, which stretches content and ignores aspect ratio.
3. `object-position` has no effect unless `object-fit` is set to a non-default value.
4. `object-position` accepts one to four values defining a 2D position.
5. The default `object-position` is `50% 50%` (center).
6. Percentage values for `object-position` are relative to the difference between the content's intrinsic size and the box's size.

#### Constraints and Limitations

- **No effect on non-replaced elements** — `object-fit` has no effect on `<div>`, `<p>`, or other non-replaced elements.
- **Background images excluded** — `object-fit` does not apply to `background-image`; use `background-size` instead.
- **`object-position` requires `object-fit`** — the positioning property is ignored when `object-fit` is `fill` (the default).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing `object-fit` Values with `aspect-ratio`

**HTML File (`object-fit-aspect.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Object Fit with Aspect Ratio</title>
    <link rel="stylesheet" href="object-fit-aspect.css">
</head>
<body>
    <div class="gallery">
        <figure>
            <img class="fit-cover" src="https://via.placeholder.com/400x300" alt="Cover">
            <figcaption>object-fit: cover</figcaption>
        </figure>
        <figure>
            <img class="fit-contain" src="https://via.placeholder.com/400x300" alt="Contain">
            <figcaption>object-fit: contain</figcaption>
        </figure>
        <figure>
            <img class="fit-fill" src="https://via.placeholder.com/400x300" alt="Fill">
            <figcaption>object-fit: fill</figcaption>
        </figure>
    </div>
</body>
</html>
```

**CSS File (`object-fit-aspect.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.gallery {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
}

figure {
    margin: 0;
    text-align: center;
}

img {
    /* Fixed box dimensions with aspect ratio */
    width: 200px;
    aspect-ratio: 1 / 1; /* Square box */
    border: 2px solid #006064;
    border-radius: 8px;
    background-color: #e0f7fa;
}

.fit-cover {
    /* Fill the box, crop overflow */
    object-fit: cover;
    object-position: center;
}

.fit-contain {
    /* Fit inside the box, letterbox */
    object-fit: contain;
}

.fit-fill {
    /* Stretch to fill, ignore ratio */
    object-fit: fill;
}

figcaption {
    margin-top: 8px;
    font-size: 0.8rem;
    color: #333;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `object-fit-aspect.html` and CSS as `object-fit-aspect.css`.
2. Open in a browser.
3. Observe the three images: `cover` fills the square box (cropping the sides), `contain` fits inside the square (letterboxing), and `fill` stretches the image (distorting it).

**Expected Output:** Three square boxes (200×200) each displaying the same 4:3 image with different `object-fit` behaviours. The differences are immediately visible: `cover` crops, `contain` letterboxes, and `fill` distorts.

**Why This Works:** The `aspect-ratio: 1 / 1` on the `img` creates a square box. Each `object-fit` value applies a different sizing algorithm to the replaced content within that box. `cover` scales the image to fill the box, cropping the overflow. `contain` scales it to fit inside, leaving empty space. `fill` stretches it to fill, ignoring the aspect ratio and distorting the image.

---

#### Example 2: `object-position` for Cropped Images

**HTML File (`object-position-aspect.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Object Position with Aspect Ratio</title>
    <link rel="stylesheet" href="object-position-aspect.css">
</head>
<body>
    <div class="avatars">
        <figure>
            <img class="avatar top" src="https://via.placeholder.com/300x400" alt="Top">
            <figcaption>object-position: top</figcaption>
        </figure>
        <figure>
            <img class="avatar center" src="https://via.placeholder.com/300x400" alt="Center">
            <figcaption>object-position: center</figcaption>
        </figure>
        <figure>
            <img class="avatar bottom" src="https://via.placeholder.com/300x400" alt="Bottom">
            <figcaption>object-position: bottom</figcaption>
        </figure>
    </div>
</body>
</html>
```

**CSS File (`object-position-aspect.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.avatars {
    display: flex;
    gap: 20px;
    justify-content: center;
}

figure {
    margin: 0;
    text-align: center;
}

.avatar {
    /* Square container for avatars */
    width: 120px;
    aspect-ratio: 1 / 1;
    border-radius: 50%;
    /* Fill the circle, cropping the overflow */
    object-fit: cover;
    border: 3px solid #006064;
}

.top {
    object-position: top;
}

.center {
    object-position: center;
}

.bottom {
    object-position: bottom;
}

figcaption {
    margin-top: 8px;
    font-size: 0.75rem;
    color: #333;
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `object-position-aspect.html` and CSS as `object-position-aspect.css`.
2. Open in a browser.
3. Observe that all three avatars are circular, but they show different portions of the portrait depending on `object-position`.

**Expected Output:** Three circular avatars. The first shows the top of the portrait, the second the centre, and the third the bottom. The `object-fit: cover` ensures the portrait fills the circle without distortion, and `object-position` controls which part is visible.

**Why This Works:** The `aspect-ratio: 1 / 1` on `.avatar` creates a square box, and `border-radius: 50%` makes it circular. `object-fit: cover` scales the portrait to fill the square, cropping the overflow. `object-position: top` aligns the top of the image with the top of the box, showing the upper portion. `object-position: center` shows the middle, and `object-position: bottom` shows the lower portion.

---

### Real-World Cases

- **Card thumbnails:** `object-fit: cover` for product cards where images of different aspect ratios must fill a uniform card image area.
- **Hero banners:** `object-fit: cover` with `object-position: center` for full-bleed hero images.
- **Avatar circles:** `object-fit: cover` with `object-position: top` for profile pictures.
- **Video players:** `object-fit: contain` for videos that should not be cropped, with letterboxing.

---

## References

- MDN Web Docs — `aspect-ratio` - https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio
- MDN Web Docs — `object-fit` - https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit
- MDN Web Docs — `object-position` - https://developer.mozilla.org/en-US/docs/Web/CSS/object-position
- W3C — CSS Box Sizing Module Level 4 - https://www.w3.org/TR/css-sizing-4/
- web.dev — CSS `aspect-ratio` property - https://web.dev/articles/aspect-ratio
- web.dev — Responsive images - https://web.dev/learn/design/responsive-images
- web.dev — Optimize Cumulative Layout Shift - https://web.dev/articles/optimize-cls
- CSS-Tricks — `aspect-ratio` - https://css-tricks.com/almanac/properties/a/aspect-ratio/
- Can I Use — `aspect-ratio` - https://caniuse.com/mdn-css_properties_aspect-ratio
- Can I Use — `object-fit` - https://caniuse.com/object-fit