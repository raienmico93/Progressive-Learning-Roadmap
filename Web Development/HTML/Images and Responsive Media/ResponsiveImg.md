# Responsive Images — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Responsive images are a set of HTML techniques that allow a single `<img>` or `<picture>` element to serve different image resources to different devices, viewports, and screen resolutions. The goal is to deliver the most appropriate image — in terms of dimensions, resolution, and format — for each user's context, without forcing mobile devices to download desktop-sized assets or forcing high-density screens to display low-resolution images.

**Technical Definition**
In the WHATWG HTML Living Standard, responsive images are implemented through four complementary mechanisms: the `srcset` attribute (a comma-separated list of image candidates with width or pixel-density descriptors), the `sizes` attribute (a list of media conditions paired with rendered-size hints), the `<picture>` element (a container for zero or more `<source>` elements followed by exactly one `<img>`), and the `<source>` element (which supplies candidate resources with `media`, `type`, and `srcset` attributes). The `srcset` attribute supports **width descriptors** (`w`) for resolution switching and **pixel-density descriptors** (`x`) for fixed-size images; when width descriptors are used, `sizes` must also be present or the `srcset` is ignored. The `<picture>` element evaluates its `<source>` children in order, selecting the first whose `media` condition and `type` are both supported, and falls back to the `<img>` element for browsers that lack `<picture>` support or when no `<source>` matches.

**Beginner-Friendly Explanation**
Responsive images solve a simple problem: a picture that looks great on a desktop can be a waste of bandwidth on a phone, and a small picture that loads quickly on a phone can look blurry on a high-resolution laptop. Responsive images let you give the browser a menu of options — several sizes, several crops, several formats — and let the browser pick the best one for the device it is running on. You do not have to detect the device yourself; the browser knows the screen size, pixel density, and format support, and it chooses accordingly.

---

### Key Characteristics

- **Browser-driven selection**: The browser, not the developer, chooses the final image from the candidates provided, based on viewport width, device pixel ratio, and format support.
- **Two problems, two solutions**: **Resolution switching** (same image, different sizes/resolutions) is solved with `srcset` and `sizes` on `<img>`; **art direction** (different crops or compositions) is solved with `<picture>` and `<source media>`.
- **Format negotiation**: `<picture>` with `<source type>` lets you offer modern formats (AVIF, WebP) with legacy fallbacks (JPEG, PNG).
- **Mandatory pairing**: `srcset` with `w` descriptors requires `sizes`; omitting `sizes` causes the browser to assume `100vw` and download the largest available image.
- **Fallback discipline**: The `<img>` element inside `<picture>` is required and provides the fallback for browsers that do not support `<picture>` or for cases where no `<source>` matches.
- **Layout stability**: Responsive images should always carry `width` and `height` attributes to reserve space and prevent cumulative layout shift (CLS).

---

### Prerequisites

1. **Basic HTML image syntax**: familiarity with `<img src alt>`.
2. **URL fundamentals**: understanding of absolute and relative URLs.
3. **CSS media queries**: understanding of `max-width`, `min-width`, and viewport units (`vw`).
4. **Device pixel ratio (DPR)**: awareness that a 2x screen uses two physical pixels per CSS pixel.
5. **Core Web Vitals basics**: understanding CLS (layout shift) and LCP (loading performance).

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Core Web Vitals** | Responsive images improve LCP by reducing payload and prevent CLS through dimensions. |
| **Accessibility (WCAG)** | `alt` text remains mandatory regardless of which candidate is selected. |
| **Performance budgets** | `srcset` and modern formats reduce bytes transferred, especially on mobile. |
| **CDN image transformation** | Services like Cloudflare Images and Imgix generate candidate URLs dynamically. |
| **Art direction** | `<picture>` enables design-specific crops for different breakpoints. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Responsive images** — the overarching discipline
2. **`srcset`** — the candidate list
3. **`sizes`** — the rendered-size hints
4. **`<picture>`** — the art-direction and format container
5. **`<source>`** — the candidate resource element
6. **Art direction** — different crops for different layouts
7. **Resolution switching** — different sizes/resolutions of the same image
8. **Format switching** — modern formats with fallbacks
9. **Responsive image best practices** — consolidated guidelines

---

## 1. Responsive Images — The Discipline

### Definitions

**Core Definition**
Responsive images are the practice of delivering appropriately sized, cropped, and formatted images to each user based on their device's viewport and pixel density, using HTML's native selection mechanisms rather than server-side device detection or JavaScript.

**Technical Definition**
Responsive images are implemented through the **descriptive syntax** (`srcset` and `sizes`), where the developer provides candidates and hints and the browser decides, and the **prescriptive syntax** (`<picture>` with `<source>`), where the developer explicitly controls which resource is used under which conditions. The descriptive syntax is the common case for most images; the prescriptive syntax is reserved for art direction and format negotiation, where the browser cannot infer the developer's intent.

**Beginner-Friendly Explanation**
Responsive images are pictures that adapt to the screen. Instead of making one image file do all the work, you provide several versions and let the browser pick the one that fits the device.

---

### Purposes

- **To reduce bandwidth waste** on mobile devices that do not need desktop-sized images.
- **To improve visual quality** on high-density screens by serving higher-resolution candidates.
- **To enable art direction** — showing different crops or compositions at different breakpoints.
- **To support modern image formats** while retaining fallbacks for older browsers.
- **To improve Core Web Vitals** by reducing image payload and preventing layout shift.

---

### Syntax Rules and Structure

