# Image Attributes — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Image attributes are the HTML attributes that control how an `<img>` element fetches, describes, sizes, and renders an image resource. They span the essential (`src`, `alt`), the layout-stabilising (`width`, `height`), the responsive (`srcset`, `sizes`), the performance-oriented (`loading`, `decoding`, `fetchpriority`), and the security/advanced (`ismap`, `usemap`, `crossorigin`, `referrerpolicy`).

**Technical Definition**
In the WHATWG HTML Living Standard, `<img>` is a void element whose content model is nothing. Its attributes are divided into several functional groups: **required content attributes** (`src`), **alternative-text attributes** (`alt`), **dimension attributes** (`width`, `height`), **responsive-image attributes** (`srcset`, `sizes`), **fetch-behaviour attributes** (`loading`, `decoding`, `fetchpriority`), **image-map attributes** (`ismap`, `usemap`), and **security/referrer attributes** (`crossorigin`, `referrerpolicy`). The `srcset` attribute supports width descriptors (`w`) and pixel-density descriptors (`x`), and when `w` descriptors are used, the `sizes` attribute must also be present to inform the browser’s resource selection. The `loading` attribute accepts `lazy` and `eager` (the default), while `decoding` accepts `sync`, `async`, and `auto`. The `fetchpriority` attribute is an enumerated hint with `high`, `low`, and `auto` values, and it should be used sparingly.

**Beginner-Friendly Explanation**
Image attributes are the “settings” on a picture tag. `src` says where the picture is, `alt` says what it shows, `width` and `height` say how big it is (so the page does not jump around), `srcset` and `sizes` let you give the browser different versions for phones and desktops, `loading` decides whether to load the image now or wait, `decoding` controls how the image is processed, `fetchpriority` tells the browser which images matter most, and the remaining attributes handle image maps and security.

---

### Key Characteristics

- **Two attributes are non-negotiable**: `src` and `alt` are required for every meaningful image.
- **Dimensions prevent layout shift**: `width` and `height` reserve space and are the most reliable fix for image-induced CLS.
- **Responsive images require both halves**: `srcset` lists candidates; `sizes` tells the browser how much space the image will occupy so it can pick the right candidate.
- **Performance attributes are contextual**: `loading="lazy"` is for below-the-fold images; `fetchpriority="high"` is for the LCP image; `decoding="async"` is a general-purpose default.
- **Advanced attributes serve narrow purposes**: `ismap`/`usemap` for image maps; `crossorigin` for canvas-safe images; `referrerpolicy` for privacy.
- **`fetchpriority` is a hint, not a command**: The browser may override it based on its own heuristics.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with `<img>`, attributes, and the DOM.
2. **URL fundamentals**: understanding of absolute and relative URLs.
3. **Core Web Vitals awareness**: understanding CLS (layout shift) and LCP (loading performance).
4. **Accessibility basics**: knowing that `alt` text is essential for screen-reader users.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Core Web Vitals** | `width`/`height` prevent CLS; `fetchpriority` and `loading` affect LCP. |
| **Accessibility (WCAG)** | `alt` satisfies WCAG 1.1.1; `usemap`/`ismap` require accessible alternatives. |
| **Responsive design** | `srcset` and `sizes` enable resolution switching and art direction. |
| **Security** | `crossorigin` enables CORS-safe images; `referrerpolicy` controls referrer leakage. |
| **Image maps** | `ismap` and `usemap` enable server-side and client-side clickable regions. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`src`** — the image source URL
2. **`alt`** — the textual alternative
3. **`width` / `height`** — intrinsic dimensions
4. **`srcset`** — responsive image candidates
5. **`sizes`** — rendered-size hints for `srcset`
6. **`loading`** — `lazy` vs `eager`
7. **`decoding`** — `sync`, `async`, `auto`
8. **`fetchpriority`** — `high`, `low`, `auto`
9. **`ismap` / `usemap`** — image maps
10. **`crossorigin` / `referrerpolicy`** — security attributes
11. **Image attribute best practices**

---

## 1. `src` — The Image Source URL

