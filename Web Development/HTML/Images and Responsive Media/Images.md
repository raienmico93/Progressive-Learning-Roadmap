# Image Fundamentals — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Images in HTML are embedded content resources that display visual information within a document. The `<img>` element is the primary mechanism for embedding a single image resource, and its two essential attributes — `src` (the image URL) and `alt` (the textual alternative) — determine what the image represents and how it is conveyed to users who cannot see it.

**Technical Definition**
In the WHATWG HTML Living Standard, `<img>` belongs to the **embedded content**, **phrasing content**, **flow content**, and **palpable content** categories. Its content model is **nothing** (it is a void element), and its tag omission rules prohibit both an end tag and any content. The `src` attribute is required and must contain a valid non-empty URL referencing a non-interactive, optionally animated image resource. The `alt` attribute is required except under specific conditions (e.g. when the image is the only content of a link or button and the link text already provides an equivalent). The `width` and `height` attributes set the intrinsic dimensions, allowing the browser to reserve space before the image loads and thereby mitigate cumulative layout shift (CLS). The `srcset` and `sizes` attributes enable responsive image selection, while `loading="lazy"` defers off-screen image fetching.

**Beginner-Friendly Explanation**
An image in HTML is a picture you place on a web page using the `<img>` tag. You tell the browser where the picture is with `src` (the file path or URL), and you describe what the picture shows with `alt` (the alternative text). The `alt` text is what screen readers announce to blind users, and it is also what appears if the image fails to load. You should also set `width` and `height` so the page layout does not jump around while the image loads.

---

### Key Characteristics

- **Void element**: `<img>` has no end tag and cannot contain other content.
- **Required `src`**: The `src` attribute must be present and must contain a valid URL to an image resource.
- **Required `alt` (with narrow exceptions)**: The `alt` attribute provides the textual alternative for accessibility and fallback; it is mandatory except in specific edge cases.
- **Intrinsic dimensions**: `width` and `height` set the image's intrinsic size, enabling the browser to reserve layout space and prevent CLS.
- **Responsive capabilities**: `srcset` and `sizes` allow the browser to select among multiple image candidates based on viewport size and device pixel density.
- **Performance attributes**: `loading="lazy"` defers off-screen images; `decoding="async"` allows non-blocking image decoding; `fetchpriority` hints at resource priority.
- **Format flexibility**: Browsers support a range of formats (JPEG, PNG, GIF, WebP, AVIF, SVG), and the `<picture>` element enables format negotiation and art direction.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, tags, and attributes.
2. **URL fundamentals**: understanding of absolute and relative URLs.
3. **The concept of accessibility**: awareness that not all users can see images.
4. **Basic CSS layout**: understanding of how images occupy space in the page flow.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | `alt` text satisfies WCAG 1.1.1 (Non-text Content); decorative images need `alt=""`. |
| **Performance (Core Web Vitals)** | `width`/`height` prevent CLS; `loading="lazy"` and `decoding="async"` reduce initial payload; `srcset`/`sizes` avoid oversized downloads. |
| **Responsive design** | `srcset`, `sizes`, and `<picture>` adapt images to viewport and pixel density. |
| **SEO** | Descriptive `alt` text and image filenames contribute to image search and content understanding. |
| **Security** | `crossorigin` and `referrerpolicy` control CORS and referrer behaviour for image requests. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **Images** — the general concept of embedded visual content
2. **`<img>`** — the image embed element
3. **`src`** — the image source URL
4. **`alt`** — the textual alternative
5. **Image best practices** — accessibility, performance, and responsive guidelines

---

## 1. Images — The General Concept

### Definitions

**Core Definition**
In HTML, an image is an embedded content resource that displays visual information — a photograph, illustration, icon, chart, diagram, or any other graphical asset — within a web document.

**Technical Definition**
Images are represented by the `<img>` element when a single image resource is used, or by the `<picture>` element when multiple candidate resources are offered for responsive selection or format negotiation. The HTML standard does not mandate support for specific image formats; user agents may support different formats, with JPEG, PNG, GIF, WebP, AVIF, and SVG being the most commonly supported on the web.