#### Complete General Syntax (Descriptive)

```html
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Description"
  width="800"
  height="600">
```

#### Complete General Syntax (Prescriptive)

```html
<picture>
  <source media="(max-width: 640px)" srcset="hero-mobile.avif" type="image/avif">
  <source media="(min-width: 641px)" srcset="hero-desktop.avif" type="image/avif">
  <img src="hero-desktop.jpg" alt="Description" width="1200" height="630">
</picture>
```

**Component breakdown**

| Component | Role |
|---|---|
| `src` | Fallback URL; required on `<img>`. |
| `srcset` | Candidate list with `w` or `x` descriptors. |
| `sizes` | Media conditions paired with rendered sizes; required with `w` descriptors. |
| `<picture>` | Container for `<source>` elements plus one `<img>`. |
| `<source>` | Candidate with `media`, `type`, and `srcset`. |

#### Syntax Rules

1. **Use `srcset` + `sizes` for resolution switching** — same image, different sizes/resolutions.
2. **Use `<picture>` + `<source media>` for art direction** — different crops or compositions.
3. **Use `<picture>` + `<source type>` for format negotiation** — AVIF/WebP with JPEG fallback.
4. **Always provide `width` and `height`** on the `<img>` to prevent CLS.
5. **Always provide `alt`** text; the selected candidate does not change the image's meaning.

#### Constraints and Limitations

- **`srcset` with `w` descriptors is ignored if `sizes` is absent**.
- **The `<img>` inside `<picture>` is mandatory** and must be the last child.
- **The browser may not choose the candidate you expect**; it uses its own heuristics based on network conditions and DPR.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Resolution Switching with `srcset` and `sizes`

**Setup Guide**: Save as `responsive-srcset.html`. Provide images at 400w, 800w, and 1200w.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Resolution Switching</title>
</head>
<body>
  <img
    src="photo-800.jpg"
    srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, (max-width: 900px) 50vw, 33vw"
    alt="A mountain landscape at sunrise"
    width="1200"
    height="800">
</body>
</html>
```

**Expected output**: On a narrow phone screen, the browser may choose `photo-400.jpg`; on a tablet, `photo-800.jpg`; on a wide desktop, `photo-1200.jpg`. The `sizes` attribute tells the browser the image occupies 100vw, 50vw, or 33vw depending on the breakpoint, so it can calculate the effective pixel density and select the smallest adequate candidate.

**Why this output occurs**: `srcset` provides the candidates with their intrinsic widths, and `sizes` provides the rendered-size hints. The browser divides each candidate's width descriptor by the source size from `sizes` to compute the effective density, then chooses the best candidate for the device's DPR.

---

#### Example 2: Art Direction with `<picture>`

**Setup Guide**: Save as `responsive-art.html`. Provide a mobile crop and a desktop crop.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Art Direction</title>
</head>
<body>
  <picture>
    <!-- Mobile: tight portrait crop -->
    <source media="(max-width: 640px)" srcset="hero-mobile.jpg">
    <!-- Desktop: wide landscape crop -->
    <source media="(min-width: 641px)" srcset="hero-desktop.jpg">
    <!-- Fallback for browsers without <picture> -->
    <img src="hero-desktop.jpg" alt="A mountain landscape" width="1200" height="630">
  </picture>
</body>
</html>
```

**Expected output**: On screens up to 640px wide, the mobile crop is displayed; on wider screens, the desktop crop is displayed. Browsers that do not support `<picture>` display the fallback `hero-desktop.jpg`.

**Why this output occurs**: The browser evaluates each `<source>` in order and selects the first whose `media` condition matches. The `<img>` provides the fallback and carries the dimensions and `alt` text.

---

#### Example 3: Format Switching with Fallbacks

**Setup Guide**: Save as `responsive-format.html`. Provide AVIF, WebP, and JPEG versions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Format Switching</title>
</head>
<body>
  <picture>
    <source type="image/avif" srcset="photo.avif">
    <source type="image/webp" srcset="photo.webp">
    <img src="photo.jpg" alt="A city skyline at dusk" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: Browsers that support AVIF load `photo.avif`; browsers that support WebP but not AVIF load `photo.webp`; browsers that support neither load `photo.jpg`.

**Why this output occurs**: The browser selects the first `<source>` whose `type` it supports. The `<img>` provides the universal fallback.

---

### Real-World Cases

1. **E-commerce product galleries**: Resolution switching ensures fast loading on mobile and crisp images on retina desktops.
2. **News hero images**: Art direction provides a tight portrait crop on phones and a wide landscape crop on desktops.
3. **Documentation screenshots**: Format switching serves AVIF to modern browsers and PNG to legacy environments.

---

### References

- MDN Web Docs — Responsive images — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images
- web.dev — Responsive images — https://web.dev/learn/images/responsive-images
- web.dev — Serve responsive images — https://web.dev/articles/serve-responsive-images
- Chrome for Developers — Performance best practices — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance

---

## 2. `srcset` — The Candidate List

### Definitions

**Core Definition**
`srcset` defines the set of images the browser is allowed to choose from, and the size or pixel density of each candidate.

**Technical Definition**
`srcset` is a content attribute that accepts a comma-separated list of image candidate strings. Each candidate consists of a URL followed by either a **width descriptor** (`w`, a positive integer) or a **pixel-density descriptor** (`x`, a positive floating-point number). Width descriptors are used for resolution switching and require `sizes`; pixel-density descriptors are used for fixed-size images where the rendered size does not change. Mixing `w` and `x` descriptors in the same `srcset` is invalid.