### Definitions

**Core Definition**
`src` specifies the URL of the image resource to embed. It is the defining attribute that gives the `<img>` element its content.

**Technical Definition**
`src` is a required content attribute on `<img>`. Its value must be a valid non-empty URL referencing a non-interactive, optionally animated image resource. Relative URLs resolve against the document base URL. The `HTMLImageElement.src` IDL attribute reflects the parsed, absolute URL.

**Beginner-Friendly Explanation**
`src` tells the browser where to find the picture.

---

### Purposes

- **To specify the image resource** that the browser should load and display.
- **To provide the fallback URL** when `srcset` is not used or when no candidate matches.

---

### Syntax Rules and Structure

```html
<img src="photo.jpg" alt="A mountain landscape">
```

| URL form | Example |
|---|---|
| Relative | `photo.jpg` |
| Root-relative | `/images/photo.jpg` |
| Absolute | `https://cdn.example.com/photo.jpg` |

**Rules**: `src` must be present and must contain a valid URL.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `src`

```html
<img src="photo.jpg" alt="A mountain landscape at sunrise">
```

**Expected output**: The image `photo.jpg` is displayed. If it fails to load, the `alt` text appears.

**Why**: The browser resolves `src` against the document URL and fetches the resource.

---

### Real-World Cases

1. **Local images**: `/images/` directories in static sites.
2. **CDN images**: Absolute URLs to content delivery networks.

---

### References

- MDN Web Docs — `<img>`: `src` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#src

---

## 2. `alt` — The Textual Alternative

### Definitions

**Core Definition**
`alt` provides a textual replacement for the image. It is announced by screen readers and displayed when the image cannot load.

**Technical Definition**
`alt` is a content attribute on `<img>` that holds a short text alternative. It is mandatory except when the image is the only content of a link or button and the link text already provides the equivalent, when the image is purely decorative, or when it is part of a group where a sibling’s `alt` already conveys the meaning. Decorative images must use `alt=""`.

**Beginner-Friendly Explanation**
`alt` describes the picture for people who cannot see it.

---

### Purposes

- **To provide an accessible name** for screen-reader users.
- **To display fallback text** when the image fails to load.
- **To satisfy WCAG 1.1.1** (Non-text Content).

---

### Syntax Rules and Structure

```html
<img src="photo.jpg" alt="A cat sleeping on a windowsill">
```

**Rules**: `alt` must be present on every meaningful image; use `alt=""` for decorative images.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Meaningful `alt`

```html
<img src="office.jpg" alt="An open-plan office with standing desks and large windows">
```

**Expected output**: Screen readers announce “An open-plan office with standing desks and large windows.”

**Why**: The `alt` text conveys the meaning of the image.

#### Example 2: Decorative Image

```html
<img src="divider.png" alt="">
```

**Expected output**: The decorative divider is ignored by screen readers.

**Why**: The empty `alt=""` explicitly marks the image as decorative.

---

### References

- W3C — Technique H37: Using `alt` attributes — https://www.w3.org/WAI/WCAG21/Techniques/html/H37
- WCAG — Understanding SC 1.1.1 — https://www.w3.org/WAI/WCAG21/Understanding/non-text-content

---

## 3. `width` and `height` — Intrinsic Dimensions

### Definitions

**Core Definition**
`width` and `height` specify the intrinsic dimensions of the image in CSS pixels, allowing the browser to reserve layout space before the image loads.

**Technical Definition**
Both are dimension attributes on `<img>`. Their values must be valid non-negative integers. The browser uses them to compute the intrinsic aspect ratio and map it to the CSS `aspect-ratio` property, reserving a box in the layout and eliminating image-induced Cumulative Layout Shift (CLS).

**Beginner-Friendly Explanation**
`width` and `height` tell the browser how big the picture will be, so the page does not jump around while the image loads.

---

### Purposes

- **To prevent Cumulative Layout Shift (CLS)** — the most common cause of poor CLS is images without dimensions.
- **To reserve layout space** before the image arrives.
- **To establish the intrinsic aspect ratio** for responsive scaling.