**Beginner-Friendly Explanation**
An image is any picture or graphic on a web page. It could be a photo, a logo, an icon, or a chart. You put images on a page with the `<img>` tag.

---

### Purposes

- **To convey visual information** that text alone cannot express.
- **To illustrate concepts, products, or places**.
- **To provide branding and visual identity** through logos and icons.
- **To support charts, diagrams, and infographics** that communicate data visually.
- **To enhance engagement and readability** on content-heavy pages.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="URL" alt="Description">
```

**Component breakdown**

| Component | Description |
|---|---|
| `<img>` | The void element that embeds the image. |
| `src` | The URL of the image resource. |
| `alt` | The textual alternative for the image. |

#### Syntax Rules

1. **Images are embedded content** and may appear wherever embedded content is expected.
2. **The `<img>` element is a void element**; it has no end tag and no children.
3. **The `src` attribute is required** and must reference a valid image resource.
4. **The `alt` attribute is required** except in specific edge cases (e.g. when the image is the only content of a link and the link text already provides the equivalent).

#### Constraints and Limitations

- **The HTML standard does not list required image formats**; support varies by browser.
- **Images are not guaranteed to load**; network errors, content blocking, and link rot can prevent display.
- **Images alone are not accessible**; they require `alt` text to be usable by screen-reader users.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Basic Image Embed

**Setup Guide**: Save as `img-basic.html` and open in a browser. Provide an image file named `photo.jpg` in the same directory.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Basic Image</title>
</head>
<body>
  <!-- src specifies the image URL; alt provides the textual alternative -->
  <img src="photo.jpg" alt="A mountain landscape at sunrise">
</body>
</html>
```

**Expected output**: The image `photo.jpg` is displayed on the page. If the image fails to load, the text “A mountain landscape at sunrise” is shown instead.

**Why this output occurs**: The `src` attribute loads the image resource, and the browser renders it. The `alt` attribute provides a fallback and an accessible name for assistive technologies.

---

#### Example 2: An Image Inside a Figure

**Setup Guide**: Save as `img-figure.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image in Figure</title>
</head>
<body>
  <figure>
    <img src="chart.png" alt="Bar chart showing revenue growth from Q1 to Q4" width="600" height="400">
    <figcaption>Figure 1: Quarterly revenue growth.</figcaption>
  </figure>
</body>
</html>
```

**Expected output**: The chart image is displayed with the caption “Figure 1: Quarterly revenue growth.” below it.

**Why this output occurs**: The `<figure>` element groups the image with its caption, and the `alt` text describes the chart’s meaning while the `<figcaption>` provides visible context.

---

### Real-World Cases

1. **Article illustrations**: News and blog posts use images to support the written content.
2. **Product photos**: E-commerce sites display product images with descriptive `alt` text.
3. **Icons and logos**: Branding and navigational icons embedded as images.

---

### References

- MDN Web Docs — `<img>`: The Image Embed element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img
- WHATWG — Images — https://html.spec.whatwg.org/dev/images.html

---

## 2. `<img>` — The Image Embed Element

### Definitions

**Core Definition**
The `<img>` element embeds an image into the document. It is a void element whose `src` attribute specifies the image resource and whose `alt` attribute provides the textual alternative.

**Technical Definition**
`<img>` belongs to the **embedded content**, **phrasing content**, **flow content**, and **palpable content** categories. Its content model is **nothing**, and both start and end tags are mandatory (no end tag is used for void elements). Supported attributes include `src` (required), `alt`, `width`, `height`, `srcset`, `sizes`, `loading`, `decoding`, `fetchpriority`, `crossorigin`, `referrerpolicy`, `ismap`, and `usemap`.

**Beginner-Friendly Explanation**
The `<img>` tag is how you put a picture on a web page. You give it a `src` (where the picture is) and an `alt` (what the picture shows), and the browser displays it.

---

### Purposes

- **To embed a single image resource** into the document.
- **To display photographs, illustrations, icons, and graphics**.
- **To provide responsive image candidates** via `srcset` and `sizes`.
- **To support performance optimisations** with `loading`, `decoding`, and `fetchpriority`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img
  src="image.jpg"
  alt="Descriptive text"
  width="800"
  height="600"
  loading="lazy"
  decoding="async">