**Beginner-Friendly Explanation**
`srcset` is a menu of image files. You list each file and how wide it is (or how dense it is), and the browser picks the one that fits the screen.

---

### Purposes

- **To provide the browser with multiple image candidates** for a single `<img>`.
- **To enable resolution switching** by declaring intrinsic widths with `w` descriptors.
- **To support high-density displays** by declaring pixel-density ratios with `x` descriptors.

---

### Syntax Rules and Structure

#### Complete General Syntax (Width Descriptors)

```html
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Description">
```

#### Complete General Syntax (Pixel-Density Descriptors)

```html
<img
  src="photo-640.jpg"
  srcset="photo-320.jpg, photo-480.jpg 1.5x, photo-640.jpg 2x"
  alt="Description">
```

**Component breakdown**

| Descriptor | Syntax | Requires `sizes`? | Use case |
|---|---|---|---|
| Width (`w`) | `photo-800.jpg 800w` | **Yes** | Variable-size images. |
| Pixel density (`x`) | `photo-640.jpg 2x` | No | Fixed-size images (icons, logos). |

#### Syntax Rules

1. **Each candidate is separated by a comma**.
2. **Each candidate has a URL and an optional descriptor**; if no descriptor is present, `1x` is assumed.
3. **Width descriptors must be positive integers followed by `w`**; pixel-density descriptors must be positive numbers followed by `x`.
4. **Do not mix `w` and `x` descriptors** in the same `srcset`.
5. **`sizes` is mandatory when `w` descriptors are used**; otherwise the `srcset` is ignored.

#### Constraints and Limitations

- **Without `sizes`, the browser assumes `100vw`** and may download the largest candidate.
- **The `src` attribute is still required** as a fallback for browsers that do not support `srcset`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Width Descriptors with `sizes`

**Setup Guide**: Save as `srcset-width.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Srcset Width Descriptors</title>
</head>
<body>
  <img
    src="elva-fairy-800w.jpg"
    srcset="elva-fairy-480w.jpg 480w, elva-fairy-800w.jpg 800w"
    sizes="(max-width: 600px) 480px, 800px"
    alt="Elva dressed as a fairy"
    width="800"
    height="600">
</body>
</html>
```

**Expected output**: On a narrow screen, the browser may choose `elva-fairy-480w.jpg`; on a wider screen, `elva-fairy-800w.jpg`. The `sizes` attribute tells the browser the image will be 480px wide on small screens and 800px wide on larger screens.

**Why this output occurs**: The browser calculates the effective density of each candidate by dividing its width descriptor by the source size from `sizes`, then selects the most appropriate candidate for the device's DPR.

---

#### Example 2: Pixel-Density Descriptors for a Fixed-Size Image

**Setup Guide**: Save as `srcset-density.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Srcset Density Descriptors</title>
</head>
<body>
  <img
    src="logo-320.png"
    srcset="logo-320.png, logo-480.png 1.5x, logo-640.png 2x"
    alt="Company logo"
    width="320"
    height="80">
</body>
</html>
```

**Expected output**: On a standard-density screen, `logo-320.png` is used; on a 1.5x screen, `logo-480.png`; on a 2x retina screen, `logo-640.png`. The rendered size stays 320px wide.

**Why this output occurs**: Pixel-density descriptors tell the browser how many physical pixels each candidate provides per CSS pixel. The browser picks the candidate that matches the device's DPR.

---

### Real-World Cases

1. **Content images**: Width descriptors with `sizes` are the standard pattern for article images.
2. **Logos and icons**: Pixel-density descriptors ensure crisp rendering on retina displays without changing the layout size.

---

### References

- MDN Web Docs — `srcset` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#srcset
- web.dev — Descriptive syntax — https://web.dev/learn/images/descriptive

---

## 3. `sizes` — The Rendered-Size Hints

### Definitions

**Core Definition**
`sizes` defines a set of media conditions and indicates what image size would be best to choose when each condition is true.

**Technical Definition**
`sizes` is a content attribute that accepts a comma-separated list of source size descriptors. Each descriptor is either a media condition followed by a length, or a bare length for the default case. The browser uses the first matching media condition to determine the image's rendered size in CSS pixels, then divides each `srcset` candidate's width descriptor by that size to compute the effective pixel density. When `sizes` is omitted and `srcset` uses `w` descriptors, the browser assumes `100vw`.

**Beginner-Friendly Explanation**
`sizes` tells the browser how wide the image will appear on different screen sizes. The browser uses this information to decide which `srcset` file is the right size to download.

---

### Purposes

- **To inform the browser's candidate selection** when `srcset` uses width descriptors.
- **To prevent downloading oversized images** on small screens.
- **To calculate effective pixel density** for each candidate.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 100vw, (max-width: 900px) 50vw, 33vw"
  src="photo-800.jpg"
  alt="Description">
```

**Component breakdown**

| Component | Description |
|---|---|
| `(max-width: 600px) 100vw` | Media condition and rendered size. |
| `(max-width: 900px) 50vw` | Second condition. |
| `33vw` | Default size if no condition matches. |

#### Syntax Rules

1. **Each descriptor is a media condition and a length**, or a bare length for the default.
2. **Use `vw` units for viewport-relative sizes** and `px` for fixed sizes.
3. **The first matching condition wins**; order matters.
4. **Always provide `sizes` when using `w` descriptors**; otherwise the `srcset` is ignored.

#### Constraints and Limitations

- **`sizes` does not control the image's rendered size**; it only tells the browser how large the image will be. CSS controls the actual layout.
- **Incorrect `sizes` values lead to suboptimal downloads**, not broken layouts.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Responsive `sizes` with Breakpoints

**Setup Guide**: Save as `sizes-breakpoints.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sizes Breakpoints</title>
</head>
<body>
  <img
    src="photo-800.jpg"
    srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, (max-width: 900px) 50vw, 33vw"
    alt="A mountain landscape"
    width="1200"
    height="800">