---

### Syntax Rules and Structure

```html
<img src="photo.jpg" alt="A city skyline" width="800" height="600">
```

**Rules**: Always set both `width` and `height` to the image’s intrinsic dimensions. CSS can override the rendered size while preserving the aspect ratio.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Dimensions That Prevent CLS

```html
<img src="hero.jpg" alt="A city skyline at night" width="1200" height="800">
```

**Expected output**: The browser reserves a 1200 × 800 space immediately, so surrounding content does not shift when the image loads.

**Why**: The `width` and `height` attributes provide the aspect ratio the browser needs to reserve space.

---

### Real-World Cases

1. **News articles**: Every content image should have dimensions.
2. **E-commerce**: Product images with fixed aspect ratios.

---

### References

- web.dev — Optimize Cumulative Layout Shift — https://web.dev/articles/optimize-cls
- Chrome for Developers — Performance best practices — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance

---

## 4. `srcset` — Responsive Image Candidates

### Definitions

**Core Definition**
`srcset` provides the browser with a list of image sources and their descriptors, allowing it to choose the most appropriate image for the user’s device and viewport.

**Technical Definition**
`srcset` is a content attribute that accepts a comma-separated list of image candidate strings. Each candidate consists of a URL and an optional descriptor: a **width descriptor** (`w`, a positive integer followed by `w`) or a **pixel-density descriptor** (`x`, a positive floating-point number followed by `x`). Width descriptors allow the browser to perform resolution switching; pixel-density descriptors are for fixed-size images. When `w` descriptors are used, `sizes` must also be present. The `src` attribute serves as a fallback for browsers that do not support `srcset`, and it is ignored by modern browsers when `w` descriptors are present.

**Beginner-Friendly Explanation**
`srcset` is a menu of different image sizes. You list the small, medium, and large versions, and the browser picks the best one for the screen it is on.

---

### Purposes

- **To implement resolution switching** — serving smaller images to small screens and larger images to large or high-DPI screens.
- **To save bandwidth** — a 2400px hero image served to a 375px phone wastes 90%+ of its bytes.
- **To improve loading performance** by downloading only the necessary bytes.

---

### Syntax Rules and Structure

```html
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="A mountain landscape">
```

| Descriptor type | Syntax | Use case |
|---|---|---|
| Width (`w`) | `photo-800.jpg 800w` | Variable-size images; requires `sizes`. |
| Pixel density (`x`) | `photo.jpg 2x` | Fixed-size images (icons, logos). |

**Rules**: Always pair `w` descriptors with `sizes`. Use 3–4 breakpoints (e.g. 400w, 800w, 1200w, 1600w) for most images.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Width Descriptors with `sizes`

```html
<img
  src="swing-400.jpg"
  srcset="swing-200.jpg 200w, swing-400.jpg 400w, swing-800.jpg 800w, swing-1600.jpg 1600w"
  sizes="(max-width: 30em) 100vw, (max-width: 50em) 50vw, calc(33vw - 100px)"
  alt="Kettlebell Swing">
```

**Expected output**: On a narrow screen, the browser may choose `swing-200.jpg` or `swing-400.jpg`; on a wider screen, it may choose `swing-800.jpg` or `swing-1600.jpg`, depending on the `sizes` conditions and the device pixel ratio.

**Why**: The `sizes` attribute tells the browser the image occupies 100vw, 50vw, or roughly 33vw depending on the viewport, and `srcset` provides candidates at those widths.

---

### References

- MDN Web Docs — `srcset` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#srcset
- web.dev — Serve responsive images — https://web.dev/articles/serve-responsive-images

---

## 5. `sizes` — Rendered-Size Hints for `srcset`

### Definitions

**Core Definition**
`sizes` tells the browser how much horizontal space the image will occupy under different viewport conditions, allowing it to select the correct candidate from `srcset`.