```

**Component breakdown**

| Attribute | Description |
|---|---|
| `src` | **Required.** URL of the image resource. |
| `alt` | **Required** (with narrow exceptions). Textual alternative. |
| `width` / `height` | Intrinsic dimensions; reserve layout space and prevent CLS. |
| `srcset` / `sizes` | Responsive image candidates and viewport-based selection hints. |
| `loading` | `lazy` or `eager`; controls when the image is fetched. |
| `decoding` | `async`, `sync`, or `auto`; controls image decoding. |
| `fetchpriority` | `high` or `low`; hints at resource priority. |

#### Syntax Rules

1. **`<img>` is a void element**; no end tag and no children.
2. **`src` must be present** and must contain a valid non-empty URL.
3. **`alt` must be present** except when the image is the only content of a link or button and the link text already provides the equivalent.
4. **`width` and `height` should be provided** to reserve space and prevent layout shift.
5. **`loading="lazy"` should not be used on the LCP image** (above-the-fold hero images), as it delays the largest contentful paint.

#### Constraints and Limitations

- **The `ismap` attribute** requires the image to be inside an `<a>` element with an `href`.
- **The `usemap` attribute** must reference a valid `<map>` element.
- **Image formats are not specified by the standard**; support depends on the browser.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Standard Image with Dimensions

**Setup Guide**: Save as `img-dimensions.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image Dimensions</title>
</head>
<body>
  <!-- width and height set the intrinsic size and reserve space -->
  <img src="hero.jpg" alt="A city skyline at night" width="1200" height="800">
</body>
</html>
```

**Expected output**: The image is displayed at 1200 × 800 CSS pixels (or scaled proportionally if constrained by CSS), and the browser reserves that space before the image loads.

**Why this output occurs**: The `width` and `height` attributes set the intrinsic dimensions; the browser uses them to calculate the aspect ratio and reserve layout space, preventing cumulative layout shift (CLS).

---

#### Example 2: Lazy-Loaded Below-the-Fold Image

**Setup Guide**: Save as `img-lazy.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Lazy Image</title>
</head>
<body>
  <p>Scroll down to see the image.</p>
  <div style="height: 2000px;"></div>

  <!-- loading="lazy" defers the fetch until the image nears the viewport -->
  <img src="footer-photo.jpg" alt="A sunset over the ocean" width="800" height="600" loading="lazy" decoding="async">
</body>
</html>
```

**Expected output**: The image is not fetched until the user scrolls near it, saving bandwidth and keeping early requests free for critical content.

**Why this output occurs**: `loading="lazy"` defers the image request until the browser determines it is close to the viewport; `decoding="async"` allows the image to be decoded off the main thread.

---

### Real-World Cases

1. **Hero images**: Above-the-fold images with `fetchpriority="high"` and `loading="eager"`.
2. **Content images**: Article illustrations with `width`/`height` and `alt` text.
3. **Thumbnails**: Small images with `loading="lazy"` and responsive `srcset`.

---

### References

- MDN Web Docs — `<img>`: The Image Embed element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img
- WHATWG — The `img` element — https://html.spec.whatwg.org/dev/images.html#the-img-element
- Chrome for Developers — Performance best practices — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance

---

## 3. `src` — The Image Source URL

### Definitions

**Core Definition**
The `src` attribute specifies the URL of the image resource that the `<img>` element should embed.

**Technical Definition**
`src` is a required content attribute on `<img>`. Its value must be a valid non-empty URL potentially surrounded by spaces, referencing a non-interactive, optionally animated image resource that is neither paged nor scripted. Relative URLs resolve against the document base URL. The `HTMLImageElement.src` IDL attribute reflects the parsed, absolute URL.

**Beginner-Friendly Explanation**
`src` tells the browser where to find the image. It can be a full web address or a path relative to the current page.

---

### Purposes

- **To specify the image resource** to load and display.
- **To enable relative and absolute URL referencing**.
- **To provide the primary image candidate** when `srcset` is not used.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="URL" alt="Description">
```