</body>
</html>
```

**Expected output**: On a 375px-wide phone, the image occupies 100vw (375px), so the browser may choose `photo-400.jpg`. On a 1200px-wide desktop, the image occupies 33vw (396px), so the browser may also choose `photo-400.jpg` rather than downloading the larger `photo-1200.jpg`.

**Why this output occurs**: The `sizes` attribute tells the browser the image's rendered width under each condition, so it can calculate the effective density of each candidate and choose the smallest adequate one.

---

#### Example 2: `sizes` with a Fixed Width

**Setup Guide**: Save as `sizes-fixed.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Fixed Sizes</title>
</head>
<body>
  <img
    src="thumbnail-200.jpg"
    srcset="thumbnail-200.jpg 200w, thumbnail-400.jpg 400w"
    sizes="200px"
    alt="A thumbnail"
    width="200"
    height="150">
</body>
</html>
```

**Expected output**: On a 1x screen, `thumbnail-200.jpg` is used; on a 2x screen, `thumbnail-400.jpg` is used.

**Why this output occurs**: The `sizes="200px"` tells the browser the image is always 200px wide, so a 2x screen needs a 400w candidate to achieve the required density.

---

### Real-World Cases

1. **Article images**: `sizes` with breakpoints matching the content column width.
2. **Grid layouts**: `sizes` reflecting the number of columns the image spans at each breakpoint.

---

### References

- MDN Web Docs — `sizes` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#sizes
- web.dev — Descriptive syntax: `sizes` — https://web.dev/learn/images/descriptive

---

## 4. `<picture>` — The Art-Direction and Format Container

### Definitions

**Core Definition**
The `<picture>` element contains zero or more `<source>` elements and exactly one `<img>` element, offering alternative versions of an image for different display or device scenarios.

**Technical Definition**
`<picture>` belongs to the **flow content**, **phrasing content**, **embedded content**, and **palpable content** categories. Its content model is zero or more `<source>` elements followed by exactly one `<img>` element. The browser evaluates each `<source>` in order, selecting the first whose `media` condition and `type` are both satisfied; if none match, the `<img>` is used. The `<picture>` element itself does not render anything; the selected image is rendered by the `<img>`.

**Beginner-Friendly Explanation**
`<picture>` is a wrapper that lets you give the browser several options and let it choose. You use it when you want to show different crops on different screens (art direction) or when you want to offer modern image formats with fallbacks (format switching).

---

### Purposes

- **To implement art direction** — serving different crops or compositions at different breakpoints.
- **To implement format negotiation** — offering AVIF, WebP, and JPEG with appropriate fallbacks.
- **To provide explicit control** over which resource is used under which conditions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<picture>
  <source media="(max-width: 640px)" srcset="hero-mobile.avif" type="image/avif">
  <source media="(min-width: 641px)" srcset="hero-desktop.avif" type="image/avif">
  <img src="hero-desktop.jpg" alt="Description" width="1200" height="630">
</picture>
```

**Component breakdown**

| Component | Role |
|---|---|
| `<picture>` | Container. |
| `<source>` | Candidate with `media`, `type`, and `srcset`. |
| `<img>` | **Required.** Fallback and rendered element. |

#### Syntax Rules

1. **`<picture>` must contain exactly one `<img>`**, typically as the last child.
2. **`<source>` elements must precede the `<img>`**.
3. **The browser evaluates `<source>` elements in order** and selects the first matching one.
4. **The `<img>` must have `src` and `alt`** and should have `width` and `height`.

#### Constraints and Limitations

- **`<picture>` does not render anything itself**; all rendering is done by the selected image, ultimately the `<img>`.
- **`<picture>` is not a replacement for `srcset` and `sizes`**; use it only when you need art direction or format switching.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Art Direction with Media Queries

**Setup Guide**: Save as `picture-art.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Picture Art Direction</title>
</head>
<body>
  <picture>
    <source media="(max-width: 799px)" srcset="elva-480w-close-portrait.jpg">
    <source media="(min-width: 800px)" srcset="elva-800w.jpg">
    <img src="elva-800w.jpg" alt="Elva dressed as a fairy" width="800" height="600">
  </picture>
</body>
</html>
```

**Expected output**: On screens up to 799px, the close portrait crop is shown; on wider screens, the wider image is shown.

**Why this output occurs**: The browser evaluates the `media` attributes and selects the first matching `<source>`.

---

#### Example 2: Format Switching with Fallbacks