**Technical Definition**
`sizes` is a content attribute that accepts a comma-separated list of source size descriptors: a media condition followed by a length, or a bare length for the default. The browser uses the first matching media condition to determine the image’s rendered size, then calculates the effective pixel density and chooses the appropriate `srcset` candidate. When `sizes` is omitted with `w` descriptors, the browser assumes `100vw`.

**Beginner-Friendly Explanation**
`sizes` tells the browser how wide the image will be on different screen sizes, so it knows which `srcset` version to download.

---

### Purposes

- **To inform the browser’s resource selection** when `srcset` uses width descriptors.
- **To prevent downloading oversized images** on small screens.

---

### Syntax Rules and Structure

```html
sizes="(max-width: 600px) 100vw, (max-width: 900px) 50vw, 33vw"
```

**Rules**: Always provide `sizes` alongside `srcset` with `w` descriptors. Use `vw` units for viewport-relative sizes and `px` for fixed sizes.

---

### References

- W3C — The `sizes` attribute — https://www.w3.org/TR/2015/WD-html51-20151008/semantics.html
- web.dev — Responsive images — https://web.dev/learn/design/responsive-images

---

## 6. `loading` — `lazy` vs `eager`

### Definitions

**Core Definition**
`loading` controls whether the browser fetches the image immediately (`eager`, the default) or defers it until it approaches the viewport (`lazy`).

**Technical Definition**
`loading` is an enumerated attribute with two values: `eager` and `lazy`. `eager` loads the image immediately, even if it is off-screen. `lazy` defers the request until the image reaches a browser-defined distance from the viewport, conserving bandwidth and keeping early requests free for the critical path. The attribute must not be used on the LCP image, as lazy-loading it delays paint.

**Beginner-Friendly Explanation**
`loading` says whether to load the picture now (`eager`) or wait until the user scrolls near it (`lazy`).

---

### Purposes

- **To save bandwidth** by deferring off-screen images.
- **To keep early requests free** for critical resources.
- **To improve initial page load performance**.

---

### Syntax Rules and Structure

```html
<!-- LCP image: eager (default) -->
<img src="hero.jpg" alt="Hero" loading="eager">

<!-- Below-the-fold image: lazy -->
<img src="footer-photo.jpg" alt="Footer photo" loading="lazy">
```

**Rules**: Never use `loading="lazy"` on the LCP image. Use `lazy` only for images that are far enough below the fold.

---

### References

- web.dev — Browser-level image lazy loading — https://web.dev/articles/browser-level-image-lazy-loading
- MDN Web Docs — `loading` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#loading

---

## 7. `decoding` — `sync`, `async`, `auto`

### Definitions

**Core Definition**
`decoding` provides a hint about whether image decoding should happen synchronously (`sync`), asynchronously (`async`), or under browser control (`auto`).

**Technical Definition**
`decoding` is an enumerated attribute. **`sync`** decodes the image synchronously, potentially blocking presentation of other content until decoding finishes. **`async`** decodes asynchronously, avoiding delays to other content but potentially causing a flash of unstyled content for images inserted into the viewport. **`auto`** lets the browser decide (the default).

**Beginner-Friendly Explanation**
`decoding` controls whether the browser pauses other work to decode the image (`sync`) or decodes it in the background (`async`).

---

### Purposes

- **To avoid main-thread jank** from decoding large images (`async`).
- **To present the image immediately** when it is critical (`sync`).
- **To let the browser balance performance** (`auto`).

---

### Syntax Rules and Structure

```html
<img src="photo.jpg" alt="A photo" decoding="async">
```

**Rules**: Use `async` as a safe default for most images. Use `sync` only when the image must appear immediately and blocking is acceptable.

---

### References

- MDN Web Docs — `decoding` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#decoding
- W3C — HTML 5.3: Image decoding — https://www.w3.org/TR/html53/embedded-content.html#image-decoding

---

## 8. `fetchpriority` — `high`, `low`, `auto`

### Definitions

**Core Definition**
`fetchpriority` allows the developer to signal that a particular image’s fetch priority is higher or lower than the browser would otherwise infer.