**Component breakdown**

| Component | Description |
|---|---|
| `src="photo.jpg"` | A relative URL to an image in the same directory. |
| `src="/images/photo.jpg"` | A root-relative URL. |
| `src="https://example.com/photo.jpg"` | An absolute URL. |

#### Syntax Rules

1. **`src` must be present and must contain a valid URL**.
2. **Relative URLs resolve against the document base URL** (which can be changed by `<base href>`).
3. **The URL must reference an image resource**, not a script or a page.

#### Constraints and Limitations

- **Empty `src=""`** may cause the browser to request the current page URL, which is usually not desired.
- **Broken URLs** show the `alt` text and may trigger a network error.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Relative vs Absolute `src`

**Setup Guide**: Save as `src-urls.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image URLs</title>
</head>
<body>
  <!-- Relative URL: same directory -->
  <img src="photo.jpg" alt="Local photo">

  <!-- Root-relative URL: relative to site root -->
  <img src="/assets/photo.jpg" alt="Photo from assets folder">

  <!-- Absolute URL: full web address -->
  <img src="https://cdn.example.com/photo.jpg" alt="Photo from CDN">
</body>
</html>
```

**Expected output**: Each image loads from the resolved URL. The relative URL resolves against the current page’s directory; the root-relative URL resolves against the site root; the absolute URL loads directly from the CDN.

**Why this output occurs**: The browser resolves the `src` value against the document base URL, producing an absolute URL for the request.

---

### Real-World Cases

1. **Local images**: `/images/` directories in static sites.
2. **CDN-hosted images**: Absolute URLs to content delivery networks.
3. **User-uploaded images**: Dynamically generated URLs from storage services.

---

### References

- MDN Web Docs — `<img>`: `src` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#src
- WHATWG — The `src` attribute — https://html.spec.whatwg.org/dev/images.html#attr-img-src

---

## 4. `alt` — The Textual Alternative

### Definitions

**Core Definition**
The `alt` attribute provides a textual replacement for an image. It is what screen readers announce to blind users, and it is what appears if the image fails to load.

**Technical Definition**
`alt` is a content attribute on `<img>` that holds a short text alternative. It is mandatory except under specific conditions: when the image is the only content of a link or button and the link text already provides an equivalent, when the image is purely decorative and accompanied by text that already conveys the information, or when the image is part of a group where the alt text of a sibling image already conveys the meaning. Decorative images should use `alt=""` (empty string) so that screen readers skip them.

**Beginner-Friendly Explanation**
`alt` is the text that describes the image. If the image shows a cat, `alt` might say “a cat sleeping on a windowsill.” If the image is just decoration, you can use `alt=""` so screen readers ignore it.

---

### Purposes

- **To provide an accessible name** for the image to screen readers.
- **To display fallback text** when the image cannot load.
- **To satisfy WCAG 1.1.1 (Non-text Content)**.
- **To contribute to image SEO** through descriptive text.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<img src="photo.jpg" alt="A cat sleeping on a windowsill">
```

**Component breakdown**

| Component | Description |
|---|---|
| `alt="A cat sleeping on a windowsill"` | A concise textual replacement describing the image’s meaning. |

#### Syntax Rules

1. **`alt` must be present** on every meaningful image.
2. **The `alt` text should convey the same meaning** as the image, not necessarily describe its visual characteristics.
3. **Decorative images should use `alt=""`** (empty string).
4. **Do not use `alt` to describe purely visual details** that are irrelevant to the content.
5. **Include words in the image if they are important** to understanding the content.

#### Constraints and Limitations

- **Omitting `alt` entirely** fails WCAG 1.1.1 and causes some screen readers to announce the file name.
- **Using `alt="image"` or `alt="photo"`** is unhelpful and fails accessibility best practices.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Meaningful `alt` Text

**Setup Guide**: Save as `alt-meaningful.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Meaningful Alt Text</title>
</head>
<body>
  <p>
    Our new office features
    <img src="office.jpg" alt="An open-plan office with standing desks and large windows">
    and plenty of natural light.
  </p>