**Setup Guide**: Save as `picture-format.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Picture Format Switching</title>
</head>
<body>
  <picture>
    <source type="image/avif" srcset="photo.avif">
    <source type="image/webp" srcset="photo.webp">
    <img src="photo.jpg" alt="A city skyline at dusk" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: Browsers that support AVIF load `photo.avif`; browsers that support WebP but not AVIF load `photo.webp`; browsers that support neither load `photo.jpg`.

**Why this output occurs**: The browser selects the first `<source>` whose `type` it supports, falling back to the `<img>` if none match.

---

### Real-World Cases

1. **Responsive hero banners**: Different crops for mobile and desktop.
2. **Modern format adoption**: Serving AVIF/WebP with JPEG fallbacks.
3. **Design-specific layouts**: Different image compositions for portrait and landscape orientations.

---

### References

- MDN Web Docs — `<picture>`: The Picture element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/picture
- web.dev — Prescriptive syntax — https://web.dev/learn/images/prescriptive

---

## 5. `<source>` — The Candidate Resource Element

### Definitions

**Core Definition**
The `<source>` element specifies multiple media resources for `<picture>`, `<video>`, and `<audio>`. It is a void element.

**Technical Definition**
`<source>` belongs to the **flow content**, **phrasing content**, and **embedded content** categories when used inside `<picture>`, `<video>`, or `<audio>`. Inside `<picture>`, it supports `media` (media query), `type` (MIME type), `srcset` (candidate list), and `sizes` (rendered-size hints). The browser evaluates each `<source>` in order and selects the first whose `media` and `type` conditions are satisfied.

**Beginner-Friendly Explanation**
`<source>` is one option inside a `<picture>`. You give it a file, a media condition, or a type, and the browser decides whether to use it.

---

### Purposes

- **To provide candidate resources** for art direction and format switching.
- **To specify media conditions** with `media`.
- **To specify MIME types** with `type` so the browser can skip unsupported formats.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<source media="(max-width: 640px)" srcset="image-mobile.jpg" type="image/jpeg">
```

**Component breakdown**

| Attribute | Role |
|---|---|
| `media` | Media query that determines when the source applies. |
| `type` | MIME type of the resource. |
| `srcset` | Candidate image URLs with descriptors. |
| `sizes` | Rendered-size hints (optional, for `srcset` inside `<source>`). |

#### Syntax Rules

1. **`<source>` is a void element**; no end tag, no children.
2. **Inside `<picture>`, the browser selects the first matching `<source>`**.
3. **`type` is evaluated before `media`** in some implementations; both must match for the source to be used.
4. **`<source>` must precede the `<img>`** inside `<picture>`.

#### Constraints and Limitations

- **`<source>` cannot be used as a standalone element** outside a `<picture>`, `<video>`, or `<audio>`.
- **The order of `<source>` elements matters**; the browser picks the first compatible one.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple Sources with Media and Type

**Setup Guide**: Save as `source-multiple.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Source Multiple</title>
</head>
<body>
  <picture>
    <source media="(max-width: 640px)" srcset="hero-mobile.avif" type="image/avif">
    <source media="(max-width: 640px)" srcset="hero-mobile.webp" type="image/webp">
    <source media="(min-width: 641px)" srcset="hero-desktop.avif" type="image/avif">
    <img src="hero-desktop.jpg" alt="A mountain landscape" width="1200" height="630">
  </picture>
</body>
</html>
```

**Expected output**: On a narrow screen with AVIF support, `hero-mobile.avif` is used; on a narrow screen with WebP but not AVIF, `hero-mobile.webp` is used; on a wide screen with AVIF support, `hero-desktop.avif` is used; otherwise, `hero-desktop.jpg` is used.

**Why this output occurs**: The browser evaluates each `<source>` in order, checking both `media` and `type`. The first source that satisfies both is selected.

---

### Real-World Cases

1. **Multi-format responsive images**: Offering AVIF, WebP, and JPEG at multiple breakpoints.
2. **Art direction with format negotiation**: Combining different crops with modern format support.

---

### References

- MDN Web Docs — `<source>`: The Media or Image Source element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/source

---

## 6. Art Direction — Different Crops for Different Layouts

### Definitions

**Core Definition**
Art direction is the practice of serving different image crops or compositions to different devices or viewport sizes, because the same image does not work well at every aspect ratio or scale.

**Technical Definition**
Art direction is implemented with `<picture>` and `<source media>`. Each `<source>` specifies a media condition and a `srcset` candidate. The browser selects the first matching source, allowing the developer to show a tight portrait crop on mobile and a wide landscape crop on desktop, or a simplified image on small screens where fine details would be lost. Art direction is distinct from resolution switching, which serves the same image at different sizes.

**Beginner-Friendly Explanation**
Art direction means showing a different version of the picture on different screens. Maybe on mobile you show a close-up of a face, and on desktop you show the whole scene. `<picture>` lets you do that.

---

### Purposes

- **To show an appropriate composition** for each layout.
- **To avoid losing important details** when an image is scaled down.
- **To optimise the visual impact** of hero images across breakpoints.
- **To reduce visual clutter** on small screens by serving a simpler crop.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<picture>
  <source media="(max-width: 640px)" srcset="crop-mobile.jpg">
  <source media="(min-width: 641px)" srcset="crop-desktop.jpg">
  <img src="crop-desktop.jpg" alt="Description" width="1200" height="630">
</picture>
```

**Component breakdown**

| Component | Role |
|---|---|
| `<source media="…">` | Media condition determining when the crop applies. |
| `srcset` | The crop file. |
| `<img>` | Fallback and rendered element. |

#### Syntax Rules

1. **Each crop gets its own `<source>` with a `media` condition**.
2. **The `<img>` provides the fallback** for browsers without `<picture>` support.
3. **Use `width` and `height` on the `<img>`** to reserve space and prevent CLS.

#### Constraints and Limitations

- **Art direction is a design decision**; the browser cannot infer the right crop on its own.
- **Too many art-direction breakpoints** increase maintenance cost.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mobile Close-Up vs Desktop Wide Shot

**Setup Guide**: Save as `art-direction.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Art Direction</title>
</head>
<body>
  <picture>
    <source media="(max-width: 640px)" srcset="hero-mobile-close.jpg">
    <source media="(min-width: 641px)" srcset="hero-desktop-wide.jpg">
    <img src="hero-desktop-wide.jpg" alt="A mountain landscape" width="1200" height="630">
  </picture>