**Technical Definition**
`fetchpriority` is an enumerated attribute with values `high`, `low`, and `auto`. It is a **hint**, not a directive; the browser may override it. It should be used sparingly — typically `high` on the LCP image and `low` on non-critical images that are technically above the fold but hidden (e.g. carousel slides, mega-menu images). For standard-priority images, omit the attribute rather than setting `auto`. It must not be combined with `loading="lazy"` on the same image.

**Beginner-Friendly Explanation**
`fetchpriority` tells the browser which images matter most, so it can load the important ones first.

---

### Purposes

- **To accelerate the LCP image** with `fetchpriority="high"`.
- **To deprioritise hidden or non-critical images** with `fetchpriority="low"`.
- **To prevent bandwidth contention** between critical and non-critical images.

---

### Syntax Rules and Structure

```html
<!-- LCP image: high priority -->
<img src="hero.jpg" alt="Hero" fetchpriority="high" width="1200" height="800">

<!-- Hidden carousel slide: low priority -->
<img src="slide-2.jpg" alt="Slide 2" fetchpriority="low" loading="lazy" width="400" height="300">

<!-- Standard image: omit fetchpriority -->
<img src="content.jpg" alt="Content" width="800" height="600">
```

**Rules**: Apply `fetchpriority="high"` to at most 1–2 critical images. Never combine `fetchpriority="high"` with `loading="lazy"`. Omit the attribute for normal priority.

---

### References

- Chrome for Developers — Optimize image priority — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/optimize-image-priority
- MDN Web Docs — `fetchpriority` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/fetchpriority

---

## 9. `ismap` and `usemap` — Image Maps

### Definitions

**Core Definition**
`ismap` declares a server-side image map; `usemap` associates the image with a client-side `<map>` element.

**Technical Definition**
`ismap` is a boolean attribute that must only be used when the `<img>` is inside an `<a>` element with an `href`. It sends the click coordinates to the server as query parameters. `usemap` is a content attribute that references a `<map>` element by its `name` (or `id`), enabling client-side clickable regions. The two attributes are mutually exclusive in practice; `usemap` is far more common.

**Beginner-Friendly Explanation**
`usemap` makes different parts of one image clickable, each linking somewhere different. `ismap` sends click coordinates to the server, which decides what to do.

---

### Syntax Rules and Structure

```html
<!-- Client-side image map -->
<img src="map.png" alt="Interactive map" usemap="#regions">
<map name="regions">
  <area shape="rect" coords="0,0,100,100" href="/north" alt="North">
</map>

<!-- Server-side image map -->
<a href="/click-handler">
  <img src="map.png" alt="Map" ismap>
</a>
```

**Rules**: `usemap` must reference a valid `<map name>`. `ismap` requires an enclosing `<a href>`.

---

### References

- W3Schools — `usemap` — https://www.w3schools.com/Tags/att_img_usemap.asp
- MDN Web Docs — `<map>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/map

---

## 10. `crossorigin` and `referrerpolicy` — Security Attributes

### Definitions

**Core Definition**
`crossorigin` configures CORS for images used in canvas; `referrerpolicy` controls how much referrer information is sent when fetching the image.

**Technical Definition**
`crossorigin` is a CORS-settings attribute that can be `anonymous` or `use-credentials`; it allows images from third-party sites that allow cross-origin access to be used with `<canvas>` without tainting. `referrerpolicy` accepts values such as `no-referrer`, `origin`, `no-referrer-when-downgrade`, `origin-when-cross-origin`, and `unsafe-url`, and controls the `Referer` header sent with the image request.

**Beginner-Friendly Explanation**
`crossorigin` lets you use images from other sites in a canvas without breaking security. `referrerpolicy` controls how much information your page shares about where the visitor came from.

---

### Syntax Rules and Structure

```html
<!-- CORS-enabled image for canvas use -->
<img src="https://cdn.example.com/photo.jpg" alt="Photo" crossorigin="anonymous">