</body>
</html>
```

**Expected output**: A screen reader announces the image as “An open-plan office with standing desks and large windows,” conveying the same meaning as the visual content.

**Why this output occurs**: The `alt` text describes the purpose and content of the image in a way that is useful to someone who cannot see it.

---

#### Example 2: Decorative Image with `alt=""`

**Setup Guide**: Save as `alt-decorative.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Decorative Image</title>
</head>
<body>
  <p>
    <img src="divider.png" alt="">
    Welcome to our site.
  </p>
</body>
</html>
```

**Expected output**: The decorative divider image is ignored by screen readers, and the text “Welcome to our site” is announced normally.

**Why this output occurs**: The empty `alt=""` explicitly marks the image as decorative, so assistive technologies skip it.

---

#### Example 3: Image Link with Descriptive `alt`

**Setup Guide**: Save as `alt-link.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image Link</title>
</head>
<body>
  <a href="/">
    <img src="logo.svg" alt="Example Company homepage">
  </a>
</body>
</html>
```

**Expected output**: The link is announced as “Example Company homepage, link” because the image’s `alt` text becomes the link’s accessible name.

**Why this output occurs**: When an image is the only content of a link, its `alt` text provides the link purpose; the `alt` must describe where the link goes.

---

### Real-World Cases

1. **Product images**: `alt` describes the product for screen-reader users.
2. **Infographics**: `alt` summarises the key data point the chart conveys.
3. **Decorative graphics**: `alt=""` prevents noise for assistive technologies.

---

### References

- W3C — Technique H37: Using `alt` attributes on `img` elements — https://www.w3.org/WAI/WCAG21/Techniques/html/H37
- MDN Web Docs — `<img>`: `alt` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img#alt
- WCAG — Understanding SC 1.1.1: Non-text Content — https://www.w3.org/WAI/WCAG21/Understanding/non-text-content

---

## 5. Image Best Practices

### Definitions

**Core Definition**
Image best practices are guidelines for embedding images that are accessible, performant, responsive, and maintainable — covering `alt` text, intrinsic dimensions, lazy loading, responsive sources, and format selection.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG 1.1.1 and Technique H37, and Core Web Vitals guidance from Chrome for Developers. They encompass providing meaningful `alt` text (or `alt=""` for decorative images), setting `width` and `height` to prevent CLS, using `loading="lazy"` for below-the-fold images and `fetchpriority="high"` for LCP images, and using `srcset`/`sizes` or `<picture>` for responsive images.

**Beginner-Friendly Explanation**
Best practices help you use images well: describe them for screen readers, set their size so the page does not jump, load them efficiently, and give the browser different versions for different screens.

---

### Purposes

- **To ensure accessibility** through proper `alt` text.
- **To prevent layout shift** by reserving space with `width`/`height`.
- **To improve loading performance** with lazy loading and responsive sources.
- **To support responsive design** across devices and pixel densities.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Provide `alt` text** | Describe the image’s meaning; use `alt=""` for decorative images. |
| **Set `width` and `height`** | Reserve layout space and prevent CLS. |
| **Use `loading="lazy"` below the fold** | Defer off-screen images; do not lazy-load the LCP image. |
| **Use `fetchpriority="high"` for LCP images** | Prioritise the largest contentful paint image. |
| **Use `srcset` and `sizes`** | Provide responsive candidates; always pair `srcset` with `sizes`. |
| **Use `<picture>` for art direction or format negotiation** | Offer AVIF/WebP with fallbacks. |
| **Use `decoding="async"`** | Allow non-blocking image decoding. |

#### Syntax Rules

1. **Accessibility first**: Every meaningful image needs `alt` text; decorative images need `alt=""`.
2. **Performance awareness**: Always set `width` and `height`; lazy-load below-the-fold images; prioritise LCP images.
3. **Responsive discipline**: Use `srcset`/`sizes` or `<picture>` instead of serving desktop-sized images to mobile devices.

#### Constraints and Limitations

- **`loading="lazy"` on the LCP image** harms performance; use `loading="eager"` or omit `loading` for above-the-fold heroes.
- **Omitting `sizes` with `srcset`** causes the browser to assume `100vw` and may download the largest image.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Fully Optimized Responsive Image

**Setup Guide**: Save as `best-practice-image.html`. Provide images at multiple widths.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Responsive Image Best Practices</title>
</head>
<body>
  <img
    src="hero-800.jpg"
    srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="A mountain landscape with a lake in the foreground"
    width="800"
    height="600"
    fetchpriority="high"
    decoding="async">
</body>
</html>
```