</body>
</html>
```

**Expected output**: On mobile, the close-up crop is shown; on desktop, the wide landscape crop is shown.

**Why this output occurs**: The `media` attributes on the `<source>` elements tell the browser which crop to use at each breakpoint.

---

### Real-World Cases

1. **News hero images**: A tight portrait crop on mobile and a wide scene on desktop.
2. **Product photography**: A detail shot on small screens and a full product view on larger screens.

---

### References

- MDN Web Docs — Responsive images: Art direction — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images#art_direction
- web.dev — Prescriptive syntax: art direction — https://web.dev/learn/images/prescriptive

---

## 7. Resolution Switching — Different Sizes of the Same Image

### Definitions

**Core Definition**
Resolution switching is the practice of serving the same image at different resolutions or sizes, so that smaller devices download smaller files and high-density screens receive higher-resolution versions.

**Technical Definition**
Resolution switching is implemented with `srcset` and `sizes` on the `<img>` element. The `srcset` attribute lists candidates with width (`w`) or pixel-density (`x`) descriptors, and `sizes` provides the rendered-size hints that allow the browser to calculate effective pixel density and choose the most appropriate candidate. This is the **descriptive syntax**: the developer describes the candidates and lets the browser decide.

**Beginner-Friendly Explanation**
Resolution switching means giving the browser several sizes of the same picture and letting it pick the one that fits the screen. A phone gets a small file; a retina laptop gets a larger one.

---

### Purposes

- **To reduce bandwidth waste** on mobile devices.
- **To improve visual quality** on high-density screens.
- **To improve loading performance** by downloading only the necessary bytes.
- **To improve Core Web Vitals** by reducing image payload.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Description"
  width="1200"
  height="800">
```

**Component breakdown**

| Component | Role |
|---|---|
| `srcset` | Candidate list with `w` descriptors. |
| `sizes` | Rendered-size hints. |
| `src` | Fallback. |

#### Syntax Rules

1. **Use `w` descriptors for variable-size images**.
2. **Always provide `sizes`** when using `w` descriptors.
3. **Provide 3–4 breakpoints** (e.g. 400w, 800w, 1200w, 1600w) for most images.
4. **Use `x` descriptors for fixed-size images** where the rendered size does not change.

#### Constraints and Limitations

- **The browser may not choose the candidate you expect**; it uses network conditions and DPR heuristics.
- **Incorrect `sizes` values lead to suboptimal downloads**, not broken layouts.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Resolution Switching with Width Descriptors

**Setup Guide**: Save as `resolution-width.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Resolution Switching</title>
</head>
<body>
  <img
    src="photo-800.jpg"
    srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="A mountain landscape"
    width="1200"
    height="800">
</body>
</html>
```

**Expected output**: On a 375px phone, the browser may choose `photo-400.jpg`; on a 1440px desktop, it may choose `photo-800.jpg` or `photo-1200.jpg` depending on the DPR.

**Why this output occurs**: The `sizes` attribute tells the browser the image occupies 100vw on small screens and 50vw on larger screens, and `srcset` provides candidates at those widths.

---

#### Example 2: Resolution Switching with Pixel-Density Descriptors

**Setup Guide**: Save as `resolution-density.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Density Switching</title>
</head>
<body>
  <img
    src="icon-64.png"
    srcset="icon-64.png 1x, icon-128.png 2x, icon-192.png 3x"
    alt="A settings icon"
    width="64"
    height="64">
</body>
</html>
```

**Expected output**: On a 1x screen, `icon-64.png` is used; on a 2x screen, `icon-128.png`; on a 3x screen, `icon-192.png`. The rendered size stays 64px.

**Why this output occurs**: Pixel-density descriptors tell the browser the density of each candidate, and it selects the one matching the device's DPR.

---

### Real-World Cases

1. **Article images**: Width descriptors with `sizes` matching the content column.
2. **Retina assets**: Density descriptors for logos and icons.

---

### References

- MDN Web Docs — Responsive images: Resolution switching — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images#resolution_switching_different_sizes
- web.dev — Descriptive syntax — https://web.dev/learn/images/descriptive

---

## 8. Format Switching — Modern Formats with Fallbacks

### Definitions

**Core Definition**
Format switching is the practice of serving modern image formats (AVIF, WebP, JPEG XL) to browsers that support them, while retaining legacy formats (JPEG, PNG) as fallbacks for browsers that do not.

**Technical Definition**
Format switching is implemented with `<picture>` and `<source type>`. Each `<source>` declares a MIME type and a `srcset` candidate; the browser selects the first source whose `type` it supports. The `<img>` provides the universal fallback. Modern formats can deliver significant savings: AVIF can save approximately 70% compared to JPEG, and WebP can save approximately 50%.

**Beginner-Friendly Explanation**
Format switching means giving the browser newer, smaller image formats if it can handle them, and older formats if it cannot. AVIF is very small but not every browser supports it, so you provide a JPEG fallback.

---

### Purposes