<!-- No referrer sent with the image request -->
<img src="https://external.example.com/photo.jpg" alt="Photo" referrerpolicy="no-referrer">
```

**Rules**: Use `crossorigin` only when the image will be drawn to canvas. Use `referrerpolicy` to limit referrer leakage to third parties.

---

### References

- W3Schools — `crossorigin` — https://www.w3schools.com/Tags/att_img_crossorigin.asp
- MDN Web Docs — `referrerpolicy` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#referrerpolicy

---

## 11. Image Attribute Best Practices

### Definitions

**Core Definition**
Image attribute best practices are the consolidated guidelines for using `src`, `alt`, `width`, `height`, `srcset`, `sizes`, `loading`, `decoding`, `fetchpriority`, and the advanced attributes to produce accessible, performant, and maintainable images.

**Technical Definition**
Best practices are informed by Core Web Vitals guidance (CLS and LCP), WCAG 1.1.1, and Chrome for Developers’ performance documentation. They combine **accessibility** (`alt`), **layout stability** (`width`/`height`), **responsive delivery** (`srcset`/`sizes`), **fetch control** (`loading`/`fetchpriority`/`decoding`), and **security** (`crossorigin`/`referrerpolicy`).

**Beginner-Friendly Explanation**
Best practices help you use image attributes correctly: describe every image, set its size, provide responsive versions, load efficiently, and use the advanced attributes only when needed.

---

### Purposes

- **To maximise accessibility** through mandatory `alt` text.
- **To eliminate layout shift** by always setting `width` and `height`.
- **To improve LCP** by prioritising the hero image and lazy-loading everything else.
- **To save bandwidth** with responsive `srcset`/`sizes` and modern formats.
- **To maintain security** with `crossorigin` and `referrerpolicy`.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Always provide `alt`** | Meaningful text for meaningful images; `alt=""` for decorative images. |
| **Always set `width` and `height`** | Prevents CLS; the browser reserves space and computes aspect ratio. |
| **Use `srcset` with `sizes`** | Provide 3–4 width candidates; always pair `w` descriptors with `sizes`. |
| **Lazy-load only below the fold** | `loading="lazy"` for off-screen images; never for the LCP image. |
| **Prioritise the LCP image** | `fetchpriority="high"` on the hero; at most 1–2 high-priority images. |
| **Use `decoding="async"`** | Safe default for most images; avoids main-thread jank. |
| **Omit `fetchpriority` for normal images** | Do not set `auto`; simply omit the attribute. |
| **Use `crossorigin` only for canvas** | `crossorigin="anonymous"` when the image will be drawn to canvas. |
| **Limit referrer leakage** | `referrerpolicy="no-referrer"` for third-party images. |

#### Syntax Rules

1. **Accessibility first**: `alt` is mandatory; `alt=""` is explicit, never omit the attribute.
2. **Layout stability**: `width` and `height` are the most reliable CLS fix.
3. **Responsive delivery**: `srcset` without `sizes` is a common mistake; always pair them.
4. **Performance context**: `loading`, `decoding`, and `fetchpriority` are tools for specific situations, not universal defaults.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Fully Optimised Hero and Content Images

```html
<!-- LCP hero: eager, high priority, dimensions, responsive -->
<img
  src="hero-800.jpg"
  srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1200.jpg 1200w"
  sizes="100vw"
  alt="A mountain landscape with a lake at sunrise"
  width="1200"
  height="800"
  fetchpriority="high"
  decoding="async">

<!-- Below-the-fold content image: lazy, responsive -->
<img
  src="content-800.jpg"
  srcset="content-400.jpg 400w, content-800.jpg 800w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="A chart showing quarterly revenue growth"
  width="800"
  height="600"
  loading="lazy"
  decoding="async">
```

**Expected output**: The hero is fetched immediately at high priority, with a candidate chosen by `srcset`/`sizes`. The content image is deferred until it nears the viewport, and both reserve layout space.

**Why**: The hero follows the LCP rule (`fetchpriority="high"`, no `lazy`); the content image follows the below-the-fold rule (`loading="lazy"`); both have `width`/`height` and `alt`; and `decoding="async"` is applied to both.

---

#### Example 2: Anti-Patterns to Avoid

```html
<!-- ANTI-PATTERN: no alt -->
<img src="photo.jpg" width="800" height="600">