**Expected output**: On a narrow screen, the browser may load `hero-400.jpg`; on a wider screen, it may load `hero-800.jpg` or `hero-1200.jpg`, depending on the `sizes` conditions and device pixel ratio. The image reserves space via `width`/`height`, and the LCP image is prioritised with `fetchpriority="high"`.

**Why this output occurs**: `srcset` and `sizes` let the browser choose the most appropriate candidate; `width` and `height` prevent CLS; `fetchpriority="high"` signals that the image is critical for the LCP.

---

#### Example 2: Format Negotiation with `<picture>`

**Setup Guide**: Save as `best-practice-picture.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Picture Best Practices</title>
</head>
<body>
  <picture>
    <!-- Modern format for supporting browsers -->
    <source type="image/avif" srcset="hero.avif">
    <source type="image/webp" srcset="hero.webp">

    <!-- Fallback with dimensions and priority -->
    <img src="hero.jpg" alt="A city skyline at dusk" width="1200" height="800" fetchpriority="high">
  </picture>
</body>
</html>
```

**Expected output**: Browsers that support AVIF load `hero.avif`; browsers that support WebP but not AVIF load `hero.webp`; browsers that support neither load `hero.jpg`.

**Why this output occurs**: The `<picture>` element evaluates each `<source>` in order and selects the first supported type, while the `<img>` provides the fallback and the required attributes.

---

### Real-World Cases

1. **E-commerce**: Responsive product images with `srcset`, `sizes`, and descriptive `alt` text.
2. **News sites**: LCP hero images with `fetchpriority="high"` and above-the-fold eager loading.
3. **Documentation**: Diagrams with `width`/`height` to prevent layout shift and `alt` text to convey meaning.

---

### References

- Chrome for Developers — Performance best practices — https://developer.chrome.com/docs/modern-web-guidance/guides/performance/performance
- MDN Web Docs — Responsive images — https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images
- web.dev — Serve responsive images — https://web.dev/articles/serve-responsive-images
- W3C — Technique H37: Using `alt` attributes on `img` elements — https://www.w3.org/WAI/WCAG21/Techniques/html/H37

---

## Summary Table of Image Concepts

| Concept | Key Takeaway |
|---|---|
| **Images** | Embedded visual content; the HTML standard does not mandate specific formats. |
| **`<img>`** | Void element; `src` required; `alt` required (with narrow exceptions); `width`/`height` recommended. |
| **`src`** | Specifies the image URL; relative URLs resolve against the base URL. |
| **`alt`** | Provides the textual alternative; mandatory for meaningful images; `alt=""` for decorative images. |
| **Best practices** | Set `alt`; set `width`/`height`; lazy-load below-the-fold images; prioritise LCP images; use `srcset`/`sizes` or `<picture>`. |

---

## Final Notes

- **`alt` is non-negotiable**: Every meaningful image needs descriptive `alt` text; decorative images need `alt=""` so screen readers skip them.
- **`width` and `height` prevent layout shift**: Always set both attributes to reserve space and improve Cumulative Layout Shift (CLS).
- **Lazy loading is contextual**: Use `loading="lazy"` for off-screen images, but never for the LCP (largest contentful paint) image.
- **Responsive images are expected**: Use `srcset`/`sizes` or `<picture>` to serve appropriate images to different devices and pixel densities.
- **Format matters**: Prefer modern formats like AVIF and WebP with fallbacks for broad compatibility.
- **Validate and test**: Use accessibility checkers to verify `alt` text, and performance tools to verify image loading behaviour.

---

This cheat sheet has covered image fundamentals — the general concept of images, the `<img>` element, the `src` and `alt` attributes, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate image markup against the WHATWG HTML Living Standard, WCAG 1.1.1, and Core Web Vitals guidance.