- **To reduce file size** with modern compression.
- **To improve loading performance** without sacrificing visual quality.
- **To maintain compatibility** with older browsers through fallbacks.
- **To future-proof** image delivery as new formats emerge.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<picture>
  <source type="image/avif" srcset="photo.avif">
  <source type="image/webp" srcset="photo.webp">
  <img src="photo.jpg" alt="Description" width="1200" height="800">
</picture>
```

**Component breakdown**

| Component | Role |
|---|---|
| `<source type="image/avif">` | Modern format candidate. |
| `<source type="image/webp">` | Intermediate format candidate. |
| `<img src="photo.jpg">` | Universal fallback. |

#### Syntax Rules

1. **Order sources from most modern to most compatible** (AVIF → WebP → JPEG).
2. **Include a `type` attribute on every `<source>`** so the browser can skip unsupported formats.
3. **The `<img>` must have a `src`** pointing to the fallback format.
4. **Combine format switching with art direction** when needed by adding `media` to the `<source>` elements.

#### Constraints and Limitations

- **Not all browsers support every format**; the fallback chain handles this.
- **AVIF encoding can be slower** than JPEG; consider build-time generation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: AVIF, WebP, and JPEG

**Setup Guide**: Save as `format-switch.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Format Switching</title>
</head>
<body>
  <picture>
    <source type="image/avif" srcset="photo.avif">
    <source type="image/webp" srcset="photo.webp">
    <img src="photo.jpg" alt="A city skyline at dusk" width="1200" height="800">
  </picture>
</body>
</html>
```

**Expected output**: Modern browsers load `photo.avif`; browsers that support WebP but not AVIF load `photo.webp`; browsers that support neither load `photo.jpg`.

**Why this output occurs**: The browser selects the first `<source>` whose `type` it supports, falling back to the `<img>` if none match.

---

#### Example 2: Format Switching with Art Direction

**Setup Guide**: Save as `format-art.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Format and Art Direction</title>
</head>
<body>
  <picture>
    <source media="(max-width: 640px)" type="image/avif" srcset="hero-mobile.avif">
    <source media="(max-width: 640px)" type="image/webp" srcset="hero-mobile.webp">
    <source media="(min-width: 641px)" type="image/avif" srcset="hero-desktop.avif">
    <img src="hero-desktop.jpg" alt="A mountain landscape" width="1200" height="630">
  </picture>
</body>
</html>
```

**Expected output**: The browser combines media-query evaluation with format support to select the best combination of crop and format.

**Why this output occurs**: Each `<source>` specifies both a `media` condition and a `type`; the browser selects the first source where both are satisfied.

---

### Real-World Cases

1. **Performance-focused sites**: Serving AVIF to modern browsers and JPEG to legacy ones.
2. **E-commerce**: Reducing product image payload without sacrificing quality.

---

### References

- Chrome for Developers — Performance best practices: modern formats — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance
- web.dev — Prescriptive syntax: format switching — https://web.dev/learn/images/prescriptive

---

## 9. Responsive Image Best Practices

### Definitions

**Core Definition**
Responsive image best practices are the consolidated guidelines for implementing `srcset`, `sizes`, `<picture>`, and `<source>` in a way that is performant, accessible, maintainable, and correct.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, Core Web Vitals guidance from Chrome for Developers, and MDN’s responsive image documentation. They encompass mandatory pairing of `srcset` and `sizes`, appropriate use of art direction versus resolution switching, modern format adoption with fallbacks, dimension and `alt` discipline, and performance-conscious loading attributes.

**Beginner-Friendly Explanation**
Best practices help you use responsive images correctly: always pair `srcset` with `sizes`, use `<picture>` only for art direction or format switching, set `width` and `height`, provide `alt`, and load images efficiently.

---

### Purposes

- **To ensure the browser can make an informed selection** by providing both candidates and size hints.
- **To avoid bandwidth waste** by preventing oversized downloads.
- **To prevent layout shift** with explicit dimensions.
- **To maintain accessibility** with `alt` text on the fallback image.
- **To improve Core Web Vitals** by combining responsive delivery with loading strategies.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Always pair `srcset` with `sizes`** | Without `sizes`, the browser assumes `100vw` and may download the largest image. |
| **Use 3–4 breakpoints** | 400w, 800w, 1200w, 1600w cover most images. |
| **Use `<picture>` for art direction** | Different crops or compositions at different breakpoints. |
| **Use `<picture>` for format switching** | AVIF → WebP → JPEG fallback chain. |
| **Always set `width` and `height`** | Prevents CLS by reserving space. |
| **Always provide `alt`** | Accessibility is independent of which candidate is chosen. |
| **Use `loading="lazy"` below the fold** | Do not lazy-load the LCP image. |
| **Use `fetchpriority="high"` on the LCP image** | Prioritise the hero image. |

#### Syntax Rules

1. **Descriptive syntax first**: Use `srcset` and `sizes` for most images; reserve `<picture>` for art direction and format switching.
2. **Pairing discipline**: `srcset` with `w` descriptors requires `sizes`; omitting `sizes` is a common mistake.
3. **Dimension discipline**: Always set `width` and `height` on the `<img>`.
4. **Accessibility discipline**: Always provide `alt` text on the `<img>`.

#### Constraints and Limitations

- **Overusing `<picture>`** for simple resolution switching adds unnecessary complexity; `srcset` and `sizes` are simpler and sufficient.
- **Too many breakpoints** increase build and maintenance cost without meaningful benefit.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fully Optimised Responsive Image

**Setup Guide**: Save as `best-practice-responsive.html`. Provide AVIF, WebP, and JPEG at multiple widths.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Best Practice Responsive Image</title>
</head>
<body>
  <picture>
    <!-- Modern formats with resolution switching -->
    <source
      type="image/avif"
      srcset="hero-400.avif 400w, hero-800.avif 800w, hero-1200.avif 1200w"
      sizes="(max-width: 600px) 100vw, 50vw">
    <source
      type="image/webp"
      srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1200.webp 1200w"
      sizes="(max-width: 600px) 100vw, 50vw">

    <!-- Fallback with dimensions and priority -->
    <img
      src="hero-800.jpg"
      srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1200.jpg 1200w"
      sizes="(max-width: 600px) 100vw, 50vw"
      alt="A mountain landscape with a lake in the foreground"
      width="1200"
      height="800"
      fetchpriority="high">
  </picture>
</body>
</html>
```