<!-- ANTI-PATTERN: no dimensions -->
<img src="photo.jpg" alt="A photo">

<!-- ANTI-PATTERN: srcset without sizes -->
<img src="photo.jpg" srcset="photo-400.jpg 400w, photo-800.jpg 800w" alt="A photo">

<!-- ANTI-PATTERN: lazy-loading the LCP image -->
<img src="hero.jpg" alt="Hero" loading="lazy">

<!-- ANTI-PATTERN: fetchpriority="high" on many images -->
<img src="a.jpg" alt="A" fetchpriority="high">
<img src="b.jpg" alt="B" fetchpriority="high">
<img src="c.jpg" alt="C" fetchpriority="high">
```

**Expected output**: Each anti-pattern degrades accessibility, layout stability, or performance. Missing `alt` fails WCAG; missing dimensions cause CLS; `srcset` without `sizes` downloads oversized images; lazy-loading the LCP image delays paint; overusing `fetchpriority="high"` causes network contention.

**Why**: These are the most common image-attribute mistakes identified in performance and accessibility audits.

---

### References

- Chrome for Developers — Performance best practices — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance
- web.dev — Optimize Cumulative Layout Shift — https://web.dev/articles/optimize-cls
- web.dev — Serve responsive images — https://web.dev/articles/serve-responsive-images
- W3C — Technique H37: Using `alt` attributes — https://www.w3.org/WAI/WCAG21/Techniques/html/H37
- MDN Web Docs — `<img>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img

---

## Summary Table of Image Attributes

| Attribute | Type | Purpose | Key Constraint |
|---|---|---|---|
| **`src`** | URL | Image source | Required; must be a valid non-empty URL. |
| **`alt`** | String | Textual alternative | Required; `alt=""` for decorative images. |
| **`width` / `height`** | Integers | Intrinsic dimensions | Prevents CLS; always set both. |
| **`srcset`** | Candidate list | Responsive candidates | `w` descriptors require `sizes`. |
| **`sizes`** | Source size list | Rendered-size hints | Always pair with `srcset` using `w`. |
| **`loading`** | `lazy` / `eager` | Fetch timing | Never `lazy` on the LCP image. |
| **`decoding`** | `sync` / `async` / `auto` | Decode behaviour | `async` is a safe default. |
| **`fetchpriority`** | `high` / `low` / `auto` | Fetch priority hint | At most 1–2 `high`; omit for normal. |
| **`ismap`** | Boolean | Server-side image map | Requires enclosing `<a href>`. |
| **`usemap`** | ID reference | Client-side image map | Must reference a valid `<map name>`. |
| **`crossorigin`** | CORS setting | Canvas-safe images | Only when drawing to canvas. |
| **`referrerpolicy`** | Referrer policy | Referrer control | Use `no-referrer` for privacy. |

---

## Final Notes

- **`src` and `alt` are non-negotiable**: Every meaningful image needs a source and a textual alternative.
- **`width` and `height` are the CLS fix**: Always set both; the browser reserves space and computes the aspect ratio.
- **`srcset` and `sizes` work together**: Never use `w` descriptors without `sizes`.
- **LCP images must not be lazy-loaded**: Use `fetchpriority="high"` and ensure `loading` is not `lazy`.
- **`fetchpriority` is a scalpel, not a hammer**: Apply it to at most 1–2 critical images; omit it for normal priority.
- **Advanced attributes have narrow use cases**: Use `crossorigin` only for canvas, `referrerpolicy` for privacy, and `ismap`/`usemap` for image maps.

---

This cheat sheet has covered image attributes — `src`, `alt`, `width`, `height`, `srcset`, `sizes`, `loading` (`lazy`/`eager`), `decoding` (`sync`/`async`/`auto`), `fetchpriority`, `ismap`, `usemap`, `crossorigin`, and `referrerpolicy` — plus best practices, with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate image markup against the WHATWG HTML Living Standard, Core Web Vitals guidance, and WCAG 1.1.1.