**Expected output**: Modern browsers load an AVIF candidate chosen by `srcset`/`sizes`; browsers that support WebP but not AVIF load a WebP candidate; browsers that support neither load a JPEG candidate. The LCP image is prioritised with `fetchpriority="high"`, and layout space is reserved with `width`/`height`.

**Why this output occurs**: The `<source>` elements provide format negotiation, and the `srcset`/`sizes` pairs on each source provide resolution switching. The `<img>` supplies the fallback with the same responsive behaviour and the required dimensions and `alt` text.

---

#### Example 2: Common Mistakes to Avoid

**Setup Guide**: Save as `responsive-mistakes.html`.

```html
<!-- MISTAKE 1: srcset without sizes -->
<img srcset="photo-400.jpg 400w, photo-800.jpg 800w" src="photo-800.jpg" alt="Photo">

<!-- MISTAKE 2: no dimensions -->
<img src="photo.jpg" alt="Photo" srcset="photo-400.jpg 400w, photo-800.jpg 800w" sizes="100vw">

<!-- MISTAKE 3: lazy-loading the LCP image -->
<img src="hero.jpg" alt="Hero" loading="lazy" width="1200" height="800">

<!-- MISTAKE 4: <picture> for simple resolution switching -->
<picture>
  <source media="(max-width: 600px)" srcset="photo-400.jpg">
  <source media="(min-width: 601px)" srcset="photo-800.jpg">
  <img src="photo-800.jpg" alt="Photo">
</picture>
```

**Expected output**: Each mistake degrades performance or maintainability: `srcset` without `sizes` downloads the largest image; missing dimensions cause layout shift; lazy-loading the LCP image delays paint; and `<picture>` for simple switching is unnecessarily verbose compared with `srcset` and `sizes`.

**Why this output occurs**: These are the most common responsive-image mistakes identified in performance and accessibility audits.

---

### Real-World Cases

1. **E-commerce**: Responsive product images with format switching and resolution switching.
2. **News sites**: Hero images with `fetchpriority="high"`, `width`/`height`, and AVIF/WebP fallbacks.
3. **Documentation**: Diagrams with `srcset` and `sizes` to avoid oversized downloads.

---

### References

- Chrome for Developers — Performance best practices — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance
- web.dev — Serve responsive images — https://web.dev/articles/serve-responsive-images
- MDN Web Docs — Responsive images — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images

---

## Summary Table of Responsive Image Concepts

| Concept | Mechanism | Problem Solved | Key Constraint |
|---|---|---|---|
| **Responsive images** | `srcset`/`sizes` or `<picture>`/`<source>` | Right image for every device | Browser chooses the candidate |
| **`srcset`** | Candidate list with `w` or `x` descriptors | Provides options | `w` descriptors require `sizes` |
| **`sizes`** | Media conditions + rendered sizes | Informs browser selection | Must match actual layout |
| **`<picture>`** | `<source>` elements + `<img>` | Art direction and format switching | Exactly one `<img>` required |
| **`<source>`** | `media`, `type`, `srcset` | Candidate resource | Evaluated in order |
| **Art direction** | `<picture>` + `<source media>` | Different crops for different layouts | Design decision |
| **Resolution switching** | `srcset` + `sizes` | Different sizes/resolutions | Descriptive syntax |
| **Format switching** | `<picture>` + `<source type>` | Modern formats with fallbacks | Order sources modern → legacy |
| **Best practices** | Combined | Performance, accessibility, maintainability | Pair `srcset`/`sizes`; set dimensions; provide `alt` |

---

## Final Notes

- **`srcset` and `sizes` are a package deal**: Using `w` descriptors without `sizes` causes the browser to assume `100vw` and may download the largest candidate.
- **`<picture>` is for art direction and format switching**: For simple resolution switching, `srcset` and `sizes` on `<img>` are sufficient and simpler.
- **Dimensions are mandatory for layout stability**: Always set `width` and `height` to prevent CLS.
- **`alt` text is non-negotiable**: The selected candidate does not change the image’s accessible name.
- **LCP images must not be lazy-loaded**: Use `fetchpriority="high"` and avoid `loading="lazy"` on above-the-fold hero images.
- **Modern formats deliver real savings**: AVIF and WebP reduce payload significantly compared with JPEG, with `<picture>` providing the fallback chain.

---

This cheat sheet has covered responsive images — the discipline, `srcset`, `sizes`, `<picture>`, `<source>`, art direction, resolution switching, format switching, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate responsive image markup against the WHATWG HTML Living Standard, MDN’s responsive image guide, and Core Web Vitals performance guidance.